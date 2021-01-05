---
title: Récupérer un message Twitter avec Azure Functions | Microsoft Docs
description: Utilisez le capteur de mouvement pour détecter les secousses et utilisez Azure Functions pour rechercher un tweet aléatoire avec un hashtag que vous spécifiez.
author: liydu
manager: jeffya
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 03/07/2018
ms.author: liydu
ms.custom: devx-track-csharp
ms.openlocfilehash: af1685f6455c0642800cba7dd604fcc836bcd7a4
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92147894"
---
# <a name="shake-shake-for-a-tweet----retrieve-a-twitter-message-with-azure-functions"></a>Secouez, secouez pour récupérer un tweet - Récupérez un message Twitter avec Azure Functions

Dans ce projet, vous allez apprendre à utiliser le capteur de mouvement pour déclencher un événement à l’aide d’Azure Functions. L’application récupère un tweet aléatoire avec un #mot-dièse que vous configurez dans votre ébauche de projet Arduino. Le tweet s’affiche sur l’écran DevKit.

## <a name="what-you-need"></a>Ce dont vous avez besoin

Suivez le [Guide de mise en route](./iot-hub-arduino-iot-devkit-az3166-get-started.md) pour :

* Connecter votre DevKit au Wi-Fi.
* Préparer l’environnement de développement.

Un abonnement Azure actif. Si vous n’en avez pas, vous pouvez vous inscrire via l’une de ces méthodes :

* Activez un [compte d’évaluation Microsoft Azure gratuit pendant 30 jours](https://azure.microsoft.com/free/).
* Réclamez votre [crédit Azure](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) si vous êtes abonné à MSDN ou Visual Studio.

## <a name="open-the-project-folder"></a>Ouvrir le dossier de projet

Commencez par ouvrir le dossier du projet. 

### <a name="start-vs-code"></a>Démarrer VS Code

* Vérifiez que votre kit de développement est connecté à votre ordinateur.

* Démarrez VS Code.

* Connectez le kit DevKit à votre ordinateur.

   > [!NOTE]
   > Quand vous lancez VS Code, vous pouvez recevoir un message d’erreur indiquant que l’IDE Arduino ou le package de la carte associée ne peut pas être trouvé. Si cette erreur se produit, fermez VS Code et lancez une nouvelle fois l’IDE Arduino. VS Code devrait maintenant localiser correctement le chemin de l’IDE Arduino.

### <a name="open-the-arduino-examples-folder"></a>Ouvrir le dossier des exemples Arduino

Développez la section **EXEMPLES ARDUINO** à gauche, accédez à **Exemples pour MXCHIP AZ3166 > AzureIoT** et sélectionnez **ShakeShake**. Une nouvelle fenêtre VS Code s’ouvre, dans laquelle apparaît le dossier de projet. Si vous ne trouvez pas la section MXCHIP AZ3166, vérifiez que votre appareil est correctement connecté et redémarrez Visual Studio Code.  
![mini-solution-examples](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/vscode_examples.png)

Vous pouvez également ouvrir l’exemple de projet à partir de la palette de commandes. Cliquez sur `Ctrl+Shift+P` (macOS : `Cmd+Shift+P`) pour ouvrir la palette de commandes, tapez **Arduino**, puis recherchez et sélectionnez **Arduino : Exemples**.

## <a name="provision-azure-services"></a>Approvisionner les services Azure

Dans la fenêtre de la solution, exécutez votre tâche par `Ctrl+P` (macOS : `Cmd+P`) en entrant `task cloud-provision`.

Dans le terminal VS Code, une ligne de commande interactive vous guide dans l’approvisionnement des services Azure nécessaires :

![Capture d’écran représentant la ligne de commande interactive du terminal Visual Studio Code.](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/cloud-provision.png)

> [!NOTE]
> Si la page se bloque dans l’état de chargement quand vous tentez de vous connecter à Azure, consultez [l’étape relative au blocage de la page de connexion du FAQ sur le DevKit IoT](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/#page-hangs-when-log-in-azure).
 
## <a name="modify-the-hashtag"></a>Modifier le #mot-dièse

Ouvrez `ShakeShake.ino` et recherchez cette ligne de code :

```cpp
static const char* iot_event = "{\"topic\":\"iot\"}";
```

Remplacez la chaîne `iot` entre des accolades par votre mot-dièse préféré. Le DevKit récupère par la suite un tweet aléatoire qui inclut le mot-dièse que vous spécifiez dans cette étape.

## <a name="deploy-azure-functions"></a>Déployer Azure Functions

Utilisez `Ctrl+P` (macOS: `Cmd+P`) pour exécuter `task cloud-deploy` et commencer à déployer le code Azure Functions :

![La capture d’écran représentant Visual Studio Code dans lequel vous pouvez exécuter task cloud-deploy pour déployer le code Azure Functions.](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/cloud-deploy.png)

> [!NOTE]
> Parfois, la fonction Azure peut ne pas fonctionner correctement. Pour résoudre ce problème quand il se produit, consultez la [section « erreur de compilation » du FAQ sur le DevKit IoT](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/#compilation-error-for-azure-function).

## <a name="build-and-upload-the-device-code"></a>Générer et charger le code de l’appareil

Ensuite, générez et chargez le code de l’appareil.

### <a name="windows"></a>Windows

1. Utilisez `Ctrl+P` pour exécuter `task device-upload`.

2. Le terminal vous invite à passer en mode de configuration. Pour ce faire :

   * Maintenez enfoncé le bouton A

   * Enfoncez et relâchez le bouton de réinitialisation.

3. L’écran affiche l’ID du kit DevKit et « Configuration ».

### <a name="macos"></a>macOS

1. Placez le DevKit en mode de configuration :

   Maintenez le bouton A enfoncé, puis appuyez sur le bouton de réinitialisation et relâchez-le. L’écran affiche « Configuration ».

2. Utilisez `Cmd+P` pour exécuter `task device-upload` afin de définir la chaîne de connexion qui est récupérée à l’étape `task cloud-provision`.

### <a name="verify-upload-and-run"></a>Vérifier, charger et exécuter

Une fois la chaîne de connexion définie, il vérifie et charge l’application, puis l’exécute. 

1. VS Code commence la vérification et le chargement de l’ébauche de projet Arduino sur votre DevKit :

   ![Capture d’écran représentant la vérification et le chargement de l’ébauche de projet Arduino dans Visual Studio Code.](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/device-upload.png)

2. Le DevKit redémarre et commence à exécuter le code.

Vous risquez d’obtenir un message d’erreur « Erreur : AZ3166 : package inconnu ». Cette erreur se produit lorsque l’index du package de la carte n’est pas actualisé correctement. Pour résoudre ce problème, consultez [l’erreur « package inconnu » dans le FAQ sur le DevKit IoT](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/#development).

## <a name="test-the-project"></a>Tester le projet

Après l’initialisation de l’application, cliquez sur le bouton A, puis relâchez-le et secouez doucement la carte DevKit. Cette action récupère un tweet aléatoire, qui contient le #mot-dièse que vous avez spécifié précédemment. Après quelques secondes, un tweet s’affiche sur votre écran DevKit :

### <a name="arduino-application-initializing"></a>Initialisation de l’application Arduino en cours...

![Arduino-application-initializing](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-1.png)

### <a name="press-a-to-shake"></a>Appuyez sur A pour secouer...

![Press-A-to-shake](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-2.png)

### <a name="ready-to-shake"></a>Prêt à secouer...

![Ready-to-shake](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-3.png)

### <a name="processing"></a>Traitement en cours...

![Traitement en cours](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-4.png)

### <a name="press-b-to-read"></a>Appuyez sur B pour lire...

![Press-B-to-read](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-5.png)

### <a name="display-a-random-tweet"></a>Afficher un tweet aléatoire...

![Display-a-random-tweet](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/result-6.png)

- Appuyez à nouveau sur le bouton A, puis secouez pour récupérer un autre tweet.
- Appuyez sur le bouton B pour parcourir le tweet.

## <a name="how-it-works"></a>Fonctionnement

![Schéma représentant un appareil mobile qui envoie un événement à Azure IoT Hub qui déclenche une application Azure Function pour demander un tweet, qui est renvoyé à l'application et transmis au concentrateur et à l'appareil mobile.](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/diagram.png)

L’ébauche de projet Arduino envoie un événement à Azure IoT Hub. Cet événement déclenche l’application Azure Functions. L’application Azure Functions contient la logique pour se connecter à l’API Twitter et récupérer un tweet. Ensuite, elle inclut dans un wrapper le texte du tweet dans un message cloud-à-appareil et le renvoie à l’appareil.

## <a name="optional-use-your-own-twitter-bearer-token"></a>Facultatif : utilisez votre propre jeton de porteur Twitter

À des fins de test, cet exemple de projet utilise un jeton de porteur Twitter préconfiguré. Toutefois, il existe une [limite de fréquence](https://dev.twitter.com/rest/reference/get/search/tweets) pour chaque compte Twitter. Si vous souhaitez utiliser votre propre jeton, procédez comme suit :

1. Accédez au [portail des développeurs Twitter](https://dev.twitter.com/) pour inscrire une nouvelle application Twitter.

2. [Récupérez la clé du client et les secrets du client](https://support.yapsody.com/hc/en-us/articles/360003291573-How-do-I-get-a-Twitter-Consumer-Key-and-Consumer-Secret-key-) de votre application.

3. Utilisez [un utilitaire](https://gearside.com/nebula/utilities/twitter-bearer-token-generator/) pour générer un jeton de porteur Twitter à partir de ces deux clés.

4. Dans le [portail Azure](https://portal.azure.com/){:target="_blank"}, accédez au **Groupe de ressources** et recherchez la fonction Azure (Type : App Service) pour votre projet « Shake, Shake ». Le nom contient toujours la chaîne « shake... ».

   ![Capture d’écran représentant un app service pour le projet dans le Portail Azure.](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/azure-function.png)

5. Mettez à jour le code de `run.csx` dans **Fonctions > shakeshake-cs** avec votre propre jeton :

   ```csharp
   string authHeader = "Bearer " + "[your own token]";
   ```
  
   ![Capture d’écran représentant le code C# de la fonction dans laquelle vous pouvez entrer votre jeton.](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/twitter-token.png)

6. Enregistrez le fichier, puis cliquez sur **Exécuter**.

## <a name="problems-and-feedback"></a>Problèmes et commentaires

Comment résoudre des problèmes ou fournir des commentaires. 

### <a name="problems"></a>Problèmes

Vous pouvez être confronté au problème suivant : l’écran affiche « No tweets » (Aucun tweet) alors que chaque étape s’est exécutée correctement. Cette condition se produit normalement la première fois que vous déployez et exécutez l’exemple, car l’application de fonction nécessite quelques secondes à une minute pour le démarrage à froid de l’application. 

Ou bien, lorsque vous exécutez le code, certains blocages entrainent un redémarrage de l’application. Lorsque cette condition se produit, l’application de l’appareil peut obtenir un délai d’expiration pour récupérer le tweet. Dans ce cas, vous pouvez essayer l’une des méthodes suivantes (ou les deux) pour résoudre le problème :

1. Cliquez sur le bouton de réinitialisation de DevKit pour exécuter à nouveau l’application de l’appareil.

2. Dans le [portail Azure](https://portal.azure.com/), recherchez l’application Azure Functions que vous avez créée et redémarrez-la :

   ![Capture d’écran représentant le Portail Azure avec une application Azure Functions et le bouton Redémarrer.](media/iot-hub-arduino-iot-devkit-az3166-retrieve-twitter-message/azure-function-restart.png)

Si vous rencontrez d’autres problèmes, consultez le [FAQ sur le DevKit IoT](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) ou contactez-nous par le biais des canaux suivants :

* [Gitter.im](https://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>Étapes suivantes

Vous avez appris à connecter un appareil DevKit à votre accélérateur de solution de supervision à distance Azure IoT et à récupérer un tweet ; nous vous suggérons à présent les étapes suivantes :

* [Vue d’ensemble des accélérateurs de solution de supervision à distance Azure IoT](/azure/iot-suite/)