---
title: Exemple de connexion d’un pont IoT Plug-and-Play s’exécutant sur Linux ou Windows à un hub IoT | Microsoft Docs
description: Créez et exécutez un exemple de code pour pont IoT Plug-and-Play sur Linux ou Windows qui se connecte à un hub IoT. Utilisez l’outil Azure IoT Explorer pour afficher les informations envoyées par l’appareil au hub.
author: usivagna
ms.author: ugans
ms.date: 09/22/2020
ms.topic: how-to
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: eedd19189d1e1ccedd3d505aecf407aca8fca831
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94413358"
---
# <a name="how-to-connect-an--iot-plug-and-play-bridge-sample-running-on-linux-or-windows-to-iot-hub"></a>Exemple de connexion d’un pont IoT Plug-and-Play s’exécutant sur Linux ou Windows à IoT Hub

Ce guide pratique montre comment créer un exemple d’adaptateur environnemental de pont IoT Plug-and-Play, comment le connecter à votre hub IoT, et comment utiliser l’outil Explorateur Azure IoT pour afficher la télémétrie qu’il envoie. Le pont IoT Plug-and-Play est écrit en C et comprend l’Azure IoT device SDK pour C. À la fin de ce didacticiel, vous devriez pouvoir exécuter le pont IoT Plug-and-Play et le voir rapporter la télémétrie dans l’Explorateur Azure IoT : :::image type="content" source="media/concepts-iot-pnp-bridge/iot-pnp-bridge-explorer-telemetry.png" alt-text="Capture d’écran montrant l’Explorateur Azure IoT avec un tableau de télémétrie (humidité, température) rapportée du pont IoT Plug-and-Play.":::

## <a name="prerequisites"></a>Prérequis

Vous pouvez exécuter ce guide de démarrage rapide sur Linux ou Windows. Les commandes de l’interpréteur de commandes de ce guide pratique suivent la convention Windows des séparateurs de chemin « `\` ». Si vous suivez ce guide sur Linux, veillez à remplacer les séparateurs par « `/` ».

Les prérequis diffèrent selon le système d’exploitation :

### <a name="linux"></a>Linux

Ce guide de démarrage rapide suppose que vous utilisez Ubuntu Linux. Les étapes de ce guide de démarrage rapide ont été testées à l’aide d’Ubuntu 18.04.

Pour suivre ce guide de démarrage rapide sur Linux, vous devez installer les logiciels suivants sur votre environnement Linux :

Installez **GCC** , **Git** , **cmake** et toutes les dépendances nécessaires à l’aide de la commande `apt-get` :

```sh
sudo apt-get update
sudo apt-get install -y git cmake build-essential curl libcurl4-openssl-dev libssl-dev uuid-dev
```

Vérifiez que la version de `cmake` est supérieure à **2.8.12** et que la version de **GCC** est supérieure à **4.4.7**.

```sh
cmake --version
gcc --version
```

### <a name="windows"></a>Windows

Pour suivre ce guide de démarrage rapide sur Windows, vous devez installer les logiciels suivants sur votre environnement Windows :

* [Visual Studio (Community, Professional ou Enterprise)](https://visualstudio.microsoft.com/downloads/) : veillez à inclure la charge de travail **Développement Desktop en C++** quand vous [installez](/cpp/build/vscpp-step-0-installation?preserve-view=true&view=vs-2019) Visual Studio.
* [Git](https://git-scm.com/download/).
* [CMake](https://cmake.org/download/).

### <a name="azure-iot-explorer"></a>Explorateur Azure IoT

Pour interagir avec l’exemple d’appareil dans la deuxième partie de ce guide de démarrage rapide, vous utilisez l’outil **Explorateur Azure IoT**. [Téléchargez et installez la dernière version de l’Explorateur Azure IoT](./howto-use-iot-explorer.md) pour votre système d’exploitation.

[!INCLUDE [iot-pnp-prepare-iot-hub.md](../../includes/iot-pnp-prepare-iot-hub.md)]

Exécutez la commande suivante pour obtenir la _chaîne de connexion IoT Hub_ pour votre hub. Prenez note de cette chaîne de connexion, car vous l’utiliserez plus loin dans ce guide de démarrage rapide :

```azurecli-interactive
az iot hub show-connection-string --hub-name <YourIoTHubName> --output table
```

> [!TIP]
> Vous pouvez également utiliser l’outil Explorateur Azure IoT pour rechercher la chaîne de connexion du hub IoT.

Exécutez la commande suivante pour obtenir la _chaîne de connexion_ à l’appareil que vous avez ajouté au hub. Prenez note de cette chaîne de connexion, car vous l’utiliserez plus loin dans ce guide de démarrage rapide :

```azurecli-interactive
az iot hub device-identity show-connection-string --hub-name <YourIoTHubName> --device-id <YourDeviceID> --output table
```

## <a name="view-the-model"></a>Afficher le modèle

Vous utiliserez l’Explorateur Azure IoT dans les étapes suivantes pour afficher l’appareil quand celui-ci se connecte à votre hub IoT. L’Explorateur Azure IoT aura besoin d’une copie locale du fichier de modèle qui correspond à l’ **ID de modèle** que votre appareil envoie. Le fichier de modèle permet à l’Explorateur IoT d’afficher les données de télémétrie, les propriétés et les commandes que votre appareil implémente.

Lorsque vous téléchargez le code à l’étape ci-dessous, il comprend les exemples de fichiers de modèle dans le dossier `pnpbridge/docs/schema`. Pour préparer l’Explorateur Azure IoT :

1. Créez un dossier nommé *models* sur votre ordinateur local.
1. Affichez [EnvironmentalSensor.json](https://aka.ms/iot-pnp-bridge-env-model) et enregistrez le fichier JSON dans le dossier *models*
1. Affichez [RootBridgeSampleDevice.json](https://aka.ms/iot-pnp-bridge-root-model) et enregistrez le fichier JSON dans le dossier *models*.

## <a name="download-the-code"></a>Téléchargement du code

Ouvrez une invite de commandes dans le répertoire de votre choix. Exécutez la commande suivante pour cloner le référentiel GitHub [Pont IoT Plug-and-Play](https://aka.ms/iotplugandplaybridge) à cet emplacement :

```cmd
git clone https://github.com/Azure/iot-plug-and-play-bridge.git
```

Après avoir cloné le référentiel Pont IoT Plug-and-Play sur votre ordinateur, ouvrez une invite de commandes d’administration et accédez au répertoire du référentiel cloné :

```cmd
cd pnpbridge
git submodule update --init --recursive
```

Attendez-vous à ce que cette opération prenne plusieurs minutes.

>[!NOTE]
> Si vous rencontrez des problèmes d’échec de la mise à jour du sous-module de clone Git, il s’agit d’un problème connu avec les chemins d’accès aux fichiers Windows et Git. Consultez : https://github.com/msysgit/git/pull/110. Vous pouvez essayer la commande suivante pour résoudre le problème : `git config --system core.longpaths true`

## <a name="setting-up-the-configuration-json"></a>Définition du fichier JSON de configuration

Après le clonage du référentiel Pont IoT Plug-and-Play sur votre ordinateur, accédez au répertoire `pnpbridge/docs/schema` du référentiel cloné où vous pouvez trouver le [JSON de configuration](https://aka.ms/iot-pnp-bridge-env-config) ou `config.json` pour l’exemple de capteur environnemental du pont. Apprenez-en davantage sur les fichiers de configuration dans le document [Concepts de pont IoT Plug-and-Play](concepts-iot-pnp-bridge.md).

Pour le champ `root-_interface_model_id`, vous devez copier l’ID de modèle IoT Plug-and-Play qui identifie le modèle de votre appareil. Dans cet exemple, il s’agit de `dtmi:com:example:SampleDevice;1`. Modifiez les paramètres suivants sous le nœud **pnp_bridge_parameters** dans le fichier `config.json` dans :

* connection_string 
* symmetric_key 

>[!NOTE]
> symmetric_key doit correspondre à la clé SAS dans la chaîne de connexion.

  ```JSON
    {
      "connection_parameters": {
        "connection_type" : "connection_string",
        "connection_string" : "[CONNECTION STRING]",
        "root_interface_model_id": "[To fill in]",
        "auth_parameters" : {
          "auth_type" : "symmetric_key",
          "symmetric_key" : "[SYMMETRIC KEY]"
        }
      }
    }
  }
  ```

 Une fois rempli, le fichier `config.json` doit ressembler à ce qui suit :

   ```JSON
    {
      "connection_parameters": {
        "connection_type" : "connection_string",
        "connection_string" : "[CONNECTION STRING]",
        "root_interface_model_id": "dtmi:com:example:SampleDevice;1",
        "auth_parameters" : {
          "auth_type" : "symmetric_key",
          "symmetric_key" : "[SYMMETRIC KEY]"
        }
      }
    }
  }
```

 Une fois que vous avez créé le pont, vous devez placer ce `config.json` dans le même répertoire que le pont ou spécifier son chemin d’accès lors de son exécution.

## <a name="build-the-iot-plug-and-play-bridge"></a>Générer le pont IoT Plug-and-Play

Accédez au dossier *pnpbridge* dans le répertoire du référentiel.

Pour Windows, exécutez la commande suivante dans une [Invite de commandes développeur pour Visual Studio](/dotnet/framework/tools/developer-command-prompt-for-vs) :

```cmd
cd scripts\windows
build.cmd
```

De même, pour Linux, exécutez la commande suivante :

```bash
cd scripts/linux
./setup.sh
./build.sh
```

>[!TIP]
>Sur Windows, vous pouvez ouvrir la solution générée par la commande cmake dans Visual Studio 2019. Ouvrez le fichier projet *azure_iot_pnp_bridge.sln* dans le répertoire cmake et définissez le projet *pnpbridge_bin* comme projet de démarrage dans la solution. Vous pouvez maintenant générer l’exemple dans Visual Studio et l’exécuter en mode débogage.

## <a name="start-the-iot-plug-and-play-bridge"></a>Démarrer le pont IoT Plug-and-Play

 Démarrez l’exemple de pont IoT Plug-and-Play Bridge pour les capteurs environnementaux en accédant au dossier *pnpbridge* et en exécutant la commande suivante dans une invite de commandes :

```bash
 cd cmake/pnpbridge_linux/src/adapters/samples/environmental_sensor/
./pnpbridge_environmentalsensor

```

```cmd
REM Windows
cd cmake\pnpbridge_x86\src\adapters\samples\environmental_sensor
Debug\pnpbridge_environmentalsensor.exe
```

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Utiliser l’Explorateur Azure IoT pour valider le code

Une fois l’exemple de client d’appareil démarré, utilisez l’outil Explorateur Azure IoT pour vérifier qu’il fonctionne.

[!INCLUDE [iot-pnp-iot-explorer.md](../../includes/iot-pnp-iot-explorer.md)]

[!INCLUDE [iot-pnp-clean-resources.md](../../includes/iot-pnp-clean-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez appris à connecter un appareil IoT Plug-and-Play à un hub IoT. Pour en savoir plus sur la création d’une solution qui interagit avec vos appareils IoT Plug-and-Play, consultez :

* [Qu’est ce qu’un pont IoT Plug-and-Play ?](./concepts-iot-pnp-bridge.md)
* [Consulter les informations de référence pour les développeurs GitHub pour le pont IoT Plug-and-Play](https://aka.ms/iot-pnp-bridge-dev-doc)
* [Pont IoT Plug-and-Play sur GitHub](https://aka.ms/iotplugandplaybridge)
