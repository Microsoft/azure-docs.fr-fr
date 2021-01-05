---
title: Charger les fichiers d’un appareil sur Azure IoT Hub avec Node | Microsoft Docs
description: Découvrez comment charger les fichiers d’un appareil sur le cloud avec le kit Azure IoT device SDK pour Node.js. Les fichiers téléchargés sont stockés dans un conteneur d’objets blob de stockage Azure.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 06/28/2017
ms.custom: mqtt, devx-track-js
ms.openlocfilehash: db4f78e14696c421adaedd16b0b3f8d598f12846
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91251896"
---
# <a name="upload-files-from-your-device-to-the-cloud-with-iot-hub-nodejs"></a>Charger des fichiers sur le cloud à partir d’un appareil avec IoT Hub (Node.js)

[!INCLUDE [iot-hub-file-upload-language-selector](../../includes/iot-hub-file-upload-language-selector.md)]

Ce tutoriel s’appuie sur le code du tutoriel [Envoyer des messages cloud-à-appareil avec IoT Hub](iot-hub-node-node-c2d.md) pour montrer comment utiliser les [fonctions de chargement de fichiers d’IoT Hub](iot-hub-devguide-file-upload.md) afin de charger un fichier sur le [Stockage Blob Azure](../storage/index.yml). Ce didacticiel explique les procédures suivantes :

* Fournissez en toute sécurité à un appareil un URI d’objet blob Azure pour le chargement d’un fichier.

* Utilisez les notifications de chargement de fichier IoT Hub pour déclencher le traitement du fichier dans votre serveur principal d’application.

Le guide de démarrage rapide [Envoyer des données de télémétrie d’un appareil à un hub IoT](quickstart-send-telemetry-node.md) présente les fonctionnalités de messages appareil-à-cloud de base d’IoT Hub. Toutefois, dans certains scénarios, vous ne pouvez pas facilement mapper les données que vos appareils envoient dans des messages appareil-à-cloud relativement petits et acceptés par IoT Hub. Par exemple :

* Fichiers volumineux qui contiennent des images
* Vidéos
* Données de vibration échantillonnées à une fréquence élevée
* Un certain type de données prétraitées.

Ces fichiers sont généralement traités par lot dans le cloud à l’aide d’outils tels que [Azure Data Factory](../data-factory/introduction.md) ou de la pile [Hadoop](../hdinsight/index.yml). Lorsque vous avez besoin de charger des fichiers à partir d’un appareil, vous pouvez quand même exploiter la sécurité et la fiabilité d’IoT Hub.

À la fin de ce didacticiel, vous exécuterez deux applications de console Node.js :

* **SimulatedDevice.js**, qui charge un fichier de stockage à l’aide d’un URI SAP fourni par votre hub IoT.

* **ReadFileUploadNotification**, qui reçoit les notifications de chargement de fichier à partir de votre hub IoT.

> [!NOTE]
> IoT Hub prend en charge de nombreuses plateformes d’appareils et de nombreux langages (notamment C, .NET, Javascript, Python et Java) par le biais des kits Azure IoT device SDK. Pour obtenir des instructions pas à pas expliquant comment connecter votre appareil à Azure IoT Hub, voir le Centre de développement Azure IoT.

[!INCLUDE [iot-hub-include-x509-ca-signed-file-upload-support-note](../../includes/iot-hub-include-x509-ca-signed-file-upload-support-note.md)]

## <a name="prerequisites"></a>Prérequis

* Node.j version 10.0.x ou ultérieure. L’article [Préparer votre environnement de développement](https://github.com/Azure/azure-iot-sdk-node/tree/master/doc/node-devbox-setup.md) décrit l’installation de Node.js pour ce didacticiel sur Windows ou sur Linux.

* Un compte Azure actif. (Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes seulement.)

* Vérifiez que le port 8883 est ouvert dans votre pare-feu. L’exemple d’appareil décrit dans cet article utilise le protocole MQTT, qui communique via le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d’informations sur les différentes façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [iot-hub-associate-storage](../../includes/iot-hub-associate-storage.md)]

## <a name="upload-a-file-from-a-device-app"></a>Charger un fichier à partir d’une application d’appareil

Dans cette section, créez l’application d’appareil pour charger un fichier sur IoT Hub.

1. Créez un dossier vide appelé ```simulateddevice```.  Dans le dossier ```simulateddevice```, créez un fichier package.json en saisissant la commande suivante dans votre invite de commandes.  Acceptez toutes les valeurs par défaut :

    ```cmd/sh
    npm init
    ```

2. Dans l’invite de commandes du dossier ```simulateddevice```, exécutez la commande suivante pour installer le package SDK d’appareil **azure-iot-device** et le package **azure-iot-device-mqtt** :

    ```cmd/sh
    npm install azure-iot-device azure-iot-device-mqtt --save
    ```

3. À l’aide d’un éditeur de texte, créez un fichier **SimulatedDevice.js** dans le dossier ```simulateddevice```.

4. Ajoutez les instructions ```require``` ci-dessous au début du fichier **SimulatedDevice.js** :

    ```javascript
    'use strict';

    var fs = require('fs');
    var mqtt = require('azure-iot-device-mqtt').Mqtt;
    var clientFromConnectionString = require('azure-iot-device-mqtt').clientFromConnectionString;
    ```

5. Ajoutez une variable `deviceconnectionstring` et utilisez-la pour créer une instance **Client**.  Remplacez `{deviceconnectionstring}` par le nom de l’appareil que vous avez créé dans la section *Créer un hub IoT* :

    ```javascript
    var connectionString = '{deviceconnectionstring}';
    var filename = 'myimage.png';
    ```

    > [!NOTE]
    > Par souci de simplicité, la chaîne de connexion est dans le code ; cette pratique n’étant pas recommandée, vous pouvez envisager des méthodes de stockage plus sécurisées pour ce secret suivant votre architecture et votre cas d’usage.

6. Ajoutez le code suivant pour connecter le client :

    ```javascript
    var client = clientFromConnectionString(connectionString);
    console.log('Client connected');
    ```

7. Créez un rappel et utilisez la fonction **uploadToBlob** pour charger le fichier.

    ```javascript
    fs.stat(filename, function (err, stats) {
        const rr = fs.createReadStream(filename);

        client.uploadToBlob(filename, rr, stats.size, function (err) {
            if (err) {
                console.error('Error uploading file: ' + err.toString());
            } else {
                console.log('File uploaded');
            }
        });
    });
    ```

8. Enregistrez et fermez le fichier **SimulatedDevice.js** .

9. Copiez un fichier image dans le dossier `simulateddevice` et renommez-le `myimage.png`.

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

Dans cet article, vous créez un service back-end pour recevoir les messages de notification de chargement de fichiers depuis l’IoT Hub que vous avez créé dans [Send telemetry from a device to an IoT hub (Envoyer des données de télémétrie d’un appareil à un IoT Hub)](quickstart-send-telemetry-node.md). Pour recevoir les messages de notification de chargement de fichiers, votre service a besoin de l'autorisation de **connexion de service**. Par défaut, chaque IoT Hub est créé avec une stratégie d’accès partagé nommée **service** qui accorde cette autorisation.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="receive-a-file-upload-notification"></a>Recevoir une notification de téléchargement de fichier

Dans cette section, vous allez créer une application console Node.js qui reçoit des messages de notification de chargement de fichiers envoyés par IoT Hub.

Vous pouvez utiliser la chaîne de connexion **iothubowner** de votre hub IoT pour suivre cette section. Vous trouverez la chaîne de connexion dans le panneau **Stratégie d’accès partagé** du [portail Azure](https://portal.azure.com/).

1. Créez un dossier vide appelé ```fileuploadnotification```.  Dans le dossier ```fileuploadnotification```, créez un fichier package.json en saisissant la commande suivante dans votre invite de commandes.  Acceptez toutes les valeurs par défaut :

    ```cmd/sh
    npm init
    ```

2. Depuis votre invite de commandes, dans le dossier ```fileuploadnotification```, exécutez la commande suivante pour installer le package SDK **azure-iothub** :

    ```cmd/sh
    npm install azure-iothub --save
    ```

3. À l’aide d’un éditeur de texte, créez un fichier **FileUploadNotification.js** dans le dossier `fileuploadnotification`.

4. Ajoutez les instructions `require` ci-dessous au début du fichier **FileUploadNotification.js** :

    ```javascript
    'use strict';

    var Client = require('azure-iothub').Client;
    ```

5. Ajoutez une variable `iothubconnectionstring` et utilisez-la pour créer une instance **Client**.  Remplacez la valeur de l’espace réservé `{iothubconnectionstring}` par la chaîne de connexion de l’IoT Hub que vous avez précédemment copiée dans [Obtention de la chaîne de connexion de l’IoT Hub](#get-the-iot-hub-connection-string) :

    ```javascript
    var connectionString = '{iothubconnectionstring}';
    ```

    > [!NOTE]
    > Par souci de simplicité, la chaîne de connexion est dans le code ; cette pratique n’étant pas recommandée, vous pouvez envisager des méthodes de stockage plus sécurisées pour ce secret suivant votre architecture et votre cas d’usage.

6. Ajoutez le code suivant pour connecter le client :

    ```javascript
    var serviceClient = Client.fromConnectionString(connectionString);
    ```

7. Ouvrez le client et utilisez la fonction **getFileNotificationReceiver** pour recevoir les mises à jour d’état.

    ```javascript
    serviceClient.open(function (err) {
      if (err) {
        console.error('Could not connect: ' + err.message);
      } else {
        console.log('Service client connected');
        serviceClient.getFileNotificationReceiver(function receiveFileUploadNotification(err, receiver){
          if (err) {
            console.error('error getting the file notification receiver: ' + err.toString());
          } else {
            receiver.on('message', function (msg) {
              console.log('File upload from device:')
              console.log(msg.getData().toString('utf-8'));
            });
          }
        });
      }
    });
    ```

8. Enregistrez et fermez le fichier **FileUploadNotification.js**.

## <a name="run-the-applications"></a>Exécution des applications

Vous êtes maintenant prêt à exécuter les applications.

Dans une invite de commandes, exécutez la commande suivante dans le dossier `fileuploadnotification` :

```cmd/sh
node FileUploadNotification.js
```

Dans une invite de commandes, exécutez la commande suivante dans le dossier `simulateddevice` :

```cmd/sh
node SimulatedDevice.js
```

La capture d’écran suivante montre la sortie de l’application **SimulatedDevice** :

![Sortie de l’application simulated-device](./media/iot-hub-node-node-file-upload/simulated-device.png)

La capture d’écran suivante montre la sortie de l’application **FileUploadNotification** :

![Sortie de l’application read-file-upload-notification](./media/iot-hub-node-node-file-upload/read-file-upload-notification.png)

Vous pouvez utiliser le portail pour afficher le fichier chargé dans le conteneur de stockage que vous avez configuré :

![Fichier chargé](./media/iot-hub-node-node-file-upload/uploaded-file.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à utiliser les fonctionnalités de téléchargement de fichier d’IoT Hub pour simplifier les chargements de fichiers à partir d’appareils. Vous pouvez continuer à explorer les scénarios et fonctionnalités d’IoT Hub avec les articles suivants :

* [Créer un IoT Hub par programmation](iot-hub-rm-template-powershell.md)

* [Présentation du SDK C](iot-hub-device-sdk-c-intro.md)

* [Kits de développement logiciel (SDK) Azure IoT](iot-hub-devguide-sdks.md)
