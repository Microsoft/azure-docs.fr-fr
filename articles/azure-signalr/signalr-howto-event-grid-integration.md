---
title: Comment envoyer des événements Azure SignalR Service à Event Grid
description: Un guide pour vous montrer comment activer les événements Event Grid pour votre SignalR Service, puis envoyer les événements de connexion/déconnexion client à exemple d’application.
services: signalr
author: chenyl
ms.service: signalr
ms.topic: conceptual
ms.date: 11/13/2019
ms.author: chenyl
ms.openlocfilehash: 84b83c1dd541418c446a89a6f51be668cb41e54e
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94562642"
---
# <a name="how-to-send-events-from-azure-signalr-service-to-event-grid"></a>Comment envoyer des événements d’Azure SignalR Service à Event Grid

Azure Event Grid est un service de routage d’événements entièrement géré qui permet une consommation d’événements uniforme avec un modèle publication-abonnement. Dans ce guide, vous utilisez l’interface Azure CLI pour créer un Azure SignalR Service, vous abonner aux événements de connexion, puis déployer un exemple d’application web pour recevoir les événements. Enfin, vous pouvez vous connecter et déconnecter et voir la charge utile d’événement dans l’exemple d’application.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Les commandes CLI de cet article sont mises en forme pour le shell **Bash**. Si vous utilisez un autre shell, comme PowerShell ou l’invite de commandes, il peut être nécessaire d’ajuster en conséquence les caractères de continuation de ligne ou les lignes d’affectation des variables. Cet article utilise des variables pour réduire le nombre de modifications nécessaires des commandes.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources Azure est un conteneur logique dans lequel vous déployez et vous gérez vos ressources Azure. La commande [az group create][az-group-create] suivante crée un groupe de ressources nommé *myResourceGroup* dans la région *eastus*. Si vous voulez utiliser un autre nom pour votre groupe de ressources, définissez `RESOURCE_GROUP_NAME` sur une autre valeur.

```azurecli-interactive
RESOURCE_GROUP_NAME=myResourceGroup

az group create --name $RESOURCE_GROUP_NAME --location eastus
```

## <a name="create-a-signalr-service"></a>Créer une ressource SignalR Service

Ensuite, déployez un Azure SignalR Service dans le groupe de ressources avec les commandes suivantes.
```azurecli-interactive
SIGNALR_NAME=SignalRTestSvc

az signalr create --resource-group $RESOURCE_GROUP_NAME --name $SIGNALR_NAME --sku Free_F1
```

Une fois le SignalR Service créé, Azure CLI retourne une sortie similaire à ceci :

```json
{
  "externalIp": "13.76.156.152",
  "hostName": "clitest.servicedev.signalr.net",
  "hostNamePrefix": "clitest",
  "id": "/subscriptions/28cf13e2-c598-4aa9-b8c8-098441f0827a/resourceGroups/clitest1/providers/Microsoft.SignalRService/SignalR/clitest",
  "location": "southeastasia",
  "name": "clitest",
  "provisioningState": "Succeeded",
  "publicPort": 443,
  "resourceGroup": "clitest1",
  "serverPort": 443,
  "sku": {
    "capacity": 1,
    "family": null,
    "name": "Free_F1",
    "size": "F1",
    "tier": "Free"
  },
  "tags": null,
  "type": "Microsoft.SignalRService/SignalR",
  "version": "1.0"
}

```

## <a name="create-an-event-endpoint"></a>Créer un point de terminaison d’événements

Dans cette section, vous utilisez un modèle Resource Manager qui se trouve dans un dépôt GitHub pour déployer un exemple d’application web prédéfinie sur Azure App Service. Plus tard, vous vous abonnez aux événements Event Grid de votre registre et vous spécifiez cette application comme point de terminaison auquel les événements sont envoyés.

Pour déployer l’exemple d’application, définissez `SITE_NAME` avec un nom unique pour votre application web et exécutez les commandes suivantes. Le nom du site doit être unique au sein d’Azure, car il fait partie du nom de domaine complet (FQDN) de l’application web. Dans une section ultérieure, vous accédez au nom de domaine complet de l’application dans un navigateur web pour visualiser les événements de votre registre.

```azurecli-interactive
SITE_NAME=<your-site-name>

az deployment group create \
    --resource-group $RESOURCE_GROUP_NAME \
    --template-uri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/master/azuredeploy.json" \
    --parameters siteName=$SITE_NAME hostingPlanName=$SITE_NAME-plan
```

Une fois que le déploiement a réussi (il peut prendre quelques minutes), ouvrez un navigateur et accédez à votre application web pour vérifier qu’elle est en cours d’exécution :

`http://<your-site-name>.azurewebsites.net`

[!INCLUDE [event-grid-register-provider-cli.md](../../includes/event-grid-register-provider-cli.md)]

## <a name="subscribe-to-registry-events"></a>S’abonner aux événements du registre

Dans Event Grid, vous vous abonnez à un *sujet* pour lui indiquer les événements qui vous intéressent, et où les envoyer. La commande [az eventgrid event-subscription create][az-eventgrid-event-subscription-create] suivante s’abonne à l’instance Azure SignalR Service que vous avez créée et spécifie l’URL de votre application web comme point de terminaison auquel il doit envoyer les événements. Les variables d’environnement que vous avez renseignées dans les sections précédentes sont réutilisées ici : aucune modification n’est donc nécessaire.

```azurecli-interactive
SIGNALR_SERVICE_ID=$(az signalr show --resource-group $RESOURCE_GROUP_NAME --name $SIGNALR_NAME --query id --output tsv)
APP_ENDPOINT=https://$SITE_NAME.azurewebsites.net/api/updates

az eventgrid event-subscription create \
    --name event-sub-signalr \
    --source-resource-id $SIGNALR_SERVICE_ID \
    --endpoint $APP_ENDPOINT
```

Une fois l’abonnement souscrit, vous devez voir un résultat similaire à celui-ci :

```JSON
{
  "deadLetterDestination": null,
  "destination": {
    "endpointBaseUrl": "https://$SITE_NAME.azurewebsites.net/api/updates",
    "endpointType": "WebHook",
    "endpointUrl": null
  },
  "filter": {
    "includedEventTypes": [
      "Microsoft.SignalRService.ClientConnectionConnected",
      "Microsoft.SignalRService.ClientConnectionDisconnected"
    ],
    "isSubjectCaseSensitive": null,
    "subjectBeginsWith": "",
    "subjectEndsWith": ""
  },
  "id": "/subscriptions/28cf13e2-c598-4aa9-b8c8-098441f0827a/resourceGroups/myResourceGroup/providers/Microsoft.SignalRService/SignalR/SignalRTestSvc/providers/Microsoft.EventGrid/eventSubscriptions/event-sub-signalr",
  "labels": null,
  "name": "event-sub-signalr",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "retryPolicy": {
    "eventTimeToLiveInMinutes": 1440,
    "maxDeliveryAttempts": 30
  },
  "topic": "/subscriptions/28cf13e2-c598-4aa9-b8c8-098441f0827a/resourceGroups/myResourceGroup/providers/microsoft.signalrservice/signalr/SignalRTestSvc",
  "type": "Microsoft.EventGrid/eventSubscriptions"
}
```

## <a name="trigger-registry-events"></a>Déclencher des événements du registre

Basculez le mode de service sur `Serverless Mode` et configurez une connexion client au SignalR Service. Vous pouvez prendre [l’exemple sans serveur](https://github.com/aspnet/AzureSignalR-samples/tree/master/samples/Serverless) comme référence.

```bash
git clone git@github.com:aspnet/AzureSignalR-samples.git

cd samples/Management

# Start the negotiation server
# Negotiation server is responsible for generating access token for clients
cd NegotitationServer
dotnet user-secrets set Azure:SignalR:ConnectionString "<Connection String>"
dotnet run

# Use a separate command line
# Start a client
cd SignalRClient
dotnet run
```

## <a name="view-registry-events"></a>Afficher des événements du registre

Vous avez désormais connecté un client au SignalR Service. Accédez à votre application web de visionneuse Event Grid : vous devriez voir un événement `ClientConnectionConnected`. Si vous arrêtez le client, vous verrez également un événement `ClientConnectionDisconnected`.

<!-- LINKS - External -->
[azure-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[sample-app]: https://github.com/dbarkol/azure-event-grid-viewer

<!-- LINKS - Internal -->
[az-eventgrid-event-subscription-create]: /cli/azure/eventgrid/event-subscription#az-eventgrid-event-subscription-create
[az-group-create]: /cli/azure/group#az-group-create
