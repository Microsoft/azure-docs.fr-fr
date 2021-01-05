---
title: Sauvegarder et restaurer des bases de données dans Azure SQL Edge
description: Découvrez les fonctionnalités de sauvegarde et de restauration dans Azure SQL Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: 114be810ea50f984c3211291691b4c4dd45ac2c7
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93395238"
---
# <a name="back-up-and-restore-databases-in-azure-sql-edge"></a>Sauvegarder et restaurer des bases de données dans Azure SQL Edge 

Azure SQL Edge repose sur les dernières versions du moteur de base de données SQL. Il fournit des fonctionnalités de sauvegarde et de restauration de base de données similaires à celles disponibles dans SQL Server sur Linux et SQL Server s’exécutant dans des conteneurs. Le composant de sauvegarde et restauration apporte une sécurité essentielle pour la protection des données stockées dans vos bases de données Azure SQL Edge. 

Pour réduire le risque de perte catastrophique de données, vous devez sauvegarder vos bases de données périodiquement afin de conserver les modifications apportées à vos données de façon régulière. Une stratégie de sauvegarde et de restauration correctement planifiée permet de protéger les bases de données contre la perte de données provoquée par différentes défaillances. Testez votre stratégie en restaurant un ensemble de sauvegardes, puis en récupérant votre base de données pour vous préparer à réagir efficacement en cas de sinistre.

Pour en savoir plus sur l’importance des sauvegardes, consultez [Sauvegarde et restauration des bases de données SQL Server](/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases/).

Azure SQL Edge vous permet d’effectuer une sauvegarde et une restauration à partir du stockage local et d’objets blob Azure. Pour plus d’informations, consultez [Sauvegarde et restauration SQL Server avec Stockage Blob Azure](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service/) et [Sauvegarde SQL Server vers une URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url).

## <a name="back-up-a-database-in-azure-sql-edge"></a>Sauvegarder une base de données dans Azure SQL Edge

Azure SQL Edge prend en charge les mêmes types de sauvegarde que SQL Server. Pour obtenir la liste complète, consultez [Présentation de la sauvegarde](/sql/relational-databases/backup-restore/backup-overview-sql-server/).

> [!IMPORTANT] 
> Les bases de données créées dans Azure SQL Edge utilisent le mode de récupération simple par défaut. Ainsi, vous ne pouvez pas effectuer de sauvegardes de fichier journal sur ces bases de données. Si vous avez besoin d’effectuer cette opération, un administrateur doit définir le mode de récupération des bases de données sur le mode de récupération complète. Pour obtenir la liste complète des modes de récupération pris en charge par SQL Server, consultez [Vue d’ensemble du mode de récupération](/sql/relational-databases/backup-restore/recovery-models-sql-server#RMov).

### <a name="back-up-to-local-disk"></a>Sauvegarder sur disque local

Dans l’exemple suivant, vous utilisez la commande Transact-SQL `BACKUP DATABASE` pour créer une sauvegarde de base de données dans le conteneur. Pour les besoins de cet exemple, vous créez un dossier nommé *backup* destiné au stockage des fichiers de sauvegarde.

1. Créez un dossier pour les sauvegardes. Exécutez cette commande sur l’hôte sur lequel votre conteneur Azure SQL Edge est en cours d’exécution. Dans la commande suivante, remplacez **<AzureSQLEdge_Container_Name>** par le nom du conteneur Azure SQL Edge de votre déploiement.

    ```bash
    sudo docker exec -it <AzureSQLEdge_Container_Name> mkdir /var/opt/mssql/backup
    ```

2. Connectez-vous à l’instance Azure SQL Edge à l’aide de SQL Server Management Studio (SSMS) ou d’Azure Data Studio. Exécutez la commande `BACKUP DATABASE` pour effectuer la sauvegarde de votre base de données utilisateur. Dans l’exemple suivant, vous effectuez la sauvegarde de la base de données *IronOreSilicaPrediction*.

    ```sql
    BACKUP DATABASE [IronOreSilicaPrediction] 
    TO DISK = N'/var/opt/mssql/backup/IronOrePredictDB.bak' 
    WITH NOFORMAT, NOINIT,  NAME = N'IronOreSilicaPrediction-Full Database Backup', 
    SKIP, NOREWIND, NOUNLOAD, COMPRESSION,  STATS = 10
    GO
    ```

3. Une fois la commande exécutée, si la sauvegarde de la base de données aboutit, des messages similaires au suivant s’affichent dans la section des résultats de SSMS ou Azure Data Studio.

    ```txt
    10 percent processed.
    20 percent processed.
    30 percent processed.
    40 percent processed.
    50 percent processed.
    60 percent processed.
    70 percent processed.
    80 percent processed.
    90 percent processed.
    100 percent processed.
    Processed 51648 pages for database 'IronOreSilicaPrediction', file 'IronOreSilicaPrediction' on file 1.
    Processed 2 pages for database 'IronOreSilicaPrediction', file 'IronOreSilicaPrediction_log' on file 1.
    BACKUP DATABASE successfully processed 51650 pages in 3.588 seconds (112.461 MB/sec).

    Completion time: 2020-04-09T23:54:48.4957691-07:00
    ```

### <a name="back-up-to-url"></a>Sauvegarde vers l’URL

Azure SQL Edge prend en charge les sauvegardes dans les objets blob de pages et les objets blob de blocs. Pour plus d’informations, consultez [Sauvegarder dans un objet blob de blocs ou un objet blob de pages](/sql/relational-databases/backup-restore/sql-server-backup-to-url#blockbloborpageblob). Dans l’exemple suivant, la base de données *IronOreSilicaPrediction* est sauvegardée dans un objet blob de blocs. 

1. Pour configurer des sauvegardes dans des objets blob de blocs, commencez par générer un jeton de signature d’accès partagé (SAS) que vous pouvez utiliser pour créer des informations d’identification SQL Server sur Azure SQL Edge. Le script crée une signature SAS associée à une stratégie d’accès stockée. Pour plus d’informations, consultez [Signatures d’accès partagé, partie 1 : Présentation du modèle SAP](../storage/common/storage-sas-overview.md). Le script écrit également la commande T-SQL requise pour créer les informations d’identification sur SQL Server. Le script suivant part du principe que vous disposez déjà d’un abonnement Azure avec un compte de stockage et un conteneur de stockage pour les sauvegardes.

    ```PowerShell
    # Define global variables for the script  
    $subscriptionName='<your subscription name>'   # the name of subscription name you will use
    $resourcegroupName = '<your resource group name>' # the name of resource group you will use
    $storageAccountName= '<your storage account name>' # the storage account name you will use for backups
    $containerName= '<your storage container name>'  # the storage container name to which you will attach the SAS policy with its SAS token  
    $policyName = 'SASPolicy' # the name of the SAS policy  

    # adds an authenticated Azure account for use in the session
    Login-AzAccount

    # set the tenant, subscription and environment for use in the rest of
    Select-AzSubscription -Subscription $subscriptionName

    # Generate the SAS token
    $sa = Get-AzStorageAccount -ResourceGroupName $resourcegroupName -Name $storageAccountName 
    $storagekey = Get-AzStorageAccountKey -ResourceGroupName $resourcegroupName -Name $storageAccountName 
    $storageContext = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storagekey[0].Value
    $cbc = Get-AzStorageContainer -Name $containerName -Context $storageContext
    $policy = New-AzStorageContainerStoredAccessPolicy -Container $containerName -Policy $policyName -Context $storageContext -ExpiryTime $(Get-Date).ToUniversalTime().AddYears(10) -Permission "rwld"
    $sas = New-AzStorageContainerSASToken -Policy $policyName -Context $storageContext -Container $containerName
    Write-Host 'Shared Access Signature= '$($sas.Substring(1))''

    # Outputs the Transact SQL to the clipboard and to the screen to create the credential using the Shared Access Signature  
    Write-Host 'Credential T-SQL'  
    $tSql = "CREATE CREDENTIAL [{0}] WITH IDENTITY='Shared Access Signature', SECRET='{1}'" -f $cbc.CloudBlobContainer.Uri.AbsoluteUri,$sas.Substring(1)
    $tSql | clip  
    Write-Host $tSql
    ```

    Après avoir exécuté le script, copiez la commande `CREATE CREDENTIAL` dans un outil de requête. Connectez-vous ensuite à une instance de SQL Server, puis exécutez la commande pour créer les informations d’identification avec la signature d’accès partagé.

2. Connectez-vous à l’instance Azure SQL Edge à l’aide de SSMS ou Azure Data Studio et créez les informations d’identification à l’aide de la commande de l’étape précédente. Veillez à remplacer la commande `CREATE CREDENTIAL` par la sortie réelle de l’étape précédente.

    ```sql
    IF NOT EXISTS  
    (SELECT * FROM sys.credentials
    WHERE name = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>')  
    CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>]
       WITH IDENTITY = 'SHARED ACCESS SIGNATURE',  
       SECRET = '<SAS_TOKEN>';
    ```

3. La commande suivante effectue une sauvegarde de la base de données *IronOreSilicaPrediction* dans le conteneur de stockage Azure.

    ```sql
    BACKUP DATABASE IronOreSilicaPrediction
    TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<mycontainername>/IronOreSilicaPrediction.bak'
    With MAXTRANSFERSIZE = 4194304,BLOCKSIZE=65536;  
    GO
    ```

## <a name="restore-a-database-in-azure-sql-edge"></a>Restaurer une base de données dans Azure SQL Edge

Dans Azure SQL Edge, vous pouvez effectuer une restauration à partir d’un disque local, d’un emplacement réseau ou d’un compte de stockage Blob Azure. Pour plus d’informations sur la restauration et la récupération dans SQL Server, consultez [Vue d’ensemble de la restauration et de la récupération](/sql/relational-databases/backup-restore/restore-and-recovery-overview-sql-server). Pour une vue d’ensemble du mode de récupération simple dans SQL Server, consultez [Restaurations complètes de bases de données (mode de récupération simple)](/sql/relational-databases/backup-restore/complete-database-restores-simple-recovery-model).

> [!IMPORTANT] 
> Les bases de données créées dans Azure SQL Edge ne peuvent pas être restaurées sur une instance de Microsoft SQL Server ou Azure SQL. En outre, une base de données créée sur Microsoft SQL Server ou SQL Azure peut être restaurée sur Azure SQL Edge, à condition que la base de données ne contienne aucune des fonctionnalités non prises en charge par Azure SQL Edge. 

### <a name="restore-from-a-local-disk"></a>Restaurer à partir d’un disque local

Cet exemple utilise la sauvegarde *IronOreSilicaPrediction* que vous avez effectuée dans l’exemple précédent. À présent, vous allez la restaurer en tant que nouvelle base de données avec un nom différent.

1. Si le fichier de sauvegarde de base de données n’est pas présent dans le conteneur, vous pouvez utiliser la commande suivante pour l’y copier. L’exemple suivant suppose la présence du fichier de sauvegarde sur l’hôte local et sa copie dans le dossier /var/opt/mssql/backup d’un conteneur Azure SQL Edge nommé *sql1*.

    ```bash
    sudo docker cp IronOrePredictDB.bak sql1:/var/opt/mssql/backup
    ```

2. Connectez-vous à l’instance Azure SQL Edge à l’aide de SSMS ou d’Azure Data Studio pour exécuter la commande de restauration. Dans l’exemple suivant, **IronOrePredictDB.bak** est restauré pour créer une base de données nommée **IronOreSilicaPrediction_2**.

    ```sql
    Restore FilelistOnly from disk = N'/var/opt/mssql/backup/IronOrePredictDB.bak'

    Restore Database IronOreSilicaPrediction_2
    From disk = N'/var/opt/mssql/backup/IronOrePredictDB.bak'
    WITH MOVE 'IronOreSilicaPrediction' TO '/var/opt/mssql/data/IronOreSilicaPrediction_Primary_2.mdf',
    MOVE 'IronOreSilicaPrediction_log' TO '/var/opt/mssql/data/IronOreSilicaPrediction_Primary_2.ldf'
    ```

3. Une fois la commande de restauration exécutée, si l’opération de restauration a abouti, des messages similaires au suivant s’affichent dans la fenêtre de sortie. 

    ```txt
    Processed 51648 pages for database 'IronOreSilicaPrediction_2', file 'IronOreSilicaPrediction' on file 1.
    Processed 2 pages for database 'IronOreSilicaPrediction_2', file 'IronOreSilicaPrediction_log' on file 1.
    RESTORE DATABASE successfully processed 51650 pages in 6.543 seconds (61.670 MB/sec).

    Completion time: 2020-04-13T23:49:21.1600986-07:00
    ```

### <a name="restore-from-url"></a>Restaurer à partir de l'URL

Azure SQL Edge prend également en charge la restauration d’une base de données à partir d’un compte Stockage Azure. Vous pouvez effectuer une restauration à partir de sauvegardes d’objets blob de blocs ou d’objets blob de pages. Dans l’exemple suivant, le fichier de sauvegarde de base de données dans un objet blob de blocs *IronOreSilicaPrediction_2020_04_16.bak* est restauré pour créer la base de données *IronOreSilicaPrediction_3*.

```sql
RESTORE DATABASE IronOreSilicaPrediction_3
FROM URL = 'https://mystorageaccount.blob.core.windows.net/mysecondcontainer/IronOreSilicaPrediction_2020_04_16.bak'
WITH MOVE 'IronOreSilicaPrediction' TO '/var/opt/mssql/data/IronOreSilicaPrediction_Primary_3.mdf', 
MOVE 'IronOreSilicaPrediction_log' TO '/var/opt/mssql/data/IronOreSilicaPrediction_Primary_3.ldf',
STATS = 10;
```