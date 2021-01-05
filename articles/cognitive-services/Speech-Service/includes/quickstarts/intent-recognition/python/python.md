---
author: trevorbye
ms.service: cognitive-services
ms.subservice: speech-service
ms.date: 04/04/2020
ms.topic: include
ms.author: trbye
zone_pivot_groups: programming-languages-set-two
ms.openlocfilehash: 44c4427540e1b94ebcaf00e6875723e0a654e9ef
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94424977"
---
## <a name="prerequisites"></a>Prérequis

Avant de commencer :

* <a href="~/articles/cognitive-services/Speech-Service/quickstarts/setup-platform.md?pivots=programming-language-python" target="_blank">Installez le SDK Speech pour votre environnement de développement, puis créez un exemple de projet vide<span class="docon docon-navigate-external x-hidden-focus"></span></a>.

## <a name="create-a-luis-app-for-intent-recognition"></a>Créer une application LUIS pour la reconnaissance de l’intention

[!INCLUDE [Create a LUIS app for intent recognition](../luis-sign-up.md)]

## <a name="open-your-project"></a>Ouvrir votre projet

1. Ouvrez l’IDE de votre choix.
2. Créez un projet ainsi qu’un fichier sous le nom `quickstart.py`, puis ouvrez-le.

## <a name="start-with-some-boilerplate-code"></a>Commencer avec du code réutilisable

Nous allons ajouter du code qui servira de squelette à notre projet

[!code-python[](~/samples-cognitive-services-speech-sdk/quickstart/python/intent-recognition/quickstart.py?range=5-7)]

## <a name="create-a-speech-configuration"></a>Créer une configuration Speech

Pour pouvoir initialiser un objet `IntentRecognizer`, vous devez au préalable créer une configuration qui utilise la clé et la localisation de votre ressource de prédiction LUIS.

Insérez ce code dans `quickstart.py`. Veillez à mettre à jour ces valeurs :

* Remplacez `"YourLanguageUnderstandingSubscriptionKey"` par votre clé de prédiction LUIS.
* Remplacez `"YourLanguageUnderstandingServiceRegion"` par votre localisation LUIS. Utilisez l’ **identificateur Région** de la [région](../../../../regions.md).

>[!TIP]
> Si vous avez besoin d’aide pour trouver ces valeurs, consultez [Créer une application LUIS pour la reconnaissance de l’intention](#create-a-luis-app-for-intent-recognition).

[!code-python[](~/samples-cognitive-services-speech-sdk/quickstart/python/intent-recognition/quickstart.py?range=12)]

Cet exemple construit l’objet `SpeechConfig` à l’aide de la clé et de la région LUIS. Pour obtenir la liste complète des méthodes disponibles, consultez la rubrique [Classe SpeechConfig](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig).

Le SDK Speech reconnaît par défaut l’utilisation de la langue en-US. Consultez [Spécifier la langue source pour la reconnaissance vocale](../../../../how-to-specify-source-language.md) pour plus d’informations sur le choix de la langue source.

## <a name="initialize-an-intentrecognizer"></a>Initialiser IntentRecognizer

Créons maintenant un `IntentRecognizer`. Insérez ce code juste en dessous de votre configuration Speech.

[!code-python[](~/samples-cognitive-services-speech-sdk/quickstart/python/intent-recognition/quickstart.py?range=15)]

## <a name="add-a-languageunderstandingmodel-and-intents"></a>Ajouter un LanguageUnderstandingModel et des intentions

Vous devez associer un `LanguageUnderstandingModel` au module de reconnaissance de l’intention et ajouter les intentions que vous souhaitez reconnaître. Nous allons utiliser les intentions du domaine prédéfini pour la domotique.

Insérez ce code en dessous de votre `IntentRecognizer`. Veillez à remplacer `"YourLanguageUnderstandingAppId"` par votre ID d’application LUIS. 

>[!TIP]
> Si vous avez besoin d’aide pour trouver cette valeur, consultez [Créer une application LUIS pour la reconnaissance de l’intention](#create-a-luis-app-for-intent-recognition).

[!code-python[](~/samples-cognitive-services-speech-sdk/quickstart/python/intent-recognition/quickstart.py?range=19-27)]

Cet exemple utilise la fonction `add_intents()` pour ajouter une liste d’intentions définies explicitement. Si vous souhaitez ajouter toutes les intentions d’un modèle, utilisez `add_all_intents(model)` et transmettez le modèle.

## <a name="recognize-an-intent"></a>Reconnaître une intention

À partir de l’objet `IntentRecognizer`, vous devez appeler la méthode `recognize_once()`. Cette méthode permet au service Speech de savoir que vous envoyez une seule expression pour reconnaissance, et d’arrêter la reconnaissance une fois que l’expression a été identifiée.

Insérez ce code sous votre modèle.

[!code-python[](~/samples-cognitive-services-speech-sdk/quickstart/python/intent-recognition/quickstart.py?range=35)]

## <a name="display-the-recognition-results-or-errors"></a>Afficher les résultats de la reconnaissance (ou les erreurs)

Lorsque le résultat de la reconnaissance est retourné par le service de reconnaissance vocale, vous pouvez effectuer une opération avec celui-ci. Nous allons faire simple et imprimer le résultat dans la console.

Sous votre appel à `recognize_once()`, ajoutez ce code.

[!code-python[](~/samples-cognitive-services-speech-sdk/quickstart/python/intent-recognition/quickstart.py?range=38-47)]

## <a name="check-your-code"></a>Vérifier votre code

À ce stade, votre code doit ressembler à ceci.

> [!NOTE]
> Nous avons ajouté des commentaires à cette version.

[!code-python[](~/samples-cognitive-services-speech-sdk/quickstart/python/intent-recognition/quickstart.py?range=5-47)]

## <a name="build-and-run-your-app"></a>Générer et exécuter votre application

Exécutez l’exemple à partir de la console ou dans votre IDE :

```
python quickstart.py
```

Les 15 secondes suivantes de saisie vocale provenant de votre microphone seront reconnues et enregistrées dans la fenêtre console.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [footer](./footer.md)]