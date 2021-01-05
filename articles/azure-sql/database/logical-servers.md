---
title: Qu’est-ce qu’un serveur dans Azure SQL Database et Azure Synapse Analytics ?
titleSuffix: ''
description: Découvrez les serveurs SQL logiques utilisés par Azure SQL Database et Azure Synapse Analytics et comment les gérer.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/12/2019
ms.openlocfilehash: e0736e58983d4c28b8f2f4a20241cd36ac84fdfa
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96445385"
---
# <a name="what-is-a-logical-sql-server-in-azure-sql-database-and-azure-synapse"></a>Qu’est-ce qu’un serveur SQL logique dans Azure SQL Database et Azure Synapse ?
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

Dans Azure SQL Database et Azure Synapse Analytics, un serveur est une construction logique qui joue le rôle d’un point d’administration central pour une collection de bases de données. Au niveau du serveur, vous pouvez administrer des [connexions](logins-create-manage.md), des [règles de pare-feu](firewall-configure.md), des [règles d’audit](../../azure-sql/database/auditing-overview.md), des [stratégies de détection des menaces](threat-detection-configure.md) et des [groupes de basculement automatique](auto-failover-group-overview.md). Un serveur logique peut se trouver dans une région différente de celle de son groupe de ressources. Le serveur doit déjà exister pour que vous puissiez créer une base de données dans Azure SQL Database ou une base de données de l’entrepôt de données dans Azure Synapse Analytics. Toutes les bases de données gérées par un serveur unique sont créées dans la même région que le serveur.

Ce serveur SQL Database est différent d’une instance que vous connaissez peut-être dans le monde local. Plus précisément, il n’existe aucune garantie concernant l’emplacement des bases de données ou de la base de données de l’entrepôt de données par rapport au serveur qui les gère. En outre, ni Azure SQL Database ni Azure Synapse n’expose des accès ou des fonctionnalités au niveau de l’instance. En revanche, les bases de données d’instance dans une instance gérée sont toutes physiquement colocalisées, de la même manière que SQL Server dans le monde des machines virtuelles ou local.

Lorsque vous créez un serveur, vous fournissez un compte de connexion au serveur et un mot de passe disposant de droits d’administration pour la base de données MASTER sur ce serveur, ainsi que sur l’ensemble des bases de données qui y sont créées. Ce compte initial est un compte de connexion SQL. Azure SQL Database et Azure Synapse Analytics prennent en charge l’authentification SQL et l’authentification Azure Active Directory. Pour en savoir plus sur les connexions et l’authentification, consultez la page [Gestion des bases de données et des connexions dans Azure SQL Database](logins-create-manage.md). L’authentification Windows n’est pas prise en charge.

Un serveur dans SQL Database et Azure Synapse :

- Est créé dans un abonnement Azure, mais peut être déplacé avec les ressources qu’il contient vers un autre abonnement
- Est la ressource parente pour les bases de données, les pools élastiques et les entrepôts de données
- Fournit un espace de noms pour les bases de données, les pools élastiques et la base de données de l’entrepôt de données
- Est un conteneur logique qui possède une forte sémantique de durée de vie : si vous supprimez un serveur, il supprime ses bases de données, ses pools élastiques et ses pools SQK
- Fait partie d’un [contrôle d’accès en fonction du rôle Azure (RBAC Azure)](../../role-based-access-control/overview.md) : les bases de données, les pools élastiques et la base de données de l’entrepôt de données d’un serveur héritent des droits d’accès du serveur
- Est un élément de poids dans l’identité des bases de données, des pools élastiques et de la base de données de l’entrepôt de données à des fins de gestion des ressources Azure (voir le schéma d’URL pour les bases de données et les pools)
- Colocalise les ressources d’une région
- Fournit un point de terminaison de connexion pour l’accès aux bases de données (`<serverName>`. database.windows.net)
- Fournit l’accès aux métadonnées concernant l’accès aux ressources contenues via les DMV en vous connectant à une base de données MASTER
- Fournit l’étendue des stratégies de gestion qui s’appliquent à ses bases de données : connexions, pare-feu, audit, détection des menaces, etc.
- Est limité par un quota dans l’abonnement parent (six serveurs par abonnement par défaut ; [consultez les Limites d’abonnement ici](../../azure-resource-manager/management/azure-subscription-service-limits.md))
- Fournit l’étendue du quota de base de données et du quota DTU ou vCore pour les ressources qu’il contient (par exemple, 45 000 DTU)
- Est la portée du contrôle de version pour les fonctionnalités activées sur les ressources qu’il contient
- Les connexions principales au niveau du serveur peuvent gérer toutes les bases de données sur un serveur
- Peut contenir des connexions semblables à celles des instances de SQL Server dans votre environnement local qui ont accès à une ou plusieurs bases de données sur le serveur et qui peuvent se voir octroyer des droits d’administration limités. Pour plus d’informations, consultez [Connexions](logins-create-manage.md).
- Le classement par défaut pour toutes les bases de données créées sur un serveur est `SQL_LATIN1_GENERAL_CP1_CI_AS`, où `LATIN1_GENERAL` est anglais (États-Unis), `CP1` est la page de codes 1252, `CI` ne respecte pas la casse et `AS` est sensible aux accents.

## <a name="manage-servers-databases-and-firewalls-using-the-azure-portal"></a>Gérer des serveurs, des bases de données et des pare-feu à l’aide du Portail Azure

Vous pouvez créer le groupe de ressources d’un serveur en avance ou lors de la création du serveur lui-même. Il existe plusieurs méthodes pour accéder à un nouveau formulaire de serveur SQL : en créant un nouveau serveur SQL ou dans le cadre de la création d’une base de données.

### <a name="create-a-blank-server"></a>Créer un serveur vide

Pour créer un serveur (sans base de données, pool élastique ni base de données de l’entrepôt de données) à l’aide du [Portail Azure](https://portal.azure.com), accédez à un formulaire de serveur SQL vide (serveur SQL logique).

### <a name="create-a-blank-or-sample-database-in-azure-sql-database"></a>Créer une base de données vide ou un échantillon dans Azure SQL Database

Pour créer une base de données dans SQL Database à l’aide du [Portail Azure](https://portal.azure.com), accédez à un formulaire SQL Database vide et renseignez les informations demandées. Vous pouvez créer le groupe de ressources et le serveur en avance ou lors de la création de la base de données elle-même. Vous pouvez créer une base de données vide ou créer un exemple de base de données reposant sur Adventure Works LT.

  ![create database-1](./media/logical-servers/create-database-1.png)

> [!IMPORTANT]
> Pour plus d’informations sur la sélection du niveau tarifaire de votre base de données, consultez [Modèle d’achat DTU](service-tiers-dtu.md) et [Modèle d’achat vCore](service-tiers-vcore.md).

Pour créer une instance gérée, consultez [Créer une instance gérée](../managed-instance/instance-create-quickstart.md)

### <a name="manage-an-existing-server"></a>Gérer un serveur existant

Pour gérer un serveur existant, accédez au serveur en utilisant plusieurs méthodes : à partir de la page d’une base de données spécifique, de la page **serveurs SQL** ou de la page **Toutes les ressources**.

Pour gérer une base de données existante, accédez à la page **Bases de données SQL** page, puis cliquez sur la base de données que vous souhaitez gérer. La capture d’écran suivante montre comment commencer à définir un pare-feu au niveau du serveur pour une base de données, à partir de la page **Vue d’ensemble** d’une base de données.

   ![règle de pare-feu de serveur](./media/single-database-create-quickstart/server-firewall-rule.png)

> [!IMPORTANT]
> Pour configurer les propriétés de performance d’une base de données, consultez [Modèle d’achat DTU](service-tiers-dtu.md) et [Modèle d’achat vCore](service-tiers-vcore.md).
> [!TIP]
> Pour un démarrage rapide du Portail Azure, consultez [Créer une base de données dans SQL Database du Portail Azure](single-database-create-quickstart.md).

## <a name="manage-servers-databases-and-firewalls-using-powershell"></a>Gérer des serveurs, des bases de données et des pare-feux à l’aide de PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge, mais tous les développements à venir sont destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm.

Pour créer et gérer des serveurs, des bases de données et des pare-feux avec Azure PowerShell, utilisez les cmdlets PowerShell suivantes. Si vous devez installer ou mettre à niveau PowerShell, consultez la section relative à [l’installation du module Azure PowerShell](/powershell/azure/install-az-ps). Pour créer et gérer des pools élastiques, consultez [Pools élastiques](elastic-pool-overview.md).

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

> [!TIP]
> Pour un guide de démarrage rapide PowerShell, consultez [Créer une base de données dans Azure SQL Database à l’aide de PowerShell](single-database-create-quickstart.md). Pour obtenir des exemples de scripts PowerShell, consultez [Utiliser PowerShell pour créer une base de données dans Azure SQL Database et configurer une règle de pare-feu](scripts/create-and-configure-database-powershell.md) et [Superviser et mettre à l’échelle une base de données dans Azure SQL Database avec PowerShell](scripts/monitor-and-scale-database-powershell.md).
>

## <a name="manage-servers-databases-and-firewalls-using-the-azure-cli"></a>Gérer des serveurs, des bases de données et des pare-feux à l’aide d’Azure CLI

Pour créer et gérer des serveurs, des bases de données et des pare-feux avec [Azure CLI](/cli/azure), utilisez les commandes [Azure CLI SQL Database](/cli/azure/sql/db) suivantes. Utilisez [Cloud Shell](../../cloud-shell/overview.md) pour exécuter l’interface CLI dans votre navigateur ou [l’installer](/cli/azure/install-azure-cli) sur macOS, Linux ou Windows. Pour créer et gérer des pools élastiques, consultez [Pools élastiques](elastic-pool-overview.md).

| Applet de commande | Description |
| --- | --- |
|[az sql db create](/cli/azure/sql/db#az-sql-db-create) |Crée une base de données|
|[az sql db list](/cli/azure/sql/db#az-sql-db-list)|Répertorie toutes les bases de données d’un serveur ou toutes les bases de données d’un pool élastique|
|[az sql db list-editions](/cli/azure/sql/db#az-sql-db-list-editions)|Répertorie les objectifs de service disponibles et les limites de stockage|
|[az sql db list-usages](/cli/azure/sql/db#az-sql-db-list-usages)|Renvoie les données d’utilisation de la base de données|
|[az sql db show](/cli/azure/sql/db#az-sql-db-show)|Obtient une base de données
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

> [!TIP]
> Pour un démarrage rapide Azure CLI, consultez [Créer une base de données Azure SQL Database à l’aide d’Azure CLI](az-cli-script-samples-content-guide.md). Pour obtenir des exemples de scripts Azure CLI, consultez [Utiliser CLI pour créer une base de données Azure SQL et configurer une règle de pare-feu](scripts/create-and-configure-database-cli.md) et [Utiliser CLI pour superviser et mettre à l’échelle une base de données Azure SQL Database](scripts/monitor-and-scale-database-cli.md).
>

## <a name="manage-servers-databases-and-firewalls-using-transact-sql"></a>Gérer des serveurs, des bases de données et des pare-feux à l’aide de Transact-SQL

Pour créer et gérer des serveurs, des bases de données et des pare-feux avec Transact-SQL, utilisez les commandes T-SQL suivantes. Vous pouvez émettre ces commandes à l’aide du Portail Azure, de [SQL Server Management Studio](/sql/ssms/use-sql-server-management-studio), de [Visual Studio Code](https://code.visualstudio.com/docs) ou de tout autre programme pouvant se connecter à un serveur et transmettre des commandes Transact-SQL. Pour gérer des pools élastiques, consultez la page [Pools élastiques](elastic-pool-overview.md).

> [!IMPORTANT]
> Vous ne pouvez pas créer ou supprimer un serveur à l’aide de Transact-SQL.

| Commande | Description |
| --- | --- |
|[CREATE DATABASE (Azure SQL Database)](/sql/t-sql/statements/create-database-transact-sql?view=azuresqldb-current) | Crée une nouvelle base de données Azure SQL Database. Vous devez être connecté à la base de données master pour créer une base de données.|
|[CREATE DATABASE (Azure Synapse)](/sql/t-sql/statements/create-database-transact-sql?view=azure-sqldw-latest) | Crée une nouvelle base de données de l’entrepôt de données dans Azure Synapse. Vous devez être connecté à la base de données master pour créer une base de données.|
| [ALTER DATABASE (Azure SQL Database)](/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current) |Modifie la base de données ou le pool élastique. |
|[ALTER DATABASE (Azure Synapse Analytics)](/sql/t-sql/statements/alter-database-transact-sql?view=sql-server-ver15)|Crée une base de données de l’entrepôt de données dans Azure Synapse.|
|[DROP DATABASE (Transact-SQL)](/sql/t-sql/statements/drop-database-transact-sql)|Supprime une base de données.|
|[sys.database_service_objectives (Azure SQL Database)](/sql/relational-databases/system-catalog-views/sys-database-service-objectives-azure-sql-database)|Renvoie l’édition (niveau de service), l’objectif de service (niveau tarifaire) et, le cas échéant, le nom du pool élastique d’une base de données. Si vous êtes connecté à la base de données MASTER d’un serveur, retourne les informations sur toutes les bases de données. Pour Azure Synapse, vous devez être connecté à la base de données MASTER.|
|[sys.dm_db_resource_stats (Azure SQL Database)](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database)| Retourne la consommation de mémoire, d’E/S et d’UC d’une base de données dans Azure SQL Database. Il existe une ligne pour chaque période de 15 secondes, même s'il n'y a aucune activité dans la base de données.|
|[sys.resource_stats (Azure SQL Database)](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database)|Renvoie les données de stockage et l’utilisation d’UC pour une base de données dans Azure SQL Database. Les données sont collectées et agrégées dans des intervalles de cinq minutes.|
|[sys.database_connection_stats (Azure SQL Database)](/sql/relational-databases/system-catalog-views/sys-database-connection-stats-azure-sql-database)|Contient des statistiques sur les événements de connectivité de Azure SQL Database, ce qui fournit une vue d’ensemble du nombre d’échecs et de réussites de connexion de base de données. |
|[sys.event_log (Azure SQL Database)](/sql/relational-databases/system-catalog-views/sys-event-log-azure-sql-database)|Renvoie les connexions de base de données, échecs de connexions et interblocages pour Azure SQL Database. Utilisez ces informations pour suivre ou détecter un problème de l'activité de base de données.|
|[sp_set_firewall_rule (Azure SQL Database)](/sql/relational-databases/system-stored-procedures/sp-set-firewall-rule-azure-sql-database)|Crée ou met à jour les paramètres de pare-feu au niveau serveur de votre serveur. Cette procédure stockée est uniquement disponible dans la base de données master à la connexion du principal au niveau du serveur. Une règle de pare-feu au niveau du serveur peut uniquement être créée à l’aide de Transact-SQL, après que la première règle de pare-feu au niveau du serveur a été créée par un utilisateur disposant des autorisations au niveau d’Azure|
|[sys.firewall_rules (Azure SQL Database)](/sql/relational-databases/system-catalog-views/sys-firewall-rules-azure-sql-database)|Retourne des informations sur les paramètres du pare-feu au niveau serveur associés à un serveur.|
|[sp_delete_firewall_rule (Azure SQL Database)](/sql/relational-databases/system-stored-procedures/sp-delete-firewall-rule-azure-sql-database)|Supprime des paramètres de pare-feu au niveau serveur d’un serveur. Cette procédure stockée est uniquement disponible dans la base de données master à la connexion du principal au niveau du serveur.|
|[sp_set_database_firewall_rule (Azure SQL Database)](/sql/relational-databases/system-stored-procedures/sp-set-database-firewall-rule-azure-sql-database)|Crée ou met à jour les règles de pare-feu au niveau base de données pour une base de données dans Azure SQL Database. Les règles de pare-feu d’une base de données peuvent être configurées pour la base de données MASTER, ainsi que pour les bases de données utilisateur sur SQL Database. Les règles de pare-feu d’une base de données sont utiles lors de l’utilisation d’utilisateurs de base de données autonome. Les règles de pare-feu de base de données ne sont pas prises en charge dans Azure Synapse.|
|[sys.database_firewall_rules (Azure SQL Database)](/sql/relational-databases/system-catalog-views/sys-database-firewall-rules-azure-sql-database)|Renvoie des informations sur les paramètres de pare-feu au niveau base de données pour une base de données dans Azure SQL Database. |
|[sp_delete_database_firewall_rule (Azure SQL Database)](/sql/relational-databases/system-stored-procedures/sp-delete-database-firewall-rule-azure-sql-database)|Supprime le paramètre de pare-feu au niveau de la base de données pour une de vos bases de données dans Azure SQL Database. |

> [!TIP]
> Pour un guide de démarrage rapide en utilisant SQL Server Management Studio sur Microsoft Windows, consultez [Azure SQL Database : Utiliser SQL Server Management Studio pour se connecter et interroger des données](connect-query-ssms.md). Pour un guide de démarrage rapide en utilisant Visual Studio Code sur macOS, Linux ou Windows, consultez [Azure SQL Database : Utiliser Visual Studio Code pour se connecter et interroger des données](connect-query-vscode.md).

## <a name="manage-servers-databases-and-firewalls-using-the-rest-api"></a>Gérer des serveurs, des bases de données et des pare-feux à l’aide de l’API REST

Pour créer et gérer des serveurs, des bases de données et des pare-feux, utilisez les requêtes suivantes de l’API REST.

| Commande | Description |
| --- | --- |
|[Servers - Create or update](/rest/api/sql/servers/createorupdate)|Crée ou met à jour un serveur.|
|[Serveurs - Delete](/rest/api/sql/servers/delete)|Supprime un serveur.|
|[Serveurs - Get](/rest/api/sql/servers/get)|Obtient un serveur.|
|[Serveurs - List](/rest/api/sql/servers/list)|Retourne une liste de serveurs.|
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

- Pour en savoir plus sur la migration d’une base de données SQL Server vers Azure SQL Database, consultez la section [Migrer une base de données Azure SQL](migrate-to-database-from-sql-server.md).
- Pour plus d’informations sur les fonctionnalités prises en charge, consultez la page [Fonctionnalités](features-comparison.md).