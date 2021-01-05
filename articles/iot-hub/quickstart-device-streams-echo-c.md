---
title: Communiquer avec une application d’appareil en C avec des flux d'appareil IoT Hub
description: Dans ce guide de démarrage rapide, vous exécutez une application C côté appareil qui communique avec un appareil IoT par le biais d’un flux d’appareil.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: c
ms.topic: quickstart
ms.custom: mvc, devx-track-azurecli
ms.date: 08/20/2019
ms.author: robinsh
ms.openlocfilehash: de310846ad0449a0dac7eccd60d82d4c68ef519b
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94832203"
---
# <a name="quickstart-communicate-to-a-device-application-in-c-via-iot-hub-device-streams-preview"></a>Démarrage rapide : Communiquer avec une application d’appareil dans C par le biais de flux d’appareil IoT Hub (préversion)

[!INCLUDE [iot-hub-quickstarts-3-selector](../../includes/iot-hub-quickstarts-3-selector.md)]

Azure IoT Hub prend actuellement en charge les flux d’appareil en tant que [fonctionnalité d’évaluation](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Les [flux d’appareil IoT Hub](iot-hub-device-streams-overview.md) permettent aux applications de service et d’appareil de communiquer de manière sécurisée à travers des pare-feux. Dans la préversion publique, le SDK C prend uniquement en charge les flux d’appareil côté appareil. Par conséquent, le présent guide de démarrage rapide couvre uniquement les instructions permettant d’exécuter l’application côté appareil. Pour exécuter une application côté service correspondante, consultez ces articles :

* [Communiquer avec des applications d’appareil en C# via les flux d'appareils IoT Hub](./quickstart-device-streams-echo-csharp.md)

* [Communiquer avec des applications d’appareil en Node.js par le biais de flux d’appareil IoT Hub](./quickstart-device-streams-echo-nodejs.md)

L’application C côté appareil utilisée dans ce guide de démarrage rapide présente les fonctionnalités suivantes :

* Établissez un flux d’appareil vers un appareil IoT.

* Recevez les données envoyées à partir de l’application côté service et renvoyez-les.

Le code illustre le processus de lancement d’un flux d’appareil, ainsi que la façon de l’utiliser pour envoyer et recevoir des données.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Vous devez respecter les prérequis suivants :

* Installez [Visual Studio 2019](https://www.visualstudio.com/vs/) avec la charge de travail **Développement Desktop en C++** activée.

* Installez la dernière version de [Git](https://git-scm.com/download/).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

Pour le moment, la préversion des flux d’appareil est uniquement prise en charge pour les hubs IoT créés dans les régions suivantes :

  * USA Centre
  * EUAP USA Centre
  * Europe Nord
  * Asie Sud-Est

## <a name="prepare-the-development-environment"></a>Préparer l’environnement de développement

Pour ce guide de démarrage rapide, vous utilisez [Azure IoT device SDK for C](iot-hub-device-sdk-c-intro.md). Vous préparez un environnement de développement pour cloner et générer le [SDK C Azure IoT](https://github.com/Azure/azure-iot-sdk-c) à partir de GitHub. Le SDK sur GitHub comprend l’exemple de code utilisé dans ce guide de démarrage rapide.

   > [!NOTE]
   > Avant de commencer cette procédure, vérifiez que Visual Studio est installé avec la charge de travail **Développement Desktop en C++**.

1. Installez le [système de build CMake](https://cmake.org/download/) comme indiqué dans la page de téléchargement.

1. Ouvrez une invite de commandes ou l’interpréteur de commandes Git Bash. Exécutez les commandes suivantes pour cloner le dépôt GitHub du [SDK Azure IoT pour C](https://github.com/Azure/azure-iot-sdk-c) :

    ```cmd/sh
    git clone -b public-preview https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Cette opération doit prendre quelques minutes.

1. Créez un sous-répertoire *cmake* dans le répertoire racine du dépôt Git et accédez à ce dossier. Exécutez les commandes suivantes à partir du répertoire *azure-iot-sdk-c* :

    ```cmd/sh
    mkdir cmake
    cd cmake
    ```

1. Exécutez les commandes suivantes à partir du répertoire *cmake* pour générer une version du SDK propre à votre plateforme cliente de développement.

   * Dans Linux :

      ```bash
      cmake ..
      make -j
      ```

   * Dans Windows, ouvrez une [invite de commandes développeur pour Visual Studio](/dotnet/framework/tools/developer-command-prompt-for-vs). Exécutez la commande correspondant à votre version de Visual Studio. Ce guide de démarrage rapide utilise Visual Studio 2019. Ces commandes créent une solution Visual Studio pour l’appareil simulé dans le répertoire *cmake*.

      ```cmd
      rem For VS2015
      cmake .. -G "Visual Studio 14 2015"

      rem Or for VS2017
      cmake .. -G "Visual Studio 15 2017"

      rem Or for VS2019
      cmake .. -G "Visual Studio 16 2019"

      rem Then build the project
      cmake --build . -- /m /p:Configuration=Release
      ```

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Inscrire un appareil

Vous devez inscrire un appareil auprès de votre hub IoT pour pouvoir vous y connecter. Dans cette section, vous utilisez Azure Cloud Shell avec l’[extension IoT](/cli/azure/ext/azure-iot/iot?view=azure-cli-latest) pour inscrire un appareil simulé.

1. Pour créer l’identité d’appareil, exécutez la commande suivante dans Cloud Shell :

   > [!NOTE]
   > * Remplacez l’espace réservé *YourIoTHubName* par le nom que vous avez choisi pour votre hub IoT.
   > * Pour le nom de l’appareil que vous êtes en train d’inscrire, nous vous recommandons d’utiliser *MyDevice* comme indiqué. Si vous choisissez un autre nom pour votre appareil, utilisez-le pour l’ensemble de cet article et mettez à jour le nom de l’appareil dans les exemples d’application avant de les exécuter.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyDevice
    ```

1. Pour obtenir la *chaîne de connexion d’appareil* pour celui que vous venez d’inscrire, exécutez la commande suivante dans Cloud Shell :

   > [!NOTE]
   > Remplacez l’espace réservé *YourIoTHubName* par le nom que vous avez choisi pour votre hub IoT.

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name {YourIoTHubName} --device-id MyDevice --output table
    ```

    Notez la chaîne de connexion d’appareil retournée ; vous en aurez besoin plus loin dans ce guide de démarrage rapide. Cela ressemble à l’exemple suivant :

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyDevice;SharedAccessKey={YourSharedAccessKey}`

## <a name="communicate-between-the-device-and-the-service-via-device-streams"></a>Communiquer entre appareil et service par le biais de flux d’appareil

Dans cette section, vous exécutez l’application côté appareil et l’application côté service et communiquez entre les deux.

### <a name="run-the-device-side-application"></a>Exécuter l’application côté appareil

Pour exécuter l’application côté appareil, effectuez les étapes suivantes :

1. Fournissez les informations d’identification de votre appareil en modifiant le fichier source **iothub_client_c2d_streaming_sample.c** dans le dossier `iothub_client/samples/iothub_client_c2d_streaming_sample` et en ajoutant la chaîne de connexion de votre appareil.

   ```C
   /* Paste in your iothub connection string  */
   static const char* connectionString = "{DeviceConnectionString}";
   ```

1. Compilez le code avec les commandes suivantes :

   ```bash
   # In Linux
   # Go to the sample's folder cmake/iothub_client/samples/iothub_client_c2d_streaming_sample
   make -j
   ```

   ```cmd
   rem In Windows
   rem Go to the cmake folder at the root of repo
   cmake --build . -- /m /p:Configuration=Release
   ```

1. Exécutez le programme compilé :

   ```bash
   # In Linux
   # Go to the sample's folder cmake/iothub_client/samples/iothub_client_c2d_streaming_sample
   ./iothub_client_c2d_streaming_sample
   ```

   ```cmd
   rem In Windows
   rem Go to the sample's release folder cmake\iothub_client\samples\iothub_client_c2d_streaming_sample\Release
   iothub_client_c2d_streaming_sample.exe
   ```

### <a name="run-the-service-side-application"></a>Exécuter l’application côté service

Comme mentionné précédemment, le SDK C IoT Hub prend uniquement en charge des flux d’appareil côté appareil. Pour générer et exécuter l’application côté service d’accompagnement, suivez les instructions données dans l’un des guides de démarrage rapide suivants :

* [Communiquer avec une application d’appareil en C# par le biais de flux d'appareil IoT Hub](./quickstart-device-streams-echo-csharp.md)

* [Communiquer avec une application d’appareil en Node.js par le biais de flux d’appareil IoT Hub](./quickstart-device-streams-echo-nodejs.md)

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [iot-hub-quickstarts-clean-up-resources-device-streams](../../includes/iot-hub-quickstarts-clean-up-resources-device-streams.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez configuré un hub IoT, inscrit un appareil, établi un flux d’appareil entre une application C sur l’appareil et une autre application côté service, puis utilisé le flux pour échanger des données entre les applications.

Pour en savoir plus sur les flux d’appareil, consultez :

> [!div class="nextstepaction"]
> [Vue d’ensemble des flux d’appareil](./iot-hub-device-streams-overview.md)
