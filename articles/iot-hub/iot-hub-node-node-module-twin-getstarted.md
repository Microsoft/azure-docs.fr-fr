---
title: Bien démarrer avec l’identité de module et le jumeau de module Azure IoT Hub (Node.js)
description: Découvrez comment créer une identité de module et comment mettre à jour un jumeau de module à l’aide des SDK IoT pour Node.js.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 04/26/2018
ms.custom: amqp, devx-track-js
ms.openlocfilehash: 9de3f45a9d62a9d131583c133440ac8a311a468d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993226"
---
# <a name="get-started-with-iot-hub-module-identity-and-module-twin-nodejs"></a>Bien démarrer avec l’identité de module et le jumeau de module IoT Hub (Node.js)

[!INCLUDE [iot-hub-selector-module-twin-getstarted](../../includes/iot-hub-selector-module-twin-getstarted.md)]

> [!NOTE]
> [Les identités de module et les jumeaux de module](iot-hub-devguide-module-twins.md) sont similaires aux identités d’appareil et aux jumeaux d’appareil Azure IoT Hub, mais offrent un plus grand niveau de détail. Contrairement à l’identité d’appareil et au jumeau d’appareil Azure IoT Hub qui permettent à l’application back-end de configurer un appareil et d’obtenir une visibilité sur les conditions de l’appareil, une identité de module et un jumeau de module fournissent ces capacités pour les composants individuels d’un appareil. Sur les appareils compatibles qui intègrent plusieurs composants, par exemple des appareils basés sur un système d’exploitation ou des appareils avec un microprogramme, ils permettent d’isoler la configuration et les conditions de chacun de ces composants.

À la fin de ce didacticiel, vous disposerez de deux applications Node.js :

* **CreateIdentities**, qui crée une identité d’appareil, une identité de module et une clé de sécurité associée pour connecter votre appareil et vos clients de module.

* **UpdateModuleTwinReportedProperties**, qui envoie à votre IoT Hub les propriétés à jour de votre jumeau de module.

> [!NOTE]
> Pour plus d’informations sur les SDK Azure IoT que vous pouvez utiliser pour générer les deux applications qui s’exécutent sur les appareils et sur le serveur de solution principal, consultez l’article [SDK Azure IoT](iot-hub-devguide-sdks.md).

## <a name="prerequisites"></a>Prérequis

* Node.j version 10.0.x ou ultérieure. L’article [Préparer votre environnement de développement](https://github.com/Azure/azure-iot-sdk-node/tree/master/doc/node-devbox-setup.md) décrit l’installation de Node.js pour ce didacticiel sur Windows ou sur Linux.

* Un compte Azure actif. (Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes seulement.)

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

[!INCLUDE [iot-hub-howto-module-twin-shared-access-policy-text](../../includes/iot-hub-howto-module-twin-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-registryrw-connection-string](../../includes/iot-hub-include-find-registryrw-connection-string.md)]

## <a name="create-a-device-identity-and-a-module-identity-in-iot-hub"></a>Créer une identité d’appareil et une identité de module dans IoT Hub

Dans cette section, vous allez créer une application Node.js qui crée une identité d’appareil et une identité de module dans le registre d’identités de votre hub IoT. Un appareil ou un module ne peut pas se connecter à IoT Hub, à moins de posséder une entrée dans le registre des identités. Pour plus d’informations, consultez la section « Registre des identités » du [Guide du développeur IoT Hub](iot-hub-devguide-identity-registry.md).0 En exécutant cette application console, une clé et un ID uniques sont générés pour chaque appareil et module. Votre appareil et le module utilisent ces valeurs pour s’identifier lorsqu’ils envoient des messages d’appareil-à-cloud à IoT Hub. Les ID sont sensibles à la casse.

1. Créez un répertoire pour stocker votre code.

2. À l’intérieur de ce répertoire, exécutez d’abord **npm init -y** pour créer un fichier package.json vide comportant les valeurs par défaut. Il s’agit du fichier projet pour votre code.

3. Exécutez **npm install -S azure-iothub\@modules-preview** pour installer le SDK du service à l’intérieur du sous-répertoire **node_modules**.

    > [!NOTE]
    > Dans le nom de sous-répertoire node_modules, le mot « module » désigne « une bibliothèque de nœuds ». En l’occurrence, le terme n’a rien à voir avec les modules IoT Hub.

4. Créez le fichier .js suivant dans votre répertoire. Appelez-le **add.js**. Copiez et collez la chaîne de connexion de hub et le nom du hub.

    ```javascript
    var Registry = require('azure-iothub').Registry;
    var uuid = require('uuid');
    // Copy/paste your connection string and hub name here
    var serviceConnectionString = '<hub connection string from portal>';
    var hubName = '<hub name>.azure-devices.net';
    // Create an instance of the IoTHub registry
    var registry = Registry.fromConnectionString(serviceConnectionString);
    // Insert your device ID and moduleId here.
    var deviceId = 'myFirstDevice';
    var moduleId = 'myFirstModule';
    // Create your device as a SAS authentication device
    var primaryKey = new Buffer(uuid.v4()).toString('base64');
    var secondaryKey = new Buffer(uuid.v4()).toString('base64');
    var deviceDescription = {
      deviceId: deviceId,
      status: 'enabled',
      authentication: {
        type: 'sas',
        symmetricKey: {
          primaryKey: primaryKey,
          secondaryKey: secondaryKey
        }
      }
    };

    // First, create a device identity
    registry.create(deviceDescription, function(err) {
      if (err) {
        console.log('Error creating device identity: ' + err);
        process.exit(1);
      }
      console.log('device connection string = "HostName=' + hubName + ';DeviceId=' + deviceId + ';SharedAccessKey=' + primaryKey + '"');

      // Then add a module to that device
      registry.addModule({ deviceId: deviceId, moduleId: moduleId }, function(err) {
        if (err) {
          console.log('Error creating module identity: ' + err);
          process.exit(1);
        }

        // Finally, retrieve the module details from the hub so we can construct the connection string
        registry.getModule(deviceId, moduleId, function(err, foundModule) {
          if (err) {
            console.log('Error getting module back from hub: ' + err);
            process.exit(1);
          }
          console.log('module connection string = "HostName=' + hubName + ';DeviceId=' + foundModule.deviceId + ';ModuleId='+foundModule.moduleId+';SharedAccessKey=' + foundModule.authentication.symmetricKey.primaryKey + '"');
          process.exit(0);
        });
      });
    });

    ```

Cette application crée une identité d’appareil avec l’ID **myFirstDevice** et une identité de module avec l’ID **myFirstModule** sous l’appareil **myFirstDevice**. (si cet ID de module existe déjà dans le registre d’identité, le code récupère simplement les informations existantes du module.) L’application affiche ensuite la clé primaire pour cette identité. Vous utilisez cette clé dans l’application de module simulé pour vous connecter à votre IoT Hub.

Exécutez-la à l’aide de la commande node add.js. Vous obtiendrez alors une chaîne de connexion pour votre identité d’appareil et une autre pour votre identité de module.

> [!NOTE]
> Le registre des identités IoT Hub stocke uniquement les identités des appareils et des modules pour permettre un accès sécurisé à IoT Hub. Le registre des identités stocke les ID et les clés d’appareil à utiliser en tant qu’informations d’identification de sécurité. Il stocke également un indicateur activé/désactivé pour chaque appareil pouvant être utilisé pour désactiver l’accès de cet appareil. Si votre application a besoin de stocker d’autres métadonnées spécifiques aux appareils, elle doit utiliser un magasin spécifique aux applications. Il n’y a aucun indicateur d’activation/désactivation pour les identités de module. Pour plus d’informations, reportez-vous au [Guide du développeur IoT Hub](iot-hub-devguide-identity-registry.md).

## <a name="update-the-module-twin-using-nodejs-device-sdk"></a>Mettre à jour le jumeau de module à l’aide du SDK d’appareil Node.js

Dans cette section, vous allez créer sur votre appareil simulé une application Node.js qui met à jour les propriétés rapportées du jumeau de module.

1. **Obtenir la chaîne de connexion de votre module** : connectez-vous au [portail Azure](https://portal.azure.com/). Accédez à votre IoT Hub, puis cliquez sur Appareils IoT. Recherchez et ouvrez myFirstDevice pour vérifier que myFirstModule a bien été créé. Copiez la chaîne de connexion du module. Vous en aurez besoin à l’étape suivante.

   ![Détails du module du Portail Azure](./media/iot-hub-node-node-module-twin-getstarted/module-detail.png)

2. Comme à l’étape précédente, créez un répertoire pour votre code d’appareil et utilisez NPM pour l’initialiser et installer le Kit de développement logiciel (SDK) de l’appareil (**npm install -S azure-iot-device-amqp\@modules-preview**).

   > [!NOTE]
   > La commande npm install peut vous sembler lente. Soyez patient ; elle récupère beaucoup de code du référentiel de packages.

   > [!NOTE]
   > Si vous voyez l’erreur « npm ERR! registry error parsing json » (Erreur npm : erreur de registre liée à une analyse json), vous pouvez l’ignorer sans risque. Si vous voyez l’erreur « npm ERR! registry error parsing json » (Erreur npm : erreur de registre liée à une analyse json), vous pouvez l’ignorer sans risque.

3. Créez un fichier appelé twin.js. Copiez et collez la chaîne d’identité de module.

    ```javascript
    var Client = require('azure-iot-device').Client;
    var Protocol = require('azure-iot-device-amqp').Amqp;
    // Copy/paste your module connection string here.
    var connectionString = '<insert module connection string here>';
    // Create a client using the Amqp protocol.
    var client = Client.fromConnectionString(connectionString, Protocol);
    client.on('error', function (err) {
      console.error(err.message);
    });
    // connect to the hub
    client.open(function(err) {
      if (err) {
        console.error('error connecting to hub: ' + err);
        process.exit(1);
      }
      console.log('client opened');
    // Create device Twin
      client.getTwin(function(err, twin) {
        if (err) {
          console.error('error getting twin: ' + err);
          process.exit(1);
        }
        // Output the current properties
        console.log('twin contents:');
        console.log(twin.properties);
        // Add a handler for desired property changes
        twin.on('properties.desired', function(delta) {
            console.log('new desired properties received:');
            console.log(JSON.stringify(delta));
        });
        // create a patch to send to the hub
        var patch = {
          updateTime: new Date().toString(),
          firmwareVersion:'1.2.1',
          weather:{
            temperature: 72,
            humidity: 17
          }
        };
        // send the patch
        twin.properties.reported.update(patch, function(err) {
          if (err) throw err;
          console.log('twin state reported');
        });
      });
    });
    ```

4. Maintenant, exécutez ceci à l’aide de la commande **node twin.js**.

   ```cmd/sh
   F:\temp\module_twin>node twin.js
   ```

   Vous verrez alors :

   ```console
   client opened
   twin contents:
   { reported: { update: [Function: update], '$version': 1 },
     desired: { '$version': 1 } }
   new desired properties received:
   {"$version":1}
   twin state reported
   ```

## <a name="next-steps"></a>Étapes suivantes

Pour continuer la prise en main de IoT Hub et explorer les autres scénarios IoT, consultez les articles suivants :

* [Prise en main de la gestion d’appareils](iot-hub-node-node-device-management-get-started.md)

* [Bien démarrer avec IoT Edge](../iot-edge/quickstart-linux.md)