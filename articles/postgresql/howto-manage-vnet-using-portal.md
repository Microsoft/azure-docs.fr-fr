---
title: Utiliser des règles de réseau virtuel - Portail Azure - Azure Database pour PostgreSQL - Serveur unique
description: Création et gestion des points de terminaison de service de réseau virtuel et des règles Azure Database pour PostgreSQL - Serveur unique à l’aide du Portail Microsoft Azure
author: niklarin
ms.author: nlarin
ms.service: postgresql
ms.topic: how-to
ms.date: 5/6/2019
ms.openlocfilehash: 377f8eba179253ca6187b10a22970d0eadcda2f6
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92489827"
---
# <a name="create-and-manage-vnet-service-endpoints-and-vnet-rules-in-azure-database-for-postgresql---single-server-by-using-the-azure-portal"></a>Création et gestion des points de terminaison de service et des règles de réseau virtuel dans Azure Database pour PostgreSQL - Serveur unique à l’aide du Portail Microsoft Azure
Les règles et points de terminaison de service de réseau virtuel étendent l’espace d’adressage privé d’un réseau virtuel à votre serveur Azure Database pour PostgreSQL. Pour une vue d’ensemble des points de terminaison de service de réseau virtuel Azure Database pour PostgreSQL, y compris les limitations, consultez [Use Virtual Network service endpoints and rules for Azure Database for PostgreSQL](concepts-data-access-and-security-vnet.md) (Utiliser des règles et points de terminaison de service de réseau virtuel pour Azure Database pour PostgreSQL). Les points de terminaison de service de réseau virtuel sont disponibles dans toutes les régions prises en charge pour Azure Database pour PostgreSQL.

> [!NOTE]
> Les points de terminaison de service de réseau virtuel sont uniquement pris en charge pour les serveurs Usage général et Mémoire optimisée.
> En cas de peering de réseau virtuel, si le trafic passe par une passerelle de réseau virtuel commune avec des points de terminaison de service et qu’il est supposé transiter par l’homologue, créez une règle ACL/VNet pour permettre aux machines virtuelles Azure du réseau virtuel de la passerelle d’accéder au serveur Azure Database pour PostgreSQL.


## <a name="create-a-vnet-rule-and-enable-service-endpoints-in-the-azure-portal"></a>Créer une règle de réseau virtuel et activer les points de terminaison de service dans le portail Azure

1. Dans la page du serveur PostgreSQL, sous le titre Paramètres, cliquez sur **Sécurité des connexions** afin d’ouvrir le volet correspondant pour Azure Database pour PostgreSQL. 

2. Vérifiez que le contrôle Autoriser l’accès aux services Azure est défini sur **DÉSACTIVÉ** .

> [!Important]
> Si vous maintenez le contrôle ACTIVÉ, votre serveur Azure PostgreSQL Database acceptera les communications à partir de n’importe quel sous-réseau. En termes de sécurité, le fait de laisser le contrôle ACTIVÉ peut avoir pour effet de multiplier excessivement les accès. La fonctionnalité de points de terminaison de service de réseau virtuel Microsoft Azure, associée à la fonctionnalité de règle de réseau virtuel d’Azure Database pour PostgreSQL, réduisent votre surface d’exposition de sécurité.

3. Cliquez ensuite sur **+ Ajout d’un réseau virtuel existant** . Si vous ne disposez d’aucun réseau virtuel, vous pouvez en créer un en cliquant sur **+ Créer un nouveau réseau virtuel** . Voir [Démarrage rapide : Créer un réseau virtuel à l’aide du portail Azure](../virtual-network/quick-create-portal.md)

   :::image type="content" source="./media/howto-manage-vnet-using-portal/1-connection-security.png" alt-text="Portail Azure - cliquez sur Sécurité des connexions":::

4. Entrez un nom de règle de réseau virtuel, sélectionnez l’abonnement, le réseau virtuel et le nom du sous-réseau, puis cliquez sur **Activer** . Les points de terminaison de service de réseau virtuel sont alors automatiquement activés sur le sous-réseau à l’aide du nom de service **Microsoft.SQL** .

   :::image type="content" source="./media/howto-manage-vnet-using-portal/2-configure-vnet.png" alt-text="Portail Azure - cliquez sur Sécurité des connexions":::

    Le compte doit avoir les autorisations nécessaires pour créer un réseau virtuel et un point de terminaison de service.

    Les points de terminaison de service peuvent être configurés indépendamment sur les réseaux virtuels par un utilisateur avec accès en écriture au réseau virtuel.
    
    Pour sécuriser les ressources du service Azure pour un réseau virtuel, l’utilisateur doit disposer des autorisations sur « Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/ » pour les sous-réseaux à ajouter. Cette autorisation est incluse par défaut dans les rôles d’administrateur de service fédérés et peut être modifiée en créant des rôles personnalisés.
    
    Apprenez-en davantage sur les [rôles intégrés](../role-based-access-control/built-in-roles.md) et l’affectation d’autorisations spécifiques aux [rôles personnalisés](../role-based-access-control/custom-roles.md).
    
    Les réseaux virtuels et les ressources du service Azure peuvent être dans des abonnements identiques ou différents. Si le réseau virtuel et les ressources de service Azure se trouvent dans différents abonnements, les ressources doivent être sous le même locataire Active Directory (AD). Assurez-vous que le fournisseur de ressources **Microsoft.Sql** est inscrit pour les deux abonnements. Pour plus d’informations, reportez-vous à [resource-manager-registration][resource-manager-portal]

   > [!IMPORTANT]
   > Il est vivement recommandé de lire cet article sur les configurations de point de terminaison de service et les considérations à prendre en compte avant de configurer les points de terminaison de service. **Point de terminaison de service de réseau virtuel :** un [point de terminaison de service de réseau virtuel](../virtual-network/virtual-network-service-endpoints-overview.md) est un sous-réseau dont les valeurs de propriétés incluent un ou plusieurs noms de type de service Azure formels. Les points de terminaison de service de réseau virtuel utilisent le nom de type de service **Microsoft.Sql** , qui fait référence au service Azure nommé SQL Database. Ce nom de service s’applique également aux services Azure SQL Database, Azure Database pour PostgreSQL et MySQL. Il est important de noter que lorsque le nom de service **Microsoft.Sql** est appliqué à un point de terminaison de service de réseau virtuel, il configure le trafic de point de terminaison de service pour l’ensemble des services Azure Database, y compris les serveurs Azure SQL Database, Azure Database pour PostgreSQL et Azure Database pour MySQL sur le sous-réseau. 
   > 

5. Une fois l’activation effectuée, cliquez sur **OK** : vous verrez que les points de terminaison de service de réseau virtuel sont activés en même temps qu’une règle de réseau virtuel.

   :::image type="content" source="./media/howto-manage-vnet-using-portal/3-vnet-service-endpoints-enabled-vnet-rule-created.png" alt-text="Portail Azure - cliquez sur Sécurité des connexions":::

## <a name="next-steps"></a>Étapes suivantes
- De la même manière, vous pouvez créer un script pour [activer des point de terminaison de service de réseau virtuel et créer une règle de réseau virtuel pour Azure Database pour PostgreSQL en utilisant Azure CLI](howto-manage-vnet-using-cli.md).
- Pour vous aider à vous connecter à un serveur Azure Database pour PostgreSQL, consultez la rubrique [Bibliothèques de connexions pour Azure Database pour PostgreSQL](./concepts-connection-libraries.md)

<!-- Link references, to text, Within this same GitHub repo. --> 
[resource-manager-portal]: ../azure-resource-manager/management/resource-providers-and-types.md