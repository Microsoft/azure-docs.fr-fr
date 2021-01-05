---
title: Connecter des appareils avec des certificats X.509 dans une application Azure IoT Central
description: Comment connecter des appareils avec des certificats X.509 à l’aide du kit de développement logiciel (SDK) d’appareil Node.js pour une application IoT Central
author: dominicbetts
ms.author: dobett
ms.date: 08/12/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.custom: device-developer
ms.openlocfilehash: d36cf2344891bb70ab5499e77699b111429a936b
ms.sourcegitcommit: b8a175b6391cddd5a2c92575c311cc3e8c820018
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96121826"
---
# <a name="how-to-connect-devices-with-x509-certificates-using-nodejs-device-sdk-for-iot-central-application"></a>Comment connecter des appareils avec des certificats X.509 à l’aide du kit de développement logiciel (SDK) d’appareil Node.js pour une application IoT Central

Pour sécuriser la communication entre un appareil et votre application, IoT Central prend en charge les signatures d’accès partagé (SAP) et les certificats X.509. Le didacticiel [Créer et connecter une application cliente à votre application Azure IoT Central](./tutorial-connect-device.md) utilise SAS. Dans cet article, vous allez apprendre à modifier l’exemple de code pour utiliser X.509.  Les certificats X.509 sont recommandés dans les environnements de production. Pour plus d’informations, consultez [Se connecter à Azure IoT Central](./concepts-get-connected.md).

Cet article montre deux façons d’utiliser X.509 : les [inscriptions de groupe](how-to-connect-devices-x509.md#use-a-group-enrollment), généralement utilisées dans un environnement de production, et les [inscriptions individuelles](how-to-connect-devices-x509.md#use-an-individual-enrollment), utiles pour les tests.

## <a name="prerequisites"></a>Prérequis

- Suivi du tutoriel [Créer et connecter une application cliente à votre application Azure IoT Central (JavaScript)](./tutorial-connect-device.md).
- [Git](https://git-scm.com/download/).
- Téléchargez et installez [OpenSSL](https://www.openssl.org/). Si vous utilisez Windows, vous pouvez utiliser les binaires de la [page OpenSSL sur SourceForge](https://sourceforge.net/projects/openssl/).

## <a name="use-a-group-enrollment"></a>Utiliser une inscription de groupe

Utilisez des certificats X.509 avec une inscription de groupe dans un environnement de production. Dans une inscription de groupe, vous ajoutez un certificat X.509 racine ou intermédiaire à votre application IoT Central. Les appareils avec des certificats feuilles dérivés du certificat racine ou intermédiaire peuvent se connecter à votre application.

## <a name="generate-root-and-device-cert"></a>Générer le certificat racine et d’appareil

Dans cette section, vous utilisez un certificat X.509 pour connecter un appareil avec un certificat dérivé du certificat du groupe d’inscription, qui peut se connecter à votre application IoT Central.

> [!WARNING]
> Cette façon de générer des certificats X.509 est destinée aux tests uniquement. Dans le cas d’un environnement de production, vous devez utiliser votre mécanisme sécurisé officiel pour la génération de certificats.

1. Ouvrez une invite de commandes. Clonez le référentiel GitHub pour les scripts de génération de certificat :

    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-node.git
    ```

1. Accédez au script du générateur de certificats et installez les packages requis :

    ```cmd/sh
    cd azure-iot-sdk-node/provisioning/tools
    npm install
    ```

1. Créez un certificat racine, puis dérivez un certificat d’appareil en exécutant le script :

    ```cmd/sh
    node create_test_cert.js root mytestrootcert
    node create_test_cert.js device sample-device-01 mytestrootcert
    ```

    > [!TIP]
    > Un ID d’appareil peut contenir des lettres, des chiffres et le caractère `-`.

Ces commandes génèrent trois fichiers pour le certificat racine et d’appareil

filename | contenu
-------- | --------
\<name\>_cert.pem | La partie publique du certificat X.509
\<name\>_key.pem | La clé privée pour le certificat X.509
\<name\>_fullchain.pem | Le rousseau entier pour le certificat X.509.

## <a name="create-a-group-enrollment"></a>Créer une inscription de groupe

1. Ouvrez votre application IoT Central et accédez à **Administration** dans le volet gauche, puis sélectionnez **Connexion de l’appareil**.

1. Sélectionnez **+ Créer un groupe d’inscriptions**, puis créez un nouveau groupe d’inscriptions appelé _MyX509Group_ avec un type d’attestation **Certificats (X.509)** .

1. Ouvrez le groupe d’inscriptions que vous avez créé, puis sélectionnez **Gérer le principal**.

1. Sélectionnez l’option Fichier et chargez le fichier de certificat racine appelé _mytestrootcert_cert.pem_ que vous avez généré précédemment :

    ![Chargement d’un certificat](./media/how-to-connect-devices-x509/certificate-upload.png)

1. Pour effectuer la vérification, générez le code de vérification, copiez-le, puis utilisez-le pour créer un certificat de vérification X.509 à l’invite de commandes :

    ```cmd/sh
    node create_test_cert.js verification --ca mytestrootcert_cert.pem --key mytestrootcert_key.pem --nonce  {verification-code}
    ```

1. Chargez le certificat de vérification signé _verification_cert.pem_ pour terminer la vérification :

    ![Certificat vérifié](./media/how-to-connect-devices-x509/verified.png)

Vous pouvez maintenant connecter des appareils qui ont un certificat X.509 dérivé de ce certificat racine principal.

Après avoir enregistré le groupe d’inscriptions, prenez note de l’étendue d’ID.

## <a name="run-sample-device-code"></a>Exécuter l’exemple de code d’appareil

1. Copiez les fichiers **sampleDevice01_key.pem** et **sampleDevice01_cert.pem** dans le dossier _azure-iot-sdk-node/device/samples/pnp_ contenant l’application **simple_thermostat.js**. Vous avez utilisé cette application quand que vous avez effectué le tutoriel [Connecter un appareil (JavaScript)](./tutorial-connect-device.md).

1. Accédez au dossier _azure-iot-sdk-node/device/samples/pnp_ contenant l’application **simple_thermostat.js**, puis exécutez la commande suivante pour installer le package X.509 :

    ```cmd/sh
    npm install azure-iot-security-x509 --save
    ```

1. Ouvrez le fichier **simple_thermostat.js** dans un éditeur de texte.

1. Modifiez les instructions `require` pour inclure les éléments suivants :

    ```javascript
    const fs = require('fs');
    const X509Security = require('azure-iot-security-x509').X509Security;
    ```

1. Ajoutez les quatre lignes suivantes à la section « Informations de connexion DPS » pour initialiser la variable `deviceCert` :

    ```javascript
    const deviceCert = {
      cert: fs.readFileSync(process.env.IOTHUB_DEVICE_X509_CERT).toString(),
      key: fs.readFileSync(process.env.IOTHUB_DEVICE_X509_KEY).toString()
    };
    ```

1. Modifiez la fonction `provisionDevice` qui crée le client en remplaçant la première ligne par ce qui suit :

    ```javascript
    var provSecurityClient = new X509Security(registrationId, deviceCert);
    ```

1. Dans la même fonction, modifiez la ligne qui définit la variable `deviceConnectionString` comme suit :

    ```javascript
    deviceConnectionString = 'HostName=' + result.assignedHub + ';DeviceId=' + result.deviceId + ';x509=true';
    ```

1. Dans la fonction `main` , ajoutez la ligne suivante après la ligne qui appelle `Client.fromConnectionString` :

    ```javascript
    client.setOptions(deviceCert);
    ```

1. Dans l’environnement de votre interpréteur de commandes, définissez les deux variables d’environnement suivantes :

    ```cmd/sh
    set IOTHUB_DEVICE_X509_CERT=sampleDevice01_cert.pem
    set IOTHUB_DEVICE_X509_KEY=sampleDevice01_key.pem
    ```

    > [!TIP]
    > Vous définissez les autres variables d’environnement requises lorsque vous avez terminé le tutoriel [Créer et connecter une application cliente à votre application Azure IoT Central](./tutorial-connect-device.md).

1. Exécutez le script et vérifiez que l’appareil a été provisionné correctement :

    ```cmd/sh
    node simple_thermostat.js
    ```

    Vous pouvez également vérifier que la télémétrie s’affiche dans le tableau de bord.

    ![Vérifier la télémétrie des appareils](./media/how-to-connect-devices-x509/telemetry.png)

## <a name="use-an-individual-enrollment"></a>Utiliser une inscription individuelle

Utilisez des certificats X.509 avec une inscription individuelle pour tester votre appareil et votre solution. Dans le cas d’une inscription individuelle, il n’existe aucun certificat X.509 racine ou intermédiaire dans votre application IoT Central. Les appareils utilisent un certificat X.509 auto-signé pour se connecter à votre application.

## <a name="generate-self-signed-device-cert"></a>Générer un certificat d’appareil auto-signé

Dans cette section, vous utilisez un certificat X.509 auto-signé pour connecter des appareils pour une inscription individuelle, afin d’inscrire un seul appareil. Les certificats auto-signés sont destinés aux tests uniquement.

Créez un certificat d’appareil X.509 auto-signé en exécutant le script. Veillez à utiliser uniquement des caractères alphanumériques minuscules et des traits d’union pour le nom du certificat :

  ```cmd/sh
    cd azure-iot-sdk-node/provisioning/tools
    node create_test_cert.js device mytestselfcertprimary
    node create_test_cert.js device mytestselfcertsecondary 
  ```

## <a name="create-individual-enrollment"></a>Créer une inscription individuelle

1. Dans l’application Azure IoT Central, sélectionnez **Appareils**, puis créez un appareil avec **ID d’appareil** comme _mytestselfcertprimary_ à partir du modèle d’appareil thermostat. Prenez note de l’**Étendue d’ID**, vous l’utiliserez plus tard.

1. Ouvrez l’appareil que vous avez créé, puis sélectionnez **Connecter**.

1. Sélectionnez **Inscriptions individuelles** comme **Méthode de connexion** et **Certificats (X.509)** comme mécanisme :

    ![Inscription individuelle](./media/how-to-connect-devices-x509/individual-device-connect.png)

1. Sélectionnez l’option Fichier sous Principal et chargez le fichier de certificat appelé _mytestselfcertprimary_cert.pem_ que vous avez généré précédemment.

1. Sélectionnez l’option Fichier pour le certificat secondaire et chargez le fichier de certificat appelé _mytestselfcertsecondary_cert.pem._ Sélectionnez ensuite **Enregistrer** :

    ![Chargement d’un certificat d’inscription individuelle](./media/how-to-connect-devices-x509/individual-enrollment.png)

L’appareil est maintenant configuré avec le certificat X.509.

## <a name="run-a-sample-individual-enrollment-device"></a>Exécuter un exemple d’appareil d’inscription individuelle

1. Copiez les fichiers _mytestselfcertprimary_key.pem_ et _mytestselfcertprimary_cert.pem_ dans le dossier _azure-iot-sdk-node/device/samples/pnp_ contenant l’application **simple_thermostat.js**. Vous avez utilisé cette application quand que vous avez effectué le tutoriel [Connecter un appareil (JavaScript)](./tutorial-connect-device.md).

1. Modifiez les variables d’environnement que vous avez utilisées, comme suit :

    ```cmd/sh
    set IOTHUB_DEVICE_DPS_DEVICE_ID=mytestselfcertprimary
    set IOTHUB_DEVICE_X509_CERT=mytestselfcertprimary_cert.pem
    set IOTHUB_DEVICE_X509_KEY=mytestselfcertprimary_key.pem
    ```

1. Exécutez le script et vérifiez que l’appareil a été provisionné correctement :

    ```cmd/sh
    node environmentalSensor.js
    ```

    Vous pouvez également vérifier que la télémétrie s’affiche dans le tableau de bord.

    ![Inscription individuelle à la télémétrie](./media/how-to-connect-devices-x509/telemetry-primary.png)

Vous pouvez également répéter les étapes ci-dessus pour le certificat _mytestselfcertsecondary_.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à connecter des appareils à l’aide de certificats X.509, l’étape suivante suggérée est de découvrir comment [superviser la connectivité des appareils à l’aide d’Azure CLI](howto-monitor-devices-azure-cli.md)
