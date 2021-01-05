---
title: Gérer les points de terminaison de réseau virtuel - Portail Azure - Azure Database for MariaDB
description: Créer et gérer des règles et points de terminaison de service de réseau virtuel Azure Database for MariaDB à l’aide du portail Azure
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: how-to
ms.date: 3/18/2020
ms.openlocfilehash: 5eaa7821c61010b322d8f9032c439df28c297f3d
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94540873"
---
# <a name="create-and-manage-azure-database-for-mariadb-vnet-service-endpoints-and-vnet-rules-by-using-the-azure-portal"></a>Créer et gérer des points de terminaison de service de réseau virtuel et des règles de réseau virtuel Azure Database for MariaDB à l’aide du portail Azure

Les règles et points de terminaison de service de réseau virtuel étendent l’espace d’adressage privé d’un réseau virtuel à votre serveur Azure Database for MariaDB. Pour avoir une vue d’ensemble des points de terminaison de service de réseau virtuel Azure Database for MariaDB, y compris les limitations, consultez [Utiliser des règles et points de terminaison de service de réseau virtuel pour Azure Database for MariaDB](concepts-data-access-security-vnet.md). Les points de terminaison de service de réseau virtuel sont disponibles dans toutes les régions prenant en charge Azure Database for MariaDB.

> [!NOTE]
> Les points de terminaison de service de réseau virtuel sont uniquement pris en charge pour les serveurs Usage général et Mémoire optimisée.

## <a name="create-a-vnet-rule-and-enable-service-endpoints"></a>Créer une règle de réseau virtuel et activer les points de terminaison de service

1. Dans la page du serveur MariaDB, sous le titre Paramètres, cliquez sur **Sécurité des connexions** pour ouvrir le volet Sécurité des connexions pour Azure Database for MariaDB.

2. Vérifiez que le contrôle Autoriser l’accès aux services Azure est défini sur **DÉSACTIVÉ**.

> [!Important]
> S’il est défini sur ACTIVÉ, votre serveur Azure MariaDB Database acceptera les communications provenant de n’importe quel sous-réseau. En termes de sécurité, le fait de laisser le contrôle ACTIVÉ peut avoir pour effet de multiplier excessivement les accès. La fonctionnalité de points de terminaison de service de réseau virtuel Microsoft Azure, associée à la fonctionnalité de règle de réseau virtuel de Azure Database for MariaDB, peuvent ensemble réduire votre surface d’exposition de sécurité.

3. Cliquez ensuite sur **+ Ajout d’un réseau virtuel existant**. Si vous ne disposez d’aucun réseau virtuel, vous pouvez en créer un en cliquant sur **+ Créer un nouveau réseau virtuel**. Voir [Démarrage rapide : Créer un réseau virtuel à l’aide du portail Azure](../virtual-network/quick-create-portal.md)

   ![Portail Azure - cliquez sur Sécurité des connexions](./media/howto-manage-vnet-portal/1-connection-security.png)

4. Entrez un nom de règle de réseau virtuel, sélectionnez l’abonnement, le réseau virtuel et le nom du sous-réseau, puis cliquez sur **Activer**. Les points de terminaison de service de réseau virtuel sont alors automatiquement activés sur le sous-réseau à l’aide du nom de service **Microsoft.SQL**.

   ![Portail Azure - Configurer le réseau virtuel](./media/howto-manage-vnet-portal/2-configure-vnet.png)

   Le compte doit avoir les autorisations nécessaires pour créer un réseau virtuel et un point de terminaison de service.

   Les points de terminaison de service peuvent être configurés indépendamment sur les réseaux virtuels par un utilisateur avec accès en écriture au réseau virtuel.
    
   Pour sécuriser les ressources du service Azure pour un réseau virtuel, l’utilisateur doit disposer des autorisations sur « Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/ » pour les sous-réseaux à ajouter. Cette autorisation est incluse par défaut dans les rôles d’administrateur de service fédérés et peut être modifiée en créant des rôles personnalisés.
    
   Apprenez-en davantage sur les [rôles intégrés](../role-based-access-control/built-in-roles.md) et l’affectation d’autorisations spécifiques aux [rôles personnalisés](../role-based-access-control/custom-roles.md).
    
   Les réseaux virtuels et les ressources du service Azure peuvent être dans des abonnements identiques ou différents. Si le réseau virtuel et les ressources de service Azure se trouvent dans différents abonnements, les ressources doivent être sous le même locataire Active Directory (AD). Assurez-vous que le fournisseur de ressources **Microsoft.Sql** est inscrit pour les deux abonnements. Pour plus d’informations, reportez-vous à [resource-manager-registration][resource-manager-portal]

   > [!IMPORTANT]
   > Il est vivement recommandé de lire cet article sur les configurations de point de terminaison de service et les considérations à prendre en compte avant de configurer les points de terminaison de service. **Point de terminaison de service de réseau virtuel :** un [point de terminaison de service de réseau virtuel](../virtual-network/virtual-network-service-endpoints-overview.md) est un sous-réseau dont les valeurs de propriétés incluent un ou plusieurs noms de type de service Azure formels. Les points de terminaison de service de réseau virtuel utilisent le nom de type de service **Microsoft.Sql**, qui fait référence au service Azure nommé SQL Database. Ce nom de service s’applique également aux services Azure SQL Database, Azure Database for MariaDB et MySQL. Il est important de noter que lorsque le nom de service **Microsoft.Sql** est appliqué à un point de terminaison de service de réseau virtuel, il configure le trafic de point de terminaison de service pour l’ensemble des services Azure Database, y compris les serveurs Azure SQL Database, Azure Database pour PostgreSQL, Azure Database for MariaDB et Azure Database pour MySQL dans le sous-réseau.
   > 

5. Une fois l’activation effectuée, cliquez sur **OK** : vous verrez que les points de terminaison de service de réseau virtuel sont activés en même temps qu’une règle de réseau virtuel.

   ![Points de terminaison de service de réseau virtuel activés et règle de réseau virtuel créée](./media/howto-manage-vnet-portal/3-vnet-service-endpoints-enabled-vnet-rule-created.png)

## <a name="next-steps"></a>Étapes suivantes
- Découvrez plus en détail la [configuration de SSL dans Azure Database for MariaDB](howto-configure-ssl.md).
- De la même manière, vous pouvez créer un script pour [activer des points de terminaison de service de réseau virtuel et créer une règle de réseau virtuel pour Azure Database for MariaDB en utilisant Azure CLI](howto-manage-vnet-cli.md).

<!-- Link references, to text, Within this same GitHub repo. --> 
[resource-manager-portal]: ../azure-resource-manager/management/resource-providers-and-types.md