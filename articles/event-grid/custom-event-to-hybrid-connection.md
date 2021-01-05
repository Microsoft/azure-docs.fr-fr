---
title: 'Tutoriel : Envoyer des événements personnalisés vers une connexion hybride - Event Grid'
description: 'Tutoriel : Utilisez Azure Event Grid et Azure CLI pour publier une rubrique et pour vous abonner à cet événement. Une connexion hybride est utilisée pour le point de terminaison.'
ms.date: 07/07/2020
ms.topic: tutorial
ms.custom: devx-track-azurecli
ms.openlocfilehash: c709d77827551860cc917c3c84c5a849d9fdc512
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94566824"
---
# <a name="tutorial-route-custom-events-to-azure-relay-hybrid-connections-with-azure-cli-and-event-grid"></a>Tutoriel : Acheminer des événements personnalisés vers des connexions hybrides Azure Relay avec Azure CLI et Event Grid

Azure Event Grid est un service de gestion d’événements pour le cloud. Les connexions hybrides Azure Relay sont l’un des gestionnaires d’événements pris en charge. Vous utilisez des connexions hybrides comme gestionnaires d’événements lorsque vous devez traiter des événements à partir d’applications qui ne possèdent pas de point de terminaison public. Ces applications peuvent se trouver dans votre réseau d’entreprise. Dans cet article, vous utilisez l’interface de ligne de commande Azure pour créer une rubrique personnalisée, vous abonner à cette rubrique personnalisée et déclencher l’événement pour afficher le résultat. Vous envoyez les événements à la connexion hybride.

## <a name="prerequisites"></a>Prérequis

- Cet article suppose que vous disposez déjà d’une connexion hybride et d’une application d’écouteur. Pour aborder les connexions hybrides, consultez la section [Prise en main des connexions hybrides Relay : .NET](../azure-relay/relay-hybrid-connections-dotnet-get-started.md) ou [Prise en main des connexions hybrides Relay : nœud](../azure-relay/relay-hybrid-connections-node-get-started.md).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Cet article nécessite la version 2.0.56 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Les rubriques Event Grid sont des ressources Azure et doivent être placées dans un groupe de ressources Azure. Un groupe de ressources est une collection logique dans laquelle des ressources Azure sont déployées et gérées.

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group#az-group-create). 

L’exemple suivant crée un groupe de ressources nommé *gridResourceGroup* à l’emplacement *westus2*.

```azurecli-interactive
az group create --name gridResourceGroup --location westus2
```

## <a name="create-a-custom-topic"></a>Créer une rubrique personnalisée

Une rubrique de grille d’événement fournit un point de terminaison défini par l’utilisateur vers lequel vous envoyez vos événements. L’exemple suivant permet de créer la rubrique personnalisée dans votre groupe de ressources. Remplacez `<topic_name>` par un nom unique pour votre rubrique personnalisée. Le nom de la rubrique de la grille d’événements doit être unique, car elle est représentée par une entrée DNS.

```azurecli-interactive
az eventgrid topic create --name <topic_name> -l westus2 -g gridResourceGroup
```

## <a name="subscribe-to-a-custom-topic"></a>S’abonner à une rubrique personnalisée

Vous vous abonnez à une rubrique de la grille d’événements pour communiquer à Event Grid les événements qui vous intéressent. L’exemple suivant vous abonne à la rubrique personnalisée que vous avez créée et transmet l’ID de ressource de la connexion hybride au point de terminaison. L’ID de connexion hybride est au format :

`/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Relay/namespaces/<relay-namespace>/hybridConnections/<hybrid-connection-name>`

Le script suivant obtient l’ID de ressource de l’espace de nom Relay. Il construit l’identifiant de la connexion hybride, puis s’abonne à une rubrique Event Grid. Le script définit le type de point de terminaison sur `hybridconnection` et utilise l’ID de connexion hybride pour le point de terminaison.

```azurecli-interactive
relayname=<namespace-name>
relayrg=<resource-group-for-relay>
hybridname=<hybrid-name>

relayid=$(az resource show --name $relayname --resource-group $relayrg --resource-type Microsoft.Relay/namespaces --query id --output tsv)
hybridid="$relayid/hybridConnections/$hybridname"
topicid=$(az eventgrid topic show --name <topic_name> -g gridResourceGroup --query id --output tsv)

az eventgrid event-subscription create \
  --source-resource-id $topicid \
  --name <event_subscription_name> \
  --endpoint-type hybridconnection \
  --endpoint $hybridid \
  --expiration-date "<yyyy-mm-dd>"
```

Notez qu’une [date d’expiration](concepts.md#event-subscription-expiration) est définie pour l’abonnement.

## <a name="create-application-to-process-events"></a>Créer l’application de traitement des événements

Vous avez besoin d’une application qui peut récupérer des événements à partir de la connexion hybride. L’[exemple de consommateur de connexions hybrides Microsoft Azure Event Grid pour C#](https://github.com/Azure-Samples/event-grid-dotnet-hybridconnection-destination) effectue cette opération. Vous avez déjà terminé les étapes requises.

1. Vérifiez que vous disposez de Visual Studio 2019 ou version ultérieure.

1. Clonez le référentiel sur votre ordinateur local.

1. Chargez le projet HybridConnectionConsumer dans Visual Studio.

1. Dans Program.cs, remplacez `<relayConnectionString>` et `<hybridConnectionName>` par la chaîne de connexion et le nom de la connexion hybride créés.

1. Compilez et exécutez l’application à partir de Visual Studio.

## <a name="send-an-event-to-your-topic"></a>Envoyer un événement à votre rubrique

Nous allons maintenant déclencher un événement pour voir comment Event Grid distribue le message à votre point de terminaison. Cet article montre comment utiliser Azure CLI pour déclencher l’événement. Vous pouvez également utiliser l’[application de serveur de publication Event Grid](https://github.com/Azure-Samples/event-grid-dotnet-publish-consume-events/tree/master/EventGridPublisher).

Tout d’abord, il nous faut l’URL et la clé de la rubrique personnalisée. Utilisez de nouveau le nom de votre rubrique personnalisée pour `<topic_name>`.

```azurecli-interactive
endpoint=$(az eventgrid topic show --name <topic_name> -g gridResourceGroup --query "endpoint" --output tsv)
key=$(az eventgrid topic key list --name <topic_name> -g gridResourceGroup --query "key1" --output tsv)
```

Pour simplifier cet article, vous allez utiliser des exemples de données d’événements à envoyer à la rubrique personnalisée. En règle générale, une application ou un service Azure envoie les données d’événements. CURL est un utilitaire qui envoie des requêtes HTTP. Dans cet article, utilisez CURL pour envoyer l’événement à la rubrique personnalisée.

```azurecli-interactive
event='[ {"id": "'"$RANDOM"'", "eventType": "recordInserted", "subject": "myapp/vehicles/motorcycles", "eventTime": "'`date +%Y-%m-%dT%H:%M:%S%z`'", "data":{ "make": "Ducati", "model": "Monster"},"dataVersion": "1.0"} ]'
curl -X POST -H "aeg-sas-key: $key" -d "$event" $endpoint
```

Votre application d’écouteur doit recevoir le message d’événement.

## <a name="clean-up-resources"></a>Nettoyer les ressources
Si vous envisagez de continuer à utiliser cet événement, ne supprimez pas les ressources créées dans cet article. Sinon, utilisez la commande suivante pour supprimer les ressources créées dans cet article.

```azurecli-interactive
az group delete --name gridResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez créer des rubriques et des abonnements d’événements, vous pouvez en apprendre davantage sur Event Grid et ce qu’il peut vous offrir :

- [À propos d’Event Grid](overview.md)
- [Acheminer des événements de stockage Blob Azure vers un point de terminaison Web personnalisé ](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json)
- [Surveiller les modifications d’une machine virtuelle avec Azure Event Grid et Azure Logic Apps](monitor-virtual-machine-changes-event-grid-logic-app.md)
- [Diffuser en continu des Big Data dans un entrepôt de données](event-grid-event-hubs-integration.md)