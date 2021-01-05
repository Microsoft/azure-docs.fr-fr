---
author: trevorbye
ms.service: cognitive-services
ms.subservice: speech-service
ms.date: 04/04/2020
ms.topic: include
ms.author: trbye
ms.custom: devx-track-csharp
zone_pivot_groups: programming-languages-set-two
ms.openlocfilehash: c242bbb803e359664eb2fb63262f0c540723f9c6
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94424937"
---
## <a name="prerequisites"></a>Prérequis

Avant de commencer :

* <a href="~/articles/cognitive-services/Speech-Service/quickstarts/setup-platform.md?tabs=dotnet&pivots=programming-language-csharp" target="_blank">Installez le SDK Speech pour votre environnement de développement, puis créez un exemple de projet vide<span class="docon docon-navigate-external x-hidden-focus"></span></a>.

## <a name="create-a-luis-app-for-intent-recognition"></a>Créer une application LUIS pour la reconnaissance de l’intention

[!INCLUDE [Create a LUIS app for intent recognition](../luis-sign-up.md)]

## <a name="open-your-project-in-visual-studio"></a>Ouvrez votre projet dans Visual Studio.

Ensuite, ouvrez votre projet dans Visual Studio.

1. Lancez Visual Studio 2019.
2. Chargez votre projet et ouvrez `Program.cs`.

## <a name="start-with-some-boilerplate-code"></a>Commencer avec du code réutilisable

Nous allons ajouter du code qui servira de squelette à notre projet Notez que vous avez créé une méthode asynchrone appelée `RecognizeIntentAsync()`.
[!code-csharp[](~/samples-cognitive-services-speech-sdk/quickstart/csharp/dotnet/intent-recognition/helloworld/Program.cs?range=7-17,77-86)]

## <a name="create-a-speech-configuration"></a>Créer une configuration Speech

Pour pouvoir initialiser un objet `IntentRecognizer`, vous devez au préalable créer une configuration qui utilise la clé et la localisation de votre ressource de prédiction LUIS.

> [!IMPORTANT]
> Votre clé de démarrage et vos clés de création ne fonctionneront pas. Vous devez utiliser la clé de prédiction et la localisation que vous avez créés précédemment. Pour plus d’informations, consultez [Créer une application LUIS pour la reconnaissance de l’intention](#create-a-luis-app-for-intent-recognition).

Insérez ce code dans la méthode `RecognizeIntentAsync()`. Veillez à mettre à jour ces valeurs :

* Remplacez `"YourLanguageUnderstandingSubscriptionKey"` par votre clé de prédiction LUIS.
* Remplacez `"YourLanguageUnderstandingServiceRegion"` par votre localisation LUIS. Utilisez l’ **identificateur Région** de la [région](../../../../regions.md).

>[!TIP]
> Si vous avez besoin d’aide pour trouver ces valeurs, consultez [Créer une application LUIS pour la reconnaissance de l’intention](#create-a-luis-app-for-intent-recognition).

[!code-csharp[](~/samples-cognitive-services-speech-sdk/quickstart/csharp/dotnet/intent-recognition/helloworld/Program.cs?range=26)]

Cet exemple utilise la méthode `FromSubscription()` pour générer la `SpeechConfig`. Pour obtenir la liste complète des méthodes disponibles, consultez la rubrique [Classe SpeechConfig](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig?preserve-view=true&view=azure-dotnet).

Le SDK Speech reconnaît par défaut l’utilisation de la langue en-US. Consultez [Spécifier la langue source pour la reconnaissance vocale](../../../../how-to-specify-source-language.md) pour plus d’informations sur le choix de la langue source.

## <a name="initialize-an-intentrecognizer"></a>Initialiser IntentRecognizer

Créons maintenant un `IntentRecognizer`. Cet objet est créé à l’intérieur d’une instruction using pour garantir la bonne libération des ressources non managées. Insérez ce code dans la méthode `RecognizeIntentAsync()`, juste en dessous de votre configuration Speech.

[!code-csharp[](~/samples-cognitive-services-speech-sdk/quickstart/csharp/dotnet/intent-recognition/helloworld/Program.cs?range=29-30,76)]

## <a name="add-a-languageunderstandingmodel-and-intents"></a>Ajouter un LanguageUnderstandingModel et des intentions

Vous devez associer un `LanguageUnderstandingModel` au module de reconnaissance de l’intention et ajouter les intentions que vous souhaitez reconnaître. Nous allons utiliser les intentions du domaine prédéfini pour la domotique. Insérez ce code dans l’instruction using de la section précédente. Veillez à remplacer `"YourLanguageUnderstandingAppId"` par votre ID d’application LUIS.

>[!TIP]
> Si vous avez besoin d’aide pour trouver cette valeur, consultez [Créer une application LUIS pour la reconnaissance de l’intention](#create-a-luis-app-for-intent-recognition).

[!code-csharp[](~/samples-cognitive-services-speech-sdk/quickstart/csharp/dotnet/intent-recognition/helloworld/Program.cs?range=33-35)]

Cet exemple utilise la fonction `AddIntent()` pour ajouter des intentions individuellement. Si vous souhaitez ajouter toutes les intentions d’un modèle, utilisez `AddAllIntents(model)` et transmettez le modèle. 

## <a name="recognize-an-intent"></a>Reconnaître une intention

À partir de l’objet `IntentRecognizer`, vous devez appeler la méthode `RecognizeOnceAsync()`. Cette méthode permet au service Speech de savoir que vous envoyez une seule expression pour reconnaissance, et d’arrêter la reconnaissance une fois que l’expression a été identifiée.

À l’intérieur de l’instruction using, ajoutez ce code sous votre modèle.

[!code-csharp[](~/samples-cognitive-services-speech-sdk/quickstart/csharp/dotnet/intent-recognition/helloworld/Program.cs?range=46)]

## <a name="display-recognition-results-or-errors"></a>Afficher les résultats de la reconnaissance (ou les erreurs)

Lorsque le résultat de la reconnaissance est retourné par le service de reconnaissance vocale, vous pouvez effectuer une opération avec celui-ci. Nous allons faire simple et imprimer les résultats dans la console.

À l’intérieur de l’instruction using, sous `RecognizeOnceAsync()`, ajoutez ce code :

[!code-csharp[](~/samples-cognitive-services-speech-sdk/quickstart/csharp/dotnet/intent-recognition/helloworld/Program.cs?range=49-75)]

## <a name="check-your-code"></a>Vérifier votre code

À ce stade, votre code doit ressembler à ceci :

> [!NOTE]
> Nous avons ajouté des commentaires à cette version.

[!code-csharp[](~/samples-cognitive-services-speech-sdk/quickstart/csharp/dotnet/intent-recognition/helloworld/Program.cs?range=7-86)]

## <a name="build-and-run-your-app"></a>Générer et exécuter votre application

Vous êtes maintenant prêt à créer votre application et à tester la reconnaissance vocale à l’aide du service de reconnaissance vocale.

1. **Compiler le code** : à partir de la barre de menus de Visual Studio, choisissez **Générer** > **Générer la solution**.
2. **Démarrer votre application**  : dans la barre de menus, choisissez **Déboguer** > **Démarrer le débogage** , ou appuyez sur <kbd>F5</kbd>.
3. **Démarrer la reconnaissance**  : vous êtes invité à prononcer une phrase. Celle-ci est envoyée au service de reconnaissance vocale, transcrite sous forme de texte, puis affichée sur la console.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [footer](./footer.md)]