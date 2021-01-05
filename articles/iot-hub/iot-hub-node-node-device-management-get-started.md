---
title: Bien démarrer avec la gestion des appareils Azure IoT Hub (Node) | Microsoft Docs
description: Découvrez comment utiliser la gestion des appareils IoT Hub pour lancer un redémarrage d’appareil à distance. Vous utilisez le SDK Azure IoT pour Node.js afin d’implémenter une application d’appareil simulé qui inclut une méthode directe et une application de service qui appelle la méthode directe.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/20/2019
ms.custom: mqtt, devx-track-js
ms.openlocfilehash: cfc0fa45c08f917b2e0b4a0b055e801173a4ba39
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91252015"
---
# <a name="get-started-with-device-management-nodejs"></a>Prise en main de la gestion d’appareils (Node.js)

[!INCLUDE [iot-hub-selector-dm-getstarted](../../includes/iot-hub-selector-dm-getstarted.md)]

Ce didacticiel vous explique les procédures suivantes :

* Utiliser le [portail Azure](https://portal.azure.com) pour créer un IoT Hub et une identité d’appareil dans celui-ci.

* Créer une application d’appareil simulé disposant d’une méthode directe permettant le redémarrage de cet appareil. Les méthodes directes sont appelées à partir du cloud.

* Créer une application console Node.js qui appelle une méthode directe de redémarrage sur l’application d’appareil simulé via votre IoT Hub.

À la fin de ce didacticiel, vous disposerez de deux applications console Node.js :

* **dmpatterns_getstarted_device.js**, qui se connecte à votre IoT Hub avec l’identité d’appareil créée précédemment, reçoit une méthode directe de redémarrage, simule un redémarrage physique et indique l’heure du dernier redémarrage.

* **dmpatterns_getstarted_service.js**, qui appelle une méthode directe sur l’application de l’appareil simulé, affiche la réponse et affiche les propriétés signalées pour la mise à jour.

## <a name="prerequisites"></a>Prérequis

* Node.j version 10.0.x ou ultérieure. L’article [Préparer votre environnement de développement](https://github.com/Azure/azure-iot-sdk-node/tree/master/doc/node-devbox-setup.md) décrit l’installation de Node.js pour ce didacticiel sur Windows ou sur Linux.

* Un compte Azure actif. (Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes seulement.)

* Vérifiez que le port 8883 est ouvert dans votre pare-feu. L’exemple d’appareil décrit dans cet article utilise le protocole MQTT, qui communique via le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d’informations sur les différentes façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Inscrire un nouvel appareil dans le hub IoT

[!INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## <a name="create-a-simulated-device-app"></a>Création d’une application de périphérique simulé

Dans cette section, vous allez :

* Créer une application console Node.js qui répond à une méthode directe appelée par le cloud

* Déclencher un redémarrage d’appareil simulé

* Utiliser les propriétés signalées pour activer les requêtes sur le jumeau d’appareil afin d’identifier les appareils et l’heure de leur dernier redémarrage

1. Créez un dossier vide nommé **manageddevice**.  Dans le dossier **simulateddevice**, créez un fichier package.json en utilisant la commande suivante à l’invite de commandes.  Acceptez toutes les valeurs par défaut :

    ```cmd/sh
    npm init
    ```

2. À l’invite de commandes, dans le dossier **manageddevice**, exécutez la commande suivante pour installer le package du kit Device SDK **azure-iot-device** et le package **azure-iot-device-mqtt** :

    ```cmd/sh
    npm install azure-iot-device azure-iot-device-mqtt --save
    ```

3. À l’aide d’un éditeur de texte, créez un fichier **dmpatterns_getstarted_device.js** dans le dossier **manageddevice**.

4. Ajoutez les instructions 'require' suivantes au début du fichier **dmpatterns_getstarted_device.js** :

    ```javascript
    'use strict';

    var Client = require('azure-iot-device').Client;
    var Protocol = require('azure-iot-device-mqtt').Mqtt;
    ```

5. Ajoutez une variable **connectionString** et utilisez-la pour créer une instance de **Client**.  Remplacez la valeur d’espace réservé `{yourdeviceconnectionstring}` par la chaîne de connexion de l’appareil que vous avez précédemment copiée dans [Inscrire un nouvel appareil dans le hub IoT](#register-a-new-device-in-the-iot-hub).  

    ```javascript
    var connectionString = '{yourdeviceconnectionstring}';
    var client = Client.fromConnectionString(connectionString, Protocol);
    ```

6. Ajoutez la fonction suivante pour implémenter la méthode directe sur l’appareil

    ```javascript
    var onReboot = function(request, response) {

        // Respond the cloud app for the direct method
        response.send(200, 'Reboot started', function(err) {
            if (err) {
                console.error('An error occurred when sending a method response:\n' + err.toString());
            } else {
                console.log('Response to method \'' + request.methodName + '\' sent successfully.');
            }
        });

        // Report the reboot before the physical restart
        var date = new Date();
        var patch = {
            iothubDM : {
                reboot : {
                    lastReboot : date.toISOString(),
                }
            }
        };

        // Get device Twin
        client.getTwin(function(err, twin) {
            if (err) {
                console.error('could not get twin');
            } else {
                console.log('twin acquired');
                twin.properties.reported.update(patch, function(err) {
                    if (err) throw err;
                    console.log('Device reboot twin state reported')
                });  
            }
        });

        // Add your device's reboot API for physical restart.
        console.log('Rebooting!');
    };
    ```

7. Ouvrez la connexion à votre hub IoT et démarrez l’écouteur de la méthode directe :

    ```javascript
    client.open(function(err) {
        if (err) {
            console.error('Could not open IotHub client');
        }  else {
            console.log('Client opened.  Waiting for reboot method.');
            client.onDeviceMethod('reboot', onReboot);
        }
    });
    ```

8. Enregistrez et fermez le fichier **dmpatterns_getstarted_device.js**.

> [!NOTE]
> Pour simplifier les choses, ce didacticiel n’implémente aucune stratégie de nouvelle tentative. Dans le code de production, vous devez implémenter des stratégies de nouvelle tentative (par exemple, une interruption exponentielle), comme indiqué dans l’article [Gestion des erreurs temporaires](/azure/architecture/best-practices/transient-faults).

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

[!INCLUDE [iot-hub-howto-device-management-shared-access-policy-text](../../includes/iot-hub-howto-device-management-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="trigger-a-remote-reboot-on-the-device-using-a-direct-method"></a>Déclencher un redémarrage à distance sur l’appareil à l’aide d’une méthode directe

Dans cette section, vous créez une application console Node.js qui lance un redémarrage à distance sur un appareil avec une méthode directe. L’application utilise des requêtes du jumeau d’appareil pour déterminer l’heure du dernier redémarrage de cet appareil.

1. Créez un dossier vide nommé **triggerrebootondevice**. Dans le dossier **triggerrebootondevice**, créez un fichier package.json en utilisant la commande suivante à l’invite de commandes. Acceptez toutes les valeurs par défaut :

    ```cmd/sh
    npm init
    ```

2. À l’invite de commandes, dans le dossier **triggerrebootondevice**, exécutez la commande suivante pour installer le package du kit Device SDK **azure-iothub** et le package **azure-iot-device-mqtt** :

    ```cmd/sh
    npm install azure-iothub --save
    ```

3. À l’aide d’un éditeur de texte, créez un fichier **dmpatterns_getstarted_service.js** dans le dossier **triggerrebootondevice**.

4. Ajoutez les instructions ’require’ suivantes au début du fichier **dmpatterns_getstarted_service.js** :

    ```javascript
    'use strict';

    var Registry = require('azure-iothub').Registry;
    var Client = require('azure-iothub').Client;
    ```

5. Ajoutez les déclarations de variable suivantes et remplacez la valeur d’espace réservé `{iothubconnectionstring}` par la chaîne de connexion du hub IoT que vous avez copiée précédemment dans [Obtenir la chaîne de connexion du hub IoT](#get-the-iot-hub-connection-string) :

    ```javascript
    var connectionString = '{iothubconnectionstring}';
    var registry = Registry.fromConnectionString(connectionString);
    var client = Client.fromConnectionString(connectionString);
    var deviceToReboot = 'myDeviceId';
    ```

6. Ajoutez la fonction suivante pour appeler la méthode device afin de redémarrer l’appareil cible :

    ```javascript
    var startRebootDevice = function(twin) {

        var methodName = "reboot";

        var methodParams = {
            methodName: methodName,
            payload: null,
            timeoutInSeconds: 30
        };

        client.invokeDeviceMethod(deviceToReboot, methodParams, function(err, result) {
            if (err) {
                console.error("Direct method error: "+err.message);
            } else {
                console.log("Successfully invoked the device to reboot.");  
            }
        });
    };
    ```

7. Ajoutez la fonction suivante pour interroger l’appareil et obtenir l’heure du dernier redémarrage :

    ```javascript
    var queryTwinLastReboot = function() {

        registry.getTwin(deviceToReboot, function(err, twin){

            if (twin.properties.reported.iothubDM != null)
            {
                if (err) {
                    console.error('Could not query twins: ' + err.constructor.name + ': ' + err.message);
                } else {
                    var lastRebootTime = twin.properties.reported.iothubDM.reboot.lastReboot;
                    console.log('Last reboot time: ' + JSON.stringify(lastRebootTime, null, 2));
                }
            } else 
                console.log('Waiting for device to report last reboot time.');
        });
    };
    ```

8. Ajoutez le code suivant pour appeler les fonctions qui déclenchent la méthode directe de redémarrage et la requête sur le dernier redémarrage :

    ```javascript
    startRebootDevice();
    setInterval(queryTwinLastReboot, 2000);
    ```

9. Enregistrez et fermez le fichier **dmpatterns_getstarted_service.js**.

## <a name="run-the-apps"></a>Exécuter les applications

Vous êtes maintenant prêt à exécuter les applications.

1. À l’invite de commandes, dans le dossier **manageddevice**, exécutez la commande suivante pour commencer à écouter la méthode directe de redémarrage.

    ```cmd/sh
    node dmpatterns_getstarted_device.js
    ```

2. À l’invite de commandes, dans le dossier **triggerrebootondevice**, exécutez la commande suivante pour déclencher le redémarrage à distance et interroger le jumeau d’appareil pour déterminer l’heure du dernier redémarrage.

    ```cmd/sh
    node dmpatterns_getstarted_service.js
    ```

3. La réponse de l’appareil à la méthode directe de redémarrage et l’état de redémarrage s’affichent dans la console.

   Voici la réponse de l’appareil à la méthode directe de redémarrage envoyée par le service :

   ![sortie de l’application manageddevice](./media/iot-hub-node-node-device-management-get-started/device.png)

   Le code suivant montre le service déclenchant le redémarrage et interrogeant le jumeau d’appareil sur le dernier redémarrage :

   ![sortie de l’application triggerrebootondevice](./media/iot-hub-node-node-device-management-get-started/service.png)

[!INCLUDE [iot-hub-dm-followup](../../includes/iot-hub-dm-followup.md)]
