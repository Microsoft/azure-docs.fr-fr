---
title: Configurer le routage des messages pour Azure IoT Hub à l’aide d’Azure CLI
description: Configurez le routage des messages pour Azure IoT Hub à l’aide d’Azure CLI. En fonction des propriétés du message, effectuez le routage vers un compte de stockage ou une file d’attente Service Bus.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: robinsh
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 239d8f2bcc1422a1098fb8f6cb3fba6706d671f2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87500195"
---
# <a name="tutorial-use-the-azure-cli-to-configure-iot-hub-message-routing"></a>Tutoriel : Utiliser Azure CLI pour configurer le routage des messages IoT Hub

[!INCLUDE [iot-hub-include-routing-intro](../../includes/iot-hub-include-routing-intro.md)]

[!INCLUDE [iot-hub-include-routing-create-resources](../../includes/iot-hub-include-routing-create-resources.md)]

## <a name="download-the-script-optional"></a>Télécharger le script (facultatif)

Dans la deuxième partie de ce tutoriel, vous téléchargez et vous exécutez une application Visual Studio qui envoie des messages au hub IoT. Le téléchargement contient un dossier où se trouvent le modèle et le fichier de paramètres Azure Resource Manager ainsi que les scripts Azure CLI et PowerShell.

Si vous souhaitez voir le script terminé, téléchargez les [exemples Azure IoT pour C#](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Décompressez le fichier master.zip. Le script Azure CLI est dans le fichier **iothub_routing_cli.azcli**, sous /iot-hub/Tutorials/Routing/SimulatedDevice/resources/.

## <a name="use-the-azure-cli-to-create-your-resources"></a>Utiliser Azure CLI pour créer vos ressources

Copiez le script ci-dessous et collez-le dans Cloud Shell, puis appuyez sur Entrée. Le script est alors exécuté ligne par ligne. Cette première section du script crée les ressources de base utilisées dans ce tutoriel, dont le compte de stockage, le hub IoT, l’espace de noms Service Bus et la file d’attente Service Bus. Dans les étapes restantes du tutoriel, vous devrez copier chaque bloc de script et le coller dans Cloud Shell pour l’exécuter.

> [!TIP]
> Une remarque à propos du débogage : ce script utilise le symbole de continuation (la barre oblique inverse `\`) pour rendre le script plus lisible. Si vous rencontrez des problèmes durant l’exécution du script, vérifiez que votre session Cloud Shell exécute `bash`, et qu’il n’existe aucun espace après les barres obliques inverses.
> 

Plusieurs noms de ressources doivent être globalement uniques, comme le nom du hub IoT et le nom du compte de stockage. Pour faciliter le nommage, une valeur alphanumérique aléatoire appelée *randomValue* est ajoutée à ces noms de ressources. La valeur randomValue est générée une fois au début du script, puis ajoutée aux noms de ressources tout au long du script en fonction des besoins. Si vous ne souhaitez pas que cette valeur soit aléatoire, vous pouvez la définir sur une chaîne vide ou sur une valeur spécifique. 

> [!IMPORTANT]
> Les variables définies dans le script initial étant également utilisées par le script de routage, vous devez exécuter le script entier dans la même session Cloud Shell. Si vous ouvrez une nouvelle session pour exécuter le script de configuration du routage, plusieurs variables n’auront pas de valeurs définies.
>

```azurecli-interactive
# This command retrieves the subscription id of the current Azure account. 
# This field is used when setting up the routing queries.
subscriptionID=$(az account show --query id -o tsv)

# Concatenate this number onto the resources that have to be globally unique.
# You can set this to "" or to a specific value if you don't want it to be random.
# This retrieves a random value.
randomValue=$RANDOM

# This command installs the IOT Extension for Azure CLI.
# You only need to install this the first time.
# You need it to create the device identity. 
az extension add --name azure-iot

# Set the values for the resource names that 
#   don't have to be globally unique.
location=westus
resourceGroup=ContosoResources
iotHubConsumerGroup=ContosoConsumers
containerName=contosoresults
iotDeviceName=Contoso-Test-Device

# Create the resource group to be used
#   for all the resources for this tutorial.
az group create --name $resourceGroup \
    --location $location

# The IoT hub name must be globally unique, 
#   so add a random value to the end.
iotHubName=ContosoTestHub$randomValue 
echo "IoT hub name = " $iotHubName

# Create the IoT hub.
az iot hub create --name $iotHubName \
    --resource-group $resourceGroup \
    --sku S1 --location $location

# Add a consumer group to the IoT hub for the 'events' endpoint.
az iot hub consumer-group create --hub-name $iotHubName \
    --name $iotHubConsumerGroup

# The storage account name must be globally unique, 
#   so add a random value to the end.
storageAccountName=contosostorage$randomValue
echo "Storage account name = " $storageAccountName

# Create the storage account to be used as a routing destination.
az storage account create --name $storageAccountName \
    --resource-group $resourceGroup \
    --location $location \
    --sku Standard_LRS

# Get the primary storage account key. 
#    You need this to create the container.
storageAccountKey=$(az storage account keys list \
    --resource-group $resourceGroup \
    --account-name $storageAccountName \
    --query "[0].value" | tr -d '"') 

# See the value of the storage account key.
echo "storage account key = " $storageAccountKey

# Create the container in the storage account. 
az storage container create --name $containerName \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --public-access off

# The Service Bus namespace must be globally unique, 
#   so add a random value to the end.
sbNamespace=ContosoSBNamespace$randomValue
echo "Service Bus namespace = " $sbNamespace

# Create the Service Bus namespace.
az servicebus namespace create --resource-group $resourceGroup \
    --name $sbNamespace \
    --location $location

# The Service Bus queue name must be globally unique, 
#   so add a random value to the end.
sbQueueName=ContosoSBQueue$randomValue
echo "Service Bus queue name = " $sbQueueName

# Create the Service Bus queue to be used as a routing destination.
az servicebus queue create --name $sbQueueName \
    --namespace-name $sbNamespace \
    --resource-group $resourceGroup

# Create the IoT device identity to be used for testing.
az iot hub device-identity create --device-id $iotDeviceName \
    --hub-name $iotHubName

# Retrieve the information about the device identity, then copy the primary key to
#   Notepad. You need this to run the device simulation during the testing phase.
az iot hub device-identity show --device-id $iotDeviceName \
    --hub-name $iotHubName
```

Maintenant que vous avez configuré les ressources de base, vous pouvez configurer le routage des messages.

## <a name="set-up-message-routing"></a>Configurer le routage de messages

[!INCLUDE [iot-hub-include-create-routing-description](../../includes/iot-hub-include-create-routing-description.md)]

Pour créer un point de terminaison de routage, utilisez [az iot hub routing-endpoint create](/cli/azure/iot/hub/routing-endpoint?view=azure-cli-latest#az-iot-hub-routing-endpoint-create). Pour créer la route des messages pour le point de terminaison, utilisez [az iot hub route create](/cli/azure/iot/hub/route?view=azure-cli-latest#az-iot-hub-route-create).

### <a name="route-to-a-storage-account"></a>Router vers un compte de stockage

[!INCLUDE [iot-hub-include-blob-storage-format](../../includes/iot-hub-include-blob-storage-format.md)]

Configurez d’abord le point de terminaison pour le compte de stockage, puis configurez la route. 

Voici les variables utilisées par le script, que vous devez définir dans votre session Cloud Shell :

**storageConnectionString** : cette valeur est récupérée auprès du compte de stockage configuré dans le script précédent. Elle est utilisée par le routage des messages pour accéder au compte de stockage.

  **resourceGroup** : il existe deux occurrences du groupe de ressources : définissez-les sur votre groupe de ressources.

**endpoint subscriptionID** : ce champ a pour valeur l’ID d’abonnement Azure pour le point de terminaison. 

**endpointType** : ce champ indique le type de point de terminaison. Il peut avoir la valeur `azurestoragecontainer`, `eventhub`, `servicebusqueue` ou `servicebustopic`. Ici, définissez-le sur `azurestoragecontainer`.

**iotHubName** : ce champ est le nom du hub qui effectuera le routage.

**containerName** : ce champ indique le nom du conteneur du compte de stockage dans lequel les données seront écrites.

**encoding** : Ce champ a la valeur `avro` ou `json`. Ceci indique le format des données stockées.

**routeName** : ce champ indique le nom de la route que vous configurez. 

**endpointName** : ce champ indique le nom qui identifie le point de terminaison. 

**enabled** : La valeur par défaut de ce champ est `true`, ce qui indique que le routage du message doit être activé après sa création.

**condition** : ce champ correspond à la requête utilisée pour filtrer les messages envoyés à ce point de terminaison. La condition de requête pour les messages routés vers le stockage est `level="storage"`.

Copiez ce script et collez-le dans la fenêtre Cloud Shell, puis exécutez-le.

```azurecli
##### ROUTING FOR STORAGE ##### 

endpointName="ContosoStorageEndpoint"
endpointType="azurestoragecontainer"
routeName="ContosoStorageRoute"
condition='level="storage"'

# Get the connection string for the storage account.
# Adding the "-o tsv" makes it be returned without the default double quotes around it.
storageConnectionString=$(az storage account show-connection-string \
  --name $storageAccountName --query connectionString -o tsv)
```

L’étape suivante consiste à créer le point de terminaison de routage pour le compte de stockage. Vous spécifiez également le conteneur où les résultats seront stockés. Le conteneur a été créé en même temps que le compte de stockage.

```azurecli
# Create the routing endpoint for storage.
az iot hub routing-endpoint create \
  --connection-string $storageConnectionString \
  --endpoint-name $endpointName \
  --endpoint-resource-group $resourceGroup \
  --endpoint-subscription-id $subscriptionID \
  --endpoint-type $endpointType \
  --hub-name $iotHubName \
  --container $containerName \
  --resource-group $resourceGroup \
  --encoding avro
```

Ensuite, vous créez la route pour le point de terminaison de stockage. La route des messages spécifie où envoyer les messages qui remplissent la spécification de la requête. 

```azurecli
# Create the route for the storage endpoint.
az iot hub route create \
  --name $routeName \
  --hub-name $iotHubName \
  --source devicemessages \
  --resource-group $resourceGroup \
  --endpoint-name $endpointName \
  --enabled \
  --condition $condition
```

### <a name="route-to-a-service-bus-queue"></a>Router vers une file d’attente Service Bus

Configurez maintenant le routage pour la file d’attente Service Bus. Pour récupérer la chaîne de connexion pour la file d’attente Service Bus, vous devez créer une règle d’autorisation ayant les droits appropriés. Le script suivant crée une règle d’autorisation pour la file d’attente Service Bus appelée `sbauthrule`, puis il définit les droits sur `Listen Manage Send`. Une fois que vous avez défini cette règle d’autorisation, vous pouvez l’utiliser pour récupérer la chaîne de connexion de la file d’attente.

```azurecli
# Create the authorization rule for the Service Bus queue.
az servicebus queue authorization-rule create \
  --name "sbauthrule" \
  --namespace-name $sbNamespace \
  --queue-name $sbQueueName \
  --resource-group $resourceGroup \
  --rights Listen Manage Send \
  --subscription $subscriptionID
```

Utilisez maintenant la règle d’autorisation pour récupérer la chaîne de connexion à la file d’attente Service Bus.

```azurecli
# Get the Service Bus queue connection string.
# The "-o tsv" ensures it is returned without the default double-quotes.
sbqConnectionString=$(az servicebus queue authorization-rule keys list \
  --name "sbauthrule" \
  --namespace-name $sbNamespace \
  --queue-name $sbQueueName \
  --resource-group $resourceGroup \
  --subscription $subscriptionID \
  --query primaryConnectionString -o tsv)

# Show the Service Bus queue connection string.
echo "service bus queue connection string = " $sbqConnectionString
```

Configurez maintenant le point de terminaison de routage et la route des messages pour la file d’attente Service Bus. Voici les variables utilisées par le script, que vous devez définir dans votre session Cloud Shell :

**endpointName** : ce champ indique le nom qui identifie le point de terminaison. 

**endpointType** : ce champ indique le type de point de terminaison. Il peut avoir la valeur `azurestoragecontainer`, `eventhub`, `servicebusqueue` ou `servicebustopic`. Ici, définissez-le sur `servicebusqueue`.

**routeName** : ce champ indique le nom de la route que vous configurez. 

**condition** : ce champ correspond à la requête utilisée pour filtrer les messages envoyés à ce point de terminaison. La condition de requête pour les messages routés vers la file d’attente Service Bus est `level="critical"`.

Voici l’interface Azure CLI pour le point de terminaison de routage et la route des messages pour la file d’attente Service Bus.

```azurecli
endpointName="ContosoSBQueueEndpoint"
endpointType="ServiceBusQueue"
routeName="ContosoSBQueueRoute"
condition='level="critical"'

# Set up the routing endpoint for the Service Bus queue.
# This uses the Service Bus queue connection string.
az iot hub routing-endpoint create \
  --connection-string $sbqConnectionString \
  --endpoint-name $endpointName \
  --endpoint-resource-group $resourceGroup \
  --endpoint-subscription-id $subscriptionID \
  --endpoint-type $endpointType \
  --hub-name $iotHubName \
  --resource-group $resourceGroup 

# Set up the message route for the Service Bus queue endpoint.
az iot hub route create --name $routeName \
  --hub-name $iotHubName \
  --source-type devicemessages \
  --resource-group $resourceGroup \
  --endpoint-name $endpointName \
  --enabled \
  --condition $condition
  ```

### <a name="view-message-routing-in-the-portal"></a>Visualiser le routage des messages dans le portail

[!INCLUDE [iot-hub-include-view-routing-in-portal](../../includes/iot-hub-include-view-routing-in-portal.md)]

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez terminé la configuration des ressources et des routes des messages, passez au tutoriel suivant pour découvrir comment envoyer des messages au hub IoT et visualiser les routes qu’ils empruntent vers les différentes destinations. 

> [!div class="nextstepaction"]
> [Partie 2 - Voir les résultats du routage des messages](tutorial-routing-view-message-routing-results.md)
