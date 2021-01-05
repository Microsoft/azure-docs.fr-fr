---
title: Créer un traducteur avec Azure Functions, Cognitive Services et IoT DevKit
description: Utiliser le microphone sur un DevKit IoT pour recevoir un message vocal, puis utiliser Azure Cognitive Services pour le traiter et afficher le texte traduit en anglais
author: liydu
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 12/19/2018
ms.author: liydu
ms.custom: devx-track-csharp
ms.openlocfilehash: be26c6fe03dac9b9ff9dbff4a2bdce391ec0837e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96024161"
---
# <a name="use-iot-devkit-az3166-with-azure-functions-and-cognitive-services-to-make-a-language-translator"></a>Utiliser le DevKit IoT AZ3166 avec Azure Functions et Cognitive Services pour créer un traducteur

Dans cet article, vous découvrez comment transformer IoT DevKit en traducteur à l’aide [d’Azure Cognitive Services](https://azure.microsoft.com/services/cognitive-services/). Il enregistre votre voix et la convertit en un texte traduit en anglais affiché sur l’écran du DevKit.

[IoT MXChip DevKit](https://aka.ms/iot-devkit) est une carte tout-en-un compatible Arduino qui est équipée de périphériques et de capteurs élaborés. Vous pouvez développer pour ce kit à l’aide du pack d’extension [Azure IoT Device Workbench](https://aka.ms/iot-workbench) ou [Azure IoT Tools](https://aka.ms/azure-iot-tools) dans Visual Studio Code. Le [catalogue de projets](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/) contient des exemples d’applications permettant de créer des prototypes de solutions IoT.

## <a name="before-you-begin"></a>Avant de commencer

Pour effectuer les étapes de ce didacticiel, commencez par exécuter les tâches suivantes :

* Préparez le DevKit en suivant les étapes contenues dans l’article [Connecter IoT DevKit AZ3166 à Azure IoT Hub dans le cloud](./iot-hub-arduino-iot-devkit-az3166-get-started.md).

## <a name="create-azure-cognitive-service"></a>Créer Azure Cognitive Service

1. Dans le portail Azure, cliquez sur **Créer une ressource**, puis recherchez **Speech**. Remplissez le formulaire pour créer le service Speech.
  ![Service Speech](media/iot-hub-arduino-iot-devkit-az3166-translator/speech-service.png)

1. Accédez au service Speech que vous venez de créer, cliquez sur la section **Clés** à copier, puis notez la valeur **Key1** correspondant au DevKit qui y accède.
  ![Copier des clés](media/iot-hub-arduino-iot-devkit-az3166-translator/copy-keys.png)

## <a name="open-sample-project"></a>Ouvrir un exemple de projet

1. Assurez-vous que votre IoT DevKit n’est **pas connecté** à votre ordinateur. Démarrez d’abord VS Code, puis connectez le DevKit à votre ordinateur.

1. Cliquez sur `F1` pour ouvrir la palette de commandes, puis tapez et sélectionnez **Azure IoT Device Workbench : Ouvrir des exemples...** Puis sélectionnez **IoT DevKit** en tant que carte.

1. Dans la page des exemples IoT Workbench, recherchez **Traducteur DevKit**, puis cliquez sur **Ouvrir l’exemple**. Sélectionnez ensuite le chemin d’accès par défaut pour télécharger l’exemple de code.
  ![Ouvrir l’exemple](media/iot-hub-arduino-iot-devkit-az3166-translator/open-sample.png)

## <a name="use-speech-service-with-azure-functions"></a>Utiliser le service Speech avec Azure Functions

1. Dans VS Code, cliquez sur `F1`, puis tapez et sélectionnez **Azure IoT Device Workbench : Approvisionner les services Azure...** ![Provisionner les services Azure](media/iot-hub-arduino-iot-devkit-az3166-translator/provision.png)

1. Suivez les étapes permettant de terminer le provisionnement d’Azure IoT Hub et d’Azure Functions.
   ![Étapes du provisionnement](media/iot-hub-arduino-iot-devkit-az3166-translator/provision-steps.png)

   Notez le nom de l’appareil Azure IoT Hub que vous avez créé.

1. Ouvrez `Functions\DevKitTranslatorFunction.cs` et mettez à jour les lignes de code suivantes avec le nom d’appareil et la clé du service Speech dont vous avez pris note.
   ```csharp
   // Subscription Key of Speech Service
   const string speechSubscriptionKey = "";

   // Region of the speech service, see https://docs.microsoft.com/azure/cognitive-services/speech-service/regions for more details.
   const string speechServiceRegion = "";

   // Device ID
   const string deviceName = "";
   ```

1. Cliquez sur `F1`, puis tapez et sélectionnez **Azure IoT Device Workbench : Déployer sur Azure...** Si VS Code invite à confirmer l’opération de redéploiement, cliquez sur **Oui**.
   ![Avertissement de déploiement](media/iot-hub-arduino-iot-devkit-az3166-translator/deploy-warning.png)

1. Vérifiez que le déploiement a réussi.
   ![Déploiement réussi](media/iot-hub-arduino-iot-devkit-az3166-translator/deploy-success.png)

1. Dans le portail Azure, accédez à la section **Applications de fonctions**, recherchez l’application Azure Function que vous venez de créer. Cliquez sur `devkit_translator`, puis sur **</> Récupérer l’URL de la fonction** pour copier l’URL.
   ![Copier l’URL de la fonction](media/iot-hub-arduino-iot-devkit-az3166-translator/get-function-url.png)

1. Collez l’URL dans le fichier `azure_config.h`.
   ![Configuration d’Azure](media/iot-hub-arduino-iot-devkit-az3166-translator/azure-config.png)

   > [!NOTE]
   > Si l’application de fonction ne fonctionne pas correctement, consultez cette section de [questions fréquentes (FAQ)](https://microsoft.github.io/azure-iot-developer-kit/docs/faq#compilation-error-for-azure-function) pour résoudre le problème.

## <a name="build-and-upload-device-code"></a>Générer et charger le code de l’appareil

1. Passez le DevKit en **mode de configuration** en effectuant les opérations suivantes :
   * Maintenez enfoncé le bouton **A**.
   * Appuyez sur le bouton **Réinitialiser**, puis relâchez-le.

   L’écran affiche l’ID et la **Configuration** du DevKit.

   ![Mode de configuration du DevKit](media/iot-hub-arduino-iot-devkit-az3166-translator/devkit-configuration-mode.png)

1. Cliquez sur `F1`, puis tapez et sélectionnez **Azure IoT Device Workbench : Configurer les paramètres de l’appareil... > Configurer la chaîne de connexion d’appareil**. Sélectionnez **Sélectionner la chaîne de connexion d'appareil IoT Hub** pour la configurer dans le DevKit.
   ![Configurer la chaîne de connexion](media/iot-hub-arduino-iot-devkit-az3166-translator/configure-connection-string.png)

1. La notification s’affiche une fois l’opération effectuée avec succès.
   ![Réussite de la configuration de la chaîne de connexion](media/iot-hub-arduino-iot-devkit-az3166-translator/configure-connection-string-success.png)

1. Cliquez à nouveau sur `F1`, puis tapez et sélectionnez **Azure IoT Device Workbench : Charger le code de l’appareil**. Cette opération démarre la compilation et le chargement du code dans le DevKit.
   ![Chargement de l’appareil](media/iot-hub-arduino-iot-devkit-az3166-translator/device-upload.png)

## <a name="test-the-project"></a>Tester le projet

Après l’initialisation de l’application, suivez les instructions sur l’écran de DevKit. La langue source par défaut est le chinois.

Pour sélectionner une autre langue pour la traduction :

1. Appuyez sur le bouton A pour entrer dans le mode de configuration.

2. Appuyez sur le bouton B pour faire défiler toutes les langues source prises en charge.

3. Appuyez sur le bouton A pour confirmer votre choix de langue source.

4. Appuyez sur le bouton B et maintenez-le enfoncé tout en parlant, puis relâchez-le pour lancer la traduction.

5. Le texte traduit en anglais s’affiche à l’écran.

![Faire défiler pour sélectionner la langue](media/iot-hub-arduino-iot-devkit-az3166-translator/select-language.jpg)

![Résultat de la traduction](media/iot-hub-arduino-iot-devkit-az3166-translator/translation-result.jpg)

Sur l’écran affichant le résultat de la traduction, vous pouvez :

- Appuyez sur les boutons A et B pour faire défiler et sélectionnez la langue source.

- Appuyez sur le bouton B pour parler. Pour envoyer le message vocal et obtenir le texte traduit, relâchez le bouton B.

## <a name="how-it-works"></a>Fonctionnement

![mini-solution-voice-to-tweet-diagram](media/iot-hub-arduino-iot-devkit-az3166-translator/diagram.png)

Le DevKit IoT enregistre votre voix, puis poste une requête HTTP pour déclencher Azure Functions. Azure Functions appelle l’API de traducteur de discours de service cognitif pour effectuer la traduction. Une fois qu’Azure Functions obtient le texte traduit, il envoie un message cloud-à-appareil à l’appareil. La traduction s’affiche ensuite sur l’écran.

## <a name="problems-and-feedback"></a>Problèmes et commentaires

Si vous rencontrez des problèmes, consultez le [FAQ sur le DevKit IoT](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) ou contactez-nous par le biais des canaux suivants :

* [Gitter.im](https://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>Étapes suivantes

Vous avez appris à utiliser le Devkit IoT comme traducteur à l’aide d’Azure Functions et de Cognitive Services. Dans ce guide pratique, vous avez appris à :

> [!div class="checklist"]
> * Utiliser une tâche Visual Studio Code pour automatiser les approvisionnements du cloud
> * Configurer la chaîne de connexion d’appareil Azure IoT
> * Déployer la fonction Azure
> * Tester la traduction d’un message vocal

Passez aux autres tutoriels pour apprendre à :

> [!div class="nextstepaction"]
> [Connecter IoT DevKit AZ3166 à l’accélérateur de solution de supervision à distance Azure IoT](./iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring.md)