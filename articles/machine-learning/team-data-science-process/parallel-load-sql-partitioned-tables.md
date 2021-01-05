---
title: Importation de données en bloc en parallèle dans des tables de partition SQL - Team Data Science Process
description: Créez des tables partitionnées pour importer rapidement des données en parallèle et en bloc dans une base de données SQL Server.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 456e881d84697f4542f972ac0798cc95a3455b3c
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322404"
---
# <a name="build-and-optimize-tables-for-fast-parallel-import-of-data-into-a-sql-server-on-an-azure-vm"></a>Créer et optimiser des tables pour une importation rapide en parallèle de données dans un serveur SQL Server sur une machine virtuelle Azure

Cet article explique comment créer une ou plusieurs tables partitionnées pour importer des données rapidement, en parallèle et en bloc dans une base de données SQL Server. Dans le cas d’un chargement ou d’un transfert volumineux dans une base de données SQL, les *vues et tables partitionnées* permettent d’améliorer l’importation des données et le traitement des requêtes. 

## <a name="create-a-new-database-and-a-set-of-filegroups"></a>Créer une base de données et un ensemble de groupes de fichiers
* [Créez une base de données](/sql/t-sql/statements/create-database-transact-sql) (si elle n’existe pas).
* Ajoutez des groupes de fichiers à la base de données qui contient les fichiers physiques partitionnés. 
* Pour cette opération, utilisez [CREATE DATABASE](/sql/t-sql/statements/create-database-transact-sql) si la base de données n’existe pas encore ou [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql-set-options) si elle existe.
* Ajoutez un ou plusieurs fichiers (selon le cas) dans chaque groupe de fichiers de base de données.
  
  > [!NOTE]
  > Spécifiez le groupe de fichiers cible qui contient les données de cette partition, ainsi que le nom du ou des fichiers physiques de base de données où sont stockés les données du groupe de fichiers.
  > 
  > 

L’exemple suivant crée une base de données avec trois groupes de fichiers autres que le groupe principal et le groupe de journalisation, chacun contenant un fichier physique Les fichiers de la base de données sont créés dans le dossier de données SQL Server par défaut configuré dans l’instance SQL Server. Pour plus d’informations sur les emplacements par défaut des fichiers, consultez l’article [Emplacement des fichiers pour les instances par défaut et nommées de SQL Server](/sql/sql-server/install/file-locations-for-default-and-named-instances-of-sql-server).

```sql
   DECLARE @data_path nvarchar(256);
   SET @data_path = (SELECT SUBSTRING(physical_name, 1, CHARINDEX(N'master.mdf', LOWER(physical_name)) - 1)
      FROM master.sys.master_files
      WHERE database_id = 1 AND file_id = 1);

   EXECUTE ('
      CREATE DATABASE <database_name>
         ON  PRIMARY 
        ( NAME = ''Primary'', FILENAME = ''' + @data_path + '<primary_file_name>.mdf'', SIZE = 4096KB , FILEGROWTH = 1024KB ), 
         FILEGROUP [filegroup_1] 
        ( NAME = ''FileGroup1'', FILENAME = ''' + @data_path + '<file_name_1>.ndf'' , SIZE = 4096KB , FILEGROWTH = 1024KB ), 
         FILEGROUP [filegroup_2] 
        ( NAME = ''FileGroup2'', FILENAME = ''' + @data_path + '<file_name_2>.ndf'' , SIZE = 4096KB , FILEGROWTH = 1024KB ), 
         FILEGROUP [filegroup_3] 
        ( NAME = ''FileGroup3'', FILENAME = ''' + @data_path + '<file_name_3>.ndf'' , SIZE = 102400KB , FILEGROWTH = 10240KB ) 
         LOG ON 
        ( NAME = ''LogFileGroup'', FILENAME = ''' + @data_path + '<log_file_name>.ldf'' , SIZE = 1024KB , FILEGROWTH = 10%)
    ')
```

## <a name="create-a-partitioned-table"></a>Créer une table partitionnée
Pour créer des tables partitionnées basées sur le schéma de données, mappé aux groupes de fichiers de base de données créé à l’étape précédente, vous devez d’abord créer une fonction de partition et un schéma. Une fois les données importées en bloc dans la ou les tables partitionnées, les enregistrements sont répartis dans les groupes de fichiers conformément à un schéma de partition, comme indiqué ci-dessous.

### <a name="1-create-a-partition-function"></a>1. Créer une fonction de partition
[Créez une fonction de partition](/sql/t-sql/statements/create-partition-function-transact-sql) Cette fonction définit la plage de valeurs/limites à inclure dans chaque table de partition, par exemple pour limiter les partitions mensuelles (un\_champ\_date_heure) de l’année 2013 :
  
```sql
   CREATE PARTITION FUNCTION <DatetimeFieldPFN>(<datetime_field>)  
      AS RANGE RIGHT FOR VALUES (
         '20130201', '20130301', '20130401',
         '20130501', '20130601', '20130701', '20130801',
         '20130901', '20131001', '20131101', '20131201' )
```

### <a name="2-create-a-partition-scheme"></a>2. Créer un schéma de partition
[Créez un schéma de partition](/sql/t-sql/statements/create-partition-scheme-transact-sql). Ce schéma mappe chaque plage de la fonction de partition à un groupe de fichiers physique, par exemple :
  
```sql
      CREATE PARTITION SCHEME <DatetimeFieldPScheme> AS  
        PARTITION <DatetimeFieldPFN> TO (
        <filegroup_1>, <filegroup_2>, <filegroup_3>, <filegroup_4>,
        <filegroup_5>, <filegroup_6>, <filegroup_7>, <filegroup_8>,
        <filegroup_9>, <filegroup_10>, <filegroup_11>, <filegroup_12> )
```
 
pour vérifier les plages de chaque partition selon la fonction et le schéma, exécutez la requête suivante :
  
```sql
   SELECT psch.name as PartitionScheme,
            prng.value AS PartitionValue,
            prng.boundary_id AS BoundaryID
   FROM sys.partition_functions AS pfun
   INNER JOIN sys.partition_schemes psch ON pfun.function_id = psch.function_id
   INNER JOIN sys.partition_range_values prng ON prng.function_id=pfun.function_id
   WHERE pfun.name = <DatetimeFieldPFN>
```

### <a name="3-create-a-partition-table"></a>3. Créer une table de partition
[Créez une ou plusieurs tables partitionnées](/sql/t-sql/statements/create-table-transact-sql) conformément à votre schéma de données, puis spécifiez le schéma de partition et le champ de contrainte utilisé pour partitionner la table, par exemple :
  
```sql
   CREATE TABLE <table_name> ( [include schema definition here] )
        ON <TablePScheme>(<partition_field>)
```

Pour plus d’informations, consultez l’article [Créer des tables partitionnées et des index](/sql/relational-databases/partitions/create-partitioned-tables-and-indexes).

## <a name="bulk-import-the-data-for-each-individual-partition-table"></a>Importer les données en bloc dans chaque table de partition

* Vous pouvez utiliser BCP, BULK INSERT ou d’autres méthodes telles que l’ [Assistant Migration SQL Server](https://sqlazuremw.codeplex.com/). L’exemple fourni utilise la méthode BCP.
* [Modifiez la base de données](/sql/t-sql/statements/alter-database-transact-sql-set-options) en remplaçant le schéma de journalisation des transactions par BULK_LOGGED pour minimiser le temps de traitement de la journalisation, par exemple :
  
   ```sql
      ALTER DATABASE <database_name> SET RECOVERY BULK_LOGGED
   ```
* Pour accélérer le chargement des données, lancez plusieurs importations en bloc en parallèle. Pour obtenir des conseils sur l’accélération de l’importation en bloc de volumes importants dans des bases de données SQL Server, consultez l’article [Charger 1 To en moins d’une heure](/archive/blogs/sqlcat/load-1tb-in-less-than-1-hour).

Le script PowerShell suivant est un exemple de chargement de données en parallèle avec BCP.

```powershell
    # Set database name, input data directory, and output log directory
    # This example loads comma-separated input data files
    # The example assumes the partitioned data files are named as <base_file_name>_<partition_number>.csv
    # Assumes the input data files include a header line. Loading starts at line number 2.

    $dbname = "<database_name>"
    $indir  = "<path_to_data_files>"
    $logdir = "<path_to_log_directory>"

    # Select authentication mode
    $sqlauth = 0

    # For SQL authentication, set the server and user credentials
    $sqlusr = "<user@server>"
    $server = "<tcp:serverdns>"
    $pass   = "<password>"

    # Set number of partitions per table - Should match the number of input data files per table
    $numofparts = <number_of_partitions>

    # Set table name to be loaded, basename of input data files, input format file, and number of partitions
    $tbname = "<table_name>"
    $basename = "<base_input_data_filename_no_extension>"
    $fmtfile = "<full_path_to_format_file>"

    # Create log directory if it does not exist
    New-Item -ErrorAction Ignore -ItemType directory -Path $logdir

    # BCP example using Windows authentication
    $ScriptBlock1 = {
       param($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $num)
       bcp ($dbname + ".." + $tbname) in ($indir + "\" + $basename + "_" + $num + ".csv") -o ($logdir + "\" + $tbname + "_" + $num + ".txt") -h "TABLOCK" -F 2 -C "RAW" -f ($fmtfile) -T -b 2500 -t "," -r \n
    }

    # BCP example using SQL authentication
    $ScriptBlock2 = {
       param($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $num, $sqlusr, $server, $pass)
       bcp ($dbname + ".." + $tbname) in ($indir + "\" + $basename + "_" + $num + ".csv") -o ($logdir + "\" + $tbname + "_" + $num + ".txt") -h "TABLOCK" -F 2 -C "RAW" -f ($fmtfile) -U $sqlusr -S $server -P $pass -b 2500 -t "," -r \n
    }

    # Background processing of all partitions
    for ($i=1; $i -le $numofparts; $i++)
    {
       Write-Output "Submit loading trip and fare partitions # $i"
       if ($sqlauth -eq 0) {
          # Use Windows authentication
          Start-Job -ScriptBlock $ScriptBlock1 -Arg ($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $i)
       } 
       else {
          # Use SQL authentication
          Start-Job -ScriptBlock $ScriptBlock2 -Arg ($dbname, $tbname, $basename, $fmtfile, $indir, $logdir, $i, $sqlusr, $server, $pass)
       }
    }

    Get-Job

    # Optional - Wait till all jobs complete and report date and time
    date
    While (Get-Job -State "Running") { Start-Sleep 10 }
    date
```

## <a name="create-indexes-to-optimize-joins-and-query-performance"></a>Créer des index pour optimiser les jointures et le traitement des requêtes
* Si vous extrayez des données de plusieurs tables à des fins de modélisation, créez des index sur les clés de jointure pour améliorer les performances des jointures.
* [Créez des index](/sql/t-sql/statements/create-index-transact-sql) (en cluster ou non) ciblant le même groupe de fichiers pour chaque partition, par exemple :
  
```sql
   CREATE CLUSTERED INDEX <table_idx> ON <table_name>( [include index columns here] )
        ON <TablePScheme>(<partition)field>)
--  or,
  
        CREATE INDEX <table_idx> ON <table_name>( [include index columns here] )
        ON <TablePScheme>(<partition)field>)
 ```
 
  > [!NOTE]
  > Vous pouvez créer les index avant d’importer les données en bloc. La création des index avant l’opération d’importation ralentit le chargement des données.
  > 
  > 

## <a name="advanced-analytics-process-and-technology-in-action-example"></a>Exemple de processus d’analyse avancé et technologie en action
Pour obtenir un exemple de procédure pas à pas complet du processus TDSP (Team Data Science Process) avec un jeu de données public, consultez [Processus TDSP (Team Data Science Process) en action : utilisation de SQL Server](sql-walkthrough.md).