---
title: Connecter Raspberry Pi à IoT Hub dans le cloud (Node.js)
description: Dans ce tutoriel, découvrez comment configurer et connecter Raspberry Pi à Azure IoT Hub pour lui permettre d’envoyer des données à la plateforme cloud Azure.
author: wesmc7777
manager: eliotgra
keywords: azure iot raspberry pi, raspberry pi iot hub, raspberry pi envoie des données vers le cloud, raspberry pi vers cloud
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 03/13/2020
ms.author: wesmc
ms.custom:
- 'Role: Cloud Development'
- devx-track-js
ms.openlocfilehash: c96f674b64401250d45542d0f59f13654cf37caa
ms.sourcegitcommit: 7e97ae405c1c6c8ac63850e1b88cf9c9c82372da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97802522"
---
# <a name="connect-raspberry-pi-to-azure-iot-hub-nodejs"></a>Connecter Raspberry Pi à Azure IoT Hub (Node.js)

[!INCLUDE [iot-hub-get-started-device-selector](../../includes/iot-hub-get-started-device-selector.md)]

Dans ce didacticiel, vous commencez par découvrir les principes fondamentaux de l’utilisation de Raspberry Pi exécutant Raspbian. Vous apprenez ensuite à connecter en toute transparence vos appareils au cloud avec [Azure IoT Hub](about-iot-hub.md). Pour obtenir des exemples Windows 10 IoT Standard, accédez au [centre de développement Windows](https://www.windowsondevices.com/).

Vous n’avez pas encore de kit ? Essayez [le simulateur en ligne Raspberry Pi](iot-hub-raspberry-pi-web-simulator-get-started.md). Ou achetez un nouveau kit [ici](https://azure.microsoft.com/develop/iot/starter-kits).

## <a name="what-you-do"></a>Procédure

* Créez un hub IoT.

* Inscrivez un appareil pour Pi dans votre IoT Hub.

* Configurez Raspberry Pi.

* Exécutez un exemple d’application sur Pi pour envoyer des données de capteur à votre IoT Hub.

## <a name="what-you-learn"></a>Contenu

* Création d’un Azure IoT Hub et obtention de la chaîne de connexion de votre nouvel appareil.

* Connexion de Pi à un capteur BME280.

* Collecte des données du capteur en exécutant un exemple d’application sur Pi.

* Envoi des données du capteur à votre IoT Hub.

## <a name="what-you-need"></a>Ce dont vous avez besoin

![Ce dont vous avez besoin](./media/iot-hub-raspberry-pi-kit-node-get-started/0-starter-kit.png)

* Une carte Raspberry Pi 2 ou Raspberry Pi 3.

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

* Un moniteur, un clavier USB et une souris connectés à Pi.

* Un Mac ou un PC sous Windows ou Linux.

* Une connexion web.

* Une carte microSD d’au moins 16 Go.

* Un adaptateur USB-SD ou une carte microSD pour graver l’image du système d’exploitation sur la carte microSD.

* Une alimentation 5 V 2 A avec le câble micro USB de 6 pieds (env. 183 cm).

Les éléments suivants sont facultatifs :

* Un capteur de température, de pression et d’humidité Adafruit BME280 assemblé.

* Une platine d’expérimentation.

* 6 câbles de liaison F/M.

* Une LED de 10 mm diffuse.

> [!NOTE]
> Si vous ne disposez pas des éléments facultatifs, vous pouvez utiliser des données de capteur simulé.

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Inscrire un nouvel appareil dans le hub IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="set-up-raspberry-pi"></a>Installer Raspberry Pi

### <a name="install-the-raspbian-operating-system-for-pi"></a>Installation du système d’exploitation Raspbian pour Pi

Préparez la carte microSD pour l’installation de l’image Raspbian.

1. Téléchargez Raspbian.

   a. [Raspbian Buster avec bureau](https://www.raspberrypi.org/software/) (fichier .zip).

   b. Extrayez l’image Raspbian dans un dossier sur votre ordinateur.

2. Installez Raspbian sur la carte microSD.

   a. [Téléchargez et installez l’utilitaire de graveur de carte SD Etcher](https://etcher.io/).

   b. Exécutez Etcher et sélectionnez l’image Raspbian extraite à l’étape 1.

   c. Sélectionnez le lecteur de carte microSD. Etcher a peut-être déjà sélectionné le bon lecteur.

   d. Cliquez sur Flash pour installer Raspbian sur la carte microSD.

   e. Retirez la carte microSD de votre ordinateur une fois l’installation terminée. Vous pouvez retirer directement la carte microSD en toute sécurité, car Etcher éjecte ou démonte automatiquement la carte microSD une fois le processus terminé.

   f. Insérez la carte microSD dans Pi.

### <a name="enable-ssh-and-i2c"></a>Activer SSH et I2C

1. Connectez Pi au moniteur, au clavier et à la souris.

2. Démarrez Pi puis connectez-vous à Raspbian avec le nom d’utilisateur `pi` et le mot de passe `raspberry`.

3. Cliquez sur l’icône Raspberry > **Préférences** > **Configuration de Raspberry Pi**.

   ![Le menu Préférences de Raspbian](./media/iot-hub-raspberry-pi-kit-node-get-started/1-raspbian-preferences-menu.png)

4. Sur l’onglet **Interfaces**, définissez **I2C** et **SSH** sur **Activer**, puis cliquez sur **OK**. Si vous ne possédez pas de capteurs physiques et que vous souhaitez utiliser des données de capteur simulé, cette étape est facultative.

   ![Activer I2C et SSH sur Raspberry Pi](./media/iot-hub-raspberry-pi-kit-node-get-started/2-enable-i2c-ssh-on-raspberry-pi.png)

> [!NOTE]
> Pour activer SSH et I2C, vous trouverez d’autres documents de référence sur [raspberrypi.org](https://www.raspberrypi.org/documentation/remote-access/ssh/) et [Adafruit.com](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-4-gpio-setup/configuring-i2c).

### <a name="connect-the-sensor-to-pi"></a>Connecter le capteur à Pi

Utilisez la platine d’expérimentation et les câbles de liaison pour connecter une LED et un BME280 à Pi comme suit. Si vous ne disposez pas de ce capteur, [ignorez cette section](#connect-pi-to-the-network).

![La connexion entre Raspberry Pi et le capteur](./media/iot-hub-raspberry-pi-kit-node-get-started/3-raspberry-pi-sensor-connection.png)

Le capteur BME280 peut collecter des données sur la température et l’humidité. La LED clignote quand l’appareil envoie un message vers le cloud.

Pour les broches du capteur, utilisez le câblage suivant :

| Début (capteur et LED)     | Fin (carte)            | Couleur du câble   |
| -----------------------  | ---------------------- | ------------: |
| VDD (broche 5G)             | ALIM 3,3 V (broche 1)       | Câble blanc   |
| GND (broche 7G)             | GND (broche 6)            | Câble marron   |
| SDI (broche 10G)            | I2C1 SDA (broche 3)       | Câble rouge     |
| SCK (broche 8G)             | I2C1 SCL (broche 5)       | Câble orange  |
| LED VDD (broche 18F)        | GPIO 24 (broche 18)       | Câble blanc   |
| LED GND (broche 17F)        | GND (broche 20)           | Câble noir   |

Cliquez pour afficher les [mappages de broches Raspberry Pi 2 et 3](/windows/iot-core/learn-about-hardware/pinmappings/pinmappingsrpi) à titre de référence.

Une fois le BME280 connecté à votre Raspberry Pi, il doit se présenter comme sur l’image ci-dessous.

![Pi et BME280 connectés](./media/iot-hub-raspberry-pi-kit-node-get-started/4-connected-pi.png)

### <a name="connect-pi-to-the-network"></a>Connexion de Pi au réseau

Mettez Pi sous tension à l’aide du câble micro USB et de l’alimentation. Utilisez le câble Ethernet pour connecter Pi à votre réseau câblé ou suivez les [instructions fournies par la Fondation Raspberry Pi](https://www.raspberrypi.org/documentation/configuration/wireless/) pour connecter Pi à votre réseau sans fil. Une fois que votre Pi est correctement connecté au réseau, vous devez relever son [adresse IP](https://www.raspberrypi.org/documentation/remote-access/ip-address.md).

![Connecté au réseau câblé](./media/iot-hub-raspberry-pi-kit-node-get-started/5-power-on-pi.png)

> [!NOTE]
> Assurez-vous que Pi est connecté au même réseau que votre ordinateur. Par exemple, si votre ordinateur est connecté à un réseau sans fil alors que Pi est connecté à un réseau câblé, il se peut que vous ne voyiez pas l’adresse IP dans la sortie devdisco.

## <a name="run-a-sample-application-on-pi"></a>Exécuter un exemple d’application sur Pi

### <a name="clone-sample-application-and-install-the-prerequisite-packages"></a>Cloner l’exemple d’application et installer les packages de composants requis

1. Connectez-vous à votre Raspberry Pi avec l’un des clients SSH suivants à partir de votre ordinateur hôte :

   **Utilisateurs Windows**

   a. Téléchargez et installez [PuTTY](https://www.putty.org/) pour Windows.

   b. Copiez l’adresse IP de votre Pi dans la section du nom d’hôte (ou adresse IP) et sélectionnez le type de connexion SSH.

   ![PuTTy](./media/iot-hub-raspberry-pi-kit-node-get-started/7-putty-windows.png)

   **Utilisateurs Mac et Ubuntu**

   Utilisez le client SSH intégré sur Ubuntu ou macOS. Vous devrez peut-être exécuter `ssh pi@<ip address of pi>` pour connecter le Pi via le protocole SSH.

   > [!NOTE]
   > Le nom d’utilisateur par défaut est `pi` et le mot de passe `raspberry`.

2. Installez Node.js et NPM sur votre Pi.

   Tout d’abord, vérifiez votre version de Node.js.

   ```bash
   node -v
   ```

   Si la version est inférieure à 10.x, ou que votre Pi ne comporte aucun Node.js, installez la dernière version.

   ```bash
   curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash
   sudo apt-get -y install nodejs
   ```

3. Clonez l’exemple d’application.

   ```bash
   git clone https://github.com/Azure-Samples/azure-iot-samples-node.git
   ```

4. Installez tous les packages de l’exemple, c’est-à-dire Azure IoT device SDK, la bibliothèque de capteur BME280 et la bibliothèque WiringPi.

   ```bash
   cd azure-iot-samples-node/iot-hub/Tutorials/RaspberryPiApp
   npm install
   ```

   > [!NOTE]
   >La procédure d’installation peut prendre plusieurs minutes, en fonction de la connexion de votre réseau.

### <a name="configure-the-sample-application"></a>Configurer l’exemple d’application

1. Ouvrez le fichier de configuration en exécutant les commandes suivantes :

   ```bash
   nano config.json
   ```

   ![Fichier de configuration](./media/iot-hub-raspberry-pi-kit-node-get-started/6-config-file.png)

   Dans ce fichier, deux éléments sont configurables. Le premier est `interval`, qui définit l’intervalle de temps (en millisecondes) entre les messages envoyés dans le cloud. Le deuxième est `simulatedData`, une valeur booléenne qui spécifie si les données de capteur simulé sont utilisées ou non.

   Si vous **ne disposez pas du capteur**, définissez la valeur `simulatedData` sur `true` pour que l’exemple d’application crée et utilise des données de capteur simulées.

   *Remarque : L’adresse i2c utilisée dans ce tutoriel est 0x77 par défaut. Selon votre configuration, elle peut également être 0x76 : si vous rencontrez une erreur i2c, essayez de définir la valeur sur 118 pour voir si cela fonctionne mieux. Pour voir l’adresse utilisée par votre capteur, exécutez `sudo i2cdetect -y 1` dans un interpréteur de commandes sur Raspberry Pi*

2. Enregistrez et quittez en appuyant sur Ctrl-O > Entrée > Ctrl-X.

### <a name="run-the-sample-application"></a>Exécuter l’exemple d’application

Exécutez l’exemple d’application en exécutant la commande suivante :

   ```bash
   sudo node index.js '<YOUR AZURE IOT HUB DEVICE CONNECTION STRING>'
   ```

   > [!NOTE]
   > Assurez-vous de copier-coller la chaîne de connexion de l’appareil entre les guillemets simples.

Vous devriez voir le résultat suivant, qui affiche les données de capteur et les messages envoyés à votre IoT Hub.

![Sortie - données de capteur envoyées depuis Raspberry Pi vers votre IoT Hub](./media/iot-hub-raspberry-pi-kit-node-get-started/8-run-output.png)

## <a name="read-the-messages-received-by-your-hub"></a>Lire les messages reçus par votre hub

Une façon de surveiller les messages reçus par votre hub IoT à partir de votre appareil consiste à utiliser Azure IoT Tools pour Visual Studio Code. Pour en savoir plus, consultez [Utiliser Azure IoT Tools pour Visual Studio Code afin d’envoyer et de recevoir des messages entre votre appareil et un IoT Hub](iot-hub-vscode-iot-toolkit-cloud-device-messaging.md).

Pour d’autres méthodes de traitement des données envoyées par votre appareil, passez à la section suivante.

## <a name="next-steps"></a>Étapes suivantes

Vous avez exécuté un exemple d’application pour collecter des données de capteur et les envoyer à votre IoT Hub.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
