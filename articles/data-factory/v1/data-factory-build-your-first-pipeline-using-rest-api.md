---
title: Générer votre première fabrique de données Data Factory (REST)
description: Dans ce didacticiel, vous allez créer un exemple de pipeline Azure Data Factory à l’aide de l’API REST Data Factory.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 11/01/2017
ms.openlocfilehash: 835f72df5c0c693c90b0cf7c45f7805b767d2bcb
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96496362"
---
# <a name="tutorial-build-your-first-azure-data-factory-using-data-factory-rest-api"></a>Didacticiel : Créer votre première fabrique de données Azure en utilisant l’API REST Data Factory
> [!div class="op_single_selector"]
> * [Vue d’ensemble et composants requis](data-factory-build-your-first-pipeline.md)
> * [Visual Studio](data-factory-build-your-first-pipeline-using-vs.md)
> * [PowerShell](data-factory-build-your-first-pipeline-using-powershell.md)
> * [Modèle Resource Manager](data-factory-build-your-first-pipeline-using-arm.md)
> * [REST API](data-factory-build-your-first-pipeline-using-rest-api.md)
>
>


> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Démarrage rapide : Créer une fabrique de données à l’aide d’Azure Data Factory](../quickstart-create-data-factory-rest-api.md).

Dans cet article, vous utilisez l’API REST Data Factory pour créer votre première fabrique de données Azure. Pour suivre le didacticiel avec d’autres outils/Kits de développement logiciel (SDK), sélectionnez une des options dans la liste déroulante.

Le pipeline dans ce didacticiel a une activité : **Activité HDInsight Hive**. Cette activité exécute un script Hive sur un cluster HDInsight qui transforme des données d’entrée pour produire des données de sortie. Le pipeline est programmé pour s’exécuter une fois par mois entre les heures de début et de fin spécifiées.

> [!NOTE]
> Cet article ne couvre pas toutes les API REST. pour obtenir une documentation complète sur les API REST, consultez [Informations de référence sur l’API REST Data Factory](/rest/api/datafactory/).
> 
> Un pipeline peut contenir plusieurs activités. En outre, vous pouvez chaîner deux activités (une après l’autre) en configurant le jeu de données de sortie d’une activité en tant que jeu de données d’entrée de l’autre activité. Pour plus d’informations, consultez [Planification et exécution dans Data Factory](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline).


## <a name="prerequisites"></a>Conditions préalables requises

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Lisez l’article [Vue d’ensemble du didacticiel](data-factory-build-your-first-pipeline.md) et effectuez les **étapes préalables requises** .
* Installez [Curl](https://curl.haxx.se/dlwiz/) sur votre ordinateur. L’outil CURL et les commandes REST vous permettent de créer une fabrique de données.
* Suivez les instructions de [cet article](../../active-directory/develop/howto-create-service-principal-portal.md) pour effectuer les opérations suivantes :
  1. Créez une application Web nommée **ADFGetStartedApp** dans Azure Active Directory.
  2. Obtenez un **ID client** et une **clé secrète**.
  3. Obtenez l’ **ID de locataire**.
  4. Attribuez l’application **ADFGetStartedApp** au rôle **Collaborateurs de fabrique de données**.
* Installez [Azure PowerShell](/powershell/azure/).
* Démarrez **PowerShell** et exécutez la commande suivante. Conservez Azure PowerShell ouvert jusqu’à la fin de ce tutoriel. Si vous fermez puis rouvrez Azure PowerShell, vous devez réexécuter ces commandes.
  1. Exécutez **Connect-AzAccount**, puis saisissez le nom d’utilisateur et le mot de passe que vous avez utilisés pour la connexion au portail Azure.
  2. Exécutez **Get-AzSubscription** pour voir tous les abonnements de ce compte.
  3. Exécutez **Get-AzSubscription -SubscriptionName NameOfAzureSubscription | Set-AzContext** pour sélectionner l’abonnement que vous souhaitez utiliser. Remplacez **NameOfAzureSubscription** par le nom de votre abonnement Azure.
* Créez un groupe de ressources Azure nommé **ADFTutorialResourceGroup** en exécutant la commande suivante dans PowerShell :

    ```powershell
    New-AzResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```

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
    "name": "FirstDataFactoryREST",
    "location": "WestUS"
}
```

### <a name="azurestoragelinkedservicejson"></a>azurestoragelinkedservice.json
> [!IMPORTANT]
> Remplacez **accountname** et **accountkey** par le nom et la clé de votre compte de stockage Azure. Pour découvrir comment obtenir votre clé d’accès au stockage, consultez [Gérer les clés d’accès au compte de stockage](../../storage/common/storage-account-keys-manage.md).
>
>

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

### <a name="hdinsightondemandlinkedservicejson"></a>hdinsightondemandlinkedservice.json

```JSON
{
    "name": "HDInsightOnDemandLinkedService",
    "properties": {
        "type": "HDInsightOnDemand",
        "typeProperties": {
            "version": "3.5",
            "clusterSize": 1,
            "timeToLive": "00:05:00",
            "osType": "Linux",
            "linkedServiceName": "AzureStorageLinkedService"
        }
    }
}
```

Le tableau suivant décrit les propriétés JSON utilisées dans l'extrait de code :

| Propriété | Description |
|:--- |:--- |
| clusterSize |Taille du cluster HDInsight. |
| timeToLive |Spécifie la durée d’inactivité du cluster HDInsight avant sa suppression. |
| linkedServiceName |Spécifie le compte de stockage utilisé pour stocker les journaux d’activité générés par HDInsight |

Notez les points suivants :

* La fabrique de données crée pour vous un cluster HDInsight **Linux** avec le code JSON ci-dessus. Pour plus d’informations, voir [Service lié à la demande Azure HDInsight](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) .
* Vous pouvez utiliser votre **propre cluster HDInsight** au lieu d’utiliser un cluster HDInsight à la demande. Pour plus d’informations, voir [Service lié Azure HDInsight](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) .
* Le cluster HDInsight crée un **conteneur par défaut** dans le stockage d’objets blob que vous avez spécifié dans le JSON (**linkedServiceName**). HDInsight ne supprime pas ce conteneur lorsque le cluster est supprimé. Ce comportement est normal. Avec le service lié HDInsight disponible à la demande, un cluster HDInsight est créé dès qu’une tranche est traitée, à moins qu’il n’existe un cluster actif (**timeToLive**). Ce cluster est supprimé, une fois le traitement terminé.

    Comme un nombre croissant de tranches sont traitées, vous voyez un grand nombre de conteneurs dans votre stockage d’objets blob Azure. Si vous n’en avez pas besoin pour dépanner les travaux, il se peut que vous deviez les supprimer pour réduire les frais de stockage. Le nom de ces conteneurs suit un modèle : « **nomdevotrefabriquededonnéesadf**-**nomduservicelié**-horodatage ». Utilisez des outils tels que l’[Explorateur Stockage Microsoft Azure](https://storageexplorer.com/) pour supprimer des conteneurs dans votre stockage Blob Azure.

Pour plus d’informations, voir [Service lié à la demande Azure HDInsight](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) .

### <a name="inputdatasetjson"></a>inputdataset.json

```JSON
{
    "name": "AzureBlobInput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "fileName": "input.log",
            "folderPath": "adfgetstarted/inputdata",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Month",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```

Le code JSON définit un jeu de données nommé **AzureBlobInput**, qui représente les données d’entrée correspondant à une activité du pipeline. En outre, ce code spécifie que les données d’entrée figurent dans le conteneur d’objets blob **adfgetstarted** et dans le dossier **inputdata**.

Le tableau suivant décrit les propriétés JSON utilisées dans l'extrait de code :

| Propriété | Description |
|:--- |:--- |
| type |La propriété type est définie sur AzureBlob, car les données se trouvent dans le stockage d’objets blob Azure. |
| linkedServiceName |fait référence au service StorageLinkedService que vous avez créé précédemment. |
| fileName |Cette propriété est facultative. Si vous omettez cette propriété, tous les fichiers spécifiés dans le paramètre folderPath sont récupérés. Dans le cas présent, seul le fichier input.log est traité. |
| type |Les fichiers journaux sont au format texte : nous utilisons donc TextFormat. |
| columnDelimiter |Les colonnes des fichiers journaux sont délimitées par une virgule (,) |
| frequency/interval |La fréquence est définie sur Mois et l’intervalle est 1, ce qui signifie que les segments d’entrée sont disponibles mensuellement. |
| external |Cette propriété a la valeur true si les données d’entrée ne sont pas générées par le service Data Factory. |

### <a name="outputdatasetjson"></a>outputdataset.json

```JSON
{
    "name": "AzureBlobOutput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "adfgetstarted/partitioneddata",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Month",
            "interval": 1
        }
    }
}
```

Le code JSON définit un jeu de données nommé **AzureBlobOutput**, qui représente les données de sortie correspondant à une activité du pipeline. En outre, ce code spécifie que les résultats sont stockés dans le conteneur d’objets blob **adfgetstarted** et dans le dossier **partitioneddata**. La section **availability** spécifie que le jeu de données de sortie est produit tous les mois.

### <a name="pipelinejson"></a>pipeline.json
> [!IMPORTANT]
> Remplacez **storageaccountname** par le nom de votre compte Stockage Azure.
>
>

```JSON
{
    "name": "MyFirstPipeline",
    "properties": {
        "description": "My first Azure Data Factory pipeline",
        "activities": [{
            "type": "HDInsightHive",
            "typeProperties": {
                "scriptPath": "adfgetstarted/script/partitionweblogs.hql",
                "scriptLinkedService": "AzureStorageLinkedService",
                "defines": {
                    "inputtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/inputdata",
                    "partitionedtable": "wasb://adfgetstarted@<storageaccountname>t.blob.core.windows.net/partitioneddata"
                }
            },
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "policy": {
                "concurrency": 1,
                "retry": 3
            },
            "scheduler": {
                "frequency": "Month",
                "interval": 1
            },
            "name": "RunSampleHiveActivity",
            "linkedServiceName": "HDInsightOnDemandLinkedService"
        }],
        "start": "2017-07-10T00:00:00Z",
        "end": "2017-07-11T00:00:00Z",
        "isPaused": false
    }
}
```

Dans l’extrait de code JSON, vous créez un pipeline qui se compose d’une seule activité utilisant Hive pour traiter des données sur un cluster HDInsight.

Le fichier de script Hive, **partitionweblogs.hql**, est stocké dans le compte de stockage Azure (spécifié par le service scriptLinkedService, appelé **StorageLinkedService**) et dans un dossier **script** du conteneur **adfgetstarted**.

La section **defines** sert à spécifier les paramètres d’exécution transmis au script Hive comme des valeurs de configuration Hive (par exemple ${hiveconf:inputtable}, ${hiveconf:partitionedtable}).

Les propriétés **start** et **end** du pipeline spécifient la période active du pipeline.

Dans l’activité JSON, vous spécifiez que le script Hive s’exécute sur le calcul spécifié par le service **linkedServiceName** – **HDInsightOnDemandLinkedService**.

> [!NOTE]
> Consultez « Pipeline JSON » dans [Pipelines et activités dans Azure Data Factory](data-factory-create-pipelines.md) pour plus d’informations sur les propriétés JSON utilisées dans l’exemple précédent.
>
>

## <a name="set-global-variables"></a>Définir des variables globales
Dans Azure PowerShell, exécutez les commandes suivantes après avoir remplacé les valeurs par les vôtres :

> [!IMPORTANT]
> Consultez la section [Configuration requise](#prerequisites) pour savoir comment obtenir un ID client, une clé secrète client, un ID de locataire et un ID d’abonnement.
>
>

```powershell
$client_id = "<client ID of application in AAD>"
$client_secret = "<client key of application in AAD>"
$tenant = "<Azure tenant ID>";
$subscription_id="<Azure subscription ID>";

$rg = "ADFTutorialResourceGroup"
$adf = "FirstDataFactoryREST"
```


## <a name="authenticate-with-aad"></a>Authentifier avec AAD

```powershell
$cmd = { .\curl.exe -X POST https://login.microsoftonline.com/$tenant/oauth2/token  -F grant_type=client_credentials  -F resource=https://management.core.windows.net/ -F client_id=$client_id -F client_secret=$client_secret };
$responseToken = Invoke-Command -scriptblock $cmd;
$accessToken = (ConvertFrom-Json $responseToken).access_token;

(ConvertFrom-Json $responseToken)
```


## <a name="create-data-factory"></a>Créer une fabrique de données
Dans cette étape, vous créez une fabrique de données Azure Data Factory nommée **FirstDataFactoryREST**. Une fabrique de données peut avoir un ou plusieurs pipelines. Un pipeline peut contenir une ou plusieurs activités. Par exemple, une activité de copie censée copier des données d’un magasin de données source vers un magasin de données de destination, et une activité Hive HDInsight pour exécuter un script Hive pour transformer des données. Exécutez les commandes suivantes pour créer la fabrique de données :

1. Attribuez la commande à une variable nommée **cmd**.

    Vérifiez que le nom de la fabrique de données que vous spécifiez ici (ADFCopyTutorialDF) correspond au nom spécifié dans **datafactory.json**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@datafactory.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/FirstDataFactoryREST?api-version=2015-10-01};
    ```
2. Exécutez la commande à l’aide de **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Affichez les résultats. Si la fabrique de données a été créée avec succès, le code JSON de la fabrique de données apparaît dans les **résultats**. Dans le cas contraire, un message d’erreur apparaît.

    ```powershell
    Write-Host $results
    ```

Notez les points suivants :

* Le nom de la fabrique de données Azure doit être un nom global unique. Si le message d’erreur **Le nom de la fabrique de données « FirstDataFactoryREST » n’est pas disponible** s’affiche dans les résultats, procédez comme suit :
  1. Modifiez le nom (par exemple, votrenomFirstDataFactoryREST) dans le fichier **datafactory.json** . Consultez la rubrique [Data Factory - Règles d’affectation des noms](data-factory-naming-rules.md) pour savoir comment nommer les artefacts Data Factory.
  2. Dans la première commande où une valeur est attribuée à la variable **$cmd** , remplacez FirstDataFactoryREST par le nouveau nom, puis exécutez la commande.
  3. Exécutez les deux commandes suivantes pour appeler l’API REST afin de créer la fabrique de données et d’imprimer les résultats de l’opération.
* Pour créer des instances de fabrique de données, vous devez avoir le statut d’administrateur/collaborateur de l’abonnement Azure
* Le nom de la fabrique de données pourra être enregistré en tant que nom DNS et devenir ainsi visible publiquement.
* Si vous recevez le message d’erreur : «**L’abonnement n’est pas inscrit pour utiliser l’espace de noms Microsoft.DataFactory**», effectuez l’une des opérations suivantes et essayez de relancer la publication :

  * Dans Azure PowerShell, exécutez la commande suivante pour enregistrer le fournisseur Data Factory :

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```

      Vous pouvez exécuter la commande suivante pour confirmer l’enregistrement du fournisseur Data Factory :
    ```powershell
    Get-AzResourceProvider
    ```
  * Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide de l’abonnement Azure et accédez à un panneau Data Factory (ou) créez une fabrique de données dans le portail Azure. Cette action enregistre automatiquement le fournisseur.

Avant de créer un pipeline, vous devez d’abord créer quelques entités de la fabrique de données. Créez d’abord des services liés pour lier des magasins de données/calculs à votre magasin de données, puis définissez des jeux de données d’entrée et de sortie pour représenter les données dans les magasins de données liés.

## <a name="create-linked-services"></a>Créez des services liés
Dans cette étape, vous liez votre compte Stockage Azure et un cluster Azure HDInsight à la demande à votre fabrique de données. Le compte Stockage Azure contient les données d’entrée et de sortie pour le pipeline de cet exemple. Le service lié HDInsight est utilisé pour exécuter le script Hive spécifié dans l’activité du pipeline de cet exemple.

### <a name="create-azure-storage-linked-service"></a>Créer le service lié Stockage Azure
Dans cette étape, vous liez votre compte Stockage Azure à votre fabrique de données. Pour les besoins de ce didacticiel, vous utilisez le même compte Stockage Azure pour stocker les données d’entrée/sortie et le fichier de script HQL.

1. Attribuez la commande à une variable nommée **cmd**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data “@azurestoragelinkedservice.json” https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/AzureStorageLinkedService?api-version=2015-10-01};
    ```
2. Exécutez la commande à l’aide de **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Affichez les résultats. Si le service lié a été créé avec succès, le code JSON du service lié apparaît dans les **résultats**. Dans le cas contraire, un message d’erreur apparaît.

    ```powershell
    Write-Host $results
    ```

### <a name="create-azure-hdinsight-linked-service"></a>Créer le service lié Azure HDInsight
Dans cette étape, vous liez un cluster HDInsight à la demande à votre fabrique de données. Le cluster HDInsight est automatiquement créé lors de l’exécution, puis supprimé une fois le traitement effectué et au terme du délai d’inactivité spécifié. Vous pouvez utiliser votre propre cluster HDInsight au lieu d’utiliser un cluster HDInsight à la demande. Pour plus d’informations, consultez [Services de calcul liés](data-factory-compute-linked-services.md) .

1. Attribuez la commande à une variable nommée **cmd**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@hdinsightondemandlinkedservice.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/hdinsightondemandlinkedservice?api-version=2015-10-01};
    ```
2. Exécutez la commande à l’aide de **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Affichez les résultats. Si le service lié a été créé avec succès, le code JSON du service lié apparaît dans les **résultats**. Dans le cas contraire, un message d’erreur apparaît.

    ```powershell
    Write-Host $results
    ```

## <a name="create-datasets"></a>Créez les jeux de données
Dans cette étape, vous créez des jeux de données afin de représenter les données d’entrée et de sortie pour le traitement Hive. Ces jeux de données font référence au service **StorageLinkedService** que vous avez créé précédemment dans ce didacticiel. Le service lié pointe vers un compte de stockage Azure, et les jeux de données spécifient le conteneur, le dossier et le nom de fichier dans le stockage qui contient les données d’entrée et de sortie.

### <a name="create-input-dataset"></a>Créer le jeu de données d’entrée
Dans cette étape, vous créez le jeu de données d’entrée pour représenter les données d’entrée stockées dans le stockage d’objets blob Azure.

1. Attribuez la commande à une variable nommée **cmd**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@inputdataset.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/AzureBlobInput?api-version=2015-10-01};
    ```
2. Exécutez la commande à l’aide de **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Affichez les résultats. Si le jeu de données a été créé avec succès, le code JSON du jeu de données apparaît dans les **résultats**. Dans le cas contraire, un message d’erreur apparaît.

    ```powershell
    Write-Host $results
    ```

### <a name="create-output-dataset"></a>Créer un jeu de données de sortie
Dans cette étape, vous créez le jeu de données de sortie pour représenter les données de sortie stockées dans le stockage d’objets blob Azure.

1. Attribuez la commande à une variable nommée **cmd**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@outputdataset.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/AzureBlobOutput?api-version=2015-10-01};
    ```
2. Exécutez la commande à l’aide de **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Affichez les résultats. Si le jeu de données a été créé avec succès, le code JSON du jeu de données apparaît dans les **résultats**. Dans le cas contraire, un message d’erreur apparaît.

    ```powershell
    Write-Host $results
    ```

## <a name="create-pipeline"></a>Création d’un pipeline
Dans cette étape, vous créez votre premier pipeline avec une activité **HDInsightHive** . La tranche d’entrée est disponible mensuellement (fréquence : Mois, intervalle : 1), la tranche de sortie est produite mensuellement et la propriété du planificateur pour l’activité est également définie sur Mensuellement. Les paramètres pour le jeu de données de sortie et le planificateur d’activité doivent correspondre. À ce stade, c'est le jeu de données de sortie qui pilote la planification : vous devez donc créer un jeu de données de sortie même si l’activité ne génère aucune sortie. Si l’activité ne prend aucune entrée, vous pouvez ignorer la création du jeu de données d’entrée.

Vérifiez que le fichier **input.log** apparaît dans le dossier **adfgetstarted/inputdata** du Stockage Blob Azure, puis exécutez la commande suivante pour déployer le pipeline. Étant donné que les valeurs pour **start** et **end** sont définies sur des valeurs antérieures au moment actuel, et que **isPaused** est défini sur false, le pipeline (activité dans le pipeline) s’exécute immédiatement après le déploiement.

1. Attribuez la commande à une variable nommée **cmd**.

    ```powershell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@pipeline.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datapipelines/MyFirstPipeline?api-version=2015-10-01};
    ```
2. Exécutez la commande à l’aide de **Invoke-Command**.

    ```powershell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. Affichez les résultats. Si le jeu de données a été créé avec succès, le code JSON du jeu de données apparaît dans les **résultats**. Dans le cas contraire, un message d’erreur apparaît.

    ```powershell
    Write-Host $results
    ```
4. Félicitations, vous avez réussi à créer votre premier pipeline avec Azure PowerShell.

## <a name="monitor-pipeline"></a>Surveillance d’un pipeline
Dans cette étape, vous utilisez l’API REST Data Factory pour surveiller les tranches produites par le pipeline.

```powershell
$ds ="AzureBlobOutput"

$cmd = {.\curl.exe -X GET -H "Authorization: Bearer $accessToken" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/$ds/slices?start=1970-01-01T00%3a00%3a00.0000000Z"&"end=2016-08-12T00%3a00%3a00.0000000Z"&"api-version=2015-10-01};

$results2 = Invoke-Command -scriptblock $cmd;

IF ((ConvertFrom-Json $results2).value -ne $NULL) {
    ConvertFrom-Json $results2 | Select-Object -Expand value | Format-Table
} else {
        (convertFrom-Json $results2).RemoteException
}
```

> [!IMPORTANT]
> La création d’un cluster HDInsight à la demande prend généralement un certain temps (environ 20 minutes). Le pipeline devrait donc traiter la tranche en **30 minutes environ** .
>
>

Exécutez Invoke-Command et la commande suivante jusqu’à ce que la tranche soit à l’état **Prêt** ou **Échec**. Quand l’état du segment est Prêt, vérifiez la présence des données de sortie dans le dossier **partitioneddata** du conteneur **adfgetstarted** de votre stockage d’objets blob.  La création d’un cluster HDInsight à la demande prend généralement un certain temps.

![données de sortie](./media/data-factory-build-your-first-pipeline-using-rest-api/three-ouptut-files.png)

> [!IMPORTANT]
> Le fichier d’entrée sera supprimé lorsque la tranche est traitée avec succès. Par conséquent, si vous souhaitez réexécuter la tranche ou refaire le didacticiel, chargez le fichier d’entrée (input.log) dans le dossier inputdata du conteneur adfgetstarted.
>
>

Vous pouvez également utiliser le portail Azure pour surveiller les tranches et résoudre les éventuels problèmes. Consultez la page [Surveillance et gestion des pipelines d’Azure Data Factory](data-factory-monitor-manage-pipelines.md) pour plus d’informations.

## <a name="summary"></a>Résumé
Dans ce didacticiel, vous avez créé une fabrique de données Azure pour traiter des données en exécutant le script Hive sur un cluster Hadoop HDInsight. Vous avez effectué les étapes suivantes dans le portail Azure à l’aide de Data Factory Editor :

1. Création d’une **fabrique de données** Azure.
2. Création de deux **services liés**:
   1. **Azure Storage** pour lier à la fabrique de données votre stockage d’objets blob Azure contenant les fichiers d’entrée/sortie.
   2. **Azure HDInsight** à la demande pour lier à la fabrique de données un cluster Hadoop HDInsight à la demande. Azure Data Factory crée un cluster Hadoop HDInsight juste-à-temps pour traiter les données d’entrée et produire des données de sortie.
3. Création de deux **jeux de données** qui décrivent les données d’entrée et de sortie pour l’activité HDInsight Hive dans le pipeline.
4. Création d’un **pipeline** avec une activité **Hive HDInsight**.

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez créé un pipeline avec une activité de transformation (Activité HDInsight) qui exécute un script Hive sur un cluster Azure HDInsight à la demande. Pour voir comment utiliser une activité de copie pour copier des données depuis un objet blob Azure vers Azure SQL, consultez le [Didacticiel : copie de données depuis un objet blob Azure vers Azure SQL](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

## <a name="see-also"></a>Voir aussi
| Rubrique | Description |
|:--- |:--- |
| [Informations de référence sur l’API REST Data Factory](/rest/api/datafactory/) |Consultez la documentation complète sur les applets de commande de Data Factory |
| [Pipelines](data-factory-create-pipelines.md) |Cet article vous aide à comprendre les pipelines et les activités dans Azure Data Factory, et à les utiliser dans l’optique de créer des workflows pilotés par les données de bout en bout pour votre scénario ou votre entreprise. |
| [Groupes de données](data-factory-create-datasets.md) |Cet article vous aide à comprendre les jeux de données dans Azure Data Factory. |
| [Planification et exécution](data-factory-scheduling-and-execution.md) |Cet article explique les aspects de la planification et de l’exécution du modèle d’application Azure Data Factory. |
| [Surveiller et gérer les pipelines Azure Data Factory à l’aide de la nouvelle application de surveillance et gestion.](data-factory-monitor-manage-app.md) |Cet article décrit comment surveiller, gérer et déboguer les pipelines à l’aide de l’application de surveillance et gestion. |
