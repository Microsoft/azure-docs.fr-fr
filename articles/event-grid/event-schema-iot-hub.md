---
title: Azure IoT Hub en tant que source Event Grid
description: Cet article fournit les propriétés et les schémas des événements IoT Hub. Il liste les types d’événements disponibles et les propriétés d’événement, et contient un exemple d’événement.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 02ecf8d4df55aa6b4319e40892778f85f94e29a7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86113647"
---
# <a name="azure-iot-hub-as-an-event-grid-source"></a>Azure IoT Hub en tant que source Event Grid
Cet article fournit les propriétés et les schémas des événements IoT Hub. Pour une présentation des schémas d’événements, consultez [Schéma d’événements Azure Event Grid](event-schema.md). 

## <a name="event-grid-event-schema"></a>Schéma d’événement Event Grid

### <a name="available-event-types"></a>Types d’événement disponibles

IoT Hub émet les types d’événements suivants :

| Type d'événement | Description |
| ---------- | ----------- |
| Microsoft.Devices.DeviceCreated | Publié quand un appareil est inscrit auprès d’un hub IoT. |
| Microsoft.Devices.DeviceDeleted | Publié quand un appareil est supprimé d’un hub IoT. | 
| Microsoft.Devices.DeviceConnected | Publié quand un appareil est connecté à un hub IoT. |
| Microsoft.Devices.DeviceDisconnected | Publié quand un appareil est déconnecté d’un hub IoT. | 
| Microsoft.Devices.DeviceTelemetry | Publié quand un message de télémétrie est envoyé à un hub IoT. |

Tous les événements d’appareil, à l’exception des événements de télémétrie, sont en disponibilité générale dans toutes les régions prises en charge par Event Grid. L’événement de télémétrie d’appareil est disponible en préversion publique dans toutes les régions à l’exception des suivantes : USA Est, USA Ouest, Europe Ouest, [Azure Government](../azure-government/documentation-government-welcome.md), [Azure China 21Vianet](/azure/china/china-welcome) et [Azure Allemagne](https://azure.microsoft.com/global-infrastructure/germany/).

### <a name="example-event"></a>Exemple d’événement

Les schémas pour les événements DeviceConnected et DeviceDisconnected ont la même structure. Cet exemple d’événement montre le schéma d’un événement déclenché quand un appareil est connecté à un hub IoT :

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
    "moduleId" : "DeviceModuleID"
  }, 
  "dataVersion": "1", 
  "metadataVersion": "1" 
}]
```

L’événement DeviceTelemetry est déclenché quand un événement de télémétrie est envoyé à un hub IoT. Voici un exemple de schéma pour cet événement.

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

Les schémas pour les événements DeviceCreated et DeviceDeleted ont la même structure. Cet exemple d’événement montre le schéma d’un événement déclenché quand un appareil est inscrit auprès d’un hub IoT :

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
      "deviceEtag": "null",
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

### <a name="event-properties"></a>Propriétés d’événement

Tous les événements contiennent les mêmes données de niveau supérieur : 

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| id | string | Identificateur unique de l’événement. |
| topic | string | Chemin d’accès complet à la source de l’événement. Ce champ n’est pas modifiable. Event Grid fournit cette valeur. |
| subject | string | Chemin de l’objet de l’événement, défini par le serveur de publication. |
| eventType | string | Un des types d’événements inscrits pour cette source d’événement. |
| eventTime | string | L’heure à quelle l’événement est généré selon l’heure UTC du fournisseur. |
| data | object | Données d’événement IoT Hub.  |
| dataVersion | string | Version du schéma de l’objet de données. Le serveur de publication définit la version du schéma. |
| metadataVersion | string | Version du schéma des métadonnées d’événement. Event Grid définit le schéma des propriétés de niveau supérieur. Event Grid fournit cette valeur. |

Pour tous les événements IoT Hub, l’objet de données contient les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| hubName | string | Nom du hub IoT où l’appareil a été créé ou supprimé. |
| deviceId | string | Identificateur unique de l’appareil. Cette chaîne qui respecte la casse peut contenir jusqu’à 128 caractères et prend en charge les caractères alphanumériques 7 bits ASCII, ainsi que les caractères spéciaux suivants :`- : . + % _ # * ? ! ( ) , = @ ; $ '`. |

Le contenu de l’objet de données est différent pour chaque serveur de publication d’événements. 

Pour les événements IoT Hub **DeviceConnected** et **DeviceDisconnected**, l’objet de données contient les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| moduleId | string | Identificateur unique du module. Ce champ est sorti uniquement pour les appareils de module. Cette chaîne qui respecte la casse peut contenir jusqu’à 128 caractères et prend en charge les caractères alphanumériques 7 bits ASCII, ainsi que les caractères spéciaux suivants :`- : . + % _ # * ? ! ( ) , = @ ; $ '`. |
| deviceConnectionStateEventInfo | object | Informations d’événement sur l’état de connexion d’appareil
| sequenceNumber | string | Un numéro qui vous aide à indiquer l’ordre des événements de connexion et de déconnexion d’appareils. Le dernier événement aura un numéro de séquence plus élevé que l’événement précédent. Ce numéro peut changer de plus d’une unité, mais il ne peut qu’augmenter. Consultez [comment utiliser le numéro de séquence](../iot-hub/iot-hub-how-to-order-connection-state-events.md). |

Pour l’événement IoT Hub **DeviceTelemetry**, l’objet de données contient le message appareil-à-cloud au [format des messages IoT Hub](../iot-hub/iot-hub-devguide-messages-construct.md) et a les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| body | string | Contenu du message reçu de l’appareil. |
| properties | string | Les propriétés de l’application sont des chaînes définies par l’utilisateur qui peuvent être ajoutées au message. Ces champs sont facultatifs. |
| system properties | string | Les [propriétés système](../iot-hub/iot-hub-devguide-routing-query-syntax.md#system-properties) permettent d’identifier le contenu et la source des messages. Le message de télémétrie d’appareil doit être dans un format JSON valide avec contentType défini sur JSON et contentEncoding défini sur UTF-8 dans les propriétés système du message. S’il n’est pas défini, IoT Hub écrit les messages dans un format codé base-64.  |

Pour les événements IoT Hub **DeviceCreated** et **DeviDeleted**, l’objet de données contient les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| twin | object | Informations sur le jumeau d’appareil, qui est la représentation cloud des métadonnées d’appareil de l’application. | 
| deviceID | string | Identificateur unique du jumeau d’appareil. | 
| etag | string | Un validateur pour garantir la cohérence des mises à jour à un jumeau d'appareil. Chaque etag est unique pour chaque jumeau d’appareil. |  
| deviceEtag| string | Un validateur pour garantir la cohérence des mises à jour à un registre d'appareil. Chaque deviceEtag est unique pour chaque registre d’appareil. |
| status | string | Indique si le jumeau d’appareil est activé ou désactivé. | 
| statusUpdateTime | string | Horodatage ISO8601 de la dernière mise à jour de l’état du jumeau d’appareil. |
| connectionState | string | Indique si l’appareil est connecté ou déconnecté. | 
| lastActivityTime | string | Horodatage ISO8601 de la dernière activité. | 
| cloudToDeviceMessageCount | entier | Nombre de messages cloud-à-appareil envoyés à cet appareil. | 
| authenticationType | string | Type d’authentification utilisé pour cet appareil : `SAS`, `SelfSigned` ou `CertificateAuthority`. |
| x509Thumbprint | string | L’empreinte numérique est une valeur unique pour le certificat x509, et sert généralement à rechercher un certificat particulier dans un magasin de certificats. L’empreinte numérique, générée dynamiquement à l’aide de l’algorithme SHA-1, n’existe pas physiquement dans le certificat. | 
| primaryThumbprint | string | Empreinte numérique principale pour le certificat x509. |
| secondaryThumbprint | string | Empreinte numérique secondaire pour le certificat x509. | 
| version | entier | Entier qui est incrémenté chaque fois que le jumeau d’appareil est mis à jour. |
| desired | object | Une partie des propriétés qui peuvent être écrites uniquement par le backend d’application et lues par l’appareil. | 
| reported | object | Une partie des propriétés qui peuvent être écrites uniquement par l’appareil et lues par le backend d’application. |
| lastUpdated | string | Horodatage ISO8601 de la dernière mise à jour de la propriété du jumeau d’appareil. | 

## <a name="tutorials-and-how-tos"></a>Tutoriels et articles de procédures
|Intitulé  |Description  |
|---------|---------|
| [Envoyer des notifications par e-mail sur des événements Azure IoT Hub à l’aide de Logic Apps](publish-iot-hub-events-to-logic-apps.md) | Une application logique envoie un e-mail de notification chaque fois qu’un appareil est ajouté à votre hub IoT. |
| [Réagir aux événements IoT Hub en utilisant Event Grid pour déclencher des actions](../iot-hub/iot-hub-event-grid.md) | Vue d’ensemble de l’intégration de hubs IoT à Event Grid. |
| [Commander des événements d’état de la connexion et de la déconnexion d’appareils](../iot-hub/iot-hub-how-to-order-connection-state-events.md) | Montre comment commander des événements d’état de la connexion d’appareils. |

## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation d’Azure Event Grid, consultez [Présentation d’Event Grid](overview.md).
* Pour en savoir plus sur le fonctionnement conjoint d’IoT Hub et d’Event Grid, consultez [Réagir aux événements IoT Hub en utilisant Event Grid pour déclencher des actions](../iot-hub/iot-hub-event-grid.md).