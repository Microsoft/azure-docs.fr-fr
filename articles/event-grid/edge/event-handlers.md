---
title: Gestionnaires d’événements et destinations – Azure Event Grid IoT Edge | Microsoft Docs
description: Gestionnaires d’événements et destinations dans Event Grid sur Edge
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 18e2b944f60ebdf8a1b0c60ba446a13df13134c5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86171582"
---
# <a name="event-handlers-and-destinations-in-event-grid-on-edge"></a>Gestionnaires d’événements et destinations dans Event Grid sur Edge

Un gestionnaire d’événements désigne l’endroit où l’événement se trouve pour y effectuer une autre action ou pour traiter l’événement. Avec le module Event Grid sur Edge, le gestionnaire d’événements peut se trouver sur le même appareil de périphérie, sur un autre appareil ou dans le cloud. Vous pouvez utiliser n’importe quel WebHook pour gérer des événements, ou envoyer des événements à l’un des gestionnaires natifs, comme Azure Event Grid.

Cet article fournit des informations sur chacune des configurations.

## <a name="webhook"></a>WebHook

Pour publier sur un point de terminaison WebHook, définissez `endpointType` sur `WebHook` et indiquez :

* endpointUrl : URL du point de terminaison du WebHook.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "<your-webhook-endpoint>"
              }
            }
          }
        }
    ```

## <a name="azure-event-grid"></a>Azure Event Grid

Pour publier sur un point de terminaison cloud Azure Event Grid, définissez `endpointType` sur `eventGrid` et indiquez :

* endpointUrl : URL de la rubrique Event Grid dans le cloud.
* sasKey : clé SAS de la rubrique Event Grid.
* topicName : nom servant à marquer tous les événements sortants vers Event Grid. Le nom de rubrique est utile lors de la publication dans une rubrique de domaine Event Grid.

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

## <a name="iot-edge-hub"></a>Hub IoT Edge

Pour publier dans un module Edge Hub, définissez `endpointType` sur `edgeHub` et indiquez :

* outputName : sortie sur laquelle le module Event Grid achemine les événements correspondant à cet abonnement vers edgeHub. Par exemple, les événements qui correspondent à l’abonnement ci-dessous seront écrits dans /messages/modules/eventgridmodule/outputs/sampleSub4.

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

## <a name="event-hubs"></a>Event Hubs

Pour publier sur un hub d’événements, définissez `endpointType` sur `eventHub` et indiquez :

* connectionString : chaîne de connexion pour l'Event Hub spécifique que vous ciblez, générée par le biais d’une stratégie d’accès partagé.

    >[!NOTE]
    > La chaîne de connexion doit être propre à l’entité. L’utilisation d’une chaîne de connexion d’espace de noms ne fonctionne pas. Vous pouvez générer une chaîne de connexion spécifique à une entité en accédant au hub d’événements sur lequel vous souhaitez publier dans le portail Azure, et en cliquant sur **Stratégies d’accès partagé** pour générer une chaîne de connexion propre à l’entité.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "eventHub",
              "properties": {
                "connectionString": "<your-event-hub-connection-string>"
              }
            }
          }
        }
    ```

## <a name="service-bus-queues"></a>Files d'attente Service Bus

Pour publier dans une file d’attente Service Bus, définissez `endpointType` sur `serviceBusQueue` et indiquez :

* connectionString : chaîne de connexion pour la file d’attente Service Bus spécifique que vous ciblez, générée par le biais d’une stratégie d’accès partagé.

    >[!NOTE]
    > La chaîne de connexion doit être propre à l’entité. L’utilisation d’une chaîne de connexion d’espace de noms ne fonctionne pas. Générez une chaîne de connexion spécifique à une entité en accédant à la file d’attente Service Bus sur laquelle vous souhaitez publier dans le portail Azure, et en cliquant sur **Stratégies d’accès partagé** pour générer une chaîne de connexion propre à l’entité.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "serviceBusQueue",
              "properties": {
                "connectionString": "<your-service-bus-queue-connection-string>"
              }
            }
          }
        }
    ```

## <a name="service-bus-topics"></a>Rubriques de Service Bus

Pour publier dans une rubrique Service Bus, définissez `endpointType` sur `serviceBusTopic` et indiquez :

* connectionString : chaîne de connexion pour la rubrique Service Bus spécifique que vous ciblez, générée par le biais d’une stratégie d’accès partagé.

    >[!NOTE]
    > La chaîne de connexion doit être propre à l’entité. L’utilisation d’une chaîne de connexion d’espace de noms ne fonctionne pas. Générez une chaîne de connexion spécifique à une entité en accédant à la rubrique Service Bus sur laquelle vous souhaitez publier dans le portail Azure, et en cliquant sur **Stratégies d’accès partagé** pour générer une chaîne de connexion propre à l’entité.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "serviceBusTopic",
              "properties": {
                "connectionString": "<your-service-bus-topic-connection-string>"
              }
            }
          }
        }
    ```

## <a name="storage-queues"></a>Files d’attente de stockage

Pour publier dans une file d’attente de stockage, définissez `endpointType` sur `storageQueue` et indiquez :

* queueName : nom de la file d’attente de stockage dans laquelle vous publiez.
* connectionString : chaîne de connexion pour le compte de stockage dans lequel se trouve la file d'attente de stockage.

    >[!NOTE]
    > Contrairement aux Event Hubs, aux rubriques et files d’attente Service Bus, la chaîne de connexion utilisée pour les files d’attente de stockage n’est pas propre à l’entité. C’est la chaîne de connexion du compte de stockage qui doit être utilisée à la place.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "storageQueue",
              "properties": {
                "queueName": "<your-storage-queue-name>",
                "connectionString": "<your-storage-account-connection-string>"
              }
            }
          }
        }
    ```