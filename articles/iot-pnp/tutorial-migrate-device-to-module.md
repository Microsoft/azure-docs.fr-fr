---
title: Tutoriel - Connecter un module générique Azure IoT Plug-and-Play | Microsoft Docs
description: Tutoriel - Utiliser l’exemple de code d’appareil IoT Plug-and-Play en C# dans un module générique.
author: ericmitt
ms.author: ericmitt
ms.date: 9/22/2020
ms.topic: tutorial
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 9ac616ddf1c3475f2ca3b3e8097bb74da72faa77
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95500269"
---
# <a name="tutorial-connect-an-iot-plug-and-play-module-c"></a>Tutoriel : Connecter un module IoT Plug-and-Play (C#)

Ce tutoriel vous montre comment connecter un [module](../iot-hub/iot-hub-devguide-module-twins.md) IoT Plug-and-Play générique.

Un appareil est un appareil IoT Plug-and-Play s’il publie son ID de modèle lorsqu’il se connecte à un hub IoT et implémente les propriétés et méthodes décrites dans le modèle DTDL (Digital Twins Definition Language) identifié par l’ID de modèle. Pour en savoir plus sur la façon dont les appareils utilisent un DTDL et un ID de modèle, consultez [Guide du développeur IoT Plug-and-Play](./concepts-developer-guide-device.md). Les modules utilisent des ID de modèle et des modèles DTDL de la même manière.

Pour illustrer la façon d’implémenter un module IoT Plug-and-Play, ce tutoriel vous montre comment convertir l'exemple d’appareil C# thermostat en module générique.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [iot-pnp-prerequisites](../../includes/iot-pnp-prerequisites.md)]

Pour effectuer ce tutoriel sur Windows, vous devez installer les logiciels suivants sur votre environnement Windows local :

* [Visual Studio (Community, Professional ou Enterprise)](https://visualstudio.microsoft.com/downloads/).
* [Git](https://git-scm.com/download/).

Utilisez l’explorateur Azure IoT pour ajouter un nouvel appareil appelé **my-module-device** à votre hub IoT.

Ajoutez un module appelé **my-module** à **my-module-device** :

1. Dans l’explorateur Azure IoT, accédez à l’appareil **my-module-device**.

1. Sélectionnez **Identité de module**, puis **+ Ajouter**.

1. Entrez **my-module** comme nom d’identité du module, puis sélectionnez **Enregistrer**.

1. Dans la liste des identités de module, sélectionnez **my-module**. Copiez ensuite la chaîne de connexion principale. Vous utiliserez cette chaîne de connexion de module plus loin dans ce tutoriel.

1. Sélectionnez l’onglet **Jumeau de module** et notez qu’il n’existe aucune propriété souhaitée ou signalée :

    ```json
    {
      "deviceId": "my-module-device",
      "moduleId": "my-module",
      "etag": "AAAAAAAAAAE=",
      "deviceEtag": "NjgzMzQ1MzQ1",
      "status": "enabled",
      "statusUpdateTime": "0001-01-01T00:00:00Z",
      "connectionState": "Disconnected",
      "lastActivityTime": "0001-01-01T00:00:00Z",
      "cloudToDeviceMessageCount": 0,
      "authenticationType": "sas",
      "x509Thumbprint": {
        "primaryThumbprint": null,
        "secondaryThumbprint": null
      },
      "modelId": "",
      "version": 2,
      "properties": {
        "desired": {
          "$metadata": {
            "$lastUpdated": "0001-01-01T00:00:00Z"
          },
          "$version": 1
          },
          "reported": {
            "$metadata": {
              "$lastUpdated": "0001-01-01T00:00:00Z"
          },
          "$version": 1
        }
      }
    }
    ```

## <a name="download-the-code"></a>Téléchargement du code

Si vous ne l’avez pas déjà fait, clonez le dépôt GitHub du kit de développement logiciel (SDK) C# Azure IOT Hub Device sur votre ordinateur local :

Ouvrez une invite de commandes dans un dossier de votre choix. Utilisez la commande suivante pour cloner le dépôt GitHub des [exemples C# Azure IoT](https://github.com/Azure-Samples/azure-iot-samples-csharp) à cet emplacement :

```cmd
git clone https://github.com/Azure-Samples/azure-iot-samples-csharp.git
```

## <a name="prepare-the-project"></a>Préparer le projet

Pour ouvrir et préparer l’exemple de projet :

1. Ouvrez le fichier de projet *azure-iot-sdk-csharp\iot-hub\Samples\device\PnpDeviceSamples\Thermostat\Thermostat.csproj* dans Visual Studio 2019.

1. Dans Visual Studio, accédez à **Projet > Propriétés du thermostat > Déboguer**. Ensuite, ajoutez les variables d’environnement suivantes au projet :

    | Nom | Valeur |
    | ---- | ----- |
    | IOTHUB_DEVICE_SECURITY_TYPE | connectionString |
    | IOTHUB_MODULE_CONNECTION_STRING | Chaîne de connexion de module que vous avez notée précédemment |

    Pour en savoir plus sur l’exemple de configuration, consultez l’[exemple de fichier Lisez-moi](https://github.com/Azure-Samples/azure-iot-samples-csharp/blob/master/iot-hub/Samples/device/PnpDeviceSamples/readme.md).

## <a name="modify-the-code"></a>Modifier ce code.

Pour modifier le code de sorte qu’il fonctionne en tant que module plutôt qu’appareil :

1. Dans Visual Studio, ouvrez *Parameter.cs* et modifiez la ligne définissant la variable **PrimaryConnectionString** comme suit :

    ```csharp
    public string PrimaryConnectionString { get; set; } = Environment.GetEnvironmentVariable("IOTHUB_MODULE_CONNECTION_STRING");
    ```

1. Dans Visual Studio, ouvrez *Program.cs* et remplacez les sept instances de la classe `DeviceClient` par la classe `ModuleClient`.

    > [!TIP]
    > Utilisez la recherche Visual Studio et remplacez la fonctionnalité par **Respecter la casse** et **Mot entier** pour remplacer `DeviceClient` par `ModuleClient`.

1. Dans Visual Studio, ouvrez *Thermostat.cs* et remplacez les deux instances de la classe `DeviceClient` par la classe `ModuleClient` comme suit.

1. Enregistrez les modifications apportées aux fichiers que vous avez modifiés.

Si vous exécutez le code, puis utilisez l’explorateur Azure IoT pour afficher le jumeau de module mis à jour, vous voyez le jumeau d’appareil mis à jour par l’ID de modèle et la propriété de module signalée :

```json
{
  "deviceId": "my-module-device",
  "moduleId": "my-mod",
  "etag": "AAAAAAAAAAE=",
  "deviceEtag": "NjgzMzQ1MzQ1",
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
  "version": 3,
  "properties": {
    "desired": {
      "$metadata": {
        "$lastUpdated": "0001-01-01T00:00:00Z"
      },
      "$version": 1
    },
    "reported": {
      "maxTempSinceLastReboot": 5,
      "$metadata": {
        "$lastUpdated": "2020-09-28T08:53:45.9956637Z",
        "maxTempSinceLastReboot": {
          "$lastUpdated": "2020-09-28T08:53:45.9956637Z"
        }
      },
      "$version": 2
    }
  }
}
```

## <a name="interact-with-a-device-module"></a>Interagir avec un module d’appareil

Les kits de développement logiciel (SDK) de service vous permettent de récupérer l’ID de modèle des appareils et des modules IoT Plug-and-Play connectés. Vous pouvez utiliser les kits de développement logiciel (SDK) de service pour définir des propriétés accessibles en écriture et appeler des commandes :

1. Dans une autre instance de Visual Studio, ouvrez le projet *azure-iot-sdk-csharp\iot-hub\Samples\service\PnpServiceSamples\Thermostat\Thermostat.csproj*.

1. Dans Visual Studio, accédez à **Projet > Propriétés du thermostat > Déboguer**. Ensuite, ajoutez les variables d’environnement suivantes au projet :

    | Nom | Valeur |
    | ---- | ----- |
    | IOTHUB_DEVICE_ID | my-module-device |
    | IOTHUB_CONNECTION_STRING | Valeur que vous avez notée à la fin de la section [Configurer votre environnement](set-up-environment.md) |

    > [!TIP]
    > Vous pouvez également rechercher chaîne de connexion de votre hub IoT dans l’explorateur Azure IoT.

1. Ouvrez le fichier *Program.cs* et modifiez la ligne appelant une commande comme suit :

    ```csharp
    CloudToDeviceMethodResult result = await s_serviceClient.InvokeDeviceMethodAsync(s_deviceId, "my-module", commandInvocation);
    ```

1. Dans le fichier *Program.cs* , modifiez la ligne récupérant le jumeau d’appareil comme suit :

    ```csharp
    Twin twin = await s_registryManager.GetTwinAsync(s_deviceId, "my-module");
    ```

1. Assurez-vous que l’exemple de client de module est toujours en cours d’exécution, puis exécutez cet exemple de service. La sortie de l’exemple de service montre l’ID de modèle du jumeau d’appareil et l’appel de commande :

    ```cmd
    [09/28/2020 10:52:55]dbug: Thermostat.Program[0]
      Initialize the service client.
    [09/28/2020 10:52:55]dbug: Thermostat.Program[0]
      Get Twin model Id and Update Twin
    [09/28/2020 10:52:59]dbug: Thermostat.Program[0]
      Model Id of this Twin is: dtmi:com:example:Thermostat;1
    [09/28/2020 10:52:59]dbug: Thermostat.Program[0]
      Invoke a command
    [09/28/2020 10:53:00]dbug: Thermostat.Program[0]
      Command getMaxMinReport invocation result status is: 200
    ```

    La sortie du client de module affiche la réponse du gestionnaire de commandes :

    ```cmd
    [09/28/2020 10:53:00]dbug: Thermostat.ThermostatSample[0]
      Command: Received - Generating max, min and avg temperature report since 28/09/2020 10:52:55.
    [09/28/2020 10:53:00]dbug: Thermostat.ThermostatSample[0]
      Command: MaxMinReport since 28/09/2020 10:52:55: maxTemp=25.4, minTemp=25.4, avgTemp=25.4, startTime=28/09/2020 10:52:56, endTime=28/09/2020 10:52:56
    ```

## <a name="convert-to-an-iot-edge-module"></a>Convertir en module IoT Edge

Pour convertir cet exemple de sorte qu’il fonctionne en tant que module IoT Edge IoT Plug-and-Play IoT, vous devez conteneuriser l’application. Aucune modification de code supplémentaire n’est requise. La variable d’environnement de la chaîne de connexion est injectée par le runtime IoT Edge au démarrage. Pour plus d’informations, consultez [Utiliser Visual Studio 2019 pour développer et déboguer des modules pour Azure IoT Edge](../iot-edge/how-to-visual-studio-develop-module.md).

Pour savoir comment déployer votre module conteneurisé, consultez :

* [Exécuter Azure IoT Edge sur des machines virtuelles Ubuntu](../iot-edge/how-to-install-iot-edge-ubuntuvm.md).
* [Installer le runtime Azure IoT Edge sur des systèmes Linux Debian](../iot-edge/how-to-install-iot-edge.md).

Vous pouvez utiliser l’explorateur Azure IoT pour voir ce qui suit :

* ID de modèle de votre appareil IoT Edge dans le jumeau de module.
* Télémétrie à partir de l’appareil IoT Edge.
* Mises à jour des propriétés du jumeau de module IoT Edge déclenchant des notifications IoT Plug-and-Play.
* Réaction du module IoT Edge à vos commandes IoT Plug-and-Play.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez découvert comment connecter un appareil IoT Plug-and-Play avec des modules à un hub IoT. Pour découvrir plus d’informations sur les modèles d’appareils IoT Plug-and-Play, consultez :

> [!div class="nextstepaction"]
> [Guide du développeur pour la modélisation d’IoT Plug-and-Play](./concepts-developer-guide-device.md)