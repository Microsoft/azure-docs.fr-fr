---
title: Créer un modèle de locataire (préversion) – Service Speech
titleSuffix: Azure Cognitive Services
description: Générez automatiquement un modèle de locataire sécurisé et compatible (Custom Speech avec données Microsoft 365) qui utilise vos données Microsoft 365 pour fournir une reconnaissance vocale optimale pour les termes propres à votre organisation.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: tutorial
ms.date: 06/25/2020
ms.author: erhopf
ms.custom: devx-track-csharp
ms.openlocfilehash: 48cde51ee9941f705aa848d121c419a8f0c9ad1a
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95013710"
---
# <a name="tutorial-create-a-tenant-model-preview"></a>Tutoriel : Créer un modèle de locataire (préversion)

Tenant Model (Custom Speech avec données Microsoft 365) est un service d’activation pour les clients d’entreprise Microsoft 365 qui génère automatiquement un modèle de reconnaissance vocale personnalisé à partir des données Microsoft 365 de votre organisation. Le modèle est optimisé pour les termes techniques, le jargon et les noms de personnes, le tout d’une manière sécurisée et conforme.

> [!IMPORTANT]
> Si votre organisation s’inscrit via le service Tenant Model, il se peut que le service Speech accède au modèle de langage de votre organisation. Un modèle est généré à partir d’e-mails et de documents de groupe public Microsoft 365 qui peuvent être consultés par quiconque au sein de votre organisation. L’administrateur de votre organisation peut activer ou désactiver l’utilisation du modèle de langage à l’échelle de l’organisation à partir du portail d’administration.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * S’inscrire à Tenant Model à partir du centre d’administration Microsoft 365
> * Obtenir une clé d’abonnement Speech
> * Créer un modèle de locataire
> * Déployer un modèle de locataire
> * Utiliser votre modèle de locataire avec le kit SDK Speech

## <a name="enroll-in-the-tenant-model-service"></a>S’inscrire au service Tenant Model

Avant de pouvoir déployer votre modèle de locataire, vous devez vous être inscrit au service Tenant Model. L’inscription s’effectue dans le centre d’administration Microsoft 365 et ne peut être réalisée que par votre administrateur.

1. Connectez-vous au [centre d’administration Microsoft 365](https://admin.microsoft.com).

1. Dans le volet gauche, sélectionnez **Paramètres**, puis sélectionnez **Paramètres** dans le menu imbriqué et **Azure Speech Services** dans la fenêtre principale.

   ![Volet « Services et compléments »](media/tenant-language-model/tenant-language-model-enrollment.png)

1. Cochez la case **Autoriser le modèle de langage à l’échelle de l’organisation**, puis sélectionnez **Enregistrer les modifications**.

   ![Volet Services de reconnaissance vocale Azure](media/tenant-language-model/tenant-language-model-enrollment-2.png)

Pour désactiver l’instance du modèle de locataire :
1. Répétez les étapes 1 et 2 précédentes.
1. Décochez la case **Autoriser le modèle de langue à l’échelle de l’organisation**, puis sélectionnez **Enregistrer les modifications**.

## <a name="get-a-speech-subscription-key"></a>Obtenir une clé d’abonnement Speech

Pour utiliser votre modèle de locataire avec le kit SDK Speech, vous avez besoin d’une ressource Speech et de sa clé d’abonnement associée.

1. Connectez-vous au [portail Azure](https://aka.ms/azureportal).
1. Sélectionnez **Créer une ressource**.
1. Dans la zone **Rechercher**, tapez **Speech**.
1. Dans la liste des résultats, sélectionnez **Speech**, puis **Créer**.
1. Suivez les instructions à l’écran pour créer votre ressource. Assurez-vous que :
   * **Emplacement** est défini sur **eastus** ou **westus**.
   * **Niveau tarifaire** est défini sur **S0**.
1. Sélectionnez **Create** (Créer).

   Après quelques minutes, votre ressource est créée. La clé d’abonnement est disponible dans la section **Vue d’ensemble** de votre ressource.

## <a name="create-a-language-model"></a>Créer un modèle de langage

Dès lors que votre administrateur a activé Tenant Model pour votre organisation, vous pouvez créer un modèle de langage basé sur vos données Microsoft 365.

1. Connectez-vous à [Speech Studio](https://speech.microsoft.com/).
1. En haut à droite, sélectionnez **Settings** (icône d’engrenage), puis sélectionnez **Tenant Model settings** (Paramètres Tenant Model).

   ![Lien « Tenant Model settings » (Paramètres Tenant Model)](media/tenant-language-model/tenant-language-settings.png)

   Speech Studio affiche un message qui vous indique si vous êtes apte à créer un modèle de locataire.

   > [!NOTE]
   > Les clients d’entreprise d’Amérique du Nord sont éligibles à la création d’un modèle de locataire (anglais). Si vous êtes un client Customer Lockbox (CLB), Customer Key (CK) ou Office 365 Government, cette fonctionnalité n’est pas disponible. Pour déterminer si vous êtes un client Customer Lockbox ou Customer Key, consultez :
   > * [Customer Lockbox](/microsoft-365/compliance/customer-lockbox-requests)
   > * [Customer Key](/microsoft-365/compliance/customer-key-overview)
   > * [Office 365 Government](https://www.microsoft.com/microsoft-365/government)

1. Sélectionnez **Activer**.

   Quand votre modèle de locataire sera prêt, vous recevrez un e-mail de confirmation avec des instructions supplémentaires.

## <a name="deploy-your-tenant-model"></a>Déployer votre modèle de locataire

Une fois que l’instance de votre modèle de locataire est prête, déployez-la en procédant comme suit :

1. Dans l’e-mail de confirmation, sélectionnez le bouton **View model** (Voir le modèle). Ou bien, connectez-vous à [Speech Studio](https://speech.microsoft.com/).
1. En haut à droite, sélectionnez **Settings** (icône d’engrenage), puis sélectionnez **Tenant Model settings** (Paramètres Tenant Model).

   ![Lien « Tenant Model settings » (Paramètres Tenant Model)](media/tenant-language-model/tenant-language-settings.png)

1. Sélectionnez **Déployer**.

   Une fois que votre modèle est déployé, son état passe à *Deployed* (Déployé).

## <a name="use-your-tenant-model-with-the-speech-sdk"></a>Utiliser votre modèle de locataire avec le kit SDK Speech

Maintenant que vous avez déployé votre modèle, vous pouvez l’utiliser avec le Kit de développement logiciel (SDK) Speech. Dans cette section, vous allez utiliser un exemple de code pour appeler le service Speech en utilisant l’authentification Azure Active Directory (Azure AD).

Examinons le code que vous utiliserez pour appeler le Kit de développement logiciel (SDK) Speech dans C#. Dans cet exemple, vous allez procéder à une reconnaissance vocale en utilisant votre modèle de locataire. Ce guide suppose que votre plateforme est déjà configurée. Si vous avez besoin d’aide pour la configuration, consultez [Démarrage rapide : Reconnaissance vocale, C# (.NET Core)](./get-started-speech-to-text.md?pivots=programming-language-csharp&tabs=dotnetcore).

Copiez ce code dans votre projet :

```csharp
namespace PrincetonSROnly.FrontEnd.Samples
{
    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Net.Http;
    using System.Text;
    using System.Text.RegularExpressions;
    using System.Threading.Tasks;
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Newtonsoft.Json.Linq;

    // ServiceApplicationId is a fixed value. No need to change it.

    public class TenantLMSample
    {
        private const string EndpointUriArgName = "EndpointUri";
        private const string SubscriptionKeyArgName = "SubscriptionKey";
        private const string UsernameArgName = "Username";
        private const string PasswordArgName = "Password";
        private const string ClientApplicationId = "f87bc118-1576-4097-93c9-dbf8f45ef0dd";
        private const string ServiceApplicationId = "18301695-f99d-4cae-9618-6901d4bdc7be";

        public static async Task ContinuousRecognitionWithTenantLMAsync(Uri endpointUri, string subscriptionKey, string audioDirPath, string username, string password)
        {
            var config = SpeechConfig.FromEndpoint(endpointUri, subscriptionKey);

            // Passing client specific information for obtaining LM
            if (string.IsNullOrEmpty(username) || string.IsNullOrEmpty(password))
            {
                config.AuthorizationToken = await AcquireAuthTokenWithInteractiveLoginAsync().ConfigureAwait(false);
            }
            else
            {
                config.AuthorizationToken = await AcquireAuthTokenWithUsernamePasswordAsync(username, password).ConfigureAwait(false);
            }

            var stopRecognition = new TaskCompletionSource<int>();

            // Creates a speech recognizer using file as audio input.
            // Replace with your own audio file name.
            using (var audioInput = AudioConfig.FromWavFileInput(audioDirPath))
            {
                using (var recognizer = new SpeechRecognizer(config, audioInput))
                {
                    // Subscribes to events
                    recognizer.Recognizing += (s, e) =>
                    {
                        Console.WriteLine($"RECOGNIZING: Text={e.Result.Text}");
                    };

                    recognizer.Recognized += (s, e) =>
                    {
                        if (e.Result.Reason == ResultReason.RecognizedSpeech)
                        {
                            Console.WriteLine($"RECOGNIZED: Text={e.Result.Text}");
                        }
                        else if (e.Result.Reason == ResultReason.NoMatch)
                        {
                            Console.WriteLine($"NOMATCH: Speech could not be recognized.");
                        }
                    };

                    recognizer.Canceled += (s, e) =>
                    {
                        Console.WriteLine($"CANCELED: Reason={e.Reason}");
                        if (e.Reason == CancellationReason.Error)
                        {
                            Exception exp = new Exception(string.Format("Error Code: {0}\nError Details{1}\nIs your subscription information updated?", e.ErrorCode, e.ErrorDetails));
                            throw exp;
                        }

                        stopRecognition.TrySetResult(0);
                    };

                    recognizer.SessionStarted += (s, e) =>
                    {
                        Console.WriteLine("\n    Session started event.");
                    };

                    recognizer.SessionStopped += (s, e) =>
                    {
                        Console.WriteLine("\n    Session stopped event.");
                        Console.WriteLine("\nStop recognition.");
                        stopRecognition.TrySetResult(0);
                    };

                    // Starts continuous recognition. Uses StopContinuousRecognitionAsync() to stop recognition.
                    await recognizer.StartContinuousRecognitionAsync().ConfigureAwait(false);

                    // Waits for completion.
                    // Use Task.WaitAny to keep the task rooted.
                    Task.WaitAny(new[] { stopRecognition.Task });

                    // Stops recognition.
                    await recognizer.StopContinuousRecognitionAsync().ConfigureAwait(false);
                }
            }
        }

        public static void Main(string[] args)
        {
            var arguments = new Dictionary<string, string>();
            string inputArgNamePattern = "--";
            Regex regex = new Regex(inputArgNamePattern);
            if (args.Length > 0)
            {
                foreach (var arg in args)
                {
                    var userArgs = arg.Split("=");
                    arguments[regex.Replace(userArgs[0], string.Empty)] = userArgs[1];
                }
            }

            var endpointString = arguments.GetValueOrDefault(EndpointUriArgName, $"wss://westus.online.princeton.customspeech.ai/msgraphcustomspeech/conversation/v1");
            var endpointUri = new Uri(endpointString);

            if (!arguments.ContainsKey(SubscriptionKeyArgName))
            {
                Exception exp = new Exception("Subscription Key missing! Please pass in a Cognitive services subscription Key using --SubscriptionKey=\"your_subscription_key\"" +
                    "Find more information on creating a Cognitive services resource and accessing your Subscription key here: https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account?tabs=multiservice%2Cwindows");
                throw exp;
            }

            var subscriptionKey = arguments[SubscriptionKeyArgName];
            var username = arguments.GetValueOrDefault(UsernameArgName, null);
            var password = arguments.GetValueOrDefault(PasswordArgName, null);

            var audioDirPath = Path.Combine(Path.GetDirectoryName(System.Reflection.Assembly.GetExecutingAssembly().Location), "../../../AudioSamples/DictationBatman.wav");
            if (!File.Exists(audioDirPath))
            {
                Exception exp = new Exception(string.Format("Audio File does not exist at path: {0}", audioDirPath));
                throw exp;
            }

            ContinuousRecognitionWithTenantLMAsync(endpointUri, subscriptionKey, audioDirPath, username, password).GetAwaiter().GetResult();
        }

        private static async Task<string> AcquireAuthTokenWithUsernamePasswordAsync(string username, string password)
        {
            var tokenEndpoint = "https://login.microsoftonline.com/common/oauth2/token";
            var postBody = $"resource={ServiceApplicationId}&client_id={ClientApplicationId}&grant_type=password&username={username}&password={password}";
            var stringContent = new StringContent(postBody, Encoding.UTF8, "application/x-www-form-urlencoded");
            using (HttpClient httpClient = new HttpClient())
            {
                var response = await httpClient.PostAsync(tokenEndpoint, stringContent).ConfigureAwait(false);

                if (response.IsSuccessStatusCode)
                {
                    var result = await response.Content.ReadAsStringAsync().ConfigureAwait(false);

                    JObject jobject = JObject.Parse(result);
                    return jobject["access_token"].Value<string>();
                }
                else
                {
                    throw new Exception($"Requesting token from {tokenEndpoint} failed with status code {response.StatusCode}: {await response.Content.ReadAsStringAsync().ConfigureAwait(false)}");
                }
            }
        }

        private static async Task<string> AcquireAuthTokenWithInteractiveLoginAsync()
        {
            var authContext = new AuthenticationContext("https://login.windows.net/microsoft.onmicrosoft.com");
            var deviceCodeResult = await authContext.AcquireDeviceCodeAsync(ServiceApplicationId, ClientApplicationId).ConfigureAwait(false);

            Console.WriteLine(deviceCodeResult.Message);

            var authResult = await authContext.AcquireTokenByDeviceCodeAsync(deviceCodeResult).ConfigureAwait(false);

            return authResult.AccessToken;
        }
    }
}
```

Vous devez ensuite regénérer et exécuter le projet à partir de la ligne de commande. Avant d’exécuter la commande, mettez à jour quelques paramètres en procédant comme suit :

1. Remplacez `<Username>` et `<Password>` par les valeurs d’un utilisateur locataire valide.
1. Remplacez `<Subscription-Key>` par la clé d’abonnement de votre ressource Speech. Cette valeur est disponible dans la section **Vue d’ensemble** de votre ressource Speech dans le [portail Azure](https://aka.ms/azureportal).
1. Remplacez `<Endpoint-Uri>` par le point de terminaison suivant. Veillez à remplacer `{your region}` par la région dans laquelle votre ressource Speech a été créée. Ces régions sont prises en charge : `westus`, `westus2` et `eastus`. Les informations sur votre région sont disponibles dans la section **Vue d’ensemble** de votre ressource Speech sur le [portail Azure](https://aka.ms/azureportal).
   ```
   "wss://{your region}.online.princeton.customspeech.ai/msgraphcustomspeech/conversation/v1".
   ```
1. Exécutez la commande suivante :

   ```bash
   dotnet TenantLMSample.dll --Username=<Username> --Password=<Password> --SubscriptionKey=<Subscription-Key> --EndpointUri=<Endpoint-Uri>
   ```

Dans ce tutoriel, vous avez découvert comment utiliser des données Microsoft 365 pour créer un modèle de reconnaissance vocale personnalisé, le déployer et l’utiliser avec le kit SDK Speech.

## <a name="next-steps"></a>Étapes suivantes

* [Speech Studio](https://speech.microsoft.com/)
* [Kit de développement logiciel (SDK) de reconnaissance vocale](speech-sdk.md)