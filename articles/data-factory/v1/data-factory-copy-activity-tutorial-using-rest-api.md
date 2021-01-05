---
title: 'Tutoriel : Utiliser l’API REST pour créer un pipeline Azure Data Factory '
description: Dans ce tutoriel, vous utilisez l’API REST pour créer un pipeline Azure Data Factory avec une activité de copie afin de copier des données d’un stockage Blob Azure dans Azure SQL Database.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: ''
editor: ''
ms.assetid: 1704cdf8-30ad-49bc-a71c-4057e26e7350
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 91a92f9dd0eaf55b8ba35f38102ee30b8cda4bfa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87053809"
---
# <a name="tutorial-use-rest-api-to-create-an-azure-data-factory-pipeline-to-copy-data"></a>Tutoriel : utiliser l’API REST pour créer un pipeline Azure Data Factory afin de copier des données 
> [!div class="op_single_selector"]
> * [Vue d’ensemble et composants requis](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Assistant de copie](data-factory-copy-data-wizard-tutorial.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Modèle Azure Resource Manager](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [API .NET](data-factory-copy-activity-tutorial-using-dotnet-api.md)
> 
> 

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez le [tutoriel sur l’activité de copie](../quickstart-create-data-factory-rest-api.md). 

Dans cet article, vous allez apprendre à utiliser l’API REST pour créer une fabrique de données avec un pipeline qui copie les données d’un stockage Blob Azure dans Azure SQL Database. Si vous débutez avec Azure Data Factory, lisez l’article [Présentation d’Azure Data Factory](data-factory-introduction.md) avant de suivre ce didacticiel.   

Dans ce tutoriel, vous créez un pipeline contenant une seule activité : une activité de copie. L’activité de copie copie les données d’un magasin de données pris en charge vers un magasin de données de récepteur pris en charge. Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs, consultez [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Elle est mise en œuvre par un service disponible dans le monde entier, capable de copier des données entre différents magasins de données de façon sécurisée, fiable et évolutive. Pour plus d’informations sur l’activité de copie, consultez [Activités de déplacement des données](data-factory-data-movement-activities.md).

Un pipeline peut contenir plusieurs activités. En outre, vous pouvez chaîner deux activités (une après l’autre) en configurant le jeu de données de sortie d’une activité en tant que jeu de données d’entrée de l’autre activité. Pour plus d’informations, consultez [Plusieurs activités dans un pipeline](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).

> [!NOTE]
> Cet article ne couvre pas toute l’API REST Data Factory. Consultez les [informations de référence sur l’API REST Data Factory](/rest/api/datafactory/) pour obtenir une documentation complète sur les applets de commande Data Factory.
>  
> Dans ce didacticiel, le pipeline de données copie les données d’un magasin de données source vers un magasin de données de destination. Pour suivre un tutoriel sur la transformation des données à l’aide d’Azure Data Factory, consultez [Tutoriel : Créer un pipeline pour transformer des données à l’aide d’un cluster Hadoop](data-factory-build-your-first-pipeline.md).

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Lisez l’article [Vue d’ensemble du didacticiel](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) et effectuez les **étapes préalables requises** .
* Installez [Curl](https://curl.haxx.se/dlwiz/) sur votre ordinateur. L’outil Curl et les commandes REST vous permettent de créer une fabrique de données. 
* Suivez les instructions de [cet article](../../active-directory/develop/howto-create-service-principal-portal.md) pour effectuer les opérations suivantes : 
  1. Créez une application Web nommée **ADFCopyTutotiralApp** dans Azure Active Directory.
  2. Obtenez un **ID client** et une **clé secrète**. 
  3. Obtenez l’ **ID de locataire**. 
  4. Attribuez l’application **ADFCopyTutorialApp** au rôle **Collaborateur de Data Factory**.  
* Installez [Azure PowerShell](/powershell/azure/).  
* Lancez **PowerShell** et procédez comme suit. Conservez Azure PowerShell ouvert jusqu’à la fin de ce tutoriel. Si vous fermez puis rouvrez Azure PowerShell, vous devez réexécuter ces commandes.
  
  1. Exécutez la commande suivante, puis saisissez le nom d’utilisateur et le mot de passe que vous avez utilisés pour la connexion au portail Azure :
    
     ```PowerShell 
     Connect-AzAccount
     ```   
  2. Exécutez la commande suivante pour afficher tous les abonnements de ce compte :

     ```PowerShell     
     Get-AzSubscription
     ``` 
  3. Exécutez la commande suivante pour sélectionner l’abonnement que vous souhaitez utiliser. Remplacez **&lt;NameOfAzureSubscription**&gt; par le nom de votre abonnement Azure. 
     
     ```PowerShell
     Get-AzSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzContext
     ```
  4. Créez un groupe de ressources Azure nommé **ADFTutorialResourceGroup** en exécutant la commande suivante dans PowerShell :  

     ```PowerShell     
      New-AzResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
     ```
     
      Si le groupe de ressources existe, indiquez s’il faut le mettre à jour (Y) ou le conserver tel quel (N). 
     
      Certaines étapes de ce didacticiel supposent que vous utilisez le groupe de ressources nommé ADFTutorialResourceGroup. Si vous utilisez un autre groupe de ressources, vous devez remplacer ADFTutorialResourceGroup par le nom de votre groupe de ressources dans ce didacticiel.

## <a name="create-json-definitions"></a>Créer des définitions JSON
Créez les fichiers JSON suivants dans le dossier où se trouve le fichier curl.exe. 

### <a name="datafactoryjson"></a>datafactory.json
> [!IMPORTANT]
> Le nom doit être globalement unique : vous pouvez ajouter un préfixe/suffixe à ADFCopyTutorialDF pour rendre ce nom unique. 
> 
> 

```JSON
{  
    "name": "ADFCopyTutorialDF",  
    "location": "WestUS"
}  
```

### <a name="azurestoragelinkedservicejson"></a>azurestoragelinkedservice.json
> [!IMPORTANT]
> Remplacez **accountname** et **accountkey** par le nom et la clé de votre compte de stockage Azure. Pour découvrir comment obtenir votre clé d’accès au stockage, consultez [Gérer les clés d’accès au compte de stockage](../../storage/common/storage-account-keys-manage.md).

```JSON
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```

Pour plus d’informations sur les propriétés JSON, consultez [Service lié Azure Storage](data-factory-azure-blob-connector.md#azure-storage-linked-service).

### <a name="azuresqllinkedservicejson"></a>azuresqllinkedservice.json
> [!IMPORTANT]
> Remplacez **servername**, **databasename**, **username** et **password** par le nom de votre serveur, le nom de la base de données SQL, le compte d’utilisateur et le mot de passe de ce compte.  
> 
>

```JSON
{
    "name": "AzureSqlLinkedService",
    "properties": {
        "type": "AzureSqlDatabase",
        "description": "",
        "typeProperties": {
            "connectionString": "Data Source=tcp:<servername>.database.windows.net,1433;Initial Catalog=<databasename>;User ID=<username>;Password=<password>;Integrated Security=False;Encrypt=True;Connect Timeout=30"
        }
    }
}
```

Pour plus d’informations sur les propriétés JSON, consultez [Service lié SQL Azure](data-factory-azure-sql-connector.md#linked-service-properties).

### <a name="inputdatasetjson"></a>inputdataset.json

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
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
    "type": "AzureBlob",
    "linkedServiceName": "AzureStorageLinkedService",
    "typeProperties": {
      "folderPath": "adftutorial/",
      "fileName": "emp.txt",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ","
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

Le tableau suivant décrit les propriétés JSON utilisées dans l'extrait de code :

| Propriété | Description |
|:--- |:--- |
| type | La propriété du type est définie sur **AzureBlob**, car les données se trouvent dans le stockage blob Azure. |
| linkedServiceName | Fait référence au service **AzureStorageLinkedService** que vous avez créé précédemment. |
| folderPath | Spécifie le **conteneur** d’objets blob et le **dossier** contenant les objets blob d’entrée. Dans ce didacticiel, adftutorial est le conteneur d’objets blob et folder est le dossier racine. | 
| fileName | Cette propriété est facultative. Si vous omettez cette propriété, tous les fichiers spécifiés dans le paramètre folderPath sont récupérés. Dans ce didacticiel, **emp.txt** est spécifié pour le paramètre fileName, si bien que seul ce fichier est récupéré pour le traitement. |
| format -> type |Le fichier d’entrée étant au format texte, nous utilisons **TextFormat**. |
| columnDelimiter | Les colonnes du fichier d’entrée sont délimitées par une **virgule (`,`)** . |
| frequency/interval | La fréquence est définie sur **Heure** et l’intervalle est **1**, ce qui signifie que les segments d’entrée sont disponibles **toutes les heures**. En d’autres termes, le service Data Factory recherche les données d’entrée toutes les heures dans le dossier racine du conteneur d’objets blob (**adftutorial**) que vous avez spécifié. Il recherche des données entre les heures de début et de fin du pipeline, pas avant ni après.  |
| external | Cette propriété a la valeur **true** si les données ne sont pas générées par ce pipeline. Dans ce didacticiel, les données d’entrée sont dans le fichier emp.txt, qui n’est pas généré par ce pipeline. Nous définissons donc cette propriété sur true. |

Pour plus d’informations sur ces propriétés JSON, voir [Connecteur de stockage Blob Azure](data-factory-azure-blob-connector.md#dataset-properties).

### <a name="outputdatasetjson"></a>outputdataset.json

```JSON
{
  "name": "AzureSqlOutput",
  "properties": {
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
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "emp"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
Le tableau suivant décrit les propriétés JSON utilisées dans l'extrait de code :

| Propriété | Description |
|:--- |:--- |
| type | La propriété du type est définie sur **AzureSqlTable** parce que les données sont copiées dans une table dans Azure SQL Database. |
| linkedServiceName | Fait référence au service **AzureSqlLinkedService** que vous avez créé précédemment. |
| tableName | Spécifie la **table** dans laquelle les données sont copiées. | 
| frequency/interval | La fréquence est définie sur **Heure** et l’intervalle sur **1**, ce qui signifie que les tranches de sortie sont produites **toutes les heures** entre les heures de début et de fin du pipeline, pas avant ni après.  |

La table emp de la base de données contient trois colonnes : **ID**, **FirstName** et **LastName**. ID étant une colonne d’identité, il vous suffit de spécifier **FirstName** et **LastName**.

Pour plus d’informations sur ces propriétés JSON, consultez l’article [Connecteur SQL Azure](data-factory-azure-sql-connector.md#dataset-properties).

### <a name="pipelinejson"></a>pipeline.json

```JSON
{
  "name": "ADFTutorialPipeline",
  "properties": {
    "description": "Copy data from a blob to Azure SQL table",
    "activities": [
      {
        "name": "CopyFromBlobToSQL",
        "description": "Push Regional Effectiveness Campaign data to Azure SQL Database",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureSqlOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink",
            "writeBatchSize": 10000,
            "writeBatchTimeout": "60:00:00"
          }
        },
        "Policy": {
          "concurrency": 1,
          "executionPriorityOrder": "NewestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ],
    "start": "2017-05-11T00:00:00Z",
    "end": "2017-05-12T00:00:00Z"
  }
}
```

Notez les points suivants :

- Dans la section des activités, il existe une seule activité dont le **type** a la valeur **Copy**. Pour plus d’informations sur l’activité de copie, consultez [Activités de déplacement des données](data-factory-data-movement-activities.md). Dans les solutions Data Factory, vous pouvez également utiliser [Activités de transformation des données](data-factory-data-transformation-activities.md).
- L’entrée de l’activité a la valeur **AzureBlobInput** et la sortie de l’activité a la valeur **AzureSqlOutput**. 
- Dans la section **typeProperties**, **BlobSource** est spécifié en tant que type de source et **SqlSink**, en tant que type de récepteur. Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs pour l’activité de copie, consultez [Magasins de données pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats). Pour apprendre à utiliser un magasin de données pris en charge spécifique en tant que source/récepteur, cliquez sur le lien dans le tableau.  
 
Remplacez la valeur de la propriété **start** par le jour actuel et la valeur **end**, par le jour suivant. Si vous le souhaitez, spécifiez uniquement la date et ignorez l'heure. Par exemple, « 2017-02-03 », qui équivaut à « 2017-02-03T00:00:00Z ».
 
Les dates/heures de début et de fin doivent toutes deux être au [format ISO](https://en.wikipedia.org/wiki/ISO_8601). Par exemple : 2016-10-14T16:32:41Z. L’heure de fin ( **end** ) est facultative, mais nous allons l’utiliser dans ce didacticiel. 
 
Si vous ne spécifiez aucune valeur pour la propriété **end**, cette dernière est calculée comme suit : « **start + 48 heures** ». Pour exécuter le pipeline indéfiniment, spécifiez **9999-09-09** comme valeur pour la propriété **end**.
 
Dans l’exemple ci-dessus, il existe 24 tranches de données, car une tranche de données est générée toutes les heures.

Pour obtenir une description des propriétés JSON dans une définition de pipeline, consultez l’article [Créer des pipelines](data-factory-create-pipelines.md). Pour obtenir une description des propriétés JSON dans une définition d’activité de copie, consultez [Activités de déplacement des données](data-factory-data-movement-activities.md). Pour obtenir une description des propriétés JSON prises en charge par BlobSource, consultez l’article [Connecteur de stockage Blob Azure](data-factory-azure-blob-connector.md). Pour obtenir une description des propriétés JSON prises en charge par SqlSink, consultez l’article [Connecteur Azure SQL Database](data-factory-azure-sql-connector.md).

## <a name="set-global-variables"></a>Définir des variables globales
Dans Azure PowerShell, exécutez les commandes suivantes après avoir remplacé les valeurs par les vôtres :

> [!IMPORTANT]
> Consultez la section [Configuration requise](#prerequisites) pour savoir comment obtenir un ID client, une clé secrète client, un ID de locataire et un ID d’abonnement.   
> 
> 

```JSON
$client_id = "<client ID of application in AAD>"
$client_secret = "<client key of application in AAD>"
$tenant = "<Azure tenant ID>";
$subscription_id="<Azure subscription ID>";

$rg = "ADFTutorialResourceGroup"
```

Exécutez la commande suivante après la mise à jour le nom de la fabrique de données que vous utilisez : 

```
$adf = "ADFCopyTutorialDF"
```

## <a name="authenticate-with-aad"></a>Authentifier avec AAD
Exécutez la commande suivante pour vous authentifier auprès d’Azure Active Directory (AAD) : 

```PowerShell
$cmd = { .\curl.exe -X POST https://login.microsoftonline.com/$tenant/oauth2/token  -F grant_type=client_credentials  -F resource=https://management.core.windows.net/ -F client_id=$client_id -F client_secret=$client_secret };
$responseToken = Invoke-Command -scriptblock $cmd;
$accessToken = (ConvertFrom-Json $responseToken).access_token;

(ConvertFrom-Json $responseToken) 
```

## <a name="create-data-factory"></a>Créer une fabrique de données
Dans cette étape, vous créez une fabrique de données Azure nommée **ADFCopyTutorialDF**. Une fabrique de données peut avoir un ou plusieurs pipelines. Un pipeline peut contenir une ou plusieurs activités. Par exemple, une activité de copie pour copier les données d’une source vers un magasin de données de destination. Une activité HDInsight Hive pour exécuter le script Hive afin de transformer les données d’entrée en données de sortie produit. Exécutez les commandes suivantes pour créer la fabrique de données : 

1. Attribuez la commande à une variable nommée **cmd**. 
   
    > [!IMPORTANT]
    > Vérifiez que le nom de la fabrique de données que vous spécifiez ici (ADFCopyTutorialDF) correspond au nom spécifié dans **datafactory.json**. 
   
    ```PowerShell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@datafactory.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/ADFCopyTutorialDF0411?api-version=2015-10-01};
    ```
2. Exécutez la commande à l’aide de **Invoke-Command**.
   
    ```PowerShell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Affichez les résultats. Si la fabrique de données a été créée avec succès, le code JSON de la fabrique de données apparaît dans les **résultats**. Dans le cas contraire, un message d’erreur apparaît.  
   
    ```
    Write-Host $results
    ```

Notez les points suivants :

* Le nom de la fabrique de données Azure doit être un nom global unique. Si vous voyez l’erreur suivante dans les résultats : **Le nom de fabrique de données « ADFCopyTutorialDF » n’est pas disponible**, effectuez les étapes suivantes :  
  
  1. Modifiez le nom (par exemple, votrenomADFCopyTutorialDF) dans le fichier **datafactory.json** .
  2. Dans la première commande où une valeur est attribuée à la variable **$cmd** , remplacez ADFCopyTutorialDF par le nouveau nom, puis exécutez la commande. 
  3. Exécutez les deux commandes suivantes pour appeler l’API REST afin de créer la fabrique de données et d’imprimer les résultats de l’opération. 
     
     Consultez la rubrique [Data Factory - Règles d’affectation des noms](data-factory-naming-rules.md) pour savoir comment nommer les artefacts Data Factory.
* Pour créer des instances de fabrique de données, vous devez avoir le statut d’administrateur/collaborateur de l’abonnement Azure
* Le nom de la fabrique de données pourra être enregistré en tant que nom DNS et devenir ainsi visible publiquement.
* Si vous recevez le message d’erreur : « **L’abonnement n’est pas inscrit pour utiliser l’espace de noms Microsoft.DataFactory** », effectuez l’une des opérations suivantes et essayez de relancer la publication : 
  
  * Dans Azure PowerShell, exécutez la commande suivante pour enregistrer le fournisseur Data Factory : 

    ```PowerShell    
    Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```
    Vous pouvez exécuter la commande suivante pour confirmer que le fournisseur Data Factory est bien enregistré. 
    
    ```PowerShell
    Get-AzResourceProvider
    ```
  * Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide de l’abonnement Azure et accédez à un panneau Data Factory (ou) créez une fabrique de données dans le portail Azure. Cette action enregistre automatiquement le fournisseur.

Avant de créer un pipeline, vous devez d’abord créer quelques entités de la fabrique de données. Vous créez d’abord les services liés pour lier les magasins de données source et de destination à votre magasin de données. Ensuite, définissez les jeux de données d’entrée et de sortie pour représenter les données des magasins liés. Enfin, créez le pipeline avec une activité qui utilise ces jeux de données.

## <a name="create-linked-services"></a>Créez des services liés
Vous allez créer des services liés dans une fabrique de données pour lier vos magasins de données et vos services de calcul à la fabrique de données. Dans ce didacticiel, vous n’allez pas utiliser n’importe quel service de calcul comme Azure HDInsight ou Azure Data Lake Analytics. Vous utilisez deux magasins de données de type Stockage Azure (source) et Azure SQL Database (destination). Vous créez ainsi deux services liés nommés AzureStorageLinkedService et AzureSqlLinkedService de types : AzureStorage et AzureSqlDatabase.  

AzureStorageLinkedService relie votre compte de stockage Azure à la fabrique de données. Ce compte de stockage est celui dans lequel vous avez créé un conteneur et chargé les données en remplissant les [conditions préalables](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).   

AzureSqlLinkedService lie Azure SQL Database à la fabrique de données. Les données copiées à partir du stockage Blob sont stockées dans cette base de données. Vous avez créé une table emp dans cette base de données dans le cadre des [conditions préalables](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).  

### <a name="create-azure-storage-linked-service"></a>Créer le service lié Stockage Azure
Dans cette étape, vous liez votre compte de stockage Azure à votre fabrique de données. Vous spécifiez le nom et la clé de votre compte Stockage Azure dans cette section. Consultez [Service lié Stockage Azure](data-factory-azure-blob-connector.md#azure-storage-linked-service) pour en savoir plus sur les propriétés JSON utilisées pour définir un service lié Stockage Azure.  

1. Attribuez la commande à une variable nommée **cmd**. 

    ```PowerShell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@azurestoragelinkedservice.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/AzureStorageLinkedService?api-version=2015-10-01};
    ```
2. Exécutez la commande à l’aide de **Invoke-Command**.

    ```PowerShell   
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Affichez les résultats. Si le service lié a été créé avec succès, le code JSON du service lié apparaît dans les **résultats**. Dans le cas contraire, un message d’erreur apparaît.

    ```PowerShell   
    Write-Host $results
    ```

### <a name="create-azure-sql-linked-service"></a>Créer un service lié Azure SQL
Dans cette étape, vous liez Azure SQL Database à votre fabrique de données. Vous spécifiez le nom du serveur SQL logique, le nom de la base de données, le nom d’utilisateur et le mot de passe de l’utilisateur dans cette section. Consultez [Service lié SQL Azure](data-factory-azure-sql-connector.md#linked-service-properties) pour en savoir plus sur les propriétés JSON utilisées pour définir un service lié SQL Azure.

1. Attribuez la commande à une variable nommée **cmd**. 
   
    ```PowerShell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@azuresqllinkedservice.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/AzureSqlLinkedService?api-version=2015-10-01};
    ```
2. Exécutez la commande à l’aide de **Invoke-Command**.
   
    ```PowerShell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Affichez les résultats. Si le service lié a été créé avec succès, le code JSON du service lié apparaît dans les **résultats**. Dans le cas contraire, un message d’erreur apparaît.
   
    ```PowerShell
    Write-Host $results
    ```

## <a name="create-datasets"></a>Créez les jeux de données
Dans l’étape précédente, vous avez créé des services pour lier votre compte de stockage Azure et Azure SQL Database à votre fabrique de données. Dans cette étape, vous définissez deux jeux de données nommés AzureBlobInput et AzureSqlOutput qui représentent les données d’entrée/sortie stockées dans les magasins de données référencés par AzureStorageLinkedService et AzureSqlLinkedService, respectivement.

Le service lié Stockage Azure spécifie la chaîne de connexion que le service Data Factory utilise au moment de l’exécution pour se connecter à votre compte de stockage Azure. Le jeu de données blob d’entrée (AzureBlobInput) spécifie quant à lui le conteneur et le dossier qui contient les données d’entrée.  

De même, le service lié Azure SQL Database spécifie la chaîne de connexion que le service Data Factory utilise au moment de l’exécution pour se connecter à Azure SQL Database. Et le jeu de données de la table SQL de sortie (OututDataset) spécifie la table de la base de données dans laquelle les données du stockage Blob sont copiées. 

### <a name="create-input-dataset"></a>Créer le jeu de données d’entrée
Dans cette étape, vous créez un jeu de données nommé AzureBlobInput qui pointe vers un fichier blob (emp.txt) dans le dossier racine d’un conteneur d’objets blob (adftutorial) du stockage Azure représenté par le service lié AzureStorageLinkedService. Si vous ne spécifiez pas de valeur pour fileName (ou si vous ignorez ce paramètre), les données de tous les objets blob du dossier d’entrée sont copiées dans la destination. Dans ce didacticiel, vous spécifiez une valeur pour fileName. 

1. Attribuez la commande à une variable nommée **cmd**. 

    ```PowerSHell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@inputdataset.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/AzureBlobInput?api-version=2015-10-01};
    ```
2. Exécutez la commande à l’aide de **Invoke-Command**.
   
    ```PowerShell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Affichez les résultats. Si le jeu de données a été créé avec succès, le code JSON du jeu de données apparaît dans les **résultats**. Dans le cas contraire, un message d’erreur apparaît.
   
    ```PowerShell
    Write-Host $results
    ```

### <a name="create-output-dataset"></a>Créer un jeu de données de sortie
Le service lié Azure SQL Database spécifie la chaîne de connexion que le service Data Factory utilise au moment de l’exécution pour se connecter à Azure SQL Database. Le jeu de données de la table SQL de sortie (OututDataset) que vous créez à cette étape spécifie la table de la base de données dans laquelle les données du stockage Blob sont copiées.

1. Attribuez la commande à une variable nommée **cmd**.

    ```PowerShell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@outputdataset.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/AzureSqlOutput?api-version=2015-10-01};
    ```
2. Exécutez la commande à l’aide de **Invoke-Command**.
    
    ```PowerShell   
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Affichez les résultats. Si le jeu de données a été créé avec succès, le code JSON du jeu de données apparaît dans les **résultats**. Dans le cas contraire, un message d’erreur apparaît.
   
    ```PowerShell
    Write-Host $results
    ``` 

## <a name="create-pipeline"></a>Création d’un pipeline
Dans cette étape, vous créez un pipeline avec une **activité de copie** qui utilise **AzureBlobInput** en entrée et **AzureSqlOutput** en sortie.

Le jeu de données de sortie pilote actuellement la planification. Dans ce didacticiel, le jeu de données de sortie est configuré pour produire une tranche par heure. Les heures de début et de fin du pipeline sont distantes d’une journée, soit 24 heures. Par conséquent, 24 tranches de jeu de données de sortie sont générées par le pipeline. 

1. Attribuez la commande à une variable nommée **cmd**.

    ```PowerShell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@pipeline.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datapipelines/MyFirstPipeline?api-version=2015-10-01};
    ```
2. Exécutez la commande à l’aide de **Invoke-Command**.

    ```PowerShell   
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Affichez les résultats. Si le jeu de données a été créé avec succès, le code JSON du jeu de données apparaît dans les **résultats**. Dans le cas contraire, un message d’erreur apparaît.  

    ```PowerShell   
    Write-Host $results
    ```

**Félicitations !** Vous avez créé une fabrique de données Azure, avec un pipeline qui copie les données depuis Stockage Blob Azure vers Azure SQL Database.

## <a name="monitor-pipeline"></a>Surveillance d’un pipeline
Dans cette étape, vous utilisez l’API REST Data Factory pour surveiller les tranches produites par le pipeline.

```PowerShell
$ds ="AzureSqlOutput"
```

> [!IMPORTANT] 
> Assurez-vous que les heures de début et de fin spécifiées dans la commande suivante correspondent aux heures de début et de fin du pipeline. 

```PowerShell
$cmd = {.\curl.exe -X GET -H "Authorization: Bearer $accessToken" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/$ds/slices?start=2017-05-11T00%3a00%3a00.0000000Z"&"end=2017-05-12T00%3a00%3a00.0000000Z"&"api-version=2015-10-01};
```

```PowerShell
$results2 = Invoke-Command -scriptblock $cmd;
```

```PowerShell
IF ((ConvertFrom-Json $results2).value -ne $NULL) {
    ConvertFrom-Json $results2 | Select-Object -Expand value | Format-Table
} else {
        (convertFrom-Json $results2).RemoteException
}
```

Exécutez Invoke-Command et la commande suivante jusqu’à ce qu’une tranche soit à l’état **Prêt** ou **Échec**. Lorsque la tranche est prête, vérifiez les données de sortie de la table **emp** dans Azure SQL Database. 

Pour chaque tranche, deux lignes de données tirées du fichier source sont copiées dans la table emp dans Azure SQL Database. Par conséquent, 24 nouveaux enregistrements apparaissent dans la table emp lorsque toutes les tranches sont traitées avec succès (état Prêt). 

## <a name="summary"></a>Résumé
Dans ce tutoriel, vous avez utilisé une API REST pour créer une fabrique de données Azure afin de copier des données d’un objet blob Azure vers Azure SQL Database. Voici les étapes de premier niveau que vous avez effectuées dans ce didacticiel :  

1. Création d’une **fabrique de données**Azure.
2. Création de **services liés**:
   1. Un service lié Azure Storage pour lier votre compte Stockage Azure contenant des données d’entrée.     
   2. Un service lié Azure SQL pour lier votre base de données contenant les données de sortie. 
3. Création des **jeux de données**qui décrivent les données d’entrée et de sortie des pipelines.
4. Création d’un **pipeline** avec une activité de copie avec BlobSource en tant que source et SqlSink en tant que récepteur. 

## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez utilisé le stockage Blob Azure comme magasin de données source et Azure SQL Database comme banque de données de destination dans une opération de copie. Le tableau ci-dessous contient la liste des magasins de données pris en charge en tant que sources et destinations par l’activité de copie : 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

Pour découvrir comment copier des données vers/depuis un magasin de données, cliquez sur le lien du magasin de données dans le tableau.
