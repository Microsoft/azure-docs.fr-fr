---
title: Recevoir des événements d’Azure Event Grid sur un point de terminaison HTTP
description: Décrit comment valider un point de terminaison HTTP, puis recevoir et désérialiser des événements d’Azure Event Grid
ms.topic: conceptual
ms.date: 11/19/2020
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 75c80fb85d39298f1130537971bc700897c039d0
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "96023711"
---
# <a name="receive-events-to-an-http-endpoint"></a>Recevoir des événements sur un point de terminaison HTTP

Cet article décrit comment [valider un point de terminaison HTTP](webhook-event-delivery.md) pour recevoir des événements à partir d’un abonnement à des événements, puis désérialiser des événements. Cet article utilise une fonction d’Azure à des fins de démonstration, mais les mêmes concepts s’appliquent quel que soit l’emplacement où l’application est hébergée.

> [!NOTE]
> Il est **fortement** recommandé d’utiliser un [déclencheur Event Grid](../azure-functions/functions-bindings-event-grid.md) lors du déclenchement d’une fonction d’Azure avec Event Grid. L’utilisation d’un déclencheur Webhook générique ici est démonstrative.

## <a name="prerequisites"></a>Prérequis

Vous avez besoin d’une application de fonction avec une fonction déclenchée via HTTP.

## <a name="add-dependencies"></a>Ajout de dépendances

Si vous développez dans .NET, [ajoutez une dépendance](../azure-functions/functions-reference-csharp.md#referencing-custom-assemblies) à votre fonction pour le [package NuGet](https://www.nuget.org/packages/Microsoft.Azure.EventGrid) `Microsoft.Azure.EventGrid`. Les exemples de cet article nécessitent la version 1.4.0 ou ultérieure.

Des Kits de développement logiciel (SDK) pour d’autres langues sont disponibles via la référence [Kits SDK de publication](./sdk-overview.md#data-plane-sdks). Ces packages incluent les modèles des types d’événement natif comme `EventGridEvent`, `StorageBlobCreatedEventData` et `EventHubCaptureFileCreatedEventData`.

Cliquez sur le lien « Afficher les fichiers » dans votre fonction Azure (volet le plus à droite dans le portail des fonctions Azure) et créez un fichier appelé project.json. Ajoutez le code suivant au fichier `project.json` et enregistrez-le :

 ```json
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "Microsoft.Azure.EventGrid": "2.0.0"
      }
    }
   }
}
```

![Package NuGet ajouté](./media/receive-events/add-dependencies.png)

## <a name="endpoint-validation"></a>Validation de point de terminaison

La première chose à faire est de traiter les événements `Microsoft.EventGrid.SubscriptionValidationEvent`. Chaque fois que quelqu’un s’abonne à un événement, Event Grid envoie un événement de validation au point de terminaison avec un `validationCode` dans la charge utile de données. Le point de terminaison est nécessaire pour reproduire cela dans le corps de réponse pour [prouver que le point de terminaison est valide et qu’il vous appartient](webhook-event-delivery.md). Si vous utilisez un [déclencheur Event Grid](../azure-functions/functions-bindings-event-grid.md) au lieu d’une fonction déclenchée par un Webhook, la validation du point de terminaison est traitée pour vous. Si vous utilisez un service d’API tiers (comme [Zapier](https://zapier.com/home) ou [IFTTT](https://ifttt.com/)), vous risquez de ne pas pouvoir envoyer le code de validation par programmation. Pour ces services, vous pouvez manuellement valider l’abonnement à l’aide d’une URL de validation qui est envoyée dans l’événement de validation de l’abonnement. Copiez cette URL dans la propriété `validationUrl` et envoyez une demande GET par le biais d’un client REST ou de votre navigateur web.

Dans C#, la fonction `DeserializeEventGridEvents()` désérialise les événements Event Grid. Elle désérialise les données d’événement dans le type approprié, tel que StorageBlobCreatedEventData. Utilisez la classe `Microsoft.Azure.EventGrid.EventTypes` pour obtenir les types et noms d’événements pris en charge.

Pour envoyer le code de validation par programmation, utilisez le code suivant. Vous trouverez des exemples associés dans l’[exemple de consommateur Event Grid](https://github.com/Azure-Samples/event-grid-dotnet-publish-consume-events/tree/master/EventGridConsumer).

```csharp
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Azure.EventGrid;
namespace Function1
{
    public static class Function1
    {
        [FunctionName("Function1")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");
            string response = string.Empty;
            string requestContent = await new StreamReader(req.Body).ReadToEndAsync();
            log.LogInformation($"Received events: {requestContent}");

            EventGridSubscriber eventGridSubscriber = new EventGridSubscriber();

            EventGridEvent[] eventGridEvents = eventGridSubscriber.DeserializeEventGridEvents(requestContent);

            foreach (EventGridEvent eventGridEvent in eventGridEvents)
            {
                if (eventGridEvent.Data is SubscriptionValidationEventData)
                {
                    var eventData = (SubscriptionValidationEventData)eventGridEvent.Data;
                    log.LogInformation($"Got SubscriptionValidation event data, validation code: {eventData.ValidationCode}, topic: {eventGridEvent.Topic}");
                    // Do any additional validation (as required) and then return back the below response

                    var responseData = new SubscriptionValidationResponse()
                    {
                        ValidationResponse = eventData.ValidationCode
                    };
                    return new OkObjectResult(responseData);
                }
            }
            return new OkObjectResult(response);
        }
   }
}
```

```javascript
module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function begun');
    var validationEventType = "Microsoft.EventGrid.SubscriptionValidationEvent";

    for (var events in req.body) {
        var body = req.body[events];
        // Deserialize the event data into the appropriate type based on event type
        if (body.data && body.eventType == validationEventType) {
            context.log("Got SubscriptionValidation event data, validation code: " + body.data.validationCode + " topic: " + body.topic);

            // Do any additional validation (as required) and then return back the below response
            var code = body.data.validationCode;
            context.res = { status: 200, body: { "ValidationResponse": code } };
        }
    }
    context.done();
};
```

### <a name="test-validation-response"></a>Tester la réponse de validation

Testez la fonction de réponse de validation en collant l’exemple d’événement dans le champ de test pour la fonction :

```json
[{
  "id": "2d1781af-3a4c-4d7c-bd0c-e34b19da4e66",
  "topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "subject": "",
  "data": {
    "validationCode": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6"
  },
  "eventType": "Microsoft.EventGrid.SubscriptionValidationEvent",
  "eventTime": "2018-01-25T22:12:19.4556811Z",
  "metadataVersion": "1",
  "dataVersion": "1"
}]
```

Lorsque vous cliquez sur Exécuter, la sortie doit être 200 OK et `{"validationResponse":"512d38b6-c7b8-40c8-89fe-f46f9e9622b6"}` dans le corps :

:::image type="content" source="./media/receive-events/validation-request.png" alt-text="Requête de validation":::

:::image type="content" source="./media/receive-events/validation-output.png" alt-text="Sortie de validation":::

## <a name="handle-blob-storage-events"></a>Traiter les événements de stockage Blob

À présent, étendons la fonction pour traiter `Microsoft.Storage.BlobCreated` :

```cs
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Microsoft.Azure.EventGrid;
using Microsoft.Azure.EventGrid.Models;

namespace FunctionApp1
{
    public static class Function1
    {
        [FunctionName("Function1")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation($"C# HTTP trigger function begun");
            string response = string.Empty;

            string requestContent = await new StreamReader(req.Body).ReadToEndAsync();
            log.LogInformation($"Received events: {requestContent}");

            EventGridSubscriber eventGridSubscriber = new EventGridSubscriber();

            EventGridEvent[] eventGridEvents = eventGridSubscriber.DeserializeEventGridEvents(requestContent);

            foreach (EventGridEvent eventGridEvent in eventGridEvents)
            {
                if (eventGridEvent.Data is SubscriptionValidationEventData)
                {
                    var eventData = (SubscriptionValidationEventData)eventGridEvent.Data;
                    log.LogInformation($"Got SubscriptionValidation event data, validation code: {eventData.ValidationCode}, topic: {eventGridEvent.Topic}");
                    // Do any additional validation (as required) and then return back the below response

                    var responseData = new SubscriptionValidationResponse()
                    {
                        ValidationResponse = eventData.ValidationCode
                    };

                    return new OkObjectResult(responseData);
                }
            }

            return new OkObjectResult(response);
        }
    }
}
```

```javascript
module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function begun');
    var validationEventType = "Microsoft.EventGrid.SubscriptionValidationEvent";
    var storageBlobCreatedEvent = "Microsoft.Storage.BlobCreated";

    for (var events in req.body) {
        var body = req.body[events];
        // Deserialize the event data into the appropriate type based on event type  
        if (body.data && body.eventType == validationEventType) {
            context.log("Got SubscriptionValidation event data, validation code: " + body.data.validationCode + " topic: " + body.topic);

            // Do any additional validation (as required) and then return back the below response
            var code = body.data.validationCode;
            context.res = { status: 200, body: { "ValidationResponse": code } };
        }

        else if (body.data && body.eventType == storageBlobCreatedEvent) {
            var blobCreatedEventData = body.data;
            context.log("Relaying received blob created event payload:" + JSON.stringify(blobCreatedEventData));
        }
    }
    context.done();
};

```

### <a name="test-blob-created-event-handling"></a>Tester le traitement de l’événement Blob Created

Testez la nouvelle fonctionnalité de la fonction en plaçant un [événement de stockage Blob](./event-schema-blob-storage.md#example-event) dans le champ de test en exécutant :

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/xstoretestaccount",
  "subject": "/blobServices/default/containers/testcontainer/blobs/testfile.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "eTag": "0x8D4BCC2E4835CD0",
    "contentType": "text/plain",
    "contentLength": 524288,
    "blobType": "BlockBlob",
    "url": "https://example.blob.core.windows.net/testcontainer/testfile.txt",
    "sequencer": "00000000000004420000000000028963",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

Vous devez voir la sortie d’URL de blob dans le journal de la fonction :

![Journal de sortie](./media/receive-events/blob-event-response.png)

Vous pouvez également effectuer le test en créant un compte de stockage Blob ou un compte de stockage à usage général V2 (GPv2), [en ajoutant un abonnement à des événements](../storage/blobs/storage-blob-event-quickstart.md) et en définissant le point de terminaison sur l’URL de la fonction :

![URL de la fonction](./media/receive-events/function-url.png)

## <a name="handle-custom-events"></a>Traiter des événements personnalisés

Enfin, étendons la fonction une fois de plus pour qu’elle puisse également traiter des événements personnalisés. 

Dans C#, le Kit de développement logiciel (SDK) prend en charge le mappage d’un nom de type d’événement au type de données d’événement. Utilisez la fonction `AddOrUpdateCustomEventMapping()` pour mapper l’événement personnalisé.

Ajoutez une vérification pour l’événement `Contoso.Items.ItemReceived`. Votre code final doit ressembler à ceci :

```cs
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Azure.EventGrid;
using Newtonsoft.Json;

namespace FunctionApp1
{
    class ContosoItemReceivedEventData
    {
        [JsonProperty("itemSku")]
        public string ItemSku { get; set; }
    }
    public static class Function1
    {
        [FunctionName("Function1")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation($"C# HTTP trigger function begun");
            string response = string.Empty;

            string requestContent = await new StreamReader(req.Body).ReadToEndAsync();
            log.LogInformation($"Received events: {requestContent}");

            EventGridSubscriber eventGridSubscriber = new EventGridSubscriber();
            eventGridSubscriber.AddOrUpdateCustomEventMapping("Contoso.Items.ItemReceived", typeof(ContosoItemReceivedEventData));
            EventGridEvent[] eventGridEvents = eventGridSubscriber.DeserializeEventGridEvents(requestContent);

            foreach (EventGridEvent eventGridEvent in eventGridEvents)
            {
                if (eventGridEvent.Data is SubscriptionValidationEventData)
                {
                    var eventData = (SubscriptionValidationEventData)eventGridEvent.Data;
                    log.LogInformation($"Got SubscriptionValidation event data, validation code: {eventData.ValidationCode}, topic: {eventGridEvent.Topic}");
                    // Do any additional validation (as required) and then return back the below response

                    var responseData = new SubscriptionValidationResponse()
                    {
                        ValidationResponse = eventData.ValidationCode
                    };

                    return new OkObjectResult(responseData);
                }
                else if (eventGridEvent.Data is StorageBlobCreatedEventData)
                {
                    var eventData = (StorageBlobCreatedEventData)eventGridEvent.Data;
                    log.LogInformation($"Got BlobCreated event data, blob URI {eventData.Url}");
                }
                else if (eventGridEvent.Data is ContosoItemReceivedEventData)
                {
                    var eventData = (ContosoItemReceivedEventData)eventGridEvent.Data;
                    log.LogInformation($"Got ContosoItemReceived event data, item SKU {eventData.ItemSku}");
                }
            }

            return new OkObjectResult(response);
        }
    }
}
```

```javascript
module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function begun');
    var validationEventType = "Microsoft.EventGrid.SubscriptionValidationEvent";
    var storageBlobCreatedEvent = "Microsoft.Storage.BlobCreated";
    var customEventType = "Contoso.Items.ItemReceived";

    for (var events in req.body) {
        var body = req.body[events];
        // Deserialize the event data into the appropriate type based on event type
        if (body.data && body.eventType == validationEventType) {
            context.log("Got SubscriptionValidation event data, validation code: " + body.data.validationCode + " topic: " + body.topic);

            // Do any additional validation (as required) and then return back the below response
            var code = body.data.validationCode;
            context.res = { status: 200, body: { "ValidationResponse": code } };
        }

        else if (body.data && body.eventType == storageBlobCreatedEvent) {
            var blobCreatedEventData = body.data;
            context.log("Relaying received blob created event payload:" + JSON.stringify(blobCreatedEventData));
        }

        else if (body.data && body.eventType == customEventType) {
            var payload = body.data;
            context.log("Relaying received custom payload:" + JSON.stringify(payload));
        }
    }
    context.done();
};
```

### <a name="test-custom-event-handling"></a>Tester le traitement des événements personnalisés

Enfin, testez que votre fonction peut maintenant traiter votre type d’événement personnalisé :

```json
[{
    "subject": "Contoso/foo/bar/items",
    "eventType": "Contoso.Items.ItemReceived",
    "eventTime": "2017-08-16T01:57:26.005121Z",
    "id": "602a88ef-0001-00e6-1233-1646070610ea",
    "data": { 
            "itemSku": "Standard"
            },
    "dataVersion": "",
    "metadataVersion": "1"
}]
```

Vous pouvez également tester cette fonctionnalité en direct en [envoyant un événement personnalisé avec CURL à partir du portail](./custom-event-quickstart-portal.md) ou en [publiant vers une rubrique personnalisée](./post-to-custom-topic.md) à l’aide d’un service ou d’une application qui peut PUBLIER sur un point de terminaison tel que [Postman](https://www.getpostman.com/). Créez une rubrique personnalisée et un abonnement à des événements avec le point de terminaison défini en tant qu’URL de la fonction.

[!INCLUDE [event-grid-message-headers](../../includes/event-grid-message-headers.md)]

## <a name="next-steps"></a>Étapes suivantes

* Explorez les [kits SDK Azure Event Grid de gestion et de publication](./sdk-overview.md)
* Découvrez comment [publier vers une rubrique personnalisée](./post-to-custom-topic.md)
* Essayez l’un des didacticiels approfondis sur Event Grid et les fonctions tel que [Automatiser le redimensionnement des images chargées à l’aide d’Event Grid](resize-images-on-storage-blob-upload-event.md)
