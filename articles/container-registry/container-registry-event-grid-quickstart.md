---
title: Démarrage rapide - Envoyer des événements à Event Grid
description: Dans ce guide de démarrage rapide, vous activez les événements Event Grid pour votre registre de conteneurs, puis vous envoyez des événements push et delete pour l’image de conteneur à un exemple d’application.
ms.topic: article
ms.date: 08/23/2018
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 2d13dd0ec5e50086e674b215d93917d6173d5af9
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97694392"
---
# <a name="quickstart-send-events-from-private-container-registry-to-event-grid"></a>Démarrage rapide : Envoyer des événements depuis le registre de conteneurs privé à Event Grid

Azure Event Grid est un service de routage d’événements entièrement géré qui permet une consommation d’événements uniforme avec un modèle publication-abonnement. Dans ce guide de démarrage rapide, vous utilisez l’interface CLI Azure pour créer un registre de conteneurs, pour vous abonner aux événements du registre, puis pour déployer un exemple d’application web pour recevoir les événements. Enfin, vous déclenchez des événements `push` et `delete` pour l’image de conteneur et vous visualisez la charge utile des événements dans l’exemple d’application.

Après avoir terminé les étapes décrites dans cet article, les événements envoyés à partir de votre registre de conteneurs à Event Grid apparaissent dans l’exemple d’application web :

![Navigateur web affichant l’exemple d’application web avec trois événements reçus][sample-app-01]

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Les commandes CLI de cet article sont mises en forme pour le shell **Bash**. Si vous utilisez un autre shell, comme PowerShell ou l’invite de commandes, il peut être nécessaire d’ajuster en conséquence les caractères de continuation de ligne ou les lignes d’affectation des variables. Cet article utilise des variables pour réduire le nombre de modifications nécessaires des commandes.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources Azure est un conteneur logique dans lequel vous déployez et vous gérez vos ressources Azure. La commande [az group create][az-group-create] suivante crée un groupe de ressources nommé *myResourceGroup* dans la région *eastus*. Si vous voulez utiliser un autre nom pour votre groupe de ressources, définissez `RESOURCE_GROUP_NAME` sur une autre valeur.

```azurecli-interactive
RESOURCE_GROUP_NAME=myResourceGroup

az group create --name $RESOURCE_GROUP_NAME --location eastus
```

## <a name="create-a-container-registry"></a>Créer un registre de conteneurs

Ensuite, déployez un registre de conteneurs dans le groupe de ressources avec les commandes suivantes. Avant d’exécuter la commande [az acr create][az-acr-create], définissez `ACR_NAME` avec un nom pour votre registre. Le nom du registre doit être unique dans Azure, il doit comporter entre 5 et 50 caractères alphanumériques.

```azurecli-interactive
ACR_NAME=<acrName>

az acr create --resource-group $RESOURCE_GROUP_NAME --name $ACR_NAME --sku Basic
```

Une fois le registre créé, Azure CLI retourne une sortie similaire à ceci :

```json
{
  "adminUserEnabled": false,
  "creationDate": "2018-08-16T20:02:46.569509+00:00",
  "id": "/subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/myregistry",
  "location": "eastus",
  "loginServer": "myregistry.azurecr.io",
  "name": "myregistry",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
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

Vous devez voir l’exemple d’application sans aucun message d’événement affiché :

![Navigateur montrant l’exemple d’application web sans aucun message d’événement affiché][sample-app-02]

[!INCLUDE [event-grid-register-provider-cli.md](../../includes/event-grid-register-provider-cli.md)]

## <a name="subscribe-to-registry-events"></a>S’abonner aux événements du registre

Dans Event Grid, vous vous abonnez à un *sujet* pour lui indiquer les événements qui vous intéressent, et où les envoyer. La commande [az eventgrid event-subscription create][az-eventgrid-event-subscription-create] suivante s’abonne au registre de conteneurs que vous avez créé et spécifie l’URL de votre application web comme point de terminaison auquel il doit envoyer les événements. Les variables d’environnement que vous avez renseignées dans les sections précédentes sont réutilisées ici : aucune modification n’est donc nécessaire.

```azurecli-interactive
ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query id --output tsv)
APP_ENDPOINT=https://$SITE_NAME.azurewebsites.net/api/updates

az eventgrid event-subscription create \
    --name event-sub-acr \
    --source-resource-id $ACR_REGISTRY_ID \
    --endpoint $APP_ENDPOINT
```

Une fois l’abonnement souscrit, vous devez voir un résultat similaire à celui-ci :

```json
{
  "destination": {
    "endpointBaseUrl": "https://eventgridviewer.azurewebsites.net/api/updates",
    "endpointType": "WebHook",
    "endpointUrl": null
  },
  "filter": {
    "includedEventTypes": [
      "All"
    ],
    "isSubjectCaseSensitive": null,
    "subjectBeginsWith": "",
    "subjectEndsWith": ""
  },
  "id": "/subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/myregistry/providers/Microsoft.EventGrid/eventSubscriptions/event-sub-acr",
  "labels": null,
  "name": "event-sub-acr",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "topic": "/subscriptions/<Subscription ID>/resourceGroups/myresourcegroup/providers/microsoft.containerregistry/registries/myregistry",
  "type": "Microsoft.EventGrid/eventSubscriptions"
}
```

## <a name="trigger-registry-events"></a>Déclencher des événements du registre

Maintenant que l’exemple d’application est opérationnel et en cours d’exécution, et que vous êtes abonné à votre registre auprès d’Event Grid, vous êtes prêt à générer des événements. Dans cette section, vous utilisez ACR Tasks pour générer et envoyer (push) une image de conteneur à votre registre. ACR Tasks est une fonctionnalité d’Azure Container Registry qui vous permet de créer des images de conteneur dans le cloud, sans devoir installer le moteur Docker sur votre machine locale.

### <a name="build-and-push-image"></a>Générer et envoyer (push) une image

Exécutez la commande Azure CLI suivante pour générer une image de conteneur à partir du contenu d’un dépôt GitHub. Par défaut, ACR Tasks envoie (push) automatiquement une image générée avec succès à votre registre, ce qui génère l’événement `ImagePushed`.

```azurecli-interactive
az acr build --registry $ACR_NAME --image myimage:v1 -f Dockerfile https://github.com/Azure-Samples/acr-build-helloworld-node.git
```

Vous devez voir une sortie similaire à ce qui suit quand ACR Tasks génère, puis envoie (push) votre image. L’exemple de sortie suivant a été tronqué par souci de concision.

```output
Sending build context to ACR...
Queued a build with build ID: aa2
Waiting for build agent...
2018/08/16 22:19:38 Using acb_vol_27a2afa6-27dc-4ae4-9e52-6d6c8b7455b2 as the home volume
2018/08/16 22:19:38 Setting up Docker configuration...
2018/08/16 22:19:39 Successfully set up Docker configuration
2018/08/16 22:19:39 Logging in to registry: myregistry.azurecr.io
2018/08/16 22:19:55 Successfully logged in
Sending build context to Docker daemon  94.72kB
Step 1/5 : FROM node:9-alpine
...
```

Pour vérifier que l’image générée est dans votre registre, exécutez la commande suivante pour voir les étiquettes dans le dépôt « myimage » :

```azurecli-interactive
az acr repository show-tags --name $ACR_NAME --repository myimage
```

L’étiquette « v1 » de l’image que vous avez générée doit apparaître dans la sortie et être similaire à ceci :

```output
[
  "v1"
]
```

### <a name="delete-the-image"></a>Supprimer l’image

À présent, générez un événement `ImageDeleted` en supprimant l’image avec la commande [az acr repository delete][az-acr-repository-delete] :

```azurecli-interactive
az acr repository delete --name $ACR_NAME --image myimage:v1
```

Vous devez voir une sortie similaire à ce qui suit, qui demande une confirmation pour supprimer le manifeste et les images associées :

```output
This operation will delete the manifest 'sha256:f15fa9d0a69081ba93eee308b0e475a54fac9c682196721e294b2bc20ab23a1b' and all the following images: 'myimage:v1'.
Are you sure you want to continue? (y/n): 
```

## <a name="view-registry-events"></a>Afficher des événements du registre

Vous avez maintenant envoyé (push) une image à votre registre, puis vous l’avez supprimée. Accédez à votre application web de visionneuse Event Grid : vous devez voir les événements `ImageDeleted` et `ImagePushed`. Vous pouvez également voir un événement de validation d’abonnement généré par l’exécution de la commande de la section [S’abonner aux événements du registre](#subscribe-to-registry-events).

La capture d’écran suivante montre l’exemple d’application avec les trois événements et l’événement `ImageDeleted` développé pour afficher ses détails.

![Navigateur web montrant l’exemple d’application avec des événements ImagePushed et ImageDeleted][sample-app-03]

Félicitations ! Si vous voyez les événements `ImagePushed` et `ImageDeleted`, c’est que votre registre envoie des événements à Event Grid, qui à son tour transfère ces événements à votre point de terminaison d’application web.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Une fois que vous en avez terminé avec les ressources créées dans ce guide de démarrage rapide, vous pouvez les supprimer toutes avec la commande Azure CLI suivante. Quand vous supprimez un groupe de ressources, toutes les ressources qu’il contient sont supprimées de façon définitive.

**AVERTISSEMENT** : cette action est irréversible. Vérifiez bien que vous n’avez plus besoin des ressources du groupe avant d’exécuter la commande.

```azurecli-interactive
az group delete --name $RESOURCE_GROUP_NAME
```

## <a name="event-grid-event-schema"></a>Schéma d’événement Event Grid

Vous pouvez trouver les informations de référence sur le schéma des messages d’événement d’Azure Container Registry dans la documentation d’Event Grid :

[Schéma d’événement Azure Event Grid pour Container Registry](../event-grid/event-schema-container-registry.md)

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez déployé un registre de conteneurs, vous avez généré une image avec ACR Tasks, vous l’avez supprimée et vous avez utilisé les événements de votre registre provenant d’Event Grid avec un exemple d’application. Passez ensuite au tutoriel ACR Tasks pour en savoir plus sur la génération d’images de conteneur dans le cloud, notamment les générations automatisées sur la mise à jour de l’image de base :

> [!div class="nextstepaction"]
> [Générer des images de conteneur dans le cloud avec ACR Tasks](container-registry-tutorial-quick-task.md)

<!-- IMAGES -->
[sample-app-01]: ./media/container-registry-event-grid-quickstart/sample-app-01.png
[sample-app-02]: ./media/container-registry-event-grid-quickstart/sample-app-02-no-events.png
[sample-app-03]: ./media/container-registry-event-grid-quickstart/sample-app-03-with-events.png

<!-- LINKS - External -->
[azure-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[sample-app]: https://github.com/dbarkol/azure-event-grid-viewer

<!-- LINKS - Internal -->
[az-acr-create]: /cli/azure/acr/repository
[az-acr-repository-delete]: /cli/azure/acr/repository#az-acr-repository-delete
[az-eventgrid-event-subscription-create]: /cli/azure/eventgrid/event-subscription#az-eventgrid-event-subscription-create
[az-group-create]: /cli/azure/group#az-group-create
