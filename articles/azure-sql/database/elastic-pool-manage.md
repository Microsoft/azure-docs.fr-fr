---
title: Gérer les pools élastiques
description: Vous pouvez créer et gérer des pools élastiques Azure SQL Database via le Portail Azure, PowerShell, Azure CLI, Transact-SQL et l’API REST.
services: sql-database
ms.service: sql-database
ms.subservice: elastic-pools
ms.topic: conceptual
author: oslake
ms.author: moslake
ms.reviewer: sstein
ms.date: 03/12/2019
ms.custom: seoapril2019 sqldbrb=1, devx-track-azurecli
ms.openlocfilehash: 9c9af6e3bc3dfd798f4b3f0cad9319aa573c425d
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96455993"
---
# <a name="manage-elastic-pools-in-azure-sql-database"></a>Gérer des pools élastiques dans Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Avec un pool élastique, on détermine la quantité de ressources qui lui sont nécessaires pour gérer la charge de travail de ses bases de données ainsi que la quantité de ressources allouées à chaque base de données mise en pool.

## <a name="azure-portal"></a>Portail Azure

Tous les paramètres du pool se trouvent au même endroit : le panneau **Configurer le pool**. Pour y accéder, recherchez un pool élastique dans le Portail Azure, puis cliquez sur **Configurer le pool** en haut du panneau ou dans le menu des ressources situé sur la gauche.

Vous pouvez effectuer n’importe quelle combinaison de modifications parmi les suivantes, et les enregistrer dans un même ensemble :

1. Modifier le niveau de service du pool
2. Mettre à l’échelle les performances (DTU ou vCore) et le stockage
3. Ajouter ou supprimer des bases de données dans le pool
4. Définir un niveau minimal (garanti) et maximal de performance pour les bases de données des pools
5. Examiner le récapitulatif des coûts pour voir toutes les modifications apportées à votre facture depuis vos nouvelles sélections

![Panneau de configuration du pool élastique](./media/elastic-pool-manage/configure-pool.png)

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge par Azure SQL Database, mais tous les développements futurs sont destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm.

Pour créer et gérer les pools élastiques SQL Database et des bases de données mises en pool avec Azure PowerShell, utilisez les cmdlets PowerShell suivantes. Si vous devez installer ou mettre à niveau PowerShell, consultez la section relative à [l’installation du module Azure PowerShell](/powershell/azure/install-az-ps). Pour créer et gérer les serveurs d’un pool élastique, consultez [Créer et gérer des serveurs](logical-servers.md). Pour créer et gérer des règles de pare-feu, consultez l’article expliquant comment [créer et gérer des règles de pare-feu à l’aide de PowerShell](firewall-configure.md#use-powershell-to-manage-server-level-ip-firewall-rules).

> [!TIP]
> Pour obtenir des exemples de scripts PowerShell, consultez [Créer des pools élastiques et déplacer les bases de données entre les pools et en dehors d’un pool à l’aide de PowerShell](scripts/move-database-between-elastic-pools-powershell.md) et [Utiliser PowerShell pour surveiller et mettre à l’échelle un pool élastique SQL dans Azure SQL Database](scripts/monitor-and-scale-pool-powershell.md).
>

| Applet de commande | Description |
| --- | --- |
|[New-AzSqlElasticPool](/powershell/module/az.sql/new-azsqlelasticpool)|Crée un pool élastique.|
|[Get-AzSqlElasticPool](/powershell/module/az.sql/get-azsqlelasticpool)|Obtient des pools élastiques et leurs valeurs de propriété.|
|[Set-AzSqlElasticPool](/powershell/module/az.sql/set-azsqlelasticpool)|Modifie les propriétés d’un pool élastique. Par exemple, utilisez la propriété **StorageMB** pour modifier le stockage maximal d’un pool élastique.|
|[Remove-AzSqlElasticPool](/powershell/module/az.sql/remove-azsqlelasticpool)|Supprime un pool élastique.|
|[Get-AzSqlElasticPoolActivity](/powershell/module/az.sql/get-azsqlelasticpoolactivity)|Obtient l’état des opérations sur un pool élastique|
|[New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase)|Crée une base de données dans un pool existant ou en tant que base de données seule. |
|[Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase)|Obtient une ou plusieurs bases de données.|
|[Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase)|Définit les propriétés d’une base de données, ou déplace une base de données existante dans un pool élastique, en dehors de celui-ci ou entre des pools élastiques.|
|[Remove-AzSqlDatabase](/powershell/module/az.sql/remove-azsqldatabase)|Supprime une base de données.|

> [!TIP]
> La création d’un grand nombre de bases de données dans un pool élastique peut prendre du temps si elle se fait par le biais du portail ou d’applets de commande PowerShell qui créent une seule base de données à la fois. Pour automatiser la création dans un pool élastique, consultez [CreateOrUpdateElasticPoolAndPopulate](https://gist.github.com/billgib/d80c7687b17355d3c2ec8042323819ae).

## <a name="azure-cli"></a>Azure CLI

Pour créer et gérer un serveur des pools élastiques avec [Azure CLI](/cli/azure), utilisez les commandes [Azure CLI SQL Database](/cli/azure/sql/db) suivantes. Utilisez [Cloud Shell](../../cloud-shell/overview.md) pour exécuter l’interface CLI dans votre navigateur ou [l’installer](/cli/azure/install-azure-cli) sur macOS, Linux ou Windows.

> [!TIP]
> Pour obtenir des exemples de scripts Azure CLI , consultez [Utiliser l’interface CLI afin de déplacer une base de données dans SQL Database dans un pool élastique SQL](scripts/move-database-between-elastic-pools-cli.md) et [Utiliser Azure CLI pour mettre un pool élastique SQL à l’échelle dans Azure SQL Database](scripts/scale-pool-cli.md).
>

| Applet de commande | Description |
| --- | --- |
|[az sql elastic-pool create](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-create)|Crée un pool élastique.|
|[az sql elastic-pool list](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-list)|Renvoie une liste de pools élastiques dans un serveur.|
|[az sql elastic-pool list-dbs](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-list-dbs)|Renvoie une liste des bases de données dans un pool élastique.|
|[az sql elastic-pool list-editions](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-list-editions)|Inclut également les paramètres DTU de pool, les limites de stockage et les paramètres par base de données disponibles. Pour réduire les détails, les limites de stockage supplémentaires et les paramètres par base de données sont masqués par défaut.|
|[az sql elastic-pool update](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-update)|Met à jour un pool élastique.|
|[az sql elastic-pool delete](/cli/azure/sql/elastic-pool#az-sql-elastic-pool-delete)|Supprime le pool élastique.|

## <a name="transact-sql-t-sql"></a>Transact-SQL (T-SQL)

Pour créer et déplacer des bases de données dans les pools élastiques existants ou pour renvoyer des informations sur un pool élastique SQL Database avec Transact-SQL, utilisez les commandes T-SQL suivantes. Vous pouvez émettre ces commandes à l’aide du Portail Azure, de [SQL Server Management Studio](/sql/ssms/use-sql-server-management-studio), de [Visual Studio Code](https://code.visualstudio.com/docs) ou de tout autre programme pouvant se connecter à un serveur et transmettre des commandes Transact-SQL. Pour créer et gérer des règles de pare-feu à l’aide de T-SQL, consultez l’article [Gérer les règles de pare-feu à l’aide de Transact-SQL](firewall-configure.md#use-transact-sql-to-manage-ip-firewall-rules).

> [!IMPORTANT]
> Vous ne pouvez pas créer, mettre à jour ou supprimer un pool élastique Azure SQL Database à l’aide de Transact-SQL. Vous pouvez ajouter ou supprimer des bases de données à partir d’un pool élastique, et vous pouvez utiliser des vues de gestion dynamiques pour renvoyer des informations sur les pools élastiques existants.
>

| Commande | Description |
| --- | --- |
|[CREATE DATABASE (Azure SQL Database)](/sql/t-sql/statements/create-database-azure-sql-database)|Crée une base de données dans un pool existant ou en tant que base de données seule. Vous devez être connecté à la base de données master pour créer une base de données.|
| [ALTER DATABASE (Azure SQL Database)](/sql/t-sql/statements/alter-database-azure-sql-database) |Déplace une base de données dans un pool élastique, en dehors de celui-ci ou entre des pools élastiques.|
|[DROP DATABASE (Transact-SQL)](/sql/t-sql/statements/drop-database-transact-sql)|Supprime une base de données.|
|[sys.elastic_pool_resource_stats (Azure SQL Database)](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database)|Renvoie les statistiques d’utilisation de ressources pour tous les pools élastiques dans un serveur. Pour chaque pool élastique, il existe une ligne pour chaque fenêtre de création de rapports de 15 secondes (quatre lignes par minute). Cela inclut la consommation de stockage, le journal, les E/S, l’UC et l’utilisation de session/requête simultanée par toutes les bases de données du pool.|
|[sys.database_service_objectives (Azure SQL Database)](/sql/relational-databases/system-catalog-views/sys-database-service-objectives-azure-sql-database)|Renvoie l’édition (niveau de service), l’objectif de service (niveau tarifaire) et, le cas échéant, le nom du pool élastique pour une base de données dans SQL Database ou Azure Synapse Analytics. Si vous êtes connecté à la base de données MASTER d’un serveur, retourne les informations sur toutes les bases de données. Pour Azure Synapse Analytics, vous devez être connecté à la base de données MASTER.|

## <a name="rest-api"></a>API REST

Pour créer et gérer des pools élastiques SQL Database et des bases de données mises en pool, utilisez ces demandes d’API REST.

| Commande | Description |
| --- | --- |
|[Elastic pools - Create or update](/rest/api/sql/elasticpools/createorupdate)|Crée un pool élastique ou met à jour un pool élastique existant.|
|[Pools élastiques - Supprimer](/rest/api/sql/elasticpools/delete)|Supprime le pool élastique.|
|[Pools élastiques - Obtenir](/rest/api/sql/elasticpools/get)|Obtenir un pool élastique.|
|[Elastic pools - List by server](/rest/api/sql/elasticpools/listbyserver)|Renvoie une liste de pools élastiques dans un serveur.|
|[Pools élastiques - Mettre à jour](/rest/api/sql/elasticpools/listbyserver)|Met à jour un pool élastique existant.|
|[Elastic pool activities](/rest/api/sql/elasticpoolactivities)|Retourne les activités de pool élastique.|
|[Elastic pool database activities](/rest/api/sql/elasticpooldatabaseactivities)|Retourne l’activité sur les bases de données à l’intérieur d’un pool élastique.|
|[Databases - Create or update](/rest/api/sql/databases/createorupdate)|Crée une base de données ou met à jour une base de données existante.|
|[Bases de données - Obtenir](/rest/api/sql/databases/get)|Obtient une base de données.|
|[Databases - List by elastic pool](/rest/api/sql/databases/listbyelasticpool)|Renvoie une liste des bases de données dans un pool élastique.|
|[Databases - List by server](/rest/api/sql/databases/listbyserver)|Retourne une liste de bases de données d’un serveur.|
|[Bases de données - Mettre à jour](/rest/api/sql/databases/update)|Met à jour une base de données existante.|

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les modèles de conception pour les applications SaaS avec des pools élastiques, consultez [Modèles de conception pour les applications SaaS multilocataires avec Azure SQL Database](saas-tenancy-app-design-patterns.md).
* Pour obtenir un didacticiel SaaS utilisant des pools élastiques, consultez [Présentation de l’application SaaS Wingtip](saas-dbpertenant-wingtip-app-overview.md).