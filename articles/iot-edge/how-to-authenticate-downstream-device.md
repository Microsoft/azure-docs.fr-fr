---
title: Authentifier des appareils en aval - Azure IoT Edge | Microsoft Docs
description: Explique comment authentifier les appareils en aval ou « appareils de nœud terminal » dans IoT Hub, et comment router leur connexion via des appareils de passerelle Azure IoT Edge.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 10/15/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: f2dd7cac8370c261f24f5587e801bd621fbdb0f0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016996"
---
# <a name="authenticate-a-downstream-device-to-azure-iot-hub"></a>Authentifier un appareil en aval auprès d’Azure IoT Hub

Dans un scénario de passerelle transparente, les appareils en aval (également appelés appareils de nœud terminal ou appareils enfants) ont besoin d’identités dans IoT Hub, comme tous les autres appareils. Cet article décrit les options permettant d’authentifier un appareil en aval dans IoT Hub, puis il montre comment déclarer la connexion de passerelle.

La configuration d’une connexion de passerelle transparente s’effectue en trois grandes étapes. Cet article concerne la deuxième étape :

1. Configurez l’appareil de passerelle en tant que serveur pour permettre aux appareils en aval de s’y connecter en toute sécurité. Configurez la passerelle pour recevoir des messages des appareils en aval et les acheminer vers la destination qui convient. Ces étapes sont décrites dans [Configurer un appareil IoT Edge en tant que passerelle transparente](how-to-create-transparent-gateway.md).
2. **Créez une identité d’appareil pour l’appareil en aval afin qu’il puisse s’authentifier sur IoT Hub. Configurez l’appareil en aval pour envoyer des messages par le biais de l’appareil de passerelle.**
3. Connectez l’appareil en aval à l’appareil de passerelle et commencez à envoyer des messages. Pour plus d’informations sur ces étapes, voir [Connecter un appareil en aval à une passerelle Azure IoT Edge](how-to-connect-downstream-device.md).

Pour s’authentifier auprès d’IoT Hub, les appareils en aval disposent de trois méthodes d’authentification : par clé symétrique (ou « clé d’accès partagé »), par certificat X.509 autosigné et par certificat X.509 signé par l’autorité de certification. Les étapes d’authentification sont similaires à celles utilisées pour configurer n’importe quel appareil IoT Edge avec IoT Hub, avec toutefois quelques différences au niveau de la déclaration de la relation de passerelle.

Le provisionnement automatique des appareils en aval avec le service Azure IoT Hub Device Provisioning (DPS) n’est pas pris en charge.

## <a name="prerequisites"></a>Prérequis

Suivez les étapes décrites dans [Configurer un appareil IoT Edge en tant que passerelle transparente](how-to-create-transparent-gateway.md).

Si vous utilisez l’authentification X.509, vous allez générer des certificats pour votre appareil en aval. Utilisez le même certificat d’autorité de certification racine et le même script de génération de certificat que ceux utilisés pour l’article relatif à la passerelle transparente.

Cet article fait référence au *nom d’hôte de passerelle* à plusieurs endroits. Le nom d’hôte de passerelle est déclaré avec le paramètre **hostname** dans le fichier config.yaml sur l’appareil de passerelle IoT Edge. Il est référencé dans la chaîne de connexion de l’appareil en aval. Le nom d’hôte de passerelle doit être résolu en une adresse IP, à l’aide du DNS ou d’une entrée de fichier hôte sur l’appareil en aval.

## <a name="register-device-with-iot-hub"></a>Inscrire un appareil auprès d’IoT Hub

Choisissez la façon dont vous souhaitez que votre appareil en aval s’authentifie auprès d’IoT Hub :

* [Authentification par clé symétrique](#symmetric-key-authentication) : IoT Hub crée une clé que vous placez sur l’appareil en aval. Lorsque l’appareil s’authentifie, IoT Hub vérifie que les deux clés correspondent. Vous n’êtes pas tenu de créer des certificats supplémentaires pour utiliser l’authentification par clé symétrique.

  Cette méthode est plus rapide pour commencer si vous testez des passerelles dans un scénario de développement ou de test.

* [Authentification par certificat X.509 autosigné](#x509-self-signed-authentication) : Parfois appelée authentification par empreinte numérique, car vous partagez l’empreinte numérique certificat X.509 de l’appareil avec IoT Hub.

  L’authentification par certificat est recommandée pour les appareils dans les scénarios de production.

* [Authentification par certificat X.509 signé par une autorité de certification](#x509-ca-signed-authentication) : Chargez le certificat d’autorité de certification racine dans IoT Hub. Lorsque les appareils présentent leur certificat X. 509 à des fins d’authentification, IoT Hub vérifie qu’il appartient à une chaîne d’approbation signée par le même certificat d’autorité de certification racine.

  L’authentification par certificat est recommandée pour les appareils dans les scénarios de production.

### <a name="symmetric-key-authentication"></a>Authentification par clé symétrique

L’authentification par clé symétrique (ou authentification par clé d’accès partagé) est la méthode la plus simple pour s’authentifier auprès d’IoT Hub. Avec l’authentification par clé symétrique, une clé base64 est associée à l’ID de votre appareil IoT dans IoT Hub. Vous devez inclure cette clé dans vos applications IoT pour que votre appareil puisse la présenter lorsqu’il se connecte à IoT Hub.

Ajoutez un nouvel appareil IoT dans votre hub IoT à l’aide du portail Azure, d’Azure CLI ou de l’extension IoT pour Visual Studio Code. N’oubliez pas que les appareils en aval doivent être identifiés dans IoT Hub comme des appareils IoT standard, et non comme des appareils IoT Edge.

Lorsque vous créez une identité d’appareil, fournissez les informations suivantes :

* Créez un ID pour votre appareil.

* Sélectionnez **Clé symétrique** comme type d’authentification.

* Sélectionnez **Définir un appareil parent**, puis sélectionnez l’appareil de passerelle IoT Edge via lequel cet appareil en aval doit se connecter. Vous pourrez toujours changer de parent par la suite.

   ![Créer un ID d’appareil avec une authentification par clé symétrique dans le portail](./media/how-to-authenticate-downstream-device/symmetric-key-portal.png)

Vous pouvez également utiliser l’[extension IoT pour Azure CLI](https://github.com/Azure/azure-iot-cli-extension) afin d’effectuer la même opération. L’exemple suivant utilise la commande [az iot hub device-identity](/cli/azure/ext/azure-iot/iot/hub/device-identity) pour créer un appareil IoT avec authentification par clé symétrique et affecter un appareil parent :

```cli
az iot hub device-identity create -n {iothub name} -d {new device ID} --pd {existing gateway device ID}
```

Ensuite, [récupérez et Modifiez la chaîne de connexion](#retrieve-and-modify-connection-string) de sorte que votre appareil sache qu’il se connecte via sa passerelle.

### <a name="x509-self-signed-authentication"></a>Authentification par certificat X.509 autosigné

Pour l’authentification par certificat X.509 autosigné, parfois appelée authentification par empreinte numérique, vous devez créer des certificats et les placer sur votre appareil en aval. Ces certificats contiennent une empreinte numérique que vous partagez avec IoT Hub pour l’authentification.

1. Avec votre certificat d’autorité de certification, créez deux certificats d’appareil (un principal et un secondaire) pour l’appareil en aval.

   Si vous n’avez pas d’autorité de certification pour créer les certificats X.509, vous pouvez utiliser les scripts des certificats de démonstration IoT Edge pour [Créer des certificats d’appareils en aval](how-to-create-test-certificates.md#create-downstream-device-certificates). Suivez les étapes de création de certificats autosignés. Utilisez le même certificat d’autorité de certification racine que celui qui a généré les certificats pour votre appareil de passerelle.

   Si vous créez vos propres certificats, assurez-vous que le nom d'objet du certificat d'appareil est défini sur l’ID d’appareil que vous allez utiliser lors de l’inscription de l’appareil IoT dans Azure IoT Hub. Ce paramètre est requis pour l’authentification.

2. Récupérez l’empreinte digitale SHA1 (ou « empreinte numérique » dans l’interface IoT Hub) de chaque certificat (il s’agit d’une chaîne de 40 caractères hexadécimaux). Pour afficher le certificat et trouver l’empreinte digitale, utilisez la commande openssl suivante :

   * Windows :

     ```PowerShell
     openssl x509 -in <path to primary device certificate>.cert.pem -text -fingerprint
     ```

   * Linux :

     ```Bash
     openssl x509 -in <path to primary device certificate>.cert.pem -text -fingerprint | sed 's/[:]//g'
     ```

   Exécutez cette commande deux fois, une fois pour le certificat principal et une fois pour le certificat secondaire. Vous fournissez des empreintes digitales pour les deux certificats lorsque vous inscrivez un nouvel appareil IoT à l’aide de certificats X.509 auto-signés.

3. Accédez à votre hub IoT dans le portail Azure et créez une nouvelle identité d’appareil IoT avec les valeurs suivantes :

   * Fournissez l’**ID d’appareil** qui correspond au nom du sujet de vos certificats d’appareil.
   * Sélectionnez **X.509 autosigné** comme type d’authentification.
   * Collez les chaînes hexadécimales que vous avez copiées à partir du certificat primaire et du certificat secondaire de votre appareil.
   * Sélectionnez **Définir un appareil parent**, puis choisissez l’appareil de passerelle IoT Edge via lequel cet appareil en aval doit se connecter. Vous pourrez toujours changer de parent par la suite.

   ![Créer un ID d’appareil avec une authentification par certificat X.509 autosigné dans le portail](./media/how-to-authenticate-downstream-device/x509-self-signed-portal.png)

4. Copiez les certificats d’appareil principal et secondaire et leurs clés vers n’importe quel emplacement sur l’appareil en aval. Déplacez également une copie du certificat d’autorité de certification racine partagée qui a généré le certificat de périphérique d’appareil de passerelle et les certificats d’appareil en aval.

   Vous allez référencer ces fichiers de certificat dans les applications d’appareil en aval qui se connectent à IoT Hub. Vous pouvez utiliser un service comme [Azure Key Vault](../key-vault/index.yml) ou une fonction comme [SCP (Secure Copy Protocol)](https://www.ssh.com/ssh/scp/) pour déplacer les fichiers de certificats.

5. Selon votre langue par défaut, consultez des exemples de référencement de certificats X.509 dans des applications IoT :

   * C# : [Configurer la sécurité X.509 dans votre Azure IoT Hub](../iot-hub/iot-hub-security-x509-get-started.md#authenticate-your-x509-device-with-the-x509-certificates)
   * C: [iotedge_downstream_device_sample.c](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client/samples/iotedge_downstream_device_sample)
   * Node.js : [simple_sample_device_x509.js](https://github.com/Azure/azure-iot-sdk-node/blob/master/device/samples/simple_sample_device_x509.js)
   * Java : [SendEventX509.java](https://github.com/Azure/azure-iot-sdk-java/tree/master/device/iot-device-samples/send-event-x509)
   * Python : [send_message_x509.py](https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-device/samples/async-hub-scenarios/send_message_x509.py)

Vous pouvez également utiliser l’[extension IoT pour Azure CLI](https://github.com/Azure/azure-iot-cli-extension) afin d’effectuer la même opération de création d’appareil. L’exemple suivant utilise la commande [az iot hub device-identity](/cli/azure/ext/azure-iot/iot/hub/device-identity) pour créer un appareil IoT avec authentification auto-signée X.509 et affecter un appareil parent :

```cli
az iot hub device-identity create -n {iothub name} -d {device ID} --pd {gateway device ID} --am x509_thumbprint --ptp {primary thumbprint} --stp {secondary thumbprint}
```

Ensuite, [récupérez et Modifiez la chaîne de connexion](#retrieve-and-modify-connection-string) de sorte que votre appareil sache qu’il se connecte via sa passerelle.

### <a name="x509-ca-signed-authentication"></a>Authentification par certificat X.509 signé par une autorité de certification

Pour l’authentification par certificat X.509 signés par l’autorité de certification, vous avez besoin d’un certificat d’autorité de certification racine inscrit dans IoT Hub, que vous utilisez pour signer les certificats de votre appareil en aval. Tous les appareils qui utilisent un certificat émis par le certificat d’autorité de certification racine ou l’un de ses certificats intermédiaires sont autorisés à s’authentifier.

Cette section est basée sur les instructions de l’article IoT Hub [Configurer la sécurité X.509 dans votre Azure IoT Hub](../iot-hub/iot-hub-security-x509-get-started.md).

1. Avec votre certificat d’autorité de certification, créez deux certificats d’appareil (un principal et un secondaire) pour l’appareil en aval.

   Si vous n’avez pas d’autorité de certification pour créer les certificats X.509, vous pouvez utiliser les scripts des certificats de démonstration IoT Edge pour [Créer des certificats d’appareils en aval](how-to-create-test-certificates.md#create-downstream-device-certificates). Suivez les étapes de création de certificats signés par une autorité de certification. Utilisez le même certificat d’autorité de certification racine que celui qui a généré les certificats pour votre appareil de passerelle.

2. Suivez les instructions de la section [Inscrire des certificats d’autorité de certification X.509 auprès de votre IoT Hub](../iot-hub/iot-hub-security-x509-get-started.md#register-x509-ca-certificates-to-your-iot-hub) dans l’article *Configurer la sécurité X.509 dans votre Azure IoT Hub*. Dans cette section, vous allez effectuer les étapes suivantes :

   1. Chargez un certificat d’autorité de certification racine. Si vous utilisez les certificats de démonstration, l’autorité de certification racine est **\<path>/certs/azure-iot-test-only.root.ca.cert.pem**.

   2. Vérifiez que vous disposez de votre propre certificat d’autorité de certification racine.

3. Suivez les instructions de la section [Créer un appareil X.509 pour votre IoT Hub](../iot-hub/iot-hub-security-x509-get-started.md#create-an-x509-device-for-your-iot-hub) dans l’article *Configurer la sécurité X.509 dans votre Azure IoT Hub*. Dans cette section, vous allez effectuer les étapes suivantes :

   1. Ajoutez un nouvel appareil. Dans **ID d’appareil**, fournissez un nom en minuscules, puis choisissez le type d’authentification **Autorité de certification X.509 signée**.

   2. Définissez un appareil parent. Sélectionnez **Définir un appareil parent**, puis choisissez l’appareil de passerelle IoT Edge qui doit fournir la connexion à IoT Hub.

4. Créez une chaîne de certificats pour votre appareil en aval. Utilisez le certificat d’autorité de certification racine que vous avez chargé dans IoT Hub pour créer cette chaîne. Utilisez l’ID d’appareil en minuscules que vous avez attribué à votre identité d’appareil dans le portail.

5. Copiez le certificat et les clés d’appareil dans n’importe quel emplacement sur l’appareil en aval. Déplacez également une copie du certificat d’autorité de certification racine partagée qui a généré le certificat de périphérique d’appareil de passerelle et les certificats d’appareil en aval.

   Vous allez référencer ces fichiers dans les applications d’appareil en aval qui se connectent à IoT Hub. Vous pouvez utiliser un service comme [Azure Key Vault](../key-vault/index.yml) ou une fonction comme [SCP (Secure Copy Protocol)](https://www.ssh.com/ssh/scp/) pour déplacer les fichiers de certificats.

6. Selon votre langue par défaut, consultez des exemples de référencement de certificats X.509 dans des applications IoT :

   * C# : [Configurer la sécurité X.509 dans votre Azure IoT Hub](../iot-hub/iot-hub-security-x509-get-started.md#authenticate-your-x509-device-with-the-x509-certificates)
   * C: [iotedge_downstream_device_sample.c](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client/samples/iotedge_downstream_device_sample)
   * Node.js : [simple_sample_device_x509.js](https://github.com/Azure/azure-iot-sdk-node/blob/master/device/samples/simple_sample_device_x509.js)
   * Java : [SendEventX509.java](https://github.com/Azure/azure-iot-sdk-java/tree/master/device/iot-device-samples/send-event-x509)
   * Python : [send_message_x509.py](https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-device/samples/async-hub-scenarios/send_message_x509.py)

Vous pouvez également utiliser l’[extension IoT pour Azure CLI](https://github.com/Azure/azure-iot-cli-extension) afin d’effectuer la même opération de création d’appareil. L’exemple suivant utilise la commande [az iot hub device-identity](/cli/azure/ext/azure-iot/iot/hub/device-identity) pour créer un appareil IoT avec authentification X.509 signée par une autorité de certification et affecter un appareil parent :

```cli
az iot hub device-identity create -n {iothub name} -d {device ID} --pd {gateway device ID} --am x509_ca
```

Ensuite, [récupérez et Modifiez la chaîne de connexion](#retrieve-and-modify-connection-string) de sorte que votre appareil sache qu’il se connecte via sa passerelle.

## <a name="retrieve-and-modify-connection-string"></a>Récupérer et Modifier la chaîne de connexion

Après avoir créé une identité d’appareil IoT dans le portail, vous pouvez récupérer ses clés primaires ou secondaires. L’une de ces clés doit être incluse dans la chaîne de connexion que les applications utilisent pour communiquer avec IoT Hub. Avec l’authentification par clé symétrique, IoT Hub fournit la chaîne de connexion intégralement formée dans les détails de l’appareil. Vous devez ajouter des informations supplémentaires concernant l’appareil de passerelle dans la chaîne de connexion.

Pour les appareils en aval, les chaînes de connexion ont besoin des composants suivants :

* Le hub IoT auquel l’appareil se connecte : `Hostname={iothub name}.azure-devices.net`
* L’ID d’appareil qui est inscrit auprès du hub : `DeviceID={device ID}`
* La méthode d’authentification, qu’il s’agisse d’une clé symétrique ou de certificats X.509
  * Si l’utilisation de l’authentification par clé symétrique fournit la clé primaire ou secondaire : `SharedAccessKey={key}`
  * Si vous utilisez l’authentification par certificat X.509, fournissez un indicateur : `x509=true`
* L’appareil de passerelle via lequel l’appareil se connecte. Fournissez la valeur **hostname** qui figure dans le fichier config.yaml de l’appareil de passerelle IoT Edge : `GatewayHostName={gateway hostname}`

Dans son intégralité, une chaîne de connexion ressemble à ceci :

```
HostName=myiothub.azure-devices.net;DeviceId=myDownstreamDevice;SharedAccessKey=xxxyyyzzz;GatewayHostName=myGatewayDevice
```

Ou :

```
HostName=myiothub.azure-devices.net;DeviceId=myDownstreamDevice;x509=true;GatewayHostName=myGatewayDevice
```

Grâce à la relation parent/enfant, vous pouvez simplifier la chaîne de connexion en appelant directement la passerelle en tant qu’hôte de connexion. Par exemple :

```
HostName=myGatewayDevice;DeviceId=myDownstreamDevice;SharedAccessKey=xxxyyyzzz
```

Vous utiliserez cette chaîne de connexion modifiée dans l’article suivant de la série de la passerelle transparente.

## <a name="next-steps"></a>Étapes suivantes

À ce stade, vous disposez d’un appareil IoT Edge inscrit auprès de votre hub IoT et configuré en tant que passerelle transparente. Vous disposez également d’un appareil en aval inscrit auprès de votre hub IoT et pointant vers son appareil de passerelle.

Maintenant, vous devez configurer votre appareil en aval pour qu’il approuve l’appareil de passerelle et s’y connecte en toute sécurité. Passez à l’article suivant de la série de la passerelle transparente, [Connecter un appareil en aval à une passerelle Azure IoT Edge](how-to-connect-downstream-device.md).