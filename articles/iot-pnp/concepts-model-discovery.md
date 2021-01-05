---
title: Utiliser les modèles IoT Plug-and-Play dans une solution | Microsoft Docs
description: Cet article destiné aux générateurs de solutions explique comment utiliser les modèles IoT Plug-and-Play dans une solution IoT.
author: arunmannengal
ms.author: arunmann
ms.date: 07/23/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 5f82206a7c86c5ae1e6477419faee0dca88ef883
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95495285"
---
# <a name="use-iot-plug-and-play-models-in-an-iot-solution"></a>Utiliser les modèles IoT Plug-and-Play dans une solution IoT

Cet article explique comment, dans une solution IoT, vous pouvez identifier l’ID de modèle d’un appareil IoT Plug-and-Play, puis récupérer sa définition de modèle.

Il existe deux grandes catégories de solution IoT :

- Une *solution spécialisée* fonctionne avec un ensemble connu de modèles pour les appareils IoT Plug-and-Play qui se connecteront à la solution. Vous utilisez ces modèles lorsque vous développez la solution.

- Une solution *pilotée par modèle* peut fonctionner avec le modèle de n’importe quel appareil IoT Plug-and-Play. La création d’une solution pilotée par modèle est plus complexe, mais ce type de solution présente l’avantage de fonctionner avec tous les appareils susceptibles d’être ajoutés à l’avenir. Une solution IoT pilotée par modèle récupère un modèle et l’utilise pour déterminer les données de télémétrie, les propriétés et les commandes implémentées par l’appareil.

Pour utiliser un modèle IoT Plug-and-Play, une solution IoT :

1. Identifie l’ID de modèle du modèle implémenté par l’appareil IoT Plug-and-Play, module ou module IoT Edge connecté à la solution.

1. Utilise cet ID de modèle pour récupérer la définition de modèle de l’appareil connecté à partir d’un référentiel de modèles ou d’un magasin personnalisé.

## <a name="identify-model-id"></a>Identifier l’ID de modèle

Lorsqu’un appareil IoT Plug-and-Play se connecte à IoT Hub, il inscrit l’ID de modèle du modèle qu’il implémente avec IoT Hub.

IoT Hub notifie la solution avec l’ID de modèle d’appareil dans le cadre du processus de connexion de l’appareil.

Une solution peut obtenir l’ID de modèle de l’appareil IoT Plug-and-Play à l’aide de l’une des trois méthodes suivantes :

### <a name="get-device-twin-api"></a>API Obtenir un jumeau d’appareil

La solution peut utiliser l’API [Obtenir un jumeau d’appareil](/java/api/com.microsoft.azure.sdk.iot.device.deviceclient.getdevicetwin?preserve-view=true&view=azure-java-stable) pour récupérer l’ID de modèle de l’appareil IoT Plug-and-Play.

> [!TIP]
> Pour les modules et les modules IoT Edge, utilisez [ModuleClient. getTwin](/java/api/com.microsoft.azure.sdk.iot.device.moduleclient.gettwin?preserve-view=true&view=azure-java-stable).

Dans l’extrait de code suivant de réponse de jumeau d’appareil, `modelId` contient l’ID de modèle d’un appareil IoT Plug-and-Play :

```json
{
    "deviceId": "sample-device",
    "etag": "AAAAAAAAAAc=",
    "deviceEtag": "NTk0ODUyODgx",
    "status": "enabled",
    "statusUpdateTime": "0001-01-01T00:00:00Z",
    "connectionState": "Disconnected",
    "lastActivityTime": "2020-07-17T06:12:26.8402249Z",
    "cloudToDeviceMessageCount": 0,
    "authenticationType": "sas",
    "x509Thumbprint": {
        "primaryThumbprint": null,
        "secondaryThumbprint": null
    },
    "modelId": "dtmi:com:example:TemperatureController;1",
    "version": 15,
    "properties": {...}
    }
}
```

### <a name="get-digital-twin-api"></a>API Obtenir le jumeau numérique

La solution peut utiliser l’API [Obtenir le jumeau numérique](/rest/api/iothub/service/digitaltwin/getdigitaltwin) pour récupérer l’ID de modèle du modèle implémenté par l’appareil IoT Plug-and-Play.

Dans l’extrait de code suivant de réponse de jumeau numérique, `$metadata.$model` contient l’ID de modèle d’un appareil IoT Plug-and-Play :

```json
{
    "$dtId": "sample-device",
    "$metadata": {
        "$model": "dtmi:com:example:TemperatureController;1",
        "serialNumber": {
            "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
        }
    }
}
```

### <a name="digital-twin-change-event-notification"></a>Notification d’événement de changement du jumeau numérique

Une connexion d’appareil génère une notification [Événement de changement du jumeau numérique](concepts-digital-twin.md#digital-twin-change-events). Une solution doit s’abonner à cette notification d’événement. Pour savoir comment activer l’acheminement des événements des jumeaux numériques, consultez [Envoi de messages appareil-à-cloud à différents points de terminaison à l’aide de l’acheminement des messages IoT Hub](../iot-hub/iot-hub-devguide-messages-d2c.md#non-telemetry-events).

La solution peut utiliser l’événement illustré dans l’extrait de code suivant pour en savoir plus sur l’appareil IoT Plug-and-Play qui se connecte et obtenir son ID de modèle :

```json
iothub-connection-device-id:sample-device
iothub-enqueuedtime:7/22/2020 8:02:27 PM
iothub-message-source:digitalTwinChangeEvents
correlation-id:100f322dc2c5
content-type:application/json-patch+json
content-encoding:utf-8
[
  {
    "op": "replace",
    "path": "/$metadata/$model",
    "value": "dtmi:com:example:TemperatureController;1"
  }
]
```

## <a name="retrieve-a-model-definition"></a>Récupérer une définition de modèle

Une solution utilise l’ID de modèle identifié ci-dessus pour récupérer la définition de modèle correspondante.

Une solution peut obtenir la définition de modèle en utilisant l’une des options suivantes :

### <a name="model-repository"></a>Référentiel de modèles

Les solutions peuvent utiliser le [référentiel de modèles](concepts-model-repository.md) pour récupérer des modèles. Les fabricants d’appareils ou les créateurs de solutions doivent charger à l’avance leurs modèles dans le référentiel pour que la solution puisse les récupérer.

Une fois que vous avez identifié l’ID de modèle pour une nouvelle connexion d’appareil, procédez comme suit :

1. Récupérez la définition de modèle à l’aide de l’ID de modèle à partir du référentiel de modèles. Pour plus d’informations, consultez [Référentiel de modèles d’appareil](concepts-model-repository.md).

1. Avec la définition de modèle de l’appareil connecté, vous pouvez énumérer les fonctionnalités de l’appareil.

1. Avec les fonctionnalités énumérées de l’appareil, vous pouvez autoriser les utilisateurs à [interagir avec l’appareil](quickstart-service.md).

### <a name="custom-store"></a>Magasin personnalisé

Les solutions peuvent stocker ces définitions de modèle dans un système de fichiers local ou un magasin de fichiers public, ou encore utiliser une implémentation personnalisée.

Une fois que vous avez identifié l’ID de modèle pour une nouvelle connexion d’appareil, procédez comme suit :

1. Récupérez la définition de modèle à l’aide de l’ID de modèle à partir du magasin personnalisé.

1. Avec la définition de modèle de l’appareil connecté, vous pouvez énumérer les fonctionnalités de l’appareil. 

1. Avec les fonctionnalités énumérées de l’appareil, vous pouvez autoriser les utilisateurs à [interagir avec l’appareil](quickstart-service.md).  

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à intégrer les modèles IoT Plug-and-Play dans une solution IoT, voici quelques étapes ultérieures suggérées :

- [Interaction avec un appareil à partir de la solution](quickstart-service.md)
- [API REST de jumeau numérique IoT](/rest/api/iothub/service/digitaltwin)
- [Explorateur Azure IoT](howto-use-iot-explorer.md)
