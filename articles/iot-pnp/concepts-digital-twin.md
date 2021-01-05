---
title: Comprendre les jumeaux numériques IoT Plug-and-Play
description: Comprendre comment IoT Plug-and-Play utilise les jumeaux numériques
author: prashmo
ms.author: prashmo
ms.date: 07/17/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: f13230c7bd88a9c3cf043fc1881a34f6b7ce6fe7
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95495319"
---
# <a name="understand-iot-plug-and-play-digital-twins"></a>Comprendre les jumeaux numériques IoT Plug-and-Play

Un appareil IoT Plug-and-Play implémente un modèle décrit par le schéma [DTDL (Digital Twins Definition Language)](https://github.com/Azure/opendigitaltwins-dtdl). Un modèle décrit l’ensemble des composants, propriétés, commandes et messages de télémétrie qu’un appareil particulier peut avoir. Un jumeau d’appareil et un jumeau numérique sont initialisés la première fois qu’un appareil IoT Plug-and-Play se connecte à un hub IoT.

IoT Plug-and-Play utilise DTDL version 2. Pour plus d’informations sur cette version, consultez la spécification [Digital Twins Definition Language (DTDL) – version 2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) sur GitHub.

DTDL n’est pas exclusif à IoT Plug-and-Play. D’autres services IoT, tels qu’[Azure Digital Twins](../digital-twins/overview.md), l’utilisent pour représenter des environnements entiers tels que des bâtiments et des réseaux énergétiques. Pour plus d’informations, consultez [Comprendre les modèles de jumeaux dans Azure Digital Twins](../digital-twins/concepts-models.md).

Cet article décrit de quelle façon les composants et les propriétés sont représentés dans les sections *Souhaité* et *Rapporté* d’un jumeau d’appareil. Il décrit également la façon dont ces concepts sont mappés au jumeau numérique correspondant.

L’appareil IoT Plug-and-Play de cet article implémente le [modèle Contrôleur de température](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) avec le composant [Thermostat](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json).

## <a name="device-twins-and-digital-twins"></a>Jumeaux d’appareil et jumeaux numériques

Les jumeaux d’appareil sont des documents JSON qui stockent des informations sur l’état des appareils (métadonnées, configurations et conditions). Pour en savoir plus, consultez [Comprendre et utiliser les jumeaux d’appareil dans IoT Hub](../iot-hub/iot-hub-devguide-device-twins.md). Les fabricants d’appareils et les développeurs de solutions peuvent continuer à utiliser le même ensemble d’API de jumeau d’appareil et de SDK pour implémenter des appareils et des solutions utilisant les conventions IoT Plug-and-Play.

Les API de jumeaux numériques opèrent sur des constructions de haut niveau en langage DTDL (Digital Twins Definition Language), telles que les composants, les propriétés et les commandes. Les API de jumeaux numériques permettent aux développeurs de solutions de créer plus facilement des solutions IoT Plug-and-Play.

Dans un jumeau d’appareil, l’état d’une propriété accessible en écriture est réparti entre les sections Souhaité et Rapporté. Toutes les propriétés en lecture seule sont accessibles dans la section Rapporté.

Dans un jumeau numérique, il existe une vue unifiée de l’état actuel et souhaité de la propriété. L’état de synchronisation d’une propriété donnée est stocké dans la section `$metadata` du composant par défaut correspondant.

### <a name="digital-twin-json-format"></a>Format JSON de jumeaux numériques

Lorsqu’il est représenté sous la forme d’un objet JSON, un jumeau numérique comprend les champs suivants :

| Nom du champ | Description |
| --- | --- |
| `$dtId` | Chaîne fournie par l’utilisateur représentant l’ID du jumeau numérique de l’appareil |
| `{propertyName}` | Valeur d’une propriété en JSON |
| `$metadata.$model` | [Facultatif] l’ID de l’interface de modèle qui caractérise ce jumeau numérique |
| `$metadata.{propertyName}.desiredValue` | [Uniquement pour les propriétés accessibles en écriture] Valeur souhaitée de la propriété spécifiée |
| `$metadata.{propertyName}.desiredVersion` | [Uniquement pour les propriétés accessibles en écriture] Version de la valeur souhaitée conservée par IoT Hub|
| `$metadata.{propertyName}.ackVersion` | [Obligatoire, uniquement pour les propriétés accessibles en écriture] Version reconnue par l’appareil qui implémente le jumeau numérique doit être supérieure ou égale à la version souhaitée |
| `$metadata.{propertyName}.ackCode` | [Obligatoire, uniquement pour les propriétés accessibles en écriture] Code `ack` retourné par l’application d’appareil qui implémente le jumeau numérique |
| `$metadata.{propertyName}.ackDescription` | [Facultatif, uniquement pour les propriétés accessibles en écriture] Description `ack` retournée par l’application d’appareil qui implémente le jumeau numérique |
| `$metadata.{propertyName}.lastUpdateTime` | IoT Hub conserve l’horodatage de la dernière mise à jour de la propriété par l’appareil. Les horodatages sont exprimés en UTC et codés au format ISO 8601 AAAA-MM-JJTHH:MM:SS.mmmZ. |
| `{componentName}` | Objet JSON contenant les valeurs de propriété et les métadonnées du composant. |
| `{componentName}.{propertyName}` | Valeur de la propriété du composant au format JSON |
| `{componentName}.$metadata` | Informations sur les métadonnées du composant. |

#### <a name="device-twin-sample"></a>Exemple de jumeau d’appareil

L’extrait de code suivant montre un jumeau d’appareil IoT Plug-and-Play formaté comme un objet JSON :

```json
{
    "deviceId": "sample-device",
    "modelId": "dtmi:com:example:TemperatureController;1",
    "version": 15,
    "properties": {
        "desired": {
            "thermostat1": {
                "__t": "c",
                "targetTemperature": 21.8
            },
            "$metadata": {...},
            "$version": 4
        },
        "reported": {
            "serialNumber": "alwinexlepaho8329",
            "thermostat1": {
                "maxTempSinceLastReboot": 25.3,
                "__t": "c",
                "targetTemperature": {
                    "value": 21.8,
                    "ac": 200,
                    "ad": "Successfully executed patch",
                }
            },
            "$metadata": {...},
            "$version": 11
        }
    }
}
```

#### <a name="digital-twin-sample"></a>Exemple de jumeau numérique

L’extrait de code suivant montre le jumeau numérique formaté comme un objet JSON :

```json
{
    "$dtId": "sample-device",
    "serialNumber": "alwinexlepaho8329",
    "thermostat1": {
        "maxTempSinceLastReboot": 25.3,
        "targetTemperature": 21.8,
        "$metadata": {
            "targetTemperature": {
                "desiredValue": 21.8,
                "desiredVersion": 4,
                "ackVersion": 4,
                "ackCode": 200,
                "ackDescription": "Successfully executed patch",
                "lastUpdateTime": "2020-07-17T06:11:04.9309159Z"
            },
            "maxTempSinceLastReboot": {
                "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
            }
        }
    },
    "$metadata": {
        "$model": "dtmi:com:example:TemperatureController;1",
        "serialNumber": {
            "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
        }
    }
}
```

### <a name="properties"></a>Propriétés

Les propriétés sont des champs de données représentant l’état d’une entité ; c’est par exemple le cas des propriétés dans de nombreux langages de programmation orientés objet.

#### <a name="read-only-property"></a>Propriété en lecture seule

Schéma :

```json
{
    "@type": "Property",
    "name": "serialNumber",
    "displayName": "Serial Number",
    "description": "Serial number of the device.",
    "schema": "string"
}
```

Dans cet exemple, `alwinexlepaho8329` est la valeur actuelle de la propriété en lecture seule `serialNumber` rapportée par l’appareil.
Les extraits de code suivants montrent la représentation JSON côte à côte de la propriété `serialNumber` :

:::row:::
   :::column span="":::
      **Jumeau d’appareil**

```json
"properties": {
    "reported": {
        "serialNumber": "alwinexlepaho8329"
    }
}
```

   :::column-end:::
   :::column span="":::
      **Jumeau numérique**

```json
"serialNumber": "alwinexlepaho8329"
```

   :::column-end:::
:::row-end:::

#### <a name="writable-property"></a>Propriété accessible en écriture

Supposons que l’appareil avait également la propriété accessible en écriture suivante dans le composant par défaut :

```json
{
    "@type": "Property",
    "name": "fanSpeed",
    "displayName": "Fan Speed",
    "writable": true,
    "schema": "double"
}
```

:::row:::
   :::column span="":::
      **Jumeau d’appareil**

```json
{
    "properties": {
        "desired": {
            "fanSpeed": 2.0,
        },
        "reported": {
            "fanSpeed": {
                "value": 3.0,
                "ac": 200,
                "av": 1,
                "ad": "Successfully executed patch version 1"
            }
        }
    },
}
```

   :::column-end:::
   :::column span="":::
      **Jumeau numérique**

```json
{
    "fanSpeed": 3.0,
    "$metadata": {
        "fanSpeed": {
            "desiredValue": 2.0,
            "desiredVersion": 2,
            "ackVersion": 1,
            "ackCode": 200,
            "ackDescription": "Successfully executed patch version 1",
            "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
        }
    }
}
```

   :::column-end:::
:::row-end:::

Dans cet exemple, `3.0` est la valeur actuelle de la propriété `fanSpeed` rapportée par l’appareil. `2.0` est la valeur souhaitée définie par la solution. La valeur souhaitée et l’état de synchronisation d’une propriété de niveau racine sont définis dans les `$metadata` de niveau racine d’un jumeau numérique. Lorsque l’appareil est en ligne, il peut appliquer cette mise à jour et renvoyer la valeur mise à jour.

### <a name="components"></a>Components

Les composants permettent de créer l’interface de modèle en tant qu’assembly d’autres interfaces.
Envisagez l’interface [Thermostat](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json), qui est définie comme modèle.
Cette interface peut désormais être incorporée en tant que composant thermostat1 (et un autre composant thermostat2) lors de la définition du [modèle Contrôleur de température](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json).

Dans un jumeau d’appareil, un composant est identifié par le marqueur `{ "__t": "c"}`. Dans un jumeau numérique, la présence de `$metadata` marque un composant.

Dans cet exemple, `thermostat1` est un composant avec deux propriétés :

- `maxTempSinceLastReboot` est une propriété en lecture seule.
- `targetTemperature` est une propriété accessible en écriture qui a été correctement synchronisée par l’appareil. La valeur souhaitée et l’état de synchronisation de ces propriétés se trouvent dans les `$metadata` du composant.

Les extraits de code suivants montrent la représentation JSON côte à côte du composant `thermostat1` :

:::row:::
   :::column span="":::
      **Jumeau d’appareil**

```json
"properties": {
    "desired": {
        "thermostat1": {
            "__t": "c",
            "targetTemperature": 21.8
        },
        "$metadata": {
        },
        "$version": 4
    },
    "reported": {
        "thermostat1": {
            "maxTempSinceLastReboot": 25.3,
            "__t": "c",
            "targetTemperature": {
                "value": 21.8,
                "ac": 200,
                "ad": "Successfully executed patch",
                "av": 4
            }
        },
        "$metadata": {
        },
        "$version": 11
    }
}
```

   :::column-end:::
   :::column span="":::
      **Jumeau numérique**

```json
"thermostat1": {
    "maxTempSinceLastReboot": 25.3,
    "targetTemperature": 21.8,
    "$metadata": {
        "targetTemperature": {
            "desiredValue": 21.8,
            "desiredVersion": 4,
            "ackVersion": 4,
            "ackCode": 200,
            "ackDescription": "Successfully executed patch",
            "lastUpdateTime": "2020-07-17T06:11:04.9309159Z"
        },
        "maxTempSinceLastReboot": {
            "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
        }
    }
}
```

   :::column-end:::
:::row-end:::

## <a name="digital-twin-apis"></a>API de jumeaux numériques

Azure Digital Twins est fourni avec les opérations **Obtenir le jumeau numérique**, **Mettre à jour un jumeau numérique**, **Appeler la commande d’un composant** et **Appeler une commande** pour gérer le jumeau numérique de l’appareil. Vous pouvez utiliser les [API REST](/rest/api/iothub/service/digitaltwin) directement ou par le biais d’un [Kit de développement logiciel (SDK) de service](../iot-pnp/libraries-sdks.md).

## <a name="digital-twin-change-events"></a>Événements de changement de jumeau numérique

Lorsque les événements de changement de jumeau numérique sont activés, un événement est déclenché chaque fois que la valeur actuelle ou souhaitée du composant ou de la propriété change. Les événements de changement de jumeau numérique sont générés au format [JSON Patch](http://jsonpatch.com/). Les événements correspondants sont générés au format du jumeau d’appareil si les événements de changement du jumeau sont activés.

Pour savoir comment activer le routage pour les événements des jumeaux d’appareil et numériques, consultez [Utiliser le routage des messages IoT Hub pour envoyer des messages appareil-à-cloud à différents points de terminaison](../iot-hub/iot-hub-devguide-messages-d2c.md#non-telemetry-events). Pour en savoir plus sur le format du message, consultez [Créer et lire des messages IoT Hub](../iot-hub/iot-hub-devguide-messages-construct.md).

Par exemple, l’événement de changement de jumeau numérique suivant est déclenché lorsque `targetTemperature` est définie par la solution :

```json
iothub-connection-device-id:sample-device
iothub-enqueuedtime:7/17/2020 6:11:04 AM
iothub-message-source:digitalTwinChangeEvents
correlation-id:275d463fa034
content-type:application/json-patch+json
content-encoding:utf-8
[
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature",
    "value": {
      "desiredValue": 21.8,
      "desiredVersion": 4
    }
  }
]
```

L’événement de changement du jumeau numérique suivant est déclenché lorsque l’appareil signale que la modification souhaitée précédente a été appliquée :

```json
iothub-connection-device-id:sample-device
iothub-enqueuedtime:7/17/2020 6:11:05 AM
iothub-message-source:digitalTwinChangeEvents
correlation-id:275d464a2c80
content-type:application/json-patch+json
content-encoding:utf-8
[
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/ackCode",
    "value": 200
  },
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/ackDescription",
    "value": "Successfully executed patch"
  },
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/ackVersion",
    "value": 4
  },
  {
    "op": "add",
    "path": "/thermostat1/$metadata/targetTemperature/lastUpdateTime",
    "value": "2020-07-17T06:11:04.9309159Z"
  },
  {
    "op": "add",
    "path": "/thermostat1/targetTemperature",
    "value": 21.8
  }
]
```

> [!NOTE]
> Les messages de notification de changement de jumeau sont doublés lorsqu’ils sont activés à la fois dans les notifications de modification d’appareil et de jumeau numérique.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert les jumeaux numériques, voici quelques ressources supplémentaires :

- [Comment utiliser les API de jumeaux numériques d’IoT Plug-and-Play](howto-manage-digital-twin.md)
- [Interagir avec un appareil à partir de votre solution](quickstart-service.md)
- [API REST de jumeau numérique IoT](/rest/api/iothub/service/digitaltwin)
- [Explorateur Azure IoT](howto-use-iot-explorer.md)