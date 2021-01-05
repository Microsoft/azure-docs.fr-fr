---
title: Azure IoT Hub et Event Grid | Microsoft Docs
description: Azure Event Grid permet de déclencher des processus en fonction d’actions qui se produisent dans IoT Hub.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 02/20/2019
ms.author: robinsh
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
ms.openlocfilehash: 0e0ca8a787145fb40087a2d99be85607404eebfa
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92152127"
---
# <a name="react-to-iot-hub-events-by-using-event-grid-to-trigger-actions"></a>Réagir aux événements IoT Hub en utilisant Event Grid pour déclencher des actions

Azure IoT Hub s’intègre à Azure Event Grid pour vous permettre d’envoyer des notifications d’événements à d’autres services et de déclencher des processus en aval. Configurez vos applications métier pour écouter les événements IoT Hub, afin de pouvoir réagir aux événements critiques de manière fiable, scalable et sécurisée.  Vous pouvez par exemple créer une application qui met à jour une base de données, crée un ticket et remet une notification par e-mail chaque fois qu’un nouvel appareil IoT est inscrit auprès de votre hub IoT.

[Azure Event Grid](../event-grid/overview.md) est un service de routage d’événement entièrement géré qui utilise le modèle publication-abonnement. Event Grid offre une prise en charge intégrée des services Azure comme [Azure Functions](../azure-functions/functions-overview.md) et [Azure Logic Apps](../logic-apps/logic-apps-overview.md), et peut fournir des alertes d’événements à des services non-Azure à l’aide de webhooks. Pour obtenir une liste complète des gestionnaires d’événements qui prennent en charge Event Grid, consultez [Présentation d’Azure Event Grid](../event-grid/overview.md).

![Architecture d’Azure Event Grid](./media/iot-hub-event-grid/event-grid-functional-model.png)

## <a name="regional-availability"></a>Disponibilité régionale

L’intégration Event Grid est disponible pour les hubs IoT situés dans les régions où Event Grid est pris en charge. Pour consulter la dernière liste des régions, reportez-vous à [Présentation d’Azure Event Grid](../event-grid/overview.md).

## <a name="event-types"></a>Types d’événements

IoT Hub publie les types d’événements suivants :

| Type d'événement | Description |
| ---------- | ----------- |
| Microsoft.Devices.DeviceCreated | Publié quand un appareil est inscrit auprès d’un hub IoT. |
| Microsoft.Devices.DeviceDeleted | Publié quand un appareil est supprimé d’un hub IoT. |
| Microsoft.Devices.DeviceConnected | Publié quand un appareil est connecté à un hub IoT. |
| Microsoft.Devices.DeviceDisconnected | Publié quand un appareil est déconnecté d’un hub IoT. |
| Microsoft.Devices.DeviceTelemetry | Publié quand un message de télémétrie d’appareil est envoyé à un hub IoT |

Utilisez le portail Azure ou Azure CLI pour configurer les événements à publier à partir de chaque hub IoT. Pour obtenir un exemple, suivez le didacticiel [Envoyer des notifications par e-mail sur des événements Azure IoT Hub à l’aide de Logic Apps](../event-grid/publish-iot-hub-events-to-logic-apps.md).

## <a name="event-schema"></a>Schéma d’événement

Les événements IoT Hub contiennent toutes les informations dont vous avez besoin pour répondre aux modifications du cycle de vie de vos appareils. Vous pouvez identifier un événement IoT Hub en vérifiant que la propriété eventType commence par **Microsoft.Devices**. Pour plus d’informations sur la façon d’utiliser des propriétés d’événements Event Grid, consultez le [Schéma d’événement Azure Event Grid](../event-grid/event-schema.md).

### <a name="device-connected-schema"></a>Schéma d’appareil connecté

L’exemple suivant montre le schéma d’un événement de connexion d’appareil :

```json
[{  
  "id": "f6bbf8f4-d365-520d-a878-17bf7238abd8",
  "topic": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>",
  "subject": "devices/LogicAppTestDevice",
  "eventType": "Microsoft.Devices.DeviceConnected",
  "eventTime": "2018-06-02T19:17:44.4383997Z",
  "data": {
      "deviceConnectionStateEventInfo": {
        "sequenceNumber":
          "000000000000000001D4132452F67CE200000002000000000000000000000001"
      },
    "hubName": "egtesthub1",
    "deviceId": "LogicAppTestDevice",
    "moduleId" : "DeviceModuleID",
  }, 
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```



### <a name="device-telemetry-schema"></a>Schéma de télémétrie d’appareil

Le message de télémétrie d’appareil doit être dans un format JSON valide avec contentType défini sur **application/json** et contentEncoding défini sur **UTF-8** dans les [propriétés système](iot-hub-devguide-routing-query-syntax.md#system-properties) du message. Ces deux propriétés sont insensibles à la casse. Si ce codage n’est pas défini, IoT Hub écrit les messages dans un format codé base-64.

Vous pouvez enrichir les événements de télémétrie d’appareil avant de les publier sur Event Grid en sélectionnant le point de terminaison en tant qu’Event Grid. Pour plus d’informations, consultez [Vue d’ensemble des enrichissements de messages](iot-hub-message-enrichments-overview.md).

L’exemple suivant montre le schéma d’un événement de télémétrie d’appareil :

```json
[{  
  "id": "9af86784-8d40-fe2g-8b2a-bab65e106785",
  "topic": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>",
  "subject": "devices/LogicAppTestDevice",
  "eventType": "Microsoft.Devices.DeviceTelemetry",
  "eventTime": "2019-01-07T20:58:30.48Z",
  "data": {
      "body": {
          "Weather": {
              "Temperature": 900
            },
            "Location": "USA"
        },
        "properties": {
            "Status": "Active"
        },
        "systemProperties": {
          "iothub-content-type": "application/json",
          "iothub-content-encoding": "utf-8",
          "iothub-connection-device-id": "d1",
          "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
          "iothub-connection-auth-generation-id": "123455432199234570",
          "iothub-enqueuedtime": "2019-01-07T20:58:30.48Z",
          "iothub-message-source": "Telemetry"
        }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

### <a name="device-created-schema"></a>Schéma de création d’appareil

L’exemple suivant montre le schéma d’un événement de création d’appareil :

```json
[{
  "id": "56afc886-767b-d359-d59e-0da7877166b2",
  "topic": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>",
  "subject": "devices/LogicAppTestDevice",
  "eventType": "Microsoft.Devices.DeviceCreated",
  "eventTime": "2018-01-02T19:17:44.4383997Z",
  "data": {
    "twin": {
      "deviceId": "LogicAppTestDevice",
      "etag": "AAAAAAAAAAE=",
      "deviceEtag":"null",
      "status": "enabled",
      "statusUpdateTime": "0001-01-01T00:00:00",
      "connectionState": "Disconnected",
      "lastActivityTime": "0001-01-01T00:00:00",
      "cloudToDeviceMessageCount": 0,
      "authenticationType": "sas",
      "x509Thumbprint": {
        "primaryThumbprint": null,
        "secondaryThumbprint": null
      },
      "version": 2,
      "properties": {
        "desired": {
          "$metadata": {
            "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
          },
          "$version": 1
        },
        "reported": {
          "$metadata": {
            "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
          },
          "$version": 1
        }
      }
    },
    "hubName": "egtesthub1",
    "deviceId": "LogicAppTestDevice"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```


> [!WARNING]
> Les *données de jumeaux* associées à un événement de création d’appareil sont une configuration par défaut et *ne doivent pas* être utilisées pour la propriété `authenticationType` réelle et autres propriétés d’appareil dans un appareil nouvellement créé. Pour `authenticationType` et les autres propriétés d’un appareil nouvellement créé, utilisez l’API de gestionnaire d’inscription fournie dans les Kits de développement logiciel (SDK) Azure IoT.

Pour obtenir une description détaillée de chaque propriété, consultez [Schéma d’événement Azure Event Grid pour IoT Hub](../event-grid/event-schema-iot-hub.md).

## <a name="filter-events"></a>Filtrer les événements

Les abonnements aux événements IoT Hub peuvent filtrer les événements en fonction du type d’événement, du contenu des données et du sujet, qui est le nom de l’appareil.

Event Grid permet le [filtrage](../event-grid/event-filtering.md) en fonction des types d’événements, des sujets et du contenu des données. Lors de la création de l’abonnement Event Grid, vous pouvez choisir de vous abonner à des événements IoT sélectionnés. Les filtres d’objets dans Event Grid fonctionnent d’après des correspondances de **Commence avec** (préfixe) et de **Se termine par** (suffixe). Le filtre utilise un opérateur `AND`. Ainsi, les événements avec un objet qui correspond à la fois au préfixe et au suffixe sont remis à l’abonné.

L’objet des événements IoT utilise le format suivant :

```json
devices/{deviceId}
```

Event Grid permet également de filtrer des attributs pour chaque événement, notamment le contenu des données. Ceci vous permet de choisir quels événements sont fournis sur la base de contenus du message de télémétrie. Consultez [Filtrage avancé](../event-grid/event-filtering.md#advanced-filtering) pour afficher des exemples. Pour le filtrage du corps du message de télémétrie, vous devez définir contentType sur **application/json** et contentEncoding sur **UTF-8** dans les [propriétés système](./iot-hub-devguide-routing-query-syntax.md#system-properties) de message. Ces deux propriétés sont insensibles à la casse.

Pour les événements autres que ceux de télémétrie, tels que DeviceConnected, DeviceDisconnected, DeviceCreated et DeviceDeleted, le filtrage d’Event Grid peut être utilisé lors de la création de l’abonnement. Pour les événements de télémétrie, outre le filtrage dans Event Grid, les utilisateurs peuvent également filtrer sur des jumeaux d’appareil, les propriétés et le corps de message et la requête de routage. 

Lorsque vous vous abonnez à des événements de télémétrie par le biais d’Event Grid, IoT Hub crée un itinéraire de message par défaut pour envoyer des messages de type source de données à Event Grid. Pour plus d’informations sur le routage des messages, consultez [Routage des messages IoT Hub](iot-hub-devguide-messages-d2c.md). Cet itinéraire sera visible dans le portail sous IoT Hub > Routage des messages. Un seul itinéraire vers Event Grid est créé, quel que soit le nombre d’abonnements EG créés pour les événements de télémétrie. Par conséquent, si vous avez besoin de plusieurs abonnements avec des filtres différents, vous pouvez utiliser l’opérateur OR dans ces requêtes sur le même itinéraire. La création et la suppression de l’itinéraire sont contrôlées par le biais de l’abonnement des événements de télémétrie via Event Grid. Vous ne pouvez pas créer ou supprimer un itinéraire vers Event Grid à l’aide du routage des messages IoT Hub.

Pour filtrer les messages avant l’envoi de données de télémétrie, vous pouvez mettre à jour votre [requête de routage](iot-hub-devguide-routing-query-syntax.md). Notez que cette requête de routage ne peut être appliquée au corps du message que s’il est au format JSON. Vous devez également définir contentType sur **application/json** et contentEncoding sur **UTF-8** dans les [propriétés système](./iot-hub-devguide-routing-query-syntax.md#system-properties) de message.

## <a name="limitations-for-device-connected-and-device-disconnected-events"></a>Limitations pour les événements d’état de la connexion et de la déconnexion d’appareils

Pour recevoir des événements de l’état de la connexion de l’appareil, un appareil doit effectuer une opération « Envoi de données de télémétrie D2C » « Réception de message C2D » avec IOT Hub. Toutefois, si un appareil utilise le protocole AMQP pour se connecter à IOT Hub, il est recommandé d’effectuer une opération« Réception de message C2D ». Sinon, les notifications d’état de la connexion peuvent être retardées de quelques minutes. Si votre appareil utilise le protocole MQTT, IoT Hub gardera le lien C2D ouvert. Pour AMQP, vous pouvez ouvrir le lien C2D en appelant l’API Receive Async pour le Kit de développement logiciel (SDK) C# IoT Hub ou le [client d’appareil pour AMQP](iot-hub-amqp-support.md#device-client).

Le lien D2C est ouvert si vous envoyez des données de télémétrie. 

Si la connexion de l’appareil est intermittente, c’est à dire que l’appareil se connecte et se déconnecte fréquemment, nous n’enverrons pas chaque état de connexion, mais nous publierons l’état de connexion actuel pris à un instantanée périodique, tant que la connexion ne sera pas rétablie. Si vous recevez un même événement d’état de connexion avec des numéros de séquence différents ou des événements d’état de connexion différents, cela signifie que l’état de connexion de l’appareil a changé.

## <a name="tips-for-consuming-events"></a>Conseils relatifs à la consommation d’événements

Les applications qui gèrent des événements IoT Hub doivent suivre ces pratiques suggérées :

* Plusieurs abonnements peuvent être configurés pour acheminer les événements vers le même gestionnaire d’événements. Il ne faut donc pas supposer que les événements proviennent d’une source particulière. Vérifiez toujours la rubrique du message pour être sûr qu’il provient du hub IoT prévu.

* Ne supposez pas que tous les événements reçus sont des types attendus. Vérifiez toujours eventType avant de traiter le message.

* Les messages peuvent arriver dans le désordre ou après un certain délai. Utilisez le champ etag pour vérifier si vos informations sur les objets sont à jour concernant l’appareil créé ou les événements supprimés de l’appareil.

## <a name="next-steps"></a>Étapes suivantes

* [Suivre le didacticiel d’événements IoT Hub](../event-grid/publish-iot-hub-events-to-logic-apps.md)

* [Apprendre à commander des événements d’état de la connexion et de la déconnexion d’appareils](iot-hub-how-to-order-connection-state-events.md)

* [En savoir plus sur Event Grid](../event-grid/overview.md)

* [Comparer les différences entre le routage des messages et des événements IoT Hub](iot-hub-event-grid-routing-comparison.md)

* [Découvrez comment utiliser les événements de télémétrie IoT pour implémenter l’analytique spatiale via Azure Maps](../azure-maps/tutorial-iot-hub-maps.md)