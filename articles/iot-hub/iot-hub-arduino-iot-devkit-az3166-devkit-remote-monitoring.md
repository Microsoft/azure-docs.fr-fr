---
title: Connecter DevKit MXChip IoT à la supervision à distance Azure IoT Hub
description: Dans ce tutoriel, découvrez comment envoyer l’état des capteurs sur IoT DevKit AZ3166 vers l’accélérateur de solution de supervision à distance Azure IoT.
author: liydu
manager: jeffya
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 02/02/2018
ms.author: liydu
ms.openlocfilehash: a5a69c35e5c7e33443c3810cad9ab4e21b048328
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92148823"
---
# <a name="connect-mxchip-iot-devkit-to-azure-iot-remote-monitoring-solution-accelerator"></a>Connecter MXChip IoT DevKit à l’accélérateur de solution de supervision à distance Azure IoT

Dans ce tutoriel, vous allez apprendre à exécuter un exemple d’application sur votre kit DevKit pour envoyer les données des capteurs vers votre accélérateur de solution de supervision à distance Azure IoT.

[IoT MXChip DevKit](https://aka.ms/iot-devkit) est une carte tout-en-un compatible Arduino qui est équipée de périphériques et de capteurs élaborés. Vous pouvez développer pour ce kit à l’aide de l’[extension Visual Studio Code pour Arduino](https://aka.ms/arduino). Par ailleurs, il s’accompagne d’un [catalogue de projets](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/) en plein développement pour vous guider dans la réalisation de prototypes de solutions IoT (Internet des objets) qui exploitent les services Microsoft Azure.

## <a name="what-you-need"></a>Ce dont vous avez besoin

Suivez le [Guide de mise en route](./iot-hub-arduino-iot-devkit-az3166-get-started.md) pour :

* Connecter votre DevKit au Wi-Fi
* Préparer l’environnement de développement

Un abonnement Azure actif. Si vous n’en avez pas, vous pouvez vous inscrire via l’une de ces deux méthodes :

* Activez un [compte d’évaluation Microsoft Azure gratuit pendant 30 jours](https://azure.microsoft.com/free/).

* Réclamez votre [crédit Azure](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) si vous êtes abonné à MSDN ou Visual Studio

## <a name="create-an-azure-iot-remote-monitoring-solution-accelerator"></a>Créer un accélérateur de solution de supervision à distance Azure IoT

1. Accédez au [site des accélérateurs de solution Azure IoT](https://www.azureiotsolutions.com/) et cliquez sur **Créer une solution**.

   ![Sélection du type d’accélérateur de solution Azure IoT](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-solution-types.png)

   > [!WARNING]
   > Par défaut, cet exemple crée un hub IoT S2 après avoir créé un accélérateur de solution de supervision à distance IoT. Si ce hub IoT n’est pas utilisé avec un très grand nombre d’appareils, nous vous recommandons vivement de le rétrograder de la version S2 à la version S1 et de supprimer l’accélérateur de solution de supervision à distance IoT pour pouvoir aussi supprimer le hub IoT associé au moment requis. 

2. Sélectionnez **Surveillance à distance**.

3. Entrez le nom d’une solution, sélectionnez un abonnement et une région, puis cliquez sur **Créer une solution**. La solution peut prendre un certain temps à configurer.
  
   ![Créer une solution](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-new-solution.png)

4. Une fois l’approvisionnement terminé, cliquez sur **Lancer**. Certains appareils simulés sont créés pour la solution pendant le processus d’approvisionnement. Cliquez sur **APPAREILS** pour les contrôler.

   ![tableau de bord](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-new-solution-created.png)
  
   ![Console](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-console.png)

5. Cliquez sur **AJOUTER UN APPAREIL**.

6. Cliquez sur **Ajouter nouveau** pour **Appareil personnalisé**.
  
   ![Ajouter un nouvel appareil](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-add-new-device.png)

7. Cliquez sur **Me laisser définir mon propre ID d’appareil**, entrez `AZ3166`, puis cliquez sur **Créer**.
  
   ![Création d’un appareil avec un ID](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-new-device-configuration.png)

8. Notez le **Nom d’hôte IoT Hub**, puis cliquez sur **Terminé**.

## <a name="open-the-remotemonitoring-sample"></a>Ouvrir l’exemple RemoteMonitoring

1. Déconnectez le kit DevKit de votre ordinateur, s’il est connecté.

2. Démarrez VS Code.

3. Connectez le kit DevKit à votre ordinateur. VS Code détecte automatiquement votre kit DevKit et ouvre les pages suivantes :

   * La page d’introduction du kit DevKit.
   * Exemples Arduino : Exemples pratiques pour démarrer avec DevKit.

4. Développez la section **EXEMPLES ARDUINO** à gauche, accédez à **Exemples pour MXCHIP AZ3166 > AzureIoT** et sélectionnez **RemoteMonitoring**. Une nouvelle fenêtre VS Code s’ouvre avec le dossier de projet qu’elle contient.

   > [!NOTE]
   > S’il vous arrive de fermer le volet par inadvertance, vous pouvez le rouvrir. Utilisez `Ctrl+Shift+P` (macOS : `Cmd+Shift+P`) pour ouvrir la palette de commandes, tapez **Arduino**, puis recherchez et sélectionnez **Arduino : Exemples**.

## <a name="provision-required-azure-services"></a>Configurer les services Azure requis

Dans la fenêtre de la solution, exécutez votre tâche par le biais de `Ctrl+P` (macOS : `Cmd+P`) en entrant `task cloud-provision` dans la zone de texte fournie.

Dans le terminal VS Code, une ligne de commande interactive vous guide dans l’approvisionnement des services Azure nécessaires.

![Approvisionner des ressources Azure](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/provision.png)

## <a name="build-and-upload-the-device-code"></a>Générer et charger le code de l’appareil

1. Utilisez `Ctrl+P` (macOS : `Cmd + P`) et tapez **task config-device-connection**.

2. Le terminal vous demande si vous souhaitez utiliser une chaîne de connexion qu’il récupère à partir de l’étape `task cloud-provision`. Vous pourriez également entrer votre propre chaîne de connexion d’appareil en cliquant sur « Créer nouveau... »

3. Le terminal vous invite à passer en mode de configuration. Pour cela, maintenez le bouton A enfoncé, puis appuyez sur le bouton de réinitialisation et relâchez-le. L’écran affiche l’ID du kit DevKit et « Configuration ».

   ![Chaîne de connexion d’entrée](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/config-device-connection.png)

4. Une fois la tâche `task config-device-connection` terminée, cliquez sur `F1` pour charger les commandes VS Code, puis sélectionnez `Arduino: Upload`. VS Code commence la vérification et le chargement de l’ébauche de projet Arduino.
  
   ![Vérification et téléchargement de l’ébauche de projet Arduino](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/arduino-upload.png)

Le DevKit redémarre et commence à exécuter le code.

## <a name="test-the-project"></a>Tester le projet

Quand l’exemple d’application s’exécute, le kit DevKit envoie les données des capteurs par WiFi à votre accélérateur de solution de supervision à distance Azure IoT. Pour voir le résultat, procédez comme suit :

1. Accédez à votre accélérateur de solution de supervision à distance Azure IoT, puis cliquez sur **TABLEAU DE BORD**.

2. Dans la console de la solution de supervision à distance, vous voyez l’état des capteurs de votre kit DevKit.

   ![Données des capteurs dans l’accélérateur de solution de supervision à distance Azure IoT](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/sensor-status.png)

## <a name="change-device-id"></a>Modifier l’ID de l’appareil

Si vous souhaitez remplacer la valeur **AZ3166** codée en dur par un ID d’appareil personnalisé dans le code, modifiez la ligne de code affichée dans l’[exemple de surveillance à distance](../iot-accelerators/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoring-v2.md).

## <a name="problems-and-feedback"></a>Problèmes et commentaires

Si vous rencontrez des problèmes, consultez les [FAQ IoT DevKit](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) ou contactez-nous par le biais des canaux suivants :

* [Gitter.im](https://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>Étapes suivantes

Vous avez appris à connecter un appareil DevKit à votre accélérateur de solution de supervision à distance Azure IoT et à visualiser les données des capteurs ; nous vous suggérons à présent les étapes suivantes :

* [Vue d’ensemble des accélérateurs de solution Azure IoT](/azure/iot-suite/)

* [Kit de développement IoT](https://microsoft.github.io/azure-iot-developer-kit/)