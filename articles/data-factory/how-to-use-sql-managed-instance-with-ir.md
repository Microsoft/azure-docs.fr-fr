---
title: Utiliser Azure SQL Managed Instance avec Azure SQL Server Integration Services (SSIS) dans Azure Data Factory
description: Apprenez à utiliser Azure SQL Managed Instance avec SQL Server Integration Services (SSIS) dans Azure Data Factory.
services: data-factory
documentationcenter: ''
author: chugugrace
ms.author: chugu
ms.reviewer: ''
manager: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 4/15/2020
ms.openlocfilehash: 86bff161e29384b10030ed3d524301f6dea6037e
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92634162"
---
# <a name="use-azure-sql-managed-instance-with-sql-server-integration-services-ssis-in-azure-data-factory"></a>Utiliser Azure SQL Managed Instance avec SQL Server Integration Services (SSIS) dans Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-xxx-md.md)]

Vous pouvez maintenant déplacer vos projets, packages et charges de travail SQL Server Integration Services (SSIS) vers le cloud Azure. Déployez, exécutez et gérez des projets et des packages SSIS dans Azure SQL Database ou SQL Managed Instance avec des outils habituels, comme SQL Server Management Studio (SSMS). Cet article met en évidence les aspects spécifiques suivants lors de l'utilisation d'Azure SQL Managed Instance avec le runtime d'intégration (IR) Azure-SSIS :

- [Approvisionner un runtime Azure-SSIS IR avec le catalogue SSIS (SSISDB) hébergé par Azure SQL Managed Instance](#provision-azure-ssis-ir-with-ssisdb-hosted-by-azure-sql-managed-instance)
- [Exécuter des packages SSIS avec le travail de l’agent Azure SQL Managed Instance](how-to-invoke-ssis-package-managed-instance-agent.md)
- [Nettoyer les journaux SSISDB avec le travail de l’agent Azure SQL Managed Instance](#clean-up-ssisdb-logs)
- [Basculement du runtime Azure-SSIS IR avec Azure SQL Managed Instance](configure-bcdr-azure-ssis-integration-runtime.md#azure-ssis-ir-failover-with-a-sql-managed-instance)
- [Migrer des charges de travail SSIS locales vers SSIS dans ADF avec Azure SQL Managed Instance en tant que destination des charges de travail de base de données](scenario-ssis-migration-overview.md#azure-sql-managed-instance-as-database-workload-destination)

## <a name="provision-azure-ssis-ir-with-ssisdb-hosted-by-azure-sql-managed-instance"></a>Approvisionner un runtime Azure-SSIS IR avec SSISDB hébergé par Azure SQL Managed Instance

### <a name="prerequisites"></a>Prérequis

1. [Activez Azure Active Directory (Azure AD) sur Azure SQL Managed Instance](enable-aad-authentication-azure-ssis-ir.md#configure-azure-ad-authentication-for-azure-sql-managed-instance) lorsque vous choisissez l'authentification Azure Active Directory.

1. Choisissez comment connecter l’instance gérée SQL, sur un point de terminaison privé ou sur un point de terminaison public :

    - Sur un point de terminaison privé (recommandé)

        1. Choisissez le réseau virtuel à joindre à Azure-SSIS IR :
            - Dans le même réseau virtuel que l’instance gérée, avec un **sous-réseau différent** .
            - Dans un autre réseau virtuel que l’instance gérée, via l’appairage de réseaux virtuels (qui est limité à la même région en raison de contraintes d’appairage des réseaux virtuels globaux) ou via une connexion de réseau virtuel à réseau virtuel.

            Pour plus d'informations sur la connectivité de l'instance gérée SQL, consultez [Connecter votre application à Azure SQL Managed Instance](https://review.docs.microsoft.com/azure/sql-database/sql-database-managed-instance-connect-app).

        1. [Configurer un réseau virtuel](#configure-virtual-network).

    - Sur un point de terminaison public

        Les instances gérées Azure SQL Managed Instance peuvent fournir une connectivité sur des [points de terminaison publics](../azure-sql/managed-instance/public-endpoint-configure.md). Les exigences entrantes et sortantes doivent être respectées pour autoriser le trafic entre l’instance gérée SQL et Azure-SSIS IR :

        - Lorsque Azure-SSIS IR n’est pas à l’intérieur d’un réseau virtuel (recommandé)

            **Exigences entrantes de l’instance gérée SQL** , pour autoriser le trafic entrant à partir d’Azure-SSIS IR.

            | Protocole de transfert | Source | Source port range (Plage de ports sources) | Destination | Destination port range |
            |---|---|---|---|---|
            |TCP|Balise de service Azure Cloud|*|VirtualNetwork|3342|

            Pour plus d’informations, consultez [Autoriser le trafic du point de terminaison public sur le groupe de sécurité réseau](../azure-sql/managed-instance/public-endpoint-configure.md#allow-public-endpoint-traffic-on-the-network-security-group).

        - Lorsque Azure-SSIS IR est à l’intérieur d’un réseau virtuel

            Il existe un scénario particulier où l’instance gérée SQL se trouve dans une région qu’Azure-SSIS IR ne prend pas en charge. Azure-SSIS IR se trouve à l’intérieur d’un réseau virtuel sans appairage de réseaux virtuels en raison d’une limitation de l’appairage des réseaux virtuels globaux. Dans ce scénario, **Azure-SSIS IR est à l’intérieur d’un réseau virtuel** et se connecte à l’instance gérée SQL **sur un point de terminaison public** . Utilisez les règles de groupe de sécurité réseau (NSG) pour autoriser le trafic entre l’instance gérée SQL et Azure-SSIS IR :

            1. **Exigences entrantes de l’instance gérée SQL** , pour autoriser le trafic entrant à partir d’Azure-SSIS IR.

                | Protocole de transfert | Source | Source port range (Plage de ports sources) | Destination |Destination port range |
                |---|---|---|---|---|
                |TCP|Adresse IP statique de Azure-SSIS IR <br> Pour plus d’informations, consultez [Apporter votre propre adresse IP publique pour Azure-SSIS IR](join-azure-ssis-integration-runtime-virtual-network.md#publicIP).|*|VirtualNetwork|3342|

             1. **Exigences sortantes d’Azure-SSIS IR** pour autoriser le trafic sortant vers l’instance gérée SQL.

                | Protocole de transfert | Source | Source port range (Plage de ports sources) | Destination |Destination port range |
                |---|---|---|---|---|
                |TCP|VirtualNetwork|*|[Adresse IP du point de terminaison public de l’instance gérée SQL](../azure-sql/managed-instance/management-endpoint-find-ip-address.md)|3342|

### <a name="configure-virtual-network"></a>Configurer un réseau virtuel

1. **Autorisation utilisateur** . L’utilisateur qui crée Azure-SSIS IR doit disposer de l’[attribution de rôle](../role-based-access-control/role-assignments-list-portal.md#list-role-assignments-for-a-user-at-a-scope) au moins sur la ressource Azure Data Factory avec l’une des options ci-dessous :

    - Utilisez le rôle Contributeur de réseaux intégré. Ce rôle inclut l’autorisation _Microsoft.Network/\*_ , mais dispose d’une étendue plus large que nécessaire.
    - Créez un rôle personnalisé qui inclut uniquement l’autorisation _Microsoft.Network/virtualNetworks/\*/join/action_ nécessaire. Si vous voulez également apporter vos propres adresses IP publiques pour Azure-SSIS IR tout en le joignant à un réseau virtuel Azure Resource Manager, incluez également l’autorisation _Microsoft.Network/publicIPAddresses/*/join/action_ dans le rôle.

1. **Réseau virtuel** .

    1. Vérifiez que le groupe de ressources du réseau virtuel peut créer et supprimer certaines ressources réseau Azure.

        Le runtime d’intégration Azure-SSIS doit créer certaines ressources réseau sous le même groupe de ressources que le réseau virtuel. Ces ressources incluent :
        - Un équilibreur de charge Azure, dont le nom est *\<Guid>-azurebatch-cloudserviceloadbalancer*
        - Un groupe de sécurité réseau, dont le nom est *\<Guid>-azurebatch-cloudservicenetworksecuritygroup
        - Une adresse IP publique Azure, avec le nom -azurebatch-cloudservicepublicip

        Ces ressources sont créées au démarrage de votre Azure-SSIS IR. Elles sont supprimées lorsqu’il est arrêté. Pour éviter de bloquer l’arrêt du runtime d’intégration Azure-SSIS IR, ne réutilisez pas ces ressources réseau dans vos autres ressources.

    1. Vérifiez qu’il n’existe aucun [verrou de ressource](../azure-resource-manager/management/lock-resources.md) sur le groupe de ressources/abonnement auquel appartient le réseau virtuel. Si vous configurez un verrou en lecture seule ou de suppression, le démarrage et l’arrêt du runtime d’intégration Azure-SSIS IR échoueront ou celui-ci ne répondra plus.

    1. Vérifiez qu’aucune stratégie Azure n’empêche la création des ressources suivantes sous le groupe de ressources/abonnement auquel appartient le réseau virtuel :
        - Microsoft.Network/LoadBalancers
        - Microsoft.Network/NetworkSecurityGroups

    1. Autorisez le trafic sur la règle de groupe de sécurité réseau (NSG) pour autoriser le trafic entre l’instance gérée SQL et Azure-SSIS IR, ainsi que le trafic requis par Azure-SSIS IR.
        1. **Exigences entrantes de l’instance gérée SQL** , pour autoriser le trafic entrant à partir d’Azure-SSIS IR.

            | Protocole de transfert | Source | Source port range (Plage de ports sources) | Destination | Destination port range | Commentaires |
            |---|---|---|---|---|---|
            |TCP|VirtualNetwork|*|VirtualNetwork|1433, 11000-11999|Si votre stratégie de connexion de serveur SQL Database est définie sur **Proxy** au lieu de **Rediriger** , seul le port 1433 est nécessaire.|

        1. **Exigences sortantes d’Azure-SSIS IR** , pour autoriser le trafic sortant vers l’instance gérée SQL, et d’autres types de trafic requis par Azure-SSIS IR.

        | Protocole de transfert | Source | Source port range (Plage de ports sources) | Destination | Destination port range | Commentaires |
        |---|---|---|---|---|---|
        | TCP | VirtualNetwork | * | VirtualNetwork | 1433, 11000-11999 |Autorisez le trafic sortant vers l’instance gérée SQL. Si la stratégie de connexion est définie sur **Proxy** au lieu de **Rediriger** , seul le port 1433 est nécessaire. |
        | TCP | VirtualNetwork | * | AzureCloud | 443 | Les nœuds de votre runtime d’intégration Azure SSIS IR sur le réseau virtuel utilisent ce port pour accéder aux services Azure comme Stockage Azure et Azure Event Hubs. |
        | TCP | VirtualNetwork | * | Internet | 80 | (Facultatif) Les nœuds de votre Azure-SSIS IR dans le réseau virtuel utilisent ce port pour télécharger une liste de révocation de certificats à partir d’Internet. Si vous bloquez ce trafic, vous risquez de constater une dégradation des performances lors du démarrage d’IR et de perdre la possibilité de vérifier la liste de révocation de certificats pour l’utilisation des certificats. Si vous souhaitez restreindre la destination à certains noms de domaine complets (FQDN), reportez-vous à [Utiliser Azure ExpressRoute ou un itinéraire défini par l’utilisateur](./join-azure-ssis-integration-runtime-virtual-network.md#route).|
        | TCP | VirtualNetwork | * | Stockage | 445 | (Facultatif) Cette règle n’est obligatoire que quand vous souhaitez exécuter un package SSIS stocké dans Azure Files. |
        |||||||

        1. **Exigence entrante d’Azure-SSIS IR** , pour autoriser le trafic requis par Azure-SSIS IR.

        | Protocole de transfert | Source | Source port range (Plage de ports sources) | Destination | Destination port range | Commentaires |
        |---|---|---|---|---|---|
        | TCP | BatchNodeManagement | * | VirtualNetwork | 29876, 29877 (si vous joignez le runtime d’intégration à un réseau virtuel Azure Resource Manager) <br/><br/>10100, 20100, 30100 (si vous joignez le runtime d’intégration à un réseau virtuel classique)| Le service Data Factory utilise ces ports pour communiquer avec les nœuds de votre runtime d’intégration Azure SSIS IR sur le réseau virtuel. <br/><br/> Que vous créiez ou non un groupe de sécurité réseau au niveau du sous-réseau, Data Factory configure toujours un groupe de sécurité réseau au niveau des cartes d’interface réseau (NIC) connectées aux machines virtuelles qui hébergent le runtime d’intégration Azure-SSIS. Ce groupe de sécurité réseau au niveau de la carte réseau n’autorise que le trafic entrant provenant d’adresses IP Data Factory sur les ports spécifiés. Même si vous ouvrez ces ports pour le trafic Internet au niveau du sous-réseau, le trafic provenant d’adresses IP qui ne sont pas des adresses IP Data Factory est bloqué au niveau de la carte réseau. |
        | TCP | CorpNetSaw | * | VirtualNetwork | 3389 | (Facultatif) Cette règle est obligatoire uniquement lorsque le support technique de Microsoft demande une ouverture pour le dépannage avancé d’un client qui peut être fermé juste après le dépannage. La balise de service **CorpNetSaw** autorise uniquement les stations de travail à accès sécurisé sur le réseau d’entreprise Microsoft à utiliser le bureau à distance. Par ailleurs, cette balise de service ne peut pas être sélectionnée à partir du portail et n’est disponible que via Azure PowerShell ou Azure CLI. <br/><br/> Dans le groupe de sécurité réseau au niveau carte réseau, le port 3389 est ouvert par défaut et nous vous autorisons à contrôler le port 3389 dans le groupe de sécurité réseau au niveau sous-réseau, tandis qu’Azure-SSIS IR a interdit le port 3389 sortant par défaut dans la règle de pare-feu Windows sur chaque nœud IR pour la protection. |
        |||||||

    1. Consultez [Configuration du réseau virtuel](join-azure-ssis-integration-runtime-virtual-network.md#virtual-network-configuration) pour plus d’informations :
        - Si vous apportez vos propres adresses IP publiques pour le runtime Azure-SSIS IR
        - Si vous utilisez votre propre serveur DNS (Domain Name System)
        - Si vous utilisez Azure ExpressRoute ou une route définie par l’utilisateur (UDR)
        - Si vous utilisez un runtime Azure-SSIS IR personnalisé

### <a name="provision-azure-ssis-integration-runtime"></a>Approvisionner Azure-SSIS Integration Runtime

1. Sélectionnez le point de terminaison privé ou public de l’instance gérée SQL.

    Lors de [l’approvisionnement d’Azure-SSIS IR](create-azure-ssis-integration-runtime.md#provision-an-azure-ssis-integration-runtime) dans l’application ADF ou le Portail Azure, sur la page Paramètres SQL, utilisez le **point de terminaison privé** ou le **point de terminaison public** de l’instance gérée SQL lors de la création du catalogue SSIS (SSISDB).

    Le nom d’hôte du point de terminaison public présente le format <mi_name>.public.<dns_zone>.database.windows.net et le port utilisé pour la connexion est 3342.  

    ![La capture d’écran montre le paramétrage du runtime d’intégration avec l’option Créer un catalogue SSIS sélectionnée et le point de terminaison du serveur de base de données du catalogue entré.](./media/how-to-use-sql-managed-instance-with-ir/catalog-public-endpoint.png)

1. Sélectionnez l’authentification Azure AD lorsque cela est nécessaire.

    ![catalog-public-endpoint](./media/how-to-use-sql-managed-instance-with-ir/catalog-aad.png)

    Pour plus d'informations sur l'activation de l'authentification Azure AD, consultez [Activer Azure AD sur Azure SQL Managed Instance](enable-aad-authentication-azure-ssis-ir.md#configure-azure-ad-authentication-for-azure-sql-managed-instance).

1. Joignez Azure-SSIS IR au réseau virtuel lorsque cela est nécessaire.

    Sur la page Paramètres avancés, sélectionnez le réseau virtuel et le sous-réseau à joindre.
    
    Lorsque vous êtes dans le même réseau virtuel que l’instance gérée SQL, choisissez un **sous-réseau différent** de celui de l’instance gérée SQL. 

    Pour plus d’informations sur la façon de joindre Azure-SSIS IR à un réseau virtuel, consultez [Joindre un runtime d’intégration Azure-SSIS à un réseau virtuel](join-azure-ssis-integration-runtime-virtual-network.md).

    ![La capture d’écran montre les paramètres avancés de configuration du runtime d’intégration, qui vous permettent de sélectionner un réseau virtuel que votre runtime doit rejoindre.](./media/how-to-use-sql-managed-instance-with-ir/join-virtual-network.png)

Pour plus d’informations sur la création d’un Azure-SSIS IR, consultez [Créer un runtime d’intégration Azure-SSIS dans Azure Data Factory](create-azure-ssis-integration-runtime.md#provision-an-azure-ssis-integration-runtime).

## <a name="clean-up-ssisdb-logs"></a>Nettoyer les journaux SSISDB

La stratégie de rétention des journaux SSISDB est définie par les propriétés ci-dessous dans [catalog.catalog_properties](/sql/integration-services/system-views/catalog-catalog-properties-ssisdb-database?view=sql-server-ver15) :

- OPERATION_CLEANUP_ENABLED

    Quand la valeur est TRUE, les détails et les messages de l’opération plus anciens que RETENTION_WINDOW (jours) sont supprimés du catalogue. Lorsque la valeur est FALSE, tous les détails et les messages de l'opération sont stockés dans le catalogue. Remarque : un travail SQL Server effectue le nettoyage de l'opération.

- RETENTION_WINDOW

    Nombre de jours que les détails et les messages de l'opération sont stockés dans le catalogue. Lorsque la valeur est -1, la période de conservation est infinie. Remarque : Si aucun nettoyage n’est souhaité, affectez à OPERATION_CLEANUP_ENABLED la valeur FALSE.

Pour supprimer les journaux d’activité SSISDB en dehors de la fenêtre de rétention définie par l’administrateur, vous pouvez déclencher la procédure stockée `[internal].[cleanup_server_retention_window_exclusive]`. Si vous le souhaitez, vous pouvez planifier l’exécution du travail de l’agent de l’instance gérée SQL pour déclencher la procédure stockée.

## <a name="next-steps"></a>Étapes suivantes

- [Exécuter des packages SSIS avec le travail de l’agent Azure SQL Managed Instance](how-to-invoke-ssis-package-managed-instance-agent.md)
- [Configurer la continuité d’activité et la reprise d’activité (BCDR)](configure-bcdr-azure-ssis-integration-runtime.md)
- [Migrer des charges de travail SSIS locales vers SSIS dans ADF](scenario-ssis-migration-overview.md)