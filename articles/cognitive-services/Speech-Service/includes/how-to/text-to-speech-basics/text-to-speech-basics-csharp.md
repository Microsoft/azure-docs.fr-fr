---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/25/2020
ms.author: trbye
ms.custom: devx-track-csharp
ms.openlocfilehash: 27eecc5560bc99520fea85baf13c0ff4d8a84e7e
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94424906"
---
Dans ce guide de démarrage rapide, vous découvrez les modèles de conception courants qui permettent d’utiliser la synthèse vocale au moyen du kit SDK Speech. Vous commencez par créer une configuration et une synthèse de base, puis passez à des exemples plus poussés en matière de développement d’applications personnalisées, notamment :

* Obtention de réponses en tant que flux en mémoire
* Personnalisation du taux d’échantillonnage de sortie et de la vitesse de transmission
* Envoi de demandes de synthèse à l’aide de SSML (Speech Synthesis Markup Language)
* Utilisation de voix neurales

## <a name="skip-to-samples-on-github"></a>Passer aux exemples sur GitHub

Si vous souhaitez passer directement à l’exemple de code, consultez les [exemples de démarrage rapide C#](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/dotnet/text-to-speech) sur GitHub.

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous disposez d’un compte Azure et d’un abonnement au service Speech. Si vous n’avez pas de compte et d’abonnement, [essayez le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free).

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
using System.IO;
using System.Text;
using System.Threading.Tasks;
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;
```

## <a name="create-a-speech-configuration"></a>Créer une configuration Speech

Pour appeler le service Speech à l’aide du SDK Speech, vous devez créer une classe [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig?preserve-view=true&view=azure-dotnet). Celle-ci comprend des informations sur votre abonnement, telles que votre clé et la région, le point de terminaison, l’hôte ou le jeton d’autorisation associés.

> [!NOTE]
> Quand vous procédez à une reconnaissance vocale, une synthèse vocale, une traduction ou une reconnaissance intentionnelle, vous devez toujours créer une configuration.

Vous pouvez initialiser une [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig?preserve-view=true&view=azure-dotnet) de plusieurs façons :

* Avec un abonnement : transmettez une clé et la région associée.
* Avec un point de terminaison : transmettez un point de terminaison de service Speech. Une clé ou un jeton d’autorisation est facultatif.
* Avec un hôte : transmettez une adresse d’hôte. Une clé ou un jeton d’autorisation est facultatif.
* Avec un jeton d’autorisation : transmettez un jeton d’autorisation et la région associée.

Dans cet exemple, vous allez créer un objet [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig?preserve-view=true&view=azure-dotnet) à partir d’une clé d’abonnement et d’une région. Pour obtenir ces informations d’identification, suivez les étapes indiquées dans [Essayer le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free). Vous pouvez aussi créer un code réutilisable de base à utiliser pour le reste de cet article, que vous modifiez pour différentes personnalisations.

```csharp
public class Program 
{
    static async Task Main()
    {
        await SynthesizeAudioAsync();
    }

    static async Task SynthesizeAudioAsync() 
    {
        var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    }
}
```

## <a name="synthesize-speech-to-a-file"></a>Synthétiser la voix dans un fichier

Ensuite, vous allez créer un objet [`SpeechSynthesizer`](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesizer?preserve-view=true&view=azure-dotnet), qui exécute des conversions de synthèse vocale et des sorties vers des haut-parleurs, des fichiers ou d’autres flux de sortie. [`SpeechSynthesizer`](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesizer?preserve-view=true&view=azure-dotnet) accepte en tant que paramètres l’objet [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig?preserve-view=true&view=azure-dotnet) créé à l’étape précédente et un objet [`AudioConfig`](/dotnet/api/microsoft.cognitiveservices.speech.audio.audioconfig?preserve-view=true&view=azure-dotnet) qui spécifie la façon dont les résultats de sortie doivent être gérés.

Pour commencer, créez un `AudioConfig` pour écrire automatiquement la sortie dans un fichier `.wav` à l’aide de la fonction `FromWavFileOutput()` et l’instancier avec une instruction `using`. Une instruction `using` dans ce contexte élimine automatiquement les ressources non managées et fait sortir l’objet de l’étendue après l’élimination.

```csharp
static async Task SynthesizeAudioAsync() 
{
    var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    using var audioConfig = AudioConfig.FromWavFileOutput("path/to/write/file.wav");
}
```

Ensuite, instanciez un `SpeechSynthesizer` avec une autre instruction `using`. Transmettez votre objet `config` et l’objet `audioConfig` en tant que paramètres. Dès lors, exécuter la synthèse vocale et écrire dans un fichier sont aussi simples que d’exécuter `SpeakTextAsync()` avec une chaîne de texte.

```csharp
static async Task SynthesizeAudioAsync() 
{
    var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    using var audioConfig = AudioConfig.FromWavFileOutput("path/to/write/file.wav");
    using var synthesizer = new SpeechSynthesizer(config, audioConfig);
    await synthesizer.SpeakTextAsync("A simple test to write to a file.");
}
```

Exécutez le programme ; un fichier `.wav` synthétisé est alors écrit à l’emplacement que vous avez spécifié. Il s’agit d’un bon exemple de l’utilisation la plus simple, mais vous allez maintenant vous pencher sur la personnalisation de la sortie et sur la gestion de la réponse de sortie sous forme de flux en mémoire pour travailler sur des scénarios personnalisés.

## <a name="synthesize-to-speaker-output"></a>Synthétiser vers la sortie du haut-parleur

Dans certains cas, vous pouvez souhaiter sortir directement la synthèse vocale vers un haut-parleur. Pour cela, il suffit d’omettre le paramètre `AudioConfig` au moment de créer le `SpeechSynthesizer` dans l’exemple ci-dessus. La sortie est alors dirigée vers l’appareil de sortie actuellement actif.

```csharp
static async Task SynthesizeAudioAsync() 
{
    var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    using var synthesizer = new SpeechSynthesizer(config);
    await synthesizer.SpeakTextAsync("Synthesizing directly to speaker output.");
}
```

## <a name="get-result-as-an-in-memory-stream"></a>Obtenir le résultat sous forme de flux en mémoire

Dans de nombreux scénarios de développement d’application vocale, vous aurez probablement besoin des données audio obtenues sous forme de flux en mémoire plutôt que directement écrites dans un fichier. Cela vous permettra de générer un comportement personnalisé, notamment :

* Résumer le tableau d’octets obtenu sous forme de flux pouvant faire l’objet de recherches pour des services en aval personnalisés.
* Intégrer le résultat à d’autres API ou services.
* Modifier les données audio, écrire des en-têtes `.wav` personnalisés, etc.

Il est facile d’apporter cette modification à partir de l’exemple précédent. Tout d’abord, supprimez le bloc `AudioConfig`, car vous allez gérer le comportement de sortie manuellement à partir de ce point pour un contrôle accru. Transmettez ensuite `null` pour `AudioConfig` dans le constructeur `SpeechSynthesizer`. 

> [!NOTE]
> Le fait de transmettre `null` pour `AudioConfig`, au lieu de l’omettre comme dans l’exemple de sortie de haut-parleur précédent, n’aura pas pour effet de lire par défaut l’audio sur l’appareil de sortie actuellement actif.

Cette fois, vous allez enregistrer le résultat dans une variable [`SpeechSynthesisResult`](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesisresult?preserve-view=true&view=azure-dotnet). La propriété `AudioData` contient un `byte []` des données de sortie. Vous pouvez utiliser ce `byte []` manuellement ou utiliser la classe [`AudioDataStream`](/dotnet/api/microsoft.cognitiveservices.speech.audiodatastream?preserve-view=true&view=azure-dotnet) pour gérer le flux en mémoire. Dans cet exemple, vous allez utiliser la fonction statique `AudioDataStream.FromResult()` pour obtenir un flux à partir du résultat.

```csharp
static async Task SynthesizeAudioAsync() 
{
    var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    using var synthesizer = new SpeechSynthesizer(config, null);
    
    var result = await synthesizer.SpeakTextAsync("Getting the response as an in-memory stream.");
    using var stream = AudioDataStream.FromResult(result);
}
```

À partir de là, vous pouvez implémenter un comportement personnalisé à partir de l’objet `stream` obtenu.

## <a name="customize-audio-format"></a>Personnaliser le format audio

La section suivante montre comment personnaliser les attributs de sortie audio, notamment :

* Le type de fichier audio
* Le taux d’échantillonnage
* La profondeur de bits

Pour changer de format audio, utilisez la fonction `SetSpeechSynthesisOutputFormat()` sur l’objet `SpeechConfig`. Cette fonction attend un `enum` de type [`SpeechSynthesisOutputFormat`](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesisoutputformat?preserve-view=true&view=azure-dotnet), dont vous allez vous servir pour sélectionner le format de sortie. Consultez les documents de référence pour obtenir la [liste des formats audio](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesisoutputformat?preserve-view=true&view=azure-dotnet) disponibles.

Il existe diverses options pour les différents types de fichiers, ce qui permet de répondre à vos besoins. Notez que par définition, les formats bruts comme `Raw24Khz16BitMonoPcm` n’incluent pas d’en-têtes audio. N’utilisez les formats bruts que si vous savez que votre implémentation en aval pourra décoder un flux binaire brut ou bien si vous envisagez de créer manuellement des en-têtes en fonction de la profondeur de bits, du taux d’échantillonnage, du nombre de canaux, etc.

Dans cet exemple, vous allez spécifier un format RIFF haute fidélité `Riff24Khz16BitMonoPcm` en définissant `SpeechSynthesisOutputFormat` sur l’objet `SpeechConfig`. Comme dans l’exemple de la section précédente, vous allez utiliser [`AudioDataStream`](/dotnet/api/microsoft.cognitiveservices.speech.audiodatastream?preserve-view=true&view=azure-dotnet) pour obtenir un flux en mémoire du résultat, puis l’écrire dans un fichier.

```csharp
static async Task SynthesizeAudioAsync() 
{
    var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    config.SetSpeechSynthesisOutputFormat(SpeechSynthesisOutputFormat.Riff24Khz16BitMonoPcm);

    using var synthesizer = new SpeechSynthesizer(config, null);
    var result = await synthesizer.SpeakTextAsync("Customizing audio output format.");

    using var stream = AudioDataStream.FromResult(result);
    await stream.SaveToWaveFileAsync("path/to/write/file.wav");
}
```

Si vous exécutez à nouveau votre programme, un fichier `.wav` est écrit dans le chemin spécifié.

## <a name="use-ssml-to-customize-speech-characteristics"></a>Utiliser SSML pour personnaliser les caractéristiques vocales

Le langage SSML (Speech Synthesis Markup Language) vous permet d’ajuster la tonalité, la prononciation, la vitesse d’élocution, le volume et d’autres éléments de sortie de la synthèse vocale en soumettant vos demandes à partir d’un schéma XML. Cette section présente quelques exemples d’utilisation pratiques, mais pour des indications plus détaillées, consultez l’[article de procédures SSML](../../../speech-synthesis-markup.md).

Pour commencer à utiliser SSML pour la personnalisation, vous allez apporter une modification simple qui change la voix.
Tout d’abord, créez un fichier XML pour la configuration SSML dans le répertoire racine de votre projet, à savoir `ssml.xml`. L’élément racine est toujours `<speak>`, et le fait d’intégrer le texte dans un élément `<voice>` vous permet de changer la voix à l’aide du paramètre `name`. Cet exemple change la voix en voix masculine de langue anglaise (Royaume-Uni). Notez que cette voix est une voix **standard** , dont le coût et la disponibilité sont différents de ceux des voix **neuronales**. Consultez la [liste complète](../../../language-support.md#standard-voices) des voix **standard** prises en charge.

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
  <voice name="en-GB-George-Apollo">
    When you're on the motorway, it's a good idea to use a sat-nav.
  </voice>
</speak>
```

Ensuite, vous devez changer la demande de synthèse vocale de sorte qu’elle fasse référence à votre fichier XML. La demande est presque totalement identique, sauf qu’au lieu d’utiliser la fonction `SpeakTextAsync()`, vous allez utiliser `SpeakSsmlAsync()`. Sachant que cette fonction attend une chaîne XML, vous commencez par charger votre configuration SSML sous forme de chaîne à l’aide de `File.ReadAllText()`. À partir de là, l’objet obtenu est exactement le même que dans les exemples précédents.

> [!NOTE]
> Si vous utilisez Visual Studio, il est probable que la configuration de votre build ne trouvera pas par défaut votre fichier XML. Pour corriger ce problème, cliquez avec le bouton droit sur le fichier et sélectionnez **Propriétés**. Faites passer **Action de build** à *Contenu* , puis **Copier dans le répertoire de sortie** à *Toujours copier*.

```csharp
public static async Task SynthesizeAudioAsync() 
{
    var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    using var synthesizer = new SpeechSynthesizer(config, null);
    
    var ssml = File.ReadAllText("./ssml.xml");
    var result = await synthesizer.SpeakSsmlAsync(ssml);

    using var stream = AudioDataStream.FromResult(result);
    await stream.SaveToWaveFileAsync("path/to/write/file.wav");
}
```

Bien que la sortie fonctionne, vous pouvez y apporter d’autres modifications simples pour obtenir un rendu sonore plus naturel. La vitesse d’élocution générale étant un peu trop rapide, nous allons ajouter une balise `<prosody>` et réduire la vitesse à **90 %** de la vitesse par défaut. De plus, la pause après la virgule dans la phrase est un peu trop courte et n’est pas naturelle à l’oreille. Pour corriger ce problème, ajoutez une balise `<break>` pour ralentir le débit de parole, puis définissez le paramètre de temps sur **200ms**. Réexécutez la synthèse pour observer l’incidence de ces personnalisations sur la sortie.

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
  <voice name="en-GB-George-Apollo">
    <prosody rate="0.9">
      When you're on the motorway,<break time="200ms"/> it's a good idea to use a sat-nav.
    </prosody>
  </voice>
</speak>
```

## <a name="neural-voices"></a>Voix neurales

Les voix neuronales sont des algorithmes de synthèse vocale qui reposent sur des réseaux neuronaux profonds. Lorsque vous utilisez une voix neuronale, il est presque impossible de distinguer la voix synthétisée des enregistrements humains. Grâce à la prosodie naturelle quasi humaine et à la bonne articulation des mots, les voix neuronales réduisent considérablement la fatigue d’écoute des utilisateurs qui interagissent avec les systèmes d’intelligence artificielle.

Pour passer à une voix neuronale, remplacez `name` par une des [options de voix neuronale](../../../language-support.md#neural-voices). Ensuite, ajoutez un espace de noms XML pour `mstts` et intégrez votre texte dans la balise `<mstts:express-as>`. Utilisez le paramètre `style` pour personnaliser le style vocal. Cet exemple utilise le style vocal `cheerful`, mais essayez de le remplacer par `customerservice` ou `chat` pour voir la différence.

> [!IMPORTANT]
> Les voix neuronales sont **uniquement** prises en charge pour les ressources vocales créées dans les régions *USA Est* , *Asie Sud-Est* et *Europe Ouest*.

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="en-US">
  <voice name="en-US-AriaNeural">
    <mstts:express-as style="cheerful">
      This is awesome!
    </mstts:express-as>
  </voice>
</speak>
```