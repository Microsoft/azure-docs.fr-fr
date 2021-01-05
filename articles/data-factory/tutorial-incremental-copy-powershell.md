---
title: Copier de façon incrémentielle une table à l’aide de PowerShell
description: Dans ce tutoriel, vous allez créer un pipeline de fabrique de données Azure qui copie de façon incrémentielle les données d’une base de données Azure SQL dans un stockage Blob Azure.
services: data-factory
author: dearandyxu
ms.author: yexu
manager: anandsub
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: seo-dt-2019
ms.date: 01/22/2018
ms.openlocfilehash: 65a2d06acc3461d881ad6f100f3720b217ef7634
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97510207"
---
# <a name="incrementally-load-data-from-azure-sql-database-to-azure-blob-storage-using-powershell"></a>Charger de façon incrémentielle les données depuis Azure SQL Database dans un stockage Blob Azure à l’aide de PowerShell

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Dans ce tutoriel, vous allez créer une fabrique de données Azure avec un pipeline qui charge les données delta d’une table dans Azure SQL Database vers un stockage Blob Azure.

Dans ce tutoriel, vous allez effectuer les étapes suivantes :

> [!div class="checklist"]
> * Préparer le magasin de données pour y stocker la valeur de limite.
> * Créer une fabrique de données.
> * créez des services liés.
> * Créer des jeux de données source, récepteur et filigrane.
> * Créer un pipeline.
> * Exécuter le pipeline.
> * Surveiller l’exécution du pipeline.

## <a name="overview"></a>Vue d’ensemble
Voici le diagramme général de la solution :

![Chargement incrémentiel de données](media/tutorial-Incrementally-copy-powershell/incrementally-load.png)

Voici les étapes importantes à suivre pour créer cette solution :

1. **Sélectionner la colonne de limite**.
    Sélectionnez une colonne dans le magasin de données sources, qui peut servir à découper les enregistrements nouveaux ou mis à jour pour chaque exécution. Normalement, les données contenues dans cette colonne sélectionnée (par exemple, last_modify_time ou ID) continuent de croître à mesure que des lignes sont créées ou mises à jour. La valeur maximale de cette colonne est utilisée comme limite.

2. **Préparer un magasin de données pour stocker la valeur de limite**.   
    Dans ce tutoriel, la valeur de filigrane est stockée dans une base de données SQL.

3. **Créer un pipeline avec le flux de travail suivant** :

    Le pipeline de cette solution compte les activités suivantes :

    * Créez deux activités de recherche. Servez-vous de la première activité de recherche pour récupérer la dernière valeur de filigrane. Utilisez la deuxième activité de recherche pour récupérer la nouvelle valeur de filigrane. Ces valeurs de filigrane sont transmises à l’activité de copie.
    * Créez une activité de copie qui copie les lignes du magasin de données source dont la valeur de la colonne de filigrane est supérieure à l’ancienne valeur de filigrane et inférieure à la nouvelle. Elle copie ensuite les données delta du magasin de données source dans un stockage d’objets blob sous la forme d’un nouveau fichier.
    * Créez une activité StoredProcedure qui met à jour la valeur de filigrane pour le pipeline s’exécutant la prochaine fois.


Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* **Azure SQL Database**. Vous utilisez la base de données comme magasin de données source. Si vous n’avez pas de base de données dans Azure SQL Database, consultez [Créer une base de données dans Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md) pour savoir comme en créer une.
* **Stockage Azure**. Vous utilisez le stockage d’objets blob comme magasin de données récepteur. Si vous ne possédez pas de compte de stockage, consultez l’article [Créer un compte de stockage](../storage/common/storage-account-create.md) pour découvrir comment en créer un. Créez un conteneur sous le nom adftutorial. 
* **Azure PowerShell**. Suivez les instructions de la page [Installation et configuration d’Azure PowerShell](/powershell/azure/install-Az-ps).

### <a name="create-a-data-source-table-in-your-sql-database"></a>Créer une table de source de données dans votre base de données SQL
1. Ouvrez SQL Server Management Studio. Dans l’**Explorateur de serveurs**, cliquez avec le bouton droit sur la base de données, puis choisissez **Nouvelle requête**.

2. Exécutez la commande SQL suivante sur votre base de données SQL pour créer une table sous le nom `data_source_table` comme magasin de source de données :

    ```sql
    create table data_source_table
    (
        PersonID int,
        Name varchar(255),
        LastModifytime datetime
    );

    INSERT INTO data_source_table
    (PersonID, Name, LastModifytime)
    VALUES
    (1, 'aaaa','9/1/2017 12:56:00 AM'),
    (2, 'bbbb','9/2/2017 5:23:00 AM'),
    (3, 'cccc','9/3/2017 2:36:00 AM'),
    (4, 'dddd','9/4/2017 3:21:00 AM'),
    (5, 'eeee','9/5/2017 8:06:00 AM');
    ```
    Dans ce didacticiel, vous allez utiliser LastModifytime comme colonne de filigrane. Les données contenues dans le magasin de source de données sont indiquées dans le tableau suivant :

    ```
    PersonID | Name | LastModifytime
    -------- | ---- | --------------
    1 | aaaa | 2017-09-01 00:56:00.000
    2 | bbbb | 2017-09-02 05:23:00.000
    3 | cccc | 2017-09-03 02:36:00.000
    4 | dddd | 2017-09-04 03:21:00.000
    5 | eeee | 2017-09-05 08:06:00.000
    ```

### <a name="create-another-table-in-your-sql-database-to-store-the-high-watermark-value"></a>Créer une autre table dans la base de données SQL pour stocker la valeur de filigrane supérieure
1. Exécutez la commande SQL suivante sur votre base de données SQL pour créer une table sous le nom `watermarktable` pour stocker la valeur de filigrane :  

    ```sql
    create table watermarktable
    (

    TableName varchar(255),
    WatermarkValue datetime,
    );
    ```
2. Définissez la valeur par défaut du filigrane supérieur avec le nom de table du magasin de données source. Dans ce didacticiel, le nom de table est data_source_table.

    ```sql
    INSERT INTO watermarktable
    VALUES ('data_source_table','1/1/2010 12:00:00 AM')    
    ```
3. Passez en revue les données contenues dans la table `watermarktable`.

    ```sql
    Select * from watermarktable
    ```
    Sortie :

    ```
    TableName  | WatermarkValue
    ----------  | --------------
    data_source_table | 2010-01-01 00:00:00.000
    ```

### <a name="create-a-stored-procedure-in-your-sql-database"></a>Créer une procédure stockée dans la base de données SQL

Exécutez la commande suivante pour créer une procédure stockée dans votre base de données SQL :

```sql
CREATE PROCEDURE usp_write_watermark @LastModifiedtime datetime, @TableName varchar(50)
AS

BEGIN

UPDATE watermarktable
SET [WatermarkValue] = @LastModifiedtime
WHERE [TableName] = @TableName

END
```

## <a name="create-a-data-factory"></a>Créer une fabrique de données
1. Définissez une variable pour le nom du groupe de ressources que vous utiliserez ultérieurement dans les commandes PowerShell. Copiez le texte de commande suivant dans PowerShell, spécifiez un nom pour le [groupe de ressources Azure](../azure-resource-manager/management/overview.md) entre des guillemets doubles, puis exécutez la commande. par exemple `"adfrg"`. 
   
     ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup";
    ```

    Si le groupe de ressources existe déjà, vous pouvez ne pas le remplacer. Attribuez une valeur différente à la variable `$resourceGroupName`, puis réexécutez la commande.

2. Définissez une variable pour l’emplacement de la fabrique de données.

    ```powershell
    $location = "East US"
    ```
3. Pour créer le groupe de ressources Azure, exécutez la commande suivante :

    ```powershell
    New-AzResourceGroup $resourceGroupName $location
    ```
    Si le groupe de ressources existe déjà, vous pouvez ne pas le remplacer. Attribuez une valeur différente à la variable `$resourceGroupName`, puis réexécutez la commande.

4. Définissez une variable pour le nom de la fabrique de données.

    > [!IMPORTANT]
    >  Mettez à jour le nom de la fabrique de données pour le rendre globalement unique. Par exemple, ADFTutorialFactorySP1127.

    ```powershell
    $dataFactoryName = "ADFIncCopyTutorialFactory";
    ```
5. Pour créer la fabrique de données, exécutez la cmdlet **Set-AzDataFactoryV2** suivante :

    ```powershell       
    Set-AzDataFactoryV2 -ResourceGroupName $resourceGroupName -Location "East US" -Name $dataFactoryName
    ```

Notez les points suivants :

* Le nom de la fabrique de données doit être un nom global unique. Si vous recevez l’erreur suivante, changez le nom, puis réessayez :

    ```
    The specified Data Factory name 'ADFv2QuickStartDataFactory' is already in use. Data Factory names must be globally unique.
    ```

* Pour créer des instances Data Factory, le compte d’utilisateur que vous utilisez pour vous connecter à Azure doit être membre des rôles Contributeur ou Propriétaire, ou administrateur de l’abonnement Azure.
* Pour obtenir la liste des régions Azure dans lesquelles Data Factory est actuellement disponible, sélectionnez les régions qui vous intéressent dans la page suivante, puis développez **Analytique** pour localiser **Data Factory** : [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/). Les magasins de données (Stockage, SQL Database, Azure SQL Managed Instance, etc.) et les services de calcul (Azure HDInsight, etc.) utilisés par la fabrique de données peuvent se trouver dans d’autres régions.


## <a name="create-linked-services"></a>Créez des services liés
Vous allez créer des services liés dans une fabrique de données pour lier vos magasins de données et vos services de calcul à la fabrique de données. Dans cette section, vous créez des services liés à votre compte de stockage et à SQL Database.

### <a name="create-a-storage-linked-service"></a>Créer un service lié pour le stockage
1. Créez un fichier JSON sous le nom AzureStorageLinkedService.json dans le dossier C:\ADF avec le contenu suivant. (Créez le dossier ADF s’il n’existe pas déjà.) Remplacez `<accountName>` et `<accountKey>` par le nom et la clé de votre compte de stockage avant d’enregistrer le fichier.

    ```json
    {
        "name": "AzureStorageLinkedService",
        "properties": {
            "type": "AzureStorage",
            "typeProperties": {
                "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountName>;AccountKey=<accountKey>"
            }
        }
    }
    ```
2. Dans PowerShell, accédez au dossier ADF.

3. Exécutez la cmdlet **Set-AzDataFactoryV2LinkedService** pour créer le service lié AzureStorageLinkedService. Dans l’exemple suivant, vous transmettez les valeurs des paramètres *ResourceGroupName* et *DataFactoryName* :

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureStorageLinkedService" -File ".\AzureStorageLinkedService.json"
    ```

    Voici l'exemple de sortie :

    ```console
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureStorageLinkedService
    ```

### <a name="create-a-sql-database-linked-service"></a>Créer un service lié à SQL Database
1. Créez un fichier JSON nommé AzureSQLDatabaseLinkedService.json dans le dossier C:\ADF avec le contenu suivant. (Créez le dossier ADF s’il n’existe pas déjà.) Avant d’enregistrer le fichier, remplacez &lt;server&gt;, &lt;database&gt;, &lt;user id&gt; et &lt;password&gt; par le nom de votre serveur, le nom de votre base de données, l’ID utilisateur et le mot de passe.

    ```json
    {
        "name": "AzureSQLDatabaseLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server = tcp:<server>.database.windows.net,1433;Initial Catalog=<database>; Persist Security Info=False; User ID=<user> ; Password=<password>; MultipleActiveResultSets = False; Encrypt = True; TrustServerCertificate = False; Connection Timeout = 30;"
            }
        }
    }
    ```
2. Dans PowerShell, accédez au dossier ADF.

3. Exécutez l’applet de commande **Set-AzDataFactoryV2LinkedService** pour créer le service lié AzureSQLDatabaseLinkedService.

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSQLDatabaseLinkedService" -File ".\AzureSQLDatabaseLinkedService.json"
    ```

    Voici l'exemple de sortie :

    ```console
    LinkedServiceName : AzureSQLDatabaseLinkedService
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDatabaseLinkedService
    ProvisioningState :
    ```

## <a name="create-datasets"></a>Créez les jeux de données
Dans cette étape, vous allez créer des jeux de données pour représenter les données sources et de réception.

### <a name="create-a-source-dataset"></a>Créer un jeu de données source

1. Créez un fichier JSON sous le nom SourceDataset.json dans le même dossier avec le contenu suivant :

    ```json
    {
        "name": "SourceDataset",
        "properties": {
            "type": "AzureSqlTable",
            "typeProperties": {
                "tableName": "data_source_table"
            },
            "linkedServiceName": {
                "referenceName": "AzureSQLDatabaseLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }

    ```
    Dans ce didacticiel, vous allez utiliser le nom de table data_source_table. Remplacez-le si vous utilisez une table d’un autre nom.

2. Exécutez la cmdlet **Set-AzDataFactoryV2Dataset** pour créer le jeu de données SourceDataset.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SourceDataset" -File ".\SourceDataset.json"
    ```

    Voici l’exemple de sortie de l’applet de commande :

    ```json
    DatasetName       : SourceDataset
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset
    ```

### <a name="create-a-sink-dataset"></a>Créer un jeu de données récepteur

1. Créez un fichier JSON sous le nom SinkDataset.json dans le même dossier avec le contenu suivant :

    ```json
    {
        "name": "SinkDataset",
        "properties": {
            "type": "AzureBlob",
            "typeProperties": {
                "folderPath": "adftutorial/incrementalcopy",
                "fileName": "@CONCAT('Incremental-', pipeline().RunId, '.txt')",
                "format": {
                    "type": "TextFormat"
                }
            },
            "linkedServiceName": {
                "referenceName": "AzureStorageLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }   
    ```

    > [!IMPORTANT]
    > Cet extrait suppose que vous disposez d’un conteneur d’objets blob nommé `adftutorial` dans votre stockage d’objets blob. Créez le conteneur s’il n’existe pas ou attribuez-lui le nom d’un conteneur existant. Le dossier de sortie `incrementalcopy` est automatiquement créé s’il n’existe pas dans le conteneur. Dans ce didacticiel, le nom de fichier est généré dynamiquement à l’aide de l’expression `@CONCAT('Incremental-', pipeline().RunId, '.txt')`.

2. Exécutez la cmdlet **Set-AzDataFactoryV2Dataset** pour créer le jeu de données SourceDataset.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SinkDataset" -File ".\SinkDataset.json"
    ```

    Voici l’exemple de sortie de l’applet de commande :

    ```json
    DatasetName       : SinkDataset
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureBlobDataset    
    ```

## <a name="create-a-dataset-for-a-watermark"></a>Créer un jeu de données pour un filigrane
Dans cette étape, vous allez créer un jeu de données pour stocker une valeur de limite supérieure.

1. Créez un fichier JSON sous le nom WatermarkDataset.json dans le même dossier avec le contenu suivant :

    ```json
    {
        "name": " WatermarkDataset ",
        "properties": {
            "type": "AzureSqlTable",
            "typeProperties": {
                "tableName": "watermarktable"
            },
            "linkedServiceName": {
                "referenceName": "AzureSQLDatabaseLinkedService",
                "type": "LinkedServiceReference"
            }
        }
    }    
    ```
2.  Exécutez la cmdlet **Set-AzDataFactoryV2Dataset** pour créer le jeu de données WatermarkDataset.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "WatermarkDataset" -File ".\WatermarkDataset.json"
    ```

    Voici l’exemple de sortie de l’applet de commande :

    ```json
    DatasetName       : WatermarkDataset
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset    
    ```

## <a name="create-a-pipeline"></a>Créer un pipeline
Dans ce didacticiel, vous allez créer un pipeline avec deux activités de recherche, une activité de copie et une activité StoredProcedure chaînées dans le même pipeline.


1. Créez un fichier JSON sous le nom IncrementalCopyPipeline.json dans le même dossier avec le contenu suivant :

    ```json
    {
        "name": "IncrementalCopyPipeline",
        "properties": {
            "activities": [
                {
                    "name": "LookupOldWaterMarkActivity",
                    "type": "Lookup",
                    "typeProperties": {
                        "source": {
                        "type": "SqlSource",
                        "sqlReaderQuery": "select * from watermarktable"
                        },

                        "dataset": {
                        "referenceName": "WatermarkDataset",
                        "type": "DatasetReference"
                        }
                    }
                },
                {
                    "name": "LookupNewWaterMarkActivity",
                    "type": "Lookup",
                    "typeProperties": {
                        "source": {
                            "type": "SqlSource",
                            "sqlReaderQuery": "select MAX(LastModifytime) as NewWatermarkvalue from data_source_table"
                        },

                        "dataset": {
                        "referenceName": "SourceDataset",
                        "type": "DatasetReference"
                        }
                    }
                },

                {
                    "name": "IncrementalCopyActivity",
                    "type": "Copy",
                    "typeProperties": {
                        "source": {
                            "type": "SqlSource",
                            "sqlReaderQuery": "select * from data_source_table where LastModifytime > '@{activity('LookupOldWaterMarkActivity').output.firstRow.WatermarkValue}' and LastModifytime <= '@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}'"
                        },
                        "sink": {
                            "type": "BlobSink"
                        }
                    },
                    "dependsOn": [
                        {
                            "activity": "LookupNewWaterMarkActivity",
                            "dependencyConditions": [
                                "Succeeded"
                            ]
                        },
                        {
                            "activity": "LookupOldWaterMarkActivity",
                            "dependencyConditions": [
                                "Succeeded"
                            ]
                        }
                    ],

                    "inputs": [
                        {
                            "referenceName": "SourceDataset",
                            "type": "DatasetReference"
                        }
                    ],
                    "outputs": [
                        {
                            "referenceName": "SinkDataset",
                            "type": "DatasetReference"
                        }
                    ]
                },

                {
                    "name": "StoredProceduretoWriteWatermarkActivity",
                    "type": "SqlServerStoredProcedure",
                    "typeProperties": {

                        "storedProcedureName": "usp_write_watermark",
                        "storedProcedureParameters": {
                            "LastModifiedtime": {"value": "@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}", "type": "datetime" },
                            "TableName":  { "value":"@{activity('LookupOldWaterMarkActivity').output.firstRow.TableName}", "type":"String"}
                        }
                    },

                    "linkedServiceName": {
                        "referenceName": "AzureSQLDatabaseLinkedService",
                        "type": "LinkedServiceReference"
                    },

                    "dependsOn": [
                        {
                            "activity": "IncrementalCopyActivity",
                            "dependencyConditions": [
                                "Succeeded"
                            ]
                        }
                    ]
                }
            ]

        }
    }
    ```


2. Exécutez la cmdlet **Set-AzDataFactoryV2Pipeline** pour créer le pipeline IncrementalCopyPipeline.

   ```powershell
   Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "IncrementalCopyPipeline" -File ".\IncrementalCopyPipeline.json"
   ```

   Voici l'exemple de sortie :

   ```console
    PipelineName      : IncrementalCopyPipeline
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    Activities        : {LookupOldWaterMarkActivity, LookupNewWaterMarkActivity, IncrementalCopyActivity, StoredProceduretoWriteWatermarkActivity}
    Parameters        :
   ```

## <a name="run-the-pipeline"></a>Exécuter le pipeline

1. Exécutez le pipeline IncrementalCopyPipeline en utilisant la cmdlet **Invoke-AzDataFactoryV2Pipeline**. Remplacez les espaces réservés par les noms de votre groupe de ressources et de votre fabrique de données.

    ```powershell
    $RunId = Invoke-AzDataFactoryV2Pipeline -PipelineName "IncrementalCopyPipeline" -ResourceGroupName $resourceGroupName -dataFactoryName $dataFactoryName
    ```
2. Vérifiez l’état du pipeline en exécutant l’applet de commande **Get-AzDataFactoryV2ActivityRun** jusqu’à ce que toutes les activités s’exécutent correctement. Remplacez les espaces réservés des paramètres *RunStartedAfter* et *RunStartedBefore* par les dates qui vous conviennent. Dans ce didacticiel, vous utilisez *-RunStartedAfter « 2017/09/14 »* et *-RunStartedBefore « 2017/09/15 »* .

    ```powershell
    Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $RunId -RunStartedAfter "<start time>" -RunStartedBefore "<end time>"
    ```

    Voici l'exemple de sortie :

    ```console
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : LookupNewWaterMarkActivity
    PipelineRunId     : d4bf3ce2-5d60-43f3-9318-923155f61037
    PipelineName      : IncrementalCopyPipeline
    Input             : {source, dataset}
    Output            : {NewWatermarkvalue}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 7:42:42 AM
    ActivityRunEnd    : 9/14/2017 7:42:50 AM
    DurationInMs      : 7777
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}

    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : LookupOldWaterMarkActivity
    PipelineRunId     : d4bf3ce2-5d60-43f3-9318-923155f61037
    PipelineName      : IncrementalCopyPipeline
    Input             : {source, dataset}
    Output            : {TableName, WatermarkValue}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 7:42:42 AM
    ActivityRunEnd    : 9/14/2017 7:43:07 AM
    DurationInMs      : 25437
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}

    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : IncrementalCopyActivity
    PipelineRunId     : d4bf3ce2-5d60-43f3-9318-923155f61037
    PipelineName      : IncrementalCopyPipeline
    Input             : {source, sink}
    Output            : {dataRead, dataWritten, rowsCopied, copyDuration...}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 7:43:10 AM
    ActivityRunEnd    : 9/14/2017 7:43:29 AM
    DurationInMs      : 19769
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}

    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : StoredProceduretoWriteWatermarkActivity
    PipelineRunId     : d4bf3ce2-5d60-43f3-9318-923155f61037
    PipelineName      : IncrementalCopyPipeline
    Input             : {storedProcedureName, storedProcedureParameters}
    Output            : {}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 7:43:32 AM
    ActivityRunEnd    : 9/14/2017 7:43:47 AM
    DurationInMs      : 14467
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}

    ```

## <a name="review-the-results"></a>Passer en revue les résultats.

1. Dans le stockage d’objets blob (magasin récepteur), vous constatez que les données ont été copiées dans le fichier défini dans SinkDataset. Dans le didacticiel actuel, le nom de fichier est `Incremental- d4bf3ce2-5d60-43f3-9318-923155f61037.txt`. Ouvrez le fichier. Vous constatez alors que les enregistrements du fichier sont les mêmes que les données contenues dans la base de données SQL.

    ```
    1,aaaa,2017-09-01 00:56:00.0000000
    2,bbbb,2017-09-02 05:23:00.0000000
    3,cccc,2017-09-03 02:36:00.0000000
    4,dddd,2017-09-04 03:21:00.0000000
    5,eeee,2017-09-05 08:06:00.0000000
    ```
2. Vérifiez la valeur la plus récente dans `watermarktable`. Vous constatez que la valeur de filigrane a été mise à jour.

    ```sql
    Select * from watermarktable
    ```

    Voici l'exemple de sortie :

    TableName | WatermarkValue
    --------- | --------------
    data_source_table | 2017-09-05 8:06:00.000

### <a name="insert-data-into-the-data-source-store-to-verify-delta-data-loading"></a>Insérer des données dans le magasin de source de données pour vérifier le chargement des données delta

1. Insérez de nouvelles données dans la base de données SQL (magasin de source de données).

    ```sql
    INSERT INTO data_source_table
    VALUES (6, 'newdata','9/6/2017 2:23:00 AM')

    INSERT INTO data_source_table
    VALUES (7, 'newdata','9/7/2017 9:01:00 AM')
    ```

    Données mises à jour dans la base de données SQL :

    ```
    PersonID | Name | LastModifytime
    -------- | ---- | --------------
    1 | aaaa | 2017-09-01 00:56:00.000
    2 | bbbb | 2017-09-02 05:23:00.000
    3 | cccc | 2017-09-03 02:36:00.000
    4 | dddd | 2017-09-04 03:21:00.000
    5 | eeee | 2017-09-05 08:06:00.000
    6 | newdata | 2017-09-06 02:23:00.000
    7 | newdata | 2017-09-07 09:01:00.000
    ```
2. Réexécutez le pipeline IncrementalCopyPipeline en utilisant l’applet de commande **Invoke-AzDataFactoryV2Pipeline**. Remplacez les espaces réservés par les noms de votre groupe de ressources et de votre fabrique de données.

    ```powershell
    $RunId = Invoke-AzDataFactoryV2Pipeline -PipelineName "IncrementalCopyPipeline" -ResourceGroupName $resourceGroupName -dataFactoryName $dataFactoryName
    ```
3. Vérifiez l’état du pipeline en exécutant l’applet de commande **Get-AzDataFactoryV2ActivityRun** jusqu’à ce que toutes les activités s’exécutent correctement. Remplacez les espaces réservés des paramètres *RunStartedAfter* et *RunStartedBefore* par les dates qui vous conviennent. Dans ce didacticiel, vous utilisez *-RunStartedAfter « 2017/09/14 »* et *-RunStartedBefore « 2017/09/15 »* .

    ```powershell
    Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $RunId -RunStartedAfter "<start time>" -RunStartedBefore "<end time>"
    ```

    Voici l'exemple de sortie :

    ```console
    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : LookupNewWaterMarkActivity
    PipelineRunId     : 2fc90ab8-d42c-4583-aa64-755dba9925d7
    PipelineName      : IncrementalCopyPipeline
    Input             : {source, dataset}
    Output            : {NewWatermarkvalue}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 8:52:26 AM
    ActivityRunEnd    : 9/14/2017 8:52:58 AM
    DurationInMs      : 31758
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}

    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : LookupOldWaterMarkActivity
    PipelineRunId     : 2fc90ab8-d42c-4583-aa64-755dba9925d7
    PipelineName      : IncrementalCopyPipeline
    Input             : {source, dataset}
    Output            : {TableName, WatermarkValue}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 8:52:26 AM
    ActivityRunEnd    : 9/14/2017 8:52:52 AM
    DurationInMs      : 25497
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}

    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : IncrementalCopyActivity
    PipelineRunId     : 2fc90ab8-d42c-4583-aa64-755dba9925d7
    PipelineName      : IncrementalCopyPipeline
    Input             : {source, sink}
    Output            : {dataRead, dataWritten, rowsCopied, copyDuration...}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 8:53:00 AM
    ActivityRunEnd    : 9/14/2017 8:53:20 AM
    DurationInMs      : 20194
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}

    ResourceGroupName : ADF
    DataFactoryName   : incrementalloadingADF
    ActivityName      : StoredProceduretoWriteWatermarkActivity
    PipelineRunId     : 2fc90ab8-d42c-4583-aa64-755dba9925d7
    PipelineName      : IncrementalCopyPipeline
    Input             : {storedProcedureName, storedProcedureParameters}
    Output            : {}
    LinkedServiceName :
    ActivityRunStart  : 9/14/2017 8:53:23 AM
    ActivityRunEnd    : 9/14/2017 8:53:41 AM
    DurationInMs      : 18502
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}

    ```
4. Dans le stockage d’objets blob, vous constatez qu’un autre fichier a été créé. Dans ce didacticiel, le nom du nouveau fichier est `Incremental-2fc90ab8-d42c-4583-aa64-755dba9925d7.txt`. Ouvrez ce fichier. Vous constatez alors qu’il contient deux lignes d’enregistrements.

5. Vérifiez la valeur la plus récente dans `watermarktable`. Vous constatez que la valeur de filigrane a été de nouveau mise à jour.

    ```sql
    Select * from watermarktable
    ```
    Exemple de sortie :

    TableName | WatermarkValue
    --------- | ---------------
    data_source_table | 2017-09-07 09:01:00.000


## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez effectué les étapes suivantes :

> [!div class="checklist"]
> * Préparer le magasin de données pour y stocker la valeur de limite.
> * Créer une fabrique de données.
> * créez des services liés.
> * Créer des jeux de données source, récepteur et filigrane.
> * Créer un pipeline.
> * Exécuter le pipeline.
> * Surveiller l’exécution du pipeline.

Dans ce tutoriel, le pipeline a copié les données d’une table unique située dans Azure SQL Database vers un stockage Blob. Passez au tutoriel suivant pour découvrir comment copier les données de plusieurs tables d’une base de données SQL Server vers SQL Database.

> [!div class="nextstepaction"]
>[Charger de façon incrémentielle des données provenant de plusieurs tables dans SQL Server vers Azure SQL Database](tutorial-incremental-copy-multiple-tables-powershell.md)
