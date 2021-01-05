---
title: Démarrage rapide - Provisionner un appareil TPM simulé auprès du service Azure IoT Hub à l’aide de Node.js
description: 'Démarrage rapide : Créer et provisionner un appareil TPM simulé pour le service Azure IoT Hub Device Provisioning (DPS) à l’aide du SDK d’appareil Node.js. Ce démarrage rapide utilise des inscriptions individuelles.'
author: wesmc7777
ms.author: wesmc
ms.date: 11/08/2018
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
ms.custom:
- mvc
- amqp
- mqtt
- devx-track-js
ms.openlocfilehash: 6e7e986f658570553763001afdd58d7bb1880f94
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968177"
---
# <a name="quickstart-create-and-provision-a-simulated-tpm-device-using-nodejs-device-sdk-for-iot-hub-device-provisioning-service"></a>Démarrage rapide : Créer et approvisionner un appareil TPM simulé auprès du service IoT Hub Device Provisioning à l’aide du Kit de développement logiciel (SDK) Node.js

[!INCLUDE [iot-dps-selector-quick-create-simulated-device-tpm](../../includes/iot-dps-selector-quick-create-simulated-device-tpm.md)]

Dans ce guide de démarrage rapide, vous allez créer un appareil IoT simulé sur un ordinateur Windows. L’appareil simulé comprend un simulateur TPM sous forme de module de sécurité matériel (HSM). Vous allez utiliser l’exemple de code Node.js de l’appareil pour connecter cet appareil simulé à votre hub IoT en utilisant une inscription individuelle auprès du service de provisionnement des appareils (DPS).

## <a name="prerequisites"></a>Prérequis

- Connaissance des concepts liés à l’[approvisionnement](about-iot-dps.md#provisioning-process).
- Exécution des étapes décrites dans [Configurer le service IoT Hub Device Provisioning avec le portail Azure](./quick-setup-auto-provision.md).
- Compte Azure avec un abonnement actif. [Créez-en un gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- [Node.js v4.0+](https://nodejs.org).
- [Git](https://git-scm.com/download/).

[!INCLUDE [IoT Device Provisioning Service basic](../../includes/iot-dps-basic.md)]

## <a name="prepare-the-environment"></a>Préparer l’environnement 

1. Vérifiez que la [version 4.0 ou supérieure de Node.js](https://nodejs.org) est installée sur votre machine.

1. Assurez-vous que l’élément `git` est installé sur votre machine et est ajouté aux variables d’environnement accessibles à la fenêtre de commande. Consultez la section relative aux [outils clients de Software Freedom Conservancy](https://git-scm.com/download/) pour accéder à la dernière version des outils `git` à installer, qui inclut **Git Bash**, l’application de ligne de commande que vous pouvez utiliser pour interagir avec votre référentiel Git local. 


## <a name="simulate-a-tpm-device"></a>Simuler un appareil TPM

1. Ouvrez une invite de commandes ou Git Bash. Clonez le référentiel GitHub `azure-utpm-c` :
    
    ```cmd/sh
    git clone https://github.com/Azure/azure-utpm-c.git --recursive
    ```

1. Accédez au dossier racine GitHub et exécutez le simulateur [TPM](/windows/device-security/tpm/trusted-platform-module-overview) pour en faire le [HSM](https://azure.microsoft.com/blog/azure-iot-supports-new-security-hardware-to-strengthen-iot-security/) de l’appareil simulé. Il écoute un socket sur les ports 2321 et 2322. Ne fermez pas cette fenêtre de commande ; vous devez laisser ce simulateur s’exécuter jusqu’à la fin de ce guide de démarrage rapide : 

    ```cmd/sh
    .\azure-utpm-c\tools\tpm_simulator\Simulator.exe
    ```

1. Créez un dossier vide appelé **registerdevice**. Dans le dossier **registerdevice**, créez un fichier package.json à l’aide de la commande ci-dessous, à l’invite de commandes. Veillez à répondre à toutes les questions posées par `npm` ou acceptez les paramètres par défaut, s’ils vous conviennent :
   
    ```cmd/sh
    npm init
    ```

1. Installez les packages précurseurs suivants :

    ```cmd/sh
    npm install node-gyp -g
    npm install ffi -g
    ```

    > [!NOTE]
    > Il existe quelques erreurs connues lors de l’installation des packages ci-dessus. Pour résoudre ces problèmes, exécutez `npm install --global --production windows-build-tools` en utilisant une invite de commandes en mode **Exécuter en tant qu’administrateur**, exécutez `SET VCTargetsPath=C:\Program Files (x86)\MSBuild\Microsoft.Cpp\v4.0\V140` après avoir remplacé le chemin par votre version installée, puis exécutez de nouveau les commandes d’installation ci-dessus.
    >

1. Installez les packages suivants qui contiennent les composants utilisés pendant l’inscription :

   - un client de sécurité qui fonctionne avec TPM : `azure-iot-security-tpm`
   - un mode de transport pour connecter l’appareil au service Device Provisioning : `azure-iot-provisioning-device-http` ou `azure-iot-provisioning-device-amqp`
   - un client pour utiliser le mode de transport et le client de sécurité : `azure-iot-provisioning-device`

     Une fois l’appareil inscrit, vous pouvez utiliser les packages du client IoT Hub Device habituels pour connecter l’appareil à l’aide des informations d’identifications fournies lors de l’inscription. Éléments requis :

   - le client d’appareil : `azure-iot-device`
   - un mode de transport : `azure-iot-device-amqp`, `azure-iot-device-mqtt` ou `azure-iot-device-http`
   - le client de sécurité, déjà installé : `azure-iot-security-tpm`

     > [!NOTE]
     > Les exemples ci-dessous utilisent les modes de transports `azure-iot-provisioning-device-http` et `azure-iot-device-mqtt`.
     > 

     Vous pouvez installer tous ces packages d’un coup en exécutant la commande suivante dans l’invite de commandes, dans le dossier **registerdevice** :

       ```cmd/sh
       npm install --save azure-iot-device azure-iot-device-mqtt azure-iot-security-tpm azure-iot-provisioning-device-http azure-iot-provisioning-device
       ```

1. Avec un éditeur de texte, créer un fichier **ExtractDevice.js** dans le dossier **registerdevice**.

1. Ajoutez les instructions `require` ci-dessous au début du fichier **ExtractDevice.js** :
   
    ```
    'use strict';

    var tpmSecurity = require('azure-iot-security-tpm');
    var tssJs = require("tss.js");

    var myTpm = new tpmSecurity.TpmSecurityClient(undefined, new tssJs.Tpm(true));
    ```

1. Ajoutez la fonction suivante pour implémenter la méthode :
   
    ```
    myTpm.getEndorsementKey(function(err, endorsementKey) {
      if (err) {
        console.log('The error returned from get key is: ' + err);
      } else {
        console.log('the endorsement key is: ' + endorsementKey.toString('base64'));
        myTpm.getRegistrationId((getRegistrationIdError, registrationId) => {
          if (getRegistrationIdError) {
            console.log('The error returned from get registration id is: ' + getRegistrationIdError);
          } else {
            console.log('The Registration Id is: ' + registrationId);
            process.exit();
          }
        });
      }
    });
    ```

1. Enregistrez et fermez le fichier **ExtractDevice.js**. Exécutez l’exemple :

    ```cmd/sh
    node ExtractDevice.js
    ```

1. La fenêtre de sortie affiche la **_paire de clés de type EK_** et l’ **_ID d’inscription_** nécessaires à l’inscription de l’appareil. Notez ces valeurs. 


## <a name="create-a-device-entry"></a>Créer une entrée d’appareil

Le service Azure IoT Device Provisioning prend en charge deux types d’inscriptions :

- [Groupes d’inscription](concepts-service.md#enrollment-group) : utilisés pour inscrire plusieurs appareils connexes.
- [Inscriptions individuelles](concepts-service.md#individual-enrollment) : utilisées pour inscrire un seul appareil.

Cet article présente les inscriptions individuelles.

1. Connectez-vous au portail Azure, sélectionnez le bouton **Toutes les ressources** dans le menu de gauche et ouvrez votre service Device Provisioning.

1. Dans le menu du service Device Provisioning, sélectionnez **Gérer les inscriptions**. Sélectionnez l’onglet **Inscriptions individuelles**, puis le bouton **Ajouter une inscription individuelle** dans la partie supérieure. 

1. Dans le volet **Ajouter une inscription**, entrez les informations suivantes :
   - Sélectionnez **TPM** comme *mécanisme* d’attestation d’identité.
   - Entrez l’*ID d’inscription* et la *paire de clés de type EK (Endorsement Key)* pour votre appareil TPM notés plus tôt.
   - Sélectionnez un hub IoT lié à votre service d’approvisionnement.
   - Si vous le souhaitez, vous pouvez fournir les informations suivantes :
       - Entrez un *ID d’appareil* unique. Veillez à éviter les données sensibles lorsque vous affectez un nom à votre appareil. Si vous choisissez de ne pas en fournir un, l’ID d’inscription est utilisé à la place pour identifier l’appareil.
       - Mettez à jour l’**état du jumeau d’appareil initial** à l’aide de la configuration initiale de votre choix pour l’appareil.
   - Cela fait, appuyez sur le bouton **Enregistrer**. 

     ![Saisir les informations d’inscription d’appareil dans le panneau du portail](./media/quick-create-simulated-device/enter-device-enrollment.png)  

   Lorsque l’inscription aboutit, *l’ID d’inscription* de votre appareil s’affiche dans la liste sous l’onglet *Inscriptions individuelles*. 


## <a name="register-the-device"></a>Inscrire l’appareil

1. Dans le portail Azure, sélectionnez le panneau **Vue d’ensemble** du service Device Provisioning et notez les valeurs **_Point de terminaison d’appareil global_** et **_Étendue de l’ID_** .

    ![Extraction des informations de point de terminaison du service Device Provisioning à partir du panneau du Portail](./media/quick-create-simulated-device/extract-dps-endpoints.png) 

1. Avec un éditeur de texte, créer un fichier **RegisterDevice.js** dans le dossier **registerdevice**.

1. Ajoutez les instructions `require` ci-dessous au début du fichier **RegisterDevice.js** :
   
    ```
    'use strict';

    var ProvisioningTransport = require('azure-iot-provisioning-device-http').Http;
    var iotHubTransport = require('azure-iot-device-mqtt').Mqtt;
    var Client = require('azure-iot-device').Client;
    var Message = require('azure-iot-device').Message;
    var tpmSecurity = require('azure-iot-security-tpm');
    var ProvisioningDeviceClient = require('azure-iot-provisioning-device').ProvisioningDeviceClient;
    ```

    > [!NOTE]
    > Le **kit de développement logiciel (SDK) Azure IoT pour Node.js** prend en charge des protocoles supplémentaires comme _AMQP_, _AMQP WS_ et _MQTT WS_.  Pour plus d’exemples, consultez les [exemples de kit de développement logiciel (SDK) du service Device Provisioning pour Node.js](https://github.com/Azure/azure-iot-sdk-node/tree/master/provisioning/device/samples).
    > 

1. Ajoutez les variables **globalDeviceEndpoint** et **idScope**, et utilisez-les pour créer une instance **ProvisioningDeviceClient**. Remplacez les valeurs **{globalDeviceEndpoint}** et **{idScope}** par les valeurs **_Global Device Endpoint_** et **_ID Scope_** de l’**Étape 1** :
   
    ```
    var provisioningHost = '{globalDeviceEndpoint}';
    var idScope = '{idScope}';

    var tssJs = require("tss.js");
    var securityClient = new tpmSecurity.TpmSecurityClient('', new tssJs.Tpm(true));
    // if using non-simulated device, replace the above line with following:
    //var securityClient = new tpmSecurity.TpmSecurityClient();

    var provisioningClient = ProvisioningDeviceClient.create(provisioningHost, idScope, new ProvisioningTransport(), securityClient);
    ```

1. Ajoutez la fonction suivante pour implémenter la méthode sur l’appareil :
   
    ```
    provisioningClient.register(function(err, result) {
      if (err) {
        console.log("error registering device: " + err);
      } else {
        console.log('registration succeeded');
        console.log('assigned hub=' + result.registrationState.assignedHub);
        console.log('deviceId=' + result.registrationState.deviceId);
        var tpmAuthenticationProvider = tpmSecurity.TpmAuthenticationProvider.fromTpmSecurityClient(result.registrationState.deviceId, result.registrationState.assignedHub, securityClient);
        var hubClient = Client.fromAuthenticationProvider(tpmAuthenticationProvider, iotHubTransport);

        var connectCallback = function (err) {
          if (err) {
            console.error('Could not connect: ' + err.message);
          } else {
            console.log('Client connected');
            var message = new Message('Hello world');
            hubClient.sendEvent(message, printResultFor('send'));
          }
        };

        hubClient.open(connectCallback);

        function printResultFor(op) {
          return function printResult(err, res) {
            if (err) console.log(op + ' error: ' + err.toString());
            if (res) console.log(op + ' status: ' + res.constructor.name);
            process.exit(1);
          };
        }
      }
    });
    ```

1. Enregistrez et fermez le fichier **RegisterDevice.js**. Exécutez l’exemple :

    ```cmd/sh
    node RegisterDevice.js
    ```

1. Notez les messages qui simulent le démarrage et la connexion de l’appareil au service d’approvisionnement d’appareil pour obtenir des informations concernant votre IoT Hub. En cas de réussite du provisionnement de votre appareil simulé auprès du hub IoT lié à votre service de provisionnement, l’ID d’appareil s’affiche sur le panneau **Appareils IoT** du hub. 

    ![L’appareil est inscrit avec le hub IoT](./media/quick-create-simulated-device/hub-registration.png) 

    Si vous avez modifié la valeur par défaut de l’*état du jumeau d’appareil initial* dans l’entrée d’inscription de votre appareil, l’état du jumeau souhaité peut être extrait du hub et agir en conséquence. Pour en savoir plus, consultez [Comprendre et utiliser les jumeaux d’appareil IoT Hub](../iot-hub/iot-hub-devguide-device-twins.md)


## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez de manipuler et d’explorer davantage l’exemple de client d’appareil, ne nettoyez pas les ressources créées dans ce guide de démarrage rapide. Sinon, effectuez les étapes suivantes pour supprimer toutes les ressources créées par ce guide.

1. Fermez la fenêtre de sortie de l’exemple de client d’appareil sur votre machine.
1. Fermez la fenêtre du simulateur TPM sur votre machine.
1. Dans le menu de gauche du portail Azure, sélectionnez **Toutes les ressources**, puis votre service Device Provisioning. Ouvrez le panneau **Gérer les inscriptions** pour votre service, puis sélectionnez l’onglet **Inscriptions individuelles**. Cochez la case à côté de l’*ID D’INSCRIPTION* de l’appareil que vous avez inscrit dans ce guide de démarrage rapide, puis appuyez sur le bouton **Supprimer** dans la partie supérieure du volet. 
1. À partir du menu de gauche, dans le portail Azure, sélectionnez **Toutes les ressources**, puis votre hub IoT. Ouvrez le panneau **Appareils IoT** pour votre hub, cochez la case en regard de l’*ID D’APPAREIL* de l’appareil que vous avez inscrit dans ce guide de démarrage rapide, puis appuyez sur le bouton **Supprimer** dans la partie supérieure du volet.


## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un appareil simulé TPM sur votre machine et l’avez provisionné dans votre hub IoT avec le service IoT Hub Device Provisioning. Pour apprendre à inscrire un appareil TPM programmatiquement, passez au guide de démarrage rapide correspondant. 

> [!div class="nextstepaction"]
> [Démarrage rapide Azure - Inscrire un appareil TPM auprès du service Azure IoT Hub Device Provisioning](quick-enroll-device-tpm-node.md)