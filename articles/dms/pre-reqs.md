---
title: Prérequis pour Azure Database Migration Service
description: Découvrez une vue d’ensemble des conditions préalables pour l’utilisation d’Azure Database Migration Service pour effectuer des migrations de bases de données.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: conceptual
ms.date: 02/25/2020
ms.openlocfilehash: f6438fb1c21ce248f6e1b766e7e10cc79043db9f
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94961581"
---
# <a name="overview-of-prerequisites-for-using-the-azure-database-migration-service"></a>Vue d’ensemble des conditions préalables pour l’utilisation d’Azure Database Migration Service

Il existe plusieurs conditions préalables requises pour vous garantir qu’Azure Database Migration Service s’exécute correctement lors de la migration de bases de données. Certaines des conditions préalables s’appliquent à tous les scénarios (paires source-cible) pris en charge par le service, tandis que d’autres sont propres à un scénario spécifique.

Les conditions préalables relatives à l’utilisation d’Azure Database Migration Service sont répertoriées dans les sections suivantes.

## <a name="prerequisites-common-across-migration-scenarios"></a>Conditions préalables communes dans l’ensemble des scénarios de migration

Les conditions préalables associées à Azure Database Migration Service communes à tous les scénarios de migration pris en charge incluent le besoin de :

* Créez un Réseau virtuel Microsoft Azure pour Azure Database Migration Service à l’aide du modèle de déploiement Azure Resource Manager, qui fournit une connectivité site à site à vos serveurs sources locaux via [ExpressRoute](../expressroute/expressroute-introduction.md) ou un [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).
* Vérifiez que les règles de groupe de sécurité de votre réseau virtuel ne bloquent pas les ports de communication suivants : 443, 53, 9354, 445, 12000. Pour plus d’informations sur le filtrage du trafic de groupe de sécurité réseau de réseau virtuel, consultez l’article [Filtrer le trafic avec les groupes de sécurité réseau](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Lorsque vous utilisez une appliance de pare-feu devant vos bases de données sources, vous devrez peut-être ajouter des règles de pare-feu pour permettre à Azure Database Migration Service d’accéder aux bases de données sources pour la migration.
* Configurez votre [pare-feu Windows pour accéder au moteur de base de données](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Activez le protocole TCP/IP, qui est désactivé par défaut pendant l’installation de SQL Server Express, en suivant les instructions de l’article [Activer ou désactiver un protocole réseau de serveur](/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure).

    > [!IMPORTANT]
    > La création d’une instance d’Azure Database Migration Service nécessite l’accès à des paramètres de réseau virtuel qui ne font normalement pas partie du même groupe de ressources. Par conséquent, l’utilisateur qui crée une instance de DMS a besoin d’une autorisation au niveau de l’abonnement. Pour créer les rôles requis, que vous pouvez affecter selon vos besoins, exécutez le script suivant :
    >
    > ```
    >
    > $readerActions = `
    > "Microsoft.Network/networkInterfaces/ipConfigurations/read", `
    > "Microsoft.DataMigration/*/read", `
    > "Microsoft.Resources/subscriptions/resourceGroups/read"
    >
    > $writerActions = `
    > "Microsoft.DataMigration/services/*/write", `
    > "Microsoft.DataMigration/services/*/delete", `
    > "Microsoft.DataMigration/services/*/action", `
    > "Microsoft.Network/virtualNetworks/subnets/join/action", `
    > "Microsoft.Network/virtualNetworks/write", `
    > "Microsoft.Network/virtualNetworks/read", `
    > "Microsoft.Resources/deployments/validate/action", `
    > "Microsoft.Resources/deployments/*/read", `
    > "Microsoft.Resources/deployments/*/write"
    >
    > $writerActions += $readerActions
    >
    > # TODO: replace with actual subscription IDs
    > $subScopes = ,"/subscriptions/00000000-0000-0000-0000-000000000000/","/subscriptions/11111111-1111-1111-1111-111111111111/"
    >
    > function New-DmsReaderRole() {
    > $aRole = [Microsoft.Azure.Commands.Resources.Models.Authorization.PSRoleDefinition]::new()
    > $aRole.Name = "Azure Database Migration Reader"
    > $aRole.Description = "Lets you perform read only actions on DMS service/project/tasks."
    > $aRole.IsCustom = $true
    > $aRole.Actions = $readerActions
    > $aRole.NotActions = @()
    >
    > $aRole.AssignableScopes = $subScopes
    > #Create the role
    > New-AzRoleDefinition -Role $aRole
    > }
    >
    > function New-DmsContributorRole() {
    > $aRole = [Microsoft.Azure.Commands.Resources.Models.Authorization.PSRoleDefinition]::new()
    > $aRole.Name = "Azure Database Migration Contributor"
    > $aRole.Description = "Lets you perform CRUD actions on DMS service/project/tasks."
    > $aRole.IsCustom = $true
    > $aRole.Actions = $writerActions
    > $aRole.NotActions = @()
    >
    >   $aRole.AssignableScopes = $subScopes
    > #Create the role
    > New-AzRoleDefinition -Role $aRole
    > }
    > 
    > function Update-DmsReaderRole() {
    > $aRole = Get-AzRoleDefinition "Azure Database Migration Reader"
    > $aRole.Actions = $readerActions
    > $aRole.NotActions = @()
    > Set-AzRoleDefinition -Role $aRole
    > }
    >
    > function Update-DmsConributorRole() {
    > $aRole = Get-AzRoleDefinition "Azure Database Migration Contributor"
    > $aRole.Actions = $writerActions
    > $aRole.NotActions = @()
    > Set-AzRoleDefinition -Role $aRole
    > }
    >
    > # Invoke above functions
    > New-DmsReaderRole
    > New-DmsContributorRole
    > Update-DmsReaderRole
    > Update-DmsConributorRole
    > ```

## <a name="prerequisites-for-migrating-sql-server-to-azure-sql-database"></a>Prérequis pour la migration de SQL Server vers Azure SQL Database

En plus des conditions préalables d’Azure Database Migration Service qui sont communes à tous les scénarios de migration, il existe également des conditions préalables s’appliquant spécifiquement à un scénario ou un autre.

Lorsque vous utilisez Azure Database Migration Service pour exécuter SQL Server sur les migrations Azure SQL Database, outre les prérequis qui sont communs à tous les scénarios de migration, veillez à respecter les prérequis supplémentaires suivants :

* Créez une instance Azure SQL Database en suivant les indications de l’article [Création d’une base de données dans Azure SQL Database dans le portail Azure](../azure-sql/database/single-database-create-quickstart.md).
* Téléchargez et installez [Data Migration Assistant](https://www.microsoft.com/download/details.aspx?id=53595) v3.3 ou version ultérieure.
* Ouvrez votre Pare-feu Windows pour permettre à Azure Database Migration Service d’accéder au serveur SQL Server source, par défaut le port TCP 1433.
* Si vous exécutez plusieurs instances nommées de SQL Server avec des ports dynamiques, vous pouvez activer le service SQL Browser et autoriser l’accès au port UDP 1434 à travers vos pare-feu, de sorte qu’Azure Database Migration Service puisse se connecter à une instance nommée sur votre serveur source.
* Créez une [règle de pare-feu](../azure-sql/database/firewall-configure.md) au niveau du serveur pour que SQL Database permette à Azure Database Migration Service d’accéder aux bases de données cibles. Fournissez la plage de sous-réseau du réseau virtuel utilisé pour Azure Database Migration Service.
* Assurez-vous que les informations d’identification utilisées pour se connecter à une instance SQL Server source disposent des autorisations [CONTROL SERVER](/sql/t-sql/statements/grant-server-permissions-transact-sql).
* Assurez-vous que les informations d’identification utilisées pour se connecter à la base de données cible disposent des autorisations CONTROL DATABASE sur la base de données cible.

   > [!NOTE]
   > Pour une liste complète des prérequis pour utiliser Azure Database Migration Service afin d’effectuer des migrations à partir de SQL Server vers Azure SQL Database, consultez le tutoriel [Migrer SQL Server vers Azure SQL Database](./tutorial-sql-server-to-azure-sql.md).
   >

## <a name="prerequisites-for-migrating-sql-server-to-azure-sql-managed-instance"></a>Conditions préalables à la migration de SQL Server vers Azure SQL Managed Instance

* Créez une instance gérée SQL en suivant les indications de l’article [Créer une instance gérée SQL Azure dans le portail Azure](../azure-sql/managed-instance/instance-create-quickstart.md).
* Ouvrez vos pare-feu afin d’autoriser le trafic SMB via le port 445 pour la plage d’adresses IP ou le sous-réseau Azure Database Migration Service.
* Ouvrez votre Pare-feu Windows pour permettre à Azure Database Migration Service d’accéder au serveur SQL Server source, par défaut le port TCP 1433.
* Si vous exécutez plusieurs instances nommées de SQL Server avec des ports dynamiques, vous pouvez activer le service SQL Browser et autoriser l’accès au port UDP 1434 à travers vos pare-feu, de sorte qu’Azure Database Migration Service puisse se connecter à une instance nommée sur votre serveur source.
* Vérifiez que les informations d’identification utilisées pour se connecter à l’instance source SQL Server et à l’instance cible Managed Instance appartiennent à des membres du rôle serveur sysadmin.
* Créez un partage réseau qu’Azure Database Migration Service peut utiliser pour sauvegarder la base de données source.
* Assurez-vous que le compte de service exécutant l’instance SQL Server source dispose de privilèges en écriture sur le partage réseau que vous avez créé, et que le compte d’ordinateur pour le serveur source a accès en lecture/écriture au même partage.
* Repérez un utilisateur Windows (et son mot de passe) disposant d’un contrôle total sur le partage réseau que vous avez créé précédemment. Azure Database Migration Service emprunte l’identité de l’utilisateur afin de charger les fichiers de sauvegarde dans le conteneur de stockage Azure pour l’opération de restauration.
* Créez un conteneur d’objets blob et récupérez son URI SAP en suivant les étapes de l’article [Gérer les ressources Stockage Blob Azure avec l’Explorateur de stockage](../vs-azure-tools-storage-explorer-blobs.md#get-the-sas-for-a-blob-container). Veillez à sélectionner toutes les autorisations (lecture, écriture, suppression, liste) dans la fenêtre de la stratégie lorsque vous créez l’URI SAP.

   > [!NOTE]
   > Pour obtenir la liste complète des conditions préalables à l’utilisation d’Azure Database Migration Service afin d’effectuer des migrations de SQL Server vers SQL Managed Instance, consultez le didacticiel [Migrer SQL Server vers SQL Managed Instance](./tutorial-sql-server-to-managed-instance.md).

## <a name="next-steps"></a>Étapes suivantes

Pour une présentation d’Azure Database Migration Service et de la mise à disponibilité régionale, consultez l’article [Qu’est-ce qu’Azure Database Migration Service ?](dms-overview.md).