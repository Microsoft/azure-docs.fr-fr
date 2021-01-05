---
title: Exemples de script Azure PowerShell
description: Utilisez des exemples de scripts Azure PowerShell pour vous aider à créer et à gérer des ressources Azure SQL Database et Azure SQL Managed Instance.
services: sql-database
ms.service: sql-db-mi
ms.subservice: development
ms.custom: sqldbrb=2
ms.devlang: PowerShell
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/25/2019
ms.openlocfilehash: 77220fabc78dd10b4299219ef84c1f6eb32ba82e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85987307"
---
# <a name="azure-powershell-samples-for-azure-sql-database-and-azure-sql-managed-instance"></a>Exemples Azure PowerShell pour Azure SQL Database et Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Azure SQL Database et Azure SQL Managed Instance vous permettent de configurer vos bases de données, instances et pools à l’aide d’Azure PowerShell.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser PowerShell en local, vous avez besoin de la version 1.4.0 d’AZ PowerShell ou d’une version ultérieure pour suivre ce tutoriel. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.

## <a name="azure-sql-database"></a>[Azure SQL Database](#tab/single-database)

Le tableau suivant comprend des liens vers des exemples de scripts Azure PowerShell pour Azure SQL Database.

|Lien|Description|
|---|---|
|**Créer et configurer des bases de données uniques et des pools élastiques**||
| [Créer une base de données unique et configurer une règle de pare-feu au niveau du serveur](scripts/create-and-configure-database-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Ce script PowerShell crée une base de données unique et configure une règle de pare-feu IP au niveau du serveur. |
| [Créer des pools élastiques et déplacer les bases de données mises en pool](scripts/move-database-between-elastic-pools-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Ce script PowerShell crée des pools élastiques, déplace des bases de données mises en pool et change les tailles de calcul.|
|**Configurer la géoréplication et le basculement**||
| [Configurer et basculer une seule base de données à l’aide de la géoréplication active](scripts/setup-geodr-and-failover-database-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Ce script PowerShell configure la géoréplication active pour une base de données unique et la fait basculer vers le réplica secondaire. |
| [Configurer et basculer une base de données mise en pool à l’aide de la géoréplication active](scripts/setup-geodr-and-failover-elastic-pool-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Ce script PowerShell configure la géoréplication active pour une base de données dans un pool élastique et la fait basculer vers le réplica secondaire. |
|**Configurer un groupe de basculement**||
| [Configurer un groupe de basculement pour une seule base de données](scripts/add-database-to-failover-group-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Ce script PowerShell crée une base de données et un groupe de basculement, ajoute la base de données au groupe de basculement et teste le basculement vers le serveur secondaire. |
| [Configurer un groupe de basculement pour un pool de bases de données élastique](scripts/add-elastic-pool-to-failover-group-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Ce script PowerShell crée une base de données, l’ajoute à un pool élastique, ajoute le pool élastique à un groupe de basculement et teste le basculement vers le serveur secondaire. |
|**Mettre à l’échelle une base de données unique et un pool élastique**||
| [Mettre à l’échelle une base de données unique](scripts/monitor-and-scale-database-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Ce script PowerShell supervise les métriques de performances d’une base de données unique, la met à l’échelle à une taille de calcul supérieure, et crée une règle d’alerte sur l’une des métriques de performances. |
| [Mettre à l’échelle un pool élastique](scripts/monitor-and-scale-pool-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Ce script PowerShell supervise les métriques de performances d’un pool élastique, l’adapte à une taille de calcul supérieure et crée une règle d’alerte sur l’une des métriques de performances. |
| **Audit et détection des menaces** |
| [Configurer l’audit et la détection des menaces](scripts/auditing-threat-detection-powershell-configure.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Ce script PowerShell configure les stratégies d’audit et de détection des menaces pour une base de données. |
| **Restaurer, copier et importer une base de données**||
| [Restaurer une base de données](scripts/restore-database-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Cet script PowerShell restaure une base de données à partir d’une sauvegarde géo-redondante et restaure une base de données supprimée dans la dernière sauvegarde. |
| [Copier une base de données sur un nouveau serveur](scripts/copy-database-to-new-server-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Ce script PowerShell crée une copie d’une base de données existante dans un nouveau serveur. |
| [Importer une base de données à partir d’un fichier bacpac](scripts/import-from-bacpac-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Ce script PowerShell importe dans Azure SQL Database une base de données à partir d’un fichier bacpac. |
| **Synchroniser des données entre des bases de données**||
| [Synchroniser des données entre des bases de données](scripts/sql-data-sync-sync-data-between-sql-databases.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Ce script PowerShell configure la synchronisation des données entre plusieurs bases de données dans Azure SQL Database. |
| [Synchroniser des données entre une base de données SQL et un serveur SQL local](scripts/sql-data-sync-sync-data-between-azure-onprem.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Ce script PowerShell configure la synchronisation des données entre une base de données dans Azure SQL Database et une base de données SQL Server locale. |
| [Mettre à jour le schéma de synchronisation SQL Data Sync](scripts/update-sync-schema-in-sync-group.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Ce script PowerShell ajoute ou supprime des éléments du schéma de synchronisation Data Sync. |
|||

Apprenez-en davantage sur l’[API de base de données unique Azure PowerShell](single-database-manage.md#powershell). 

## <a name="azure-sql-managed-instance"></a>[Azure SQL Managed Instance](#tab/managed-instance)

Le tableau suivant comprend des liens vers des exemples de scripts Azure PowerShell pour Azure SQL Managed Instance.

|Lien|Description|
|---|---|
|**Créer et configurer des instances managées**||
| [Créer et gérer une instance managée](../managed-instance/scripts/create-configure-managed-instance-powershell.md) | Ce script PowerShell montre comment créer et gérer une instance managée à l’aide d’Azure PowerShell. |
| [Créer et gérer une instance managée à l’aide du modèle Azure Resource Manager](../managed-instance/scripts/create-powershell-azure-resource-manager-template.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Ce script PowerShell montre comment créer et gérer une instance managée à l’aide d’Azure PowerShell et du modèle Azure Resource Manager.|
| [Restaurer une base de données vers une instance managée dans une autre région de zone géographique](../managed-instance/scripts/restore-geo-backup.md) | Ce script PowerShell effectue une sauvegarde d’une base de données et la restaure dans une autre région. Il s’agit d’un scénario de reprise d’activité après sinistre par géorestauration. |
| **Configurer Transparent Data Encryption**||
| [Gérer Transparent Data Encryption dans une instance managée à l’aide de votre propre clé Azure Key Vault](../managed-instance/scripts/transparent-data-encryption-byok-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Ce script PowerShell configure Transparent Data Encryption dans le scénario Bring Your Own Key pour Azure SQL Managed Instance, à l’aide d’une clé Azure Key Vault.|
|**Configurer un groupe de basculement**||
| [Configurer un groupe de basculement pour une instance gérée](../managed-instance/scripts/add-to-failover-group-powershell.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Ce script PowerShell crée deux instances gérées, les ajoute à un groupe de basculement, puis teste le basculement de l’instance gérée principale vers l’instance gérée secondaire. |
|||

Découvrez-en plus sur les [applets de commande PowerShell pour Azure SQL Managed Instance](../managed-instance/api-references-create-manage-instance.md#powershell-create-and-configure-managed-instances).

---

## <a name="additional-resources"></a>Ressources supplémentaires

Les exemples listés dans cette page utilisent les [applets de commande PowerShell](/powershell/module/az.sql/) pour la création et la gestion des ressources Azure SQL. Applets de commande supplémentaires pour l’exécution de requêtes et pour l’exécution de nombreuses tâches de base de données situées dans le module [sqlserver](/powershell/module/sqlserver/). Pour plus d’informations, consultez [SQL Server PowerShell](/sql/powershell/sql-server-powershell/).
