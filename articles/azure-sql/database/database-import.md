---
title: Importer un fichier BACPAC pour créer une base de données dans Azure SQL Database
description: Créez une base de données dans Azure SQL Database ou Azure SQL Managed Instance à partir d’un fichier BACPAC.
services: sql-database
ms.service: sql-db-mi
ms.subservice: migrate
ms.custom: sqldbrb=1, devx-track-azurepowershell
ms.devlang: ''
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 10/29/2020
ms.openlocfilehash: 30a511caec82ead406f0a80f107e4261a707bfdb
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93040163"
---
# <a name="quickstart-import-a-bacpac-file-to-a-database-in-azure-sql-database-or-azure-sql-managed-instance"></a>Démarrage rapide : Importer un fichier BACPAC Ruby dans une base de données dans Azure SQL Database ou Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Vous pouvez importer une base de données SQL Server dans Azure SQL Database ou SQL Managed Instance en utilisant un fichier [BACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac). Vous pouvez importer les données d’un fichier BACPAC stocké dans Stockage Blob Azure (stockage standard uniquement) ou à partir d’un stockage local à un emplacement local. Pour optimiser la vitesse d’importation en fournissant des ressources plus nombreuses et plus rapides, mettez à l’échelle votre base de données un niveau de service et une taille de calcul supérieurs pendant le processus d’importation. Vous pouvez ensuite effectuer un scale-down après l’importation.

> [!NOTE]
> Le niveau de compatibilité de la base de données importée est basé sur celui de la base de données source.

> [!IMPORTANT]
> Après avoir importé votre base de données, vous pouvez choisir de l’utiliser à son niveau de compatibilité actuel (niveau 100 pour la base de données AdventureWorks2008R2) ou à un niveau supérieur. Pour plus d’informations sur les implications et les options du fonctionnement d’une base de données à un niveau de compatibilité spécifique, consultez [Niveau de compatibilité ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level). Consultez également [ALTER DATABASE SCOPED CONFIGURATION](/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql) pour plus d’informations sur des paramètres supplémentaires au niveau de la base de données relatifs aux niveaux de compatibilité.

## <a name="using-azure-portal"></a>En passant par le portail Azure

Regardez cette vidéo pour savoir comment importer à partir d’un fichier BACPAC sur le portail Azure ou poursuivez la lecture ci-dessous :

> [!VIDEO https://channel9.msdn.com/Shows/Data-Exposed/Its-just-SQL-Restoring-a-database-to-Azure-SQL-DB-from-backup/player?WT.mc_id=dataexposed-c9-niner]

Le [portail Microsoft Azure](https://portal.azure.com) prend *uniquement* en charge la création d’une base de données unique dans Azure SQL Database et *uniquement* à partir d’un fichier BACPAC stocké dans le stockage Blob Azure.

Pour migrer une base de données vers une [instance managée Azure SQL](../managed-instance/sql-managed-instance-paas-overview.md) à partir d’un fichier BACPAC, utilisez SQL Server Management Studio ou SQLPackage. Le portail Azure et Azure PowerShell ne sont pas actuellement pris en charge.

> [!NOTE]
> Les machines qui traitent les requêtes d’importation/exportation soumises par le biais du portail Microsoft Azure ou de PowerShell doivent stocker le fichier BACPAC et les fichiers temporaires générés par Data-Tier Application Framework (DacFX). L’espace disque requis varie considérablement entre bases de données de même taille, et peut nécessiter jusqu’à 3 fois la taille de la base de données. Les ordinateurs exécutant les requêtes d’importation/exportation ne disposent que de 450 Go d’espace disque local. Par conséquent, certaines demandes peuvent échouer avec l’erreur `There is not enough space on the disk`. Dans ce cas, la solution de contournement consiste à exécuter sqlpackage.exe sur un ordinateur ayant suffisamment d’espace disque local. Pour éviter ce problème, nous vous conseillons d’utiliser SqlPackage pour importer/exporter des bases de données de plus de 150 Go.

1. Pour importer un fichier BACPAC dans une base de données unique à l’aide du Portail Microsoft Azure, ouvrez la page appropriée du serveur, puis, dans la barre d’outils, sélectionnez **Importer la base de données**.  

   ![Importation1 de base de données](./media/database-import/sql-server-import-database.png)

1. Sélectionnez le compte de stockage et le conteneur pour le fichier BACPAC, puis sélectionnez le fichier BACPAC à partir duquel importer.

1. Spécifiez la taille de la nouvelle base de données (généralement la même que celle d’origine) et indiquez les informations d’identification du serveur SQL Server de destination. Pour obtenir la liste des valeurs possibles pour une nouvelle base de données dans Azure SQL Database, consultez [Créer une base de données](/sql/t-sql/statements/create-database-transact-sql?view=azuresqldb-current).

   ![Importation2 de base de données](./media/database-import/sql-server-import-database-settings.png)

1. Cliquez sur **OK**.

1. Pour superviser la progression d’une importation, ouvrez la page du serveur de la base de données et, sous **Paramètres** , sélectionnez **Historique d’importation/exportation**. Une fois achevée, l’importation présente l’état **Terminé**.

   ![Statut d’importation de la base de données](./media/database-import/sql-server-import-database-history.png)

1. Pour vérifier que la base de données est active sur le serveur, sélectionnez **Bases de données SQL** et assurez-vous que la nouvelle base de données est **En ligne**.

## <a name="using-sqlpackage"></a>Utilisation de SqlPackage

Pour importer une base de données SQL Server à l’aide de l’utilitaire de ligne de commande [SqlPackage](/sql/tools/sqlpackage), consultez les [paramètres et propriétés d’importation](/sql/tools/sqlpackage#import-parameters-and-properties). [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) et [SQL Server Data Tools for Visual Studio](/sql/ssdt/download-sql-server-data-tools-ssdt) incluent SqlPackage. Vous pouvez aussi télécharger la dernière version de [SqlPackage](https://www.microsoft.com/download/details.aspx?id=53876) sur le Centre de téléchargement Microsoft. 

Pour bénéficier d’une mise à l’échelle et de performances optimales, nous vous recommandons d’utiliser SqlPackage dans la plupart des environnements de production au lieu du portail Azure. Pour consulter le billet du blog SQL Server Customer Advisory Team sur la migration de SQL Server vers Azure SQL Database à l’aide de fichiers `BACPAC` (en anglais), rendez-vous [ici](/archive/blogs/sqlcat/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files).

Le modèle de provisionnement basé sur DTU prend en charge les valeurs de taille maximale de sélection de base de données pour chaque niveau. Lorsque vous importez une base de données, [utilisez l’une de ces valeurs prises en charge](/sql/t-sql/statements/create-database-transact-sql). 

La commande SqlPackage ci-dessous importe la base de données **AdventureWorks2008R2** du stockage local vers un serveur SQL logique nommé **mynewserver20170403**. Elle crée une base de données appelée **myMigratedDatabase** avec un niveau de service **Premium** et un objectif de service **P6**. Changez ces valeurs en fonction de votre environnement.

```cmd
sqlpackage.exe /a:import /tcs:"Data Source=<serverName>.database.windows.net;Initial Catalog=<migratedDatabase>;User Id=<userId>;Password=<password>" /sf:AdventureWorks2008R2.bacpac /p:DatabaseEdition=Premium /p:DatabaseServiceObjective=P6
```

> [!IMPORTANT]
> Pour vous connecter à Azure SQL Database derrière un pare-feu d’entreprise, le port 1433 de ce dernier doit être ouvert. Pour vous connecter à SQL Managed Instance, vous devez avoir une [connexion point à site](../managed-instance/point-to-site-p2s-configure.md) ou une connexion Express Route.

Cet exemple montre comment importer une base de données à l’aide de SqlPackage avec l’authentification universelle Active Directory.

```cmd
sqlpackage.exe /a:Import /sf:testExport.bacpac /tdn:NewDacFX /tsn:apptestserver.database.windows.net /ua:True /tid:"apptest.onmicrosoft.com"
```

## <a name="using-powershell"></a>Utilisation de PowerShell

> [!NOTE]
> Une [instance managée SQL](../managed-instance/sql-managed-instance-paas-overview.md) ne prend actuellement pas en charge la migration d’une base de données vers une base de données d’instance à partir d’un fichier BACPAC à l’aide d’Azure PowerShell. Pour importer dans une instance managée SQL, utilisez SQL Server Management Studio ou SQLPackage.

> [!NOTE]
> Les machines qui traitent les requêtes d’importation/exportation soumises par le biais du portail ou de Powershell doivent stocker le fichier bacpac et les fichiers temporaires générés par Data-Tier Application Framework (DacFX). L’espace disque requis varie considérablement parmi les bases de données de même taille, et peut prendre jusqu’à trois fois la taille de la base de données. Les ordinateurs exécutant les requêtes d’importation/exportation ne disposent que de 450 Go d’espace disque local. Par conséquent, certaines requêtes peuvent échouer avec l’erreur « Espace insuffisant sur le disque ». Dans ce cas, la solution de contournement consiste à exécuter sqlpackage.exe sur un ordinateur ayant suffisamment d’espace disque local. Pour importer/exporter des bases de données de plus de 150 Go, utilisez SqlPackage pour éviter ce problème.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager (RM) est toujours pris en charge, mais tous les développements à venir sont destinés au module Az.Sql. Le module AzureRM continue à recevoir des résolutions de bogues jusqu’à au moins décembre 2020.  Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm. Pour en savoir plus sur leur compatibilité, consultez [Présentation du nouveau module Az Azure PowerShell](/powershell/azure/new-azureps-module-az).

Utilisez l’applet de commande [New-AzSqlDatabaseImport](/powershell/module/az.sql/new-azsqldatabaseimport) pour soumettre une demande d’importation de base de données à Azure. Selon la taille de la base de données, l’importation peut prendre un certain temps avant d’aboutir. Le modèle de provisionnement basé sur DTU prend en charge les valeurs de taille maximale de sélection de base de données pour chaque niveau. Lorsque vous importez une base de données, [utilisez l’une de ces valeurs prises en charge](/sql/t-sql/statements/create-database-transact-sql). 

```powershell
$importRequest = New-AzSqlDatabaseImport -ResourceGroupName "<resourceGroupName>" `
    -ServerName "<serverName>" -DatabaseName "<databaseName>" `
    -DatabaseMaxSizeBytes "<databaseSizeInBytes>" -StorageKeyType "StorageAccessKey" `
    -StorageKey $(Get-AzStorageAccountKey `
        -ResourceGroupName "<resourceGroupName>" -StorageAccountName "<storageAccountName>").Value[0] `
        -StorageUri "https://myStorageAccount.blob.core.windows.net/importsample/sample.bacpac" `
        -Edition "Standard" -ServiceObjectiveName "P6" `
        -AdministratorLogin "<userId>" `
        -AdministratorLoginPassword $(ConvertTo-SecureString -String "<password>" -AsPlainText -Force)
```

Vous pouvez utiliser la cmdlet [Get-AzSqlDatabaseImportExportStatus](/powershell/module/az.sql/get-azsqldatabaseimportexportstatus) pour vérifier la progression de l’importation. L’exécution de l’applet de commande de suite après la demande retourne généralement `Status: InProgress`. L’importation est terminée dès que vous voyez `Status: Succeeded`.

```powershell
$importStatus = Get-AzSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink

[Console]::Write("Importing")
while ($importStatus.Status -eq "InProgress") {
    $importStatus = Get-AzSqlDatabaseImportExportStatus -OperationStatusLink $importRequest.OperationStatusLink
    [Console]::Write(".")
    Start-Sleep -s 10
}

[Console]::WriteLine("")
$importStatus
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Utilisez la commande [az-sql-db-import](/cli/azure/sql/db#az-sql-db-import) pour soumettre une demande d’importation de base de données à Azure. Selon la taille de la base de données, l’importation peut prendre un certain temps avant d’aboutir. Le modèle de provisionnement basé sur DTU prend en charge les valeurs de taille maximale de sélection de base de données pour chaque niveau. Lorsque vous importez une base de données, [utilisez l’une de ces valeurs prises en charge](/sql/t-sql/statements/create-database-transact-sql). 

```azurecli
# get the storage account key
az storage account keys list --resource-group "<resourceGroup>" --account-name "<storageAccount>"

az sql db import --resource-group "<resourceGroup>" --server "<server>" --name "<database>" `
    --storage-key-type "StorageAccessKey" --storage-key "<storageAccountKey>" `
    --storage-uri "https://myStorageAccount.blob.core.windows.net/importsample/sample.bacpac" `
    -u "<userId>" -p "<password>"
```

* * *

> [!TIP]
> Pour un autre exemple de script, consultez [Importation d’une base de données à partir d’un fichier BACPAC](scripts/import-from-bacpac-powershell.md).

## <a name="limitations"></a>Limites

- L’importation dans une base de données d’un pool élastique n’est pas prise en charge. Vous pouvez importer les données dans une base de données unique, puis déplacer cette dernière vers le pool élastique.
- Le service d’importation/exportation ne fonctionne pas lorsque Autoriser l’accès aux services Azure est défini sur DÉSACTIVÉ. Toutefois, vous pouvez contourner le problème en exécutant manuellement sqlpackage.exe à partir d’une machine virtuelle Azure ou en effectuant l’exportation directement dans votre code à l’aide de l’API DACFx.
- L’importation ne prend pas en charge la spécification d’une redondance de stockage de sauvegarde lors de la création d’une base de données et la création s’effectue avec la redondance de stockage de sauvegarde géo-redondante par défaut. Pour contourner ce problème, commencez par créer une base de données vide avec la redondance de stockage de sauvegarde souhaitée à l’aide de Portail Azure ou PowerShell, puis importez le BACPAC dans cette base de données vide. 

> [!NOTE]
> La redondance de stockage de sauvegarde Azure SQL Database configurable est actuellement disponible en préversion publique dans la région Azure Asie Sud-Est uniquement.

## <a name="import-using-wizards"></a>Importer à l’aide d’Assistants

Vous pouvez aussi utiliser ces Assistants.

- [Assistant Importation de l’application de la couche Données dans SQL Server Management Studio](/sql/relational-databases/data-tier-applications/import-a-bacpac-file-to-create-a-new-user-database#using-the-import-data-tier-application-wizard).
- [Assistant Importation et Exportation de SQL Server](/sql/integration-services/import-export-data/start-the-sql-server-import-and-export-wizard).

## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment vous connecter à une base de données dans Azure SQL Database et comment l’interroger, consultez [Démarrage rapide : Azure SQL Database : Utiliser SQL Server Management Studio pour se connecter et interroger des données](connect-query-ssms.md).
- Pour consulter le billet du blog SQL Server Customer Advisory Team sur la migration de SQL Server vers Azure SQL Database à l’aide de fichiers BACPAC (en anglais), rendez-vous [ici](https://techcommunity.microsoft.com/t5/DataCAT/Migrating-from-SQL-Server-to-Azure-SQL-Database-using-Bacpac/ba-p/305407).
- Pour obtenir une description de l’ensemble du processus de migration de base de données SQL Server ainsi que des recommandations relatives aux performances, consultez [Migration de base de données SQL Server vers Azure SQL Database](migrate-to-database-from-sql-server.md).
- Pour savoir comment gérer et partager de façon sécurisée les clés de stockage et les signatures d’accès partagé, consultez le [Guide de sécurité de Stockage Azure](../../storage/blobs/security-recommendations.md).
