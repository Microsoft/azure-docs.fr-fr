---
title: Connecter IoT DevKit AZ3166 à IoT Hub
description: Dans ce didacticiel, découvrez comment configurer le kit IoT DevKit AZ3166 et le connecter à Azure IoT Hub pour qu’il puisse envoyer des données sur la plateforme cloud Azure.
author: wesmc7777
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 06/25/2019
ms.author: wesmc
ms.custom:
- mqtt
- 'Role: Cloud Development'
- devx-track-azurecli
ms.openlocfilehash: 3e5ab1667ee0cda459785efa624bd7f4fc6818b8
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97562954"
---
# <a name="connect-iot-devkit-az3166-to-azure-iot-hub"></a>Connecter IoT DevKit AZ3166 à Azure IoT Hub

[!INCLUDE [iot-hub-get-started-device-selector](../../includes/iot-hub-get-started-device-selector.md)]

Vous pouvez utiliser le kit [MXChip IoT DevKit](https://microsoft.github.io/azure-iot-developer-kit/) pour développer et réaliser des prototypes de solutions IoT (Internet of Things) qui tirent parti des services Microsoft Azure. Il comprend une carte compatible Arduino avec un grand choix de périphériques et de capteurs élaborés, un package de carte open source et une [galerie d’exemples](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/) enrichie.

## <a name="what-you-learn"></a>Contenu

* Comment créer un hub IoT et enregistrer un appareil auprès du kit MXChip IoT DevKit.
* Comment connecter le DevKit IoT au Wi-Fi et configurer la chaîne de connexion Hub IoT.
* Comment transmettre les données de télémétrie du capteur DevKit à votre hub IoT.
* Comment préparer l’environnement de développement et développer des applications pour le DevKit IoT.

Vous n’avez pas encore de DevKit ? Essayez le [simulateur DevKit](https://azure-samples.github.io/iot-devkit-web-simulator/) ou [achetez un DevKit](https://aka.ms/iot-devkit-purchase).

Le code source de tous les didacticiels de DevKit figure dans la [galerie d’exemples de code](/samples/browse/?term=mxchip).

## <a name="what-you-need"></a>Ce dont vous avez besoin

- Une carte MXChip IoT DevKit avec câble micro-USB. [Achetez-en une maintenant](https://aka.ms/iot-devkit-purchase).
- Un ordinateur exécutant Windows 10, macOS 10.10 ou Ubuntu 18.04+.
- Un abonnement Azure actif. [Activez un compte Microsoft Azure pour un essai gratuit de 30 jours](https://azureinfo.microsoft.com/us-freetrial.html).
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]
  
## <a name="prepare-your-hardware"></a>Préparation du matériel

Raccordez le matériel suivant à votre ordinateur :

* Carte DevKit
* Câble micro-USB

![Matériel requis](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/hardware.jpg)

Pour brancher le kit DevKit sur votre ordinateur, procédez comme suit :

1. Branchez l’extrémité USB sur l’ordinateur.

2. Branchez la fiche micro-USB sur le kit DevKit.

3. Le témoin vert de l’alimentation confirme le branchement.

   ![Branchements du matériel](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/connect.jpg)

## <a name="quickstart-send-telemetry-from-devkit-to-an-iot-hub"></a>Démarrage rapide : Envoyer des données de télémétrie d’un DevKit vers un Hub IoT

Le démarrage rapide utilise le microprogramme précompilé de DevKit pour transmettre les données de télémétrie vers le Hub IoT. Avant de l’exécuter, vous créez un hub IoT et inscrivez un appareil auprès de ce hub.

### <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

### <a name="register-a-device"></a>Inscrire un appareil

Un appareil doit être inscrit dans votre hub IoT pour pouvoir se connecter. Dans ce démarrage rapide, vous utilisez Azure Cloud Shell pour inscrire un appareil simulé.

1. Exécutez les commandes suivantes dans Azure Cloud Shell pour créer l’identité d’appareil.

   **YourIoTHubName** : Remplacez l’espace réservé ci-dessous par le nom que vous avez choisi pour votre hub IoT.

   **MyNodeDevice** : Nom de l’appareil que vous inscrivez. Utilisez **MyNodeDevice** comme indiqué. Si vous choisissez un autre nom pour votre appareil, vous devez utiliser ce nom pour l’ensemble de cet article et mettre à jour le nom de l’appareil dans les exemples d’application avant de les exécuter.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name YourIoTHubName --device-id MyNodeDevice
    ```

   > [!NOTE]
   > Si vous obtenez une erreur lors de l’exécution de `device-identity`, installez l’[Extension Azure IoT pour Azure CLI](https://github.com/Azure/azure-iot-cli-extension/blob/dev/README.md).
   > Exécutez la commande suivante afin d’ajouter l’extension Microsoft Azure IoT pour Azure CLI à votre instance Cloud Shell. L’extension IoT ajoute des commandes propres à IoT Hub, à IoT Edge et au service IoT Device Provisioning (DPS) à Azure CLI.
   > 
   > ```azurecli-interactive
   > az extension add --name azure-iot
   >  ```
   >
  
1. Exécutez les commandes suivantes dans Azure Cloud Shell pour obtenir la _chaîne de connexion_ à l’appareil que vous venez d’inscrire :

   **YourIoTHubName** : Remplacez l’espace réservé ci-dessous par le nom que vous avez choisi pour votre hub IoT.

    ```azurecli-interactive
    az iot hub device-identity connection-string show --hub-name YourIoTHubName --device-id MyNodeDevice --output table
    ```

    Notez la chaîne de connexion à l’appareil, qui ressemble à ce qui suit :

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyNodeDevice;SharedAccessKey={YourSharedAccessKey}`

    Vous utiliserez cette valeur plus loin dans ce démarrage rapide.

### <a name="send-devkit-telemetry"></a>Transférer les données de télémétrie DevKit

Le DevKit se connecte à un point de terminaison spécifique de l’appareil sur votre IoT Hub et envoie les données de télémétrie (température et humidité).

1. Téléchargez la dernière version du [microprogramme GetStarted](https://aka.ms/devkit/prod/getstarted/latest) pour le DevKit IoT.

1. Assurez-vous que le DevKit IoT se connecte à votre ordinateur via une connexion USB. Ouvrez l’Explorateur de fichiers. Vous devriez voir un périphérique de stockage de masse USB nommé **AZ3166**.

    ![Ouvrir l'Explorateur Windows](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/az3166-usb.png)

1. Faites glisser et déposez le microprogramme téléchargé dans le périphérique de stockage de masse. Il doit clignoter automatiquement.

    ![Copier le microprogramme](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/copy-firmware.png)

1. Dans DevKit, maintenez le **bouton B** enfoncé, appuyez brièvement sur le bouton **Réinitialiser**, puis relâchez le **bouton B**. Votre DevKit passe en mode Point d’accès. Pour la confirmation, l’écran affiche l’identificateur SSID (Service Set Identifier) du kit DevKit et l’adresse IP du portail de configuration.

    ![Bouton de réinitialisation, bouton B et SSID](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/wifi-ap.jpg)

    ![Définir le mode « Point d’accès »](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/set-ap-mode.gif)

1. Utilisez un navigateur web sur un autre appareil compatible Wi-Fi (un téléphone mobile ou un ordinateur) pour vous connecter au SSID du DevKit IoT affiché lors de l’étape précédente. Si le système demande un mot de passe, laissez-le vide.

    ![Connecter le SSID](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/connect-ssid.png)

1. Ouvrez **192.168.0.1** dans le navigateur. Sélectionnez le Wi-Fi auquel vous voulez connecter le DevKit IoT, entrez le mot de passe Wi-Fi, puis collez la chaîne de connexion d’appareil que vous avez notée précédemment. Cliquez ensuite sur Enregistrer.

    ![Interface utilisateur de configuration](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/configuration-ui.png)

    > [!NOTE]
    > Le DevKit IoT prend uniquement en charge le réseau 2,4 GHz. Pour plus d’informations, consultez le [FAQ](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/#wi-fi-configuration).

1. Les informations Wi-Fi et la chaîne de connexion de l’appareil seront stockées dans le DevKit IoT lorsque la page de résultat s’affiche.

    ![Résultat de la configuration](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/configuration-ui-result.png)

    > [!NOTE]
    > Dès lors que le Wi-Fi est configuré, vos informations d’identification pour cette connexion sont persistantes sur l’appareil, même s’il est débranché.

1. Le DevKit IoT redémarre au bout de quelques secondes. Dans l’écran DevKit, l’adresse IP du DevKit s’affiche, suivie par les données de télémétrie, telles que les valeurs de température et d’humidité, ainsi que le nombre de messages envoyés vers Azure IoT Hub.

    ![Adresse IP Wi-Fi](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/wifi-ip.jpg)

    ![Envoi de données](media/iot-hub-arduino-devkit-az3166-get-started/quickstarts/sending-data.jpg)

1. Pour vérifier les données de télémétrie envoyées à Azure, exécutez la commande suivante dans Azure Cloud Shell :

    ```azurecli
    az iot hub monitor-events --hub-name YourIoTHubName --output table
    ```

## <a name="prepare-the-development-environment"></a>Préparer l’environnement de développement

Suivez ces étapes pour préparer l’environnement de développement pour le DevKit :

### <a name="install-visual-studio-code-with-azure-iot-tools-extension-package"></a>Installer Visual Studio Code avec le package d’extension Azure IoT Tools

1. Installez [l’IDE Arduino](https://www.arduino.cc/en/Main/Software). Il fournit la chaîne d’outils nécessaire pour la compilation et le chargement du code Arduino.
    * **Windows** : utilisez la version Windows Installer. N’installez pas depuis l’App Store.
    * **macOS** : faites glisser et déposez le fichier **Arduino.app** extrait dans le dossier `/Applications`.
    * **Ubuntu** : décompressez-le dans un dossier tel que `$HOME/Downloads/arduino-1.8.8`.

2. Installez [Visual Studio Code](https://code.visualstudio.com/), un éditeur de code source inter-plateformes avec des outils de complétion de code et de prise en charge du débogage puissants, tels qu’IntelliSense, ainsi que des extensions complètes que vous pouvez installer à partir du marketplace.

3. Lancez VS Code, recherchez **Arduino** sur la Place de marché des extensions, puis installez-le. Cette extension fournit les expériences améliorées pour le développement sur la plateforme Arduino.

    ![Installer Arduino](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/install-arduino.png)

4. Recherchez [Azure IoT Tools](https://aka.ms/azure-iot-tools) sur la Place de marché des extensions, puis installez-le.

    ![Capture d’écran montrant Azure IoT Tools sur le marketplace d’extensions.](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/install-azure-iot-tools.png)

    Ou copiez et collez cette URL dans une fenêtre de navigateur : `vscode:extension/vsciot-vscode.azure-iot-tools`

    > [!NOTE]
    > Le pack d’extension Azure IoT Tools contient [Azure IoT Device Workbench](https://aka.ms/iot-workbench), qui est utilisé pour le développement et le déboguage sur divers appareils d’IoT devkit. L’[extension Azure IoT Hub](https://aka.ms/iot-toolkit), également fournie avec le pack d’extension Azure IoT Tools, est utilisée pour gérer Azure IoT Hubs et interagir avec cet outil.

5. Configurez VS Code avec les paramètres Arduino.

    Dans Visual Studio Code, cliquez sur **Fichier > Préférences > Paramètres** (sur macOS, **Code > Préférences > Paramètres**). Cliquez ensuite sur l’icône **Ouvrir les paramètres (JSON)** dans le coin supérieur droit de la page *Paramètres*.

    ![Installer Azure IoT Tools](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/user-settings-arduino.png)

    Ajoutez les lignes suivantes pour configurer Arduino en fonction de votre plateforme : 

    * **Windows** :

        ```json
        "arduino.path": "C:\\Program Files (x86)\\Arduino",
        "arduino.additionalUrls": "https://raw.githubusercontent.com/VSChina/azureiotdevkit_tools/master/package_azureboard_index.json"
        ```

    * **macOS** :

        ```json
        "arduino.path": "/Applications",
        "arduino.additionalUrls": "https://raw.githubusercontent.com/VSChina/azureiotdevkit_tools/master/package_azureboard_index.json"
        ```

    * **Ubuntu** :

        Remplacez l'espace réservé **{username}** ci-dessous par votre nom d'utilisateur.

        ```json
        "arduino.path": "/home/{username}/Downloads/arduino-1.8.8",
        "arduino.additionalUrls": "https://raw.githubusercontent.com/VSChina/azureiotdevkit_tools/master/package_azureboard_index.json"
        ```

6. Cliquez sur `F1` pour ouvrir la palette de commandes, puis entrez et sélectionnez **Arduino : Gestionnaire de cartes**. Recherchez **AZ3166** et installez la dernière version.

    ![Installer le kit SDK DevKit](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/install-az3166-sdk.png)

### <a name="install-st-link-drivers"></a>Installer les pilotes ST-Link

[ST-Link/V2](https://www.st.com/en/development-tools/st-link-v2.html) est l’interface USB utilisée par IoT DevKit pour communiquer avec votre ordinateur de développement. Vous devez l’installer sur Windows pour flasher le code d’appareil compilé pour le DevKit. Suivez les étapes spécifiques au système d’exploitation pour permettre à l’ordinateur d’accéder à votre appareil.

* **Windows** : téléchargez et installez le pilote USB à partir du [site web STMicroelectronics](https://www.st.com/en/development-tools/stsw-link009.html).
* **macOS** : aucun pilote n'est requis pour macOS.
* **Ubuntu** : Exécutez les commandes dans le terminal et déconnectez-vous, puis reconnectez-vous, afin que la modification de groupe prenne effet :

    ```bash
    # Copy the default rules. This grants permission to the group 'plugdev'
    sudo cp ~/.arduino15/packages/AZ3166/tools/openocd/0.10.0/linux/contrib/60-openocd.rules /etc/udev/rules.d/
    sudo udevadm control --reload-rules

    # Add yourself to the group 'plugdev'
    # Logout and log back in for the group to take effect
    sudo usermod -a -G plugdev $(whoami)
    ```

Maintenant, vous êtes prêt à préparer et à configurer votre environnement de développement. Laissez-nous générer l’exemple de GetStarted que vous venez d’exécuter.

## <a name="build-your-first-project"></a>Générer votre premier projet

### <a name="open-sample-code-from-sample-gallery"></a>Ouvrez l’exemple de code à partir de la galerie d’exemples

IoT DevKit contient une galerie enrichie d’exemples que vous pouvez utiliser pour apprendre à connecter DevKit à divers services Azure.

1. Assurez-vous que votre IoT DevKit n’est **pas connecté** à votre ordinateur. Démarrez d’abord VS Code, puis connectez le DevKit à votre ordinateur.

1. Cliquez sur `F1` pour ouvrir la palette de commandes, puis tapez et sélectionnez **Azure IoT Device Workbench : Ouvrir des exemples...** Puis sélectionnez **IoT DevKit** en tant que carte.

1. Dans la page des exemples IoT Workbench, recherchez **Bien démarrer** et cliquez sur **Ouvrir l’exemple**. Sélectionnez ensuite le chemin d’accès par défaut pour télécharger l’exemple de code.

    ![Ouvrir l'exemple](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/open-sample.png)

### <a name="provision-azure-iot-hub-and-device"></a>Approvisionner Azure IoT Hub et votre appareil

Au lieu d’approvisionner Azure IoT Hub et votre appareil à partir du portail Microsoft Azure, vous pouvez le faire dans VS Code sans quitter l’environnement de développement.

1. Dans la nouvelle fenêtre de projet ouverte, cliquez sur `F1` pour ouvrir la palette de commandes, puis tapez et sélectionnez **Azure IoT Device Workbench : Approvisionner les services Azure...** Suivez le guide détaillé pour terminer le provisionnement de votre instance Azure IoT Hub et la création de l’appareil IoT Hub.

    ![Commande d'approvisionnement](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/provision.png)

    > [!NOTE]
    > Si vous n’êtes pas connecté dans Azure. Suivez la notification de la fenêtre contextuelle pour vous connecter.

1. Sélectionnez l’abonnement que vous souhaitez utiliser.

    ![Sélectionner l'abonnement](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/select-subscription.png)

1. Puis sélectionnez ou créez un nouveau [groupe de ressources](../azure-resource-manager/management/overview.md#terminology).

    ![Sélection du groupe de ressources](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/select-resource-group.png)

1. Dans le groupe de ressources que vous avez spécifié, suivez le guide pour sélectionner ou créer une instance d’Azure IoT Hub.

    ![Étapes de sélection d'IoT Hub](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/iot-hub-provision.png)

    ![Sélectionner IoT Hub](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/select-iot-hub.png)

    ![IoT Hub sélectionné](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/iot-hub-selected.png)

1. Dans la fenêtre de sortie, vous verrez l'instance d'Azure IoT Hub approvisionnée.

    ![Instance d'IoT Hub approvisionnée](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/iot-hub-provisioned.png)

1. Sélectionnez ou créez un appareil dans Azure IoT Hub que vous avez provisionné.

    ![Étapes de sélection d'un appareil IoT](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/iot-device-provision.png)

    ![Sélectionner un appareil IoT provisionné](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/select-iot-device.png)

1. Vous disposez maintenant d’une instance provisionnée d’Azure IoT Hub sur laquelle un appareil a été créé. De même, la chaîne de connexion d’appareil est enregistrée dans VS Code pour configurer l’IoT DevKit plus tard.

    ![Approvisionnement terminé](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/provision-done.png)

### <a name="configure-and-compile-device-code"></a>Configurer et compiler le code de l’appareil

1. Dans la barre d’état en bas à droite, vérifiez que **MXCHIP AZ3166** est indiqué en tant que carte sélectionnée et que le port série avec **STMicroelectronics** est utilisé.

    ![Sélectionner la carte et COM](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/select-com.png)

1. Cliquez sur `F1` pour ouvrir la palette de commandes, puis tapez et sélectionnez **Azure IoT Device Workbench : Configurer les paramètres de l’appareil...** , puis sélectionnez **Configurer la chaîne de connexion d'appareil > Sélectionner la chaîne de connexion d’appareil IoT Hub**.

1. Dans DevKit, maintenez le **bouton A** enfoncé, appuyez sur le bouton de **réinitialisation** et relâchez-le, puis relâchez le **bouton A**. Votre DevKit passe en mode de configuration et enregistre la chaîne de connexion.

    ![Chaîne de connexion](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/connection-string.png)

1. Cliquez à nouveau sur `F1`, puis tapez et sélectionnez **Azure IoT Device Workbench : Charger le code de l’appareil**. Cette opération démarre la compilation et le chargement du code dans le DevKit.

    ![Chargement Arduino](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/arduino-upload.png)

Le DevKit redémarre et commence à exécuter le code.

> [!NOTE]
> En cas d’erreurs ou d’interruptions, vous pouvez toujours effectuer une récupération en exécutant à nouveau la commande.

## <a name="test-the-project"></a>Tester le projet

### <a name="view-the-telemetry-sent-to-azure-iot-hub"></a>Afficher les données de télémétrie envoyées à Azure IoT Hub

Cliquez sur l'icône de branchement « power » de la barre d'état pour ouvrir l'analyse en série :

![Serial Monitor](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/serial-monitor.png)

L’exemple d’application s’exécute correctement si les résultats suivants s’affichent :

* Serial Monitor affiche le message envoyé à IoT Hub.
* Le témoin lumineux du kit MXChip IoT DevKit clignote.

![Sortie de Serial Monitor](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/result-serial-output.png)

> [!NOTE]
> Vous pouvez rencontrer une erreur pendant le test, suite à quoi la LED ne clignote pas, le portail Azure n’affiche pas les données entrantes provenant de l’appareil, alors que l’écran OLED de l’appareil s’affiche comme étant **En cours d’exécution...** . Pour résoudre le problème, dans le portail Azure, dans l’IoT Hub accédez à l’appareil et envoyez-lui un message. Si vous voyez la réponse suivante dans le moniteur de série dans VS Code, il est possible que la communication directe à partir de l’appareil soit bloquée au niveau du routeur. Vérifiez les règles du pare-feu et du routeur qui sont configurées pour les appareils connectés. Assurez-vous également que le port de sortie 1833 est ouvert.
> 
> ERREUR : mqtt_client. c (ln 454) : Erreur : échec lors de l’ouverture de la connexion au point de terminaison  
> INFO : >>>État de connexion : déconnecté  
> ERREUR : tlsio_mbedtls. c (ln 604) : Échec de l’ouverture des E/S sous-jacentes  
> ERROR : mqtt_client.c (ln 1042) : Erreur : échec d’io_open  
> ERREUR : iothubtransport_mqtt_common.c (ln 2283) : échec de la connexion à l’adresse atcsliothub.azure-devices.net.  
> INFO : >>>Reconnectez-vous.  
> INFO : Version d’IoThub : 1.3.6  

### <a name="view-the-telemetry-received-by-azure-iot-hub"></a>Afficher les données de télémétrie reçues par Azure IoT Hub

Vous pouvez utiliser les [Outils IoT Azure](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) pour superviser les messages d’appareil-à-cloud (D2C) dans IoT Hub.

1. Connectez-vous au [portail Azure](https://portal.azure.com/), puis recherchez le hub IoT que vous avez créé.

    ![Portail Azure](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/azure-iot-hub-portal.png)

1. Dans le volet **Stratégies d’accès partagé**, cliquez sur la **stratégie iothubowner** et notez la chaîne de connexion de votre IoT Hub.

    ![Chaîne de connexion IoT Hub](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/azure-portal-conn-string.png)

1. Dans VS Code, cliquez sur `F1`, puis tapez et sélectionnez **Azure IoT Hub : Définir la chaîne de connexion IoT Hub**. Copier-y la chaîne de connexion.

    ![Définir la chaîne de connexion IoT Hub](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/set-iothub-connection-string.png)

1. Développez le volet **APPAREILS AZURE IOT HUB** situé à gauche, cliquez avec le bouton de droite sur le nom de l’appareil que vous avez créé, puis sélectionnez **Démarrer l’analyse du point de terminaison d’événements intégrés**.

    ![Superviser le message D2C](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/monitor-d2c.png)

1. Dans le volet **SORTIE**, les messages appareil-à-cloud entrants vers l’IoT Hub s’affichent.

    ![Capture d’écran montrant les messages D2C entrants dans le hub IoT.](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/d2c-output.png)

## <a name="review-the-code"></a>Vérifier le code

`GetStarted.ino` est le principal fichier de croquis Arduino.

![Messages appareil-à-cloud](media/iot-hub-arduino-devkit-az3166-get-started/getting-started/code.png)

Pour voir comment les données de télémétrie de l’appareil sont transmises à Azure IoT Hub, ouvrez le fichier `utility.cpp` dans le même dossier. Consultez le [référenciel de l’API](https://microsoft.github.io/azure-iot-developer-kit/docs/apis/arduino-language-reference/) pour apprendre à utiliser des capteurs et des périphériques sur IoT DevKit.

Le `DevKitMQTTClient` utilisé est un wrapper de **iothub_client** provenant des [Kits de développement logiciel Microsoft Azure IoT et des bibliothèques pour C](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client), afin d’interagir avec Azure IoT Hub.

## <a name="problems-and-feedback"></a>Problèmes et commentaires

Si vous rencontrez des problèmes, vous pouvez rechercher une solution dans la [FAQ de l’IoT DevKit](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) ou contactez-nous à partir de [Gitter](https://gitter.im/Microsoft/azure-iot-developer-kit). Vous pouvez également nous faire part de vos remarques en laissant un commentaire sur cette page.

## <a name="next-steps"></a>Étapes suivantes

Vous avez correctement connecté un kit MXChip IoT DevKit à votre hub IoT et avez envoyé des données de capteur collectées à votre hub IoT.

[!INCLUDE [iot-hub-get-started-az3166-next-steps](../../includes/iot-hub-get-started-az3166-next-steps.md)]
