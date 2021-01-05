---
title: Transférer des événements Event Grid vers IoTHub – Azure Event Grid vers IoT Edge | Microsoft Docs
description: Transférer des événements Event Grid vers IoTHub
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 36dc7d098892fb2be7c2ba3d75de7c7adef1a4f1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86171548"
---
# <a name="tutorial-forward-events-to-iothub"></a>Tutoriel : Transférer des événements vers IoT Hub

Cet article vous guide tout au long des étapes nécessaires pour transférer des événements Event Grid vers d’autres modules IoT Edge, IoTHub à l’aide d’itinéraires. Vous pouvez le faire pour les raisons suivantes :

* Continuer à utiliser des investissements existants déjà en place avec le routage de edgeHub
* Préférer router tous les événements à partir d’un seul appareil via IoT Hub

Pour suivre ce didacticiel, vous devez comprendre les concepts suivants :

- [Concepts d’Event Grid](concepts.md)
- [Hub IoT Edge](../../iot-edge/module-composition.md) 

## <a name="prerequisites"></a>Prérequis 
Pour suivre ce didacticiel, vous avez besoin des éléments suivants :

* **Abonnement Azure** : Créez un [compte gratuit](https://azure.microsoft.com/free) si vous n’en avez pas encore. 
* **Appareil Azure IoT Hub et IoT Edge** : suivez les étapes du guide de démarrage rapide pour les appareils [Linux](../../iot-edge/quickstart-linux.md) ou [Windows](../../iot-edge/quickstart.md) si vous n’en avez pas encore.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-deploy-iot-edge.md)]

## <a name="create-topic"></a>Créer une rubrique

En tant qu’éditeur d’un événement, vous devez créer une rubrique Event Grid. La rubrique fait référence à un point de terminaison auquel les éditeurs peuvent ensuite envoyer des événements.

1. Créez topic4 avec le contenu suivant. Pour plus d’informations sur la charge utile, voir la [documentation sur l’API](api.md).

   ```json
    {
          "name": "sampleTopic4",
          "properties": {
            "inputschema": "eventGridSchema"
          }
    }
    ```
1. Pour créer la rubrique, exécutez la commande suivante. Le code d’état HTTP 200 OK doit être retourné.

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @topic4.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic4?api-version=2019-01-01-preview
    ```

1. Exécutez la commande suivante pour vérifier la bonne création de la rubrique. Le code d’état HTTP 200 OK doit être retourné.

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic4?api-version=2019-01-01-preview
    ```

   Exemple de sortie :

   ```json
        [
          {
            "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic4",
            "name": "sampleTopic4",
            "type": "Microsoft.EventGrid/topics",
            "properties": {
              "endpoint": "https://<edge-vm-ip>:4438/topics/sampleTopic4/events?api-version=2019-01-01-preview",
              "inputSchema": "EventGridSchema"
            }
          }
        ]
   ```

## <a name="create-event-subscription"></a>Créer un abonnement aux événements

Les abonnés peuvent s’inscrire aux événements publiés dans une rubrique. Pour recevoir des événements, ils doivent créer un abonnement Event Grid sur une rubrique intéressante.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-edge-persist-event-subscriptions.md)]

1. Créez subscription4.json avec le contenu ci-dessous. Pour plus d’informations sur la charge utile, voir la [documentation sur l’API](api.md).

   ```json
    {
          "properties": {
                "destination": {
                      "endpointType": "edgeHub",
                      "properties": {
                            "outputName": "sampleSub4"
                      }
                }
          }
    }
   ```

   >[!NOTE]
   > `endpointType` spécifie que l’abonné est `edgeHub`. `outputName` spécifie la sortie sur laquelle le module Event Grid acheminera les événements qui correspondent à cet abonnement vers edgeHub. Par exemple, les événements qui correspondent à l’abonnement ci-dessus seront écrits dans `/messages/modules/eventgridmodule/outputs/sampleSub4`.
2. Exécutez la commande suivante pour créer l’abonnement. Le code d’état HTTP 200 OK doit être retourné.

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @subscription4.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic4/eventSubscriptions/sampleSubscription4?api-version=2019-01-01-preview
    ```
3. Exécutez la commande suivante pour vérifier la bonne création de l’abonnement. Le code d’état HTTP 200 OK doit être retourné.

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic4/eventSubscriptions/sampleSubscription4?api-version=2019-01-01-preview
    ```

    Exemple de sortie :

   ```json
        {
          "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic4/eventSubscriptions/sampleSubscription4",
          "type": "Microsoft.EventGrid/eventSubscriptions",
          "name": "sampleSubscription4",
          "properties": {
            "Topic": "sampleTopic4",
            "destination": {
                      "endpointType": "edgeHub",
                      "properties": {
                            "outputName": "sampleSub4"
                      }
                }
          }
        }
    ```

## <a name="set-up-an-edge-hub-route"></a>Configurer un itinéraire Edge Hub

Mettez à jour l’itinéraire du Hub Edge pour transférer les événements de l’abonnement qui sont à transférer à IoTHub comme suit :

1. Connectez-vous au [portail Azure](https://ms.portal.azure.com)
1. Accédez au **IoT Hub**.
1. Sélectionnez **IoT Edge** dans le menu
1. Cliquez sur l’ID de l’appareil cible dans la liste des appareils.
1. Sélectionnez **Définir modules**.
1. Sélectionnez **Suivant** et à la section des itinéraires.
1. Dans les itinéraires, ajoutez un itinéraire

  ```sh
  "fromEventGridToIoTHub":"FROM /messages/modules/eventgridmodule/outputs/sampleSub4 INTO $upstream"
  ```

  Par exemple,

  ```json
  {
      "routes": {
        "fromEventGridToIoTHub": "FROM /messages/modules/eventgridmodule/outputs/sampleSub4 INTO $upstream"
      }
  }
  ```

   >[!NOTE]
   > L’itinéraire ci-dessus transfère tous les événements correspondant à cet abonnement qui sont à transférer vers IoT Hub. Vous pouvez utiliser les fonctionnalités de [routage Edge Hub](../../iot-edge/module-composition.md) pour affiner le filtrage et acheminer les événements Event Grid vers d’autres modules IoT Edge.

## <a name="setup-iot-hub-route"></a>Configurer l’itinéraire IoT Hub

Consultez le [didacticiel sur le routage IoT Hub](../../iot-hub/tutorial-routing.md) pour configurer un itinéraire à partir de l’IoT Hub afin de pouvoir afficher les événements transférés à partir du module Event Grid. Utilisez `true` pour la requête afin de garder le didacticiel simple.  



## <a name="publish-an-event"></a>Publier un événement

1. Créez event4.json avec le contenu suivant. Pour plus d’informations sur la charge utile, voir la [documentation sur l’API](api.md).

    ```json
        [
          {
            "id": "eventId-iothub-1",
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

1. Exécutez la commande suivante pour publier un événement :

    ```sh
    curl -k -H "Content-Type: application/json" -X POST -g -d @event4.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic4/events?api-version=2019-01-01-preview
    ```

## <a name="verify-event-delivery"></a>Vérifier la livraison de l’événement

Pour connaître les étapes permettant d’afficher les événements, voir le [didacticiel sur le routage IoT Hub](../../iot-hub/tutorial-routing.md).

## <a name="cleanup-resources"></a>Nettoyer les ressources

* Exécutez la commande suivante pour supprimer la rubrique et tous ses abonnements à la périphérie :

    ```sh
    curl -k -H "Content-Type: application/json" -X DELETE https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic4?api-version=2019-01-01-preview
    ```
* Supprimez également toutes les ressources créées lors de la configuration du routage IoTHub dans le cloud.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez créé une rubrique Event Grid, un abonnement Edge Hub et des événements publiés. Maintenant que vous connaissez les étapes de base à suivre pour transférer vers un Edge Hub, consultez les articles suivants :

* Pour résoudre les problèmes liés à l’utilisation d’Azure Event Grid sur IoT Edge, voir le [Guide de dépannage](troubleshoot.md).
* Utiliser des filtres de routage [Edge Hub](../../iot-edge/module-composition.md) pour des événements de partition
* Configurer la persistance du module Event Grid sur [Linux](persist-state-linux.md) ou [Windows](persist-state-windows.md)
* Suivre la [documentation](configure-client-auth.md) pour configurer l’authentification du client
* Transférer les événements vers Azure Event Grid dans le cloud en suivant ce [didacticiel](forward-events-event-grid-cloud.md)
* [Superviser les rubriques et les abonnements à la périphérie](monitor-topics-subscriptions.md)