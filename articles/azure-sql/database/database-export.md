---
title: Exporter une base de données Azure SQL Database dans un fichier BACPAC (à l’aide du Portail Azure)
titleSuffix: Azure SQL Database & Azure SQL Managed Instance
description: Exportez une base de données dans un fichier BACPAC à l’aide du Portail Azure.
services: sql-database
ms.service: sql-db-mi
ms.subservice: data-movement
author: stevestein
ms.custom: sqldbrb=2
ms.author: sstein
ms.reviewer: ''
ms.date: 07/16/2019
ms.topic: how-to
ms.openlocfilehash: 7dc6cd580687544226b61a29ca9ccf2d1b8dff42
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92671539"
---
# <a name="export-to-a-bacpac-file---azure-sql-database-and-azure-sql-managed-instance"></a>Exporter vers un fichier BACPAC - Azure SQL Database et SQL Managed Instance

[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Si vous avez besoin d’exporter une base de données afin de l’archiver ou de la déplacer vers une autre plateforme, vous pouvez exporter le schéma et les données de cette base de données vers un fichier [BACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications#Anchor_4). Un fichier BACPAC est un fichier ZIP avec extension BACPAC contenant les métadonnées et les données de la base de données. Il peut être stocké dans Stockage Blob Azure ou dans un stockage local, à un emplacement local, puis réimporté vers Azure SQL Database, Azure SQL Managed Instance ou une instance de SQL Server.

## <a name="considerations"></a>Considérations

- Pour qu'une exportation soit cohérente sur le plan transactionnel, vous devez vous assurer qu'aucune activité d'écriture n'a lieu pendant l'exportation, ou que vous exportez à partir d'une copie [cohérente sur le plan transactionnel](database-copy.md) de votre base de données.
- Si vous exportez dans le stockage Blob, la taille maximale d’un fichier BACPAC est de 200 Go. Pour archiver un fichier BACPAC plus volumineux, exportez dans le stockage local.
- L’exportation d’un fichier BACPAC vers le stockage Premium Azure en utilisant les méthodes décrites dans cet article n’est pas prise en charge.
- Le stockage derrière un pare-feu n’est actuellement pas pris en charge.
- Si l’opération d’exportation dure plus de 20 heures, elle peut être annulée. Pour améliorer les performances lors de l’exportation, vous pouvez :

  - Augmentez temporairement votre taille de calcul.
  - Interrompez toutes les activités de lecture et d’écriture lors de l’exportation.
  - Utilisez un [index cluster](/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described) avec des valeurs non nulles sur toutes les tables de grande taille. Sans index cluster, une exportation peut échouer si elle dure plus de 6 à 12 heures. Cela est dû au fait que les services d’exportation doivent effectuer une analyse complète de la table pour tenter de l’exporter en entier. Une bonne méthode pour déterminer si vos tables sont optimisées pour l’exportation consiste à exécuter **DBCC SHOW_STATISTICS** et à vérifier que *RANGE_HI_KEY* n’est pas nul et que sa valeur a une bonne distribution. Pour plus d’informations, consultez [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql).

> [!NOTE]
> Les BACPAC ne sont pas conçus pour être utilisés pour les opérations de sauvegarde et de restauration. Azure crée automatiquement des sauvegardes pour chaque base de données utilisateur. Pour plus d’informations, consultez [Vue d’ensemble de la continuité de l’activité avec la base de données SQL Azure](business-continuity-high-availability-disaster-recover-hadr-overview.md) et [Découvrir les sauvegardes SQL Database](automated-backups-overview.md).

## <a name="the-azure-portal"></a>Le portail Azure

L'exportation d'un fichier BACPAC de base de données à partir d'une instance d'[Azure SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md) à l'aide du Portail Azure n'est actuellement pas prise en charge. Veuillez plutôt utiliser SQL Server Management Studio ou SQLPackage.

> [!NOTE]
> Les machines qui traitent les requêtes d’importation/exportation soumises par le biais du portail Microsoft Azure ou de PowerShell doivent stocker le fichier BACPAC et les fichiers temporaires générés par Data-Tier Application Framework (DacFX). L’espace disque requis varie considérablement entre bases de données de même taille, et peut nécessiter jusqu’à 3 fois la taille de la base de données. Les ordinateurs exécutant les requêtes d’importation/exportation ne disposent que de 450 Go d’espace disque local. Par conséquent, certaines demandes peuvent échouer avec l’erreur `There is not enough space on the disk`. Dans ce cas, la solution de contournement consiste à exécuter sqlpackage.exe sur un ordinateur ayant suffisamment d’espace disque local. Pour éviter ce problème, nous vous conseillons d’utiliser [SqlPackage](#sqlpackage-utility) pour importer/exporter des bases de données supérieures à 150 Go.

1. Pour exporter une base de données à l’aide du [portail Azure](https://portal.azure.com), ouvrez la page de votre base de données, puis cliquez sur **Exporter** dans la barre d’outils.

   ![Capture d’écran mettant en évidence le bouton Exporter.](./media/database-export/database-export1.png)

2. Spécifiez le nom du fichier BACPAC, sélectionnez un compte de stockage Azure existant ainsi que le conteneur pour l’exportation, puis fournissez les informations d’identification qui conviennent à des fins de connexion à la base de données source. Une **connexion d'administrateur SQL Server** est requise, même si vous êtes l'administrateur Azure, car un administrateur Azure ne dispose pas des mêmes autorisations dans Azure SQL Database ou Azure SQL Managed Instance.

    ![Exportation de base de données](./media/database-export/database-export2.png)

3. Cliquez sur **OK** .

4. Pour surveiller la progression de l'opération d'exportation, ouvrez la page du serveur qui contient la base de données en cours d'exportation. Sous **Paramètres** , cliquez sur **Historique d'importation/exportation** .

   ![historique d'exportation](./media/database-export/export-history.png)

## <a name="sqlpackage-utility"></a>Utilitaire SQLPackage

Pour exporter une base de données SQL Database à l’aide de l’utilitaire de ligne de commande [SqlPackage](/sql/tools/sqlpackage), consultez [Paramètres et propriétés d’exportation](/sql/tools/sqlpackage#export-parameters-and-properties). L’utilitaire SQLPackage est fourni avec les dernières versions de [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) et de [SQL Server Data Tools pour Visual Studio](/sql/ssdt/download-sql-server-data-tools-ssdt). Vous pouvez également télécharger la dernière version de [SqlPackage](https://www.microsoft.com/download/details.aspx?id=53876) directement à partir du Centre de téléchargement Microsoft.

Nous recommandons l’utilisation de l’utilitaire SQLPackage afin de bénéficier de l’évolutivité et des performances dans la plupart des environnements de production. Pour consulter le billet du blog SQL Server Customer Advisory Team sur la migration de SQL Server vers Azure SQL Database à l’aide de fichiers BACPAC (en anglais), rendez-vous [ici](/archive/blogs/sqlcat/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files).

Cet exemple montre comment exporter une base de données à l’aide de SqlPackage.exe avec l’authentification Active Directory universelle :

```cmd
SqlPackage.exe /a:Export /tf:testExport.bacpac /scs:"Data Source=apptestserver.database.windows.net;Initial Catalog=MyDB;" /ua:True /tid:"apptest.onmicrosoft.com"
```

## <a name="sql-server-management-studio-ssms"></a>SQL Server Management Studio (SSMS)

Les dernières versions de SQL Server Management Studio incluent un assistant permettant d'exporter une base de données Azure SQL Database ou SQL Managed Instance vers un fichier BACPAC. Consultez [Exporter une application de la couche Données](/sql/relational-databases/data-tier-applications/export-a-data-tier-application).

## <a name="powershell"></a>PowerShell

> [!NOTE]
> [Azure SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md) ne prend actuellement pas en charge l'exportation d'une base de données vers un fichier BACPAC à l'aide d'Azure PowerShell. Pour exporter une instance gérée vers un fichier BACPAC, utilisez SQL Server Management Studio ou SQLPackage.

Utilisez la cmdlet [New-AzSqlDatabaseExport](/powershell/module/az.sql/new-azsqldatabaseexport) pour soumettre une requête d’exportation de base de données au service Azure SQL Database. Selon la taille de votre base de données, l'opération d'exportation peut prendre plus ou moins longtemps.

```powershell
$exportRequest = New-AzSqlDatabaseExport -ResourceGroupName $ResourceGroupName -ServerName $ServerName `
  -DatabaseName $DatabaseName -StorageKeytype $StorageKeytype -StorageKey $StorageKey -StorageUri $BacpacUri `
  -AdministratorLogin $creds.UserName -AdministratorLoginPassword $creds.Password
```

Pour vérifier l’état de la requête d’exportation, utilisez la cmdlet [Get-AzSqlDatabaseImportExportStatus](/powershell/module/az.sql/get-azsqldatabaseimportexportstatus). L’exécution immédiatement après la requête retourne généralement **État : En cours** . Lorsque **État : réussite** s’affiche, l’exportation est terminée.

```powershell
$exportStatus = Get-AzSqlDatabaseImportExportStatus -OperationStatusLink $exportRequest.OperationStatusLink
[Console]::Write("Exporting")
while ($exportStatus.Status -eq "InProgress")
{
    Start-Sleep -s 10
    $exportStatus = Get-AzSqlDatabaseImportExportStatus -OperationStatusLink $exportRequest.OperationStatusLink
    [Console]::Write(".")
}
[Console]::WriteLine("")
$exportStatus
```

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la conservation des copies de sauvegarde à long terme des bases de données uniques et des bases de données mises en pool comme alternative à l’exportation d’une base de données à des fins d’archivage, consultez [Conservation à long terme](long-term-retention-overview.md). Vous pouvez utiliser des travaux SQL Agent pour planifier des [sauvegardes de base de données en copie seule](/sql/relational-databases/backup-restore/copy-only-backups-sql-server) comme alternative à la rétention des sauvegardes à long terme.
- Pour consulter le billet du blog SQL Server Customer Advisory Team sur la migration de SQL Server vers Azure SQL Database à l’aide de fichiers BACPAC (en anglais), rendez-vous [ici](/archive/blogs/sqlcat/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files).
- Pour en savoir plus sur l’importation d’un fichier BACPAC dans une base de données SQL Server, consultez [Importer un fichier BACPAC dans une base de données SQL Server](/sql/relational-databases/data-tier-applications/import-a-bacpac-file-to-create-a-new-user-database).
- Pour en savoir plus sur l’exportation d’un fichier BACPAC à partir d’une base de données SQL Server, consultez [Exporter une application de la couche Données](/sql/relational-databases/data-tier-applications/export-a-data-tier-application)
- Pour en savoir plus sur l’utilisation du service de migration de données afin de migrer une base de données, consultez [Migrer depuis SQL Server vers Azure SQL Database hors connexion à l’aide de DMS](../../dms/tutorial-sql-server-to-azure-sql.md).
- Si vous exportez depuis SQL Server avant de procéder à la migration vers Azure SQL Database, consultez [Migrer une base de données SQL Server vers Azure SQL Database](migrate-to-database-from-sql-server.md).
- Pour savoir comment gérer et partager de façon sécurisée les clés de stockage et les signatures d’accès partagé, consultez le [Guide de sécurité de Stockage Azure](../../storage/blobs/security-recommendations.md).