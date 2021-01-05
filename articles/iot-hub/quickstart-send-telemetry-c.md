---
title: Démarrage rapide (C) pour envoyer des données de télémétrie à Azure IoT Hub | Microsoft Docs
description: Dans ce guide de démarrage rapide, vous exécutez deux exemples d’application C pour envoyer des données de télémétrie simulée à un hub IoT, et pour lire les données de télémétrie provenant du hub IoT à traiter dans le cloud.
author: wesmc7777
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.devlang: c
ms.topic: quickstart
ms.custom:
- mvc
- mqtt
- 'Role: Cloud Development'
- devx-track-azurecli
ms.date: 04/10/2019
ms.author: wesmc
ms.openlocfilehash: 7311c7e700edbf8565a449ecbd045a91846e72c3
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94844759"
---
# <a name="quickstart-send-telemetry-from-a-device-to-an-iot-hub-and-read-it-with-a-back-end-application-c"></a>Démarrage rapide : Envoyer des données de télémétrie d’un appareil à un hub IoT et les lire avec une application back-end (C)

[!INCLUDE [iot-hub-quickstarts-1-selector](../../includes/iot-hub-quickstarts-1-selector.md)]

IoT Hub est un service Azure qui vous permet de traiter de gros volumes de données de télémétrie envoyées par vos appareils IoT dans le cloud à des fins de stockage ou de traitement. Dans ce guide de démarrage rapide, vous envoyez des données de télémétrie à partir d’une application d’appareil simulé, via IoT Hub, à une application back-end pour traitement.

Le guide de démarrage rapide utilise un exemple d’application C à partir du [kit SDK Azure IoT device SDK for C](iot-hub-device-sdk-c-intro.md) pour envoyer des données de télémétrie à un hub IoT. Les kits SDK Azure IoT device sont écrits en [ANSI C (C99)](https://wikipedia.org/wiki/C99) pour la portabilité et la compatibilité avec de nombreuses plateformes. Avant d’exécuter l’exemple de code, vous devez créer un hub IoT et inscrire l’appareil simulé avec ce hub.

Cet article est écrit pour Windows, mais vous pouvez également suivre ce guide de démarrage rapide sur Linux.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* Installez [Visual Studio 2019](https://www.visualstudio.com/vs/) avec la charge de travail [« Développement Desktop en C++ »](https://www.visualstudio.com/vs/support/selecting-workloads-visual-studio-2017/) activée.

* Installez la dernière version de [Git](https://git-scm.com/download/).

* Assurez-vous que le port 8883 est ouvert dans votre pare-feu. L’exemple d’appareil de ce guide de démarrage rapide utilise le protocole MQTT, lequel communique sur le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d’informations sur les façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="prepare-the-development-environment"></a>Préparer l’environnement de développement

Pour ce guide de démarrage rapide, vous devez utiliser [Azure IoT device SDK for C](iot-hub-device-sdk-c-intro.md). 

Pour les environnements suivants, vous pouvez utiliser le SDK en installant les packages et bibliothèques suivants :

* **Linux** : les packages apt-get sont disponibles pour Ubuntu 16.04 et 18.04, avec les architectures d'UC suivantes : amd64, arm64, armhf et i386. Pour plus d’informations, consultez [Utilisation de apt-get pour créer un projet client d’appareil C sur Ubuntu](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/ubuntu_apt-get_sample_setup.md).

* **mbed** : pour les développeurs créant des applications d'appareil sur la plateforme mbed, nous avons publié une bibliothèque et des exemples qui les aideront à se familiariser avec Azure IoT Hub en quelques minutes. Pour plus d’informations, consultez [Utiliser la bibliothèque mbed](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/readme.md#mbed).

* **Arduino** : si vous développez sur Arduino, vous pouvez exploiter la bibliothèque Azure IoT disponible dans le Gestionnaire de bibliothèques de l'IDE Arduino. Pour plus d’informations, consultez [La bibliothèque Azure IoT Hub pour Arduino](https://github.com/azure/azure-iot-arduino).

* **iOS** : le kit SDK IoT Hub Device SDK est disponible en tant que CocoaPods pour le développement d’appareils iOS et Mac. Pour plus d’informations, consultez [Exemples iOS pour Microsoft Azure IoT](https://cocoapods.org/pods/AzureIoTHubClient).

Dans cette section, en revanche, vous allez préparer un environnement de développement pour cloner et générer le [kit Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c) à partir de GitHub. Le kit SDK sur GitHub comprend l’exemple de code utilisé dans ce guide de démarrage rapide.

1. Téléchargez le [système de génération CMake](https://cmake.org/download/).

    Il est important que les composants requis Visual Studio (Visual Studio et la charge de travail « Développement Desktop en C++ ») soient installés sur votre machine, **avant** de commencer l’installation de l’élément `CMake`. Une fois les composants requis en place et le téléchargement effectué, installez le système de génération de CMake.

2. Recherchez le nom de balise de la [version la plus récente](https://github.com/Azure/azure-iot-sdk-c/releases/latest) du kit de développement logiciel (SDK).

3. Ouvrez une invite de commandes ou l’interpréteur de commandes Git Bash. Exécutez les commandes suivantes pour cloner la dernière version du dépôt GitHub du [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c). Utilisez l’étiquette obtenue à l’étape précédente comme valeur pour le paramètre `-b` :

    ```cmd/sh
    git clone -b <release-tag> https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Attendez-vous à ce que cette opération prenne plusieurs minutes.

4. Créez un sous-répertoire `cmake` dans le répertoire racine du référentiel Git et accédez à ce dossier. Exécutez les commandes suivantes à partir du répertoire `azure-iot-sdk-c` :

    ```cmd/sh
    mkdir cmake
    cd cmake
    ```

5. Exécutez la commande suivante qui génère une version du SDK propre à votre plateforme cliente de développement. Une solution Visual Studio pour l’appareil simulé est générée dans le répertoire `cmake`.

    ```cmd
    cmake ..
    ```

    Si `cmake` ne trouve pas votre compilateur C++, vous risquez de rencontrer des erreurs de build lors de l'exécution de la commande ci-dessus. Si cela se produit, essayez d’exécuter cette commande dans [l’invite de commandes de Visual Studio](/dotnet/framework/tools/developer-command-prompt-for-vs). 

    Une fois la génération terminée, les dernières lignes de sortie doivent ressembler à la sortie suivante :

    ```cmd/sh
    $ cmake ..
    -- Building for: Visual Studio 15 2017
    -- Selecting Windows SDK version 10.0.16299.0 to target Windows 10.0.17134.
    -- The C compiler identification is MSVC 19.12.25835.0
    -- The CXX compiler identification is MSVC 19.12.25835.0

    ...

    -- Configuring done
    -- Generating done
    -- Build files have been written to: E:/IoT Testing/azure-iot-sdk-c/cmake
    ```

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Inscrire un appareil

Un appareil doit être inscrit dans votre hub IoT pour pouvoir se connecter. Dans cette section, vous allez utiliser Azure Cloud Shell avec l'[extension IoT](/cli/azure/ext/azure-iot/iot?view=azure-cli-latest) pour inscrire un appareil simulé.

1. Exécutez les commandes suivantes dans Azure Cloud Shell pour créer l’identité d’appareil.

   **YourIoTHubName** : Remplacez l’espace réservé ci-dessous par le nom que vous avez choisi pour votre hub IoT.

   **MyCDevice** : il s’agit du nom de l’appareil que vous inscrivez. Il est recommandé d’utiliser **MyCDevice** comme indiqué. Si vous choisissez un autre nom pour votre appareil, vous devrez utiliser ce nom tout au long de cet article et mettre à jour le nom de l'appareil dans les exemples d'application avant de les exécuter.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyCDevice
    ```

2. Exécutez la commande suivante dans Azure Cloud Shell pour obtenir la _chaîne de connexion d’appareil_ pour celui que vous venez d’inscrire :

   **YourIoTHubName** : Remplacez l’espace réservé ci-dessous par le nom que vous avez choisi pour votre hub IoT.

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name {YourIoTHubName} --device-id MyCDevice --output table
    ```

    Notez la chaîne de connexion à l’appareil, qui ressemble à ce qui suit :

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyCDevice;SharedAccessKey={YourSharedAccessKey}`

    Vous utiliserez cette valeur plus loin dans ce guide de démarrage rapide.

## <a name="send-simulated-telemetry"></a>Envoyer des données de télémétrie simulée

L’application d’appareil simulé se connecte à un point de terminaison spécifique à l’appareil sur votre hub IoT et envoie une chaîne sous la forme de données de télémétrie simulée.

1. À l’aide d’un éditeur de texte, ouvrez le fichier source iothub_convenience_sample.c et examinez l’exemple de code pour l’envoi des données de télémétrie. Le fichier se trouve à l’emplacement suivant dans le répertoire de travail où vous avez cloné le SDK Azure IoT C :

    ```
    azure-iot-sdk-c\iothub_client\samples\iothub_convenience_sample\iothub_convenience_sample.c
    ```

2. Recherchez la déclaration de la constante `connectionString` :

    ```C
    /* Paste in your device connection string  */
    static const char* connectionString = "[device connection string]";
    ```

    Remplacez la valeur de la constante `connectionString` par la chaîne de connexion d’appareil que vous avez notée précédemment. Enregistrez ensuite vos modifications dans **iothub_convenience_sample.c**.

3. Dans une fenêtre de terminal local, accédez au répertoire du projet *iothub_convenience_sample*, dans le répertoire CMake que vous avez créé dans le Kit de développement logiciel (SDK) Azure IoT C. Entrez la commande suivante à partir de votre répertoire de travail :

    ```cmd/sh
    cd azure-iot-sdk-c/cmake/iothub_client/samples/iothub_convenience_sample
    ```

4. Exécutez CMake dans la fenêtre du terminal local pour générer l’exemple avec votre valeur `connectionString` mise à jour :

    ```cmd/sh
    cmake --build . --target iothub_convenience_sample --config Debug
    ```

5. Dans la fenêtre du terminal local, exécutez la commande suivante pour démarrer l’application d’appareil simulé :

    ```cmd/sh
    Debug\iothub_convenience_sample.exe
    ```

    La capture d’écran suivante montre la sortie lorsque l’application d’appareil simulé envoie des données de télémétrie au hub IoT :

    ![Exécuter l’appareil simulé](media/quickstart-send-telemetry-c/simulated-device-app.png)

## <a name="read-the-telemetry-from-your-hub"></a>Lire les données de télémétrie envoyées par votre hub

Dans cette section, vous allez utiliser Azure Cloud Shell avec l'[extension IoT](/cli/azure/ext/azure-iot/iot?view=azure-cli-latest) pour surveiller les messages envoyés par l'appareil simulé.

1. À l’aide d’Azure Cloud Shell, exécutez la commande suivante pour vous connecter et lire les messages à partir de votre hub IoT :

   **YourIoTHubName** : Remplacez l’espace réservé ci-dessous par le nom que vous avez choisi pour votre hub IoT.

    ```azurecli-interactive
    az iot hub monitor-events --hub-name {YourIoTHubName} --output table
    ```

    ![Lire les messages d’appareil à l’aide de l’interface Azure CLI](media/quickstart-send-telemetry-c/read-device-to-cloud-messages-app.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez configuré un hub IoT, inscrit un appareil, envoyé des données de télémétrie simulée au hub avec une application C et lu les données de télémétrie à partir du hub en utilisant Azure Cloud Shell.

Pour en apprendre davantage sur le développement avec le kit Azure IoT Hub C SDK, continuez avec le guide pratique suivant :

> [!div class="nextstepaction"]
> [Développer avec le kit SDK C Azure IoT Hub](iot-hub-devguide-develop-for-constrained-devices.md)
