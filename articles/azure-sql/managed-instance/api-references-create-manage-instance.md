---
title: Informations de référence sur l’API de gestion pour Azure SQL Managed Instance
description: Découvrez la création et la configuration d’instances managées d’Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: devx-track-azurecli
ms.devlang: ''
ms.topic: reference
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/12/2019
ms.openlocfilehash: cf100861705bf1654b7206445c884b5fe315b06d
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792631"
---
# <a name="managed-api-reference-for-azure-sql-managed-instance"></a>Informations de référence sur l’API de gestion pour Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Vous pouvez créer et configurer des instances managées d’Azure SQL Managed Instance via le portail Azure, PowerShell, Azure CLI, l’API REST et Transact-SQL. Dans cet article, vous trouverez une vue d’ensemble des fonctions et de l’API que vous pouvez utiliser pour créer et configurer des instances managées.

## <a name="azure-portal-create-a-managed-instance"></a>Portail Azure : Créer une instance managée

Pour un guide de démarrage rapide vous expliquant comment créer une instance managée, consultez [Démarrage rapide : Créer une instance managée](instance-create-quickstart.md).

## <a name="powershell-create-and-configure-managed-instances"></a>PowerShell : Créer et configurer des instances managées

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge par Azure SQL Database, mais tous les développements futurs sont destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRM.

Pour créer et gérer des instances managées avec Azure PowerShell, utilisez les applets de commande PowerShell suivantes. Si vous devez installer ou mettre à niveau PowerShell, consultez la section relative à l’[installation du module Azure PowerShell](/powershell/azure/install-az-ps).

> [!TIP]
> Pour obtenir des exemples de scripts PowerShell, consultez [Script de démarrage rapide : Créer une instance managée à l’aide de la bibliothèque PowerShell](/archive/blogs/sqlserverstorageengine/quick-start-script-create-azure-sql-managed-instance-using-powershell).

| Applet de commande | Description |
| --- | --- |
|[New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance)|Crée une instance managée. |
|[Get-AzSqlInstance](/powershell/module/az.sql/get-azsqlinstance)|Retourne des informations sur une instance managée.|
|[Set-AzSqlInstance](/powershell/module/az.sql/set-azsqlinstance)|Définit les propriétés d’une instance managée.|
|[Remove-AzSqlInstance](/powershell/module/az.sql/remove-azsqlinstance)|Supprime une instance managée.|
|[Get-AzSqlInstanceOperation](/powershell/module/az.sql/get-azsqlinstanceoperation)|Obtient une liste des opérations de gestion effectuées sur l’instance gérée ou l’opération spécifique.|
|[Stop-AzSqlInstanceOperation](/powershell/module/az.sql/stop-azsqlinstanceoperation)|Annule l’opération de gestion spécifique effectuée sur l’instance gérée.|
|[Nouvelle AzSqlInstanceDatabase](/powershell/module/az.sql/new-azsqlinstancedatabase)|Crée une base de données SQL Managed Instance.|
|[Get-AzSqlInstanceDatabase](/powershell/module/az.sql/get-azsqlinstancedatabase)|Retourne des informations sur une base de données Azure SQL Managed Instance.|
|[Nouvelle AzSqlInstanceDatabase](/powershell/module/az.sql/remove-azsqlinstancedatabase)|Supprime une base de données SQL Managed Instance.|
|[Restore-AzSqlInstanceDatabase](/powershell/module/az.sql/restore-azsqlinstancedatabase)|Restaure une base de données SQL Managed Instance.|

## <a name="azure-cli-create-and-configure-managed-instances"></a>Azure CLI : Créer et configurer des instances managées

Pour créer et configurer des instances managées avec [Azure CLI](/cli/azure), utilisez les [commandes Azure CLI pour SQL Managed Instance](/cli/azure/sql/mi) suivantes. Utilisez [Azure Cloud Shell](../../cloud-shell/overview.md) pour exécuter l’interface CLI dans votre navigateur ou l’[installer](/cli/azure/install-azure-cli) sur macOS, Linux ou Windows.

> [!TIP]
> Pour un guide de démarrage rapide Azure CLI, consultez [Working with SQL Managed Instance using Azure CLI](https://medium.com/azure-sqldb-managed-instance/working-with-sql-managed-instance-using-azure-cli-611795fe0b44).

| Applet de commande | Description |
| --- | --- |
|[az sql mi create](/cli/azure/sql/mi#az-sql-mi-create) |Crée une instance managée.|
|[az sql mi list](/cli/azure/sql/mi#az-sql-mi-list)|Liste les instances managées disponibles.|
|[az sql mi show](/cli/azure/sql/mi#az-sql-mi-show)|Obtient les détails concernant une instance managée.|
|[az sql mi update](/cli/azure/sql/mi#az-sql-mi-update)|Met à jour une instance managée.|
|[az sql mi delete](/cli/azure/sql/mi#az-sql-mi-delete)|Supprime une instance managée.|
|[az sql mi op list](/cli/azure/sql/mi/op#az_sql_mi_op_list)|Obtient une liste des opérations de gestion effectuées sur l’instance gérée.|
|[az sql mi op show](/cli/azure/sql/mi/op#az_sql_mi_op_show)|Obtient l’opération de gestion spécifique effectuée sur l’instance managée.|
|[az sql mi op cancel](/cli/azure/sql/mi/op#az_sql_mi_op_cancel)|Annule l’opération de gestion spécifique effectuée sur l’instance gérée.|
|[az sql midb create](/cli/azure/sql/midb#az-sql-midb-create) |Crée une base de données managée.|
|[az sql midb list](/cli/azure/sql/midb#az-sql-midb-list)|Liste les bases de données managées disponibles.|
|[az sql midb restore](/cli/azure/sql/midb#az-sql-midb-restore)|Restaure une base de données managée.|
|[az sql midb delete](/cli/azure/sql/midb#az-sql-midb-delete)|Supprime une base de données managée.|

## <a name="transact-sql-create-and-configure-instance-databases"></a>Transact-SQL : Créer et configurer des bases de données d’instance

Pour créer et configurer des bases de données d’instance après avoir créé l’instance managée, utilisez les commandes T-SQL suivantes. Vous pouvez émettre ces commandes en utilisant le portail Azure, [SQL Server Management Studio](/sql/ssms/use-sql-server-management-studio), [Azure Data Studio](/sql/azure-data-studio/what-is) et [Visual Studio Code](https://code.visualstudio.com/docs) ou tout autre programme pouvant se connecter à un serveur et transmettre des commandes Transact-SQL.

> [!TIP]
> Pour des guides de démarrage rapide vous expliquant comment configurer une instance managée et vous y connecter en utilisant SQL Server Management Studio sur Microsoft Windows, consultez [Démarrage rapide : Configurer la machine virtuelle Azure pour qu’elle se connecte à Azure SQL Managed Instance](connect-vm-instance-configure.md) et [Démarrage rapide : Configurer une connexion point à site à Azure SQL Managed Instance à partir d’un emplacement local](point-to-site-p2s-configure.md).

> [!IMPORTANT]
> Vous ne pouvez pas créer ou supprimer une instance managée à l’aide de Transact-SQL.

| Commande | Description |
| --- | --- |
|[CREATE DATABASE](/sql/t-sql/statements/create-database-transact-sql?preserve-view=true&view=azuresqldb-mi-current)|Crée une base de données d’instance dans SQL Managed Instance. Vous devez être connecté à la base de données master pour créer une base de données.|
| [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql?preserve-view=true&view=azuresqldb-mi-current) |Modifie une base de données d’instance dans SQL Managed Instance.|

## <a name="rest-api-create-and-configure-managed-instances"></a>API REST : Créer et configurer des instances managées

Pour créer et configurer des instances managées, utilisez ces requêtes d’API REST.

| Commande | Description |
| --- | --- |
|[Instances managées - Créer ou Mettre à jour](/rest/api/sql/managedinstances/createorupdate)|Crée ou met à jour une instance managée.|
|[Instances managées - Supprimer](/rest/api/sql/managedinstances/delete)|Supprime une instance managée.|
|[Instances managées - Obtenir](/rest/api/sql/managedinstances/get)|Obtient une instance managée.|
|[Instances managées - Lister](/rest/api/sql/managedinstances/list)|Retourne la liste des instances managées d’un abonnement.|
|[Instances managées - Lister par groupe de ressources](/rest/api/sql/managedinstances/listbyresourcegroup)|Retourne la liste des instances managées d’un groupe de ressources.|
|[Instances managées - Mettre à jour](/rest/api/sql/managedinstances/update)|Met à jour une instance managée.|
|[Opérations d’instance gérée - Liste par instance gérée](/rest/api/sql/managedinstanceoperations/listbymanagedinstance)|Obtient une liste des opérations de gestion effectuées sur l’instance gérée.|
|[Opérations d’instance gérée - Obtenir](/rest/api/sql/managedinstanceoperations/get)|Obtient l’opération de gestion spécifique effectuée sur l’instance managée.|
|[Opérations d’instance gérée - Annuler](/rest/api/sql/managedinstanceoperations/cancel)|Annule l’opération de gestion spécifique effectuée sur l’instance gérée.|

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la migration d’une base de données SQL Server, consultez la section [Migrer une base de données Azure SQL](../database/migrate-to-database-from-sql-server.md).
- Pour plus d’informations sur les fonctionnalités prises en charge, consultez la page [Fonctionnalités](../database/features-comparison.md).