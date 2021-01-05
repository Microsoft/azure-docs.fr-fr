---
title: Utiliser Azure Event Grid avec des événements dans le schéma CloudEvents
description: Décrit comment utiliser le schéma CloudEvents pour les événements dans Azure Event Grid. Le service prend en charge les événements dans l’implémentation JSON de CloudEvents.
ms.topic: conceptual
ms.date: 11/10/2020
ms.custom: devx-track-js, devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: e13c3635da7e7a86f4fa2d31215303152167741c
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97109247"
---
# <a name="use-cloudevents-v10-schema-with-event-grid"></a>Utiliser le schéma CloudEvents v1.0 avec Event Grid
En plus de son [schéma d’événement par défaut](event-schema.md), Azure Event Grid prend en charge en mode natif les événements dans l’[implémentation JSON de CloudEvents v1.0](https://github.com/cloudevents/spec/blob/v1.0/json-format.md) et la [liaison de protocole HTTP](https://github.com/cloudevents/spec/blob/v1.0/http-protocol-binding.md). [CloudEvents](https://cloudevents.io/) est une [spécification ouverte](https://github.com/cloudevents/spec/blob/v1.0/spec.md) qui décrit les données d’événement.

CloudEvents simplifie l’interopérabilité grâce à un schéma d’événement commun pour la publication et la consommation des événements basés sur le cloud. Ce schéma permet l’utilisation d’outils uniformes, la mise en œuvre de méthodes de routage et de gestion des événements standard, ainsi que de méthodes universelles pour désérialiser le schéma d’événement externe. Avec un schéma commun, vous pouvez plus facilement intégrer le travail entre plusieurs plateformes.

Plusieurs [collaborateurs](https://github.com/cloudevents/spec/blob/master/community/contributors.md), dont Microsoft, travaillent à l’élaboration de CloudEvents par le biais de la [Cloud Native Computing Foundation](https://www.cncf.io/). Il est actuellement disponible en version 1.0.

Cet article décrit comment utiliser le schéma CloudEvents avec Event Grid.


## <a name="cloudevent-schema"></a>Schéma CloudEvents

Voici un exemple d’événement Stockage Blob Azure dans le format CloudEvents :

``` JSON
{
    "specversion": "1.0",
    "type": "Microsoft.Storage.BlobCreated",  
    "source": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-account}",
    "id": "9aeb0fdf-c01e-0131-0922-9eb54906e209",
    "time": "2019-11-18T15:13:39.4589254Z",
    "subject": "blobServices/default/containers/{storage-container}/blobs/{new-file}",
    "dataschema": "#",
    "data": {
        "api": "PutBlockList",
        "clientRequestId": "4c5dd7fb-2c48-4a27-bb30-5361b5de920a",
        "requestId": "9aeb0fdf-c01e-0131-0922-9eb549000000",
        "eTag": "0x8D76C39E4407333",
        "contentType": "image/png",
        "contentLength": 30699,
        "blobType": "BlockBlob",
        "url": "https://gridtesting.blob.core.windows.net/testcontainer/{new-file}",
        "sequencer": "000000000000000000000000000099240000000000c41c18",
        "storageDiagnostics": {
            "batchId": "681fe319-3006-00a8-0022-9e7cde000000"
        }
    }
}
```

Une description détaillée des champs disponibles, de leurs types et définitions dans CloudEvents v1.0 est [disponible ici](https://github.com/cloudevents/spec/blob/v1.0/spec.md#required-attributes).

Les valeurs des en-têtes pour les événements remis dans le schéma CloudEvents et le schéma Event Grid sont identiques, à l’exception de `content-type`. Pour le schéma CloudEvents, cette valeur d’en-tête est `"content-type":"application/cloudevents+json; charset=utf-8"`. Pour le schéma Event Grid, cette valeur d’en-tête est `"content-type":"application/json; charset=utf-8"`.

## <a name="configure-event-grid-for-cloudevents"></a>Configurer Event Grid pour CloudEvents

Vous pouvez utiliser Event Grid pour l’entrée et la sortie des événements dans le schéma CloudEvents. Le tableau suivant décrit les transformations possibles :

 Ressource Event Grid | Schéma d’entrée       | Schéma de remise
|---------------------|-------------------|---------------------
| Rubriques sur le système       | Schéma Event Grid | Schéma Event Grid ou schéma CloudEvent
| Rubriques/domaines utilisateur | Schéma Event Grid | Schéma Event Grid
| Rubriques/domaines utilisateur | Schéma CloudEvent | Schéma CloudEvent
| Rubriques/domaines utilisateur | Schéma personnalisé     | Schéma personnalisé OU schéma Event Grid OU schéma CloudEvent
| PartnerTopics       | Schéma CloudEvent | Schéma CloudEvent


Pour tous les schémas d’événement, Event Grid requiert une validation au moment de la publication sur une rubrique Event Grid et de la création d’un abonnement aux événements.

Pour en savoir plus, consultez la page [Sécurité et authentification pour Event Grid](security-authentication.md).

### <a name="input-schema"></a>Schéma d’entrée

Vous définissez le schéma d’entrée pour une rubrique personnalisée lorsque vous la créez.

Pour l’interface de ligne de commande Azure, consultez :

```azurecli-interactive
az eventgrid topic create \
  --name <topic_name> \
  -l westcentralus \
  -g gridResourceGroup \
  --input-schema cloudeventschemav1_0
```

Pour PowerShell, utilisez la commande suivante :

```azurepowershell-interactive
New-AzEventGridTopic `
  -ResourceGroupName gridResourceGroup `
  -Location westcentralus `
  -Name <topic_name> `
  -InputSchema CloudEventSchemaV1_0
```

### <a name="output-schema"></a>Schéma de sortie

Vous définissez le schéma de sortie lorsque vous créez l’abonnement aux événements.

Pour l’interface de ligne de commande Azure, consultez :

```azurecli-interactive
topicID=$(az eventgrid topic show --name <topic-name> -g gridResourceGroup --query id --output tsv)

az eventgrid event-subscription create \
  --name <event_subscription_name> \
  --source-resource-id $topicID \
  --endpoint <endpoint_URL> \
  --event-delivery-schema cloudeventschemav1_0
```

Pour PowerShell, utilisez la commande suivante :
```azurepowershell-interactive
$topicid = (Get-AzEventGridTopic -ResourceGroupName gridResourceGroup -Name <topic-name>).Id

New-AzEventGridSubscription `
  -ResourceId $topicid `
  -EventSubscriptionName <event_subscription_name> `
  -Endpoint <endpoint_URL> `
  -DeliverySchema CloudEventSchemaV1_0
```

 Actuellement, vous ne pouvez pas utiliser un déclencheur Event Grid pour une application Azure Functions quand l’événement est remis dans le schéma CloudEvents. Utilisez un déclencheur HTTP. Pour obtenir des exemples d’implémentation d’un déclencheur HTTP qui reçoit des événements dans le schéma CloudEvents, consultez [Utiliser CloudEvents avec Azure Functions](#azure-functions).

 ## <a name="endpoint-validation-with-cloudevents-v10"></a>Validation de point de terminaison avec CloudEvents v1.0

Si vous êtes déjà familiarisé avec Event Grid, vous pouvez être conscient de la négociation de validation de point de terminaison de Event Grid pour empêcher les abus. CloudEvents v1.0 implémente sa propre [sémantique de protection contre les abus](webhook-event-delivery.md) à l’aide de la méthode HTTP OPTIONS. Pour plus d'informations à ce sujet, cliquez [ici](https://github.com/cloudevents/spec/blob/v1.0/http-webhook.md#4-abuse-protection). Lors de l’utilisation du schéma CloudEvents pour la sortie, Event Grid l’utilise avec la protection contre les abus CloudEvents v1.0 à la place du mécanisme d’événement de validation Event Grid.

<a name="azure-functions"></a>

## <a name="use-with-azure-functions"></a>Utiliser avec Azure Functions

La [liaison Event Grid Azure Functions](../azure-functions/functions-bindings-event-grid.md) ne prend pas en charge CloudEvents de façon native. Des fonctions déclenchées par HTTP sont donc utilisées pour lire les messages CloudEvents. Si vous utilisez un déclencheur HTTP pour lire CloudEvents, vous devrez écrire du code pour accomplir les actions que le déclencheur Event Grid effectue automatiquement :

* Envoyer une réponse de validation à une [requête de validation d’abonnement](../event-grid/webhook-event-delivery.md).
* Appeler la fonction une fois par élément du tableau d’événements contenu dans le corps de la requête.

Pour plus d’informations sur l’URL à utiliser pour appeler la fonction en local ou en cas d’exécution dans Azure, consultez la [documentation de référence des liaisons de déclencheurs HTTP](../azure-functions/functions-bindings-http-webhook.md).

L’exemple de code C# suivant pour un déclencheur HTTP simule le comportement d’un déclencheur Event Grid.  Utilisez cet exemple pour les événements remis dans le schéma CloudEvents.

```csharp
[FunctionName("HttpTrigger")]
public static async Task<HttpResponseMessage> Run([HttpTrigger(AuthorizationLevel.Anonymous, "post", "options", Route = null)]HttpRequestMessage req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");
    if (req.Method == HttpMethod.Options)
    {
        // If the request is for subscription validation, send back the validation code
        
        var response = req.CreateResponse(HttpStatusCode.OK);
        response.Headers.Add("Webhook-Allowed-Origin", "eventgrid.azure.net");

        return response;
    }

    var requestmessage = await req.Content.ReadAsStringAsync();
    var message = JToken.Parse(requestmessage);

    // The request is not for subscription validation, so it's for an event.
    // CloudEvents schema delivers one event at a time.
    log.LogInformation($"Source: {message["source"]}");
    log.LogInformation($"Time: {message["eventTime"]}");
    log.LogInformation($"Event data: {message["data"].ToString()}");

    return req.CreateResponse(HttpStatusCode.OK);
}
```

L’exemple de code JavaScript suivant pour un déclencheur HTTP simule le comportement d’un déclencheur Event Grid. Utilisez cet exemple pour les événements remis dans le schéma CloudEvents.

```javascript
module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');
    
    if (req.method == "OPTIONS") {
        // If the request is for subscription validation, send back the validation code
        
        context.log('Validate request received');
        context.res = { status: 200 };
        context.res.headers.append('Webhook-Allowed-Origin', 'eventgrid.azure.net');
    }
    else
    {
        var message = req.body;
        
        // The request is not for subscription validation, so it's for an event.
        // CloudEvents schema delivers one event at a time.
        var event = JSON.parse(message);
        context.log('Source: ' + event.source);
        context.log('Time: ' + event.eventTime);
        context.log('Data: ' + JSON.stringify(event.data));
    }
 
    context.done();
};
```

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’information sur la surveillance des remises des événements, consultez [Surveiller la remise des messages Event Grid](monitor-event-delivery.md).
* Nous vous encourageons à tester et à commenter CloudEvents, ainsi qu’à y [contribuer](https://github.com/cloudevents/spec/blob/master/community/CONTRIBUTING.md).
* Pour plus d’informations sur la création d’un abonnement Azure Event Grid, consultez [Schéma d’abonnement à Event Grid](subscription-creation-schema.md).
