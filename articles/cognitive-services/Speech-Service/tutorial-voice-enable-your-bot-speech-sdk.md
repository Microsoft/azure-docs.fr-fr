---
title: 'Tutoriel : Activer les fonctions vocales dans votre bot à l’aide du SDK Speech - Service Speech'
titleSuffix: Azure Cognitive Services
description: Dans ce tutoriel, vous allez créer un bot d’écho à l’aide de Microsoft Bot Framework, le déployer dans Azure, puis l’inscrire auprès du canal Bot Framework Direct Line Speech. Ensuite, vous allez configurer un exemple d’application cliente pour Windows qui vous permet de parler à votre bot et d’entendre ses réponses vocales.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/25/2020
ms.author: trbye
ms.custom: devx-track-csharp
ms.openlocfilehash: 6b231b5af208a51cbe8f4370c2dc532be1ba3af9
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024923"
---
# <a name="tutorial-voice-enable-your-bot-using-the-speech-sdk"></a>Tutoriel : Activer les fonctions vocales dans votre bot à l’aide du SDK Speech

Vous pouvez désormais utiliser le service Speech pour activer les fonctions vocales dans un bot conversationnel.

Dans ce tutoriel, vous allez créer un bot qui répète ce que vous lui dites.
Vous allez créer un bot d’écho à l’aide de Microsoft Bot Framework, le déployer dans Azure, puis l’inscrire auprès du canal Bot Framework Direct Line Speech.
Ensuite, vous allez configurer un exemple d’application cliente pour Windows qui vous permet de parler à votre bot et d’entendre ses réponses vocales.

Ce tutoriel est destiné aux développeurs qui débutent avec Azure, les bots Bot Framework, Direct Line Speech ou le SDK Speech, et qui souhaitent créer rapidement un système opérationnel avec peu de code. Aucune connaissance ni expérience de ces services ne sont nécessaires.

Le bot de conversation vocal que vous faites dans ce tutoriel suit les étapes suivantes :

1. L’exemple d’application cliente est configuré pour se connecter au canal Direct Line Speech et au bot d’écho.
1. Lorsque l'utilisateur appuie sur un bouton, le son de la voix est transmis par le microphone. (Ou l’audio est enregistré en continu lorsqu’un mot clé personnalisé est utilisé.)
1. Si un mot clé personnalisé est utilisé, la détection de mot clé se produit sur l’appareil local, en conservant la diffusion audio en continu vers le Cloud.
1. À l’aide du SDK Speech, l’application se connecte au canal Direct Line Speech et diffuse l’audio en continu.
1. Si vous le souhaitez, il est possible de configurer avec plus de précision la vérification du mot clé dans le service.
1. L’audio est transmis au service de reconnaissance vocale, puis il est retranscrit en texte.
1. Le texte reconnu est passé au bot d’écho en tant qu’activité Bot Framework.
1. Le texte de la réponse est converti en audio par le service TTS (Text-to-Speech), puis il diffusé en continu vers l’application cliente afin d’être lu.

<!-- svg src in User Story 1754106 -->
![diagram-tag](media/tutorial-voice-enable-your-bot-speech-sdk/diagram.png "Flux du canal Speech")

> [!NOTE]
> Les étapes de ce tutoriel ne nécessitent pas l’utilisation d’un service payant. Pour suivre ce tutoriel, en tant que nouvel utilisateur Azure, vous pouvez utiliser les crédits de votre essai gratuit Azure, ainsi que le niveau de service gratuit du service Speech.

Voici ce qui est couvert par ce tutoriel :
> [!div class="checklist"]
> * Créer des ressources Azure
> * Générer, tester et déployer l’exemple de bot d’écho dans Azure App Service
> * Inscrire votre bot auprès d’un canal Direct Line Speech
> * Générer et exécuter le client Assistant vocal Windows pour interagir avec votre Echo Bot
> * Ajouter l’activation du mot clé personnalisée
> * Découvrir comment modifier la langue du texte reconnu et prononcé

## <a name="prerequisites"></a>Prérequis

Voici ce dont vous aurez besoin pour suivre ce tutoriel :

- Un poste de travail Windows 10 avec un micro et des haut-parleurs (ou des écouteurs) qui fonctionnent
- [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/)ou version supérieure, avec la **charge de travail Développement ASP.NET et web**
- [.NET Framework Runtime 4.6.1](https://dotnet.microsoft.com/download) ou version ultérieure
- Un compte Azure. [Inscrivez-vous gratuitement](https://azure.microsoft.com/free/cognitive-services/)
- Un compte [GitHub](https://github.com/)
- [Git pour Windows](https://git-scm.com/download/win)

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

L’application cliente que vous allez créer dans ce tutoriel utilise un petit nombre de services Azure. Pour réduire la durée d’aller-retour des réponses envoyées par votre bot, vous devez placer ces services dans la même région Azure. Dans cette section, vous allez créer un groupe de ressources dans la région **USA Ouest**. Ce groupe de ressources sera utilisé lors de la création de ressources pour Bot Framework, le canal Direct Line Speech et le service Speech.

1. <a href="https://ms.portal.azure.com/#create/Microsoft.ResourceGroup" target="_blank">Créer un groupe de ressources <span class="docon docon-navigate-external x-hidden-focus"></span></a>
1. Vous serez invité à fournir certaines informations :
   * Configurez **Abonnement** sur **Essai gratuit** (vous pouvez également utiliser un abonnement existant).
   * Entrez le nom de votre **Groupe de ressources**. Nous recommandons d’utiliser le nom suivant : **SpeechEchoBotTutorial-ResourceGroup**.
   * Dans la liste déroulante **Région**, sélectionnez **USA Ouest**.
1. Cliquez sur **Vérifier et créer**. Vous devez voir une bannière indiquant : **Validation réussie**.
1. Cliquez sur **Créer**. La création du groupe de ressources peut prendre quelques minutes.
1. Comme pour les ressources que vous allez créer plus loin dans ce tutoriel, il est recommandé d’épingler ce groupe de ressources à votre tableau de bord afin d’y accéder plus facilement. Si vous souhaitez épingler ce groupe de ressources, cliquez sur l’icône d’épingle à droite du nom du groupe de ressources.

### <a name="choosing-an-azure-region"></a>Sélection d’une région Azure

Si vous souhaitez utiliser une autre région pour ce tutoriel, les facteurs suivants peuvent limiter vos choix :

* Veillez à utiliser une [région Azure prise en charge](regions.md#voice-assistants).
* Le canal Direct Line Speech utilise le service de conversion de texte par synthèse vocale, qui comprend les options de voix standard et neurale. L’option de voix neurale peut être [limitée à certaines régions Azure](regions.md#standard-and-neural-voices).

Pour plus d’informations sur les régions, consultez [Régions Azure](https://azure.microsoft.com/global-infrastructure/locations/).

## <a name="create-resources"></a>Créer des ressources

Maintenant que vous disposez d’un groupe de ressources dans une région prise en charge, l’étape suivante consiste à créer des ressources pour chaque service que vous allez utiliser dans ce tutoriel.

### <a name="create-a-speech-service-resource"></a>Créer une ressource de service Speech

Pour créer une ressource Speech, suivez ces instructions :

1. <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices" target="_blank">Créer une ressource de service de reconnaissance vocale <span class="docon docon-navigate-external x-hidden-focus"></span></a>
4. Vous serez invité à fournir certaines informations :
   * Donnez un **Nom** à votre ressource. Nous recommandons d’utiliser le nom suivant : **SpeechEchoBotTutorial-Speech**.
   * Pour **Abonnement**, vérifiez que l’option **Essai gratuit** est bien sélectionnée.
   * Pour **Emplacement**, sélectionnez **USA Ouest**.
   * Pour **Niveau tarifaire**, sélectionnez **F0**. Il s’agit du niveau de service gratuit.
   * Pour **Groupe de ressources**, sélectionnez **SpeechEchoBotTutorial-ResourceGroup**.
5. Après avoir entré toutes les informations nécessaires, cliquez sur **Créer**. La création de la ressource peut prendre plusieurs minutes.
6. Plus loin dans ce tutoriel, vous aurez besoin d’une clé d’abonnement pour ce service. Vous pouvez accéder à ces clés à tout moment à partir de la page **Vue d’ensemble** (Gérer les clés) ou **Clés** de votre ressource.

À ce stade, vérifiez que votre groupe de ressources (**SpeechEchoBotTutorial-ResourceGroup**) comprend une ressource Speech :

| Nom | Type  | Emplacement |
|------|-------|----------|
| SpeechEchoBotTutorial-Speech | Cognitive Services | USA Ouest |

### <a name="create-an-azure-app-service-plan"></a>Créer un plan Azure App Service

L’étape suivante consiste à créer un plan App Service. Un plan App Service définit un ensemble de ressources de calcul nécessaires à l’exécution d’une application web.

1. <a href="https://ms.portal.azure.com/#create/Microsoft.AppServicePlanCreate" target="_blank">Créer un plan Azure App Service <span class="docon docon-navigate-external x-hidden-focus"></span></a>
4. Vous serez invité à fournir certaines informations :
   * Configurez **Abonnement** sur **Essai gratuit** (vous pouvez également utiliser un abonnement existant).
   * Pour **Groupe de ressources**, sélectionnez **SpeechEchoBotTutorial-ResourceGroup**.
   * Donnez un **Nom** à votre ressource. Nous recommandons d’utiliser le nom suivant : **SpeechEchoBotTutorial-AppServicePlan**.
   * Pour **Système d’exploitation**, sélectionnez **Windows**.
   * Pour **Région**, sélectionnez **USA Ouest**.
   * Pour **Niveau tarifaire**, vérifiez que l’option **Standard S1** est bien sélectionnée. Il doit s’agir de la valeur par défaut. Si ce n’est pas le cas, veillez à définir le **système d’exploitation** sur **Windows** comme décrit ci-dessus.
5. Cliquez sur **Vérifier et créer**. Vous devez voir une bannière indiquant : **Validation réussie**.
6. Cliquez sur **Créer**. La création du groupe de ressources peut prendre quelques minutes.

À ce stade, vérifiez que votre groupe de ressources (**SpeechEchoBotTutorial-ResourceGroup**) comprend les deux ressources suivantes :

| Nom | Type  | Emplacement |
|------|-------|----------|
| SpeechEchoBotTutorial-AppServicePlan | Plan App Service | USA Ouest |
| SpeechEchoBotTutorial-Speech | Cognitive Services | USA Ouest |

## <a name="build-an-echo-bot"></a>Créer un bot d’écho

Maintenant que vous avez créé des ressources, vous allez créer un bot. Vous allez commencer par l’exemple de bot d’écho, qui, comme son nom l’indique, renvoie par écho le texte que vous avez entré comme réponse. L’exemple de code est prêt à être utilisé et ne nécessite aucune modification de votre part. Il est configuré pour fonctionner avec le canal Direct Line Speech, auquel nous nous connecterons après avoir déployé le bot dans Azure.

> [!NOTE]
> Les instructions qui suivent, ainsi que les informations supplémentaires sur le bot d’écho, sont disponibles dans le [fichier README de l’exemple sur GitHub](https://github.com/microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/02.echo-bot/README.md).

### <a name="run-the-bot-sample-on-your-machine"></a>Exécuter l’exemple de bot sur votre machine

1. Clonez le référentiel de l’exemple :

   ```bash
   git clone https://github.com/Microsoft/botbuilder-samples.git
   ```

2. Lancez Visual Studio.
3. À partir de la barre d’outils, sélectionnez **Fichier** > **Ouvrir** > **Projet/Solution**, puis ouvrez la solution du projet Echo Bot :

   ```
   samples\csharp_dotnetcore\02.echo-bot\EchoBot.sln
   ```

4. Une fois le projet chargé, appuyez sur <kbd>F5</kbd> pour le générer et l’exécuter.
5. Un navigateur doit se lancer et un écran similaire à celui-ci s’affiche.
    > [!div class="mx-imgBorder"]
    > [![Capture d’écran montrant la page EchoBot affichant le message Votre bot est prêt !](media/tutorial-voice-enable-your-bot-speech-sdk/echobot-running-on-localhost.png "EchoBot s’exécutant sur localhost")](media/tutorial-voice-enable-your-bot-speech-sdk/echobot-running-on-localhost.png#lightbox)

### <a name="test-the-bot-sample-with-the-bot-framework-emulator"></a>Tester l’exemple de bot avec Bot Framework Emulator

[Bot Framework Emulator](https://github.com/microsoft/botframework-emulator) est une application de bureau qui permet aux développeurs de bots de tester et de déboguer leurs bots localement ou à distance, via un tunnel. L’émulateur prend en charge le texte tapé en tant qu’entrée (pas la voix). Le bot répond également avec le texte. Suivez ces étapes pour utiliser Bot Framework Emulator pour tester votre Echo Bot exécuté localement, avec une entrée texte et une sortie texte. Une fois le bot déployé sur Azure, nous le testerons avec une entrée vocale et une sortie vocale.

1. Installez [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases/latest) version 4.3.0 ou ultérieure.
2. Lancez Bot Framework Emulator, puis ouvrez votre bot :
   * **Fichier** -> **Ouvrir le bot**.
3. Entrez l’URL de votre bot. Par exemple :

   ```
   http://localhost:3978/api/messages
   ```
   Puis appuyez sur « Se connecter ».
4. Le bot est censé vous accueillir avec un message « Bonjour et bienvenue ! » . Tapez un message texte et vérifiez que vous recevez une réponse de la part du bot.
5. Voici à quoi pourrait ressembler un échange de communication avec une instance Echo Bot :  [![Capture d’écran montrant le Bot Framework Emulator.](media/tutorial-voice-enable-your-bot-speech-sdk/bot-framework-emulator.png "Bot Framework Emulator")](media/tutorial-voice-enable-your-bot-speech-sdk/bot-framework-emulator.png#lightbox)

## <a name="deploy-your-bot-to-an-azure-app-service"></a>Déployer votre bot dans Azure App Service

L’étape suivante consiste à déployer le bot d’écho dans Azure. Il existe plusieurs façons de déployer un bot. Cela dit, dans ce tutoriel, nous allons nous concentrer sur la méthode de publication qui est effectuée directement à partir de Visual Studio.

> [!NOTE]
> Vous pouvez également déployer un bot à l’aide d’[Azure CLI](/azure/bot-service/bot-builder-deploy-az-cli) et de [modèles de déploiement](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/adaptive-dialog/03.core-bot).

> [!NOTE]
> Si **Publier...** n’apparaît pas au fur et à mesure que vous effectuez les étapes suivantes, utilisez Visual Studio Installer pour ajouter la charge de travail **ASP.NET et développement web**.

1. Dans Visual Studio, ouvrez le bot d’écho qui a été configuré pour être utilisé avec le canal Direct Line Speech :

   ```
   samples\csharp_dotnetcore\02.echo-bot\EchoBot.sln
   ```

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet **EchoBot**, puis sélectionnez **Publier...** .
1. Une nouvelle fenêtre intitulée **Publier** s’ouvre.
1. Sélectionnez **Azure**, cliquez sur **Suivant**, sélectionnez **Azure App Service (Windows)** , cliquez sur **Suivant**, puis cliquez sur **Créer un Azure App Service...** par le signe plus vert.
1. Lorsque la fenêtre **App Service (Windows)** s’affiche :
   * Cliquez sur **Ajouter un compte**, puis connectez-vous avec les informations d’identification de votre compte Azure. Si vous êtes déjà connecté, sélectionnez le compte que vous souhaitez dans la liste déroulante.
   * Dans **Nom**, vous devez entrer un nom global unique pour votre bot. Ce nom sera utilisé pour créer l’URL unique du bot. Une valeur par défaut comprenant la date et l’heure (par exemple, « EchoBot20190805125647 ») sera fournie. Pour ce tutoriel, vous pouvez utiliser le nom par défaut.
   * Pour **Abonnement**, sélectionnez **Essai gratuit**.
   * Pour **Groupe de ressources**, sélectionnez **SpeechEchoBotTutorial-ResourceGroup**.
   * Pour **Plan d’hébergement**, sélectionnez **SpeechEchoBotTutorial-AppServicePlan**.
1. Cliquez sur **Créer**. Dans l’écran final de l’Assistant, cliquez sur **Terminer**.
1. Cliquez sur **Publier** sur le côté droit de l’écran Publish. Visual Studio déploie le bot sur Azure.
1. Un message de réussite similaire à celui-ci doit s’afficher dans la fenêtre de sortie Visual Studio :

   ```
   Publish Succeeded.
   Web App was published successfully https://EchoBot20190805125647.azurewebsites.net/
   ```

1. Votre navigateur par défaut doit s’ouvrir sur une page indiquant ceci : « Votre bot est prêt ! ».
1. Maintenant, dans le portail Azure, vérifiez que les trois ressources suivantes se trouvent dans votre groupe de ressources **SpeechEchoBotTutorial-ResourceGroup** :

| Nom | Type  | Emplacement |
|------|-------|----------|
| EchoBot20190805125647 | App Service | USA Ouest |
| SpeechEchoBotTutorial-AppServicePlan | Plan App Service | USA Ouest |
| SpeechEchoBotTutorial-Speech | Cognitive Services | USA Ouest |

## <a name="enable-web-sockets"></a>Activer les sockets web

Vous devez apporter une petite modification à la configuration afin que votre bot puisse communiquer avec le canal Direct Line Speech à l’aide de WebSockets. Effectuez les étapes suivantes pour activer les WebSockets :

1. Accédez au [Portail Azure](https://portal.azure.com), puis cliquez sur votre App Service. La ressource doit porter un nom similaire à **EchoBot20190805125647** (le nom unique de votre application).
2. Dans le volet de navigation de gauche, sous **Paramètres**, cliquez sur **Configuration**.
3. Sélectionnez l’onglet **Paramètres généraux**.
4. Localisez le bouton bascule des **Web Sockets** et sélectionnez la valeur **Activé**.
5. Cliquez sur **Enregistrer**.

> [!TIP]
> Vous pouvez utiliser les contrôles situés en haut de la page Azure App Service pour arrêter ou redémarrer le service. Cela peut être utile lors de la résolution des problèmes.

## <a name="create-a-channel-registration"></a>Créer une inscription de canal

Maintenant que vous avez créé une instance Azure App Service pour héberger votre bot, vous devez créer une inscription de canal de bot (**Bot Channels Registration** ). La création d’une inscription de canal est un prérequis à l’inscription de votre bot auprès de canaux Bot Framework, notamment auprès du canal Direct Line Speech. Si vous souhaitez en savoir plus sur la façon dont les bots utilisent les canaux, consultez [Connecter un bot à des canaux](/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0).

1. <a href="https://ms.portal.azure.com/#create/Microsoft.BotServiceConnectivityGalleryPackage" target="_blank">Créer une inscription de chaînes de bot Azure <span class="docon docon-navigate-external x-hidden-focus"></span></a>
2. Vous serez invité à fournir certaines informations :
   * Pour **Descripteur du bot**, entrez **SpeechEchoBotTutorial-BotRegistration-####** et remplacez **####** par un nombre de votre choix. Notez que le descripteur du bot doit être globalement unique. Si vous entrez un descripteur de bot mais recevez le message d’erreur _L’ID de bot demandé n’est pas disponible_, choisissez un autre nombre. Dans les exemples ci-dessous, nous avons utilisé 8726
   * Pour **Abonnement**, sélectionnez **Essai gratuit**.
   * Pour **Groupe de ressources**, sélectionnez **SpeechEchoBotTutorial-ResourceGroup**.
   * Pour **Emplacement**, sélectionnez **USA Ouest**.
     * Pour **Niveau tarifaire**, sélectionnez **F0**.
     * Pour **Point de terminaison de messagerie**, entrez l’URL de votre application web, à la fin de laquelle vous ajoutez le chemin `/api/messages`. Par exemple, si votre nom d’application global unique est **EchoBot20190805125647**, votre point de terminaison de messagerie sera : `https://EchoBot20190805125647.azurewebsites.net/api/messages/`.
     * Pour **Application Insights**, vous pouvez sélectionner la valeur **Désactivé**. Pour plus d’informations, consultez [Analytique des bots](/azure/bot-service/bot-service-manage-analytics?view=azure-bot-service-4.0).
     * Ignorez **Création automatique de l’ID d’application et du mot de passe**.
5. Dans le bas du panneau **Inscription de chaînes de bot**, cliquez sur **Canaux**.

À ce stade, accédez à votre groupe de ressources **SpeechEchoBotTutorial-ResourceGroup** dans le portail Azure. Il doit maintenant afficher quatre ressources :

| Nom | Type  | Emplacement |
|------|-------|----------|
| EchoBot20190805125647 | App Service | USA Ouest |
| SpeechEchoBotTutorial-AppServicePlan | Plan App Service | USA Ouest |
| SpeechEchoBotTutorial-BotRegistration-8726 | Bot Channels Registration | Global |
| SpeechEchoBotTutorial-Speech | Cognitive Services | USA Ouest |

> [!IMPORTANT]
> La ressource Bot Channels Registration affiche la région Globale alors que vous avez sélectionné USA Ouest. Ceci est normal.

## <a name="optional-test-in-web-chat"></a>Facultatif : Tester dans une discussion web

La page Azure d’inscription de chaînes de bot contient une option **Tester dans une discussion web** sous **Gestion des bots**. Elle ne fonctionnera pas par défaut avec votre bot, car la discussion web doit s’authentifier auprès de votre bot. Si vous souhaitez tester votre bot déployé avec l’entrée de texte, suivez les étapes ci-dessous. Notez que ces étapes sont facultatives et ne sont pas requises pour passer aux étapes suivantes de ce tutoriel. 

1. Recherchez et ouvrez votre ressource **EchoBotTutorial-BotRegistration-####** dans le [portail Azure](https://portal.azure.com)
1. Dans le volet de navigation **Gestion des bots**, sélectionnez **Paramètres**. Copiez la valeur figurant sous **ID d’application Microsoft**
1. Ouvrez la solution Visual Studio EchoBot. Dans l’Explorateur de solutions, recherchez et double-cliquez sur **appsettings.json**
1. Remplacez la chaîne vide en regard de **MicrosoftAppId** dans le fichier JSON par la valeur d’ID copiée
1. Revenez au portail Azure, dans le volet de navigation **Gestion des bots**, sélectionnez **Paramètres**, puis cliquez sur **(Gérer)** en regard de **ID d’application Microsoft**
1. Cliquez sur **Nouveau secret client**. Ajoutez une description (par exemple, « discussion web »), puis cliquez sur **Ajouter**. Copiez le nouveau secret
1. Remplacez la chaîne vide en regard de **MicrosoftAppPassword** dans le fichier JSON par la valeur de secret copiée
1. Enregistrez le fichier JSON. Il doit se présenter comme suit :
```json
{
  "MicrosoftAppId": "3be0abc2-ca07-475e-b6c3-90c4476c4370",
  "MicrosoftAppPassword": "-zRhJZ~1cnc7ZIlj4Qozs_eKN.8Cq~U38G"
}
```
9. Republiez l’application (cliquez avec le bouton droit sur le projet **EchoBot** dans l’Explorateur de solutions Visual Studio, sélectionnez **Publier...** , puis cliquez sur le bouton **Publier**)
10. Vous êtes maintenant prêt à tester le bot dans une discussion web.

## <a name="register-the-direct-line-speech-channel"></a>Inscrire le canal Direct Line Speech

Il est temps d’inscrire votre bot auprès du canal Direct Line Speech. Ce canal est utilisé pour créer une connexion entre votre bot et une application cliente compilée avec le SDK Speech.

1. Recherchez et ouvrez votre ressource **SpeechEchoBotTutorial-BotRegistration-####** dans le [portail Azure](https://portal.azure.com).
1. Dans le volet de navigation **Gestion des bots**, sélectionnez **Chaînes**.
   * Recherchez **More channels** (Autres canaux), puis cliquez sur **Direct Line Speech**.
   * Passez en revue le texte de la page intitulée **Configurer Direct line Speech**, puis développez le menu déroulant intitulé **Compte Cognitive Services**.
   * Sélectionnez la ressource Speech que vous avez créée précédemment (par exemple, **SpeechEchoBotTutorial-Speech**) dans le menu pour associer votre bot à votre clé d’abonnement Speech.
   * Ignorez les autres champs facultatifs
   * Cliquez sur **Enregistrer**.

1. Dans le volet de navigation **Gestion des bots**, cliquez sur **Paramètres**.
   * Cochez la case **Enable Streaming Endpoint** (Activer le point de terminaison de streaming). Cela permet de créer un protocole de communication reposant sur les WebSockets entre votre bot et le canal Direct Line Speech.
   * Cliquez sur **Enregistrer**.

> [!TIP]
> Pour plus d’informations, consultez [Connecter un bot à Direct Line Speech](/azure/bot-service/bot-service-channel-connect-directlinespeech?view=azure-bot-service-4.0). Cette page contient des informations supplémentaires et mentionne les problèmes connus.

## <a name="run-the-windows-voice-assistant-client"></a>Exécuter le client Assistant vocal Windows

Dans le cadre de cette étape, vous allez exécuter le client Assistant vocal Windows. Le client est une application Windows Presentation Foundation (WPF) en langage C# qui utilise le [SDK Speech](./speech-sdk.md) pour gérer les communications avec votre bot à l’aide du canal Direct Line Speech. Utilisez-le pour interagir avec votre bot et pour le tester avant d’écrire une application cliente personnalisée. Il est open source et vous pouvez donc télécharger l'exécutable et l’exécuter, ou le générer vous-même.

Le client Assistant vocal Windows offre une interface utilisateur simple qui vous permet de configurer la connexion à votre bot, d’afficher la conversation textuelle, d’afficher les activités Bot Framework au format JSON et d’afficher les cartes adaptatives. Il prend également en charge l’utilisation de mots clés personnalisés. Vous utiliserez ce client pour parler avec votre bot et recevoir ses réponses vocales.

> [!NOTE]
> Avant de continuer, vérifiez que votre micro et vos haut-parleurs sont activés et qu’ils fonctionnent.

1. Accédez au dépôt GitHub du [client Assistant vocal Windows](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/blob/master/clients/csharp-wpf/README.md).
1. Suivez les instructions fournies pour
   * Télécharger un fichier exécutable prédéfini dans un package ZIP à exécuter, ou
   * générer l'exécutable vous-même, en clonant le référentiel et en générant le projet.

1. Lancez l’application cliente `VoiceAssistantClient.exe` et configurez-la pour qu’elle se connecte à votre bot, en fonction des instructions fournies dans le dépôt GitHub
1. Cliquez sur **Reconnexion** et vérifiez que vous voyez le message **Nouvelle conversation démarrée, ou appuyez sur le bouton microphone**.
1. Nous allons tester cette fonctionnalité. Cliquez sur le bouton du micro et prononcez quelques mots en anglais. Le texte reconnu s’affichera à mesure que vous parlez. Lorsque vous avez fini de parler, le bot répond de sa propre voix, en disant « écho », puis les mots qu’il a reconnus.
1. Vous pouvez également utiliser du texte pour communiquer avec le bot. Il vous suffit pour cela de taper du texte dans la barre inférieure. 

### <a name="troubleshooting-errors-in-windows-voice-assistant-client"></a>Résolution d’erreurs dans le client Assistant vocal Windows

Si un message d’erreur s’affiche dans la fenêtre principale de votre application, utilisez le tableau suivant pour identifier et résoudre le problème :

| Error | Que faire ? |
|-------|----------------------|
|Error (AuthenticationFailure) : WebSocket Upgrade failed with an authentication error (401). Check for correct subscription key (or authorization token) and region name| Dans la page des paramètres de l’application, vérifiez que vous avez entré correctement la clé d’abonnement Speech ainsi que sa région.<br>Vérifiez que votre clé Speech et sa région ont été entrées correctement. |
|Error (ConnectionFailure) : Connection was closed by the remote host. Code d’erreur : 1011. Détails de l’erreur : We could not connect to the bot before sending a message | Vérifiez que vous avez bien [coché la case « Enable Streaming Endpoint »](#register-the-direct-line-speech-channel) et/ou que les [**Web Sockets**](#enable-web-sockets) sont activés.<br>Vérifiez qu’Azure App Service est en cours d’exécution. Si c’est le cas, essayez de le redémarrer.|
|Error (ConnectionFailure) : Connection was closed by the remote host. Code d’erreur : 1002. Détails de l’erreur : Le serveur a retourné le code d’état ’503’ alors que le code d’état attendu était ’101’. | Vérifiez que vous avez bien [coché la case « Enable Streaming Endpoint »](#register-the-direct-line-speech-channel) et/ou que les [**Web Sockets**](#enable-web-sockets) sont activés.<br>Vérifiez qu’Azure App Service est en cours d’exécution. Si c’est le cas, essayez de le redémarrer.|
|Error (ConnectionFailure) : Connection was closed by the remote host. Code d’erreur : 1011. Détails de l’erreur : Response status code does not indicate success: 500 (InternalServerError)| Votre bot a spécifié une voix neurale dans le champ [Speak](https://github.com/microsoft/botframework-sdk/blob/master/specs/botframework-activity/botframework-activity.md#speak) de l’activité de sortie, mais la région Azure qui est associée à votre clé d’abonnement Speech ne prend pas en charge les voix neurales. Voir [Voix standard et neurales](./regions.md#standard-and-neural-voices).|

Si vous n’avez pas trouvé la solution à votre problème dans le tableau, consultez [Assistants vocaux : questions fréquentes](faq-voice-assistants.md). Si la procédure décrite dans ce didacticiel ne vous a pas permis de résoudre votre problème, entrez un nouveau problème dans la [page GitHub Voice Assistant](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/issues).

#### <a name="a-note-on-connection-time-out"></a>Note relative à l’expiration du délai de connexion

Si vous êtes connecté à un bot et qu’aucune activité ne s’est produite au cours des 5 dernières minutes, le service ferme automatiquement la connexion WebSocket avec le client et le bot. C'est la procédure normale. Un message s’affiche dans la barre inférieure : *« Active connection timed out but ready to reconnect on demand »* (La connexion active a expiré mais est prête à se reconnecter à la demande). Vous n’avez pas besoin d’appuyer sur le bouton « Reconnecter ». Il vous suffit d’appuyer sur le bouton du microphone et de commencer à parler, de taper un message, ou de prononcer le mot clé (si un mot clé est activé). La connexion est automatiquement rétablie.  
### <a name="view-bot-activities"></a>Afficher les activités du bot

Chaque bot envoie et reçoit des messages liés aux **activités**. Dans la fenêtre **Journal d’activité** du client Assistant vocal Windows, vous verrez des journaux horodatés pour chaque activité que le client reçoit du bot. Vous voyez également les activités que le client a envoyées au bot à l’aide de la méthode [`DialogServiceConnector.SendActivityAsync`](/dotnet/api/microsoft.cognitiveservices.speech.dialog.dialogserviceconnector.sendactivityasync). Lorsque vous sélectionnez un élément du journal, les détails de l’activité associée sont affichés au format JSON.

Voici un exemple de code JSON provenant d’une activité reçue par le client :

```json
{
    "attachments":[],
    "channelData":{
        "conversationalAiData":{
             "requestInfo":{
                 "interactionId":"8d5cb416-73c3-476b-95fd-9358cbfaebfa",
                 "version":"0.2"
             }
         }
    },
    "channelId":"directlinespeech",
    "conversation":{
        "id":"129ebffe-772b-47f0-9812-7c5bfd4aca79",
        "isGroup":false
    },
    "entities":[],
    "from":{
        "id":"SpeechEchoBotTutorial-BotRegistration-8726"
    },
    "id":"89841b4d-46ce-42de-9960-4fe4070c70cc",
    "inputHint":"acceptingInput",
    "recipient":{
        "id":"129ebffe-772b-47f0-9812-7c5bfd4aca79|0000"
    },
    "replyToId":"67c823b4-4c7a-4828-9d6e-0b84fd052869",
    "serviceUrl":"urn:botframework:websocket:directlinespeech",
    "speak":"<speak version='1.0' xmlns='https://www.w3.org/2001/10/synthesis' xml:lang='en-US'><voice name='Microsoft Server Speech Text to Speech Voice (en-US, AriaRUS)'>Echo: Hello and welcome.</voice></speak>",
    "text":"Echo: Hello and welcome.",
    "timestamp":"2019-07-19T20:03:51.1939097Z",
    "type":"message"
}
```

Pour plus d’informations sur la sortie JSON, consultez [Champs de l’activité](https://github.com/microsoft/botframework-sdk/blob/master/specs/botframework-activity/botframework-activity.md). Pour ce tutoriel, vous pouvez vous concentrer sur les champs [Text](https://github.com/microsoft/botframework-sdk/blob/master/specs/botframework-activity/botframework-activity.md#text) et [Speak](https://github.com/microsoft/botframework-sdk/blob/master/specs/botframework-activity/botframework-activity.md#speak).

### <a name="view-client-source-code-for-calls-to-the-speech-sdk"></a>Afficher le code source client pour les appels au SDK Speech

Le client Assistant vocal Windows utilise le package NuGet [Microsoft.CognitiveServices.Speech](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech/) qui contient le Kit de développement logiciel (SDK) Speech. Pour passer en revue l’exemple de code, nous vous conseillons de commencer par la méthode InitSpeechConnector() dans le fichier [`VoiceAssistantClient\MainWindow.xaml.cs`](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/blob/master/clients/csharp-wpf/VoiceAssistantClient/MainWindow.xaml.cs), qui crée les deux objets suivants dans le SDK Speech :
- [`DialogServiceConfig`](/dotnet/api/microsoft.cognitiveservices.speech.dialog.dialogserviceconfig) - Pour les paramètres de configuration (par exemple, clé d’abonnement Speech, région de la clé)
- [`DialogServiceConnector`](/dotnet/api/microsoft.cognitiveservices.speech.dialog.dialogserviceconnector.-ctor) - Pour gérer les événements de connexion au canal et d’abonnement client dans le cadre de la gestion des mots reconnus et des réponses du bot.

## <a name="add-custom-keyword-activation"></a>Ajouter l’activation du mot clé personnalisée

Le SDK Speech prend en charge l’activation du mot clé personnalisé. Comme avec « Hey Cortana » pour l’Assistant Microsoft, vous pouvez écrire une application qui écoute en permanence dans le but de détecter le mot clé de votre choix. Le mot clé peut comporter un ou plusieurs mots.

> [!NOTE]
> Les termes *phrase de sortie de veille* et *mot clé* sont interchangeables. Vous pouvez donc rencontrer les deux dans la documentation Microsoft.

La détection du mot clé est effectuée dans l’application cliente. Si vous utilisez un mot clé, l’audio ne sera diffusé en continu vers le canal Direct Line Speech que si le mot clé est détecté. Le canal Direct Line Speech comprend un composant appelé *Key Word Verification (KWV)* (Vérification du mot clé), qui effectue un traitement plus complexe dans le cloud afin de vérifier que le mot clé que vous avez choisi se trouve au début du flux audio. Si la vérification du mot clé aboutit à un résultat positif, le canal communiquera avec le bot.

Procédez comme suit pour créer un modèle de mot clé, configurer le client Assistant vocal Windows de sorte qu’il utilise ce modèle, puis pour le tester avec votre bot.

1. Suivez ces instructions pour [créer un mot clé personnalisé à l’aide du Speech Services](./custom-keyword-basics.md).
2. Décompressez le fichier de modèle que vous avez téléchargé à l’étape précédente. Son nom doit se composer de votre mot clé. Le fichier que vous recherchez doit se nommer `kws.table`.
3. Dans le client Assistant vocal Windows, accédez au menu **Paramètres**. Celui-ci est accessible via l’icône en forme de roue dentée qui est située dans l’angle supérieur droit. Dans **Model file path** (Chemin du fichier de modèle), entrez le nom complet du chemin du fichier `kws.table` de l’étape 2.
4. Veillez à cocher la case **Enabled** (Activé). Le message suivant doit s’afficher à côté de la case à cocher : « Will listen for the keyword upon next connection ». (Écoutera pour détecter le mot clé à la prochaine connexion) Si vous avez fourni le mauvais fichier ou un chemin non valide, un message d’erreur doit s’afficher.
5. Entrez la **clé d’abonnement** Speech et la **région de la clé d’abonnement**, puis cliquez sur **OK** pour fermer le menu des **paramètres**.
6. Cliquez sur **Se reconnecter**. Le message suivant doit s’afficher : « New conversation started - type, press the microphone button, or say the keyword » (Une nouvelle conversation a démarré : tapez du texte, appuyez sur le bouton du micro ou prononcez le mot clé). L’application écoute désormais de façon continue.
7. Prononcez n’importe quelle phrase précédée de votre mot clé. Par exemple: « **{mot clé**}, what time is it? ». Il n’est pas nécessaire de faire une pause après avoir prononcé le mot clé. Lorsque vous avez terminé, deux événements se produisent :
   * Vous verrez la transcription de ce que vous venez de dire.
   * Vous entendrez tout de suite après la réponse du bot.
8. Continuez de tester les trois types d’entrées pris en charge par le bot :
   * Taper du texte dans la barre inférieure
   * Appuyer sur l’icône du micro et parler
   * Prononcer une phrase précédée de votre mot clé

### <a name="view-the-source-code-that-enables-keyword"></a>Afficher le code source qui active le mot clé

Dans le code source du client Assistant vocal Windows, examinez les fichiers suivants pour passer en revue le code qui est utilisé pour activer la détection du mot clé :

1. [`VoiceAssistantClient\Models.cs`](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/blob/master/clients/csharp-wpf/VoiceAssistantClient/Models.cs) comprend un appel à la méthode [`KeywordRecognitionModel.fromFile()`](/javascript/api/microsoft-cognitiveservices-speech-sdk/keywordrecognitionmodel?preserve-view=true&view=azure-node-latest#fromfile-string-)du SDK Speech, qui est utilisé pour instancier le modèle à partir d’un fichier local situé sur le disque.
1. [`VoiceAssistantClient\MainWindow.xaml.cs`](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/blob/master/clients/csharp-wpf/VoiceAssistantClient/MainWindow.xaml.cs) comprend un appel à la méthode [`DialogServiceConnector.StartKeywordRecognitionAsync()`](/dotnet/api/microsoft.cognitiveservices.speech.dialog.dialogserviceconnector.startkeywordrecognitionasync)du SDK Speech, qui active la détection continue du mot clé.

## <a name="optional-change-the-language-and-bot-voice"></a>(Facultatif) Changer la langue et la voix du bot

Le bot que vous avez créé est à l’écoute et répond en anglais, avec par défaut une voix de synthèse vocale en anglais US. Vous n’êtes cependant pas limité à l’utilisation de l’anglais ni à une voix par défaut. Dans cette section, vous allez découvrir comment changer la langue utilisée par votre bot pour écouter et répondre. Vous allez aussi découvrir comment sélectionner une autre voix pour cette langue.

### <a name="change-the-language"></a>Changer la langue

Vous pouvez choisir une des langues mentionnées dans le tableau [reconnaissance vocale](language-support.md#speech-to-text). Dans l’exemple ci-dessous, nous allons changer la langue et choisir l’allemand.

1. Ouvrez l’application client Assistant vocal Windows, cliquez sur le bouton des paramètres (l’icône en forme de roue dentée en haut à droite), puis entrez `de-de` dans le champ Langue (il s’agit de la valeur de Paramètres régionaux mentionnée dans le tableau [reconnaissance vocale](language-support.md#speech-to-text)). Cela définit la langue qui doit être reconnue, et remplace donc la valeur par défaut (`en-us`). Cela indique également au canal Direct Line Speech d’utiliser une voix allemande par défaut pour la réponse du bot.
2. Fermez la page des paramètres, puis cliquez sur le bouton Reconnecter pour établir une nouvelle connexion à votre bot d’écho.
3. Cliquez sur le bouton du microphone et prononcez une phrase en allemand. Vous verrez le texte reconnu et le bot d’écho répondant à la voix allemande par défaut.

### <a name="change-the-default-bot-voice"></a>Changer la voix par défaut du bot

La sélection de la voix de synthèse vocale et le contrôle de la prononciation sont possibles si le bot spécifie la réponse sous forme de [SSML (Speech Synthesis Markup Language)](speech-synthesis-markup.md) au lieu d’un texte simple. Le bot d’écho n’utilise pas SSML, mais nous pouvons facilement modifier le code pour que ce soit le cas. Dans l’exemple ci-dessous, nous ajoutons SSML à la réponse du bot d’écho, de sorte que la voix allemande de Stephan Apollo (une voix d’homme) sera utilisée à la place de la voix de femme par défaut. Consultez la liste des [voix standard](language-support.md#standard-voices) et des [voix neuronales](language-support.md#neural-voices) prises en charge pour votre langue.

1. Commençons par ouvrir `samples\csharp_dotnetcore\02.echo-bot\echo-bot.cs`.
2. Recherchez ces deux lignes :
    ```csharp
    var replyText = $"Echo: {turnContext.Activity.Text}";
    await turnContext.SendActivityAsync(MessageFactory.Text(replyText, replyText), cancellationToken);
    ```
3. Remplacez-les par :
    ```csharp
    var replyText = $"Echo: {turnContext.Activity.Text}";
    var replySpeak = @"<speak version='1.0' xmlns='https://www.w3.org/2001/10/synthesis' xml:lang='de-DE'>
                    <voice name='Microsoft Server Speech Text to Speech Voice (de-DE, Stefan, Apollo)'>" +
                    $"{replyText}" + "</voice></speak>";
    await turnContext.SendActivityAsync(MessageFactory.Text(replyText, replySpeak), cancellationToken);
    ```
4. Générez votre solution dans Visual Studio et corrigez les éventuelles erreurs de build.

Le deuxième argument de la méthode « MessageFactory.Text » définit le [champ speak d’activité](https://github.com/Microsoft/botframework-sdk/blob/master/specs/botframework-activity/botframework-activity.md#speak) dans la réponse du bot. Avec la modification ci-dessus, il a été remplacé et changé de texte simple en SSML afin de spécifier une voix allemande non définie par défaut.

### <a name="redeploy-your-bot"></a>Redéployer le bot

Maintenant que vous avez apporté les modifications nécessaires au bot, l’étape suivante consiste à le republier dans Azure App Service et à le tester :

1. Dans la fenêtre de l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **EchoBot**, puis sélectionnez **Publier**.
2. Votre configuration de déploiement précédente a déjà été chargée comme configuration par défaut. Cliquez sur **Publier** à côté de **EchoBot20190805125647 - Web Deploy**.
3. Le message **Publication réussie** s’affiche dans la fenêtre sortie de Visual Studio, et une page web s’ouvre avec le message « Votre bot est prêt ! ».
4. Ouvrez l’application client Assistant vocal Windows, cliquez sur le bouton des paramètres (l’icône en forme de roue dentée en haut à droite), puis entrez `de-de` dans le champ Langue.
5. Suivez les instructions fournies dans [Exécuter le client Assistant vocal Windows](#run-the-windows-voice-assistant-client) pour vous reconnecter au bot nouvellement déployé, prononcez des mots dans la nouvelle langue et attendez la réponse du bot dans la langue et la voix définies.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne souhaitez pas continuer à utiliser le bot d’écho déployé dans ce tutoriel, vous pouvez le supprimer, ainsi que toutes les ressources Azure associées, en supprimant le groupe de ressources Azure **SpeechEchoBotTutorial-ResourceGroup**.

1. Dans le [portail Azure](https://portal.azure.com), cliquez sur **Groupes de ressources** dans le volet de navigation **Services Azure**.
2. Recherchez le groupe de ressources nommé **SpeechEchoBotTutorial-ResourceGroup**. Cliquez sur les points de suspension (...).
3. Sélectionnez **Supprimer le groupe de ressources**.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer votre propre application cliente avec le SDK Speech](./quickstarts/voice-assistants.md?pivots=programming-language-csharp)

## <a name="see-also"></a>Voir aussi

* Effectuer un déploiement dans une [région Azure près de chez vous](https://azure.microsoft.com/global-infrastructure/locations/) pour voir l’amélioration du temps de réponse des bots
* Effectuer un déploiement dans une [région Azure qui prend en charge les voix de synthèse vocale neurales haute qualité](./regions.md#standard-and-neural-voices)
* Tarifs associés au canal Direct Line Speech :
  * [Tarification de Service Bot](https://azure.microsoft.com/pricing/details/bot-service/)
  * [Service Speech](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/)
* Création et déploiement de votre propre bot avec fonctions vocales :
  * Créez un [bot Bot Framework](https://dev.botframework.com/). Inscrivez le bot auprès du [canal Direct Line Speech](/azure/bot-service/bot-service-channel-connect-directlinespeech?view=azure-bot-service-4.0) et [personnalisez-le avec les fonctions vocales](/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0).
  * Explorez les [solutions Bot Framework](https://microsoft.github.io/botframework-solutions/index) existantes : Créer un [assistant virtuel](https://microsoft.github.io/botframework-solutions/overview/virtual-assistant-solution/) et [l’étendre à Direct Line Speech](https://microsoft.github.io/botframework-solutions/clients-and-channels/tutorials/enable-speech/1-intro/)