---
title: Déplacer des données vers une machine virtuelle SQL Server - Team Data Science Process
description: Déplacer des données à partir de fichiers plats ou d'un SQL Server local vers SQL Server sur une machine virtuelle Azure.
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
ms.openlocfilehash: c80a90b07e25942e751d52cafa47f6e3e94852ab
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93320336"
---
# <a name="move-data-to-sql-server-on-an-azure-virtual-machine"></a>Déplacer des données vers SQL Server sur une machine virtuelle Azure

Cet article présente les options de déplacement des données à partir de fichiers plats (format CSV ou TSV) ou d’un serveur SQL Server local vers un serveur SQL Server sur une machine virtuelle Azure. Ces tâches permettant de déplacer des données vers le cloud font partie du processus TDSP (Team Data Science Process).

Pour la rubrique présentant les options de déplacement de données sur une base de données Azure SQL pour Machine Learning, consultez [Déplacement de données vers une base de données Azure SQL pour Azure Machine Learning](move-sql-azure.md).

Le tableau suivant récapitule les options de déplacement de données vers SQL Server sur une machine virtuelle Azure.

| <b>SOURCE</b> | <b>DESTINATION : SQL Server sur les machines virtuelles Azure</b> |
| --- | --- |
| <b>Fichier plat</b> |1. <a href="#insert-tables-bcp">Utilitaire de copie en bloc à ligne de commande (BCP)</a><br> 2. <a href="#insert-tables-bulkquery">Requête SQL Bulk Insert</a><br> 3. <a href="#sql-builtin-utilities">Utilitaires graphiques intégrés dans SQL Server</a> |
| <b>Serveur SQL Server local</b> |1. <a href="#deploy-a-sql-server-database-to-a-microsoft-azure-vm-wizard">Assistant de déploiement d'une base de données SQL Server sur une machine virtuelle Microsoft Azure</a><br> 2. <a href="#export-flat-file">Exportation dans un fichier plat</a><br> 3. <a href="#sql-migration">Assistant Migration de la base de données SQL</a> <br> 4. <a href="#sql-backup">Sauvegarde et restauration de base de données</a><br> |

Ce document suppose que les commandes SQL sont exécutées à partir de SQL Server Management Studio ou Visual Studio Database Explorer.

> [!TIP]
> Comme autre solution, vous pouvez utiliser [Azure Data Factory](https://azure.microsoft.com/services/data-factory/) pour créer et planifier un pipeline qui déplace des données vers une machine virtuelle SQL Server sur Azure. Pour plus d’informations, consultez [Copie de données avec Azure Data Factory (activité de copie)](../../data-factory/copy-activity-overview.md)
>
>

## <a name="prerequisites"></a><a name="prereqs"></a>Configuration requise
Ce didacticiel part du principe que vous disposez de :

* Un **abonnement Azure**. Si vous n’avez pas d’abonnement, vous pouvez vous inscrire à un [essai gratuit](https://azure.microsoft.com/pricing/free-trial/).
* Un **compte de stockage Azure**. Dans ce didacticiel, vous allez utiliser un compte de stockage Azure pour stocker des données. Si vous ne possédez pas de compte de stockage Azure, consultez l’article [Créer un compte de stockage](../../storage/common/storage-account-create.md) . Après avoir créé le compte de stockage, vous devrez obtenir la clé du compte utilisée pour accéder au stockage. Consultez [Gérer les clés d’accès au compte de stockage](../../storage/common/storage-account-keys-manage.md).
* Approvisionnement d’un **serveur SQL Server sur une machine virtuelle Azure**. Pour obtenir des instructions, consultez [Configurer une machine virtuelle Azure SQL Server comme serveur IPython Notebook pour des analyses avancées](../data-science-virtual-machine/overview.md).
* **Azure PowerShell** installé et configuré localement. Pour obtenir des instructions, consultez la rubrique [Installation et configuration d'Azure PowerShell](/powershell/azure/).

## <a name="moving-data-from-a-flat-file-source-to-sql-server-on-an-azure-vm"></a><a name="filesource_to_sqlonazurevm"></a> Déplacement des données à partir d’un fichier plat source vers SQL Server sur une machine virtuelle Azure
Si vos données se trouvent dans un fichier plat (au format ligne/colonne), les méthodes suivantes permettent de les copier dans l’instance SQL Server VM on Azure :

1. [Utilitaire de copie en bloc à ligne de commande (BCP)](#insert-tables-bcp)
2. [Requête SQL Bulk Insert](#insert-tables-bulkquery)
3. [Utilitaires graphiques intégrés dans SQL Server (Importation/Exportation, SSIS)](#sql-builtin-utilities)

### <a name="command-line-bulk-copy-utility-bcp"></a><a name="insert-tables-bcp"></a>Utilitaire de copie en bloc à ligne de commande (BCP)
BCP est un utilitaire à ligne de commande, installé avec SQL Server. C’est l’un des outils les plus rapides pour déplacer des données. Il fonctionne sur les trois variantes de SQL Server (instance SQL Server locale, SQL Azure et machine virtuelle SQL Server sur Azure).

> [!NOTE]
> **Où mes données doivent-elles se trouver pour BCP ?**  
> Ce n’est pas une obligation, mais le transfert est plus rapide si les fichiers contenant les données source résident sur la même machine que l’instance SQL Server cible (débit du réseau par rapport au débit d’E/S du disque local). Vous pouvez déplacer les fichiers plats contenant les données vers la machine hébergeant SQL Server, en utilisant différents outils de copie, tels que [AZCopy](../../storage/common/storage-use-azcopy-v10.md), [Explorateur de stockage Azure](https://storageexplorer.com/) ou le copier/coller Windows via le protocole RDP (Remote Desktop Protocol).
>
>

1. Vérifiez que la base de données et les tables sont créées dans la base de données SQL Server cible. Voici un exemple montrant comment faire à l’aide des commandes `Create Database` et `Create Table` :

    ```sql
    CREATE DATABASE <database_name>
    
    CREATE TABLE <tablename>
    (
        <columnname1> <datatype> <constraint>,
        <columnname2> <datatype> <constraint>,
        <columnname3> <datatype> <constraint>
    )
    ```

1. Générez le fichier de format qui décrit le schéma de la table en exécutant la commande suivante dans la ligne de commande de la machine où BCP est installé.

    `bcp dbname..tablename format nul -c -x -f exportformatfilename.xml -S servername\sqlinstance -T -t \t -r \n`
1. Insérez les données dans la base de données à l’aide de la commande BCP, qui doit fonctionner à partir de la ligne de commande lorsque SQL Server est installé sur la même machine :

    `bcp dbname..tablename in datafilename.tsv -f exportformatfilename.xml -S servername\sqlinstancename -U username -P password -b block_size_to_move_in_single_attempt -t \t -r \n`

> **Optimisation des insertions BCP** Consultez l’article [Conseils pour optimiser l’importation en bloc](/previous-versions/sql/sql-server-2008-r2/ms177445(v=sql.105)) pour optimiser ces insertions.
>
>

### <a name="parallelizing-inserts-for-faster-data-movement"></a><a name="insert-tables-bulkquery-parallel"></a>Parallélisation des insertions pour accélérer le déplacement des données
Si le volume de données déplacées est important, vous pouvez accélérer l’opération en exécutant en parallèle plusieurs commandes BCP simultanément dans un script PowerShell.

> [!NOTE]
> **Ingestion de Big Data** Pour optimiser le chargement de données de jeux de données importants et très importants, partitionnez vos tables de base de données logiques et physiques en utilisant plusieurs groupes de fichiers et tables de partition. Pour plus d’informations sur la création et le chargement de données dans des tables de partition, consultez l’article [Importer des données en bloc et en parallèle à l’aide de tables de partition SQL](parallel-load-sql-partitioned-tables.md).
>
>

L’exemple de script PowerShell suivant montre comment effectuer des insertions en parallèle à l’aide de BCP :

```powershell
$NO_OF_PARALLEL_JOBS=2

Set-ExecutionPolicy RemoteSigned #set execution policy for the script to execute
# Define what each job does
$ScriptBlock = {
    param($partitionnumber)

    #Explicitly using SQL username password
    bcp database..tablename in datafile_path.csv -F 2 -f format_file_path.xml -U username@servername -S tcp:servername -P password -b block_size_to_move_in_single_attempt -t "," -r \n -o path_to_outputfile.$partitionnumber.txt

    #Trusted connection w.o username password (if you are using windows auth and are signed in with that credentials)
    #bcp database..tablename in datafile_path.csv -o path_to_outputfile.$partitionnumber.txt -h "TABLOCK" -F 2 -f format_file_path.xml  -T -b block_size_to_move_in_single_attempt -t "," -r \n
}


# Background processing of all partitions
for ($i=1; $i -le $NO_OF_PARALLEL_JOBS; $i++)
{
    Write-Debug "Submit loading partition # $i"
    Start-Job $ScriptBlock -Arg $i      
}


# Wait for it all to complete
While (Get-Job -State "Running")
{
    Start-Sleep 10
    Get-Job
}

# Getting the information back from the jobs
Get-Job | Receive-Job
Set-ExecutionPolicy Restricted #reset the execution policy
```

### <a name="bulk-insert-sql-query"></a><a name="insert-tables-bulkquery"></a>Requête SQL Bulk Insert
La [requête d’insertion en bloc SQL](/sql/t-sql/statements/bulk-insert-transact-sql) permet d’importer des données provenant de fichiers à lignes/colonnes dans la base de données (les types pris en charge sont présentés dans la section [Préparer les donner pour l’exportation ou l’importation en bloc (SQL Server)](/sql/relational-databases/import-export/prepare-data-for-bulk-export-or-import-sql-server)).

Voici quelques exemples de requêtes Bulk Insert :  

1. Analysez vos données et configurez les options de personnalisation souhaitées avant de procéder à l’importation, pour vérifier que la base de données SQL Server attend le même format pour les champs particuliers, comme les dates. Voici un exemple de configuration du format de date année-mois-jour (si vos données contiennent des dates dans ce format) :

    ```sql
    SET DATEFORMAT ymd;
    ```
2. Importez des données avec des instructions import en bloc :

    ```sql
    BULK INSERT <tablename>
    FROM
    '<datafilename>'
    WITH
    (
        FirstRow = 2,
        FIELDTERMINATOR = ',', --this should be column separator in your data
        ROWTERMINATOR = '\n'   --this should be the row separator in your data
    )
    ```

### <a name="built-in-utilities-in-sql-server"></a><a name="sql-builtin-utilities"></a>Utilitaires intégrés dans SQL Server
Vous pouvez utiliser SQL Server Integrations Services (SSIS) pour importer les données sur une machine virtuelle SQL Server sur Azure à partir d’un fichier plat.
SSIS est disponible dans deux environnements Studio. Pour en savoir plus, consultez l’article [Services d’intégration (SSIS) et environnements Studio](/sql/integration-services/integration-services-ssis-development-and-management-tools):

* Pour en savoir plus sur les outils SQL Server Data Tools, consultez l’article [Microsoft SQL Server Data Tools](/sql/ssdt/download-sql-server-data-tools-ssdt)  
* Pour en savoir plus sur l’Assistant Importation et Exportation, consultez l’article [Assistant Importation et Exportation SQL Server](/sql/integration-services/import-export-data/import-and-export-data-with-the-sql-server-import-and-export-wizard)

## <a name="moving-data-from-on-premises-sql-server-to-sql-server-on-an-azure-vm"></a><a name="sqlonprem_to_sqlonazurevm"></a>Déplacement des données à partir d’un serveur SQL Server local vers un serveur SQL Server sur une machine virtuelle Azure
Vous pouvez également utiliser les stratégies de migration suivantes :

1. [Assistant de déploiement d'une base de données SQL Server sur une machine virtuelle Microsoft Azure](#deploy-a-sql-server-database-to-a-microsoft-azure-vm-wizard)
2. [Exporter dans un fichier plat](#export-flat-file)
3. [Assistant Migration de la base de données SQL](#sql-migration)
4. [Sauvegarde et restauration de base de données](#sql-backup)

Chacune de ces options est décrite ci-après :

### <a name="deploy-a-sql-server-database-to-a-microsoft-azure-vm-wizard"></a>Assistant de déploiement d'une base de données SQL Server sur une machine virtuelle Microsoft Azure
L’ **Assistant de déploiement d’une base de données SQL Server sur une machine virtuelle Microsoft Azure** est une méthode simple et recommandée pour déplacer des données d’une instance SQL Server locale vers un serveur SQL Server sur une machine virtuelle Azure. Pour des instructions détaillées, ainsi qu’une présentation des autres possibilités, consultez [Migration d’une base de données vers SQL Server sur une machine virtuelle Azure](../../azure-sql/virtual-machines/windows/migrate-to-vm-from-sql-server.md).

### <a name="export-to-flat-file"></a><a name="export-flat-file"></a>Exporter dans un fichier plat
Plusieurs méthodes peuvent être utilisées pour exporter en bloc des données à partir d’un serveur SQL Server local et sont documentées dans la section [Importation et exportation de données en bloc (SQL Server)](/sql/relational-databases/import-export/bulk-import-and-export-of-data-sql-server) . Ce document utilise l’outil BCP (Bulk Copy Program) à titre d’exemple. Une fois les données exportées dans un fichier plat, il est possible de les importer en bloc dans une autre instance SQL Server.

1. Pour exporter les données de l’instance SQL Server locale vers un fichier à l’aide de l’utilitaire BCP, procédez comme suit :

    `bcp dbname..tablename out datafile.tsv -S    servername\sqlinstancename -T -t \t -t \n -c`
2. Créez la base de données et la table sur la machine virtuelle SQL Server sur Azure à l’aide de `create database` et de `create table` pour le schéma de table exporté à l’étape 1.
3. Créez un fichier de format décrivant le schéma de table des données exportées ou importées. Les détails du fichier de format sont décrits dans [créer un fichier de Format (SQL Server)](/sql/relational-databases/import-export/create-a-format-file-sql-server).

    Génération du fichier de format en cas d’exécution de BCP à partir de l’ordinateur SQL Server

    `bcp dbname..tablename format nul -c -x -f exportformatfilename.xml -S servername\sqlinstance -T -t \t -r \n`

    Génération du fichier de formation en cas d’exécution de BCP à distance sur une instance SQL Server

    `bcp dbname..tablename format nul -c -x -f  exportformatfilename.xml  -U username@servername.database.windows.net -S tcp:servername -P password  --t \t -r \n`
4. Utilisez l’une des méthodes décrites dans la section [Fichiers plats](#filesource_to_sqlonazurevm) pour déplacer les données de fichiers plats vers une instance SQL Server.

### <a name="sql-database-migration-wizard"></a><a name="sql-migration"></a>Assistant Migration de la base de données SQL
[Assistant Migration de la base de données SQL Server](https://sqlazuremw.codeplex.com/) fournit une manière conviviale pour déplacer des données entre deux instances de SQL Server. Il permet à l’utilisateur de mapper le schéma de données entre les tables sources et cibles, de choisir les types de colonne et d’utiliser d’autres fonctionnalités. Il exécute BCP en arrière-plan. Voici une copie de l’écran d’accueil de SQL Database Migration Wizard :  

![Assistant Migration de SQL Server][2]

### <a name="database-back-up-and-restore"></a><a name="sql-backup"></a>Sauvegarde et restauration de base de données
SQL Server prend en charge :

1. La [fonctionnalité de sauvegarde et de restauration de base de données](/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases) (dans un fichier local ou par exportation d’un bacpac dans un objet blob) et les [applications de la couche Données](/sql/relational-databases/data-tier-applications/data-tier-applications) (à l’aide de bacpac).
2. La possibilité de créer directement des machines virtuelles SQL Server sur Azure avec une base de données copiée ou de copier une base de données existante dans SQL Database. Pour plus d'informations, consultez [Use the Copy Database Wizard](/sql/relational-databases/databases/use-the-copy-database-wizard).

Voici une copie d’écran des options de sauvegarde/restauration de base de données dans SQL Server Management Studio.

![Outil d’importation SQL Server][1]

## <a name="resources"></a>Ressources
[Migration d'une base de données vers SQL Server sur une machine virtuelle Azure](../../azure-sql/virtual-machines/windows/migrate-to-vm-from-sql-server.md)

[Vue d’ensemble de SQL Server sur les machines virtuelles Azure](../../azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md)

[1]: ./media/move-sql-server-virtual-machine/sqlserver_builtin_utilities.png
[2]: ./media/move-sql-server-virtual-machine/database_migration_wizard.png