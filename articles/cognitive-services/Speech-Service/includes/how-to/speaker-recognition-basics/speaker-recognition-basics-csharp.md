---
author: v-jaswel
ms.service: cognitive-services
ms.topic: include
ms.date: 09/28/2020
ms.author: v-jawe
ms.custom: references_regions
ms.openlocfilehash: 2b5a34e8f3e7132a16ad3683b846d57e9ece2cb6
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95015467"
---
Dans ce guide de démarrage rapide, vous allez découvrir les modèles de conception de base pour la reconnaissance de l’orateur à l’aide du SDK de reconnaissance vocale, notamment :

* Vérification dépendante ou indépendante du texte
* Identification de l’orateur pour identifier un échantillon vocal parmi un groupe de voix
* Suppression de profils vocaux

Pour obtenir une vue d’ensemble des concepts de reconnaissance vocale, consultez l’article de [présentation](../../../speaker-recognition-overview.md).

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous disposez d’un compte Azure et d’un abonnement au service Speech. Si vous n’avez pas de compte et d’abonnement, [essayez le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free).

> [!IMPORTANT]
> La fonctionnalité Reconnaissance de l’orateur est actuellement *uniquement* prise en charge dans les ressources Azure Speech créées dans la région `westus`.

## <a name="install-the-speech-sdk"></a>Installer le Kit de développement logiciel (SDK) Speech

Avant de pouvoir faire quoi que ce soit, vous devez installer le SDK Speech. Suivez les instructions ci-dessous, en fonction de votre plateforme :

* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=dotnet&pivots=programming-language-csharp" target="_blank">.NET Framework <span class="docon docon-navigate-external x-hidden-focus"></span></a>
* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=dotnetcore&pivots=programming-language-csharp" target="_blank">.NET Core <span class="docon docon-navigate-external x-hidden-focus"></span></a>
* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=unity&pivots=programming-language-csharp" target="_blank">Unity <span class="docon docon-navigate-external x-hidden-focus"></span></a>
* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=uwps&pivots=programming-language-csharp" target="_blank">UWP <span class="docon docon-navigate-external x-hidden-focus"></span></a>
* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=xaml&pivots=programming-language-csharp" target="_blank">Xamarin <span class="docon docon-navigate-external x-hidden-focus"></span></a>

## <a name="import-dependencies"></a>Importer les dépendances

Pour exécuter les exemples de cet article, incluez les instructions `using` suivantes en haut de votre script.

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;
```

## <a name="create-a-speech-configuration"></a>Créer une configuration Speech

Pour appeler le service Speech à l’aide du SDK Speech, vous devez créer une classe [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig?preserve-view=true&view=azure-dotnet). Dans cet exemple, vous allez créer un objet [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig?preserve-view=true&view=azure-dotnet) à partir d’une clé d’abonnement et d’une région. Vous pouvez aussi créer un code réutilisable de base à utiliser pour le reste de cet article, que vous modifiez pour différentes personnalisations.

Notez que la région est définie sur `westus`, car il s’agit de la seule région prise en charge pour le service.

```csharp
public class Program 
{
    static async Task Main(string[] args)
    {
        // replace with your own subscription key 
        string subscriptionKey = "YourSubscriptionKey";
        string region = "westus";
        var config = SpeechConfig.FromSubscription(subscriptionKey, region);
    }
}
```

## <a name="text-dependent-verification"></a>Vérification dépendante du texte

La vérification de l’orateur est l’acte qui consiste à confirmer qu’un orateur correspond à une voix **inscrite** (connue). La première étape consiste à **inscrire** un profil vocal qui permet ensuite au service de comparer les futurs exemples de voix. Dans cet exemple, vous inscrivez le profil à l’aide d’une stratégie **dépendante du texte**, qui requiert une phrase secrète spécifique à utiliser à la fois pour l’inscription et la vérification. Pour obtenir la liste des phrases secrètes prises en charge, consultez la [documentation de référence](/rest/api/speakerrecognition/).

Commencez par créer la fonction suivante dans votre classe `Program` pour inscrire un profil vocal.

```csharp
public static async Task VerificationEnroll(SpeechConfig config, Dictionary<string, string> profileMapping)
{
    using (var client = new VoiceProfileClient(config))
    using (var profile = await client.CreateProfileAsync(VoiceProfileType.TextDependentVerification, "en-us"))
    {
        using (var audioInput = AudioConfig.FromDefaultMicrophoneInput())
        {
            Console.WriteLine($"Enrolling profile id {profile.Id}.");
            // give the profile a human-readable display name
            profileMapping.Add(profile.Id, "Your Name");

            VoiceProfileEnrollmentResult result = null;
            while (result is null || result.RemainingEnrollmentsCount > 0)
            {
                Console.WriteLine("Speak the passphrase, \"My voice is my passport, verify me.\"");
                result = await client.EnrollProfileAsync(profile, audioInput);
                Console.WriteLine($"Remaining enrollments needed: {result.RemainingEnrollmentsCount}");
                Console.WriteLine("");
            }
            
            if (result.Reason == ResultReason.EnrolledVoiceProfile)
            {
                await SpeakerVerify(config, profile, profileMapping);
            }
            else if (result.Reason == ResultReason.Canceled)
            {
                var cancellation = VoiceProfileEnrollmentCancellationDetails.FromResult(result);
                Console.WriteLine($"CANCELED {profile.Id}: ErrorCode={cancellation.ErrorCode} ErrorDetails={cancellation.ErrorDetails}");
            }
        }
    }
}
```

Dans cette fonction, `await client.CreateProfileAsync()` est la commande qui crée le nouveau profil vocal. Une fois la fonction créée, vous spécifiez de quelle façon vous allez entrer des exemples audio, en utilisant `AudioConfig.FromDefaultMicrophoneInput()` dans cet exemple pour capturer l’audio de votre appareil d’entrée par défaut. Ensuite, inscrivez des exemples audio dans une boucle `while` qui effectue le suivi du nombre d’échantillons restants, et requis, pour l’inscription. Dans chaque itération, `client.EnrollProfileAsync(profile, audioInput)` vous invite à prononcer la phrase secrète dans votre microphone, puis ajoute l’exemple au profil vocal.

Une fois l’inscription terminée, vous appelez `await SpeakerVerify(config, profile, profileMapping)` pour effectuer la vérification par rapport au profil que vous venez de créer. Ajoutez une autre fonction pour définir `SpeakerVerify`.

```csharp
public static async Task SpeakerVerify(SpeechConfig config, VoiceProfile profile, Dictionary<string, string> profileMapping)
{
    var speakerRecognizer = new SpeakerRecognizer(config, AudioConfig.FromDefaultMicrophoneInput());
    var model = SpeakerVerificationModel.FromProfile(profile);

    Console.WriteLine("Speak the passphrase to verify: \"My voice is my passport, please verify me.\"");
    var result = await speakerRecognizer.RecognizeOnceAsync(model);
    Console.WriteLine($"Verified voice profile for speaker {profileMapping[result.ProfileId]}, score is {result.Score}");
}
```

Dans cette fonction, vous transmettez l’objet `VoiceProfile` que vous venez de créer pour initialiser un modèle à vérifier. Ensuite, `await speakerRecognizer.RecognizeOnceAsync(model)` vous invite à prononcer à nouveau la phrase secrète, cette fois-ci en la validant par rapport à votre profil vocal, puis retourne un score de similarité compris entre 0,0 et 1,0. L’objet `result` retourne également `Accept` ou `Reject`, selon que la phrase secrète correspond ou non.

Ensuite, modifiez votre fonction `Main()` pour qu’elle appelle les nouvelles fonctions que vous avez créées. Par ailleurs, notez que vous créez un `Dictionary<string, string>` à passer par référence via vos appels de fonction. Cela est dû au fait que le service n’autorise pas le stockage d’un nom lisible par l’utilisateur avec un `VoiceProfile` créé et qu’il stocke uniquement un ID à des fins de confidentialité. Dans la fonction `VerificationEnroll`, ajoutez à ce dictionnaire une entrée avec l’ID nouvellement créé, ainsi qu’un nom de texte. Dans les scénarios de développement d’applications où vous devez afficher un nom lisible par l’utilisateur, **vous devez stocker ce mappage quelque part, car le service ne peut pas le stocker**.

```csharp
static async Task Main(string[] args)
{
    string subscriptionKey = "YourSubscriptionKey";
    string region = "westus";
    var config = SpeechConfig.FromSubscription(subscriptionKey, region);

    // persist profileMapping if you want to store a record of who the profile is
    var profileMapping = new Dictionary<string, string>();
    await VerificationEnroll(config, profileMapping);

    Console.ReadLine();
}
```

Exécutez le script. Vous êtes ensuite invité à prononcer l’expression *Ma voix est mon passeport. Vérifiez-moi* trois fois pour l’inscription et une autre fois pour la vérification. Le résultat retourné est le score de similarité, que vous pouvez utiliser pour créer vos propres seuils personnalisés pour la vérification.

```shell
Enrolling profile id 87-2cef-4dff-995b-dcefb64e203f.
Speak the passphrase, "My voice is my passport, verify me."
Remaining enrollments needed: 2

Speak the passphrase, "My voice is my passport, verify me."
Remaining enrollments needed: 1

Speak the passphrase, "My voice is my passport, verify me."
Remaining enrollments needed: 0

Speak the passphrase to verify: "My voice is my passport, verify me."
Verified voice profile for speaker Your Name, score is 0.915581
```

## <a name="text-independent-verification"></a>Vérification indépendante du texte

Contrairement à la vérification **dépendante du texte**, la vérification **indépendante du texte** :

* Ne requiert pas la lecture d’une phrase secrète. Le texte à prononcer est choisi par l’utilisateur.
* Ne nécessite pas trois échantillons audio, mais *nécessite* 20 secondes d’audio en tout.

Apportez quelques modifications simples à votre fonction `VerificationEnroll` pour basculer vers la vérification **indépendante du texte**. Tout d’abord, définissez le type de vérification sur `VoiceProfileType.TextIndependentVerification`. Ensuite, modifiez la boucle `while` pour suivre `result.RemainingEnrollmentsSpeechLength`, qui continuera à vous demander de parler jusqu’à ce que 20 secondes d’audio aient été capturées.

```csharp
public static async Task VerificationEnroll(SpeechConfig config, Dictionary<string, string> profileMapping)
{
    using (var client = new VoiceProfileClient(config))
    using (var profile = await client.CreateProfileAsync(VoiceProfileType.TextIndependentVerification, "en-us"))
    {
        using (var audioInput = AudioConfig.FromDefaultMicrophoneInput())
        {
            Console.WriteLine($"Enrolling profile id {profile.Id}.");
            // give the profile a human-readable display name
            profileMapping.Add(profile.Id, "Your Name");

            VoiceProfileEnrollmentResult result = null;
            while (result is null || result.RemainingEnrollmentsSpeechLength > TimeSpan.Zero)
            {
                Console.WriteLine("Continue speaking to add to the profile enrollment sample.");
                result = await client.EnrollProfileAsync(profile, audioInput);
                Console.WriteLine($"Remaining enrollment audio time needed: {result.RemainingEnrollmentsSpeechLength}");
                Console.WriteLine("");
            }
            
            if (result.Reason == ResultReason.EnrolledVoiceProfile)
            {
                await SpeakerVerify(config, profile, profileMapping);
            }
            else if (result.Reason == ResultReason.Canceled)
            {
                var cancellation = VoiceProfileEnrollmentCancellationDetails.FromResult(result);
                Console.WriteLine($"CANCELED {profile.Id}: ErrorCode={cancellation.ErrorCode} ErrorDetails={cancellation.ErrorDetails}");
            }
        }
    }
}
```

Réexécutez le programme et prononcez le texte de votre choix pendant la phase de vérification, car aucune phrase secrète n’est requise. Là encore, le score de similarité est retourné.

```shell
Enrolling profile id 4tt87d4-f2d3-44ae-b5b4-f1a8d4036ee9.
Continue speaking to add to the profile enrollment sample.
Remaining enrollment audio time needed: 00:00:15.3200000

Continue speaking to add to the profile enrollment sample.
Remaining enrollment audio time needed: 00:00:09.8100008

Continue speaking to add to the profile enrollment sample.
Remaining enrollment audio time needed: 00:00:05.1900000

Continue speaking to add to the profile enrollment sample.
Remaining enrollment audio time needed: 00:00:00.8700000

Continue speaking to add to the profile enrollment sample.
Remaining enrollment audio time needed: 00:00:00

Speak the passphrase to verify: "My voice is my passport, please verify me."
Verified voice profile for speaker Your Name, score is 0.849409
```

## <a name="speaker-identification"></a>Identification de l’orateur

L’identification de l’orateur est utilisée pour déterminer **qui** parle dans un groupe donné de voix inscrites. Le processus est très similaire à la **vérification indépendante du texte**. La principale différence est la possibilité de vérifier plusieurs profils vocaux à la fois, au lieu d’un seul profil.

Créez une fonction `IdentificationEnroll` pour inscrire plusieurs profils vocaux. Le processus d’inscription de chaque profil est le même que le processus d’inscription pour une **vérification indépendante du texte** et nécessite 20 secondes d’audio pour chaque profil. Cette fonction accepte une liste de chaînes `profileNames` et crée un profil vocal pour chaque nom de la liste. La fonction retourne une liste d’objets `VoiceProfile`, que vous utilisez dans la fonction suivante pour identifier un orateur.

```csharp
public static async Task<List<VoiceProfile>> IdentificationEnroll(SpeechConfig config, List<string> profileNames, Dictionary<string, string> profileMapping)
{
    List<VoiceProfile> voiceProfiles = new List<VoiceProfile>();
    using (var client = new VoiceProfileClient(config))
    {
        foreach (string name in profileNames)
        {
            using (var audioInput = AudioConfig.FromDefaultMicrophoneInput())
            {
                var profile = await client.CreateProfileAsync(VoiceProfileType.TextIndependentIdentification, "en-us");
                Console.WriteLine($"Creating voice profile for {name}.");
                profileMapping.Add(profile.Id, name);

                VoiceProfileEnrollmentResult result = null;
                while (result is null || result.RemainingEnrollmentsSpeechLength > TimeSpan.Zero)
                {
                    Console.WriteLine($"Continue speaking to add to the profile enrollment sample for {name}.");
                    result = await client.EnrollProfileAsync(profile, audioInput);
                    Console.WriteLine($"Remaining enrollment audio time needed: {result.RemainingEnrollmentsSpeechLength}");
                    Console.WriteLine("");
                }
                voiceProfiles.Add(profile);
            }
        }
    }
    return voiceProfiles;
}
```

Créez la fonction `SpeakerIdentification` suivante pour soumettre une demande d’identification. La principale différence dans cette fonction par rapport à une demande de **vérification de l’orateur** est l’utilisation de `SpeakerIdentificationModel.FromProfiles()`, qui accepte une liste d’objets `VoiceProfile`. 

```csharp
public static async Task SpeakerIdentification(SpeechConfig config, List<VoiceProfile> voiceProfiles, Dictionary<string, string> profileMapping) 
{
    var speakerRecognizer = new SpeakerRecognizer(config, AudioConfig.FromDefaultMicrophoneInput());
    var model = SpeakerIdentificationModel.FromProfiles(voiceProfiles);

    Console.WriteLine("Speak some text to identify who it is from your list of enrolled speakers.");
    var result = await speakerRecognizer.RecognizeOnceAsync(model);
    Console.WriteLine($"The most similar voice profile is {profileMapping[result.ProfileId]} with similarity score {result.Score}");
}
```

Remplacez votre fonction `Main()` par ce qui suit. Créez une liste de chaînes `profileNames`que vous transmettez à votre fonction `IdentificationEnroll()`. Vous êtes alors invité à créer un profil vocal pour chaque nom de cette liste. Vous pouvez donc ajouter d’autres noms pour créer des profils supplémentaires pour vos amis ou vos collègues.

```csharp
static async Task Main(string[] args)
{
    // replace with your own subscription key 
    string subscriptionKey = "YourSubscriptionKey";
    string region = "westus";
    var config = SpeechConfig.FromSubscription(subscriptionKey, region);

    // persist profileMapping if you want to store a record of who the profile is
    var profileMapping = new Dictionary<string, string>();
    var profileNames = new List<string>() { "Your name", "A friend's name" };
    
    var enrolledProfiles = await IdentificationEnroll(config, profileNames, profileMapping);
    await SpeakerIdentification(config, enrolledProfiles, profileMapping);

    foreach (var profile in enrolledProfiles)
    {
        profile.Dispose();
    }
    Console.ReadLine();
}
```

Exécutez le script. Vous êtes ensuite invité à entrer des exemples d’inscription de voix pour le premier profil. Une fois l’inscription terminée, vous êtes invité à répéter ce processus pour chaque nom figurant dans la liste `profileNames`. Une fois chaque inscription terminée, vous êtes invité à demander à une **personne** de parler. Le service tente ensuite d’identifier cette personne parmi vos profils vocaux inscrits.

Cet exemple retourne uniquement la correspondance la plus proche et le score de similarité, mais vous pouvez obtenir la réponse complète qui inclut les cinq meilleurs scores de similarité en ajoutant `string json = result.Properties.GetProperty(PropertyId.SpeechServiceResponse_JsonResult)` à votre fonction `SpeakerIdentification`.

## <a name="changing-audio-input-type"></a>Modification du type d’entrée audio

Les exemples de cet article utilisent le microphone d’appareil par défaut comme entrée pour les exemples audio. Toutefois, dans les scénarios où vous devez utiliser des fichiers audio à la place d’une entrée de microphone, définissez simplement une instance de `AudioConfig.FromDefaultMicrophoneInput()` sur `AudioConfig.FromWavFileInput(path/to/your/file.wav)` pour basculer vers une entrée de fichier. Vous pouvez également avoir des entrées mixtes, à l’aide d’un microphone pour l’inscription et de fichiers pour la vérification par exemple.

## <a name="deleting-voice-profile-enrollments"></a>Suppression des inscriptions de profil vocal

Pour supprimer un profil inscrit, utilisez la fonction `DeleteProfileAsync()` sur l’objet `VoiceProfileClient`. L’exemple de fonction suivant montre comment supprimer un profil vocal d’un ID de profil vocal connu.

```csharp
public static async Task DeleteProfile(SpeechConfig config, string profileId) 
{
    using (var client = new VoiceProfileClient(config))
    {
        var profile = new VoiceProfile(profileId);
        await client.DeleteProfileAsync(profile);
    }
}
```