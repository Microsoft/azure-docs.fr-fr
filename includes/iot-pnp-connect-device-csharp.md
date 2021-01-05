---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 13d0bdf82052ff2c61c5b2c6010956c8fb27574d
ms.sourcegitcommit: b8a175b6391cddd5a2c92575c311cc3e8c820018
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96122452"
---
Ce guide de démarrage rapide vous montre comment créer un exemple d’application d’appareil IoT Plug-and-Play, comment le connecter à votre hub IoT et comment utiliser l’outil Explorateur Azure IoT pour afficher les données de télémétrie qu’il envoie. L’exemple d’application est écrit en C# et est inclus dans les exemples Azure IoT pour C#. Un créateur de solutions peut utiliser l’outil Explorateur Azure IoT pour comprendre les fonctionnalités d’un appareil IoT Plug-and-Play sans avoir besoin d’examiner le code d’appareil.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Pour suivre ce démarrage rapide sur Windows, vous devez avoir installé les logiciels suivants sur votre machine de développement :

* [Visual Studio (Community, Professional ou Enterprise)](https://visualstudio.microsoft.com/downloads/).
* [Git](https://git-scm.com/download/).

## <a name="download-the-code"></a>Téléchargement du code

Dans ce guide de démarrage rapide, vous préparez un environnement de développement pour cloner et générer le dépôt des exemples Azure IoT pour C#.

Ouvrez une invite de commandes dans un dossier de votre choix. Exécutez la commande suivante pour cloner le dépôt GitHub [Exemples Microsoft Azure IoT pour C# (.NET)](https://github.com/Azure-Samples/azure-iot-samples-csharp) à cet emplacement :

```cmd
git clone  https://github.com/Azure-Samples/azure-iot-samples-csharp.git
```

## <a name="build-the-code"></a>Générer le code

Vous pouvez maintenant générer l’exemple dans Visual Studio et l’exécuter en mode débogage.

1. Ouvrez le fichier de projet *azure-iot-samples-csharp\iot-hub\Samples\device\PnpDeviceSamples\Thermostat\Thermostat.csproj* dans Visual Studio 2019.

1. Dans Visual Studio, accédez à **Projet > Propriétés du thermostat > Déboguer**. Ensuite, ajoutez les variables d’environnement suivantes au projet :

    | Nom | Valeur |
    | ---- | ----- |
    | IOTHUB_DEVICE_SECURITY_TYPE | DPS |
    | IOTHUB_DEVICE_DPS_ENDPOINT | global.azure-devices-provisioning.net |
    | IOTHUB_DEVICE_DPS_ID_SCOPE | Valeur que vous avez notée à la fin de la section [Configurer votre environnement](../articles/iot-pnp/set-up-environment.md) |
    | IOTHUB_DEVICE_DPS_DEVICE_ID | my-pnp-device |
    | IOTHUB_DEVICE_DPS_DEVICE_KEY | Valeur que vous avez notée à la fin de la section [Configurer votre environnement](../articles/iot-pnp/set-up-environment.md) |

Vous pouvez maintenant générer l’exemple dans Visual Studio et l’exécuter en mode débogage.

## <a name="run-the-device-sample"></a>Exécuter l’exemple d’appareil

Pour effectuer le suivi de l’exécution du code dans Visual Studio sur Windows, ajoutez un point d’arrêt sur la fonction `main` dans le fichier program.cs.

L’appareil est désormais prêt à recevoir des commandes et des mises à jour de propriétés, et a commencé à envoyer des données de télémétrie au hub. Laissez l’exemple s’exécuter pendant que vous effectuez les étapes suivantes.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Utiliser l’Explorateur Azure IoT pour valider le code

Une fois l’exemple de client d’appareil démarré, utilisez l’outil Explorateur Azure IoT pour vérifier qu’il fonctionne.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]

## <a name="review-the-code"></a>Vérifier le code

Cet exemple implémente un appareil à thermostat IoT Plug-and-Play simple. Le modèle implémenté par cet exemple n’utilise pas de [composants](../articles/iot-pnp/concepts-components.md) IoT Plug-and-Play. Le [fichier de modèle DTDL (Digital Twins Definition Language) pour l’appareil à thermostat](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json) définit la télémétrie, les propriétés et les commandes implémentées par l’appareil.

Le code de l’appareil se connecte à votre hub IoT en utilisant la méthode `CreateFromConnectionString` standard. L’appareil envoie l’ID de modèle du modèle DTDL qu’il implémente dans la demande de connexion. Un appareil qui envoie un ID de modèle est un appareil IoT Plug-and-Play :

```csharp
private static void InitializeDeviceClientAsync()
{
  var options = new ClientOptions
  {
    ModelId = ModelId,
  };
  s_deviceClient = DeviceClient.CreateFromConnectionString(s_deviceConnectionString, TransportType.Mqtt, options);
  s_deviceClient.SetConnectionStatusChangesHandler((status, reason) =>
  {
     s_logger.LogDebug($"Connection status change registered - status={status}, reason={reason}.");
  });
}
```

L’ID de modèle est stocké dans le code, comme indiqué dans l’extrait de code suivant :

```csharp
private const string ModelId = "dtmi:com:example:Thermostat;1";
```

Le code qui met à jour les propriétés, gère les commandes et envoie les données de télémétrie est identique au code d’un appareil qui n’utilise pas les conventions IoT Plug-and-Play.

L’exemple utilise une bibliothèque JSON pour analyser les objets JSON dans les charges utiles envoyées à partir de votre hub IoT :

```csharp
using Newtonsoft.Json;

...

DateTime since = JsonConvert.DeserializeObject<DateTime>(request.DataAsJson);
```
