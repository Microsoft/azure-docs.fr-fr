---
title: Développer pour la plateforme Android Things à l’aide des Kits de développement logiciel (SDK) Azure IoT | Microsoft Docs
description: 'Guide du développeur : découvrez comment développer sur Android Things à l’aide des Kits de développement logiciel (SDK) Azure IoT Hub.'
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/30/2019
ms.author: robinsh
ms.openlocfilehash: 6008ca1549629bcbb582f38de2ab1b453cd73a99
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94843755"
---
# <a name="develop-for-android-things-platform-using-azure-iot-sdks"></a>Développer pour la plateforme Android Things à l’aide des Kits de développement logiciel (SDK) Azure IoT

Les [Kits de développement logiciel (SDK) Azure IoT Hub](./iot-hub-devguide-sdks.md) assurent la prise en charge de premier niveau pour les plateformes populaires telles que Windows, Linux, OSX, MBED ainsi que les plateformes mobiles telles qu’iOS et Android.  Dans le cadre de notre engagement à offrir davantage de choix et de flexibilité dans les déploiements IoT, le Kit de développement logiciel (SDK) Java prend également en charge la plateforme [Android Things](https://developer.android.com/things/).  Les développeurs peuvent tirer parti des avantages du système d’exploitation Android Things côté appareil, tout en utilisant [Azure IoT Hub](about-iot-hub.md) en tant que plateforme de messagerie centrale qui se met à l’échelle de millions d’appareils connectés simultanément.

Ce didacticiel décrit les étapes à suivre pour créer une application côté appareil sur Android Things à l’aide du Kit de développement logiciel (SDK) Java Azure IoT.

## <a name="prerequisites"></a>Prérequis

* Matériel compatible Android Things avec le système d’exploitation Android Things en cours d’exécution.  Pour savoir comment flasher le système d’exploitation Android Things, vous pouvez suivre la [documentation d’Android Things](https://developer.android.com/things/get-started/kits#flash-at).  Assurez-vous que votre appareil Android Things est connecté à Internet et dispose des périphériques essentiels tels qu’un clavier, un écran et une souris.  Ce didacticiel utilise Raspberry Pi 3.

* Dernière version d’[Android Studio](https://developer.android.com/studio/)

* Dernière version de [Git](https://git-scm.com/)

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Inscrire un appareil

Un appareil doit être inscrit dans votre hub IoT pour pouvoir se connecter. Dans ce démarrage rapide, vous utilisez Azure Cloud Shell pour inscrire un appareil simulé.

1. Exécutez les commandes suivantes dans Azure Cloud Shell pour ajouter l’extension CLI IoT Hub et créer l’identité d’appareil.

   **YourIoTHubName** : Remplacez l’espace réservé ci-dessous par le nom que vous avez choisi pour votre hub IoT.

   **MyAndroidThingsDevice** : il s’agit du nom donné à l’appareil inscrit. Utilisez MyAndroidThingsDevice comme indiqué. Si vous choisissez un autre nom pour votre appareil, vous devez également utiliser ce nom pour l’ensemble de cet article et mettre à jour le nom de l’appareil dans les exemples d’application avant de les exécuter.

    ```azurecli-interactive
    az extension add --name azure-iot
    az iot hub device-identity create --hub-name YourIoTHubName --device-id MyAndroidThingsDevice
    ```

2. Exécutez les commandes suivantes dans Azure Cloud Shell pour obtenir la *chaîne de connexion d’appareil* pour celui que vous venez d’inscrire. Remplacez `YourIoTHubName` ci-dessous par le nom que vous avez choisi pour votre IoT Hub.

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name YourIoTHubName --device-id MyAndroidThingsDevice --output table
    ```

    Notez la chaîne de connexion à l’appareil, qui ressemble à ce qui suit :

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyAndroidThingsDevice;SharedAccessKey={YourSharedAccessKey}`

    Vous utiliserez cette valeur plus loin dans ce démarrage rapide.

## <a name="building-an-android-things-application"></a>Création d’une application Android Things

1. La première étape pour créer une application Android Things consiste à se connecte à vos appareils Android Things. Connectez votre appareil Android Things à un écran, puis connectez le à internet. Android Things fournit une [documentation](https://developer.android.com/things/get-started/kits) sur la façon de se connecter au WiFi. Une fois connecté à internet, prenez note de l’adresse IP répertoriée sous Réseaux.

2. Servez-vous de l’outil [adb](https://developer.android.com/studio/command-line/adb) outil pour vous connecter à votre appareil Android Things en utilisant l’adresse IP indiquée ci-dessus. Vérifiez soigneusement la connexion à l’aide de cette commande à partir de votre terminal. Vous voyez normalement vos appareils listés comme étant connectés.

   ```
   adb devices
   ```

3. Téléchargez notre exemple pour Android/Android Things à partir de ce [référentiel](https://github.com/Azure-Samples/azure-iot-samples-java) ou utilisez Git.

   ```
   git clone https://github.com/Azure-Samples/azure-iot-samples-java.git
   ```

4. Dans Android Studio, ouvrez le projet Android situé dans « \azure-iot-samples-java\iot-hub\Samples\device\AndroidSample ».

5. Ouvrez le fichier gradle.properties et remplacez « Device_connection_string » par la chaîne de connexion de votre appareil indiquée précédemment.
 
6. Cliquez sur Run – Debug, puis sélectionnez votre appareil pour déployer ce code sur vos appareils Android Things.

7. Une fois l’application correctement démarrée, vous pouvez voir une application en cours d’exécution sur votre appareil Android Things. Cet exemple d’application envoie des relevés de température générés de manière aléatoire.

## <a name="read-the-telemetry-from-your-hub"></a>Lire les données de télémétrie envoyées par votre hub

Vous pouvez voir les données dans votre hub IoT, au fur et à mesure de leur réception. L’extension CLI IoT Hub peut se connecter au point de terminaison **Événements** côté service sur votre IoT Hub. L’extension reçoit les messages appareil-à-cloud envoyés à partir de votre appareil simulé. Une application back-end IoT Hub s’exécute généralement dans le cloud pour recevoir et traiter les messages appareil-à-cloud.

Exécutez les commandes suivantes dans Azure Cloud Shell, en remplaçant `YourIoTHubName` par le nom de votre hub IoT :

```azurecli-interactive
az iot hub monitor-events --device-id MyAndroidThingsDevice --hub-name YourIoTHubName
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [gérer la connectivité et la messagerie fiable](iot-hub-reliability-features-in-sdks.md) à l’aide des kits IoT Hub SDK.
* Découvrez comment [développer pour des plateformes mobiles](iot-hub-how-to-develop-for-mobile-devices.md) telles qu’iOS et Android.
* [Prise en charge des plateformes par le kit de développement logiciel (SDK) Azure IoT](iot-hub-device-sdk-platform-support.md)