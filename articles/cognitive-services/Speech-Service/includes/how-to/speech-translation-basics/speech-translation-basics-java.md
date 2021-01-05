---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 04/13/2020
ms.custom: devx-track-java
ms.author: trbye
ms.openlocfilehash: 050ba52115a3d46963ea91095025b43ead03e5fc
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96026280"
---
L’une des principales fonctionnalités du service Speech est la possibilité de reconnaître la parole humaine et de la traduire dans d’autres langues. Dans ce démarrage rapide, vous allez apprendre à utiliser le Kit de développement logiciel (SDK) Speech dans vos applications et produits afin d’effectuer une traduction vocale de qualité. Ce démarrage rapide couvre les sujets suivants :

* Conversion de parole en texte
* Traduction de la parole en plusieurs langues cibles
* Réalisation d’une traduction directe de la parole en parole

## <a name="skip-to-samples-on-github"></a>Passer aux exemples sur GitHub

Si vous souhaitez passer directement à l’exemple de code, consultez les [exemples de démarrage rapide Java](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/jre/translate-speech-to-text) sur GitHub.

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous disposez d’un compte Azure et d’un abonnement au service Speech. Si vous n’avez pas de compte et d’abonnement, [essayez le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Installer le Kit de développement logiciel (SDK) Speech

Avant de pouvoir faire quoi que ce soit, vous devez installer le SDK Speech. Selon votre plateforme, suivez les instructions fournies dans la section <a href="https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/speech-sdk#get-the-speech-sdk" target="_blank">Obtenir le kit SDK Speech <span class="docon docon-navigate-external x-hidden-focus"></span></a> de l’article _À propos du kit SDK Speech_.

## <a name="import-dependencies"></a>Importer les dépendances

Pour exécuter les exemples de cet article, ajoutez les instructions `import` suivantes en haut de votre fichier de code * *.Java*.

```java
package speech;

import java.io.*;
import java.util.*;
import java.util.concurrent.*;
import com.microsoft.cognitiveservices.speech.*;
import com.microsoft.cognitiveservices.speech.audio.*;
import com.microsoft.cognitiveservices.speech.translation.*;
```

## <a name="sensitive-data-and-environment-variables"></a>Données sensibles et variables d’environnement

L’exemple de code source de cet article dépend de variables d’environnement pour le stockage des données sensibles, par exemple la clé et la région de l’abonnement aux ressources Speech. Le fichier de code Java contient deux valeurs `static final String` qui sont affectées à partir des variables d’environnement des ordinateurs hôtes, à savoir `SPEECH__SUBSCRIPTION__KEY` et `SPEECH__SERVICE__REGION`. Ces deux champs ont pour portée la classe, ce qui les rend accessibles dans le corps des méthodes de la classe. Pour plus d’informations sur les variables d’environnement, consultez [Variables d’environnement et configuration de l’application](../../../../cognitive-services-security.md#environment-variables-and-application-configuration).

```java
public class App {

    static final String SPEECH__SUBSCRIPTION__KEY = System.getenv("SPEECH__SUBSCRIPTION__KEY");
    static final String SPEECH__SERVICE__REGION = System.getenv("SPEECH__SERVICE__REGION");

    public static void main(String[] args) { }
}
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

Examinons comment créer une classe [`SpeechTranslationConfig`][config] à l’aide d’une clé et d’une région. Pour obtenir ces informations d’identification, procédez comme suit dans [Essayer le service vocal gratuitement](../../../overview.md#try-the-speech-service-for-free).

```java
public class App {

    static final String SPEECH__SUBSCRIPTION__KEY = System.getenv("SPEECH__SERVICE__KEY");
    static final String SPEECH__SERVICE__REGION = System.getenv("SPEECH__SERVICE__REGION");

    public static void main(String[] args) {
        try {
            translateSpeech();
            System.exit(0);
        } catch (Exception ex) {
            System.out.println(ex);
            System.exit(1);
        }
    }

    static void translateSpeech() {
        SpeechTranslationConfig config = SpeechTranslationConfig.fromSubscription(
            SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
    }
}
```

## <a name="change-source-language"></a>Changer la langue source

L’une des tâches courantes de la traduction vocale consiste à spécifier la langue d’entrée (ou source). Examinons comment choisir l’italien comme langue d’entrée. Dans votre code, interagissez avec l’instance [`SpeechTranslationConfig`][config], en appelant la méthode `setSpeechRecognitionLanguage`.

```java
static void translateSpeech() {
    SpeechTranslationConfig translationConfig = SpeechTranslationConfig.fromSubscription(
        SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
    
    // Source (input) language
    translationConfig.setSpeechRecognitionLanguage("it-IT");
}
```

La fonction [`setSpeechRecognitionLanguage`][recognitionlang] attend une chaîne au format langue-paramètres régionaux. Vous pouvez fournir n’importe quelle valeur figurant dans la colonne **Paramètres régionaux** dans la liste des [paramètres régionaux/langues](../../../language-support.md) pris en charge.

## <a name="add-translation-language"></a>Ajout d’une langue de traduction

Une autre tâche courante de traduction vocale consiste à spécifier les langues de traduction cibles. S’il en faut au moins une, il est aussi possible d’en indiquer plusieurs. Dans l’extrait de code suivant, le français et l’allemand sont définis comme langues de traduction cibles.

```java
static void translateSpeech() {
    SpeechTranslationConfig translationConfig = SpeechTranslationConfig.fromSubscription(
        SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
    
    translationConfig.setSpeechRecognitionLanguage("it-IT");

    // Translate to languages. See, https://aka.ms/speech/sttt-languages
    translationConfig.addTargetLanguage("fr");
    translationConfig.addTargetLanguage("de");
}
```

À chaque appel à [`addTargetLanguage`][addlang], une nouvelle langue de traduction cible est spécifiée. En d’autres termes, lorsqu’un message est reconnu dans la langue source, chaque traduction cible est disponible dans le cadre de l’opération de traduction qui en résulte.

## <a name="initialize-a-translation-recognizer"></a>Initialisation d’un moteur de reconnaissance de traduction

Une fois que vous avez créé une [`SpeechTranslationConfig`][config], l’étape suivante consiste à initialiser un [`TranslationRecognizer`][recognizer]. Quand vous initialisez un [`TranslationRecognizer`][recognizer], vous devez lui transmettre votre `translationConfig`. L’objet de configuration fournit les informations d’identification dont le service Speech a besoin pour valider la demande.

Si vous effectuez une reconnaissance vocale en utilisant le microphone par défaut de votre appareil, voici à quoi doit ressembler le [`TranslationRecognizer`][recognizer] :

```java
static void translateSpeech() {
    SpeechTranslationConfig translationConfig = SpeechTranslationConfig.fromSubscription(
        SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
    
    String fromLanguage = "en-US";
    String[] toLanguages = { "it", "fr", "de" };
    translationConfig.setSpeechRecognitionLanguage(fromLanguage);
    for (String language : toLanguages) {
        translationConfig.addTargetLanguage(language);
    }

    try (TranslationRecognizer recognizer = new TranslationRecognizer(translationConfig)) {
    }
}
```

Si vous souhaitez spécifier le périphérique d’entrée audio, vous devez créer une classe [`AudioConfig`][audioconfig] et fournir le paramètre `audioConfig` lors de l’initialisation de votre [`TranslationRecognizer`][recognizer].

> [!TIP]
> [Découvrez comment obtenir l’ID de votre périphérique d’entrée audio](../../../how-to-select-audio-input-devices.md).

Tout d’abord, faites référence à l’objet `AudioConfig` de la façon suivante :

```java
static void translateSpeech() {
    SpeechTranslationConfig translationConfig = SpeechTranslationConfig.fromSubscription(
        SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
    
    String fromLanguage = "en-US";
    String[] toLanguages = { "it", "fr", "de" };
    translationConfig.setSpeechRecognitionLanguage(fromLanguage);
    for (String language : toLanguages) {
        translationConfig.addTargetLanguage(language);
    }

    AudioConfig audioConfig = AudioConfig.fromDefaultMicrophoneInput();
    try (TranslationRecognizer recognizer = new TranslationRecognizer(translationConfig, audioConfig)) {
        
    }
}
```

Si vous souhaitez fournir un fichier audio au lieu d’utiliser un microphone, vous devez quand même fournir un objet `audioConfig`. Toutefois, quand vous créerez une classe [`AudioConfig`][audioconfig], au lieu d’appeler `fromDefaultMicrophoneInput`, vous appellerez `fromWavFileInput` et transmettrez le paramètre `filename`.

```java
static void translateSpeech() {
    SpeechTranslationConfig translationConfig = SpeechTranslationConfig.fromSubscription(
        SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
    
    String fromLanguage = "en-US";
    String[] toLanguages = { "it", "fr", "de" };
    translationConfig.setSpeechRecognitionLanguage(fromLanguage);
    for (String language : toLanguages) {
        translationConfig.addTargetLanguage(language);
    }

    AudioConfig audioConfig = AudioConfig.fromWavFileInput("YourAudioFile.wav");
    try (TranslationRecognizer recognizer = new TranslationRecognizer(translationConfig, audioConfig)) {
        
    }
}
```

## <a name="translate-speech"></a>Traduction vocale

Pour effectuer une traduction vocale, le kit SDK Speech utilise un microphone ou une entrée de type fichier audio. La reconnaissance vocale se produit avant la traduction vocale. Une fois que tous les objets ont été initialisés, appelez la fonction recognize_once et récupérez le résultat.

```java
static void translateSpeech() throws ExecutionException, InterruptedException {
    SpeechTranslationConfig translationConfig = SpeechTranslationConfig.fromSubscription(
        SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
    
    String fromLanguage = "en-US";
    String[] toLanguages = { "it", "fr", "de" };
    translationConfig.setSpeechRecognitionLanguage(fromLanguage);
    for (String language : toLanguages) {
        translationConfig.addTargetLanguage(language);
    }

    try (TranslationRecognizer recognizer = new TranslationRecognizer(translationConfig)) {
        System.out.printf("Say something in '%s' and we'll translate...", fromLanguage);

        TranslationRecognitionResult result = recognizer.recognizeOnceAsync().get();
        if (result.getReason() == ResultReason.TranslatedSpeech) {
            System.out.printf("Recognized: \"%s\"\n", result.getText());
            for (Map.Entry<String, String> pair : result.getTranslations().entrySet()) {
                System.out.printf("Translated into '%s': %s\n", pair.getKey(), pair.getValue());
            }
        }
    }
}
```

Pour plus d’informations sur la conversion de parole en texte, consultez [Informations de base sur la reconnaissance vocale](../../../get-started-speech-to-text.md).

## <a name="synthesize-translations"></a>Synthèse des traductions

À l’issue d’une reconnaissance et d’une traduction vocales, le résultat contient toutes les traductions dans un dictionnaire. La fonction [`getTranslations`][translations] retourne un dictionnaire dont la clé correspond à la langue de traduction cible et la valeur au texte traduit. Le texte obtenu après reconnaissance vocale peut être traduit, puis synthétisé dans une autre langue (conversion de parole en parole).

### <a name="event-based-synthesis"></a>Synthèse basée sur les événements

L’objet `TranslationRecognizer` expose un événement `synthesizing`. Celui-ci se déclenche plusieurs fois et fournit un mécanisme permettant de récupérer les données audio synthétisées à partir du résultat de la reconnaissance de traduction. Si vous effectuez une traduction dans plusieurs langues, consultez [Synthèse manuelle](#manual-synthesis). Spécifiez la voix de synthèse en affectant une propriété [`setVoiceName`][voicename] et fournissez un gestionnaire d’événements pour l’événement `synthesizing`, puis récupérez l’audio. L’exemple suivant enregistre l’audio traduit sous la forme d’un fichier *. wav*.

> [!IMPORTANT]
> La synthèse basée sur les événements fonctionne uniquement avec une seule traduction. N’ajoutez **pas** plusieurs langues de traduction cibles. En outre, la langue de la propriété [`setVoiceName`][voicename] doit être identique à la langue de traduction cible, par exemple `"de"` peut être associé à `"de-DE-Hedda"`.

```java
static void translateSpeech() throws ExecutionException, FileNotFoundException, InterruptedException, IOException {
    SpeechTranslationConfig translationConfig = SpeechTranslationConfig.fromSubscription(
        SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    String fromLanguage = "en-US";
    String toLanguage = "de";
    translationConfig.setSpeechRecognitionLanguage(fromLanguage);
    translationConfig.addTargetLanguage(toLanguage);

    // See: https://aka.ms/speech/sdkregion#standard-and-neural-voices
    translationConfig.setVoiceName("de-DE-Hedda");

    try (TranslationRecognizer recognizer = new TranslationRecognizer(translationConfig)) {
        recognizer.synthesizing.addEventListener((s, e) -> {
            byte[] audio = e.getResult().getAudio();
            int size = audio.length;
            System.out.println("Audio synthesized: " + size + " byte(s)" + (size == 0 ? "(COMPLETE)" : ""));

            if (size > 0) {
                try (FileOutputStream file = new FileOutputStream("translation.wav")) {
                    file.write(audio);
                } catch (IOException ex) {
                    ex.printStackTrace();
                }
            }
        });

        System.out.printf("Say something in '%s' and we'll translate...", fromLanguage);

        TranslationRecognitionResult result = recognizer.recognizeOnceAsync().get();
        if (result.getReason() == ResultReason.TranslatedSpeech) {
            System.out.printf("Recognized: \"%s\"\n", result.getText());
            for (Map.Entry<String, String> pair : result.getTranslations().entrySet()) {
                String language = pair.getKey();
                String translation = pair.getValue();
                System.out.printf("Translated into '%s': %s\n", language, translation);
            }
        }
    }
}
```

### <a name="manual-synthesis"></a>Synthèse manuelle

La fonction [`getTranslations`][translations] retourne un dictionnaire qui peut être utilisé pour synthétiser l’audio à partir du texte de traduction. Bouclez sur les différentes traductions pour les synthétiser. Lors de la création d’une instance `SpeechSynthesizer`, la propriété [`setSpeechSynthesisVoiceName`][speechsynthesisvoicename] de l’objet `SpeechConfig` doit être définie sur la voix souhaitée. L’exemple suivant traduit en cinq langues ; chaque traduction est ensuite synthétisée en un fichier audio dans le langage neuronal correspondant.

```java
static void translateSpeech() throws ExecutionException, InterruptedException {
    SpeechTranslationConfig translationConfig = SpeechTranslationConfig.fromSubscription(
        SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
    
    String fromLanguage = "en-US";
    String[] toLanguages = { "de", "en", "it", "pt", "zh-Hans" };
    translationConfig.setSpeechRecognitionLanguage(fromLanguage);
    for (String language : toLanguages) {
        translationConfig.addTargetLanguage(language);
    }

    try (TranslationRecognizer recognizer = new TranslationRecognizer(translationConfig)) {
        System.out.printf("Say something in '%s' and we'll translate...", fromLanguage);

        TranslationRecognitionResult result = recognizer.recognizeOnceAsync().get();
        if (result.getReason() == ResultReason.TranslatedSpeech) {
            // See: https://aka.ms/speech/sdkregion#standard-and-neural-voices
            Map<String, String> languageToVoiceMap = new HashMap<String, String>();
            languageToVoiceMap.put("de", "de-DE-KatjaNeural");
            languageToVoiceMap.put("en", "en-US-AriaNeural");
            languageToVoiceMap.put("it", "it-IT-ElsaNeural");
            languageToVoiceMap.put("pt", "pt-BR-FranciscaNeural");
            languageToVoiceMap.put("zh-Hans", "zh-CN-XiaoxiaoNeural");

            System.out.printf("Recognized: \"%s\"\n", result.getText());
            for (Map.Entry<String, String> pair : result.getTranslations().entrySet()) {
                String language = pair.getKey();
                String translation = pair.getValue();
                System.out.printf("Translated into '%s': %s\n", language, translation);

                SpeechConfig speechConfig =
                    SpeechConfig.fromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
                speechConfig.setSpeechSynthesisVoiceName(languageToVoiceMap.get(language));

                AudioConfig audioConfig = AudioConfig.fromWavFileOutput(language + "-translation.wav");
                try (SpeechSynthesizer synthesizer = new SpeechSynthesizer(speechConfig, audioConfig)) {
                    synthesizer.SpeakTextAsync(translation).get();
                }
            }
        }
    }
}
```

Pour plus d’informations sur la synthèse vocale, consultez [Informations de base sur la synthèse vocale](../../../get-started-text-to-speech.md).

[config]: /java/api/com.microsoft.cognitiveservices.speech.translation.SpeechTranslationConfig
[audioconfig]: /java/api/com.microsoft.cognitiveservices.speech.audio.AudioConfig
[recognizer]: /java/api/com.microsoft.cognitiveservices.speech.translation.TranslationRecognizer
[recognitionlang]: /java/api/com.microsoft.cognitiveservices.speech.speechconfig.setspeechrecognitionlanguage
[addlang]: /java/api/com.microsoft.cognitiveservices.speech.translation.speechtranslationconfig.addtargetlanguage
[translations]: /java/api/com.microsoft.cognitiveservices.speech.translation.translationrecognitionresult.gettranslations
[voicename]: /java/api/com.microsoft.cognitiveservices.speech.translation.speechtranslationconfig.setvoicename
[speechsynthesisvoicename]: /java/api/com.microsoft.cognitiveservices.speech.speechconfig.setspeechsynthesisvoicename