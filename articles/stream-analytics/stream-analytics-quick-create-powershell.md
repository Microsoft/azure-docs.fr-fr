---
title: Démarrage rapide – Créer une tâche Stream Analytics à l’aide d’Azure PowerShell
description: Ce guide de démarrage rapide montre comment utiliser le module Azure PowerShell pour déployer et exécuter un travail Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.date: 12/20/2018
ms.topic: quickstart
ms.service: stream-analytics
ms.custom: mvc, devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 2ea92ad2e9c81b568e11ff97d7b6a88eeb4f188e
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93346568"
---
# <a name="quickstart-create-a-stream-analytics-job-using-azure-powershell"></a>Démarrage rapide : Créer un travail Stream Analytics à l’aide d’Azure PowerShell

Le module Azure PowerShell est utilisé pour créer et gérer des ressources Azure à l’aide des applets de commande ou des scripts PowerShell. Ce démarrage rapide décrit en détail comment utiliser le module Azure PowerShell pour déployer et exécuter un travail Azure Stream Analytics.

Cet exemple de travail lit les données de streaming à partir d’un appareil IoT Hub. Les données d’entrée sont générées par un simulateur en ligne Raspberry Pi. Ensuite, le travail Stream Analytics transforme les données en utilisant le langage de requête Stream Analytics pour filtrer les messages indiquant une température supérieure à 27 degrés. Enfin, ce travail écrit les événements de sortie résultants dans un fichier du stockage d’objets blob.

## <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/).

* Ce démarrage rapide nécessite le module Azure PowerShell. Exécutez `Get-Module -ListAvailable Az` pour rechercher la version installée sur votre ordinateur local. Si vous devez installer ou mettre à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps).

* Certaines actions IoT Hub ne sont pas prises en charge par Azure PowerShell et doivent être effectuées à l’aide d’Azure CLI version 2.0.70 ou ultérieure, ainsi que l’extension IoT pour Azure CLI. [Installez Azure CLI](/cli/azure/install-azure-cli) et utilisez `az extension add --name azure-iot` pour installer l’extension IoT.


## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous à votre abonnement Azure avec la commande `Connect-AzAccount`, puis entrez vos informations d’identification Azure dans le navigateur contextuel :

```powershell
# Connect to your Azure account
Connect-AzAccount
```

Si vous avez plusieurs abonnements, sélectionnez l’abonnement que vous souhaitez utiliser dans ce guide de démarrage rapide en exécutant les applets de commande suivantes. Assurez-vous de remplacer `<your subscription name>` par le nom de votre abonnement :

```powershell
# List all available subscriptions.
Get-AzSubscription

# Select the Azure subscription you want to use to create the resource group and resources.
Get-AzSubscription -SubscriptionName "<your subscription name>" | Select-AzSubscription
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources Azure avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

```powershell
$resourceGroup = "StreamAnalyticsRG"
$location = "WestUS2"
New-AzResourceGroup `
    -Name $resourceGroup `
    -Location $location
```

## <a name="prepare-the-input-data"></a>Préparer les données d’entrée

Avant de définir le travail Stream Analytics, préparez les données qui seront configurées en tant qu’entrée pour le travail.

Le bloc de code Azure CLI ci-après exécute de nombreuses commandes pour préparer les données d’entrée requises par le travail. Passez en revue les différentes sections pour comprendre le code.

1. Dans la fenêtre PowerShell, exécutez la commande [az login](/cli/azure/authenticate-azure-cli) pour vous connecter à votre compte Azure.

    Une fois que vous êtes connecté, Azure CLI retourne une liste de vos abonnements. Copiez l’abonnement que vous utilisez dans ce guide de démarrage rapide et exécutez la commande [az account set](/cli/azure/manage-azure-subscriptions-azure-cli#change-the-active-subscription) pour sélectionner cet abonnement. Choisissez le même abonnement que vous avez sélectionné dans la section précédente avec PowerShell. Veillez à remplacer `<your subscription name>` par le nom de votre abonnement.

    ```azurecli
    az login

    az account set --subscription "<your subscription>"
    ```

2. Créez un hub IoT à l’aide de la commande [az iot hub create](../iot-hub/iot-hub-create-using-cli.md#create-an-iot-hub). Cet exemple crée un hub IoT appelé **MyASAIoTHub**. Comme les noms de hub IoT sont uniques, vous devez spécifier votre propre nom de hub IoT. Définissez la référence SKU sur F1 pour utiliser le niveau Gratuit s’il est disponible avec votre abonnement. Si ce n’est pas le cas, choisissez le niveau le plus bas suivant.

    ```azurecli
    az iot hub create --name "<your IoT Hub name>" --resource-group $resourceGroup --sku S1
    ```

    Après avoir créé le hub IoT, obtenez la chaîne de connexion du hub IoT à l’aide de la commande [az iot hub show-connection-string](/cli/azure/iot/hub). Copiez la chaîne de connexion complète et enregistrez-la, car vous en aurez besoin quand vous ajouterez le hub IoT comme entrée dans votre travail Stream Analytics.

    ```azurecli
    az iot hub show-connection-string --hub-name "MyASAIoTHub"
    ```

3. Ajoutez un appareil au hub IoT à l’aide de la commande [az iothub device-identity create](../iot-hub/quickstart-send-telemetry-c.md#register-a-device). Cet exemple crée un appareil appelé **MyASAIoTDevice**.

    ```azurecli
    az iot hub device-identity create --hub-name "MyASAIoTHub" --device-id "MyASAIoTDevice"
    ```

4. Obtenez la chaîne de connexion de l’appareil à l’aide de la commande [az iot hub device-identity show-connection-string](/cli/azure/ext/azure-iot/iot/hub/device-identity#ext-azure-iot-az-iot-hub-device-identity-show-connection-string). Copiez la chaîne de connexion complète et enregistrez-la, car vous en aurez besoin quand vous créerez le simulateur Raspberry Pi.

    ```azurecli
    az iot hub device-identity show-connection-string --hub-name "MyASAIoTHub" --device-id "MyASAIoTDevice" --output table
    ```

    **Exemple de sortie :**

    ```output
    HostName=MyASAIoTHub.azure-devices.net;DeviceId=MyASAIoTDevice;SharedAccessKey=a2mnUsg52+NIgYudxYYUNXI67r0JmNubmfVafojG8=
    ```

## <a name="create-blob-storage"></a>Créer un stockage d’objets blob

Le bloc de code Azure PowerShell suivant utilise des commandes afin de créer le stockage d’objets blob utilisé pour la sortie du travail. Passez en revue les différentes sections pour comprendre le code.

1. Créez un compte de stockage standard à usage général avec l’applet de commande [New-AzStorageAccount](/powershell/module/az.storage/New-azStorageAccount).  Cet exemple crée un compte de stockage nommé **myasaquickstartstorage** avec les options de stockage localement redondant (LRS) et de chiffrement des objets blob (options activées par défaut).

2. Récupérez le contexte de compte de stockage `$storageAccount.Context` qui définit le compte de stockage à utiliser. Lorsque vous utilisez des comptes de stockage, référencez le contexte au lieu d’entrer les informations d’identification à plusieurs reprises.

3. Créez un conteneur de stockage avec la commande [New-AzStorageContainer](/powershell/module/az.storage/new-azstoragecontainer).

4. Copiez la clé de stockage générée par le code et enregistrez-la, car vous en aurez besoin plus tard pour créer la sortie du travail de streaming.

    ```powershell
    $storageAccountName = "myasaquickstartstorage"
    $storageAccount = New-AzStorageAccount `
      -ResourceGroupName $resourceGroup `
      -Name $storageAccountName `
      -Location $location `
      -SkuName Standard_LRS `
      -Kind Storage

    $ctx = $storageAccount.Context
    $containerName = "container1"

    New-AzStorageContainer `
      -Name $containerName `
      -Context $ctx

    $storageAccountKey = (Get-AzStorageAccountKey `
      -ResourceGroupName $resourceGroup `
      -Name $storageAccountName).Value[0]

    Write-Host "The <storage account key> placeholder needs to be replaced in your output json files with this key value:"
    Write-Host $storageAccountKey -ForegroundColor Cyan
    ```

## <a name="create-a-stream-analytics-job"></a>Création d’un travail Stream Analytics

Créez un travail Stream Analytics avec l’applet de commande [New-AzStreamAnalyticsJob](/powershell/module/az.streamanalytics/new-azstreamanalyticsjob). Cette applet de commande prend le nom du travail, le nom du groupe de ressources et la définition du travail comme paramètres. Le nom du travail peut correspondre à n’importe quel nom convivial identifiant votre travail. Ce nom peut contenir uniquement des caractères alphanumériques, des traits d’union et des traits de soulignement, et doit comporter entre 3 et 63 caractères. La définition du travail est un fichier JSON qui contient les propriétés nécessaires à la création d’un travail. Sur votre ordinateur local, créez un fichier nommé `JobDefinition.json` et ajoutez-y les données JSON suivantes :

```json
{
  "location":"WestUS2",
  "properties":{
    "sku":{
      "name":"standard"
    },
    "eventsOutOfOrderPolicy":"adjust",
    "eventsOutOfOrderMaxDelayInSeconds":10,
    "compatibilityLevel": 1.1
  }
}
```

Ensuite, exécutez l’applet de commande `New-AzStreamAnalyticsJob`. Remplacez la valeur de la variable `jobDefinitionFile` par le chemin où vous avez enregistré le fichier JSON de définition du travail.

```powershell
$jobName = "MyStreamingJob"
$jobDefinitionFile = "C:\JobDefinition.json"
New-AzStreamAnalyticsJob `
  -ResourceGroupName $resourceGroup `
  -File $jobDefinitionFile `
  -Name $jobName `
  -Force
```

## <a name="configure-input-to-the-job"></a>Configurer l’entrée du travail

Ajoutez une entrée à votre travail avec l’applet de commande [New-AzStreamAnalyticsInput](/powershell/module/az.streamanalytics/new-azstreamanalyticsinput). Cette applet de commande prend le nom du travail, le nom de l’entrée du travail, le nom du groupe de ressources et la définition de l’entrée du travail comme paramètres. La définition de l’entrée du travail est un fichier JSON qui contient les propriétés nécessaires à la configuration de l’entrée du travail. Dans cet exemple, vous allez créer un stockage d’objets blob comme entrée.

Sur votre ordinateur local, créez un fichier nommé `JobInputDefinition.json` et ajoutez-y les données JSON ci-après. Remplacez la valeur de `accesspolicykey` par la partie `SharedAccessKey` de la chaîne de connexion du hub IoT que vous avez enregistrée dans une section précédente.

```json
{
    "properties": {
        "type": "Stream",
        "datasource": {
            "type": "Microsoft.Devices/IotHubs",
            "properties": {
                "iotHubNamespace": "MyASAIoTHub",
                "sharedAccessPolicyName": "iothubowner",
                "sharedAccessPolicyKey": "accesspolicykey",
                "endpoint": "messages/events",
                "consumerGroupName": "$Default"
                }
        },
        "compression": {
            "type": "None"
        },
        "serialization": {
            "type": "Json",
            "properties": {
                "encoding": "UTF8"
            }
        }
    },
    "name": "IoTHubInput",
    "type": "Microsoft.StreamAnalytics/streamingjobs/inputs"
}
```

Ensuite, exécutez l’applet de commande `New-AzStreamAnalyticsInput` et remplacez la valeur de la variable `jobDefinitionFile` par le chemin où vous avez enregistré le fichier JSON de définition de l’entrée du travail.

```powershell
$jobInputName = "IoTHubInput"
$jobInputDefinitionFile = "C:\JobInputDefinition.json"
New-AzStreamAnalyticsInput `
  -ResourceGroupName $resourceGroup `
  -JobName $jobName `
  -File $jobInputDefinitionFile `
  -Name $jobInputName
```

## <a name="configure-output-to-the-job"></a>Configurer la sortie du travail

Ajoutez une sortie à votre travail avec l’applet de commande [New-AzStreamAnalyticsOutput](/powershell/module/az.streamanalytics/new-azstreamanalyticsoutput). Cette applet de commande prend le nom du travail, le nom de la sortie du travail, le nom du groupe de ressources et la définition de la sortie du travail comme paramètres. La définition de la sortie du travail est un fichier JSON qui contient les propriétés nécessaires à la configuration de la sortie du travail. Cet exemple utilise le stockage blob en guise de sortie.

Sur votre ordinateur local, créez un fichier nommé `JobOutputDefinition.json` et ajoutez-y les données JSON ci-après. Veillez à remplacer la valeur de `accountKey` par la clé d’accès de votre compte de stockage qui correspond à la valeur stockée dans $storageAccountKey.

```json
{
    "properties": {
        "datasource": {
            "type": "Microsoft.Storage/Blob",
            "properties": {
                "storageAccounts": [
                    {
                      "accountName": "asaquickstartstorage",
                      "accountKey": "<storage account key>"
                    }
                ],
                "container": "container1",
                "pathPattern": "output/",
                "dateFormat": "yyyy/MM/dd",
                "timeFormat": "HH"
            }
        },
        "serialization": {
            "type": "Json",
            "properties": {
                "encoding": "UTF8",
                "format": "LineSeparated"
            }
        }
    },
    "name": "BlobOutput",
    "type": "Microsoft.StreamAnalytics/streamingjobs/outputs"
}
```

Ensuite, exécutez l’applet de commande `New-AzStreamAnalyticsOutput`. Veillez à remplacer la valeur de la variable `jobOutputDefinitionFile` par le chemin dans lequel vous avez stocké le fichier JSON de définition de la sortie du travail.

```powershell
$jobOutputName = "BlobOutput"
$jobOutputDefinitionFile = "C:\JobOutputDefinition.json"
New-AzStreamAnalyticsOutput `
  -ResourceGroupName $resourceGroup `
  -JobName $jobName `
  -File $jobOutputDefinitionFile `
  -Name $jobOutputName -Force
```

## <a name="define-the-transformation-query"></a>Définir la requête de transformation

Ajoutez une transformation à votre travail avec l’applet de commande [New-AzStreamAnalyticsTransformation](/powershell/module/az.streamanalytics/new-azstreamanalyticstransformation). Cette applet de commande prend le nom du travail, le nom de la transformation du travail, le nom du groupe de ressources et la définition de la transformation du travail comme paramètres. Sur votre ordinateur local, créez un fichier nommé `JobTransformationDefinition.json` et ajoutez-y les données JSON ci-après. Le fichier JSON contient un paramètre de requête qui définit la requête de transformation :

```json
{
    "name":"MyTransformation",
    "type":"Microsoft.StreamAnalytics/streamingjobs/transformations",
    "properties":{
        "streamingUnits":1,
        "script":null,
        "query":" SELECT * INTO BlobOutput FROM IoTHubInput HAVING Temperature > 27"
    }
}
```

Ensuite, exécutez l’applet de commande `New-AzStreamAnalyticsTransformation`. Remplacez la valeur de la variable `jobTransformationDefinitionFile` par le chemin où vous avez enregistré le fichier JSON de définition de la transformation du travail.

```powershell
$jobTransformationName = "MyJobTransformation"
$jobTransformationDefinitionFile = "C:\JobTransformationDefinition.json"
New-AzStreamAnalyticsTransformation `
  -ResourceGroupName $resourceGroup `
  -JobName $jobName `
  -File $jobTransformationDefinitionFile `
  -Name $jobTransformationName -Force
```

## <a name="run-the-iot-simulator"></a>Exécuter le simulateur IoT

1. Ouvrez le [simulateur en ligne Azure IoT Raspberry Pi](https://azure-samples.github.io/raspberry-pi-web-simulator/).

2. Remplacez l’espace réservé à la ligne 15 par la chaîne de connexion entière de l’appareil Azure IoT Hub que vous avez enregistrée dans une section précédente.

3. Cliquez sur **Exécuter**. La sortie doit indiquer les données de capteur et les messages qui sont envoyés à votre hub IoT.

    ![Simulateur en ligne Azure IoT Raspberry Pi](./media/stream-analytics-quick-create-powershell/ras-pi-connection-string.png)

## <a name="start-the-stream-analytics-job-and-check-the-output"></a>Démarrer le travail Stream Analytics et observer le résultat

Démarrez le travail avec l’applet de commande [Start-AzStreamAnalyticsJob](/powershell/module/az.streamanalytics/start-azstreamanalyticsjob). Cette applet de commande prend le nom du travail, le nom du groupe de ressources, le mode de démarrage de la sortie et l’heure de démarrage comme paramètres. `OutputStartMode` accepte les valeurs de `JobStartTime`, `CustomTime` ou `LastOutputEventTime`. Pour plus d’informations sur les éléments auxquels ces valeurs font référence, consultez la section [Paramètres](/powershell/module/az.streamanalytics/start-azstreamanalyticsjob) de la documentation PowerShell.

Après son exécution, l’applet de commande ci-après renvoie la valeur `True` en guise de sortie si le travail démarre. Dans le conteneur de stockage, un dossier de sortie est créé avec les données transformées.

```powershell
Start-AzStreamAnalyticsJob `
  -ResourceGroupName $resourceGroup `
  -Name $jobName `
  -OutputStartMode 'JobStartTime'
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, le travail de streaming et toutes les ressources associées. La suppression du travail évite la facturation des unités de streaming consommées par le travail. Si vous envisagez d’utiliser le travail à l’avenir, vous pouvez vous abstenir de le supprimer et vous contenter de l’arrêter à ce stade. Si vous n’avez plus besoin de ce travail, supprimez toutes les ressources créées dans le cadre de ce guide de démarrage rapide en exécutant l’applet de commande suivante :

```powershell
Remove-AzResourceGroup `
  -Name $resourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez déployé un travail Stream Analytics simple à l’aide de PowerShell. Vous pouvez également déployer des travaux Stream Analytics à l’aide du [Portail Azure](stream-analytics-quick-create-portal.md) et de [Visual Studio](stream-analytics-quick-create-vs.md).

Pour savoir comment configurer d’autres sources d’entrée et effectuer une détection en temps réel, passez à l’article suivant :

> [!div class="nextstepaction"]
> [Détection des fraudes en temps réel à l’aide d’Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)