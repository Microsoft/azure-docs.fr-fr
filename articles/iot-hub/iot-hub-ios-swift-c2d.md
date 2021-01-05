---
title: Messages cloud-à-appareil avec Azure IoT Hub (iOS) | Microsoft Docs
description: Envoi de messages cloud-à-appareil vers un appareil depuis un Azure IoT Hub à l’aide des kits de développement logiciel Azure IoT pour iOS.
author: kgremban
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/19/2018
ms.author: kgremban
ms.custom: mqtt
ms.openlocfilehash: 15c0df33b8f09ec71f2be913d72f0785dc766375
ms.sourcegitcommit: daab0491bbc05c43035a3693a96a451845ff193b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "93027531"
---
# <a name="send-cloud-to-device-messages-with-iot-hub-ios"></a>Envoi de messages cloud à appareil avec IoT Hub (iOS)

[!INCLUDE [iot-hub-selector-c2d](../../includes/iot-hub-selector-c2d.md)]

Azure IoT Hub est un service entièrement géré qui permet d’autoriser des communications bidirectionnelles fiables et sécurisées entre des millions d’appareils et un serveur principal de solution. Le guide de démarrage rapide [Envoyer des données de télémétrie d’un appareil à un hub IoT](quickstart-send-telemetry-ios.md) explique comment créer un hub IoT, y provisionner une identité d’appareil et coder une application d’appareil simulé qui envoie des messages appareil-à-cloud.

Ce didacticiel vous explique les procédures suivantes :

* À partir du serveur principal de votre application, envoyez des messages cloud-à-appareil vers un appareil unique via IoT Hub.

* Recevez des messages cloud-à-appareil sur un appareil.

* À partir du back-end de votre solution, demandez l’accusé de réception ( *commentaires* ) pour les messages envoyés à un appareil depuis IoT Hub.

Vous trouverez des informations supplémentaires sur les messages cloud-à-appareil dans la [section sur les messages du Guide du développeur IoT Hub](iot-hub-devguide-messaging.md).

À la fin de cet article, vous exécutez deux projets iOS Swift :

* **sample-device** , la même application créée dans [Send telemetry from a device to an IoT hub (Envoyer des données de télémétrie d’un appareil à un IoT Hub)](quickstart-send-telemetry-ios.md), qui se connecte à votre IoT Hub et reçoit des messages cloud-à-appareil.

* **sample-service** , qui envoie un message cloud-à-appareil à l’application d’appareil simulé par le biais d’IoT Hub, puis reçoit son accusé de réception.

> [!NOTE]
> IoT Hub offre la prise en charge de Kits de développement logiciel (SDK) pour plusieurs plateformes d’appareils et plusieurs langages (notamment C, Java, Python et Javascript) par le biais des Kits Azure IoT device SDK. Pour obtenir des instructions détaillées sur la façon de connecter votre appareil au code de ce didacticiel, et à Azure IoT Hub de manière générale, consultez le [Centre de développement Azure IoT](https://www.azure.com/develop/iot).

## <a name="prerequisites"></a>Conditions préalables requises

* Un compte Azure actif. (Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes seulement.)

* Un IoT Hub actif dans Azure.

* L’exemple de code dans les [exemples Azure](https://github.com/Azure-Samples/azure-iot-samples-ios/archive/master.zip).

* La dernière version de [XCode](https://developer.apple.com/xcode/), exécutant la dernière version du Kit de développement logiciel (SDK) iOS. Ce démarrage rapide a été testé avec XCode 9.3 et iOS 11.3.

* La dernière version de [CocoaPods](https://guides.cocoapods.org/using/getting-started.html).

* Assurez-vous que le port 8883 est ouvert dans votre pare-feu. L’exemple d’appareil décrit dans cet article utilise le protocole MQTT, qui communique via le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d’informations sur les façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="simulate-an-iot-device"></a>Simuler un appareil IoT

Dans cette section, vous simulez un appareil iOS exécutant une application Swift pour recevoir des messages cloud-à-appareil provenant de l’Iot Hub. 

Il s’agit de l’exemple d’appareil que vous créez dans l’article [Send telemetry from a device to an IoT hub (Envoyer des données de télémétrie d’un appareil à un IoT Hub)](quickstart-send-telemetry-ios.md). Si vous avez déjà cet appareil, vous pouvez ignorer cette section.

### <a name="install-cocoapods"></a>Installer les CocoaPods

Les CocoaPods gèrent les dépendances des projets iOS qui utilisent des bibliothèques tierces.

Dans une fenêtre de terminal, accédez au dossier Azure-IoT-Samples-iOS que vous avez téléchargé dans les conditions préalables. Ensuite, accédez à l’exemple de projet :

```sh
cd quickstart/sample-device
```

Assurez-vous que XCode est fermé, puis exécutez la commande suivante pour installer les CocoaPods déclarés dans le fichier **podfile** :

```sh
pod install
```

Outre l’installation des pods nécessaires à votre projet, la commande d’installation crée également un fichier d’espace de travail XCode qui est déjà configuré pour utiliser les blocs pour les dépendances.

### <a name="run-the-sample-device-application"></a>Exécuter l’application d’appareil exemple

1. Récupérez la chaîne de connexion de votre appareil. Vous pouvez copier cette chaîne à partir du [portail Azure](https://portal.azure.com) dans le panneau des détails de l’appareil ou la récupérer avec la commande CLI suivante :

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name {YourIoTHubName} --device-id {YourDeviceID} --output table
    ```

1. Ouvrez l’exemple d’espace de travail dans XCode.

   ```sh
   open "MQTT Client Sample.xcworkspace"
   ```

2. Développez le projet **MQTT Client Sample** , puis le dossier du même nom.  

3. Ouvrez **ViewController.swift** pour le modifier dans XCode. 

4. Recherchez la variable **connectionString** et mettez à jour sa valeur en fonction de la chaîne de connexion d’appareil que vous avez copiée à la première étape.

5. Enregistrez vos modifications. 

6. Exécutez le projet dans l’émulateur d’appareil avec le bouton **Build and run (Générer et exécuter)** ou la combinaison de touches **commande + r**.

   ![Capture d’écran affichant le bouton Générer et exécuter dans l’émulateur d’appareil.](media/iot-hub-ios-swift-c2d/run-sample.png)

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

Dans cet article, vous créez un service back-end pour envoyer des messages cloud-à-appareil via l’IoT Hub que vous avez créé dans [Send telemetry from a device to an IoT hub (Envoyer des données de télémétrie d’un appareil à un IoT Hub)](quickstart-send-telemetry-ios.md). Pour envoyer des messages cloud-à-appareil, votre service a besoin de l'autorisation de **connexion de service**. Par défaut, chaque IoT Hub est créé avec une stratégie d’accès partagé nommée **service** qui accorde cette autorisation.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="simulate-a-service-device"></a>Simuler un appareil de service

Dans cette section, vous simulez un deuxième appareil iOS avec une application Swift qui envoie des messages cloud-à-appareil via l’IoT Hub. Cette configuration est utile pour les scénarios d’IoT dans lequel un iPhone ou un iPad fonctionne comme un contrôleur pour les autres appareils iOS connectés à un IoT Hub.

### <a name="install-cocoapods"></a>Installer les CocoaPods

Les CocoaPods gèrent les dépendances des projets iOS qui utilisent des bibliothèques tierces.

Accédez au dossier Azure IoT iOS Samples que vous avez téléchargé dans les conditions préalables. Ensuite, accédez à l’exemple de projet de service :

```sh
cd quickstart/sample-service
```

Assurez-vous que XCode est fermé, puis exécutez la commande suivante pour installer les CocoaPods déclarés dans le fichier **podfile** :

```sh
pod install
```

Outre l’installation des pods nécessaires à votre projet, la commande d’installation crée également un fichier d’espace de travail XCode qui est déjà configuré pour utiliser les blocs pour les dépendances.

### <a name="run-the-sample-service-application"></a>Exécuter l’exemple d’application de service

1. Ouvrez l’exemple d’espace de travail dans XCode.

   ```sh
   open AzureIoTServiceSample.xcworkspace
   ```

2. Développez le projet **AzureIoTServiceSample** , puis le dossier du même nom.  

3. Ouvrez **ViewController.swift** pour le modifier dans XCode. 

4. Recherchez la variable **connectionString** et mettez à jour sa valeur en fonction de la chaîne de connexion de service que vous avez copiée précédemment dans [Get the IoT hub connection string](#get-the-iot-hub-connection-string).

5. Enregistrez vos modifications.

6. Dans Xcode, remplacez les paramètres d’émulateur par un appareil iOS autre que celui utilisé pour exécuter le périphérique IoT. XCode ne peut pas exécuter plusieurs émulateurs du même type.

   ![Changer l’émulateur d’appareil](media/iot-hub-ios-swift-c2d/change-device.png)

7. Exécutez le projet dans l’émulateur d’appareil avec le bouton **Build and run (Générer et exécuter)** ou la combinaison de touches **commande + r**.

   ![Capture d’écran affichant le bouton Générer et exécuter.](media/iot-hub-ios-swift-c2d/run-app.png)

## <a name="send-a-cloud-to-device-message"></a>Envoi d’un message cloud vers appareil

Vous êtes maintenant prêt à utiliser les deux applications pour envoyer et recevoir des messages cloud-à-appareil.

1. Dans l’application **iOS App Sample** exécutée sur l’appareil IoT simulé, cliquez sur **Démarrer**. L’application commence à envoyer des messages appareil-à-cloud, mais aussi à écouter les messages cloud-à-appareil.

   ![Afficher l’exemple d’application d’appareil IoT](media/iot-hub-ios-swift-c2d/view-d2c.png)

2. Dans l’application **IoTHub Service Client Sample** exécutée sur l’appareil de service simulé, entrez l’ID de l’appareil IoT auquel vous souhaitez envoyer un message. 

3. Écrivez un message en texte brut, puis cliquez sur **Envoyer**.

    Plusieurs actions s’exécutent lorsque vous cliquez sur Envoyer. L’exemple de service envoie le message à votre IoT Hub, auquel l’application a accès, selon la chaîne de connexion de service que vous avez fournie. Votre IoT Hub vérifie l’ID d’appareil, envoie le message à l’appareil de destination et envoie un accusé de réception à l’appareil source. L’application exécutée sur votre appareil IoT simulé recherche les messages provenant de l’IoT Hub et affiche le texte du message le plus récent.

    Votre sortie doit ressembler à l’exemple suivant :

   ![Afficher les messages cloud-à-appareil](media/iot-hub-ios-swift-c2d/view-c2d.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à envoyer et recevoir des messages cloud-à-appareil.

Pour voir des exemples de solutions de bout en bout complètes, qui utilisent IoT Hub, consultez la [documentation sur les accélérateurs de solution Azure IoT](https://azure.microsoft.com/documentation/suites/iot-suite/).

Pour en savoir plus sur le développement de solutions avec IoT Hub, consultez le [Guide du développeur d’IoT Hub](iot-hub-devguide.md).
