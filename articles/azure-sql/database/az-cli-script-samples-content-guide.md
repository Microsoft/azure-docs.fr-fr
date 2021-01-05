---
title: Exemples de script Azure CLI
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Exemples de scripts Azure CLI permettant de créer et de gérer des ressources Azure SQL Database et Azure SQL Managed Instance
services: sql-database
ms.service: sql-db-mi
ms.subservice: service
ms.custom: overview-samples, mvc, sqldbrb=2, devx-track-azurecli
ms.devlang: azurecli
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 02/03/2019
ms.openlocfilehash: 439167f29bb53d4a6e90b95826faa56e3c3170da
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94563356"
---
# <a name="azure-cli-samples-for-azure-sql-database-and-sql-managed-instance"></a>Exemples Azure CLI pour Azure SQL Database et SQL Managed Instance 
 
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Vous pouvez configurer Azure SQL Database et SQL Managed Instance à l’aide d’<a href="/cli/azure">Azure CLI</a>.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Ce tutoriel nécessite l’interface Azure CLI version 2.0 ou ultérieure. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

# <a name="azure-sql-database"></a>[Azure SQL Database](#tab/single-database)

Le tableau suivant comprend des liens vers des exemples de scripts Azure CLI permettant de gérer des bases de données mises en pool ou uniques dans Azure SQL Database. 

|Domaine|Description|
|---|---|
|**Créer des bases de données dans Azure SQL Database**||
| [Créer une base de données unique et configurer une règle de pare-feu](scripts/create-and-configure-database-cli.md) | Crée une base de données SQL et configure une règle de pare-feu au niveau du serveur. |
| [Créer des pools élastiques et déplacer les bases de données mises en pool](scripts/move-database-between-elastic-pools-cli.md) | Crée des pools élastiques, déplace des bases de données mises en pool et change les tailles de calcul. |
|**Met à l’échelle des bases de données dans Azure SQL Database**||
| [Mettre à l’échelle une base de données unique](scripts/monitor-and-scale-database-cli.md) | Met à l’échelle une base de données dans SQL Database vers une autre taille de calcul après avoir déterminé la taille de la base de données. |
| [Mettre à l’échelle un pool élastique](scripts/scale-pool-cli.md) | Met à l’échelle un pool élastique SQL à une taille de calcul différente. |
|**Configurer la géoréplication et le basculement**||
| [Ajouter une base de données unique à un groupe de basculement](scripts/add-database-to-failover-group-cli.md)| Crée une base de données et un groupe de basculement, ajoute la base de données au groupe de basculement, puis teste le basculement sur le serveur secondaire. |
| [Configurer un groupe de basculement pour un pool de bases de données élastique](../../sql-database/scripts/sql-database-add-elastic-pool-to-failover-group-cli.md) | Crée une base de données, l’ajoute à un pool élastique, ajoute le pool élastique à un groupe de basculement, puis teste le basculement sur le serveur secondaire. |
| [Configurer et basculer une base de données unique à l’aide de la géoréplication active](../../sql-database/scripts/sql-database-setup-geodr-and-failover-database-cli.md)| Configure la géoréplication active pour une base de données dans Azure SQL Database et la fait basculer vers le réplica secondaire. |
| [Configurer et basculer une base de données mise en pool à l’aide de la géoréplication active](../../sql-database/scripts/sql-database-setup-geodr-and-failover-pool-cli.md)| Configure la géoréplication active pour une base de données dans un pool élastique, puis la fait basculer vers le réplica secondaire. |
| **Audit et détection des menaces** |
| [Configurer l’audit et la détection des menaces](../../sql-database/scripts/sql-database-auditing-and-threat-detection-cli.md)| Configure les stratégies d’audit et de détection des menaces pour une base de données dans Azure SQL Database. |
| **Sauvegarder, restaurer, copier et importer une base de données**||
| [Sauvegarder une base de données](../../sql-database/scripts/sql-database-backup-database-cli.md)| Sauvegarde une base de données dans SQL Database au sein d’une sauvegarde de stockage Azure. |
| [Restaurer une base de données](../../sql-database/scripts/sql-database-restore-database-cli.md)| Restaure une base de données dans SQL Database à partir d’une sauvegarde géo-redondante et restaure une base de données supprimée dans la dernière sauvegarde. |
| [Copier une base de données sur un nouveau serveur](../../sql-database/scripts/sql-database-copy-database-to-new-server-cli.md) | Crée une copie d’une base de données existante dans SQL Database au sein d’un nouveau serveur. |
| [Importer une base de données à partir d’un fichier BACPAC](../../sql-database/scripts/sql-database-import-from-bacpac-cli.md)| Importe une base de données dans SQL Database à partir d’un fichier BACPAC. |
|||

Découvrez-en plus sur l’[API de base de données unique Azure CLI](single-database-manage.md#the-azure-cli).

# <a name="azure-sql-managed-instance"></a>[Azure SQL Managed Instance](#tab/managed-instance)

Le tableau suivant comprend des liens vers des exemples de scripts Azure CLI pour Azure SQL Managed Instance.

|Domaine|Description|
|---|---|
| **Créer une instance managée SQL**||
| [Créer une instance managée SQL](../../sql-database/scripts/sql-database-create-configure-managed-instance-cli.md)| Crée une instance managée SQL. |
| **Configurer Transparent Data Encryption (TDE)**||
| [Gérer Transparent Data Encryption dans une instance managée SQL à l’aide d’Azure Key Vault](../../sql-database/scripts/transparent-data-encryption-byok-sql-managed-instance-cli.md)| Configure Transparent Data Encryption (TDE) dans SQL Managed Instance à l’aide d’Azure Key Vault avec différents scénarios clés. |
|**Configurer un groupe de basculement**||
| [Configurer un groupe de basculement pour SQL Managed Instance](../../sql-database/scripts/sql-database-add-managed-instance-to-failover-group-cli.md) | Crée deux instances de SQL Managed Instance, les ajoute à un groupe de basculement, puis teste le basculement de l’instance managée SQL principale vers l’instance managée SQL secondaire. |
|||

Pour obtenir des exemples supplémentaires sur SQL Managed Instance, consultez les scripts de [création](/archive/blogs/sqlserverstorageengine/create-azure-sql-managed-instance-using-azure-cli), de [mise à jour](/archive/blogs/sqlserverstorageengine/modify-azure-sql-database-managed-instance-using-azure-cli), de [déplacement d’une base de données](/archive/blogs/sqlserverstorageengine/cross-instance-point-in-time-restore-in-azure-sql-database-managed-instance) et d’[utilisation](https://medium.com/azure-sqldb-managed-instance/working-with-sql-managed-instance-using-azure-cli-611795fe0b44).

Découvrez-en plus sur l’[API SQL Managed Instance Azure CLI](../managed-instance/api-references-create-manage-instance.md#azure-cli-create-and-configure-managed-instances).

---