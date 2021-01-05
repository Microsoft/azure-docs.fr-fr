---
title: Copier des données en bloc avec PowerShell
description: Utilisez Azure Data Factory et l’activité de copie pour copier en bloc les données d’un magasin de données source dans un magasin de données de destination.
services: data-factory
author: linda33wj
ms.author: jingwang
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: seo-lt-2019
ms.date: 01/22/2018
ms.openlocfilehash: bf40353a8f29200ab2a33859473dbc504c29bf7d
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97510432"
---
# <a name="copy-multiple-tables-in-bulk-by-using-azure-data-factory-using-powershell"></a>Copier plusieurs tables en bloc en utilisant Azure Data Factory avec PowerShell

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Ce tutoriel montre comment **copier des tables d’Azure SQL Database dans Azure Synapse Analytics**. Vous pouvez appliquer le même modèle à d’autres scénarios de copie. Par exemple : copie de tables à partir de SQL Server/Oracle dans Azure SQL Database/Data Warehouse/Azure Blob, copie de différents chemins à partir de Blob dans des tables Azure SQL Database.

Globalement, ce tutoriel implique les étapes suivantes :

> [!div class="checklist"]
> * Créer une fabrique de données.
> * Créez des services liés Azure SQL Database, Azure Synapse Analytics et Stockage Azure.
> * Créer des jeux de données Azure SQL Database et Azure Synapse Analytics
> * Créer un pipeline pour rechercher les tables à copier et un autre pipeline pour effectuer l’opération de copie. 
> * Démarrer une exécution de pipeline.
> * Surveiller les exécutions de pipeline et d’activité.

Ce tutoriel utilise Azure PowerShell. Pour en savoir plus sur l’utilisation d’autres outils/SDK pour créer une fabrique de données, consultez [Démarrages rapides](quickstart-create-data-factory-dot-net.md). 

## <a name="end-to-end-workflow"></a>Workflow de bout en bout
Dans ce scénario, nous disposons d’un certain nombre de tables dans Azure SQL Database que nous souhaitons copier dans Azure Synapse Analytics. Voici l’ordre logique des étapes du workflow qui se produit dans les pipelines :

![Workflow](media/tutorial-bulk-copy/tutorial-copy-multiple-tables.png)

* Le premier pipeline recherche la liste des tables à copier dans les banques de données du récepteur.  Vous pouvez également conserver une table de métadonnées qui répertorie toutes les tables à copier dans la banque de données du récepteur. Le pipeline déclenche ensuite un autre pipeline qui itère chaque table dans la base de données et effectue l’opération de copie de données.
* Le second pipeline effectue la copie. Il prend la liste des tables comme paramètre. Pour obtenir de meilleurs résultats, copiez chaque table de la liste entre Azure SQL Database et la table correspondante dans Azure Synapse Analytics à l’aide d’une [copie intermédiaire effectuée via le Stockage Blob et PolyBase](connector-azure-sql-data-warehouse.md#use-polybase-to-load-data-into-azure-synapse-analytics). Dans cet exemple, le premier pipeline passe la liste des tables comme valeur pour le paramètre. 

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* **Azure PowerShell**. Suivez les instructions de la page [Installation et configuration d’Azure PowerShell](/powershell/azure/install-Az-ps).
* **Compte Stockage Azure**. Le compte Stockage Azure est utilisé comme stockage d’objets blob intermédiaire dans l’opération de copie en bloc. 
* **Azure SQL Database**. Cette base de données contient les données sources. 
* **Azure Synapse Analytics**. Cet entrepôt de données conserve les données copiées à partir de SQL Database. 

### <a name="prepare-sql-database-and-azure-synapse-analytics"></a>Préparer SQL Database et Azure Synapse Analytics

**Préparer la base de données Azure SQL source** :

Créez une base de données avec l’exemple de données Adventure Works LT dans SQL Database en suivant les instructions données dans l’article [Créer une base de données dans Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md). Ce tutoriel copie toutes les tables de cet exemple de base de données dans un magasin de données Azure Synapse Analytics.

**Préparer le récepteur Azure Synapse Analytics**  :

1. Si vous n’avez pas d’espace de travail Azure Synapse Analytics, consultez l’article [Bien démarrer avec Azure Synapse Analytics](..\synapse-analytics\get-started.md) pour savoir comment en créer un.

2. Créez les schémas de table correspondants dans Azure Synapse Analytics. Plus tard, vous utiliserez Azure Data Factory pour migrer/copier les données.

## <a name="azure-services-to-access-sql-server"></a>Services Azure pour accéder au serveur SQL

Pour SQL Database et Azure Synapse Analytics, autorisez les services Azure à accéder au serveur SQL. Vérifiez que le paramètre **Autoriser l’accès aux services Azure** est **activé** sur votre serveur. Ce paramètre permet au service Data Factory de lire les données d’Azure SQL Database et de les écrire dans le magasin de données Azure Synapse Analytics. Pour vérifier et activer ce paramètre, procédez comme suit :

1. Cliquez sur **Tous les services** sur la gauche, puis sur **Serveurs SQL**.
2. Sélectionnez votre serveur, puis cliquez sur **Pare-feu** sous **PARAMÈTRES**.
3. Dans la page **Paramètres de pare-feu**, cliquez sur **ACTIVER** pour **Autoriser l’accès aux services Azure**.

## <a name="create-a-data-factory"></a>Créer une fabrique de données

1. Lancez **PowerShell**. Conservez Azure PowerShell ouvert jusqu’à la fin de ce tutoriel. Si vous fermez puis rouvrez Azure PowerShell, vous devez réexécuter ces commandes.

    Exécutez la commande suivante, puis saisissez le nom d’utilisateur et le mot de passe que vous avez utilisés pour la connexion au portail Azure :
        
    ```powershell
    Connect-AzAccount
    ```
    Exécutez la commande suivante pour afficher tous les abonnements de ce compte :

    ```powershell
    Get-AzSubscription
    ```
    Exécutez la commande suivante pour sélectionner l’abonnement que vous souhaitez utiliser. Remplacez **SubscriptionId** par l’ID de votre abonnement Azure :

    ```powershell
    Select-AzSubscription -SubscriptionId "<SubscriptionId>"
    ```
2. Exécutez la cmdlet **Set-AzDataFactoryV2** pour créer une fabrique de données. Avant d’exécuter la commande, remplacez les espaces réservés par vos propres valeurs. 

    ```powershell
    $resourceGroupName = "<your resource group to create the factory>"
    $dataFactoryName = "<specify the name of data factory to create. It must be globally unique.>"
    Set-AzDataFactoryV2 -ResourceGroupName $resourceGroupName -Location "East US" -Name $dataFactoryName
    ```

    Notez les points suivants :

    * Le nom de la fabrique de données Azure doit être un nom global unique. Si vous recevez l’erreur suivante, changez le nom, puis réessayez.

        ```
        The specified Data Factory name 'ADFv2QuickStartDataFactory' is already in use. Data Factory names must be globally unique.
        ```

    * Pour créer des instances de fabrique de données, vous devez être un administrateur/collaborateur de l’abonnement Azure.
    * Pour obtenir la liste des régions Azure dans lesquelles Data Factory est actuellement disponible, sélectionnez les régions qui vous intéressent dans la page suivante, puis développez **Analytique** pour localiser **Data Factory** : [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/). Les magasins de données (Stockage Azure, Azure SQL Database, etc.) et les services de calcul (HDInsight, etc.) utilisés par la fabrique de données peuvent être proposés dans d’autres régions.

## <a name="create-linked-services"></a>Créez des services liés

Dans ce tutoriel, vous allez créer trois services liés (un pour la source, un pour le récepteur et un pour l’objet blob intermédiaire), avec des connexions à vos magasins de données :

### <a name="create-the-source-azure-sql-database-linked-service"></a>Créer le service lié Azure SQL Database pour la source

1. Créez un fichier JSON nommé **AzureSqlDatabaseLinkedService.json** dans le dossier **C:\ADFv2TutorialBulkCopy** avec le contenu suivant : (Créez le dossier ADFv2TutorialBulkCopy s’il n’existe pas déjà.)

    > [!IMPORTANT]
    > Remplacez &lt;servername&gt;, &lt;databasename&gt;, &lt;username&gt;@&lt;servername&gt; et &lt;password&gt; par les valeurs de votre base de données Azure SQL avant d’enregistrer le fichier.

    ```json
    {
        "name": "AzureSqlDatabaseLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
    }
    ```

2. Dans **Azure PowerShell**, passez au dossier **ADFv2TutorialBulkCopy**.

3. Exécutez la cmdlet **Set-AzDataFactoryV2LinkedService** pour créer le service lié : **AzureSqlDatabaseLinkedService**. 

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSqlDatabaseLinkedService" -File ".\AzureSqlDatabaseLinkedService.json"
    ```

    Voici l'exemple de sortie :

    ```console
    LinkedServiceName : AzureSqlDatabaseLinkedService
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDatabaseLinkedService
    ```

### <a name="create-the-sink-azure-synapse-analytics-linked-service"></a>Créer un service lié du récepteur Azure Synapse Analytics

1. Créez un fichier JSON nommé **AzureSqlDWLinkedService.json** dans le dossier **C:\ADFv2TutorialBulkCopy** avec le contenu suivant :

    > [!IMPORTANT]
    > Remplacez &lt;servername&gt;, &lt;databasename&gt;, &lt;username&gt;@&lt;servername&gt; et &lt;password&gt; par les valeurs de votre base de données Azure SQL avant d’enregistrer le fichier.

    ```json
    {
        "name": "AzureSqlDWLinkedService",
        "properties": {
            "type": "AzureSqlDW",
            "typeProperties": {
                "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
    }
    ```

2. Pour créer le service lié : **AzureSqlDWLinkedService**, exécutez la cmdlet **Set-AzDataFactoryV2LinkedService**.

    ```powershell
    Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSqlDWLinkedService" -File ".\AzureSqlDWLinkedService.json"
    ```

    Voici l'exemple de sortie :

    ```console
    LinkedServiceName : AzureSqlDWLinkedService
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDWLinkedService
    ```

### <a name="create-the-staging-azure-storage-linked-service"></a>Créer le service lié Stockage Azure intermédiaire

Dans ce tutoriel, vous allez utiliser Stockage Blob Azure comme zone intermédiaire pour améliorer les performances de copie de PolyBase.

1. Créez un fichier JSON nommé **AzureSqlDWLinkedService.json** dans le dossier **C:\ADFv2TutorialBulkCopy** avec le contenu suivant :

    > [!IMPORTANT]
    > Remplacez &lt;accountName&gt; et &lt;accountKey&gt; par le nom et la clé de votre compte de stockage Azure avant d’enregistrer le fichier.

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

2. Pour créer le service lié : **AzureStorageLinkedService**, exécutez la cmdlet **Set-AzDataFactoryV2LinkedService**.

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

## <a name="create-datasets"></a>Créez les jeux de données

Dans ce tutoriel, vous créez des jeux de données (source et récepteur) qui spécifient l’emplacement de stockage des données :

### <a name="create-a-dataset-for-source-sql-database"></a>Créer un jeu de données pour la base de données SQL source

1. Créez un fichier JSON nommé **AzureSqlDatabaseDataset.json** dans le dossier **C:\ADFv2TutorialBulkCopy** avec le contenu suivant : « tableName » est un nom factice. Plus tard, vous utiliserez la requête SQL dans l’activité de copie pour récupérer des données.

    ```json
    {
        "name": "AzureSqlDatabaseDataset",
        "properties": {
            "type": "AzureSqlTable",
            "linkedServiceName": {
                "referenceName": "AzureSqlDatabaseLinkedService",
                "type": "LinkedServiceReference"
            },
            "typeProperties": {
                "tableName": "dummy"
            }
        }
    }
    ```

2. Pour créer le jeu de données : **AzureSqlDatabaseDataset**, exécutez la cmdlet **Set-AzDataFactoryV2Dataset**.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSqlDatabaseDataset" -File ".\AzureSqlDatabaseDataset.json"
    ```

    Voici l'exemple de sortie :

    ```console
    DatasetName       : AzureSqlDatabaseDataset
    ResourceGroupName : <resourceGroupname>
    DataFactoryName   : <dataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlTableDataset
    ```

### <a name="create-a-dataset-for-sink-azure-synapse-analytics"></a>Créer un jeu de données pour le récepteur Azure Synapse Analytics

1. Créez un fichier JSON nommé **AzureSqlDWDataset.json** dans le dossier **C:\ADFv2TutorialBulkCopy** avec le contenu suivant : La valeur « tableName » est définie en tant que paramètre. Plus tard, l’activité de copie qui fait référence à ce jeu de données transmet la valeur réelle au jeu de données.

    ```json
    {
        "name": "AzureSqlDWDataset",
        "properties": {
            "type": "AzureSqlDWTable",
            "linkedServiceName": {
                "referenceName": "AzureSqlDWLinkedService",
                "type": "LinkedServiceReference"
            },
            "typeProperties": {
                "tableName": {
                    "value": "@{dataset().DWTableName}",
                    "type": "Expression"
                }
            },
            "parameters":{
                "DWTableName":{
                    "type":"String"
                }
            }
        }
    }
    ```

2. Pour créer le jeu de données : **AzureSqlDWDataset**, exécutez la cmdlet **Set-AzDataFactoryV2Dataset**.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureSqlDWDataset" -File ".\AzureSqlDWDataset.json"
    ```

    Voici l'exemple de sortie :

    ```console
    DatasetName       : AzureSqlDWDataset
    ResourceGroupName : <resourceGroupname>
    DataFactoryName   : <dataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureSqlDwTableDataset
    ```

## <a name="create-pipelines"></a>Créer des pipelines

Dans ce tutoriel, vous allez créer deux pipelines :

### <a name="create-the-pipeline-iterateandcopysqltables"></a>Créer le pipeline « IterateAndCopySQLTables »

Ce pipeline prend une liste de tables comme paramètre. Pour chaque table de la liste, il copie les données de la table Azure SQL Database vers Azure Synapse Analytics en utilisant la copie intermédiaire et PolyBase.

1. Créez un fichier JSON nommé **IterateAndCopySQLTables.json** dans le dossier **C:\ADFv2TutorialBulkCopy** avec le contenu suivant :

    ```json
    {
        "name": "IterateAndCopySQLTables",
        "properties": {
            "activities": [
                {
                    "name": "IterateSQLTables",
                    "type": "ForEach",
                    "typeProperties": {
                        "isSequential": "false",
                        "items": {
                            "value": "@pipeline().parameters.tableList",
                            "type": "Expression"
                        },
                        "activities": [
                            {
                                "name": "CopyData",
                                "description": "Copy data from Azure SQL Database to Azure Synapse Analytics",
                                "type": "Copy",
                                "inputs": [
                                    {
                                        "referenceName": "AzureSqlDatabaseDataset",
                                        "type": "DatasetReference"
                                    }
                                ],
                                "outputs": [
                                    {
                                        "referenceName": "AzureSqlDWDataset",
                                        "type": "DatasetReference",
                                        "parameters": {
                                            "DWTableName": "[@{item().TABLE_SCHEMA}].[@{item().TABLE_NAME}]"
                                        }
                                    }
                                ],
                                "typeProperties": {
                                    "source": {
                                        "type": "SqlSource",
                                        "sqlReaderQuery": "SELECT * FROM [@{item().TABLE_SCHEMA}].[@{item().TABLE_NAME}]"
                                    },
                                    "sink": {
                                        "type": "SqlDWSink",
                                        "preCopyScript": "TRUNCATE TABLE [@{item().TABLE_SCHEMA}].[@{item().TABLE_NAME}]",
                                        "allowPolyBase": true
                                    },
                                    "enableStaging": true,
                                    "stagingSettings": {
                                        "linkedServiceName": {
                                            "referenceName": "AzureStorageLinkedService",
                                            "type": "LinkedServiceReference"
                                        }
                                    }
                                }
                            }
                        ]
                    }
                }
            ],
            "parameters": {
                "tableList": {
                    "type": "Object"
                }
            }
        }
    }
    ```

2. Pour créer le pipeline : **IterateAndCopySQLTables**, exécutez la cmdlet **Set-AzDataFactoryV2Pipeline**.

    ```powershell
    Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "IterateAndCopySQLTables" -File ".\IterateAndCopySQLTables.json"
    ```

    Voici l'exemple de sortie :

    ```console
    PipelineName      : IterateAndCopySQLTables
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Activities        : {IterateSQLTables}
    Parameters        : {[tableList, Microsoft.Azure.Management.DataFactory.Models.ParameterSpecification]}
    ```

### <a name="create-the-pipeline-gettablelistandtriggercopydata"></a>Créer le pipeline « GetTableListAndTriggerCopyData »

Ce pipeline exécute deux étapes :

* Recherche la table système Azure SQL Database pour obtenir la liste des tables à copier.
* Déclenche le pipeline « IterateAndCopySQLTables » pour copier les données.

1. Créez un fichier JSON nommé **GetTableListAndTriggerCopyData.json** dans le dossier **C:\ADFv2TutorialBulkCopy** avec le contenu suivant :

    ```json
    {
        "name":"GetTableListAndTriggerCopyData",
        "properties":{
            "activities":[
                { 
                    "name": "LookupTableList",
                    "description": "Retrieve the table list from Azure SQL dataabse",
                    "type": "Lookup",
                    "typeProperties": {
                        "source": {
                            "type": "SqlSource",
                            "sqlReaderQuery": "SELECT TABLE_SCHEMA, TABLE_NAME FROM information_schema.TABLES WHERE TABLE_TYPE = 'BASE TABLE' and TABLE_SCHEMA = 'SalesLT' and TABLE_NAME <> 'ProductModel'"
                        },
                        "dataset": {
                            "referenceName": "AzureSqlDatabaseDataset",
                            "type": "DatasetReference"
                        },
                        "firstRowOnly": false
                    }
                },
                {
                    "name": "TriggerCopy",
                    "type": "ExecutePipeline",
                    "typeProperties": {
                        "parameters": {
                            "tableList": {
                                "value": "@activity('LookupTableList').output.value",
                                "type": "Expression"
                            }
                        },
                        "pipeline": {
                            "referenceName": "IterateAndCopySQLTables",
                            "type": "PipelineReference"
                        },
                        "waitOnCompletion": true
                    },
                    "dependsOn": [
                        {
                            "activity": "LookupTableList",
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

2. Pour créer le pipeline : **GetTableListAndTriggerCopyData**, exécutez la cmdlet **Set-AzDataFactoryV2Pipeline**.

    ```powershell
    Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "GetTableListAndTriggerCopyData" -File ".\GetTableListAndTriggerCopyData.json"
    ```

    Voici l'exemple de sortie :

    ```console
    PipelineName      : GetTableListAndTriggerCopyData
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Activities        : {LookupTableList, TriggerCopy}
    Parameters        :
    ```

## <a name="start-and-monitor-pipeline-run"></a>Démarrer et surveiller une exécution de pipeline

1. Démarrez une exécution de pipeline pour le pipeline principal « GetTableListAndTriggerCopyData » et capturez l’ID d’exécution du pipeline pour une analyse ultérieure. Ceci déclenche l’exécution du pipeline « IterateAndCopySQLTables », comme indiqué dans l’activité ExecutePipeline.

    ```powershell
    $runId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName 'GetTableListAndTriggerCopyData'
    ```

2.  Exécutez le script suivant pour vérifier en permanence l’état de l’exécution du pipeline **GetTableListAndTriggerCopyData**, puis imprimez le résultat final de l’exécution de pipeline et de l’exécution d’activité.

    ```powershell
    while ($True) {
        $run = Get-AzDataFactoryV2PipelineRun -ResourceGroupName $resourceGroupName -DataFactoryName $DataFactoryName -PipelineRunId $runId

        if ($run) {
            if ($run.Status -ne 'InProgress') {
                Write-Host "Pipeline run finished. The status is: " $run.Status -foregroundcolor "Yellow"
                Write-Host "Pipeline run details:" -foregroundcolor "Yellow"
                $run
                break
            }
            Write-Host  "Pipeline is running...status: InProgress" -foregroundcolor "Yellow"
        }

        Start-Sleep -Seconds 15
    }

    $result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
    Write-Host "Activity run details:" -foregroundcolor "Yellow"
    $result
    ```

    Voici la sortie de l’exemple d’exécution :

    ```console
    Pipeline run details:
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    RunId             : 0000000000-00000-0000-0000-000000000000
    PipelineName      : GetTableListAndTriggerCopyData
    LastUpdated       : 9/18/2017 4:08:15 PM
    Parameters        : {}
    RunStart          : 9/18/2017 4:06:44 PM
    RunEnd            : 9/18/2017 4:08:15 PM
    DurationInMs      : 90637
    Status            : Succeeded
    Message           : 

    Activity run details:
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    ActivityName      : LookupTableList
    PipelineRunId     : 0000000000-00000-0000-0000-000000000000
    PipelineName      : GetTableListAndTriggerCopyData
    Input             : {source, dataset, firstRowOnly}
    Output            : {count, value, effectiveIntegrationRuntime}
    LinkedServiceName : 
    ActivityRunStart  : 9/18/2017 4:06:46 PM
    ActivityRunEnd    : 9/18/2017 4:07:09 PM
    DurationInMs      : 22995
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}

    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    ActivityName      : TriggerCopy
    PipelineRunId     : 0000000000-00000-0000-0000-000000000000
    PipelineName      : GetTableListAndTriggerCopyData
    Input             : {pipeline, parameters, waitOnCompletion}
    Output            : {pipelineRunId}
    LinkedServiceName : 
    ActivityRunStart  : 9/18/2017 4:07:11 PM
    ActivityRunEnd    : 9/18/2017 4:08:14 PM
    DurationInMs      : 62581
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    ```

3. Vous pouvez obtenir l’ID d’exécution du pipeline « **IterateAndCopySQLTables** », puis vérifier le résultat détaillé de l’exécution d’activité comme suit.

    ```powershell
    Write-Host "Pipeline 'IterateAndCopySQLTables' run result:" -foregroundcolor "Yellow"
    ($result | Where-Object {$_.ActivityName -eq "TriggerCopy"}).Output.ToString()
    ```

    Voici la sortie de l’exemple d’exécution :

    ```json
    {
        "pipelineRunId": "7514d165-14bf-41fb-b5fb-789bea6c9e58"
    }
    ```

    ```powershell
    $result2 = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId <copy above run ID> -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
    $result2
    ```

3. Connectez-vous à votre récepteur Azure Synapse Analytics et vérifiez que les données ont bien été copiées à partir d’Azure SQL Database.

## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez effectué les étapes suivantes : 

> [!div class="checklist"]
> * Créer une fabrique de données.
> * Créez des services liés Azure SQL Database, Azure Synapse Analytics et Stockage Azure.
> * Créer des jeux de données Azure SQL Database et Azure Synapse Analytics
> * Créer un pipeline pour rechercher les tables à copier et un autre pipeline pour effectuer l’opération de copie. 
> * Démarrer une exécution de pipeline.
> * Surveiller les exécutions de pipeline et d’activité.

Passez au tutoriel suivant pour découvrir comment copier des données de manière incrémentielle d’une source vers une destination :
> [!div class="nextstepaction"]
>[Copier des données de façon incrémentielle](tutorial-incremental-copy-powershell.md)
