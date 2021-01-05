---
title: 'Tutoriel : Créer un pipeline à l’aide d’un modèle Azure Resource Manager '
description: Dans ce didacticiel, vous créez un pipeline Azure Data Factory en utilisant un modèle Azure Resource Manager. Ce pipeline copie des données d’un stockage Blob Azure dans Azure SQL Database.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: ''
editor: ''
ms.assetid: 1274e11a-e004-4df5-af07-850b2de7c15e
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: a007e64a7bd034397c2030c435a5ad349bd4acc7
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97608746"
---
# <a name="tutorial-use-azure-resource-manager-template-to-create-a-data-factory-pipeline-to-copy-data"></a>Tutoriel : utiliser un modèle Azure Resource Manager pour créer un pipeline Data Factory afin de copier des données 
> [!div class="op_single_selector"]
> * [Vue d’ensemble et composants requis](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Assistant de copie](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Modèle Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)
> 

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez le [tutoriel sur l’activité de copie](../quickstart-create-data-factory-dot-net.md). 

Ce didacticiel vous montre comment utiliser un modèle Azure Resource Manager pour créer une fabrique de données Azure. Dans ce didacticiel, le pipeline de données copie les données d’un magasin de données source vers un magasin de données de destination. Il ne transforme pas les données d’entrée pour produire des données de sortie. Pour suivre un tutoriel sur la transformation des données à l’aide d’Azure Data Factory, consultez [Tutoriel : Créer un pipeline pour transformer des données à l’aide d’un cluster Hadoop](data-factory-build-your-first-pipeline.md).

Dans ce tutoriel, vous créez un pipeline contenant une seule activité : une activité de copie. L’activité de copie copie les données d’un magasin de données pris en charge vers un magasin de données de récepteur pris en charge. Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs, consultez [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Elle est mise en œuvre par un service disponible dans le monde entier, capable de copier des données entre différents magasins de données de façon sécurisée, fiable et évolutive. Pour plus d’informations sur l’activité de copie, consultez [Activités de déplacement des données](data-factory-data-movement-activities.md).

Un pipeline peut contenir plusieurs activités. En outre, vous pouvez chaîner deux activités (une après l’autre) en configurant le jeu de données de sortie d’une activité en tant que jeu de données d’entrée de l’autre activité. Pour plus d’informations, consultez [Plusieurs activités dans un pipeline](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline). 

> [!NOTE] 
> Dans ce didacticiel, le pipeline de données copie les données d’un magasin de données source vers un magasin de données de destination. Pour suivre un tutoriel sur la transformation des données à l’aide d’Azure Data Factory, consultez [Tutoriel : Créer un pipeline pour transformer des données à l’aide d’un cluster Hadoop](data-factory-build-your-first-pipeline.md). 

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Lisez l’article [Vue d’ensemble et étapes préalables requises](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) et effectuez les **étapes préalables requises**.
* Suivez les instructions de l’article [Installation et configuration d’Azure PowerShell](/powershell/azure/) pour installer la dernière version d’Azure PowerShell sur votre ordinateur. Dans ce didacticiel, vous utilisez PowerShell pour déployer des entités Data Factory. 
* (Facultatif) Consultez [Création de modèles Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md) pour en savoir plus sur les modèles Azure Resource Manager.

## <a name="in-this-tutorial"></a>Dans ce didacticiel
Dans ce didacticiel, vous créez une fabrique de données avec les entités Data Factory suivantes :

| Entité | Description |
| --- | --- |
| Service lié Stockage Azure |Lie votre compte Stockage Azure à la fabrique de données. Stockage Azure est le magasin de données source et Azure SQL Database est le magasin de données récepteur de l’activité de copie dans le tutoriel. Il spécifie le compte de stockage contenant les données d’entrée pour l’activité de copie. |
| Service lié Azure SQL Database |Lie notre base de données dans Azure SQL Database à la fabrique de données. Il spécifie la base de données qui conserve les données de sortie de l’activité de copie. |
| Jeu de données d'entrée d'objet Blob Azure |Fait référence au service Stockage Azure lié. Le service lié fait référence à un compte de stockage Azure, et les jeux de données d’objet Blob Azure spécifie le conteneur, le dossier et le nom de fichier dans le stockage contenant les données d’entrée. |
| Jeu de données de sortie SQL Azure |Fait référence au service SQL Azure lié. Le service lié SQL Azure fait référence à un serveur SQL logique et le jeu de données SQL Azure spécifie le nom de la table qui contient les données de sortie. |
| Pipeline de données |Le pipeline a une activité de type copie qui sélectionne le jeu de données d’objets blob Azure comme entrée et le jeu de données SQL Azure comme sortie. L’activité copie les données d’un objet blob Azure vers une table située dans Azure SQL Database. |

Une fabrique de données peut avoir un ou plusieurs pipelines. Un pipeline peut contenir une ou plusieurs activités. Il existe deux types d’activités : les [activités de déplacement des données](data-factory-data-movement-activities.md) et les [activités de transformation des données](data-factory-data-transformation-activities.md). Dans ce didacticiel, vous créez un pipeline avec une activité (activité de copie).

![Copie de données d’un objet blob Azure vers Azure SQL Database](media/data-factory-copy-activity-tutorial-using-azure-resource-manager-template/CopyBlob2SqlDiagram.png) 

La section suivante fournit le modèle Resource Manager complet permettant de définir des entités Data Factory pour que vous puissiez rapidement parcourir le didacticiel et tester le modèle. Pour comprendre comment chaque entité Data Factory est définie, consultez la section [Entités Data Factory dans le modèle](#data-factory-entities-in-the-template).

## <a name="data-factory-json-template"></a>Modèle JSON Data Factory
Le modèle Resource Manager de niveau supérieur pour la définition d’une fabrique de données est : 

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": { ...
    },
    "variables": { ...
    },
    "resources": [
        {
            "name": "[parameters('dataFactoryName')]",
            "apiVersion": "[variables('apiVersion')]",
            "type": "Microsoft.DataFactory/datafactories",
            "location": "westus",
            "resources": [
                { ... },
                { ... },
                { ... },
                { ... }
            ]
        }
    ]
}
```
Créez un fichier JSON nommé **ADFCopyTutorialARM.json** dans le dossier **C:\ADFGetStarted** avec le contenu suivant :

```json
{
    "contentVersion": "1.0.0.0",
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "parameters": {
      "storageAccountName": { "type": "string", "metadata": { "description": "Name of the Azure storage account that contains the data to be copied." } },
      "storageAccountKey": { "type": "securestring", "metadata": { "description": "Key for the Azure storage account." } },
      "sourceBlobContainer": { "type": "string", "metadata": { "description": "Name of the blob container in the Azure Storage account." } },
      "sourceBlobName": { "type": "string", "metadata": { "description": "Name of the blob in the container that has the data to be copied to Azure SQL Database table" } },
      "sqlServerName": { "type": "string", "metadata": { "description": "Name of the logical SQL server that will hold the output/copied data." } },
      "databaseName": { "type": "string", "metadata": { "description": "Name of the Azure SQL Database in the logical SQL server." } },
      "sqlServerUserName": { "type": "string", "metadata": { "description": "Name of the user that has access to the logical SQL server." } },
      "sqlServerPassword": { "type": "securestring", "metadata": { "description": "Password for the user." } },
      "targetSQLTable": { "type": "string", "metadata": { "description": "Table in the Azure SQL Database that will hold the copied data." } 
      } 
    },
    "variables": {
      "dataFactoryName": "[concat('AzureBlobToAzureSQLDatabaseDF', uniqueString(resourceGroup().id))]",
      "azureSqlLinkedServiceName": "AzureSqlLinkedService",
      "azureStorageLinkedServiceName": "AzureStorageLinkedService",
      "blobInputDatasetName": "BlobInputDataset",
      "sqlOutputDatasetName": "SQLOutputDataset",
      "pipelineName": "Blob2SQLPipeline"
    },
    "resources": [
      {
        "name": "[variables('dataFactoryName')]",
        "apiVersion": "2015-10-01",
        "type": "Microsoft.DataFactory/datafactories",
        "location": "West US",
        "resources": [
          {
            "type": "linkedservices",
            "name": "[variables('azureStorageLinkedServiceName')]",
            "dependsOn": [
              "[variables('dataFactoryName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
              "type": "AzureStorage",
              "description": "Azure Storage linked service",
              "typeProperties": {
                "connectionString": "[concat('DefaultEndpointsProtocol=https;AccountName=',parameters('storageAccountName'),';AccountKey=',parameters('storageAccountKey'))]"
              }
            }
          },
          {
            "type": "linkedservices",
            "name": "[variables('azureSqlLinkedServiceName')]",
            "dependsOn": [
              "[variables('dataFactoryName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
              "type": "AzureSqlDatabase",
              "description": "Azure SQL linked service",
              "typeProperties": {
                "connectionString": "[concat('Server=tcp:',parameters('sqlServerName'),'.database.windows.net,1433;Database=', parameters('databaseName'), ';User ID=',parameters('sqlServerUserName'),';Password=',parameters('sqlServerPassword'),';Trusted_Connection=False;Encrypt=True;Connection Timeout=30')]"
              }
            }
          },
          {
            "type": "datasets",
            "name": "[variables('blobInputDatasetName')]",
            "dependsOn": [
              "[variables('dataFactoryName')]",
              "[variables('azureStorageLinkedServiceName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
              "type": "AzureBlob",
              "linkedServiceName": "[variables('azureStorageLinkedServiceName')]",
              "structure": [
                {
                  "name": "Column0",
                  "type": "String"
                },
                {
                  "name": "Column1",
                  "type": "String"
                }
              ],
              "typeProperties": {
                "folderPath": "[concat(parameters('sourceBlobContainer'), '/')]",
                "fileName": "[parameters('sourceBlobName')]",
                "format": {
                  "type": "TextFormat",
                  "columnDelimiter": ","
                }
              },
              "availability": {
                "frequency": "Hour",
                "interval": 1
              },
              "external": true
            }
          },
          {
            "type": "datasets",
            "name": "[variables('sqlOutputDatasetName')]",
            "dependsOn": [
              "[variables('dataFactoryName')]",
              "[variables('azureSqlLinkedServiceName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
              "type": "AzureSqlTable",
              "linkedServiceName": "[variables('azureSqlLinkedServiceName')]",
              "structure": [
                {
                  "name": "FirstName",
                  "type": "String"
                },
                {
                  "name": "LastName",
                  "type": "String"
                }
              ],
              "typeProperties": {
                "tableName": "[parameters('targetSQLTable')]"
              },
              "availability": {
                "frequency": "Hour",
                "interval": 1
              }
            }
          },
          {
            "type": "datapipelines",
            "name": "[variables('pipelineName')]",
            "dependsOn": [
              "[variables('dataFactoryName')]",
              "[variables('azureStorageLinkedServiceName')]",
              "[variables('azureSqlLinkedServiceName')]",
              "[variables('blobInputDatasetName')]",
              "[variables('sqlOutputDatasetName')]"
            ],
            "apiVersion": "2015-10-01",
            "properties": {
              "activities": [
                {
                  "name": "CopyFromAzureBlobToAzureSQL",
                  "description": "Copy data frm Azure blob to Azure SQL",
                  "type": "Copy",
                  "inputs": [
                    {
                      "name": "[variables('blobInputDatasetName')]"
                    }
                  ],
                  "outputs": [
                    {
                      "name": "[variables('sqlOutputDatasetName')]"
                    }
                  ],
                  "typeProperties": {
                    "source": {
                      "type": "BlobSource"
                    },
                    "sink": {
                      "type": "SqlSink",
                      "sqlWriterCleanupScript": "$$Text.Format('DELETE FROM {0}', 'emp')"
                    },
                    "translator": {
                      "type": "TabularTranslator",
                      "columnMappings": "Column0:FirstName,Column1:LastName"
                    }
                  },
                  "Policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 3,
                    "timeout": "01:00:00"
                  }
                }
              ],
              "start": "2017-05-11T00:00:00Z",
              "end": "2017-05-12T00:00:00Z"
            }
          }
        ]
      }
    ]
  }
```

## <a name="parameters-json"></a>Paramètres JSON
Créez un fichier JSON nommé **ADFCopyTutorialARM-Parameters** contient les paramètres du modèle Azure Resource Manager. 

> [!IMPORTANT]
> Spécifiez le nom et la clé de votre compte de stockage Azure pour les paramètres storageAccountName et storageAccountKey.  
> 
> Spécifiez le serveur SQL logique, la base de données, l’utilisateur et le mot de passe pour les paramètres sqlServerName, databaseName, sqlServerUserName et sqlServerPassword.  

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": { 
        "storageAccountName": { "value": "<Name of the Azure storage account>"    },
        "storageAccountKey": {
            "value": "<Key for the Azure storage account>"
        },
        "sourceBlobContainer": { "value": "adftutorial" },
        "sourceBlobName": { "value": "emp.txt" },
        "sqlServerName": { "value": "<Name of the logical SQL server>" },
        "databaseName": { "value": "<Name of the database>" },
        "sqlServerUserName": { "value": "<Name of the user who has access to the database>" },
        "sqlServerPassword": { "value": "<password for the user>" },
        "targetSQLTable": { "value": "emp" }
    }
}
```

> [!IMPORTANT]
> Vous pouvez utiliser des fichiers JSON de paramètres distincts pour les environnements de développement, de test et de production avec le même modèle JSON Data Factory. En utilisant un script PowerShell, vous pouvez automatiser le déploiement des entités Data Factory dans ces environnements.  
> 
> 

## <a name="create-data-factory"></a>Créer une fabrique de données
1. Démarrez **Azure PowerShell** et exécutez la commande suivante :
   * Exécutez la commande suivante, puis saisissez le nom d’utilisateur et le mot de passe que vous avez utilisés pour la connexion au portail Azure.
   
    ```PowerShell
    Connect-AzAccount       
    ```  
   * Exécutez la commande suivante pour afficher tous les abonnements de ce compte.
   
    ```PowerShell
    Get-AzSubscription
    ```   
   * Exécutez la commande suivante pour sélectionner l’abonnement que vous souhaitez utiliser.
    
    ```PowerShell
    Get-AzSubscription -SubscriptionName <SUBSCRIPTION NAME> | Set-AzContext
    ```    
2. Exécutez la commande suivante pour déployer des entités Data Factory à l’aide du modèle Resource Manager que vous avez créé à l’étape 1.

    ```PowerShell   
    New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile C:\ADFGetStarted\ADFCopyTutorialARM.json -TemplateParameterFile C:\ADFGetStarted\ADFCopyTutorialARM-Parameters.json
    ```

## <a name="monitor-pipeline"></a>Surveillance d’un pipeline

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec votre compte Azure.

1. Cliquez sur **Fabriques de données** dans le menu à gauche (ou) cliquez sur **Tous les services** puis sur **Fabriques de données** sous la catégorie **Intelligence et analyse**.
   
    ![Menu Fabriques de données](media/data-factory-copy-activity-tutorial-using-azure-resource-manager-template/data-factories-menu.png)

1. Sur la page **Fabriques de données**, recherchez votre fabrique de données (AzureBlobToAzureSQLDatabaseDF). 
   
    ![Recherche d’une fabrique de données](media/data-factory-copy-activity-tutorial-using-azure-resource-manager-template/search-for-data-factory.png)  

1. Cliquez sur votre fabrique de données Azure. La page d’accueil de la fabrique de données apparaît.
   
    ![Page d’accueil de la fabrique de données](media/data-factory-copy-activity-tutorial-using-azure-resource-manager-template/data-factory-home-page.png)  

1. Suivez les instructions dans [Surveiller les jeux de données et le pipeline](data-factory-monitor-manage-pipelines.md) pour surveiller le pipeline et les jeux de données que vous avez créés dans ce didacticiel. Pour le moment, Visual Studio ne prend pas en charge la surveillance des pipelines Data Factory.

1. Lorsqu’une tranche est à l’état **Prêt**, vérifiez que les données sont copiées vers la table **emp** dans Azure SQL Database.

Pour plus d’informations sur l’utilisation des panneaux du portail Azure afin de surveiller le pipeline et les jeux de données que vous avez créés dans ce didacticiel, consultez [Surveiller les jeux de données et le pipeline](data-factory-monitor-manage-pipelines.md).

Pour plus d’informations sur l’utilisation de l’application Surveiller et gérer afin de surveiller vos pipelines de données, consultez [Surveillance et gestion des pipelines d’Azure Data Factory à l’aide d’application Surveiller](data-factory-monitor-manage-app.md).

## <a name="data-factory-entities-in-the-template"></a>Entités Data Factory dans le modèle

### <a name="define-data-factory"></a>Définir une fabrique de données

Vous définissez une fabrique de données dans le modèle Resource Manager, comme indiqué dans l’exemple suivant :

```json
{
  "resources": [
    {
      "name": "[variables('dataFactoryName')]",
      "apiVersion": "2015-10-01",
      "type": "Microsoft.DataFactory/datafactories",
      "location": "West US"
    }
  ]
}
```

La propriété dataFactoryName est défini en tant que : 

```json
{
    "dataFactoryName": "[concat('AzureBlobToAzureSQLDatabaseDF', uniqueString(resourceGroup().id))]"
}
```

Il s’agit d’une chaîne unique basée sur l’ID du groupe de ressources.

### <a name="defining-data-factory-entities"></a>Définition des entités Data Factory

Les entités Data Factory suivantes sont définies dans le modèle JSON : 

1. [Service lié Azure Storage](#azure-storage-linked-service)
2. [Service lié SQL Azure](#azure-sql-database-linked-service)
3. [Jeu de données d’objet blob Azure](#azure-blob-dataset)
4. [Jeu de données SQL Azure](#azure-sql-dataset)
5. [Pipeline de données avec une activité de copie](#data-pipeline)

#### <a name="azure-storage-linked-service"></a>Service lié Stockage Azure

AzureStorageLinkedService relie votre compte de stockage Azure à la fabrique de données. Vous avez créé un conteneur et chargé des données dans ce compte de stockage en remplissant les [conditions préalables](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md). Vous spécifiez le nom et la clé de votre compte Stockage Azure dans cette section. Consultez [Service lié Stockage Azure](data-factory-azure-blob-connector.md#azure-storage-linked-service) pour en savoir plus sur les propriétés JSON utilisées pour définir un service lié Stockage Azure. 

```json
{
    "type": "linkedservices",
    "name": "[variables('azureStorageLinkedServiceName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
        "type": "AzureStorage",
        "description": "Azure Storage linked service",
        "typeProperties": {
            "connectionString": "[concat('DefaultEndpointsProtocol=https;AccountName=',parameters('storageAccountName'),';AccountKey=',parameters('storageAccountKey'))]"
        }
    }
}
```

La propriété connectionString utilise les paramètres storageAccountName et storageAccountKey. Les valeurs de ces paramètres sont transmises à l’aide d’un fichier de configuration. La définition utilise également les variables azureStorageLinkedService et dataFactoryName définies dans le modèle. 

#### <a name="azure-sql-database-linked-service"></a>Service lié Azure SQL Database

AzureSqlLinkedService lie votre base de données dans Azure SQL Database à la fabrique de données. Les données copiées à partir du stockage Blob sont stockées dans cette base de données. Vous avez créé une table emp dans cette base de données dans le cadre des [conditions préalables](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md). Vous spécifiez le nom du serveur SQL logique, le nom de la base de données, le nom d’utilisateur et le mot de passe de l’utilisateur dans cette section. Consultez [Service lié SQL Azure](data-factory-azure-sql-connector.md#linked-service-properties) pour en savoir plus sur les propriétés JSON utilisées pour définir un service lié SQL Azure.  

```json
{
    "type": "linkedservices",
    "name": "[variables('azureSqlLinkedServiceName')]",
    "dependsOn": [
      "[variables('dataFactoryName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
      "type": "AzureSqlDatabase",
      "description": "Azure SQL linked service",
      "typeProperties": {
        "connectionString": "[concat('Server=tcp:',parameters('sqlServerName'),'.database.windows.net,1433;Database=', parameters('databaseName'), ';User ID=',parameters('sqlServerUserName'),';Password=',parameters('sqlServerPassword'),';Trusted_Connection=False;Encrypt=True;Connection Timeout=30')]"
      }
    }
}
```

La propriété connectionString utilise les paramètres sqlServerName, databaseName, sqlServerUserName et sqlServerPassword, dont les valeurs sont transmises à l’aide d’un fichier de configuration. La définition utilise également les variables suivantes du modèle : azureSqlLinkedServiceName, dataFactoryName.

#### <a name="azure-blob-dataset"></a>Jeu de données d’objet blob Azure
Le service lié Stockage Azure spécifie la chaîne de connexion que le service Data Factory utilise au moment de l’exécution pour se connecter à votre compte de stockage Azure. Dans la définition du jeu de données d’objets blob Azure, vous spécifiez les noms du conteneur d’objets blob, du dossier et du fichier contenant les données d’entrée. Consultez [Propriétés du jeu de données d’objet blob Azure](data-factory-azure-blob-connector.md#dataset-properties) pour en savoir plus sur les propriétés JSON permettant de définir un jeu de données d’objets blob Azure. 

```json
{
    "type": "datasets",
    "name": "[variables('blobInputDatasetName')]",
    "dependsOn": [
      "[variables('dataFactoryName')]",
      "[variables('azureStorageLinkedServiceName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
        "type": "AzureBlob",
          "linkedServiceName": "[variables('azureStorageLinkedServiceName')]",
        "structure": [
        {
              "name": "Column0",
              "type": "String"
        },
        {
              "name": "Column1",
              "type": "String"
        }
          ],
          "typeProperties": {
            "folderPath": "[concat(parameters('sourceBlobContainer'), '/')]",
            "fileName": "[parameters('sourceBlobName')]",
            "format": {
                  "type": "TextFormat",
                  "columnDelimiter": ","
            }
          },
          "availability": {
            "frequency": "Hour",
            "interval": 1
          },
          "external": true
    }
}
```

#### <a name="azure-sql-dataset"></a>Jeu de données SQL Azure
Vous spécifiez le nom de la table dans Azure SQL Database contenant les données copiées à partir du stockage Blob Azure. Consultez [Propriétés du jeu de données SQL Azure](data-factory-azure-sql-connector.md#dataset-properties) pour en savoir plus sur les propriétés JSON permettant de définir un jeu de données SQL Azure. 

```json
{
    "type": "datasets",
    "name": "[variables('sqlOutputDatasetName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]",
          "[variables('azureSqlLinkedServiceName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
          "type": "AzureSqlTable",
          "linkedServiceName": "[variables('azureSqlLinkedServiceName')]",
          "structure": [
        {
              "name": "FirstName",
              "type": "String"
        },
        {
              "name": "LastName",
              "type": "String"
        }
          ],
          "typeProperties": {
            "tableName": "[parameters('targetSQLTable')]"
          },
          "availability": {
            "frequency": "Hour",
            "interval": 1
          }
    }
}
```

#### <a name="data-pipeline"></a>Pipeline de données
Vous définissez un pipeline qui copie les données du jeu de données d’objets blob Azure vers le jeu de données SQL Azure. Consultez [Pipeline JSON](data-factory-create-pipelines.md#pipeline-json) pour obtenir des descriptions des éléments JSON permettant de définir un pipeline dans cet exemple. 

```json
{
    "type": "datapipelines",
    "name": "[variables('pipelineName')]",
    "dependsOn": [
        "[variables('dataFactoryName')]",
          "[variables('azureStorageLinkedServiceName')]",
          "[variables('azureSqlLinkedServiceName')]",
          "[variables('blobInputDatasetName')]",
          "[variables('sqlOutputDatasetName')]"
    ],
    "apiVersion": "2015-10-01",
    "properties": {
          "activities": [
        {
              "name": "CopyFromAzureBlobToAzureSQL",
              "description": "Copy data frm Azure blob to Azure SQL",
              "type": "Copy",
              "inputs": [
            {
                  "name": "[variables('blobInputDatasetName')]"
            }
              ],
              "outputs": [
            {
                  "name": "[variables('sqlOutputDatasetName')]"
            }
              ],
              "typeProperties": {
                "source": {
                      "type": "BlobSource"
                },
                "sink": {
                      "type": "SqlSink",
                      "sqlWriterCleanupScript": "$$Text.Format('DELETE FROM {0}', 'emp')"
                },
                "translator": {
                      "type": "TabularTranslator",
                      "columnMappings": "Column0:FirstName,Column1:LastName"
                }
              },
              "Policy": {
                "concurrency": 1,
                "executionPriorityOrder": "NewestFirst",
                "retry": 3,
                "timeout": "01:00:00"
              }
        }
          ],
          "start": "2017-05-11T00:00:00Z",
          "end": "2017-05-12T00:00:00Z"
    }
}
```

## <a name="reuse-the-template"></a>Réutiliser le modèle
Dans ce didacticiel, vous avez créé un modèle pour définir des entités Data Factory et un modèle pour transmettre les valeurs des paramètres. Le pipeline copie les données d’un compte Stockage Azure vers le service Azure SQL Database spécifié par le biais de paramètres. Pour utiliser le même modèle afin de déployer des entités Data Factory dans des environnements différents, vous créez un fichier de paramètres pour chaque environnement et l’utiliser lors du déploiement de cet environnement.     

Exemple :  

```PowerShell
New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFCopyTutorialARM.json -TemplateParameterFile ADFCopyTutorialARM-Parameters-Dev.json
```
```PowerShell
New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFCopyTutorialARM.json -TemplateParameterFile ADFCopyTutorialARM-Parameters-Test.json
```
```PowerShell
New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFCopyTutorialARM.json -TemplateParameterFile ADFCopyTutorialARM-Parameters-Production.json
```

Notez que la première commande utilise le fichier de paramètres pour l’environnement de développement, la deuxième pour l’environnement de test et la troisième pour l’environnement de production.  

Vous pouvez également réutiliser le modèle pour effectuer des tâches répétitives. Par exemple, vous devez créer plusieurs fabriques de données avec un ou plusieurs pipelines qui implémentent la même logique, mais chaque fabrique de données utilise des comptes Stockage et SQL Database différents. Dans ce scénario, vous utilisez le même modèle dans le même environnement (développement, test ou production) avec différents fichiers de paramètres pour créer des fabriques de données.   

## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez utilisé le stockage Blob Azure comme magasin de données source et Azure SQL Database comme banque de données de destination dans une opération de copie. Le tableau ci-dessous contient la liste des magasins de données pris en charge en tant que sources et destinations par l’activité de copie : 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

Pour découvrir comment copier des données vers/depuis un magasin de données, cliquez sur le lien du magasin de données dans le tableau.
