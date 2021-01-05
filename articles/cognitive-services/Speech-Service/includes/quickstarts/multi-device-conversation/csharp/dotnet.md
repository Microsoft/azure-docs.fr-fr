---
services: cognitive-services
author: ralphe
manager: cpoulain
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 04/04/2020
ms.author: ralphe
ms.custom: devx-track-csharp
ms.openlocfilehash: a8aa2fc0b5ba76a92a0aa0234635dadee5a00f28
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91377268"
---
## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous de :

> [!div class="checklist"]
> * [Créer une ressource Azure Speech](../../../../overview.md#try-the-speech-service-for-free)
> * [Configurer votre environnement de développement et créer un projet vide](../../../../quickstarts/setup-platform.md?tabs=dotnet&pivots=programming-language-csharp)

## <a name="add-sample-code"></a>Ajouter un exemple de code

1. Ouvrez **Program.cs** et remplacez l’intégralité du code existant par le code suivant :

    ```csharp
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Transcription;
    using System;
    using System.Threading.Tasks;
    
    namespace HelloWorld
    {
        class Program
        {
            static async Task Main(string[] args)
            {
                await CreateConversationAsync();
            }
    
            static async Task CreateConversationAsync()
            {
                // Replace these values with the details of your Cognitive Speech subscription
                string subscriptionKey = "YourSubscriptionKey";

                // Replace below with your region identifier from here: https://aka.ms/speech/sdkregion
                string region = "YourServiceRegion";
    
                // Sets source and target languages.
                // Replace with the languages of your choice, from list found here: https://aka.ms/speech/sttt-languages
                string fromLanguage = "en-US";
                string toLanguage = "de";
    
                // Set this to the display name you want for the conversation host
                string displayName = "The host";
    
                // Create the task completion source that will be used to wait until the user presses Ctrl + C
                var completionSource = new TaskCompletionSource<bool>();
    
                // Register to listen for Ctrl + C
                Console.CancelKeyPress += (s, e) =>
                {
                    completionSource.TrySetResult(true);
                    e.Cancel = true; // don't terminate the current process
                };
    
                // Create an instance of the speech translation config
                var config = SpeechTranslationConfig.FromSubscription(subscriptionKey, region);
                config.SpeechRecognitionLanguage = fromLanguage;
                config.AddTargetLanguage(toLanguage);
    
                // Create the conversation
                using (var conversation = await Conversation.CreateConversationAsync(config).ConfigureAwait(false))
                {
                    // Start the conversation so the host user and others can join
                    await conversation.StartConversationAsync().ConfigureAwait(false);
    
                    // Get the conversation ID. It will be up to your scenario to determine how this is shared with other participants.
                    string conversationId = conversation.ConversationId;
                    Console.WriteLine($"Created '{conversationId}' conversation");
    
                    // At this point, you can use the conversation object to manage the conversation. 
                    // For example, to mute everyone else in the room you can call this method:
                    await conversation.MuteAllParticipantsAsync().ConfigureAwait(false);
    
                    // Configure which audio source you want to use. If you are using a text only language, you 
                    // can use the other overload of the constructor that takes no arguments
                    var audioConfig = AudioConfig.FromDefaultMicrophoneInput();
                    using (var conversationTranslator = new ConversationTranslator(audioConfig))
                    {
                        // You should connect all the event handlers you need at this point
                        conversationTranslator.SessionStarted += (s, e) =>
                        {
                            Console.WriteLine($"Session started: {e.SessionId}");
                        };
                        conversationTranslator.SessionStopped += (s, e) =>
                        {
                            Console.WriteLine($"Session stopped: {e.SessionId}");
                        };
                        conversationTranslator.Canceled += (s, e) =>
                        {
                            switch (e.Reason)
                            {
                                case CancellationReason.EndOfStream:
                                    Console.WriteLine($"End of audio reached");
                                    break;
    
                                case CancellationReason.Error:
                                    Console.WriteLine($"Canceled due to error. {e.ErrorCode}: {e.ErrorDetails}");
                                    break;
                            }
                        };
                        conversationTranslator.ConversationExpiration += (s, e) =>
                        {
                            Console.WriteLine($"Conversation will expire in {e.ExpirationTime.TotalMinutes} minutes");
                        };
                        conversationTranslator.ParticipantsChanged += (s, e) =>
                        {
                            Console.Write("The following participant(s) have ");
                            switch (e.Reason)
                            {
                                case ParticipantChangedReason.JoinedConversation:
                                    Console.Write("joined");
                                    break;
    
                                case ParticipantChangedReason.LeftConversation:
                                    Console.Write("left");
                                    break;
    
                                case ParticipantChangedReason.Updated:
                                    Console.Write("been updated");
                                    break;
                            }
    
                            Console.WriteLine(":");
    
                            foreach (var participant in e.Participants)
                            {
                                Console.WriteLine($"\t{participant.DisplayName}");
                            }
                        };
                        conversationTranslator.TextMessageReceived += (s, e) =>
                        {
                            Console.WriteLine($"Received an instant message from '{e.Result.ParticipantId}': '{e.Result.Text}'");
                            foreach (var entry in e.Result.Translations)
                            {
                                Console.WriteLine($"\tTranslated into '{entry.Key}': '{entry.Value}'");
                            }
                        };
                        conversationTranslator.Transcribed += (s, e) =>
                        {
                            Console.WriteLine($"Received a transcription from '{e.Result.ParticipantId}': '{e.Result.Text}'");
                            foreach (var entry in e.Result.Translations)
                            {
                                Console.WriteLine($"\tTranslated into '{entry.Key}': '{entry.Value}'");
                            }
                        };
                        conversationTranslator.Transcribing += (s, e) =>
                        {
                            Console.WriteLine($"Received a partial transcription from '{e.Result.ParticipantId}': '{e.Result.Text}'");
                            foreach (var entry in e.Result.Translations)
                            {
                                Console.WriteLine($"\tTranslated into '{entry.Key}': '{entry.Value}'");
                            }
                        };
    
                        // Enter the conversation to start receiving events
                        await conversationTranslator.JoinConversationAsync(conversation, displayName).ConfigureAwait(false);
    
                        // You can now send an instant message to all other participants in the room
                        await conversationTranslator.SendTextMessageAsync("The instant message to send").ConfigureAwait(false);
    
                        // If specified a speech-to-text language, you can start capturing audio
                        await conversationTranslator.StartTranscribingAsync().ConfigureAwait(false);
                        Console.WriteLine("Started transcribing. Press Ctrl + c to stop");
    
                        // At this point, you should start receiving transcriptions for what you are saying using the default microphone. Press Ctrl+c to stop audio capture
                        await completionSource.Task.ConfigureAwait(false);
    
                        // Stop audio capture
                        await conversationTranslator.StopTranscribingAsync().ConfigureAwait(false);
    
                        // Leave the conversation. After this you will no longer receive events
                        await conversationTranslator.LeaveConversationAsync().ConfigureAwait(false);
                    }
    
                    // End the conversation
                    await conversation.EndConversationAsync().ConfigureAwait(false);
    
                    // Delete the conversation. Any other participants that are still in the conversation will be removed
                    await conversation.DeleteConversationAsync().ConfigureAwait(false);
                }
            }
        }
    }
    ```

1. Dans le même fichier, remplacez la chaîne `YourSubscriptionKey` par votre clé d’abonnement Cognitive Speech.

1. Remplacez la chaîne `YourServiceRegion` par la [région](~/articles/cognitive-services/Speech-Service/regions.md) associée à votre abonnement.

1. Dans la barre de menus, choisissez **Fichier** > **Enregistrer tout**.

## <a name="build-and-run-the-application-to-create-a-new-conversation"></a>Générer et exécuter l’application pour créer une conversation

1. Dans la barre de menus, sélectionnez **Générer** > **Générer la solution** pour générer l’application. Le code doit maintenant se compiler sans erreurs.

1. Choisissez **Déboguer** > **Démarrer le débogage** (ou appuyez sur **F5**) pour démarrer l’application **helloworld**.

1. Lorsque vous voyez s’afficher le message `Started transcribing`, vous pouvez commencer à parler. Vous verrez que les transcriptions s’affichent à mesure que vous parlez.
    - Si vous partagez le code de conversation avec les autres et qu’ils rejoignent la conversation, vous verrez également leurs transcriptions.

1. Une fois que vous avez fini de parler, appuyez sur <kbd>Ctrl + C</kbd> pour arrêter la capture audio et terminer la conversation.

## <a name="build-and-run-the-application-to-join-an-existing-conversation"></a>Générer et exécuter l’application pour rejoindre une conversation existante

1. Copiez et collez la fonction suivante dans le fichier **Program.cs** :

    ```csharp
    static async Task JoinConversationAsync(string conversationId)
    {
        // Set this to the display name you want for the participant
        string displayName = "participant";

        // Set the speech to text, or text language you want to use
        string language = "en-US";

        // Create the task completion source that will be used to wait until the user presses Ctrl + c
        var completionSource = new TaskCompletionSource<bool>();

        // Register to listen for Ctrl+C
        Console.CancelKeyPress += (s, e) =>
        {
            completionSource.TrySetResult(true);
            e.Cancel = true; // don't terminate the current process
        };

        // As a participant, you don't need to specify any subscription key, or region. You can directly create
        // the conversation translator object
        var audioConfig = AudioConfig.FromDefaultMicrophoneInput();
        using (var conversationTranslator = new ConversationTranslator(audioConfig))
        {
            // Register for any events you are interested here. For now let's just register for
            // transcription, and instant message events
            conversationTranslator.TextMessageReceived += (s, e) =>
            {
                Console.WriteLine($"Received an instant message from '{e.Result.ParticipantId}': '{e.Result.Text}'");
                foreach (var entry in e.Result.Translations)
                {
                    Console.WriteLine($"\tTranslated into '{entry.Key}': '{entry.Value}'");
                }
            };
            conversationTranslator.Transcribed += (s, e) =>
            {
                Console.WriteLine($"Received a transcription from '{e.Result.ParticipantId}': '{e.Result.Text}'");
                foreach (var entry in e.Result.Translations)
                {
                    Console.WriteLine($"\tTranslated into '{entry.Key}': '{entry.Value}'");
                }
            };
            conversationTranslator.Transcribing += (s, e) =>
            {
                Console.WriteLine($"Received a partial transcription from '{e.Result.ParticipantId}': '{e.Result.Text}'");
                foreach (var entry in e.Result.Translations)
                {
                    Console.WriteLine($"\tTranslated into '{entry.Key}': '{entry.Value}'");
                }
            };
    
            // To start receiving events, you will need to join the conversation
            await conversationTranslator.JoinConversationAsync(conversationId, displayName, language).ConfigureAwait(false);

            // You can now send an instant message
            await conversationTranslator.SendTextMessageAsync("Message from participant").ConfigureAwait(false);

            // Start capturing audio if you specified a speech-to-text language
            await conversationTranslator.StartTranscribingAsync().ConfigureAwait(false);
            Console.WriteLine("Started transcribing. Press Ctrl-C to stop");

            // At this point, you should start receiving transcriptions for what you are saying using
            // the default microphone. Press Ctrl+C to stop audio capture
            await completionSource.Task.ConfigureAwait(false);

            // Stop audio capture
            await conversationTranslator.StopTranscribingAsync().ConfigureAwait(false);

            // Leave the conversation. You will stop receiving events after this
            await conversationTranslator.LeaveConversationAsync().ConfigureAwait(false);
        }
    }
    ```

2. Remplacez `CreateConversationAsync();` dans votre fonction `public static async Task Main(string[] args)` par :

    ```csharp
    // Set this to the conversation you want to join
    JoinConversationAsync("YourConversationId");
    ```

[!INCLUDE [create-from-web](../create-from-web.md)]

4. Retournez dans Visual Studio et remplacez la chaîne `YourConversationId` par l’ID de conversation que vous avez créé dans l’étape précédente.

5. Dans la barre de menus, sélectionnez **Générer** > **Générer la solution** pour générer l’application. Le code doit maintenant se compiler sans erreurs.

6. Choisissez **Déboguer** > **Démarrer le débogage** (ou appuyez sur **F5**) pour démarrer l’application **helloworld**.

7. Lorsque vous voyez s’afficher le message `Started transcribing`, vous pouvez commencer à parler. Vous verrez que les transcriptions s’affichent à mesure que vous parlez.
    - Si vous revenez dans votre navigateur, vous verrez que vos transcriptions s’affichent également à mesure que vous parlez.

8. Une fois que vous avez fini de parler, appuyez sur <kbd>Ctrl + C</kbd> pour arrêter la capture audio et terminer la conversation.

9. Revenez à votre navigateur et quittez la conversation à l’aide du bouton Quitter situé dans le coin supérieur droit.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [footer](./footer.md)]
