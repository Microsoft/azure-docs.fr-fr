---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 4308dd2b63b33604af83b360e5c1c0f02a3dec27
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95487789"
---
IoT Plug-and-Play simplifie l’IoT en vous permettant d’interagir avec les fonctionnalités d’un appareil sans avoir connaissance de l’implémentation de l’appareil sous-jacent. Ce guide de démarrage rapide vous montre comment utiliser C# pour vous connecter à un appareil IoT Plug-and-Play connecté à votre solution et le contrôler.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Pour suivre ce démarrage rapide sur Windows, vous devez avoir installé les logiciels suivants sur votre machine de développement :

* [Visual Studio (Community, Professional ou Enterprise)](https://visualstudio.microsoft.com/downloads/).
* [Git](https://git-scm.com/download/).

### <a name="clone-the-sdk-repository-with-the-sample-code"></a>Clonez le référentiel SDK avec l’exemple de code

Si vous avez terminé [Démarrage rapide : Connecter un exemple d’application pour appareil IoT Plug-and-Play s’exécutant sur Windows à IoT Hub (C#)](../articles/iot-pnp/quickstart-connect-device.md), vous avez déjà cloné le dépôt.

Clonez les échantillons à partir du dépôt GitHub des échantillons Azure IoT pour C#. Ouvrez une invite de commandes dans un dossier de votre choix. Exécutez la commande suivante pour cloner le dépôt GitHub [Exemples Microsoft Azure IoT pour .NET](https://github.com/Azure-Samples/azure-iot-samples-csharp) :

```cmd
git clone https://github.com/Azure-Samples/azure-iot-samples-csharp.git
```

## <a name="run-the-sample-device"></a>Exécuter l’exemple d’appareil

Dans ce démarrage rapide, vous utilisez un exemple de thermostat écrit en C# en tant qu’appareil IoT Plug-and-Play. Pour exécuter l’exemple d’appareil :

1. Ouvrez le fichier de projet *azure-iot-samples-csharp\iot-hub\Samples\device\PnpDeviceSamples\Thermostat\Thermostat.csproj* dans Visual Studio 2019.

1. Dans Visual Studio, accédez à **Projet > Propriétés du thermostat > Déboguer**. Ensuite, ajoutez les variables d’environnement suivantes au projet :

    | Nom | Valeur |
    | ---- | ----- |
    | IOTHUB_DEVICE_SECURITY_TYPE | DPS |
    | IOTHUB_DEVICE_DPS_ENDPOINT | global.azure-devices-provisioning.net |
    | IOTHUB_DEVICE_DPS_ID_SCOPE | Valeur que vous avez notée à la fin de la section [Configurer votre environnement](../articles/iot-pnp/set-up-environment.md) |
    | IOTHUB_DEVICE_DPS_DEVICE_ID | my-pnp-device |
    | IOTHUB_DEVICE_DPS_DEVICE_KEY | Valeur que vous avez notée à la fin de la section [Configurer votre environnement](../articles/iot-pnp/set-up-environment.md) |

1. Vous pouvez maintenant générer l’exemple dans Visual Studio et l’exécuter en mode débogage.

1. Des messages s’affichent pour indiquer que l’appareil a envoyé des informations et qu’il s’est signalé comme étant en ligne. Ces messages indiquent que l’appareil a commencé à envoyer des données de télémétrie au hub et est maintenant prêt à recevoir des commandes et des mises à jour de propriétés. Ne fermez pas cette instance de Visual Studio. Vous en aurez besoin pour confirmer que l’exemple de service fonctionne.

## <a name="run-the-sample-solution"></a>Exécuter l’exemple de solution

Dans [Configurer votre environnement pour les guides de démarrage rapide et tutoriels IoT Plug-and-Play](../articles/iot-pnp/set-up-environment.md), vous avez créé deux variables d’environnement pour configurer l’exemple afin qu’il se connecte à votre hub IoT et à votre appareil :

* **IOTHUB_CONNECTION_STRING** : la chaîne de connexion de hub IoT que vous avez notée précédemment.
* **IOTHUB_DEVICE_ID** : `"my-pnp-device"`.

Dans ce guide de démarrage rapide, vous utilisez un exemple de solution IoT écrit en C# pour interagir avec l’exemple d’appareil que vous venez de configurer.

1. Dans une autre instance de Visual Studio, ouvrez le projet *azure-iot-samples-csharp\iot-hub\Samples\service\PnpServiceSamples\Thermostat\Thermostat.csproj*.

1. Dans Visual Studio, accédez à **Projet > Propriétés du thermostat > Déboguer**. Ensuite, ajoutez les variables d’environnement suivantes au projet :

    | Nom | Valeur |
    | ---- | ----- |
    | IOTHUB_DEVICE_ID | my-pnp-device |
    | IOTHUB_CONNECTION_STRING | Valeur que vous avez notée à la fin de la section [Configurer votre environnement](../articles/iot-pnp/set-up-environment.md) |

1. Vous pouvez maintenant générer l’exemple dans Visual Studio et l’exécuter en mode débogage.

### <a name="get-device-twin"></a>Obtenir un jumeau d’appareil

L’extrait de code suivant montre comment l’application de service récupère le jumeau d’appareil :

```C#
// Get a Twin and retrieves model Id set by Device client
Twin twin = await s_registryManager.GetTwinAsync(s_deviceId);
s_logger.LogDebug($"Model Id of this Twin is: {twin.ModelId}");
```

> [!NOTE]
> Cet exemple utilise l’espace de noms **Microsoft.Azure.Devices.Client** à partir du **client du service IoT Hub**. Pour en savoir plus sur les API, y compris l’API Digital Twins, consultez le [Guide du développeur sur les services](../articles/iot-pnp/concepts-developer-guide-service.md).

Ce code génère la sortie suivante :

```cmd
[09/21/2020 11:26:04]dbug: Thermostat.Program[0]
      Model Id of this Twin is: dtmi:com:example:Thermostat;1
```

L’extrait de code suivant montre comment utiliser un *correctif* pour mettre à jour les propriétés par le biais du jumeau d’appareil :

```C#
// Update the twin
var twinPatch = new Twin();
twinPatch.Properties.Desired[PropertyName] = PropertyValue;
await s_registryManager.UpdateTwinAsync(s_deviceId, twinPatch, twin.ETag);
```

Ce code génère la sortie suivante à partir de l’appareil lorsque le service met à jour la propriété `targetTemperature` :

```cmd
[09/21/2020 11:26:05]dbug: Thermostat.ThermostatSample[0]
      Property: Received - { "targetTemperature": 60°C }.
[09/21/2020 11:26:05]dbug: Thermostat.ThermostatSample[0]
      Property: Update - {"targetTemperature": 60°C } is InProgress.

...

[09/21/2020 11:26:17]dbug: Thermostat.ThermostatSample[0]
      Property: Update - {"targetTemperature": 60°C } is Completed.
```

### <a name="invoke-a-command"></a>Appeler une commande

L’exemple de code suivant montre comment appeler une commande :

```csharp
private static async Task InvokeCommandAsync()
{
    var commandInvocation = new CloudToDeviceMethod(CommandName) { ResponseTimeout = TimeSpan.FromSeconds(30) };

    // Set command payload
    string componentCommandPayload = JsonConvert.SerializeObject(s_dateTime);
    commandInvocation.SetPayloadJson(componentCommandPayload);

    CloudToDeviceMethodResult result = await s_serviceClient.InvokeDeviceMethodAsync(s_deviceId, commandInvocation);

    if (result == null)
    {
        throw new Exception($"Command {CommandName} invocation returned null");
    }

    s_logger.LogDebug($"Command {CommandName} invocation result status is: {result.Status}");
}
```

Ce code génère la sortie suivante à partir de l’appareil lorsque le service appelle la commande `getMaxMinReport` :

```cmd
[09/21/2020 11:26:05]dbug: Thermostat.ThermostatSample[0]
      Command: Received - Generating max, min and avg temperature report since 21/09/2020 11:25:58.
[09/21/2020 11:26:05]dbug: Thermostat.ThermostatSample[0]
      Command: MaxMinReport since 21/09/2020 11:25:58: maxTemp=32, minTemp=32, avgTemp=32, startTime=21/09/2020 11:25:59, endTime=21/09/2020 11:26:04
```
