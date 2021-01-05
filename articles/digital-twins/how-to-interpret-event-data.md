---
title: Interpréter les données d’événement
titleSuffix: Azure Digital Twins
description: Découvrez comment interpréter différents types d’événements et leurs messages de notification.
author: baanders
ms.author: baanders
ms.date: 6/23/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 22bedcf7921e3c8d4f2566a70515eef3e3b136b6
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92461020"
---
# <a name="understand-event-data"></a>Comprendre les données d’événements

Dans Azure Digital Twins, plusieurs événements peuvent produire des **notifications** . Celles-ci permettent au back-end de la solution de savoir quand des actions se produisent. Elles sont ensuite [routées](concepts-route-events.md) vers différents emplacements, à l’intérieur et à l’extérieur d’Azure Digital Twins, qui peuvent utiliser ces informations pour prendre des mesures.

Plusieurs types de notifications peuvent être générés. En outre, les messages de notification peuvent être différents selon le type d’événement qui les a générés. Cet article donne des détails sur les différents types de messages, ainsi que sur leur aspect.

Ce graphique montre les différents types de notifications :

[!INCLUDE [digital-twins-notifications.md](../../includes/digital-twins-notifications.md)]

En général, les notifications sont constituées de deux parties : l’en-tête et le corps. 

### <a name="event-notification-headers"></a>En-têtes des notifications d’événements

Les en-têtes de message de notification sont représentés par des paires clé-valeur. Les en-têtes de message sont sérialisés différemment, selon le protocole utilisé (MQTT, AMQP ou HTTP). Cette section donne des informations générales sur les en-têtes des messages de notification, sans prendre en compte le protocole et la sérialisation choisis.

Certaines notifications sont conformes à la norme [CloudEvents](https://cloudevents.io/). La conformité CloudEvents est la suivante.
* Les notifications émises par les appareils continuent de suivre les spécifications existantes concernant les notifications.
* Les notifications traitées et émises par IoT Hub continuent de suivre les spécifications existantes concernant les notifications, sauf lorsque IoT Hub choisit de prendre en charge CloudEvents, par exemple, via Event Grid.
* Les notifications émises par des [jumeaux numériques](concepts-twins-graph.md) avec un [modèle](concepts-models.md) se conforment à CloudEvents.
* Les notifications traitées et émises par Azure Digital Twins sont conformes à CloudEvents.

Les services doivent ajouter un numéro séquentiel à toutes les notifications pour indiquer l’ordre dans lequel elles doivent s’afficher, ou sinon, les organiser d’une autre façon. 

Les notifications émises par Azure Digital Twins pour Event Grid sont automatiquement mises en forme selon le schéma CloudEvents ou le schéma EventGridEvent, en fonction du type de schéma défini dans la rubrique Event Grid. 

Les attributs d’extension des en-têtes sont ajoutés en tant que propriétés dans le schéma Event Grid, à l’intérieur de la charge utile. 

### <a name="event-notification-bodies"></a>Corps des notifications d’événements

Le corps des messages de notification est présenté ici en JSON. Le corps du message peut être sérialisé différemment selon la sérialisation choisie (par exemple, avec JSON, CBOR, Protobuf, etc).

L’ensemble de champs que le corps contient varie en fonction du type de notification. Pour avoir une idée de ce à quoi ils ressemblent et de ce qu’ils peuvent inclure, voici deux exemples de corps de message.

Message de télémétrie :

```json
{
  "specversion": "1.0",
  "id": "df5a5992-817b-4e8a-b12c-e0b18d4bf8fb",
  "type": "microsoft.iot.telemetry",
  "source": "contoso-adt.api.wus2.digitaltwins.azure.net/digitaltwins/room1",
  "data": {
    "Temperature": 10
  },
  "dataschema": "dtmi:example:com:floor4;2",
  "datacontenttype": "application/json",
  "traceparent": "00-7e3081c6d3edfb4eaf7d3244b2036baa-23d762f4d9f81741-01"
}
```

Message de notification concernant le cycle de vie :

```json
{
  "specversion": "1.0",
  "id": "d047e992-dddc-4a5a-b0af-fa79832235f8",
  "type": "Microsoft.DigitalTwins.Twin.Create",
  "source": "contoso-adt.api.wus2.digitaltwins.azure.net",
  "data": {
    "$dtId": "floor1",
    "$etag": "W/\"e398dbf4-8214-4483-9d52-880b61e491ec\"",
    "$metadata": {
      "$model": "dtmi:example:Floor;1"
    }
  },
  "subject": "floor1",
  "time": "2020-06-23T19:03:48.9700792Z",
  "datacontenttype": "application/json",
  "traceparent": "00-18f4e34b3e4a784aadf5913917537e7d-691a71e0a220d642-01"
}
```

## <a name="message-format-detail-for-different-event-types"></a>Informations sur la mise en forme des messages selon le type d’événement

Cette section décrit en détail les différents types de notifications émises par IoT Hub et Azure Digital Twins (ou d’autres services Azure IoT). Vous verrez ce qui déclenche chaque type de notification, et vous verrez l’ensemble de champs qui est inclus avec chaque type de corps de notification.

### <a name="digital-twin-life-cycle-notifications"></a>Notifications de cycle de vie pour les jumeaux numériques

Tous les [jumeaux numériques](concepts-twins-graph.md) émettent des notifications, qu’ils représentent ou non des [appareils IoT Hub dans Azure Digital Twins](how-to-ingest-iot-hub-data.md). Ceci est dû aux **notifications de cycle de vie** , qui concernent uniquement le jumeau numérique.

Les notifications de cycle de vie sont déclenchées dans les cas suivants :
* La création d’un jumeau numérique
* La suppression d’un jumeau numérique

#### <a name="properties"></a>Propriétés

Voici les champs compris dans le corps d’une notification de cycle de vie.

| Nom | Valeur |
| --- | --- |
| `id` | Identificateur de la notification, tel qu’un UUID ou un compteur géré par le service. `source` + `id` est unique pour chaque événement. |
| `source` | Nom de l’instance IoT Hub ou Azure Digital Twins, par exemple *myhub.azure-devices.net* ou *mydigitaltwins.westus2.azuredigitaltwins.net* |
| `specversion` | *1.0*<br>Le message est conforme à cette version de la [spécification CloudEvents](https://github.com/cloudevents/spec). |
| `type` | `Microsoft.DigitalTwins.Twin.Create`<br>`Microsoft.DigitalTwins.Twin.Delete` |
| `datacontenttype` | `application/json` |
| `subject` | ID du jumeau numérique |
| `time` | Horodatage du moment où l’opération s’est produite au niveau du jumeau |
| `traceparent` | Contexte de trace W3C pour l’événement |

#### <a name="body-details"></a>Détails du corps

Le corps correspond au jumeau numérique affecté, représenté au format JSON. Le schéma est le suivant : *Digital Twins Resource 7.1* .

Pour les événements de création, la charge utile reflète l’état du jumeau après la création de la ressource. Par conséquent, elle doit inclure tous les éléments générés par le système, tout comme un appel `GET`.

Voici un exemple de corps pour un appareil [IoT Plug-and-Play (PnP)](../iot-pnp/overview-iot-plug-and-play.md), avec des composants et aucune propriété de niveau supérieur. Les propriétés qui n’ont pas d’utilité pour les appareils (telles que les propriétés signalées) doivent être omises.

```json
{
  "$dtId": "device-digitaltwin-01",
  "$etag": "W/\"e59ce8f5-03c0-4356-aea9-249ecbdc07f9\"",
  "thermostat": {
    "temperature": 80,
    "humidity": 45,
    "$metadata": {
      "$model": "dtmi:com:contoso:Thermostat;1",
      "temperature": {
        "desiredValue": 85,
        "desiredVersion": 3,
        "ackVersion": 2,
        "ackCode": 200,
        "ackDescription": "OK"
      },
      "humidity": {
        "desiredValue": 40,
        "desiredVersion": 1,
        "ackVersion": 1,
        "ackCode": 200,
        "ackDescription": "OK"
      }
    }
  },
  "$metadata": {
    "$model": "dtmi:com:contoso:Thermostat_X500;1",
  }
}
```

Voici un exemple de jumeau numérique. Celui-ci est basé sur un [modèle](concepts-models.md) et ne prend pas en charge les composants :

```json
{
  "$dtId": "logical-digitaltwin-01",
  "$etag": "W/\"e59ce8f5-03c0-4356-aea9-249ecbdc07f9\"",
  "avgTemperature": 70,
  "comfortIndex": 85,
  "$metadata": {
    "$model": "dtmi:com:contoso:Building;1",
    "avgTemperature": {
      "desiredValue": 72,
      "desiredVersion": 5,
      "ackVersion": 4,
      "ackCode": 200,
      "ackDescription": "OK"
    },
    "comfortIndex": {
      "desiredValue": 90,
      "desiredVersion": 1,
      "ackVersion": 3,
      "ackCode": 200,
      "ackDescription": "OK"
    }
  }
}
```

### <a name="digital-twin-relationship-change-notifications"></a>Notifications de modification d’une relation de jumeau numérique

Les **notifications de modification de relation** sont déclenchées lorsqu’une relation d’un jumeau numérique est créée, mise à jour ou supprimée. 

#### <a name="properties"></a>Propriétés

Voici les champs compris dans le corps d’une notification de modification de périphérie.

| Nom    | Valeur |
| --- | --- |
| `id` | Identificateur de la notification, tel qu’un UUID ou un compteur géré par le service. `source` + `id` est unique pour chaque événement. |
| `source` | Nom de l’instance Azure Digital Twins, comme *mydigitaltwins.westus2.azuredigitaltwins.net* |
| `specversion` | *1.0*<br>Le message est conforme à cette version de la [spécification CloudEvents](https://github.com/cloudevents/spec). |
| `type` | `Microsoft.DigitalTwins.Relationship.Create`<br>`Microsoft.DigitalTwins.Relationship.Update`<br>`Microsoft.DigitalTwins.Relationship.Delete`
|`datacontenttype`| `application/json` |
| `subject` | ID de la relation, comme `<twinID>/relationships/<relationshipID>` |
| `time` | Horodatage du moment où l’opération s’est produite au niveau de la relation |
| `traceparent` | Contexte de trace W3C pour l’événement |

#### <a name="body-details"></a>Détails du corps

Le corps est la charge utile d’une relation, et se présente au format JSON. Il utilise le même format qu’une requête `GET` pour une relation via l’[API DigitalTwins](/rest/api/digital-twins/dataplane/twins). 

Le message « Updating a relationship » (Mise à jour d’une relation) signifie que les propriétés de la relation ont été modifiées. 

Voici un exemple de notification de mise à jour de relation concernant la mise à jour d’une propriété :

```json
{
    "modelId": "dtmi:example:Floor;1",
    "patch": [
      {
        "value": "user3",
        "path": "/ownershipUser",
        "op": "replace"
      }
    ]
  }
```

Pour `Relationship.Delete`, le corps est le même que celui de la requête `GET`, et il obtient l’état le plus récent avant la suppression.

Voici un exemple de notification de création ou de suppression de relation :

```json
{
    "$relationshipId": "device_to_device",
    "$etag": "W/\"72479873-0083-41a8-83e2-caedb932d881\"",
    "$relationshipName": "Connected",
    "$targetId": "device2",
    "connectionType": "WIFI"
}
```

### <a name="digital-twin-change-notifications"></a>Notification de changement du jumeau numérique

Les **notifications de changement de jumeau numérique** sont déclenchées lors de la mise à jour d’un jumeau numérique, par exemple :
* Lorsque les valeurs de propriété ou les métadonnées sont modifiées.
* En cas de modification des métadonnées d’un jumeau numérique ou d’un composant. C’est le cas, par exemple, lorsque vous changez le modèle d’un jumeau numérique.

#### <a name="properties"></a>Propriétés

Voici les champs compris dans le corps d’une notification de modification d’un jumeau numérique.

| Nom    | Valeur |
| --- | --- |
| `id` | Identificateur de la notification, tel qu’un UUID ou un compteur géré par le service. `source` + `id` est unique pour chaque événement. |
| `source` | Nom de l’instance IoT Hub ou Azure Digital Twins, par exemple *myhub.azure-devices.net* ou *mydigitaltwins.westus2.azuredigitaltwins.net*
| `specversion` | *1.0*<br>Le message est conforme à cette version de la [spécification CloudEvents](https://github.com/cloudevents/spec). |
| `type` | `Microsoft.DigitalTwins.Twin.Update` |
| `datacontenttype` | `application/json` |
| `subject` | ID du jumeau numérique |
| `time` | Horodatage du moment où l’opération s’est produite au niveau du jumeau numérique |
| `traceparent` | Contexte de trace W3C pour l’événement |

#### <a name="body-details"></a>Détails du corps

Le corps de la notification `Twin.Update` est un document de correctif JSON contenant la mise à jour du jumeau numérique.

Par exemple, imaginons qu’un jumeau numérique ait été mis à jour à l’aide du correctif suivant.

```json
[
    {
        "op": "replace",
        "value": 40,
        "path": "/Temperature"
    },
    {
        "op": "add",
        "value": 30,
        "path": "/comp1/prop1"
    }
]
```

La notification correspondante (si elle est exécutée de façon synchrone par le service, comme lorsqu’Azure Digital Twins met à jour un jumeau numérique) aurait un corps comme le suivant :

```json
{
    "modelId": "dtmi:example:com:floor4;2",
    "patch": [
      {
        "value": 40,
        "path": "/Temperature",
        "op": "replace"
      },
      {
        "value": 30,
        "path": "/comp1/prop1",
        "op": "add"
      }
    ]
  }
```

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment créer des points de terminaison et des routes pour remettre des événements :
* [*Guide pratique : Gérer les points de terminaison et les itinéraires*](how-to-manage-routes-apis-cli.md)

Pour plus d’informations sur les API Azure Digital Twins et les options du SDK, consultez :
* [*Guide pratique : Utiliser les API et les Kits de développement logiciel (SDK) Azure Digital Twins*](how-to-use-apis-sdks.md)