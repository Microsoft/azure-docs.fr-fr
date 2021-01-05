---
title: Messages cloud-à-appareil avec Azure IoT Hub (Node) | Microsoft Docs
description: Envoi de messages cloud-à-appareil vers un appareil depuis un Azure IoT Hub à l’aide des kits de développement logiciel Azure IoT pour Node.js. Vous modifiez une application d’appareil simulé pour recevoir des messages cloud-à-appareil et modifiez une application principale pour envoyer des messages cloud-à-appareil.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: javascript
ms.topic: conceptual
ms.date: 06/16/2017
ms.custom:
- amqp
- mqtt
- devx-track-js
ms.openlocfilehash: e398138f12c38e5235a0004679d9574dbde607db
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91446887"
---
# <a name="send-cloud-to-device-messages-with-iot-hub-nodejs"></a>Envoi de messages cloud à appareil avec IoT Hub (Node.js)

[!INCLUDE [iot-hub-selector-c2d](../../includes/iot-hub-selector-c2d.md)]

Azure IoT Hub est un service entièrement géré qui permet d’autoriser des communications bidirectionnelles fiables et sécurisées entre des millions d’appareils et un serveur principal de solution. Le guide de démarrage rapide [Envoyer des données de télémétrie d’un appareil à un hub IoT](quickstart-send-telemetry-node.md) explique comment créer un hub IoT, y provisionner une identité d’appareil et coder une application d’appareil simulé qui envoie des messages appareil-à-cloud.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Ce tutoriel est basé sur [Envoyer des données de télémétrie d’un appareil à un hub IoT](quickstart-send-telemetry-node.md). Cette rubrique vous explique les procédures suivantes :

* À partir du serveur principal de votre application, envoyez des messages cloud-à-appareil vers un appareil unique via IoT Hub.
* Recevez des messages cloud-à-appareil sur un appareil.
* À partir du back-end de votre solution, demandez l’accusé de réception (*commentaires*) pour les messages envoyés à un appareil depuis IoT Hub.

Vous trouverez des informations supplémentaires sur les messages cloud-à-appareil dans le [Guide du développeur IoT Hub](iot-hub-devguide-messaging.md).

À la fin de ce didacticiel, vous exécuterez deux applications de console Node.js :

* **SimulatedDevice**, une version modifiée de l’application créée dans [Envoyer des données de télémétrie d’un appareil à un hub IoT](quickstart-send-telemetry-node.md), qui se connecte à votre hub IoT et reçoit des messages cloud-à-appareil.

* **SendCloudToDeviceMessage**, qui envoie un message cloud-à-appareil à l’application pour appareil simulée par le biais d’IoT Hub, puis reçoit son accusé de réception.

> [!NOTE]
> IoT Hub offre la prise en charge de Kits de développement logiciel (SDK) pour plusieurs plateformes d’appareils et plusieurs langages (notamment C, Java, Python et Javascript) par le biais des Kits Azure IoT device SDK. Pour obtenir des instructions détaillées sur la façon de connecter votre appareil au code de ce didacticiel, et à Azure IoT Hub de manière générale, consultez le [Centre de développement Azure IoT](https://azure.microsoft.com/develop/iot).
>

## <a name="prerequisites"></a>Prérequis

* Node.j version 10.0.x ou ultérieure. L’article [Préparer votre environnement de développement](https://github.com/Azure/azure-iot-sdk-node/tree/master/doc/node-devbox-setup.md) décrit l’installation de Node.js pour ce didacticiel sur Windows ou sur Linux.

* Un compte Azure actif. (Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial) en quelques minutes seulement.)

* Vérifiez que le port 8883 est ouvert dans votre pare-feu. L’exemple d’appareil décrit dans cet article utilise le protocole MQTT, qui communique via le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d'informations sur les différentes façons de contourner ce problème, consultez [Se connecter à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="receive-messages-in-the-simulated-device-app"></a>Recevoir des messages dans l’application d’appareil simulé

Dans cette section, vous modifiez l’application d’appareil simulé créée dans [Envoyer des données de télémétrie d’un appareil à un IoT Hub](quickstart-send-telemetry-node.md) pour recevoir des messages cloud-à-appareil en provenance du hub IoT.

1. À l'aide d'un éditeur de texte, ouvrez le fichier **SimulatedDevice.js**. Ce fichier se trouve dans le dossier **iot-hub\Quickstarts\simulated-device** du dossier racine de l'exemple de code Node.js que vous avez téléchargé dans le guide de démarrage rapide [Envoyer des données de télémétrie d'un appareil vers un hub IoT](quickstart-send-telemetry-node.md).

2. Enregistrez un gestionnaire sur le client d'appareil pour recevoir les messages envoyés depuis IoT Hub. Ajoutez l'appel à `client.on` juste après la ligne qui crée le client d'appareil, comme dans l'extrait de code suivant :

    ```javascript
    var client = DeviceClient.fromConnectionString(connectionString, Mqtt);

    client.on('message', function (msg) {
      console.log('Id: ' + msg.messageId + ' Body: ' + msg.data);
      client.complete(msg, function (err) {
        if (err) {
          console.error('complete error: ' + err.toString());
        } else {
          console.log('complete sent');
        }
      });
    });
    ```

Dans cet exemple, l'appareil appelle la fonction **complete** afin de notifier IoT Hub qu'il a traité le message et que celui-ci peut être supprimé de la file d'attente de l'appareil. L'appel de la fonction **complete** n'est pas nécessaire et peut être omis si vous utilisez le transport MQTT. En revanche, il est obligatoire pour AMQP et HTTPS.

Avec AMQP et HTTPS, mais pas MQTT, l'appareil peut également :

* abandonner un message - IoT Hub conserve alors le message dans la file d'attente de l'appareil pour un traitement ultérieur ;
* rejeter un message, ce qui le supprime définitivement de la file d'attente de l'appareil.

S'il se produit un événement qui empêche l'appareil de traiter, d'abandonner ou de rejeter le message, IoT Hub le met à nouveau en file d'attente après un délai d'attente déterminé. C'est la raison pour laquelle la logique de traitement des messages de l'application pour périphérique doit être *idempotente* pour qu'un message identique reçu plusieurs fois produise le même résultat.

Pour plus d'informations sur la façon dont IoT Hub traite les messages cloud-à-appareil, y compris sur le cycle de vie des messages cloud-à-appareil, consultez [Envoyer des messages cloud-à-appareil à partir d'un hub IoT](iot-hub-devguide-messages-c2d.md).
  
> [!NOTE]
> Si vous utilisez HTTPS plutôt que MQTT ou AMQP comme moyen de transport, l'instance **DeviceClient** ne vérifie pas très souvent les messages provenant d'IoT Hub (au minimum toutes les 25 minutes). Pour plus d'informations sur les différences de prise en charge entre MQTT, AMQP et HTTPS, consultez [Conseils sur les communications cloud-à-appareil](iot-hub-devguide-c2d-guidance.md) et [Choisir un protocole de communication](iot-hub-devguide-protocols.md).
>

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

Dans cet article, vous créez un service back-end pour envoyer des messages cloud-à-appareil via le hub IoT que vous avez créé dans [Envoyer des données de télémétrie d'un appareil vers un hub IoT](quickstart-send-telemetry-node.md). Pour envoyer des messages cloud-à-appareil, votre service a besoin de l'autorisation de **connexion de service**. Par défaut, chaque IoT Hub est créé avec une stratégie d’accès partagé nommée **service** qui accorde cette autorisation.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="send-a-cloud-to-device-message"></a>Envoi d’un message cloud vers appareil

Dans cette section, vous allez créer une application de console Node.js qui envoie des messages cloud-à-appareil à l’application de l’appareil simulé. Vous avez besoin de l’ID de l’appareil que vous avez ajouté dans le guide de démarrage rapide [Envoyer des données de télémétrie d’un appareil à un hub IoT](quickstart-send-telemetry-node.md). Vous avez également besoin de la chaîne de connexion du hub IoT que vous avez précédemment copiée dans [Obtenir la chaîne de connexion du hub IoT](#get-the-iot-hub-connection-string).

1. Créez un dossier vide appelé **sendcloudtodevicemessage**. Dans le dossier **sendcloudtodevicemessage** , créez un fichier package.json à l’aide de la commande ci-dessous, à l’invite de commandes. Acceptez toutes les valeurs par défaut :

    ```shell
    npm init
    ```

2. À l’invite de commandes, dans le dossier **sendcloudtodevicemessage**, exécutez la commande suivante pour installer le package **azure-iothub** :

    ```shell
    npm install azure-iothub --save
    ```

3. À l’aide d’un éditeur de texte, créez un fichier **SendCloudToDeviceMessage.js** dans le dossier **sendcloudtodevicemessage**.

4. Ajoutez les instructions `require` ci-dessous au début du fichier **SendCloudToDeviceMessage.js** :

    ```javascript
    'use strict';

    var Client = require('azure-iothub').Client;
    var Message = require('azure-iot-common').Message;
    ```

5. Ajoutez le code suivant au fichier **SendCloudToDeviceMessage.js** . Remplacez les valeurs d’espace réservé "{iot hub connection string}" et "{device id}" par la chaîne de connexion de l’IoT Hub et l’identité d'appareil que vous avez notées précédemment :

    ```javascript
    var connectionString = '{iot hub connection string}';
    var targetDevice = '{device id}';

    var serviceClient = Client.fromConnectionString(connectionString);
    ```

6. Ajoutez la fonction suivante pour imprimer les résultats de l’opération sur la console :

    ```javascript
    function printResultFor(op) {
      return function printResult(err, res) {
        if (err) console.log(op + ' error: ' + err.toString());
        if (res) console.log(op + ' status: ' + res.constructor.name);
      };
    }
    ```

7. Ajoutez la fonction suivante pour imprimer les messages d’accusé de réception sur la console :

    ```javascript
    function receiveFeedback(err, receiver){
      receiver.on('message', function (msg) {
        console.log('Feedback message:')
        console.log(msg.getData().toString('utf-8'));
      });
    }
    ```

8. Ajoutez le code suivant pour envoyer un message à votre appareil et gérer le message de commentaires lorsque l’appareil accuse réception d’un message cloud-à-appareil :

    ```javascript
    serviceClient.open(function (err) {
      if (err) {
        console.error('Could not connect: ' + err.message);
      } else {
        console.log('Service client connected');
        serviceClient.getFeedbackReceiver(receiveFeedback);
        var message = new Message('Cloud to device message.');
        message.ack = 'full';
        message.messageId = "My Message ID";
        console.log('Sending message: ' + message.getData());
        serviceClient.send(targetDevice, message, printResultFor('send'));
      }
    });
    ```

9. Enregistrez et fermez le fichier **SendCloudToDeviceMessage.js** .

## <a name="run-the-applications"></a>Exécution des applications

Vous êtes maintenant prêt à exécuter les applications.

1. À l'invite de commandes du dossier **simulated-device**, exécutez la commande suivante pour envoyer les données de télémétrie à IoT Hub et écouter les messages cloud-à-appareil :

    ```shell
    node SimulatedDevice.js
    ```

    ![Exécution de l’application de périphérique simulé](./media/iot-hub-node-node-c2d/receivec2d.png)

2. À l’invite de commandes dans le dossier **sendcloudtodevicemessage**, exécutez la commande suivante pour envoyer un message cloud-à-appareil, puis attendez de recevoir l’accusé de réception :

    ```shell
    node SendCloudToDeviceMessage.js
    ```

    ![Exécutez l’application pour envoyer la commande cloud-à-appareil](./media/iot-hub-node-node-c2d/sendc2d.png)

   > [!NOTE]
   > Par souci de simplicité, ce didacticiel n’implémente aucune stratégie de nouvelle tentative. Dans le code de production, vous devez implémenter des stratégies de nouvelle tentative (par exemple, une interruption exponentielle), comme indiqué dans l’article [Gestion des erreurs temporaires](/azure/architecture/best-practices/transient-faults).
   >

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à envoyer et recevoir des messages cloud-à-appareil.

Pour voir des exemples de solutions de bout en bout qui utilisent IoT Hub, consultez [Accélérateur de solution de supervision à distance Azure IoT](https://azure.microsoft.com/documentation/suites/iot-suite/).

Pour en savoir plus sur le développement de solutions avec IoT Hub, consultez le [Guide du développeur d’IoT Hub](iot-hub-devguide.md).
