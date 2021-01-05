---
title: Démarrage rapide (Android) pour envoyer des données de télémétrie à Azure IoT Hub | Microsoft Docs
description: Dans ce guide de démarrage rapide, vous exécutez un exemple d’application Android pour envoyer des données de télémétrie simulée à un hub IoT et lire les données de télémétrie provenant du hub IoT afin de les traiter dans le cloud.
author: wesmc7777
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: quickstart
ms.custom:
- mvc
- mqtt
- devx-track-java
- devx-track-azurecli
ms.date: 03/15/2019
ms.author: wesmc
ms.openlocfilehash: ef5f81090f4d06f15f1a7263699961cd360e12e5
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94842190"
---
# <a name="quickstart-send-iot-telemetry-from-an-android-device"></a>Démarrage rapide : Envoyer des données de télémétrie IoT à partir d’un appareil Android

[!INCLUDE [iot-hub-quickstarts-1-selector](../../includes/iot-hub-quickstarts-1-selector.md)]

Dans ce guide de démarrage rapide, vous envoyez la télémétrie à un hub Azure IoT à partir d’une application Android en cours d’exécution sur un appareil physique ou simulé. IoT Hub est un service Azure qui vous permet de traiter de gros volumes de données de télémétrie envoyées par vos appareils IoT dans le cloud à des fins de stockage ou de traitement. Ce guide de démarrage rapide utilise une application Android prédéfinie pour envoyer la télémétrie. Les données de télémétrie seront lues à partir du hub IoT à l’aide d’Azure Cloud Shell. Avant d’exécuter l’application, vous créez un hub IoT et inscrivez un appareil auprès de ce hub.

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. [Créez-en un gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

* [Android Studio avec le SDK Android 27](https://developer.android.com/studio/). Pour plus d’informations, consultez [android-installation](https://developer.android.com/studio/install). Android SDK 27 est utilisé par l’exemple dans cet article.

* [Un exemple d’application Android](https://github.com/Azure-Samples/azure-iot-samples-java/tree/master/iot-hub/Samples/device/AndroidSample). Cet exemple fait partie du dépôt [azure-iot-samples-java](https://github.com/Azure-Samples/azure-iot-samples-java).

* Le port 8883 ouvert dans votre pare-feu. L’exemple d’appareil de ce guide de démarrage rapide utilise le protocole MQTT, lequel communique sur le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d'informations sur les différentes façons de contourner ce problème, consultez [Se connecter à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Inscrire un appareil

Un appareil doit être inscrit dans votre hub IoT pour pouvoir se connecter. Dans ce démarrage rapide, vous utilisez Azure Cloud Shell pour inscrire un appareil simulé.

1. Exécutez les commandes suivantes dans Azure Cloud Shell pour créer l’identité d’appareil.

   **YourIoTHubName** : Remplacez l’espace réservé ci-dessous par le nom que vous avez choisi pour votre hub IoT.

   **MyAndroidDevice** : il s’agit du nom de l’appareil que vous inscrivez. Il est recommandé d’utiliser **MyAndroidDevice** comme indiqué. Si vous choisissez un autre nom pour votre appareil, vous devrez utiliser ce nom tout au long de cet article et mettre à jour le nom de l'appareil dans les exemples d'application avant de les exécuter.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyAndroidDevice
    ```

2. Exécutez la commande suivante dans Azure Cloud Shell pour obtenir la _chaîne de connexion d’appareil_ pour celui que vous venez d’inscrire :

    **YourIoTHubName** : Remplacez l’espace réservé ci-dessous par le nom que vous avez choisi pour votre hub IoT.

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name {YourIoTHubName} --device-id MyAndroidDevice --output table
    ```

    Notez la chaîne de connexion à l’appareil, qui ressemble à ce qui suit :

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyAndroidDevice;SharedAccessKey={YourSharedAccessKey}`

    Vous utiliserez cette valeur plus loin dans ce guide de démarrage rapide pour envoyer des données de télémétrie.

## <a name="send-simulated-telemetry"></a>Envoyer des données de télémétrie simulée

1. Ouvrez l’exemple de projet Android GitHub dans Android Studio. Le projet se trouve dans le répertoire suivant de votre copie clonée ou téléchargée du dépôt [azure-iot-sample-java](https://github.com/Azure-Samples/azure-iot-samples-java) : *\azure-iot-samples-java\iot-hub\Samples\device\AndroidSample*.

2. Dans Android Studio, ouvrez *gradle.properties* pour l’exemple de projet et remplacez l’espace réservé **Device_Connection_String** par la chaîne de connexion d’appareil que vous avez notée précédemment.

    ```
    DeviceConnectionString=HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyAndroidDevice;SharedAccessKey={YourSharedAccessKey}
    ```

3. Dans Android Studio, cliquez sur **Fichier** > **Synchronisation du projet avec les fichiers Gradle**. Vérifiez que la build est terminée.

   > [!NOTE]
   > Si la synchronisation du projet échoue, c’est peut-être pour l’une des raisons suivantes :
   >
   > * La version du plug-in Android Gradle et celle de Gradle référencées dans le projet sont obsolètes pour votre version d’Android Studio. Suivez [ces instructions](https://developer.android.com/studio/releases/gradle-plugin) pour référencer et installer les bonnes versions du plug-in et de Gradle pour votre installation.
   > * Le contrat de licence du kit de développement logiciel Android SDK n’a pas été signé. Suivez les instructions dans la sortie de la build pour signer le contrat de licence et télécharger le SDK.

4. Une fois la build terminée, cliquez sur **Exécuter** > **Exécuter « application »** . Configurez l’application pour l’exécuter sur un appareil Android physique ou un émulateur Android. Pour plus d’informations sur l’exécution d’une application Android sur un appareil physique ou un émulateur, consultez [Exécuter votre application](https://developer.android.com/training/basics/firstapp/running-app).

5. Une fois l’application chargée, cliquez sur le bouton **Démarrer** pour commencer à envoyer les données de télémétrie à votre hub IoT :

    ![Application](media/quickstart-send-telemetry-android/sample-screenshot.png)


## <a name="read-the-telemetry-from-your-hub"></a>Lire les données de télémétrie envoyées par votre hub

Dans cette section, vous allez utiliser Azure Cloud Shell avec l’[extension IoT](/cli/azure/ext/azure-iot/iot?view=azure-cli-latest) pour surveiller les messages d’appareil envoyés par l’appareil Android.

1. À l’aide d’Azure Cloud Shell, exécutez la commande suivante pour vous connecter et lire les messages à partir de votre hub IoT :

   **YourIoTHubName** : Remplacez l’espace réservé ci-dessous par le nom que vous avez choisi pour votre hub IoT.

    ```azurecli-interactive
    az iot hub monitor-events --hub-name {YourIoTHubName} --output table
    ```

    La capture d’écran suivante montre la sortie lorsque le hub IoT reçoit les données de télémétrie envoyées par l’appareil Android :

      ![Lire les messages d’appareil à l’aide de l’interface Azure CLI](media/quickstart-send-telemetry-android/read-data.png)
## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez configuré un hub IoT, inscrit un appareil, envoyé des données de télémétrie simulée au hub à l’aide d’une application Android et lu les données de télémétrie à partir du hub en utilisant Azure Cloud Shell.

Pour savoir comment contrôler votre appareil simulé à partir d’une application back-end, passez au démarrage rapide suivant.

> [!div class="nextstepaction"]
> [Démarrage rapide : Contrôler un appareil connecté à un hub IoT](quickstart-control-device-android.md)
