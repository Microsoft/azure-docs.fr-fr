---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 6b861baea93c2c57b8f66ebac928a7e4fd3adfa8
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95487784"
---
IoT Plug-and-Play simplifie l’IoT en vous permettant d’interagir avec le modèle d’un appareil sans avoir connaissance de l’implémentation de l’appareil sous-jacent. Ce guide de démarrage rapide vous montre comment utiliser Python pour vous connecter à un appareil IoT Plug-and-Play connecté à votre solution et le contrôler.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Pour suivre ce guide de démarrage rapide, vous avez besoin de Python 3.7 sur votre ordinateur de développement. Vous pouvez télécharger la dernière version recommandée pour plusieurs plateformes à partir de [python.org](https://www.python.org/). Vous pouvez vérifier votre version de Python à l’aide de la commande suivante :  

```cmd/sh
python --version
```

Le package **azure-iot-device** est publié en tant que PIP.

Dans votre environnement Python local, installez le package comme suit :

```cmd/sh
pip install azure-iot-device
```

Installez le package **azure-iot-hub** en exécutant la commande suivante :

```cmd/sh
pip install azure-iot-hub
```

## <a name="run-the-sample-device"></a>Exécuter l’exemple d’appareil

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Pour en savoir plus sur l’exemple de configuration, consultez l’[exemple de fichier Lisez-moi](https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-device/samples/pnp/README.md).

Dans ce guide de démarrage rapide, vous utilisez un exemple d’appareil à thermostat, écrit en Python, en tant qu’appareil IoT Plug-and-Play. Pour exécuter l’exemple d’appareil :

1. Ouvrez une fenêtre de terminal dans un dossier de votre choix. Exécutez la commande suivante pour cloner le dépôt GitHub du [SDK Python Azure IoT](https://github.com/Azure/azure-iot-sdk-python) à cet emplacement :

    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-python
    ```

1. Cette fenêtre de terminal est utilisée en tant que terminal **device**. Accédez au dossier de votre dépôt cloné, puis accédez au dossier */azure-iot-sdk-python/azure-iot-device/samples/pnp*.

1. Pour exécutez l’exemple d’appareil à thermostat, utilisez la commande suivante :

    ```cmd/sh
    python simple_thermostat.py
    ```

1. Des messages s’affichent pour indiquer que l’appareil a envoyé des informations et qu’il s’est signalé comme étant en ligne. Ces messages indiquent que l’appareil a commencé à envoyer des données de télémétrie au hub et est maintenant prêt à recevoir des commandes et des mises à jour de propriétés. Ne fermez pas ce terminal. Vous en aurez besoin pour confirmer que l’exemple de service fonctionne également.

## <a name="run-the-sample-solution"></a>Exécuter l’exemple de solution

Dans ce guide de démarrage rapide, vous utilisez un exemple de solution IoT en Python pour interagir avec l’exemple d’appareil que vous venez de configurer.

1. Ouvrez une autre fenêtre de terminal à utiliser comme terminal **service**.

1. Accédez au dossier */azure-iot-sdk-python/azure-iot-hub/samples* du référentiel du SDK Python cloné.

1. Ouvrez le fichier *registry_manager_pnp_sample.py* et passez en revue le code. Cet exemple montre comment utiliser la classe **IoTHubRegistryManager** pour interagir avec votre appareil IoT Plug-and-Play.

> [!NOTE]
> Ces exemples de services utilisent la classe **IoTHubRegistryManager** à partir du **client du service IoT Hub**. Pour en savoir plus sur les API, y compris l’API Digital Twins, consultez le [Guide du développeur sur les services](../articles/iot-pnp/concepts-developer-guide-service.md).

### <a name="get-the-device-twin"></a>Obtenir le jumeau d’appareil

Dans [Configurer votre environnement pour les guides de démarrage rapide et tutoriels IoT Plug-and-Play](../articles/iot-pnp/set-up-environment.md), vous avez créé deux variables d’environnement pour configurer l’exemple afin qu’il se connecte à votre hub IoT et à votre appareil :

* **IOTHUB_CONNECTION_STRING** : la chaîne de connexion de hub IoT que vous avez notée précédemment.
* **IOTHUB_DEVICE_ID** : `"my-pnp-device"`.

Utilisez la commande suivante dans le terminal **service** pour exécuter cet exemple :

```cmd/sh
set IOTHUB_METHOD_NAME="getMaxMinReport"
set IOTHUB_METHOD_PAYLOAD="hello world"
python registry_manager_pnp_sample.py
```

> [!NOTE]
> Si vous exécutez cet exemple sur Linux, utilisez `export` à la place de `set`.

La sortie affiche le jumeau d’appareil et son ID de modèle :

```cmd/sh
The Model ID for this device is:
dtmi:com:example:Thermostat;1
```

L’extrait de code suivant montre l’exemple de code de *registry_manager_pnp_sample.py* :

```python
    # Create IoTHubRegistryManager
    iothub_registry_manager = IoTHubRegistryManager(iothub_connection_str)

    # Get device twin
    twin = iothub_registry_manager.get_twin(device_id)
    print("The device twin is: ")
    print("")
    print(twin)
    print("")

    # Print the device's model ID
    additional_props = twin.additional_properties
    if "modelId" in additional_props:
        print("The Model ID for this device is:")
        print(additional_props["modelId"])
        print("")
```

### <a name="update-a-device-twin"></a>Mettre à jour un jumeau d’appareil

Cet exemple montre comment mettre à jour la propriété `targetTemperature` accessible en écriture dans l’appareil :

```python
    # Update twin
    twin_patch = Twin()
    twin_patch.properties = TwinProperties(
        desired={"targetTemperature": 42}
    )  # this is relevant for the thermostat device sample
    updated_twin = iothub_registry_manager.update_twin(device_id, twin_patch, twin.etag)
    print("The twin patch has been successfully applied")
    print("")
```

Vous pouvez vérifier que la mise à jour est appliquée dans le terminal **device** qui affiche la sortie suivante :

```cmd/sh
the data in the desired properties patch was: {'targetTemperature': 42, '$version': 2}
```

Le terminal **service** confirme que le correctif a réussi :

```cmd/sh
The twin patch has been successfully applied
```

### <a name="invoke-a-command"></a>Appeler une commande

L’exemple appelle ensuite une commande :

Le terminal **service** affiche un message de confirmation de l’appareil :

```cmd/sh
The device method has been successfully invoked
```

Dans le terminal **device**, vous voyez que l’appareil reçoit la commande :

```cmd/sh
Command request received with payload
hello world
Will return the max, min and average temperature from the specified time hello to the current time
Done generating
{"tempReport": {"avgTemp": 34.2, "endTime": "09/07/2020 09:58:11", "maxTemp": 49, "minTemp": 10, "startTime": "09/07/2020 09:56:51"}}
```
