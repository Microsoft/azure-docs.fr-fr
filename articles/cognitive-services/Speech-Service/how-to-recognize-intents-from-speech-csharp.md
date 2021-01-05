---
title: Guide pratique pour reconnaître des intentions vocales avec le SDK Speech C#
titleSuffix: Azure Cognitive Services
description: Dans ce guide, vous apprenez à reconnaître des intentions vocales avec le SDK Speech pour C#.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/10/2020
ms.author: trbye
ms.custom: devx-track-csharp
ms.openlocfilehash: 93a3adf00203e317be912e3e72de7a3f7ca666c6
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "96001090"
---
# <a name="how-to-recognize-intents-from-speech-using-the-speech-sdk-for-c"></a>Guide pratique pour reconnaître des intentions vocales avec le SDK Speech pour C#

Le [SDK Speech](speech-sdk.md) de Cognitive Services s’intègre au [Language Understanding Intelligent Service (LUIS)](https://www.luis.ai/home) pour fournir la **reconnaissance des intentions**. Une intention est quelque chose que l’utilisateur souhaite faire : réserver un vol, vérifier la météo ou effectuer un appel. L’utilisateur peut utiliser les termes qui lui semblent naturels. À l’aide du machine learning, LUIS mappe les demandes des utilisateurs aux intentions que vous avez définies.

> [!NOTE]
> Une application LUIS définit les intentions et entités à reconnaître. Elle est distincte de l’application C# qui utilise le service Speech. Dans cet article, « appli » fait référence à l’application LUIS tandis que « application » renvoie au code C#.

Dans ce guide, vous utilisez le SDK Speech pour développer une application console C# qui dégage des intentions à partir des énoncés utilisateur obtenus par le microphone de votre appareil. Vous découvrirez comment effectuer les actions suivantes :

> [!div class="checklist"]
>
> - Créer un projet Visual Studio faisant référence au package NuGet du kit SDK Speech
> - Créer une configuration de reconnaissance vocale et obtenir un module de reconnaissance de l’intention
> - Obtenir le modèle pour votre appli LUIS et ajouter les intentions dont vous avez besoin
> - Spécifier la langue pour la reconnaissance vocale
> - Utiliser la reconnaissance vocale à partir d’un fichier
> - Utiliser la reconnaissance continue pilotée par événements, asynchrone

## <a name="prerequisites"></a>Prérequis

Veillez à disposer des éléments suivants avant de commencer à suivre ce guide :

- Un compte LUIS. Vous pouvez en obtenir un gratuitement via le [portail LUIS](https://www.luis.ai/home).
- [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) (toute édition).

## <a name="luis-and-speech"></a>LUIS et Speech

LUIS s’intègre au service Speech pour reconnaître les intentions à partir de la reconnaissance vocale. Vous n’avez besoin que de LUIS, aucun abonnement au service Speech n’est nécessaire.

LUIS utilise trois types de clés :

| Type de clé  | Objectif                                               |
| --------- | ----------------------------------------------------- |
| Création | Permet de créer et de modifier des applications LUIS programmatiquement |
| Starter   | Permet de tester l’application LUIS en utilisant uniquement du texte   |
| Point de terminaison  | Autorise l’accès à une application LUIS particulière            |

Pour ce guide, vous avez besoin du type de clé du point de terminaison. Ce guide utilise l’exemple d’application domotique LUIS Home Automation, que vous pouvez créer en suivant le guide de démarrage rapide [Utiliser une application domotique prédéfinie](../luis/luis-get-started-create-app.md). Si vous avez créé votre propre application LUIS, vous pouvez aussi l’utiliser.

Lors de la création d’une application LUIS, une clé de démarrage est générée automatiquement par LUIS pour que vous puissiez tester l’application à l’aide de requêtes de texte. Cette clé n’active pas l’intégration du service Speech et ne fonctionne pas avec ce guide. Créez une ressource LUIS dans le tableau de bord Azure et affectez-la à l’application LUIS. Vous pouvez utiliser le niveau d’abonnement gratuit pour suivre ce guide.

Après avoir créé la ressource LUIS dans le tableau de bord Azure, connectez-vous au [portail LUIS](https://www.luis.ai/home), choisissez votre application dans la page **My apps** (Mes applications) et basculez vers la page **Manage** (Gérer) de l’application. Enfin, sélectionnez **Keys and Endpoints** (Clés et points de terminaison) dans la barre latérale.

![Paramètres des clés et point de terminaison du portail LUIS](media/sdk/luis-keys-endpoints-page.png)

Dans la page **Keys and Endpoint settings** (Paramètres des clés et du point de terminaison) :

1. Faites défiler vers le bas jusqu’à la section **Resources and Keys** (Ressources et clés) et des clés, puis sélectionnez **Assign resource** (Attribuer une ressource).
1. Dans la boîte de dialogue **Assign a key to your app** (Attribuer une clé à votre application), apportez les modifications suivantes :

   - Sous **Tenant** (Locataire), choisissez **Microsoft**.
   - Sous **Subscription Name** (Nom de l’abonnement), sélectionnez l’abonnement Azure qui contient la ressource LUIS à utiliser.
   - Sous **Key** (Clé), choisissez la ressource LUIS à utiliser avec l’application.

   Le nouvel abonnement apparaît dans le tableau, en bas de la page, au bout de quelques instants.

1. Sélectionnez l’icône en regard d’une clé pour la copier dans le Presse-papiers. (Vous pouvez utiliser l’une ou l’autre de ces clés.)

![Clés d’abonnement de l’appli LUIS](media/sdk/luis-keys-assigned.png)

## <a name="create-a-speech-project-in-visual-studio"></a>Créer un projet de reconnaissance vocale dans Visual Studio

[!INCLUDE [Create project](../../../includes/cognitive-services-speech-service-create-speech-project-vs-csharp.md)]

## <a name="add-the-code"></a>Ajouter le code

Ensuite, vous ajoutez du code au projet.

1. Dans l’**Explorateur de solutions**, ouvrez le fichier **Program.cs**.

1. Remplacez le bloc d’instructions `using` au début du fichier par les déclarations suivantes :

   [!code-csharp[Top-level declarations](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/intent_recognition_samples.cs#toplevel)]

1. Remplacez la méthode `Main()` fournie par l’équivalent asynchrone suivant :

   ```csharp
   public static async Task Main()
   {
       await RecognizeIntentAsync();
       Console.WriteLine("Please press Enter to continue.");
       Console.ReadLine();
   }
   ```

1. Créez une méthode asynchrone vide `RecognizeIntentAsync()`, comme indiqué ici :

   ```csharp
   static async Task RecognizeIntentAsync()
   {
   }
   ```

1. Dans le corps de cette nouvelle méthode, ajoutez ce code :

   [!code-csharp[Intent recognition by using a microphone](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/intent_recognition_samples.cs#intentRecognitionWithMicrophone)]

1. Remplacez les espaces réservés de cette méthode par votre clé d’abonnement, votre région et votre ID d’appli LUIS, comme suit.

   | Espace réservé | Remplacer par |
   | ----------- | ------------ |
   | `YourLanguageUnderstandingSubscriptionKey` | Votre clé de point de terminaison LUIS. Là encore, vous devez obtenir cet élément à partir de votre tableau de bord Azure, et non d’une « clé de démarrage ». Vous pouvez la trouver dans la page **Keys and Endpoints** (Clés et points de terminaison) de votre application, sous **Manage** (Gérer) dans le [portail LUIS](https://www.luis.ai/home). |
   | `YourLanguageUnderstandingServiceRegion` | L’identificateur court pour la région de votre abonnement LUIS est, par exemple, `westus` pour la région USA Ouest. Voir [Régions](regions.md). |
   | `YourLanguageUnderstandingAppId` | L’ID d’application LUIS. Vous pouvez le trouver dans la page **Settings** (Paramètres) de votre application dans le [portail LUIS](https://www.luis.ai/home). |

Ces modifications étant apportées, vous pouvez générer (**Ctrl+Maj+B**) et exécuter (**F5**) l’application. Lorsque vous y êtes invité, essayez de dire « Éteins les lumières » dans le micro de votre PC. L’application affiche le résultat dans la fenêtre de console.

Les sections suivantes comportent un développement du code.

## <a name="create-an-intent-recognizer"></a>Créer un module de reconnaissance de l’intention

Tout d’abord, vous devez créer une configuration de reconnaissance vocale à partir de la clé et de la région de votre point de terminaison LUIS. Vous pouvez utiliser les configurations de reconnaissance vocale pour créer des modules de reconnaissance pour les différentes fonctionnalités du SDK Speech. La configuration de reconnaissance vocale offre plusieurs moyens pour spécifier l’abonnement à utiliser. Ici, nous utilisons `FromSubscription`, qui accepte la région et la clé d’abonnement.

> [!NOTE]
> Utilisez la clé et la région de votre abonnement LUIS, et non celles d’un abonnement Speech.

Ensuite, créez un module de reconnaissance de l’intention à l’aide de `new IntentRecognizer(config)`. La configuration sachant déjà quel abonnement utiliser, vous n’avez pas besoin de spécifier à nouveau la clé d’abonnement et le point de terminaison lors de la création du module de reconnaissance.

## <a name="import-a-luis-model-and-add-intents"></a>Importer un modèle LUIS et ajouter des intentions

À présent, importez le modèle à partir de l’appli LUIS avec `LanguageUnderstandingModel.FromAppId()`, et ajoutez les intentions LUIS à reconnaître par le biais de la méthode `AddIntent()` du module de reconnaissance. Ces deux étapes améliorent la précision de la reconnaissance vocale en indiquant les mots que l’utilisateur est susceptible d’utiliser dans ses requêtes. Il n’est pas nécessaire d’ajouter toutes les intentions de l’application si vous n’avez pas besoin de toutes les identifier dans votre application.

Pour ajouter des intentions, vous devez fournir trois arguments : le modèle LUIS (qui a été créé et qui est nommé `model`), le nom de l’intention et un ID d’intention. La différence entre l’ID et le nom s’établit comme suit.

| `AddIntent()`&nbsp;argument | Objectif |
| --------------------------- | ------- |
| `intentName` | Nom de l’intention, tel que défini dans l’appli LUIS. Cette valeur doit correspondre exactement au nom d’intention LUIS. |
| `intentID` | ID assigné à une intention reconnue par le kit SDK Speech. Cette valeur peut être ce que vous voulez ; il n’est pas nécessaire qu’elle corresponde au nom de l’intention, tel que défini dans l’application LUIS. Par exemple, si plusieurs intentions sont gérées par le même code, vous pouvez utiliser le même ID pour celles-ci. |

L’application LUIS Home Automation a deux intentions : une pour mettre un appareil sous tension, l’autre pour le mettre hors tension. Les lignes ci-dessous ajoutent ces intentions au module de reconnaissance de l’intention ; remplacez les trois lignes `AddIntent` dans la méthode `RecognizeIntentAsync()` par ce code.

```csharp
recognizer.AddIntent(model, "HomeAutomation.TurnOff", "off");
recognizer.AddIntent(model, "HomeAutomation.TurnOn", "on");
```

Au lieu d’ajouter des intentions individuelles, vous pouvez aussi utiliser la méthode `AddAllIntents` pour ajouter toutes les intentions d’un modèle au module de reconnaissance.

## <a name="start-recognition"></a>Démarrer la reconnaissance

Le module de reconnaissance étant créé, et les intentions ajoutées, la reconnaissance peut commencer. Le kit SDK Speech prend en charge la reconnaissance ponctuelle et continue.

| Mode de reconnaissance | Méthodes à appeler | Résultats |
| ---------------- | --------------- | ------ |
| Ponctuel | `RecognizeOnceAsync()` | Retourne l’intention reconnue, si elle existe, après un énoncé. |
| Continue | `StartContinuousRecognitionAsync()`<br>`StopContinuousRecognitionAsync()` | Reconnaît plusieurs énoncés ; émet des événements (par exemple, `IntermediateResultReceived`) lorsque des résultats sont disponibles. |

L’application utilise le mode ponctuel et appelle donc `RecognizeOnceAsync()` pour débuter la reconnaissance. Le résultat est un objet `IntentRecognitionResult` contenant des informations sur l’intention reconnue. Vous extrayez la réponse JSON LUIS à l’aide de l’expression suivante :

```csharp
result.Properties.GetProperty(PropertyId.LanguageUnderstandingServiceResponse_JsonResult)
```

L’application n’analyse pas le résultat JSON. Elle affiche uniquement le texte JSON dans la fenêtre de console.

![Résultats de la reconnaissance LUIS uniques](media/sdk/luis-results.png)

## <a name="specify-recognition-language"></a>Préciser une langue de reconnaissance

Par défaut, LUIS reconnaît les intentions en anglais (États-Unis) (`en-us`). En attribuant un code de paramètres régionaux à la propriété `SpeechRecognitionLanguage` de la configuration de reconnaissance vocale, vous pouvez reconnaître des intentions dans d’autres langues. Par exemple, ajoutez `config.SpeechRecognitionLanguage = "de-de";` dans notre application avant de créer le module de reconnaissance pour reconnaître les intentions en allemand. Pour plus d’informations, consultez les [langues prises en charge par LUIS](../LUIS/luis-language-support.md#languages-supported).

## <a name="continuous-recognition-from-a-file"></a>Reconnaissance continue depuis un fichier

Le code suivant illustre deux caractéristiques supplémentaires de la reconnaissance de l’intention avec le kit SDK Speech. La première, mentionnée précédemment, est la reconnaissance continue, par laquelle le module de reconnaissance émet des événements lorsque des résultats sont disponibles. Ces événements peuvent ensuite être traités par les gestionnaires d’événements que vous fournissez. Avec la reconnaissance continue, vous appelez la méthode `StartContinuousRecognitionAsync()` du module de reconnaissance pour lancer la reconnaissance à la place de `RecognizeOnceAsync()`.

L’autre caractéristique lit l’audio contenant la reconnaissance vocale à traiter à partir d’un fichier WAV. L’implémentation implique la création d’une configuration audio utilisable lors de la création du module de reconnaissance de l’intention. Le fichier doit être de canal unique (mono) avec un taux d’échantillonnage de 16 kHz.

Pour tester ces fonctionnalités, supprimez ou mettez en commentaires le corps de la méthode `RecognizeIntentAsync()`, puis ajoutez le code suivant à la place.

[!code-csharp[Intent recognition by using events from a file](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/intent_recognition_samples.cs#intentContinuousRecognitionWithFile)]

Révisez le code pour inclure votre clé de point de terminaison, votre région et votre ID d’appli LUIS, et ajouter des intentions domotiques, comme effectué précédemment. Remplacez `whatstheweatherlike.wav` par le nom de votre fichier audio enregistré. Ensuite, générez, copiez le fichier audio dans le répertoire de build, puis exécutez l’application.

Par exemple, si vous dites « Éteins les lumières », faites une pause, puis dites « Allume les lumières » dans votre fichier audio enregistré, une sortie de console similaire à la suivante peut s’afficher :

![Résultats de la reconnaissance LUIS de fichiers audio](media/sdk/luis-results-2.png)

[!INCLUDE [Download the sample](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]
Retrouvez le code de cet article dans le dossier **samples/csharp/sharedcontent/console**.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Démarrage rapide : Reconnaître la voix à partir d’un microphone](./get-started-speech-to-text.md?pivots=programming-language-csharp&tabs=dotnetcore)