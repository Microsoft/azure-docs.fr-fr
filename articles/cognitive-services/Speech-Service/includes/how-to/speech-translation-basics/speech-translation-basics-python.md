---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 04/13/2020
ms.author: trbye
ms.openlocfilehash: ce542702f691c6cb2ab62f6f99b48763ab886bb7
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94425201"
---
L’une des principales fonctionnalités du service Speech est la possibilité de reconnaître la parole humaine et de la traduire dans d’autres langues. Dans ce démarrage rapide, vous allez apprendre à utiliser le Kit de développement logiciel (SDK) Speech dans vos applications et produits afin d’effectuer une traduction vocale de qualité. Ce démarrage rapide couvre les sujets suivants :

* Conversion de parole en texte
* Traduction de la parole en plusieurs langues cibles
* Réalisation d’une traduction directe de la parole en parole

## <a name="skip-to-samples-on-github"></a>Passer aux exemples sur GitHub

Si vous souhaitez passer directement à l’exemple de code, consultez les [exemples de démarrage rapide Python](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/python) sur GitHub.

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous disposez d’un compte Azure et d’un abonnement au service Speech. Si vous n’avez pas de compte et d’abonnement, [essayez le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Installer le Kit de développement logiciel (SDK) Speech

Avant de pouvoir faire quoi que ce soit, vous devez installer le SDK Speech. Selon votre plateforme, suivez les instructions fournies dans la section <a href="https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/speech-sdk#get-the-speech-sdk" target="_blank">Obtenir le kit SDK Speech <span class="docon docon-navigate-external x-hidden-focus"></span></a> de l’article _À propos du kit SDK Speech_.

## <a name="import-dependencies"></a>Importer les dépendances

Pour exécuter les exemples de cet article, ajoutez les instructions `import` suivantes en haut de votre fichier de code Python.

```python
import os
import azure.cognitiveservices.speech as speechsdk
```

## <a name="sensitive-data-and-environment-variables"></a>Données sensibles et variables d’environnement

L’exemple de code source de cet article dépend de variables d’environnement pour le stockage des données sensibles, par exemple la clé et la région de l’abonnement aux ressources Speech. Le fichier de code Python contient deux valeurs qui sont affectées à partir des variables d’environnement des ordinateurs hôtes, à savoir `SPEECH__SUBSCRIPTION__KEY` et `SPEECH__SERVICE__REGION`. Ces deux variables ont une portée globale, ce qui les rend accessibles dans la définition de fonction du fichier de code. Pour plus d’informations sur les variables d’environnement, consultez [Variables d’environnement et configuration de l’application](../../../../cognitive-services-security.md#environment-variables-and-application-configuration).

```python
speech_key, service_region = os.environ['SPEECH__SUBSCRIPTION__KEY'], os.environ['SPEECH__SERVICE__REGION']
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

```python
from_language, to_language = 'en-US', 'de'

def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)
```

## <a name="change-source-language"></a>Changer la langue source

L’une des tâches courantes de la traduction vocale consiste à spécifier la langue d’entrée (ou source). Examinons comment choisir l’italien comme langue d’entrée. Dans votre code, interagissez avec l’instance [`SpeechTranslationConfig`][config], en définissant sa propriété `speech_recognition_language`.

```python
def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    # Source (input) language
    translation_config.speech_recognition_language = from_language
```

La propriété [`speech_recognition_language`][recognitionlang] attend une chaîne au format langue-paramètres régionaux. Vous pouvez fournir n’importe quelle valeur figurant dans la colonne **Paramètres régionaux** dans la liste des [paramètres régionaux/langues](../../../language-support.md) pris en charge.

## <a name="add-translation-language"></a>Ajout d’une langue de traduction

Une autre tâche courante de traduction vocale consiste à spécifier les langues de traduction cibles. S’il en faut au moins une, il est aussi possible d’en indiquer plusieurs. Dans l’extrait de code suivant, le français et l’allemand sont définis comme langues de traduction cibles.

```python
def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = "it-IT"

    # Translate to languages. See, https://aka.ms/speech/sttt-languages
    translation_config.add_target_language("fr")
    translation_config.add_target_language("de")
```

À chaque appel à [`add_target_language`][addlang], une nouvelle langue de traduction cible est spécifiée. En d’autres termes, lorsqu’un message est reconnu dans la langue source, chaque traduction cible est disponible dans le cadre de l’opération de traduction qui en résulte.

## <a name="initialize-a-translation-recognizer"></a>Initialisation d’un moteur de reconnaissance de traduction

Une fois que vous avez créé une [`SpeechTranslationConfig`][config], l’étape suivante consiste à initialiser un [`TranslationRecognizer`][recognizer]. Quand vous initialisez un [`TranslationRecognizer`][recognizer], vous devez lui transmettre votre `translation_config`. L’objet de configuration fournit les informations d’identification dont le service Speech a besoin pour valider la demande.

Si vous effectuez une reconnaissance vocale en utilisant le microphone par défaut de votre appareil, voici à quoi doit ressembler le [`TranslationRecognizer`][recognizer] :

```python
def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    translation_config.add_target_language(to_language)

    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config)
```

Si vous souhaitez spécifier le périphérique d’entrée audio, vous devez créer une classe [`AudioConfig`][audioconfig] et fournir le paramètre `audio_config` lors de l’initialisation de votre [`TranslationRecognizer`][recognizer].

> [!TIP]
> [Découvrez comment obtenir l’ID de votre périphérique d’entrée audio](../../../how-to-select-audio-input-devices.md).

Tout d’abord, faites référence à l’objet `AudioConfig` de la façon suivante :

```python
def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    for lang in to_languages:
        translation_config.add_target_language(lang)

    audio_config = speechsdk.audio.AudioConfig(use_default_microphone=True)
    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config, audio_config=audio_config)
```

Si vous souhaitez fournir un fichier audio au lieu d’utiliser un microphone, vous devez quand même fournir un objet `audioConfig`. Toutefois, quand vous créerez un [`AudioConfig`][audioconfig], vous effectuerez l’appel avec `filename="path-to-file.wav"` (et non `use_default_microphone=True`) et transmettrez le paramètre `filename`.

```python
def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    for lang in to_languages:
        translation_config.add_target_language(lang)

    audio_config = speechsdk.audio.AudioConfig(filename="path-to-file.wav")
    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config, audio_config=audio_config)
```

## <a name="translate-speech"></a>Traduction vocale

Pour effectuer une traduction vocale, le kit SDK Speech utilise un microphone ou une entrée de type fichier audio. La reconnaissance vocale se produit avant la traduction vocale. Une fois que tous les objets ont été initialisés, appelez la fonction recognize_once et récupérez le résultat.

```python
import os
import azure.cognitiveservices.speech as speechsdk

speech_key, service_region = os.environ['SPEECH__SERVICE__KEY'], os.environ['SPEECH__SERVICE__REGION']
from_language, to_languages = 'en-US', 'de'

def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    translation_config.add_target_language(to_language)

    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config)
    
    print('Say something...')
    result = recognizer.recognize_once()
    print(get_result_text(reason=result.reason, result=result))

def get_result_text(reason, result):
    reason_format = {
        speechsdk.ResultReason.TranslatedSpeech:
            f'RECOGNIZED "{from_language}": {result.text}\n' +
            f'TRANSLATED into "{to_language}"": {result.translations[to_language]}',
        speechsdk.ResultReason.RecognizedSpeech: f'Recognized: "{result.text}"',
        speechsdk.ResultReason.NoMatch: f'No speech could be recognized: {result.no_match_details}',
        speechsdk.ResultReason.Canceled: f'Speech Recognition canceled: {result.cancellation_details}'
    }
    return reason_format.get(reason, 'Unable to recognize speech')

translate_speech_to_text()
```

Pour plus d’informations sur la conversion de parole en texte, consultez [Informations de base sur la reconnaissance vocale](../../../get-started-speech-to-text.md).

## <a name="synthesize-translations"></a>Synthèse des traductions

À l’issue d’une reconnaissance et d’une traduction vocales, le résultat contient toutes les traductions dans un dictionnaire. La clé de dictionnaire [`translations`][translations] correspond à la langue de traduction cible et la valeur au texte traduit. Le texte obtenu après reconnaissance vocale peut être traduit, puis synthétisé dans une autre langue (conversion de parole en parole).

### <a name="event-based-synthesis"></a>Synthèse basée sur les événements

L’objet `TranslationRecognizer` expose un événement `Synthesizing`. Celui-ci se déclenche plusieurs fois et fournit un mécanisme permettant de récupérer les données audio synthétisées à partir du résultat de la reconnaissance de traduction. Si vous effectuez une traduction dans plusieurs langues, consultez [Synthèse manuelle](#manual-synthesis). Spécifiez la voix de synthèse en affectant une propriété [`voice_name`][voicename] et fournissez un gestionnaire d’événements pour l’événement `Synthesizing`, puis récupérez l’audio. L’exemple suivant enregistre l’audio traduit sous la forme d’un fichier *. wav*.

> [!IMPORTANT]
> La synthèse basée sur les événements fonctionne uniquement avec une seule traduction. N’ajoutez **pas** plusieurs langues de traduction cibles. En outre, la langue de la propriété [`voice_name`][voicename] doit être identique à la langue de traduction cible, par exemple `"de"` peut être associé à `"de-DE-Hedda"`.

```python
import os
import azure.cognitiveservices.speech as speechsdk

speech_key, service_region = os.environ['SPEECH__SERVICE__KEY'], os.environ['SPEECH__SERVICE__REGION']
from_language, to_language = 'en-US', 'de'

def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    translation_config.add_target_language(to_language)

    # See: https://aka.ms/speech/sdkregion#standard-and-neural-voices
    translation_config.voice_name = "de-DE-Hedda"

    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config)

    def synthesis_callback(evt):
        size = len(evt.result.audio)
        print(f'Audio synthesized: {size} byte(s) {"(COMPLETED)" if size == 0 else ""}')

        if size > 0:
            file = open('translation.wav', 'wb+')
            file.write(evt.result.audio)
            file.close()

    recognizer.synthesizing.connect(synthesis_callback)

    print(f'Say something in "{from_language}" and we\'ll translate into "{to_language}".')

    result = recognizer.recognize_once()
    print(get_result_text(reason=result.reason, result=result))

def get_result_text(reason, result):
    reason_format = {
        speechsdk.ResultReason.TranslatedSpeech:
            f'Recognized "{from_language}": {result.text}\n' +
            f'Translated into "{to_language}"": {result.translations[to_language]}',
        speechsdk.ResultReason.RecognizedSpeech: f'Recognized: "{result.text}"',
        speechsdk.ResultReason.NoMatch: f'No speech could be recognized: {result.no_match_details}',
        speechsdk.ResultReason.Canceled: f'Speech Recognition canceled: {result.cancellation_details}'
    }
    return reason_format.get(reason, 'Unable to recognize speech')

translate_speech_to_text()
```

### <a name="manual-synthesis"></a>Synthèse manuelle

Le dictionnaire [`translations`][translations] peut être utilisé pour synthétiser l’audio à partir du texte de traduction. Bouclez sur les différentes traductions pour les synthétiser. Lors de la création d’une instance `SpeechSynthesizer`, la propriété [`speech_synthesis_voice_name`][speechsynthesisvoicename] de l’objet `SpeechConfig` doit être définie sur la voix souhaitée. L’exemple suivant traduit en cinq langues ; chaque traduction est ensuite synthétisée en un fichier audio dans le langage neuronal correspondant.

```python
import os
import azure.cognitiveservices.speech as speechsdk

speech_key, service_region = os.environ['SPEECH__SERVICE__KEY'], os.environ['SPEECH__SERVICE__REGION']
from_language, to_languages = 'en-US', [ 'de', 'en', 'it', 'pt', 'zh-Hans' ]

def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    for lang in to_languages:
        translation_config.add_target_language(lang)

    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config)
    
    print('Say something...')
    result = recognizer.recognize_once()
    synthesize_translations(result=result)

def synthesize_translations(result):
    language_to_voice_map = {
        "de": "de-DE-KatjaNeural",
        "en": "en-US-AriaNeural",
        "it": "it-IT-ElsaNeural",
        "pt": "pt-BR-FranciscaNeural",
        "zh-Hans": "zh-CN-XiaoxiaoNeural"
    }
    print(f'Recognized: "{result.text}"')

    for language in result.translations:
        translation = result.translations[language]
        print(f'Translated into "{language}": {translation}')

        speech_config = speechsdk.SpeechConfig(subscription=speech_key, region=service_region)
        speech_config.speech_synthesis_voice_name = language_to_voice_map.get(language)
        
        audio_config = speechsdk.audio.AudioOutputConfig(filename=f'{language}-translation.wav')
        speech_synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config, audio_config=audio_config)
        speech_synthesizer.speak_text_async(translation).get()

translate_speech_to_text()
```

Pour plus d’informations sur la synthèse vocale, consultez [Informations de base sur la synthèse vocale](../../../get-started-text-to-speech.md).

[config]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.translation.speechtranslationconfig?view=azure-python
[audioconfig]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.audio.audioconfig?view=azure-python
[recognizer]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.translation.translationrecognizer?view=azure-python
[recognitionlang]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig?view=azure-python
[addlang]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.translation.speechtranslationconfig?view=azure-python#add-target-language-language--str-
[translations]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.translation.translationrecognitionresult?view=azure-python#translations
[voicename]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.translation.speechtranslationconfig?view=azure-python#voice-name
[speechsynthesisvoicename]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig?view=azure-python#speech-synthesis-voice-name