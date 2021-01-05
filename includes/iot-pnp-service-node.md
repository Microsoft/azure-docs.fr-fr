---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 7ab17f0d34ba7682778120e11aab562e106b5df7
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95487788"
---
IoT Plug-and-Play simplifie l’IoT en vous permettant d’interagir avec les fonctionnalités d’un appareil sans avoir connaissance de l’implémentation de l’appareil sous-jacent. Ce démarrage rapide vous montre comment utiliser Node.js pour vous connecter à un appareil IoT Plug-and-Play connecté à votre solution et le contrôler.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Pour suivre ce guide de démarrage rapide, vous avez besoin de Node.js sur votre ordinateur de développement. Vous pouvez télécharger la dernière version recommandée pour plusieurs plateformes à partir de [nodejs.org](https://nodejs.org).

Vous pouvez vérifier la version actuelle de Node.js sur votre machine de développement à l’aide de la commande suivante :

```cmd/sh
node --version
```

### <a name="clone-the-sdk-repository-with-the-sample-code"></a>Clonez le référentiel SDK avec l’exemple de code

Clonez les exemples à partir du [dépôt SDK Node](https://github.com/Azure/azure-iot-sdk-node). Ouvrez une fenêtre de terminal dans un dossier de votre choix. Exécutez la commande suivante pour cloner le dépôt GitHub [SDK Microsoft Azure IoT pour Node.js](https://github.com/Azure/azure-iot-sdk-node) :

```cmd/sh
git clone https://github.com/Azure/azure-iot-sdk-node
```

## <a name="run-the-sample-device"></a>Exécuter l’exemple d’appareil

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Pour en savoir plus sur l’exemple de configuration, consultez l’[exemple de fichier Lisez-moi](https://github.com/Azure/azure-iot-sdk-node/blob/master/device/samples/pnp/readme.md).

Dans ce guide de démarrage rapide, vous utilisez un exemple de thermostat écrit en Node.js en tant qu’appareil IoT Plug-and-Play. Pour exécuter l’exemple d’appareil :

1. Ouvrez une fenêtre de terminal, puis accédez au dossier local contenant le référentiel SDK Microsoft Azure IoT pour Node.js que vous avez cloné depuis GitHub.

1. Cette fenêtre de terminal est utilisée en tant que terminal **device**. Accédez au dossier de votre dépôt cloné, puis accédez au dossier */azure-iot-sdk-node/device/samples/pnp*. Installez toutes les dépendances en exécutant la commande suivante :

    ```cmd/sh
    npm install
    ```

1. Pour exécutez l’exemple d’appareil à thermostat, utilisez la commande suivante :

    ```cmd/sh
    node simple_thermostat.js
    ```

1. Des messages s’affichent pour indiquer que l’appareil a envoyé des informations et qu’il s’est signalé comme étant en ligne. Ces messages indiquent que l’appareil a commencé à envoyer des données de télémétrie au hub et est maintenant prêt à recevoir des commandes et des mises à jour de propriétés. Ne fermez pas ce terminal. Vous en aurez besoin pour confirmer que l’exemple de service fonctionne également.

## <a name="run-the-sample-solution"></a>Exécuter l’exemple de solution

Dans [Configurer votre environnement pour les guides de démarrage rapide et tutoriels IoT Plug-and-Play](../articles/iot-pnp/set-up-environment.md), vous avez créé deux variables d’environnement pour configurer l’exemple afin qu’il se connecte à votre hub IoT et à votre appareil :

* **IOTHUB_CONNECTION_STRING** : la chaîne de connexion de hub IoT que vous avez notée précédemment.
* **IOTHUB_DEVICE_ID** : `"my-pnp-device"`.

Dans ce guide de démarrage rapide, vous utilisez un exemple de solution IoT en Node.js pour interagir avec l’exemple d’appareil que vous venez de configurer.

1. Ouvrez une autre fenêtre de terminal à utiliser comme terminal **service**.

1. Dans le dépôt SDK Node cloné, accédez au dossier */azure-iot-sdk-node/service/samples/javascript*. Installez toutes les dépendances en exécutant la commande suivante :

    ```cmd/sh
    npm install
    ```

### <a name="read-a-property"></a>Lire une propriété

1. Lorsque vous avez exécuté l’exemple de thermostat dans le terminal de l’**​​appareil**, vous avez vu les messages suivants indiquant son état en ligne :

    ```cmd/sh
    properties have been reported for component
    sending telemetry message 0...
    ```

1. Accédez au terminal de **service** et utilisez la commande suivante pour exécuter l’exemple de lecture des informations de l’appareil :

    ```cmd/sh
    node twin.js
    ```

1. Dans la sortie de terminal du **service**, notez la réponse du jumeau d’appareil. Vous voyez l’ID de modèle d’appareil et les propriétés associées signalées :

    ```json
    Model Id: dtmi:com:example:Thermostat;1
    {
      "deviceId": "my-pnp-device",
      "etag": "AAAAAAAAAAE=",
      "deviceEtag": "Njc3MDMxNDcy",
      "status": "enabled",
      "statusUpdateTime": "0001-01-01T00:00:00Z",
      "connectionState": "Connected",
      "lastActivityTime": "0001-01-01T00:00:00Z",
      "cloudToDeviceMessageCount": 0,
      "authenticationType": "sas",
      "x509Thumbprint": {
        "primaryThumbprint": null,
        "secondaryThumbprint": null
      },
      "modelId": "dtmi:com:example:Thermostat;1",
      "version": 4,
      "properties": {
        "desired": {
          "$metadata": {
            "$lastUpdated": "2020-10-05T11:35:19.4574755Z"
          },
          "$version": 1
        },
        "reported": {
          "maxTempSinceLastReboot": 31.343640523762232,
          "serialNumber": "123abc",
          "$metadata": {
            "$lastUpdated": "2020-10-05T11:35:23.7339042Z",
            "maxTempSinceLastReboot": {
              "$lastUpdated": "2020-10-05T11:35:23.7339042Z"
            },
            "serialNumber": {
              "$lastUpdated": "2020-10-05T11:35:23.7339042Z"
            }
          },
          "$version": 3
        }
      },
      "capabilities": {
        "iotEdge": false
      },
      "tags": {}
    }
    ```

1. L’extrait de code suivant montre le code dans *twin.js* qui extrait l’ID de modèle du jumeau d’appareil :

    ```javascript
    var registry = Registry.fromConnectionString(connectionString);
    registry.getTwin(deviceId, function(err, twin) {
      if (err) {
        console.error(err.message);
      } else {
        console.log('Model Id: ' + twin.modelId);
        //...
      }
      //...
    }
    ```

Dans ce scénario, il génère la sortie suivante : `Model Id: dtmi:com:example:Thermostat;1`.

> [!NOTE]
> Ces exemples de services utilisent la classe **Registry** à partir du **client du service IoT Hub**. Pour en savoir plus sur les API, y compris l’API Digital Twins, consultez le [Guide du développeur sur les services](../articles/iot-pnp/concepts-developer-guide-service.md).

### <a name="update-a-writable-property"></a>Mettre à jour une propriété accessible en écriture

1. Ouvrez le fichier *twin.js* dans un éditeur de code.

1. Examinez l’exemple de code ; il vous montre deux façons de mettre à jour le jumeau d’appareil. Pour utiliser la première méthode, modifiez la variable `twinPatch` comme suit :

    ```javascript
    var twinPatch = {
      tags: {
        city: "Redmond"
      },
      properties: {
        desired: {
          targetTemperature: 42
        }
      }
    };
    ```

    La propriété `targetTemperature` est définie en tant que propriété accessible en écriture dans le modèle d’appareil à thermostat.

1. Dans le terminal du **service**, utilisez la commande suivante pour exécuter l’exemple de mise à jour de la propriété :

    ```cmd/sh
    node twin.js
    ```

1. Dans votre terminal d’**appareil**, vous voyez que l’appareil a reçu la mise à jour :

    ```cmd/sh
    The following properties will be updated for the default component:
    {
      targetTemperature: {
        value: 42,
        ac: 200,
        ad: 'Successfully executed patch for targetTemperature',
        av: 2
      }
    }
    updated the property
    ```

1. Dans votre terminal de **service**, exécutez la commande suivante pour confirmer la mise à jour de la propriété :

    ```cmd/sh
    node twin.js
    ```

1. Dans la sortie de terminal du **service**, dans la section des propriétés `reported`, vous voyez la température de la cible mise à jour signalée. L’appareil peut prendre un certain temps pour terminer la mise à jour. Répétez cette étape jusqu’à ce que l’appareil ait traité la mise à jour de propriété.

    ```json
    "reported": {
      //...
      "targetTemperature": {
        "value": 42,
        "ac": 200,
        "ad": "Successfully executed patch for targetTemperature",
        "av": 4
      },
      //...
    }
    ```

### <a name="invoke-a-command"></a>Appeler une commande

1. Ouvrez le fichier *device_method.js* et passez en revue le code.

1. Accédez au terminal de **service**. Utilisez la commande suivante pour exécuter exemple d’appel de la commande :

    ```cmd/sh
    set IOTHUB_METHOD_NAME=getMaxMinReport
    set IOTHUB_METHOD_PAYLOAD=commandpayload
    node device_method.js
    ```

1. La sortie du terminal de **service** montre la confirmation suivante :

    ```cmd/sh
    getMaxMinReport on my-pnp-device:
    {
      "status": 200,
      "payload": {
        "maxTemp": 23.460596940801928,
        "minTemp": 23.460596940801928,
        "avgTemp": 23.460596940801928,
        "endTime": "2020-10-05T12:48:08.562Z",
        "startTime": "2020-10-05T12:47:54.450Z"
      }
    }
    ```

1. Dans le terminal de l’**appareil**, vous voyez que la commande a été reconnue :

    ```cmd/sh
    MaxMinReport commandpayload
    Response to method 'getMaxMinReport' sent successfully.
    ```
