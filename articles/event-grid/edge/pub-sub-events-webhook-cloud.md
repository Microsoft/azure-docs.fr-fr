---
title: Publier des événements et s’y abonner dans le cloud – Azure Event Grid IoT Edge | Microsoft Docs
description: Publier des événements et s’y abonner dans le cloud en utilisant Webhook avec Event Grid sur IoT Edge
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: 12bcb54f4bfdf17209324febeba380ff7789fc0f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88998983"
---
# <a name="tutorial-publish-subscribe-to-events-in-cloud"></a>Tutoriel : Publier des événements et s’y abonner dans le cloud

Cet article vous guide tout au long des étapes nécessaires pour publier des événements avec et vous abonner en utilisant Event Grid sur IoT Edge. Ce didacticiel utilise une fonction Azure comme gestionnaire d’événements. Pour accéder à d’autres types de destinations, consultez [Gestionnaires d’événements](event-handlers.md).

Consultez [Concepts d’Event Grid](concepts.md) pour comprendre ce que sont une rubrique et un abonnement Event Grid avant de continuer.

## <a name="prerequisites"></a>Prérequis 
Pour suivre ce didacticiel, vous avez besoin des éléments suivants :

* **Abonnement Azure** : Créez un [compte gratuit](https://azure.microsoft.com/free) si vous n’en avez pas encore. 
* **Appareil Azure IoT Hub et IoT Edge** : suivez les étapes du guide de démarrage rapide pour les appareils [Linux](../../iot-edge/quickstart-linux.md) ou [Windows](../../iot-edge/quickstart.md) si vous n’en avez pas encore.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-deploy-iot-edge.md)]

## <a name="create-an-azure-function-in-the-azure-portal"></a>Créer une fonction Azure dans le portail Azure

Suivez les étapes décrites dans le [didacticiel](../../azure-functions/functions-create-first-azure-function.md) pour créer une fonction Azure. 

Remplacez l’extrait de code par le code suivant :

```csharp
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;
using Newtonsoft.Json;

public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);

    log.LogInformation($"C# HTTP trigger received {data}.");
    return data != null
        ? (ActionResult)new OkResult()
        : new BadRequestObjectResult("Please pass in the request body");
}
```

Dans votre nouvelle fonction, sélectionnez **Obtenir l’URL de la fonction** en haut à droite, l’option par défaut (**touche de fonction**), puis **Copier**. Vous allez utiliser la valeur de l’URL de la fonction plus loin dans le didacticiel.

> [!NOTE]
> Pour plus d’exemples et de didacticiels sur la réaction aux événements et l’utilisation de déclencheurs d’événements EventGrid, voir la documentation [Azure Functions](../../azure-functions/functions-overview.md).

## <a name="create-a-topic"></a>Création d'une rubrique

En tant qu’éditeur d’un événement, vous devez créer une rubrique Event Grid. La rubrique fait référence à un point de terminaison auquel les éditeurs peuvent envoyer des événements.

1. Créez topic2.json avec le contenu suivant. Pour plus d’informations sur la charge utile, voir la [documentation sur l’API](api.md).

    ```json
         {
          "name": "sampleTopic2",
          "properties": {
            "inputschema": "eventGridSchema"
          }
        }
    ```
1. Pour créer la rubrique, exécutez la commande suivante. Le code d’état HTTP 200 OK doit être retourné.

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @topic2.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2?api-version=2019-01-01-preview
    ```
1. Exécutez la commande suivante pour vérifier la bonne création de la rubrique. Le code d’état HTTP 200 OK doit être retourné.

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2?api-version=2019-01-01-preview
    ```

   Exemple de sortie :

   ```json
        [
          {
            "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic2",
            "name": "sampleTopic2",
            "type": "Microsoft.EventGrid/topics",
            "properties": {
              "endpoint": "https://<edge-vm-ip>:4438/topics/sampleTopic2/events?api-version=2019-01-01-preview",
              "inputSchema": "EventGridSchema"
            }
          }
        ]
   ```

## <a name="create-an-event-subscription"></a>Créer un abonnement d’événement

Les abonnés peuvent s’inscrire aux événements publiés dans une rubrique. Pour recevoir des événements, les abonnés doivent créer un abonnement Event Grid sur une rubrique intéressante.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-edge-persist-event-subscriptions.md)]

1. Créez subscription2.json avec le contenu suivant. Pour plus d’informations sur la charge utile, voir la [documentation sur l’API](api.md).

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "<your-az-func-cloud-url>"
              }
            }
          }
        }
    ```

   >[!NOTE]
   > La propriété **endpointType** spécifie que l’abonné est un webhook.  **endpointUrl** spécifie l’URL à laquelle l’abonné écoute les événements. Cette URL correspond à l’exemple de fonction Azure que vous avez configuré précédemment.
2. Exécutez la commande suivante pour créer l’abonnement. Le code d’état HTTP 200 OK doit être retourné.

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @subscription2.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2/eventSubscriptions/sampleSubscription2?api-version=2019-01-01-preview
    ```
3. Exécutez la commande suivante pour vérifier la bonne création de l’abonnement. Le code d’état HTTP 200 OK doit être retourné.

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2/eventSubscriptions/sampleSubscription2?api-version=2019-01-01-preview
    ```

    Exemple de sortie :

   ```json
        {
          "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic2/eventSubscriptions/sampleSubscription2",
          "type": "Microsoft.EventGrid/eventSubscriptions",
          "name": "sampleSubscription2",
          "properties": {
            "Topic": "sampleTopic2",
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "<your-az-func-cloud-url>"
              }
            }
          }
        }
    ```

## <a name="publish-an-event"></a>Publier un événement

1. Créez event2.json avec le contenu suivant. Pour plus d’informations sur la charge utile, voir la [documentation sur l’API](api.md).

    ```json
        [
          {
            "id": "eventId-func-1",
            "eventType": "recordInserted",
            "subject": "myapp/vehicles/motorcycles",
            "eventTime": "2019-07-28T21:03:07+00:00",
            "dataVersion": "1.0",
            "data": {
              "make": "Ducati",
              "model": "Monster"
            }
          }
        ]
    ```
1. Exécutez la commande suivante pour publier un événement

    ```sh
    curl -k -H "Content-Type: application/json" -X POST -g -d @event2.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2/events?api-version=2019-01-01-preview
    ```

## <a name="verify-event-delivery"></a>Vérifier la livraison de l’événement

Vous pouvez afficher l’événement fourni dans le Portail Azure sous l’option **Analyser** de votre fonction.

## <a name="cleanup-resources"></a>Nettoyer les ressources

* Exécutez la commande suivante pour supprimer la rubrique et tous ses abonnements

    ```sh
    curl -k -H "Content-Type: application/json" -X DELETE https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2?api-version=2019-01-01-preview
    ```

* Supprimez la fonction Azure créée dans le portail Azure.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez créé une rubrique Event Grid, un abonnement et des événements publiés. Maintenant que vous connaissez les étapes de base, consultez les articles suivants :

* Pour résoudre les problèmes liés à l’utilisation d’Azure Event Grid sur IoT Edge, voir le [Guide de dépannage](troubleshoot.md).
* Créer/mettre à jour un abonnement avec des [filtres](advanced-filtering.md).
* Configurer la persistance du module Event Grid sur [Linux](persist-state-linux.md) ou [Windows](persist-state-windows.md)
* Suivre la [documentation](configure-client-auth.md) pour configurer l’authentification du client
* Transférer les événements vers Azure Event Grid dans le cloud en suivant ce [didacticiel](forward-events-event-grid-cloud.md)
* [Superviser les rubriques et les abonnements à la périphérie](monitor-topics-subscriptions.md)
