---
title: 'Démarrage rapide : Traduire la voix en voix, C# (.NET Framework Windows) – Service Speech'
titleSuffix: Azure Cognitive Services
description: TBD
services: cognitive-services
author: wolfma61
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 04/04/2020
ms.author: erhopf
ms.openlocfilehash: be20a6e4b6c89bf9d371cdc8620441f066249b94
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/14/2020
ms.locfileid: "88226188"
---
## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous de :

> [!div class="checklist"]
> * [Créer une ressource Azure Speech](../../../../get-started.md)
> * [Configurer votre environnement de développement et créer un projet vide](../../../../quickstarts/setup-platform.md?tabs=dotnet&pivots=programming-language-csharp)

## <a name="add-sample-code"></a>Ajouter un exemple de code

1. Ouvrez **Program.cs** et remplacez tout le code existant par le suivant.

   ```csharp
   using System;
   using System.Threading.Tasks;
   using Microsoft.CognitiveServices.Speech;
   using Microsoft.CognitiveServices.Speech.Translation;

   namespace helloworld
   {
       class Program
       {
           public static async Task TranslateSpeechToSpeech()
           {
               // Creates an instance of a speech translation config with specified subscription key and service region.
               // Replace with your own subscription key and region identifier from here: https://aka.ms/speech/sdkregion
               var config = SpeechTranslationConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");

               // Sets source and target languages.
               // Replace with the languages of your choice, from list found here: https://aka.ms/speech/sttt-languages
               string fromLanguage = "en-US";
               string toLanguage = "de";
               config.SpeechRecognitionLanguage = fromLanguage;
               config.AddTargetLanguage(toLanguage);

               // Sets the synthesis output voice name.
               // Replace with the languages of your choice, from list found here: https://aka.ms/speech/tts-languages
               config.VoiceName = "de-DE-Hedda";

               // Creates a translation recognizer using the default microphone audio input device.
               using (var recognizer = new TranslationRecognizer(config))
               {
                   // Prepare to handle the synthesized audio data.
                   recognizer.Synthesizing += (s, e) =>
                   {
                       var audio = e.Result.GetAudio();
                       Console.WriteLine(audio.Length != 0
                           ? $"AUDIO SYNTHESIZED: {audio.Length} byte(s)"
                           : $"AUDIO SYNTHESIZED: {audio.Length} byte(s) (COMPLETE)");
                   };

                   // Starts translation, and returns after a single utterance is recognized. The end of a
                   // single utterance is determined by listening for silence at the end or until a maximum of 15
                   // seconds of audio is processed. The task returns the recognized text as well as the translation.
                   // Note: Since RecognizeOnceAsync() returns only a single utterance, it is suitable only for single
                   // shot recognition like command or query.
                   // For long-running multi-utterance recognition, use StartContinuousRecognitionAsync() instead.
                   Console.WriteLine("Say something...");
                   var result = await recognizer.RecognizeOnceAsync();

                   // Checks result.
                   if (result.Reason == ResultReason.TranslatedSpeech)
                   {
                       Console.WriteLine($"RECOGNIZED '{fromLanguage}': {result.Text}");
                       Console.WriteLine($"TRANSLATED into '{toLanguage}': {result.Translations[toLanguage]}");
                   }
                   else if (result.Reason == ResultReason.RecognizedSpeech)
                   {
                       Console.WriteLine($"RECOGNIZED '{fromLanguage}': {result.Text} (text could not be translated)");
                   }
                   else if (result.Reason == ResultReason.NoMatch)
                   {
                       Console.WriteLine($"NOMATCH: Speech could not be recognized.");
                   }
                   else if (result.Reason == ResultReason.Canceled)
                   {
                       var cancellation = CancellationDetails.FromResult(result);
                       Console.WriteLine($"CANCELED: Reason={cancellation.Reason}");

                       if (cancellation.Reason == CancellationReason.Error)
                       {
                           Console.WriteLine($"CANCELED: ErrorCode={cancellation.ErrorCode}");
                           Console.WriteLine($"CANCELED: ErrorDetails={cancellation.ErrorDetails}");
                           Console.WriteLine($"CANCELED: Did you update the subscription info?");
                       }
                   }
               }
           }

           static void Main(string[] args)
           {
               TranslateSpeechToSpeech().Wait();
           }
       }
   }
   ```

1. Dans le même fichier, remplacez la chaîne `YourSubscriptionKey` par votre clé d’abonnement.

1. Remplacez la chaîne `YourServiceRegion` par la [région](~/articles/cognitive-services/Speech-Service/regions.md) associée à votre abonnement.

1. Dans la barre de menus, choisissez **Fichier** > **Enregistrer tout**.

## <a name="build-and-run-the-application"></a>Génération et exécution de l’application

1. Dans la barre de menus, sélectionnez **Générer** > **Générer la solution** pour générer l’application. Le code doit maintenant se compiler sans erreurs.

1. Choisissez **Déboguer** > **Démarrer le débogage** (ou appuyez sur **F5**) pour démarrer l’application **helloworld**.

1. Prononcez une phrase ou quelques mots en anglais. L’application transmet vos paroles au service Speech, qui les traduit et transcrit en texte (dans ce cas, en allemand). Le service Speech renvoie ensuite l’audio synthétisé et le texte à l’application pour que celle-ci l’affiche.

````
Say something...
AUDIO SYNTHESIZED: 76784 byte(s)
AUDIO SYNTHESIZED: 0 byte(s)(COMPLETE)
RECOGNIZED 'en-US': What's the weather in Seattle?
TRANSLATED into 'de': Wie ist das Wetter in Seattle?
````

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [footer](./footer.md)]
