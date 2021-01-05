---
title: Prise en main des représentations physiques Azure IoT Hub (Node) | Microsoft Docs
description: Guide d’utilisation des jumeaux d’appareils Azure IoT Hub pour ajouter des balises, puis utiliser une requête IoT Hub. Vous utilisez les SDK Azure IoT pour Node.js afin d’implémenter l’application d’appareil simulé et une application de service qui ajoute les balises et exécute la requête IoT Hub.
author: fsautomata
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 08/26/2019
ms.author: elioda
ms.custom: mqtt, devx-track-js
ms.openlocfilehash: 65ced3812072bd2650fc36bbb7a7b0f3f75e0def
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91336783"
---
# <a name="get-started-with-device-twins-nodejs"></a>Prise en main des représentations d’appareils (Node.js)

[!INCLUDE [iot-hub-selector-twin-get-started](../../includes/iot-hub-selector-twin-get-started.md)]

À la fin de ce didacticiel, vous disposerez de deux applications console Node.js :

* **AddTagsAndQuery.js**, application Node.js qui ajoute des balises et interroge des représentations d’appareil.

* **TwinSimulatedDevice.js**, application Node.js qui simule un appareil se connectant à votre hub IoT avec l’identité d’appareil créée précédemment, et signale son état de connectivité.

> [!NOTE]
> L’article relatif aux [Kits de développement logiciel (SDK) Azure IoT](iot-hub-devguide-sdks.md) fournit des informations sur les Kits de développement logiciel (SDK) Azure que l’on peut utiliser pour générer des applications pour périphérique et des applications principales.
>

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous avez besoin des éléments suivants :

* Node.j version 10.0.x ou ultérieure.

* Un compte Azure actif. (Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes seulement.)

* Vérifiez que le port 8883 est ouvert dans votre pare-feu. L’exemple d’appareil décrit dans cet article utilise le protocole MQTT, qui communique via le port 8883. Ce port peut être bloqué dans certains environnements réseau professionnels et scolaires. Pour plus d’informations sur les différentes façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Inscrire un nouvel appareil dans le hub IoT

[!INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## <a name="get-the-iot-hub-connection-string"></a>Obtenir la chaîne de connexion du hub IoT

[!INCLUDE [iot-hub-howto-twin-shared-access-policy-text](../../includes/iot-hub-howto-twin-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-custom-connection-string](../../includes/iot-hub-include-find-custom-connection-string.md)]

## <a name="create-the-service-app"></a>Créer l’application de service

Dans cette section, vous créez une application console Node.js qui ajoute des métadonnées d’emplacement à la représentation d’appareil associée à **myDeviceId**. Elle interroge ensuite les jumeaux d’appareil stockés dans le hub IoT en sélectionnant les appareils situés aux États-Unis, puis ceux qui signalent une connexion mobile.

1. Créez un dossier vide nommé **addtagsandqueryapp**. Dans le dossier **addtagsandqueryapp**, créez un fichier package.json en utilisant la commande suivante à l’invite de commandes. Le paramètre `--yes` accepte toutes les valeurs par défaut.

    ```cmd/sh
    npm init --yes
    ```

2. À l’invite de commandes, dans le dossier **addtagsandqueryapp**, exécutez la commande suivante pour installer le package **azure-iothub** :

    ```cmd/sh
    npm install azure-iothub --save
    ```

3. À l’aide d’un éditeur de texte, créez un fichier **AddTagsAndQuery.js** dans le dossier **addtagsandqueryapp**.

4. Ajoutez le code suivant au fichier **AddTagsAndQuery.js**. Remplacez `{iot hub connection string}` par la chaîne de connexion IoT Hub que vous avez copiée dans [Obtenir la chaîne de connexion du hub IoT](#get-the-iot-hub-connection-string).

   ``` javascript
        'use strict';
        var iothub = require('azure-iothub');
        var connectionString = '{iot hub connection string}';
        var registry = iothub.Registry.fromConnectionString(connectionString);

        registry.getTwin('myDeviceId', function(err, twin){
            if (err) {
                console.error(err.constructor.name + ': ' + err.message);
            } else {
                var patch = {
                    tags: {
                        location: {
                            region: 'US',
                            plant: 'Redmond43'
                      }
                    }
                };

                twin.update(patch, function(err) {
                  if (err) {
                    console.error('Could not update twin: ' + err.constructor.name + ': ' + err.message);
                  } else {
                    console.log(twin.deviceId + ' twin updated successfully');
                    queryTwins();
                  }
                });
            }
        });
   ```

    L’objet **Registry** expose toutes les méthodes requises pour interagir avec des représentations d’appareil à partir du service. Le code précédent initialise l’objet **Registry**, récupère la représentation d’appareil de **myDeviceId**, puis met à jour ses balises avec les informations d’emplacement souhaitées.

    Une fois les balises mises à jour, il appelle la fonction **queryTwins**.

5. Ajoutez le code suivant à la fin de  **AddTagsAndQuery.js** pour implémenter la fonction **queryTwins**:

   ```javascript
        var queryTwins = function() {
            var query = registry.createQuery("SELECT * FROM devices WHERE tags.location.plant = 'Redmond43'", 100);
            query.nextAsTwin(function(err, results) {
                if (err) {
                    console.error('Failed to fetch the results: ' + err.message);
                } else {
                    console.log("Devices in Redmond43: " + results.map(function(twin) {return twin.deviceId}).join(','));
                }
            });

            query = registry.createQuery("SELECT * FROM devices WHERE tags.location.plant = 'Redmond43' AND properties.reported.connectivity.type = 'cellular'", 100);
            query.nextAsTwin(function(err, results) {
                if (err) {
                    console.error('Failed to fetch the results: ' + err.message);
                } else {
                    console.log("Devices in Redmond43 using cellular network: " + results.map(function(twin) {return twin.deviceId}).join(','));
                }
            });
        };
   ```

    Le code précédent exécute deux requêtes : la première sélectionne uniquement les représentations des appareils situés dans l’usine **Redmond43**et la seconde affine la requête pour sélectionner uniquement les appareils qui sont également connectés via un réseau cellulaire.

    Quand le code crée l’objet **query**, il spécifie le nombre maximal de documents retournés dans le deuxième paramètre. L’objet **query** contient une propriété booléenne **hasMoreResults** permettant d’appeler les méthodes **nextAsTwin** plusieurs fois afin de récupérer tous les résultats. Une méthode appelée **next** est disponible pour les résultats qui ne sont pas des jumeaux d’appareil, par exemple les résultats de requêtes d’agrégation.

6. Exécutez l’application avec :

    ```cmd/sh
        node AddTagsAndQuery.js
    ```

   Vous devriez voir un appareil dans les résultats de la requête demandant tous les appareils situés à **Redmond43**, et aucun pour la requête limitant les résultats aux appareils utilisant un réseau cellulaire.

   ![Affiche un appareil spécifique dans les résultats d’un requête](media/iot-hub-node-node-twin-getstarted/service1.png)

Dans la section suivante, vous allez créer une application d’appareil qui transmet les informations de connectivité et modifie le résultat de la requête de la section précédente.

## <a name="create-the-device-app"></a>Créer l’application pour appareil

Dans cette section, vous allez créer une application console Node.js qui se connecte à votre hub en tant que **myDeviceId**, puis met à jour les propriétés signalées de sa représentation d’appareil afin qu’elles contiennent les informations indiquant qu’elle est connectée par le biais d’un réseau cellulaire.

1. Créez un dossier vide nommé **reportconnectivity**. Dans le dossier **reportconnectivity**, créez un fichier package.json en utilisant la commande suivante à l’invite de commandes. Le paramètre `--yes` accepte toutes les valeurs par défaut.

    ```cmd/sh
    npm init --yes
    ```

2. Dans votre invite de commandes dans le dossier **reportconnectivity**, exécutez la commande suivante pour installer les packages **azure-iot-device** et **azure-iot-device-mqtt** :

    ```cmd/sh
    npm install azure-iot-device azure-iot-device-mqtt --save
    ```

3. À l’aide d’un éditeur de texte, créez un fichier **ReportConnectivity.js** dans le dossier **reportconnectivity**.

4. Ajoutez le code suivant au fichier **ReportConnectivity.js**. Remplacez `{device connection string}` par la chaîne de connexion de l’appareil que vous avez copiée quand vous avez créé l’identité d’appareil **myDeviceId** dans[Inscrire un nouvel appareil dans le hub IoT](#register-a-new-device-in-the-iot-hub).

    ```javascript
        'use strict';
        var Client = require('azure-iot-device').Client;
        var Protocol = require('azure-iot-device-mqtt').Mqtt;

        var connectionString = '{device connection string}';
        var client = Client.fromConnectionString(connectionString, Protocol);

        client.open(function(err) {
        if (err) {
            console.error('could not open IotHub client');
        }  else {
            console.log('client opened');

            client.getTwin(function(err, twin) {
            if (err) {
                console.error('could not get twin');
            } else {
                var patch = {
                    connectivity: {
                        type: 'cellular'
                    }
                };

                twin.properties.reported.update(patch, function(err) {
                    if (err) {
                        console.error('could not update twin');
                    } else {
                        console.log('twin state reported');
                        process.exit();
                    }
                });
            }
            });
        }
        });
    ```

    L’objet **Client** expose toutes les méthodes requises pour interagir avec des jumeaux d’appareil à partir de l’appareil. Le code précédent, après avoir initialisé l’objet **Client**, récupère la représentation d’appareil de **myDeviceId**, puis met à jour sa propriété signalée avec les informations de connectivité.

5. Exécuter l’application d’appareil

    ```cmd/sh
        node ReportConnectivity.js
    ```

    Vous devriez voir le message `twin state reported`.

6. À présent que l’appareil a signalé ses informations de connectivité, il doit apparaître dans les deux requêtes. Accédez au dossier **addtagsandqueryapp**, puis réexécutez les requêtes :

    ```cmd/sh
        node AddTagsAndQuery.js
    ```

    Cette fois, **myDeviceId** doit apparaître dans les résultats des deux requêtes.

    ![Afficher myDeviceId dans les résultats des deux requêtes](media/iot-hub-node-node-twin-getstarted/service2.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez configuré un nouveau IoT Hub dans le portail Azure, puis créé une identité d’appareil dans le registre des identités de l’IoT Hub. Vous avez ajouté des métadonnées d’appareil en tant que balises à partir d’une application principale et écrit une application pour appareil simulée pour signaler des informations de connectivité d’appareil dans le jumeau d’appareil. Vous avez également appris à interroger ces informations à l’aide du langage de requête IoT Hub de type SQL.

Utilisez les ressources suivantes :

* Envoyez les données de télémétrie à partir d’appareils en suivant le didacticiel [Bien démarrer avec IoT Hub](quickstart-send-telemetry-node.md).

* Pour savoir comment configurer des appareils à l’aide des propriétés de jumeau d’appareil souhaitées, suivez le tutoriel [Utiliser des propriétés souhaitées pour configurer des appareils](tutorial-device-twins.md).

* Pour savoir comment contrôler les appareils de façon interactive (par exemple pour mettre en marche un ventilateur à partir d’une application contrôlée par l’utilisateur), suivez le didacticiel [Utiliser des méthodes directes](quickstart-control-device-node.md).
