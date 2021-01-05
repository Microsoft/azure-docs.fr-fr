---
title: Créer et gérer des serveurs et des bases de données uniques
description: Apprenez à créer et à gérer des serveurs et des bases de données uniques dans Azure SQL Database à l’aide du portail Azure, de PowerShell, d’Azure CLI, de Transact-SQL (T-SQL) et de l’API REST.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/12/2019
ms.openlocfilehash: e50cce74f7291a6673e5d43f3485a1c63c81d827
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93319277"
---
# <a name="create-and-manage-servers-and-single-databases-in-azure-sql-database"></a>Créer et gérer des serveurs et des bases de données uniques dans Azure SQL Database

Vous pouvez créer et gérer des serveurs et des bases de données uniques dans Azure SQL Database à l’aide du portail Azure, de PowerShell, de l’interface Azure CLI, de l’API REST et de Transact-SQL.

## <a name="the-azure-portal"></a>Le portail Azure

Vous pouvez créer le groupe de ressources pour Azure SQL Database en avance ou lors de la création du serveur lui-même.

### <a name="create-a-server"></a>Créer un serveur

Pour créer un serveur à l’aide du [portail Azure](https://portal.azure.com), créez une ressource de [serveur](logical-servers.md) à partir de la Place de Marché Azure. Vous pouvez également créer le serveur lorsque vous déployez une instance Azure SQL Database.

  ![Créer un serveur](./media/single-database-manage/create-logical-sql-server.png)

### <a name="create-a-blank-or-sample-database"></a>Créer un exemple de base de données ou une base de données vide

Pour créer une base de données Azure SQL unique à l’aide du [portail Azure](https://portal.azure.com), choisissez la ressource Azure SQL Database dans la Place de Marché Azure. Vous pouvez créer le groupe de ressources et le serveur en avance ou lors de la création de la base de données unique elle-même. Vous pouvez créer une base de données vide ou créer un exemple de base de données reposant sur Adventure Works LT.

  ![create database-1](./media/single-database-manage/create-database-1.png)

> [!IMPORTANT]
> Pour plus d’informations sur la sélection du niveau tarifaire de votre base de données, consultez [Modèle d’achat DTU](service-tiers-dtu.md) et [Modèle d’achat vCore](service-tiers-vcore.md).

## <a name="manage-an-existing-server"></a>Gérer un serveur existant

Pour gérer un serveur existant, accédez au serveur en utilisant plusieurs méthodes : à partir de la page d’une base de données spécifique, de la page **serveurs SQL** ou de la page **Toutes les ressources**.

Pour gérer une base de données existante, accédez à la page **Bases de données SQL** , puis sélectionnez la base de données que vous souhaitez gérer. La capture d’écran suivante montre comment commencer à définir un pare-feu au niveau du serveur pour une base de données, à partir de la page **Vue d’ensemble** d’une base de données.

   ![règle de pare-feu de serveur](./media/single-database-manage/server-firewall-rule.png)

> [!IMPORTANT]
> Pour configurer les propriétés de performance d’une base de données, consultez [Modèle d’achat DTU](service-tiers-dtu.md) et [Modèle d’achat vCore](service-tiers-vcore.md).
> [!TIP]
> Pour un démarrage rapide du portail Azure, consultez [Créer une base de données dans SQL Database dans le portail Azure](single-database-create-quickstart.md).

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge par Azure SQL Database, mais tous les développements futurs sont destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm.

Pour créer et gérer des serveurs, des bases de données uniques et mises en pool ainsi que des pare-feu de niveau serveur avec Azure PowerShell, utilisez les applets de commande PowerShell suivantes. Si vous devez installer ou mettre à niveau PowerShell, consultez la section relative à [l’installation du module Azure PowerShell](/powershell/azure/install-az-ps).

> [!TIP]
> Pour obtenir des exemples de scripts PowerShell, consultez [Utiliser PowerShell pour créer une base de données dans SQL Database et configurer une règle de pare-feu de niveau serveur](scripts/create-and-configure-database-powershell.md) et [Superviser et mettre à l’échelle une base de données dans SQL Database avec PowerShell](scripts/monitor-and-scale-database-powershell.md).

| Applet de commande | Description |
| --- | --- |
|[New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase)|Crée une base de données |
|[Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase)|Obtient une ou plusieurs bases de données|
|[Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase)|Définit les propriétés d’une base de données, ou déplace une base de données existante dans un pool élastique|
|[Remove-AzSqlDatabase](/powershell/module/az.sql/remove-azsqldatabase)|Supprime une base de données|
|[New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup)|Crée un groupe de ressources|
|[New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver)|Crée un serveur|
|[Get-AzSqlServer](/powershell/module/az.sql/get-azsqlserver)|Renvoie des informations concernant les serveurs|
|[Set-AzSqlServer](/powershell/module/az.sql/set-azsqlserver)|Modifie les propriétés d’un serveur|
|[Remove-AzSqlServer](/powershell/module/az.sql/remove-azsqlserver)|Supprime un serveur|
|[New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule)|Crée une règle de pare-feu au niveau du serveur |
|[Get-AzSqlServerFirewallRule](/powershell/module/az.sql/get-azsqlserverfirewallrule)|Obtient les règles de pare-feu d’un serveur|
|[Set-AzSqlServerFirewallRule](/powershell/module/az.sql/set-azsqlserverfirewallrule)|Modifie une règle de pare-feu sur un serveur|
|[Remove-AzSqlServerFirewallRule](/powershell/module/az.sql/remove-azsqlserverfirewallrule)|Supprime une règle de pare-feu d’un serveur|
| New-AzSqlServerVirtualNetworkRule | Crée une [*règle de réseau virtuel*](vnet-service-endpoint-rule-overview.md), basée sur un sous-réseau qui est un point de terminaison de service de réseau virtuel. |

## <a name="the-azure-cli"></a>L’interface Azure CLI

Pour créer et gérer les serveurs, bases de données et pare-feu avec [Azure CLI](/cli/azure), utilisez les commandes [Azure CLI](/cli/azure/sql/db) suivantes. Utilisez [Cloud Shell](../../cloud-shell/overview.md) pour exécuter l’interface CLI dans votre navigateur ou [l’installer](/cli/azure/install-azure-cli) sur macOS, Linux ou Windows. Pour créer et gérer des pools élastiques, consultez [Pools élastiques](elastic-pool-overview.md).

> [!TIP]
> Pour un démarrage rapide d’Azure CLI, consultez [Créer une base de données Azure SQL unique à l’aide d’Azure CLI](az-cli-script-samples-content-guide.md). Pour obtenir des exemples de scripts Azure CLI, consultez [Utiliser CLI pour créer une base de données Azure SQL et configurer une règle de pare-feu de base de données SQL](scripts/create-and-configure-database-cli.md) et [Utiliser CLI pour superviser et mettre à l’échelle une base de données Azure SQL](scripts/monitor-and-scale-database-cli.md).
>

| Applet de commande | Description |
| --- | --- |
|[az sql db create](/cli/azure/sql/db#az-sql-db-create) |Crée une base de données|
|[az sql db list](/cli/azure/sql/db#az-sql-db-list)|Répertorie toutes les bases de données et les entrepôts de données d’un serveur, ou toutes les bases de données d’un pool élastique|
|[az sql db list-editions](/cli/azure/sql/db#az-sql-db-list-editions)|Répertorie les objectifs de service disponibles et les limites de stockage|
|[az sql db list-usages](/cli/azure/sql/db#az-sql-db-list-usages)|Renvoie les données d’utilisation de la base de données|
|[az sql db show](/cli/azure/sql/db#az-sql-db-show)|Obtient une base de données ou un entrepôt de données|
|[az sql db update](/cli/azure/sql/db#az-sql-db-update)|Met à jour une base de données|
|[az sql db delete](/cli/azure/sql/db#az-sql-db-delete)|Supprime une base de données|
|[az group create](/cli/azure/group#az-group-create)|Crée un groupe de ressources|
|[az sql server create](/cli/azure/sql/server#az-sql-server-create)|Crée un serveur|
|[az sql server list](/cli/azure/sql/server#az-sql-server-list)|Répertorie les serveurs|
|[az sql server list-usages](/cli/azure/sql/server#az-sql-server-list-usages)|Renvoie les données d’utilisation d’un serveur|
|[az sql server show](/cli/azure/sql/server#az-sql-server-show)|Obtient un serveur|
|[az sql server update](/cli/azure/sql/server#az-sql-server-update)|Met à jour un serveur|
|[az sql server delete](/cli/azure/sql/server#az-sql-server-delete)|Supprime un serveur.|
|[az sql server firewall-rule create](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-create)|Crée la règle de pare-feu d’un serveur|
|[az sql server firewall-rule list](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-list)|Répertorie les règles de pare-feu sur un serveur|
|[az sql server firewall-rule show](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-show)|Affiche les détails d’une règle de pare-feu|
|[az sql server firewall-rule update](/cli/azure/sql/server/firewall-rule##az-sql-server-firewall-rule-update)|Met à jour une règle de pare-feu|
|[az sql server firewall-rule delete](/cli/azure/sql/server/firewall-rule#az-sql-server-firewall-rule-delete)|Supprime une règle de pare-feu|

## <a name="transact-sql-t-sql"></a>Transact-SQL (T-SQL)

Pour créer et gérer les serveurs, bases de données et pare-feu avec Transact-SQL, utilisez les commandes T-SQL suivantes. Vous pouvez émettre ces commandes à l’aide du portail Azure, de [SQL Server Management Studio](/sql/ssms/use-sql-server-management-studio), de [Visual Studio Code](https://code.visualstudio.com/docs) ou de tout autre programme pouvant se connecter à un serveur dans SQL Database et transmettre des commandes Transact-SQL. Pour gérer des pools élastiques, consultez la page [Pools élastiques](elastic-pool-overview.md).

> [!TIP]
> Pour un guide de démarrage rapide en utilisant SQL Server Management Studio sur Microsoft Windows, consultez [Azure SQL Database : Utiliser SQL Server Management Studio pour se connecter et interroger des données](connect-query-ssms.md). Pour un guide de démarrage rapide en utilisant Visual Studio Code sur macOS, Linux ou Windows, consultez [Azure SQL Database : Utiliser Visual Studio Code pour se connecter et interroger des données](connect-query-vscode.md).
> [!IMPORTANT]
> Vous ne pouvez pas créer ou supprimer un serveur à l’aide de Transact-SQL.

| Commande | Description |
| --- | --- |
|[CREATE DATABASE](/sql/t-sql/statements/create-database-transact-sql?view=azuresqldb-current&preserve-view=true)|Crée une base de données unique. Vous devez être connecté à la base de données master pour créer une base de données.|
| [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current&preserve-view=true) |Modifie une base de données ou un pool élastique. |
|[DROP DATABASE](/sql/t-sql/statements/drop-database-transact-sql)|Supprime une base de données.|
|[sys.database_service_objectives](/sql/relational-databases/system-catalog-views/sys-database-service-objectives-azure-sql-database)|Renvoie l’édition (niveau de service), l’objectif de service (niveau tarifaire) et, le cas échéant, le nom du pool élastique pour Azure SQL Database ou un pool SQL dédié dans Azure Synapse Analytics. Si vous êtes connecté à la base de données MASTER d’un serveur dans SQL Database, retourne les informations sur toutes les bases de données. Pour Azure Synapse Analytics, vous devez être connecté à la base de données MASTER.|
|[sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database)| Retourne la consommation de mémoire, d’E/S et d’UC d’une base de données dans Azure SQL Database. Il existe une ligne pour chaque période de 15 secondes, même s'il n'y a aucune activité dans la base de données.|
|[sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database)|Renvoie les données de stockage et l’utilisation d’UC pour une base de données dans Azure SQL Database. Les données sont collectées et agrégées dans des intervalles de cinq minutes.|
|[sys.database_connection_stats](/sql/relational-databases/system-catalog-views/sys-database-connection-stats-azure-sql-database)|Contient des statistiques pour les événements de connectivité SQL Database, ce qui fournit une vue d’ensemble du nombre d’échecs et de réussites de connexion de base de données. |
|[sys.event_log](/sql/relational-databases/system-catalog-views/sys-event-log-azure-sql-database)|Renvoie les blocages, les échecs de connexion et les réussites de connexion Azure SQL Database. Vous pouvez utiliser ces informations pour suivre ou dépanner votre activité de base de données avec SQL Database.|
|[sp_set_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-set-firewall-rule-azure-sql-database)|Crée ou met à jour les paramètres de pare-feu de niveau serveur pour votre serveur. Cette procédure stockée est uniquement disponible dans la base de données master à la connexion du principal au niveau du serveur. Une règle de pare-feu au niveau du serveur peut uniquement être créée à l’aide de Transact-SQL, après que la première règle de pare-feu au niveau du serveur a été créée par un utilisateur disposant des autorisations au niveau d’Azure|
|[sys.firewall_rules](/sql/relational-databases/system-catalog-views/sys-firewall-rules-azure-sql-database)|Renvoie des informations sur les paramètres de pare-feu de niveau serveur associés à votre base de données dans Azure SQL Database.|
|[sp_delete_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-delete-firewall-rule-azure-sql-database)|Supprime des paramètres de pare-feu de niveau serveur de votre serveur. Cette procédure stockée est uniquement disponible dans la base de données master à la connexion du principal au niveau du serveur.|
|[sp_set_database_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-set-database-firewall-rule-azure-sql-database)|Crée ou met à jour les règles de pare-feu au niveau base de données pour votre base de données dans Azure SQL Database. Les règles de pare-feu d’une base de données peuvent être configurées pour la base de données MASTER, ainsi que pour les bases de données utilisateur sur SQL Database. Les règles de pare-feu d’une base de données sont utiles lors de l’utilisation d’utilisateurs de base de données autonome. |
|[sys.database_firewall_rules](/sql/relational-databases/system-catalog-views/sys-database-firewall-rules-azure-sql-database)|Renvoie des informations sur les paramètres de pare-feu de niveau base de données associés à votre base de données dans Azure SQL Database. |
|[sp_delete_database_firewall_rule](/sql/relational-databases/system-stored-procedures/sp-delete-database-firewall-rule-azure-sql-database)|Supprime un paramètre de pare-feu de niveau base de données d’une base de données. |

## <a name="rest-api"></a>API REST

Pour créer et gérer les serveurs, bases de données et pare-feu, utilisez les demandes d’API REST suivantes.

| Commande | Description |
| --- | --- |
|[Servers - Create or update](/rest/api/sql/servers/createorupdate)|Crée ou met à jour un serveur.|
|[Serveurs - Delete](/rest/api/sql/servers/delete)|Supprime un serveur SQL.|
|[Serveurs - Get](/rest/api/sql/servers/get)|Obtient un serveur.|
|[Serveurs - List](/rest/api/sql/servers/list)|Retourne la liste des serveurs d’un abonnement.|
|[Servers - List by resource Group](/rest/api/sql/servers/listbyresourcegroup)|Retourne une liste de serveurs dans un groupe de ressources.|
|[Serveurs - Update](/rest/api/sql/servers/update)|Met à jour un serveur existant.|
|[Databases - Create or update](/rest/api/sql/databases/createorupdate)|Crée une base de données ou met à jour une base de données existante.|
|[Bases de données - Supprimer](/rest/api/sql/databases/delete)|Supprime une base de données.|
|[Bases de données - Obtenir](/rest/api/sql/databases/get)|Obtient une base de données.|
|[Databases - List by elastic pool](/rest/api/sql/databases/listbyelasticpool)|Renvoie une liste des bases de données dans un pool élastique.|
|[Databases - List by server](/rest/api/sql/databases/listbyserver)|Retourne une liste de bases de données d’un serveur.|
|[Bases de données - Mettre à jour](/rest/api/sql/databases/update)|Met à jour une base de données existante.|
|[Firewall rules - Create or update](/rest/api/sql/firewallrules/createorupdate)|Crée ou met à jour une règle de pare-feu.|
|[Firewall rules - Delete](/rest/api/sql/firewallrules/delete)|Supprime une règle de pare-feu.|
|[Firewall rules - Get](/rest/api/sql/firewallrules/get)|Obtient une règle de pare-feu.|
|[Firewall rules - List by server](/rest/api/sql/firewallrules/listbyserver)|Retourne une liste de règles de pare-feu.|

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la migration d’une base de données SQL Server, consultez la section [Migrer une base de données Azure SQL](migrate-to-database-from-sql-server.md).
- Pour plus d’informations sur les fonctionnalités prises en charge, consultez la page [Fonctionnalités](features-comparison.md).
 
