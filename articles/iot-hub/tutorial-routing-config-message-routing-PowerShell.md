---
title: Configurer le routage des messages pour Azure IoT Hub à l’aide d’Azure PowerShell
description: Configurez le routage des messages pour Azure IoT Hub à l’aide d’Azure PowerShell. En fonction des propriétés du message, effectuez le routage vers un compte de stockage ou une file d’attente Service Bus.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: robinsh
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 92c30287e836ccadda915dab70a4d40e9c4e851a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89073401"
---
# <a name="tutorial-use-azure-powershell-to-configure-iot-hub-message-routing"></a>Tutoriel : Utiliser Azure PowerShell pour configurer le routage des messages IoT Hub

[!INCLUDE [iot-hub-include-routing-intro](../../includes/iot-hub-include-routing-intro.md)]

[!INCLUDE [iot-hub-include-routing-create-resources](../../includes/iot-hub-include-routing-create-resources.md)]

## <a name="download-the-script-optional"></a>Télécharger le script (facultatif)

Dans la deuxième partie de ce tutoriel, vous téléchargez et vous exécutez une application Visual Studio qui envoie des messages au hub IoT. Le téléchargement contient un dossier où se trouvent le modèle et le fichier de paramètres Azure Resource Manager ainsi que les scripts Azure CLI et PowerShell. 

Si vous souhaitez voir le script terminé, téléchargez les [exemples Azure IoT pour C#](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Décompressez le fichier master.zip. Le script Azure CLI est dans le fichier **iothub_routing_psh.ps1**, sous /iot-hub/Tutorials/Routing/SimulatedDevice/resources/.

## <a name="create-your-resources"></a>Créer vos ressources

Commencez par créer les ressources avec PowerShell.

### <a name="use-powershell-to-create-your-base-resources"></a>Utiliser PowerShell pour créer vos ressources de base

Copiez le script ci-dessous et collez-le dans Cloud Shell, puis appuyez sur Entrée. Le script est alors exécuté ligne par ligne. Cette première section du script crée les ressources de base utilisées dans ce tutoriel, dont le compte de stockage, le hub IoT, l’espace de noms Service Bus et la file d’attente Service Bus. Dans les étapes suivantes du tutoriel, vous devrez copier chaque bloc de script et le coller dans Cloud Shell pour l’exécuter.

Plusieurs noms de ressources doivent être globalement uniques, comme le nom du hub IoT et le nom du compte de stockage. Pour faciliter le nommage, une valeur alphanumérique aléatoire appelée *randomValue* est ajoutée à ces noms de ressources. La valeur randomValue est générée une fois au début du script, puis ajoutée aux noms de ressources tout au long du script en fonction des besoins. Si vous ne souhaitez pas que cette valeur soit aléatoire, vous pouvez la définir sur une chaîne vide ou sur une valeur spécifique. 

> [!IMPORTANT]
> Les variables définies dans le script initial étant également utilisées par le script de routage, vous devez exécuter le script entier dans la même session Cloud Shell. Si vous ouvrez une nouvelle session pour exécuter le script de configuration du routage, plusieurs variables n’auront pas de valeurs définies. 
>

```azurepowershell-interactive
# This command retrieves the subscription id of the current Azure account.
# This field is used when setting up the routing queries.
$subscriptionID = (Get-AzContext).Subscription.Id

# Concatenate this number onto the resources that have to be globally unique.
# You can set this to "" or to a specific value if you don't want it to be random.
# This retrieves the first 6 digits of a random value.
$randomValue = "$(Get-Random)".Substring(0,6)

# Set the values for the resource names that don't have to be globally unique.
$location = "West US"
$resourceGroup = "ContosoResources"
$iotHubConsumerGroup = "ContosoConsumers"
$containerName = "contosoresults"

# Create the resource group to be used 
#   for all resources for this tutorial.
New-AzResourceGroup -Name $resourceGroup -Location $location

# The IoT hub name must be globally unique, 
#   so add a random value to the end.
$iotHubName = "ContosoTestHub" + $randomValue
Write-Host "IoT hub name is " $iotHubName

# Create the IoT hub.
New-AzIotHub -ResourceGroupName $resourceGroup `
    -Name $iotHubName `
    -SkuName "S1" `
    -Location $location `
    -Units 1 

# Add a consumer group to the IoT hub for the 'events' endpoint.
Add-AzIotHubEventHubConsumerGroup -ResourceGroupName $resourceGroup `
  -Name $iotHubName `
  -EventHubConsumerGroupName $iotHubConsumerGroup `
  -EventHubEndpointName "events"

# The storage account name must be globally unique, so add a random value to the end.
$storageAccountName = "contosostorage" + $randomValue
Write-Host "storage account name is " $storageAccountName

# Create the storage account to be used as a routing destination.
# Save the context for the storage account 
#   to be used when creating a container.
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
    -Name $storageAccountName `
    -Location $location `
    -SkuName Standard_LRS `
    -Kind Storage
# Retrieve the connection string from the context. 
$storageConnectionString = $storageAccount.Context.ConnectionString
Write-Host "storage connection string = " $storageConnectionString 

# Create the container in the storage account.
New-AzStorageContainer -Name $containerName `
    -Context $storageAccount.Context

# The Service Bus namespace must be globally unique,
#   so add a random value to the end.
$serviceBusNamespace = "ContosoSBNamespace" + $randomValue
Write-Host "Service Bus namespace is " $serviceBusNamespace

# Create the Service Bus namespace.
New-AzServiceBusNamespace -ResourceGroupName $resourceGroup `
    -Location $location `
    -Name $serviceBusNamespace 

# The Service Bus queue name must be globally unique,
#  so add a random value to the end.
$serviceBusQueueName  = "ContosoSBQueue" + $randomValue
Write-Host "Service Bus queue name is " $serviceBusQueueName 

# Create the Service Bus queue to be used as a routing destination.
New-AzServiceBusQueue -ResourceGroupName $resourceGroup `
    -Namespace $serviceBusNamespace `
    -Name $serviceBusQueueName  `
    -EnablePartitioning $False 
```

### <a name="create-a-simulated-device"></a>Créez un appareil simulé.

[!INCLUDE [iot-hub-include-create-simulated-device-portal](../../includes/iot-hub-include-create-simulated-device-portal.md)]

Maintenant que vous avez configuré les ressources de base, vous pouvez configurer le routage des messages.

## <a name="set-up-message-routing"></a>Configurer le routage de messages

[!INCLUDE [iot-hub-include-create-routing-description](../../includes/iot-hub-include-create-routing-description.md)]

Pour créer un point de terminaison de routage, utilisez [Add-AzIotHubRoutingEndpoint](/powershell/module/az.iothub/Add-AzIotHubRoutingEndpoint). Pour créer la route des messages vers le point de terminaison, utilisez [Add-AzIotHubRoute](/powershell/module/az.iothub/Add-AzIoTHubRoute).

### <a name="route-to-a-storage-account"></a>Router vers un compte de stockage 

Configurez d’abord le point de terminaison pour le compte de stockage, puis créez la route des messages.

[!INCLUDE [iot-hub-include-blob-storage-format](../../includes/iot-hub-include-blob-storage-format.md)]

Voici les variables utilisées par le script, que vous devez définir dans votre session Cloud Shell :

**resourceGroup** : ce champ apparaît à deux endroits. Définissez chaque occurrence sur votre groupe de ressources.

**nom** : ce champ indique le nom du hub IoT auquel s’applique le routage.

**endpointName** : ce champ indique le nom qui identifie le point de terminaison. 

**endpointType** : ce champ indique le type de point de terminaison. Il peut avoir la valeur `azurestoragecontainer`, `eventhub`, `servicebusqueue` ou `servicebustopic`. Ici, définissez-le sur `azurestoragecontainer`.

**subscriptionID** : ce champ a pour valeur l’ID d’abonnement associé à votre compte Azure.

**storageConnectionString** : cette valeur est récupérée auprès du compte de stockage configuré dans le script précédent. Elle est utilisée par le routage pour accéder au compte de stockage.

**containerName** : ce champ indique le nom du conteneur du compte de stockage dans lequel les données seront écrites.

**Encoding** : définissez ce champ à la valeur `AVRO` ou `JSON`. Cette valeur spécifie le format des données stockées. La valeur par défaut est AVRO.

**routeName** : ce champ indique le nom de la route que vous configurez. 

**condition** : ce champ correspond à la requête utilisée pour filtrer les messages envoyés à ce point de terminaison. La condition de requête pour les messages routés vers le stockage est `level="storage"`.

**enabled** : La valeur par défaut de ce champ est `true`, ce qui indique que le routage du message doit être activé après sa création.

Copiez ce script et collez-le dans la fenêtre Cloud Shell.

```powershell
##### ROUTING FOR STORAGE #####

$endpointName = "ContosoStorageEndpoint"
$endpointType = "azurestoragecontainer"
$routeName = "ContosoStorageRoute"
$condition = 'level="storage"'
```

L’étape suivante consiste à créer le point de terminaison de routage pour le compte de stockage. Vous spécifiez également le conteneur où les résultats seront stockés. Le conteneur a été créé en même temps que le compte de stockage.

```powershell
# Create the routing endpoint for storage.
# Specify 'AVRO' or 'JSON' for the encoding of the data.
Add-AzIotHubRoutingEndpoint `
  -ResourceGroupName $resourceGroup `
  -Name $iotHubName `
  -EndpointName $endpointName `
  -EndpointType $endpointType `
  -EndpointResourceGroup $resourceGroup `
  -EndpointSubscriptionId $subscriptionId `
  -ConnectionString $storageConnectionString  `
  -ContainerName $containerName `
  -Encoding AVRO
```

Ensuite, vous créez la route des messages vers le point de terminaison de stockage. La route des messages spécifie où envoyer les messages qui remplissent la spécification de la requête.

```powershell
# Create the route for the storage endpoint.
Add-AzIotHubRoute `
   -ResourceGroupName $resourceGroup `
   -Name $iotHubName `
   -RouteName $routeName `
   -Source DeviceMessages `
   -EndpointName $endpointName `
   -Condition $condition `
   -Enabled 
```

### <a name="route-to-a-service-bus-queue"></a>Router vers une file d’attente Service Bus

Configurez maintenant le routage pour la file d’attente Service Bus. Pour récupérer la chaîne de connexion pour la file d’attente Service Bus, vous devez créer une règle d’autorisation ayant les droits appropriés. Le script suivant crée une règle d’autorisation pour la file d’attente Service Bus appelée `sbauthrule`, puis il définit les droits sur `Listen Manage Send`. Une fois que vous avez configuré cette règle d’autorisation, vous pouvez l’utiliser pour récupérer la chaîne de connexion de la file d’attente.

```powershell
##### ROUTING FOR SERVICE BUS QUEUE #####

# Create the authorization rule for the Service Bus queue.
New-AzServiceBusAuthorizationRule `
  -ResourceGroupName $resourceGroup `
  -NamespaceName $serviceBusNamespace `
  -Queue $serviceBusQueueName `
  -Name "sbauthrule" `
  -Rights @("Manage","Listen","Send")
```

Utilisez maintenant la règle d’autorisation pour récupérer la clé de la file d’attente Service Bus. Cette règle d’autorisation permettra de récupérer la chaîne de connexion plus loin dans le script.

```powershell
$sbqkey = Get-AzServiceBusKey `
    -ResourceGroupName $resourceGroup `
    -NamespaceName $serviceBusNamespace `
    -Queue $servicebusQueueName `
    -Name "sbauthrule"
```

Configurez maintenant le point de terminaison de routage et la route des messages pour la file d’attente Service Bus. Voici les variables utilisées par le script, que vous devez définir dans votre session Cloud Shell :

**endpointName** : ce champ indique le nom qui identifie le point de terminaison. 

**endpointType** : ce champ indique le type de point de terminaison. Il peut avoir la valeur `azurestoragecontainer`, `eventhub`, `servicebusqueue` ou `servicebustopic`. Ici, définissez-le sur `servicebusqueue`.

**routeName** : ce champ indique le nom de la route que vous configurez. 

**condition** : ce champ correspond à la requête utilisée pour filtrer les messages envoyés à ce point de terminaison. La condition de requête pour les messages routés vers la file d’attente Service Bus est `level="critical"`.

Voici le script Azure PowerShell configuré pour le routage des messages vers la file d’attente Service Bus.

```powershell
$endpointName = "ContosoSBQueueEndpoint"
$endpointType = "servicebusqueue"
$routeName = "ContosoSBQueueRoute"
$condition = 'level="critical"'

# Add the routing endpoint, using the connection string property from the key.
Add-AzIotHubRoutingEndpoint `
  -ResourceGroupName $resourceGroup `
  -Name $iotHubName `
  -EndpointName $endpointName `
  -EndpointType $endpointType `
  -EndpointResourceGroup $resourceGroup `
  -EndpointSubscriptionId $subscriptionId `
  -ConnectionString $sbqkey.PrimaryConnectionString

# Set up the message route for the Service Bus queue endpoint.
Add-AzIotHubRoute `
   -ResourceGroupName $resourceGroup `
   -Name $iotHubName `
   -RouteName $routeName `
   -Source DeviceMessages `
   -EndpointName $endpointName `
   -Condition $condition `
   -Enabled 
```

### <a name="view-message-routing-in-the-portal"></a>Visualiser le routage des messages dans le portail

[!INCLUDE [iot-hub-include-view-routing-in-portal](../../includes/iot-hub-include-view-routing-in-portal.md)]

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez terminé la configuration des ressources et des routes des messages, passez au tutoriel suivant pour découvrir comment envoyer des messages au hub IoT et visualiser les routes qu’ils empruntent vers les différentes destinations. 

> [!div class="nextstepaction"]
> [Partie 2 - Voir les résultats du routage des messages](tutorial-routing-view-message-routing-results.md)
