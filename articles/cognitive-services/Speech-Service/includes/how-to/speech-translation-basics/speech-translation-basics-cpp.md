---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 04/13/2020
ms.author: trbye
ms.openlocfilehash: d5489c5c548d1242ebac9f17faa8ed55fa79108b
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94425033"
---
L’une des principales fonctionnalités du service Speech est la possibilité de reconnaître la parole humaine et de la traduire dans d’autres langues. Dans ce démarrage rapide, vous allez apprendre à utiliser le Kit de développement logiciel (SDK) Speech dans vos applications et produits afin d’effectuer une traduction vocale de qualité. Ce démarrage rapide couvre les sujets suivants :

* Conversion de parole en texte
* Traduction de la parole en plusieurs langues cibles
* Réalisation d’une traduction directe de la parole en parole

## <a name="skip-to-samples-on-github"></a>Passer aux exemples sur GitHub

Si vous souhaitez passer directement à l’exemple de code, consultez les [exemples de démarrage rapide C++](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/cpp/windows/translate-speech-to-text) sur GitHub.

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous disposez d’un compte Azure et d’un abonnement au service Speech. Si vous n’avez pas de compte et d’abonnement, [essayez le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Installer le Kit de développement logiciel (SDK) Speech

Avant de pouvoir faire quoi que ce soit, vous devez installer le SDK Speech. Selon votre plateforme, suivez les instructions fournies dans la section <a href="https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/speech-sdk#get-the-speech-sdk" target="_blank">Obtenir le kit SDK Speech <span class="docon docon-navigate-external x-hidden-focus"></span></a> de l’article _À propos du kit SDK Speech_.

## <a name="import-dependencies"></a>Importer les dépendances

Pour exécuter les exemples de cet article, ajoutez les instructions `#include` et `using` suivantes en haut de votre fichier de code C++.

```cpp
#include <iostream> // cin, cout
#include <fstream>
#include <string>
#include <stdio.h>
#include <stdlib.h>
#include <speechapi_cxx.h>

using namespace std;
using namespace Microsoft::CognitiveServices::Speech;
using namespace Microsoft::CognitiveServices::Speech::Audio;
using namespace Microsoft::CognitiveServices::Speech::Translation;
```

## <a name="sensitive-data-and-environment-variables"></a>Données sensibles et variables d’environnement

L’exemple de code source de cet article dépend de variables d’environnement pour le stockage des données sensibles, par exemple la clé et la région de l’abonnement aux ressources Speech. Le fichier de code C++ contient deux valeurs de chaîne qui sont affectées à partir des variables d’environnement des ordinateurs hôtes, à savoir `SPEECH__SUBSCRIPTION__KEY` et `SPEECH__SERVICE__REGION`. Ces deux champs ont pour portée la classe, ce qui les rend accessibles dans le corps des méthodes de la classe. Pour plus d’informations sur les variables d’environnement, consultez [Variables d’environnement et configuration de l’application](../../../../cognitive-services-security.md#environment-variables-and-application-configuration).

```cpp
auto SPEECH__SUBSCRIPTION__KEY = getenv("SPEECH__SUBSCRIPTION__KEY");
auto SPEECH__SERVICE__REGION = getenv("SPEECH__SERVICE__REGION");
```

## <a name="create-a-speech-translation-configuration"></a>Création d’une configuration de traduction vocale

Pour appeler le service Speech à l’aide du SDK Speech, vous devez créer une classe [`SpeechTranslationConfig`][config]. Celle-ci comprend des informations sur votre abonnement, telles que votre clé et la région, le point de terminaison, l’hôte ou le jeton d’autorisation associés.

> [!TIP]
> Quand vous procédez à une reconnaissance vocale, une synthèse vocale, une traduction ou une reconnaissance intentionnelle, vous devez toujours créer une configuration.

Vous pouvez initialiser une [`SpeechTranslationConfig`][config] de plusieurs façons :

* Avec un abonnement : transmettez une clé et la région associée.
* Avec un point de terminaison : transmettez un point de terminaison de service Speech. Une clé ou un jeton d’autorisation est facultatif.
* Avec un hôte : transmettez une adresse d’hôte. Une clé ou un jeton d’autorisation est facultatif.
* Avec un jeton d’autorisation : transmettez un jeton d’autorisation et la région associée.

Examinons comment créer une classe [`SpeechTranslationConfig`][config] à l’aide d’une clé et d’une région. Pour obtenir ces informations d’identification, suivez les étapes décrites dans [Essayer le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free).

```cpp
auto SPEECH__SUBSCRIPTION__KEY = getenv("SPEECH__SUBSCRIPTION__KEY");
auto SPEECH__SERVICE__REGION = getenv("SPEECH__SERVICE__REGION");

void translateSpeech() {
    auto config =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
}

int main(int argc, char** argv) {
    setlocale(LC_ALL, "");
    translateSpeech();
    return 0;
}
```

## <a name="change-source-language"></a>Changer la langue source

L’une des tâches courantes de la traduction vocale consiste à spécifier la langue d’entrée (ou source). Examinons comment choisir l’italien comme langue d’entrée. Dans votre code, interagissez avec l’instance [`SpeechTranslationConfig`][config], en appelant la méthode `SetSpeechRecognitionLanguage`.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    // Source (input) language
    translationConfig->SetSpeechRecognitionLanguage("it-IT");
}
```

La propriété [`SpeechRecognitionLanguage`][recognitionlang] attend une chaîne au format langue-paramètres régionaux. Vous pouvez fournir n’importe quelle valeur figurant dans la colonne **Paramètres régionaux** dans la liste des [paramètres régionaux/langues](../../../language-support.md) pris en charge.

## <a name="add-translation-language"></a>Ajout d’une langue de traduction

Une autre tâche courante de traduction vocale consiste à spécifier les langues de traduction cibles. S’il en faut au moins une, il est aussi possible d’en indiquer plusieurs. Dans l’extrait de code suivant, le français et l’allemand sont définis comme langues de traduction cibles.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    translationConfig->SetSpeechRecognitionLanguage("it-IT");

    // Translate to languages. See, https://aka.ms/speech/sttt-languages
    translationConfig->AddTargetLanguage("fr");
    translationConfig->AddTargetLanguage("de");
}
```

À chaque appel à [`AddTargetLanguage`][addlang], une nouvelle langue de traduction cible est spécifiée. En d’autres termes, lorsqu’un message est reconnu dans la langue source, chaque traduction cible est disponible dans le cadre de l’opération de traduction qui en résulte.

## <a name="initialize-a-translation-recognizer"></a>Initialisation d’un moteur de reconnaissance de traduction

Une fois que vous avez créé une [`SpeechTranslationConfig`][config], l’étape suivante consiste à initialiser un [`TranslationRecognizer`][recognizer]. Quand vous initialisez un [`TranslationRecognizer`][recognizer], vous devez lui transmettre votre `translationConfig`. L’objet de configuration fournit les informations d’identification dont le service Speech a besoin pour valider la demande.

Si vous effectuez une reconnaissance vocale en utilisant le microphone par défaut de votre appareil, voici à quoi doit ressembler le [`TranslationRecognizer`][recognizer] :

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    auto fromLanguage = "en-US";
    auto toLanguages = { "it", "fr", "de" };
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    for (auto language : toLanguages) {
        translationConfig->AddTargetLanguage(language);
    }

    auto recognizer = TranslationRecognizer::FromConfig(translationConfig);
}
```

Si vous souhaitez spécifier le périphérique d’entrée audio, vous devez créer une classe [`AudioConfig`][audioconfig] et fournir le paramètre `audioConfig` lors de l’initialisation de votre [`TranslationRecognizer`][recognizer].

> [!TIP]
> [Découvrez comment obtenir l’ID de votre périphérique d’entrée audio](../../../how-to-select-audio-input-devices.md).

Tout d’abord, faites référence à l’objet `AudioConfig` de la façon suivante :

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    auto fromLanguage = "en-US";
    auto toLanguages = { "it", "fr", "de" };
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    for (auto language : toLanguages) {
        translationConfig->AddTargetLanguage(language);
    }

    auto audioConfig = AudioConfig::FromDefaultMicrophoneInput();
    auto recognizer = TranslationRecognizer::FromConfig(translationConfig, audioConfig);
}
```

Si vous souhaitez fournir un fichier audio au lieu d’utiliser un microphone, vous devez quand même fournir un objet `audioConfig`. Toutefois, quand vous créerez une classe [`AudioConfig`][audioconfig], au lieu d’appeler `FromDefaultMicrophoneInput`, vous appellerez `FromWavFileInput` et transmettrez le paramètre `filename`.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    auto fromLanguage = "en-US";
    auto toLanguages = { "it", "fr", "de" };
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    for (auto language : toLanguages) {
        translationConfig->AddTargetLanguage(language);
    }

    auto audioConfig = AudioConfig::FromWavFileInput("YourAudioFile.wav");
    auto recognizer = TranslationRecognizer::FromConfig(translationConfig, audioConfig);
}
```

## <a name="translate-speech"></a>Traduction vocale

Pour effectuer une traduction vocale, le kit SDK Speech utilise un microphone ou une entrée de type fichier audio. La reconnaissance vocale se produit avant la traduction vocale. Une fois que tous les objets ont été initialisés, appelez la fonction recognize_once et récupérez le résultat.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    string fromLanguage = "en-US";
    string toLanguages[3] = { "it", "fr", "de" };
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    for (auto language : toLanguages) {
        translationConfig->AddTargetLanguage(language);
    }

    auto recognizer = TranslationRecognizer::FromConfig(translationConfig);
    cout << "Say something in '" << fromLanguage << "' and we'll translate...\n";

    auto result = recognizer->RecognizeOnceAsync().get();
    if (result->Reason == ResultReason::TranslatedSpeech)
    {
        cout << "Recognized: \"" << result->Text << "\"" << std::endl;
        for (auto pair : result->Translations)
        {
            auto language = pair.first;
            auto translation = pair.second;
            cout << "Translated into '" << language << "': " << translation << std::endl;
        }
    }
}
```

Pour plus d’informations sur la conversion de parole en texte, consultez [Informations de base sur la reconnaissance vocale](../../../get-started-speech-to-text.md).

## <a name="synthesize-translations"></a>Synthèse des traductions

À l’issue d’une reconnaissance et d’une traduction vocales, le résultat contient toutes les traductions dans un dictionnaire. La clé de dictionnaire [`Translations`][translations] correspond à la langue de traduction cible et la valeur au texte traduit. Le texte obtenu après reconnaissance vocale peut être traduit, puis synthétisé dans une autre langue (conversion de parole en parole).

### <a name="event-based-synthesis"></a>Synthèse basée sur les événements

L’objet `TranslationRecognizer` expose un événement `Synthesizing`. Celui-ci se déclenche plusieurs fois et fournit un mécanisme permettant de récupérer les données audio synthétisées à partir du résultat de la reconnaissance de traduction. Si vous effectuez une traduction dans plusieurs langues, consultez [Synthèse manuelle](#manual-synthesis). Spécifiez la voix de synthèse en affectant une propriété [`SetVoiceName`][voicename] et fournissez un gestionnaire d’événements pour l’événement `Synthesizing`, puis récupérez l’audio. L’exemple suivant enregistre l’audio traduit sous la forme d’un fichier *. wav*.

> [!IMPORTANT]
> La synthèse basée sur les événements fonctionne uniquement avec une seule traduction. N’ajoutez **pas** plusieurs langues de traduction cibles. En outre, la langue de la propriété [`SetVoiceName`][voicename] doit être identique à la langue de traduction cible, par exemple `"de"` peut être associé à `"de-DE-Hedda"`.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    auto fromLanguage = "en-US";
    auto toLanguage = "de";
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    translationConfig->AddTargetLanguage(toLanguage);

    // See: https://aka.ms/speech/sdkregion#standard-and-neural-voices
    translationConfig->SetVoiceName("de-DE-Hedda");

    auto recognizer = TranslationRecognizer::FromConfig(translationConfig);
    recognizer->Synthesizing.Connect([](const TranslationSynthesisEventArgs& e)
        {
            auto audio = e.Result->Audio;
            auto size = audio.size();
            cout << "Audio synthesized: " << size << " byte(s)" << (size == 0 ? "(COMPLETE)" : "") << std::endl;

            if (size > 0) {
                ofstream file("translation.wav", ios::out | ios::binary);
                auto audioData = audio.data();
                file.write((const char*)audioData, sizeof(audio[0]) * size);
                file.close();
            }
        });

    cout << "Say something in '" << fromLanguage << "' and we'll translate...\n";

    auto result = recognizer->RecognizeOnceAsync().get();
    if (result->Reason == ResultReason::TranslatedSpeech)
    {
        cout << "Recognized: \"" << result->Text << "\"" << std::endl;
        for (auto pair : result->Translations)
        {
            auto language = pair.first;
            auto translation = pair.second;
            cout << "Translated into '" << language << "': " << translation << std::endl;
        }
    }
}
```

### <a name="manual-synthesis"></a>Synthèse manuelle

Le dictionnaire [`Translations`][translations] peut être utilisé pour synthétiser l’audio à partir du texte de traduction. Bouclez sur les différentes traductions pour les synthétiser. Lors de la création d’une instance `SpeechSynthesizer`, la propriété [`SetSpeechSynthesisVoiceName`][speechsynthesisvoicename] de l’objet `SpeechConfig` doit être définie sur la voix souhaitée. L’exemple suivant traduit en cinq langues ; chaque traduction est ensuite synthétisée en un fichier audio dans le langage neuronal correspondant.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    auto fromLanguage = "en-US";
    auto toLanguages = { "de", "en", "it", "pt", "zh-Hans" };
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    for (auto language : toLanguages) {
        translationConfig->AddTargetLanguage(language);
    }

    auto recognizer = TranslationRecognizer::FromConfig(translationConfig);

    cout << "Say something in '" << fromLanguage << "' and we'll translate...\n";

    auto result = recognizer->RecognizeOnceAsync().get();
    if (result->Reason == ResultReason::TranslatedSpeech)
    {
        // See: https://aka.ms/speech/sdkregion#standard-and-neural-voices
        map<string, string> languageToVoiceMap;
        languageToVoiceMap["de"] = "de-DE-KatjaNeural";
        languageToVoiceMap["en"] = "en-US-AriaNeural";
        languageToVoiceMap["it"] = "it-IT-ElsaNeural";
        languageToVoiceMap["pt"] = "pt-BR-FranciscaNeural";
        languageToVoiceMap["zh-Hans"] = "zh-CN-XiaoxiaoNeural";

        cout << "Recognized: \"" << result->Text << "\"" << std::endl;
        for (auto pair : result->Translations)
        {
            auto language = pair.first;
            auto translation = pair.second;
            cout << "Translated into '" << language << "': " << translation << std::endl;

            auto speech_config =
                SpeechConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
            speech_config->SetSpeechSynthesisVoiceName(languageToVoiceMap[language]);

            auto audio_config = AudioConfig::FromWavFileOutput(language + "-translation.wav");
            auto synthesizer = SpeechSynthesizer::FromConfig(speech_config, audio_config);

            synthesizer->SpeakTextAsync(translation).get();
        }
    }
}
```

Pour plus d’informations sur la synthèse vocale, consultez [Informations de base sur la synthèse vocale](../../../get-started-text-to-speech.md).

[config]: /cpp/cognitive-services/speech/translation-speechtranslationconfig
[audioconfig]: /cpp/cognitive-services/speech/audio-audioconfig
[recognizer]: /cpp/cognitive-services/speech/translation-translationrecognizer
[recognitionlang]: /cpp/cognitive-services/speech/speechconfig#setspeechrecognitionlanguage
[addlang]: /cpp/cognitive-services/speech/translation-speechtranslationconfig#addtargetlanguage
[translations]: /cpp/cognitive-services/speech/translation-translationrecognitionresult#translations
[voicename]: /cpp/cognitive-services/speech/translation-speechtranslationconfig#setvoicename
[speechsynthesisvoicename]: /cpp/cognitive-services/speech/speechconfig#setspeechsynthesisvoicename