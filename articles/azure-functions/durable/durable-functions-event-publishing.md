---
title: Publication de Fonctions durables sur Azure Event Grid (version préliminaire)
description: Découvrez comment configurer la publication automatique de l’extension Fonctions durables sur Azure Event Grid.
ms.topic: conceptual
ms.date: 04/25/2020
ms.openlocfilehash: e4651dd7548ba76380bfc2d1b314e67d7abe63d9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87081744"
---
# <a name="durable-functions-publishing-to-azure-event-grid-preview"></a>Publication de Fonctions durables sur Azure Event Grid (version préliminaire)

Cet article explique comment configurer Durable Functions de façon à publier les événements de cycle de vie d’orchestration (création, achèvement, échec, etc.) sur une [Rubrique Azure Event Grid](../../event-grid/overview.md) personnalisée.

Voici certains scénarios pour lesquels cette fonctionnalité est utile :

* **Scénarios DevOps, comme les déploiements bleus/verts** : vous avez intérêt à savoir si des tâches sont en cours d’exécution avant d’implémenter la [stratégie de déploiement côte à côte](durable-functions-versioning.md#side-by-side-deployments).

* **Prise en charge du monitoring et des diagnostics avancés** : vous pouvez effectuer le suivi des données sur l’état d’orchestration dans un magasin externe optimisé pour les requêtes, comme Azure SQL Database ou Azure Cosmos DB.

* **Activité longue en arrière-plan** : si vous utilisez Durable Functions pour une activité longue en arrière-plan, cette fonctionnalité vous donne des indications sur l’état actuel.

## <a name="prerequisites"></a>Prérequis

* Installez [Microsoft.Azure.WebJobs.Extensions.DurableTask](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.DurableTask) dans votre projet Durable Functions.
* Installez l’[émulateur de Stockage Azure](../../storage/common/storage-use-emulator.md) (Windows uniquement) ou utilisez un compte de Stockage Azure existant.
* Installez [Azure CLI](/cli/azure/?view=azure-cli-latest) ou utilisez [Azure Cloud Shell](../../cloud-shell/overview.md).

## <a name="create-a-custom-event-grid-topic"></a>Créer une rubrique Event Grid personnalisée

Créez une rubrique Event Grid afin d’envoyer des événements à partir de l’extension Fonctions durables. Les instructions suivantes vous expliquent comment créer une rubrique à l’aide de l’interface Azure CLI. Pour créer la rubrique, vous pouvez également [utiliser PowerShell](../../event-grid/custom-event-quickstart-powershell.md) ou [utiliser le portail Azure](../../event-grid/custom-event-quickstart-portal.md).

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande `az group create`. Azure Event Grid ne prend pas en charge toutes les régions. Pour plus d’informations sur les régions prises en charge, consultez [Vue d’ensemble d’Azure Event Grid](../../event-grid/overview.md).

```azurecli
az group create --name eventResourceGroup --location westus2
```

### <a name="create-a-custom-topic"></a>Créer une rubrique personnalisée

Une rubrique Event Grid fournit un point de terminaison défini par l’utilisateur vers lequel vous envoyez vos événements. Remplacez `<topic_name>` par un nom unique pour votre rubrique. Le nom de la rubrique doit être unique, car il devient une entrée DNS.

```azurecli
az eventgrid topic create --name <topic_name> -l westus2 -g eventResourceGroup
```

## <a name="get-the-endpoint-and-key"></a>Obtenir le point de terminaison et la clé

Obtenez le point de terminaison de la rubrique. Remplacez `<topic_name>` par le nom de votre choix.

```azurecli
az eventgrid topic show --name <topic_name> -g eventResourceGroup --query "endpoint" --output tsv
```

Obtenez la clé de la rubrique. Remplacez `<topic_name>` par le nom de votre choix.

```azurecli
az eventgrid topic key list --name <topic_name> -g eventResourceGroup --query "key1" --output tsv
```

Vous pouvez maintenant envoyer des événements à la rubrique.

## <a name="configure-event-grid-publishing"></a>Configurer la publication Event Grid

Dans votre projet Fonctions durables, recherchez le fichier `host.json`.

### <a name="durable-functions-1x"></a>Durable Functions 1.x

Ajoutez `eventGridTopicEndpoint` et `eventGridKeySettingName` dans une propriété `durableTask`.

```json
{
  "durableTask": {
    "eventGridTopicEndpoint": "https://<topic_name>.westus2-1.eventgrid.azure.net/api/events",
    "eventGridKeySettingName": "EventGridKey"
  }
}
```

### <a name="durable-functions-2x"></a>Durable Functions 2.x

Ajoutez une section `notifications` à la propriété `durableTask` du fichier, en remplaçant `<topic_name>` par le nom de votre choix. Si les propriétés `durableTask` ou `extensions` n’existent pas, créez-les comme dans l’exemple suivant :

```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "notifications": {
        "eventGrid": {
          "topicEndpoint": "https://<topic_name>.westus2-1.eventgrid.azure.net/api/events",
          "keySettingName": "EventGridKey"
        }
      }
    }
  }
}
```

Les propriétés de configuration d’Azure Event Grid disponibles se trouvent dans la [documentation host.json](../functions-host-json.md#durabletask). Une fois le fichier `host.json` configuré, votre application de fonction envoie des événements de cycle de vie à la rubrique Event Grid. Cette action démarre quand vous exécutez votre application de fonction tant localement que dans Azure.

Définissez le paramètre d’application pour la clé de rubrique dans l’application Function App et `local.settings.json`. Le fichier JSON suivant est un exemple de l’instance `local.settings.json` pour le débogage local. Remplacez `<topic_key>` par la clé de rubrique.  

```json
{
    "IsEncrypted": false,
    "Values": {
        "AzureWebJobsStorage": "UseDevelopmentStorage=true",
        "AzureWebJobsDashboard": "UseDevelopmentStorage=true",
        "EventGridKey": "<topic_key>"
    }
}
```

Si vous utilisez l’[émulateur de stockage](../../storage/common/storage-use-emulator.md) (Windows uniquement), assurez-vous qu’il fonctionne. Il est judicieux d’exécuter la commande `AzureStorageEmulator.exe clear all` avant l’exécution.

Si vous utilisez un compte de Stockage Azure existant, remplacez `UseDevelopmentStorage=true` dans `local.settings.json` par sa chaîne de connexion.

## <a name="create-functions-that-listen-for-events"></a>Créer des fonctions qui écoutent les événements

À l’aide du portail Azure, créez une autre application de fonction pour écouter les événements publiés par votre application Durable Functions. Il est préférable de la placer dans la même région que la rubrique Event Grid.

### <a name="create-an-event-grid-trigger-function"></a>Créer une fonction de déclenchement Event Grid

1. Dans votre application de fonction, sélectionnez **Fonctions**, puis **+ Ajouter**. 

   :::image type="content" source="./media/durable-functions-event-publishing/function-add-function.png" alt-text="Ajouter une fonction dans le portail Azure." border="true":::

1. Recherchez **Event Grid**, puis sélectionnez le modèle **Déclencheur Azure Event Grid**. 

    :::image type="content" source="./media/durable-functions-event-publishing/function-select-event-grid-trigger.png" alt-text="Ajouter une fonction dans le portail Azure." border="true":::

1. Nommez le nouveau déclencheur, puis sélectionnez **Créer une fonction**.

    :::image type="content" source="./media/durable-functions-event-publishing/function-name-event-grid-trigger.png" alt-text="Ajouter une fonction dans le portail Azure." border="true":::


    Une fonction présentant le code suivant est créée :

    # <a name="c-script"></a>[Script C#](#tab/csharp-script)

    ```csharp
    #r "Newtonsoft.Json"
    using Newtonsoft.Json;
    using Newtonsoft.Json.Linq;
    using Microsoft.Extensions.Logging;

    public static void Run(JObject eventGridEvent, ILogger log)
    {
        log.LogInformation(eventGridEvent.ToString(Formatting.Indented));
    }
    ```

   # <a name="javascript"></a>[JavaScript](#tab/javascript)

   ```javascript
   module.exports = async function(context, eventGridEvent) {
       context.log(typeof eventGridEvent);
       context.log(eventGridEvent);
   }
   ```

---

### <a name="add-an-event-grid-subscription"></a>Ajouter un abonnement Event Grid

Vous pouvez à présent ajouter un abonnement Event Grid à la rubrique Event Grid que vous avez créée. Pour plus d’informations, consultez [Concepts utilisés dans Azure Event Grid](../../event-grid/concepts.md).

1. Dans votre nouvelle fonction, sélectionnez **Intégration**, puis **Déclencheur Event Grid (eventGridEvent)** . 

    :::image type="content" source="./media/durable-functions-event-publishing/eventgrid-trigger-link.png" alt-text="Ajouter une fonction dans le portail Azure." border="true":::

1. Sélectionnez **Créer une description Event Grid**.

    :::image type="content" source="./media/durable-functions-event-publishing/create-event-grid-subscription.png" alt-text="Ajouter une fonction dans le portail Azure." border="true":::

1. Nommez votre abonnement aux événements et sélectionnez le type de rubrique **Rubriques Event Grid**. 

1. Sélectionnez l’abonnement. Sélectionnez ensuite le groupe de ressources créé pour la rubrique Event Grid. 

1. Sélectionnez **Create** (Créer).

    :::image type="content" source="./media/durable-functions-event-publishing/event-grid-subscription-details.png" alt-text="Ajouter une fonction dans le portail Azure." border="true":::

Vous êtes maintenant prêt à recevoir les événements du cycle de vie.

## <a name="run-durable-functions-app-to-send-the-events"></a>Exécuter une application Durable Functions pour envoyer les événements

Dans le projet Durable Functions que vous avez configuré précédemment, commencez à déboguer sur votre ordinateur local, puis démarrez une orchestration. L’application publie des événements de cycle de vie de Durable Functions sur Event Grid. Vérifiez qu’Event Grid déclenche la fonction d’écouteur que vous avez créée en consultant ses journaux dans le portail Azure.

```
2019-04-20T09:28:21.041 [Info] Function started (Id=3301c3ef-625f-40ce-ad4c-9ba2916b162d)
2019-04-20T09:28:21.104 [Info] {
    "id": "054fe385-c017-4ce3-b38a-052ac970c39d",
    "subject": "durable/orchestrator/Running",
    "data": {
        "hubName": "DurableFunctionsHub",
        "functionName": "Sample",
        "instanceId": "055d045b1c8a415b94f7671d8df693a6",
        "reason": "",
        "runtimeStatus": "Running"
    },
    "eventType": "orchestratorEvent",
    "eventTime": "2019-04-20T09:28:19.6492068Z",
    "dataVersion": "1.0",
    "metadataVersion": "1",
    "topic": "/subscriptions/<your_subscription_id>/resourceGroups/eventResourceGroup/providers/Microsoft.EventGrid/topics/durableTopic"
}

2019-04-20T09:28:21.104 [Info] Function completed (Success, Id=3301c3ef-625f-40ce-ad4c-9ba2916b162d, Duration=65ms)
2019-04-20T09:28:37.098 [Info] Function started (Id=36fadea5-198b-4345-bb8e-2837febb89a2)
2019-04-20T09:28:37.098 [Info] {
    "id": "8cf17246-fa9c-4dad-b32a-5a868104f17b",
    "subject": "durable/orchestrator/Completed",
    "data": {
        "hubName": "DurableFunctionsHub",
        "functionName": "Sample",
        "instanceId": "055d045b1c8a415b94f7671d8df693a6",
        "reason": "",
        "runtimeStatus": "Completed"
    },
    "eventType": "orchestratorEvent",
    "eventTime": "2019-04-20T09:28:36.5061317Z",
    "dataVersion": "1.0",
    "metadataVersion": "1",
    "topic": "/subscriptions/<your_subscription_id>/resourceGroups/eventResourceGroup/providers/Microsoft.EventGrid/topics/durableTopic"
}
2019-04-20T09:28:37.098 [Info] Function completed (Success, Id=36fadea5-198b-4345-bb8e-2837febb89a2, Duration=0ms)
```

## <a name="event-schema"></a>Schéma d’événement

La liste suivante explique le schéma des événements du cycle de vie :

* **`id`**  : identificateur unique de l’événement Event Grid.
* **`subject`**  : chemin d’accès de l’objet de l’événement. `durable/orchestrator/{orchestrationRuntimeStatus}`. `{orchestrationRuntimeStatus}` sera `Running`, `Completed`, `Failed` et `Terminated`.  
* **`data`**  : paramètres propres à Durable Functions.
  * **`hubName`**  : nom [TaskHub](durable-functions-task-hubs.md).
  * **`functionName`**  : nom de la fonction d’orchestrateur.
  * **`instanceId`**  : identificateur de l’instance Durable Functions.
  * **`reason`**  : données supplémentaires associées à l’événement de suivi. Pour en savoir plus, consultez la section [Diagnostics dans Fonctions durables (Azure Functions)](durable-functions-diagnostics.md).
  * **`runtimeStatus`**  : état du runtime d’orchestration. Running, Completed, Failed, Canceled. (En cours d’exécution, Terminé, En échec ou Annulé)
* **`eventType`**  : "orchestratorEvent"
* **`eventTime`**  : heure de l’événement (UTC).
* **`dataVersion`**  : version du schéma d’événement du cycle de vie.
* **`metadataVersion`**  :  version des métadonnées.
* **`topic`**  : ressource de rubrique Event Grid.

## <a name="how-to-test-locally"></a>Procédure de test local

Pour tester localement, consultez [Débogage local des déclencheurs Event Grid des fonctions Azure](../functions-debug-event-grid-trigger-local.md).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur la gestion des instances dans Fonctions durables](durable-functions-instance-management.md)

> [!div class="nextstepaction"]
> [En savoir plus sur le contrôle de version dans Fonctions durables](durable-functions-versioning.md)
