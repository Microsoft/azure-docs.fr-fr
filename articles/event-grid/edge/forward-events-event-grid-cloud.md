---
title: Transférer les événements de périphérie vers le cloud Event Grid – Azure Event Grid sur IoT Edge | Microsoft Docs
description: Transférer des événements de périphérie vers le cloud Event Grid
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: bfe150a45c70bc5bed18f8e929c9567905cd38f5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86171599"
---
# <a name="tutorial-forward-events-to-event-grid-cloud"></a>Tutoriel : Transférer des événements vers le cloud Event Grid

Cet article vous guide tout au long des étapes nécessaires pour transférer des événements de périphérie vers Event Grid dans le cloud Azure. Vous pouvez le faire pour les raisons suivantes :

* Réagissez aux événements de périphérie dans le cloud.
* Transférez les événements à Event Grid dans le cloud et utilisez Azure Event Hubs ou des files d’attente de Stockage Azure pour mettre en mémoire tampon les événements avant de les traiter dans le cloud.

 Pour suivre ce didacticiel, vous devez comprendre les concepts d’Event Grid à la [périphérie](concepts.md) et dans [Azure](../concepts.md). Pour accéder à d'autres types de destinations, consultez [Gestionnaires d'événements](event-handlers.md). 

## <a name="prerequisites"></a>Prérequis 
Pour suivre ce didacticiel, vous avez besoin des éléments suivants :

* **Abonnement Azure** : Créez un [compte gratuit](https://azure.microsoft.com/free) si vous n’en avez pas encore. 
* **Appareil Azure IoT Hub et IoT Edge** : suivez les étapes du guide de démarrage rapide pour les appareils [Linux](../../iot-edge/quickstart-linux.md) ou [Windows](../../iot-edge/quickstart.md) si vous n’en avez pas encore.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-deploy-iot-edge.md)] 
## <a name="create-event-grid-topic-and-subscription-in-cloud"></a>Créer une rubrique et un abonnement Event Grid dans le cloud

Créez une rubrique et un abonnement Event Grid dans le cloud en suivant [ce didacticiel](../custom-event-quickstart-portal.md). Notez les valeurs `topicURL`, `sasKey` et `topicName` de la rubrique que vous venez de créer. Vous les utiliserez ultérieurement dans le didacticiel.

Par exemple, si vous avez créé une rubrique nommée `testegcloudtopic` dans la région USA Ouest, les valeurs ressemblent à ceci :

* **TopicUrl** : `https://testegcloudtopic.westus2-1.eventgrid.azure.net/api/events`
* **TopicName** : `testegcloudtopic`
* **SasKey** : Disponible sous l’**AccessKey** de votre rubrique. Utilisez **touche1**.

## <a name="create-event-grid-topic-at-the-edge"></a>Créer une rubrique Event Grid à la périphérie

1. Créez topic3.json avec le contenu suivant. Pour plus d’informations sur la charge utile, voir la [documentation sur l’API](api.md).

    ```json
        {
          "name": "sampleTopic3",
          "properties": {
            "inputschema": "eventGridSchema"
          }
        }
    ```
1. Pour créer la rubrique, exécutez la commande suivante. Le code d’état HTTP 200 OK doit être retourné.

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @topic3.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3?api-version=2019-01-01-preview
    ```
1. Exécutez la commande suivante pour vérifier la bonne création de la rubrique. Le code d’état HTTP 200 OK doit être retourné.

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3?api-version=2019-01-01-preview
    ```

   Exemple de sortie :

   ```json
        [
          {
            "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic3",
            "name": "sampleTopic3",
            "type": "Microsoft.EventGrid/topics",
            "properties": {
              "endpoint": "https://<edge-vm-ip>:4438/topics/sampleTopic3/events?api-version=2019-01-01-preview",
              "inputSchema": "EventGridSchema"
            }
          }
        ]
   ```
  
## <a name="create-event-grid-subscription-at-the-edge"></a>Créer un abonnement Event Grid à la périphérie

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-edge-persist-event-subscriptions.md)]

1. Créez subscription3.json avec le contenu suivant. Pour plus d’informations sur la charge utile, voir la [documentation sur l’API](api.md).

   ```json
        {
          "properties": {
            "destination": {
              "endpointType": "eventGrid",
              "properties": {
                        "endpointUrl": "<your-event-grid-cloud-topic-endpoint-url>?api-version=2018-01-01",
                        "sasKey": "<your-event-grid-topic-saskey>",
                        "topicName": null
              }
            }
          }
    }
   ```

   >[!NOTE]
   > La valeur **endpointUrl** spécifie que l’URL de la rubrique Event Grid est dans le Cloud. La valeur **sasKey** fait référence à la clé de la rubrique cloud Event Grid. La valeur **topicname** sera utilisée pour marquer tous les événements sortants sur Event Grid. Cela peut être utile lors de la publication dans une rubrique de domaine Event Grid. Pour plus d’informations sur les rubriques de domaine Event Grid, voir [Domaines d’événements](../event-domains.md)

    Par exemple,
  
    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "eventGrid",
              "properties": {
                "endpointUrl": "https://testegcloudtopic.westus2-1.eventgrid.azure.net/api/events?api-version=2018-01-01",
                 "sasKey": "<your-event-grid-topic-saskey>",
                 "topicName": null
              }
            }
          }
        }
    ```

2. Exécutez la commande suivante pour créer l’abonnement. Le code d’état HTTP 200 OK doit être retourné.

     ```sh
     curl -k -H "Content-Type: application/json" -X PUT -g -d @subscription3.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3/eventSubscriptions/sampleSubscription3?api-version=2019-01-01-preview
     ```

3. Exécutez la commande suivante pour vérifier la bonne création de l’abonnement. Le code d’état HTTP 200 OK doit être retourné.

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3/eventSubscriptions/sampleSubscription3?api-version=2019-01-01-preview
    ```

    Exemple de sortie :

    ```json
         {
          "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic3/eventSubscriptions/sampleSubscription3",
          "type": "Microsoft.EventGrid/eventSubscriptions",
          "name": "sampleSubscription3",
          "properties": {
            "Topic": "sampleTopic3",
            "destination": {
              "endpointType": "eventGrid",
              "properties": {
                "endpointUrl": "https://testegcloudtopic.westus2-1.eventgrid.azure.net/api/events?api-version=2018-01-01",
                "sasKey": "<your-event-grid-topic-saskey>",
                "topicName": null
              }
            }
          }
        }
    ```

## <a name="publish-an-event-at-the-edge"></a>Publier un événement à la périphérie

1. Créez event3.json avec le contenu suivant. Pour plus d’informations sur la charge utile, voir la [documentation sur l’API](api.md).

    ```json
        [
          {
            "id": "eventId-egcloud-0",
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

1. Exécutez la commande suivante :

    ```sh
    curl -k -H "Content-Type: application/json" -X POST -g -d @event3.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3/events?api-version=2019-01-01-preview
    ```

## <a name="verify-edge-event-in-cloud"></a>Vérifier l’événement session dans le Cloud

Pour plus d’informations sur l’affichage des événements livrés par la rubrique cloud, voir le [didacticiel](../custom-event-quickstart-portal.md).

## <a name="cleanup-resources"></a>Nettoyer les ressources

* Exécutez la commande suivante pour supprimer la rubrique et tous ses abonnements

    ```sh
    curl -k -H "Content-Type: application/json" -X DELETE https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic3?api-version=2019-01-01-preview
    ```

* Supprimez également la rubrique et les abonnements créés dans le cloud (Azure Event Grid).

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez publié un événement à la périphérie et l’avez transféré vers Event Grid dans le cloud Azure. Maintenant que vous connaissez les étapes de base à suivre pour Event Grid dans le cloud :

* Pour résoudre les problèmes liés à l’utilisation d’Azure Event Grid sur IoT Edge, voir le [Guide de dépannage](troubleshoot.md).
* Transférer des événements à IoTHub en suivant ce [didacticiel](forward-events-iothub.md)
* Transférer des événements à Webhook dans le cloud en suivant ce [didacticiel](pub-sub-events-webhook-cloud.md)
* [Superviser les rubriques et les abonnements à la périphérie](monitor-topics-subscriptions.md)
