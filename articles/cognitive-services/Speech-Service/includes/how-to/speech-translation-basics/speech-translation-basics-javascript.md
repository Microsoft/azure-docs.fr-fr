---
author: v-demjoh
ms.service: cognitive-services
ms.topic: include
ms.date: 07/14/2020
ms.author: v-demjoh
ms.custom: devx-track-js
ms.openlocfilehash: c7e9f1ee6a347caa71be2057905eb4ec6e6a1b0b
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94425249"
---
L’une des principales fonctionnalités du service Speech est la possibilité de reconnaître la parole humaine et de la traduire dans d’autres langues. Dans ce démarrage rapide, vous allez apprendre à utiliser le Kit de développement logiciel (SDK) Speech dans vos applications et produits afin d’effectuer une traduction vocale de qualité. Ce démarrage rapide couvre les sujets suivants :

* Conversion de parole en texte
* Traduction de la parole en plusieurs langues cibles
* Réalisation d’une traduction directe de la parole en parole

## <a name="skip-to-samples-on-github"></a>Passer aux exemples sur GitHub

Si vous souhaitez passer directement à l’exemple de code, consultez les [exemples de démarrage rapide JavaScript](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/node) sur GitHub.

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous disposez d’un compte Azure et d’un abonnement au service Speech. Si vous n’avez pas de compte et d’abonnement, [essayez le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Installer le Kit de développement logiciel (SDK) Speech

Avant de pouvoir faire quoi que ce soit, vous devez installer le <a href="https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk" target="_blank">SDK Speech pour JavaScript<span class="docon docon-navigate-external x-hidden-focus"></span></a>. Suivez les instructions ci-dessous, en fonction de votre plateforme :
- <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk?tabs=nodejs#get-the-speech-sdk" target="_blank">Node.js <span 
class="docon docon-navigate-external x-hidden-focus"></span></a>
- <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk?tabs=browser#get-the-speech-sdk" target="_blank">Navigateur web <span class="docon docon-navigate-external x-hidden-focus"></span></a>

De plus, selon l’environnement cible, utilisez l’un des éléments suivants :

# <a name="script"></a>[script](#tab/script)

Téléchargez et extrayez le fichier *microsoft.cognitiveservices.speech.sdk.bundle.js* du <a href="https://aka.ms/csspeech/jsbrowserpackage" target="_blank">SDK Speech pour JavaScript <span class="docon docon-navigate-external x-hidden-focus"></span></a>, puis placez-le dans un dossier accessible à votre fichier HTML.

```html
<script src="microsoft.cognitiveservices.speech.sdk.bundle.js"></script>;
```

> [!TIP]
> Si vous ciblez un navigateur web et que vous utilisez l’étiquette `<script>`, le préfixe `sdk` n’est pas nécessaire. Le préfixe `sdk` est un alias utilisé pour nommer le module `require`.

# <a name="import"></a>[import](#tab/import)

```javascript
import * from "microsoft-cognitiveservices-speech-sdk";
```

Pour plus d’informations sur `import`, consultez la page consacrée aux directives <a href="https://javascript.info/import-export" target="_blank">export et import <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

# <a name="require"></a>[require](#tab/require)

```javascript
const sdk = require("microsoft-cognitiveservices-speech-sdk");
```

Pour plus d’informations sur `require`, consultez la <a href="https://nodejs.org/en/knowledge/getting-started/what-is-require/" target="_blank">présentation de require <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

---

## <a name="create-a-translation-configuration"></a>Créer une configuration de traduction

Pour appeler le service de traduction à l’aide du kit SDK Speech, vous devez créer une classe [`SpeechTranslationConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechtranslationconfig?preserve-view=true&view=azure-node-latest). Celle-ci comprend des informations sur votre abonnement, telles que votre clé et la région, le point de terminaison, l’hôte ou le jeton d’autorisation associés.

> [!NOTE]
> Quand vous procédez à une reconnaissance vocale, une synthèse vocale, une traduction ou une reconnaissance intentionnelle, vous devez toujours créer une configuration.
Vous pouvez initialiser une [`SpeechTranslationConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechtranslationconfig?preserve-view=true&view=azure-node-latest) de plusieurs façons :

* Avec un abonnement : transmettez une clé et la région associée.
* Avec un point de terminaison : transmettez un point de terminaison de service Speech. Une clé ou un jeton d’autorisation est facultatif.
* Avec un hôte : transmettez une adresse d’hôte. Une clé ou un jeton d’autorisation est facultatif.
* Avec un jeton d’autorisation : transmettez un jeton d’autorisation et la région associée.

Examinons comment créer une classe [`SpeechTranslationConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechtranslationconfig?preserve-view=true&view=azure-node-latest) à l’aide d’une clé et d’une région. Pour obtenir ces informations d’identification, suivez les étapes décrites dans [Essayer le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free).

```javascript
const speechTranslationConfig = SpeechTranslationConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
```

## <a name="initialize-a-translator"></a>Initialiser un traducteur

Une fois que vous avez créé une [`SpeechTranslationConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechtranslationconfig?preserve-view=true&view=azure-node-latest), l’étape suivante consiste à initialiser un [`TranslationRecognizer`](/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?preserve-view=true&view=azure-node-latest). Quand vous initialisez un [`TranslationRecognizer`](/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?preserve-view=true&view=azure-node-latest), vous devez lui transmettre votre `speechTranslationConfig`. Cela fournit les informations d’identification dont le service de traduction a besoin pour valider votre demande.

Si vous traduisez une entrée vocale obtenue via le microphone par défaut de votre appareil, voici à quoi doit ressembler [`TranslationRecognizer`](/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?preserve-view=true&view=azure-node-latest) :

```javascript
const translator = new TranslationRecognizer(speechTranslationConfig);
```

Si vous souhaitez spécifier le périphérique d’entrée audio, vous devez créer une classe [`AudioConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/audioconfig?preserve-view=true&view=azure-node-latest) et fournir le paramètre `audioConfig` lors de l’initialisation de votre [`TranslationRecognizer`](/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?preserve-view=true&view=azure-node-latest).

> [!TIP]
> [Découvrez comment obtenir l’ID de votre périphérique d’entrée audio](../../../how-to-select-audio-input-devices.md).
Référencez l’objet `AudioConfig` comme suit :

```javascript
const audioConfig = AudioConfig.fromDefaultMicrophoneInput();
const recognizer = new TranslationRecognizer(speechTranslationConfig, audioConfig);
```

Si vous souhaitez fournir un fichier audio au lieu d’utiliser un microphone, vous devez quand même fournir un objet `audioConfig`. Toutefois, cette opération ne peut être effectuée que si vous ciblez **Node.js** , et quand vous créez un [`AudioConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/audioconfig?preserve-view=true&view=azure-node-latest), au lieu d’appeler `fromDefaultMicrophoneInput`, vous devez appeler `fromWavFileOutput` et passer le paramètre `filename`.

```javascript
const audioConfig = AudioConfig.fromWavFileInput("YourAudioFile.wav");
const recognizer = new TranslationRecognizer(speechTranslationConfig, audioConfig);
```

## <a name="translate-speech"></a>Traduction vocale

La [classe TranslationRecognizer](/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?preserve-view=true&view=azure-node-latest) pour le kit SDK Speech pour JavaScript expose quelques méthodes que vous pouvez utiliser pour la traduction vocale.

* Traduction unique (async) – Effectue la traduction en mode non bloquant (asynchrone). Cela permet de traduire un énoncé unique. La fin d’un énoncé unique est déterminée par la détection du silence à la fin, ou après que 15 secondes d’audio ont été traitées.
* Traduction continue (async) – Lance de façon asynchrone une opération de traduction continue. L’utilisateur s’inscrit à des événements et gère divers états d’application. Pour arrêter la traduction continue asynchrone, appelez [`stopContinuousRecognitionAsync`](/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?preserve-view=true&view=azure-node-latest#stopcontinuousrecognitionasync).

> [!NOTE]
> Pour en savoir plus sur la façon de choisir un mode de reconnaissance vocale, consultez [cet article](../../../get-started-speech-to-text.md).
### <a name="specify-a-target-language"></a>Spécifier une langue cible

Pour traduire, vous devez spécifier une langue source et au moins une langue cible.
Vous pouvez choisir une langue source à l’aide des paramètres régionaux indiqués dans la [Table de traduction vocale](../../../language-support.md#speech-translation). Recherchez dans le même lien vos options de langue cible de traduction. Vos options pour les langues cibles diffèrent lorsque vous souhaitez afficher du texte ou entendre la traduction synthétisée. Pour traduire de l’anglais en allemand, modifiez l’objet de configuration de traduction :

```javascript
speechTranslationConfig.speechRecognitionLanguage = "en-US";
speechTranslationConfig.addTargetLanguage("de");
```

### <a name="single-shot-recognition"></a>Reconnaissance unique

Voici un exemple de traduction asynchrone unique utilisant [`recognizeOnceAsync`](/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?preserve-view=true&view=azure-node-latest#recognizeonceasync) :

```javascript
recognizer.recognizeOnceAsync(result => {
    // Interact with result
});
```

Vous devrez écrire du code pour gérer le résultat. Cet exemple évalue [`result.reason`](/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognitionresult?preserve-view=true&view=azure-node-latest) pour une traduction en allemand :

```javascript
recognizer.recognizeOnceAsync(
  function (result) {
    let translation = result.translations.get("de");
    window.console.log(translation);
    recognizer.close();
  },
  function (err) {
    window.console.log(err);
    recognizer.close();
});
```

Votre code peut également gérer les mises à jour fournies pendant le traitement de la traduction.
Vous pouvez utiliser ces mises à jour pour fournir un retour visuel sur la progression de la traduction.
Consultez [cet exemple JavaScript Node.js](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/js/node/translation.js) pour obtenir un exemple de code qui affiche les mises à jour fournies pendant le processus de traduction. Le code suivant affiche également les détails générés pendant le processus de traduction.

```javascript
recognizer.recognizing = function (s, e) {
    var str = ("(recognizing) Reason: " + SpeechSDK.ResultReason[e.result.reason] +
            " Text: " +  e.result.text +
            " Translation:");
    str += e.result.translations.get("de");
    console.log(str);
};
recognizer.recognized = function (s, e) {
    var str = "\r\n(recognized)  Reason: " + SpeechSDK.ResultReason[e.result.reason] +
            " Text: " + e.result.text +
            " Translation:";
    str += e.result.translations.get("de");
    str += "\r\n";
    console.log(str);
};
```

### <a name="continuous-translation"></a>Traduction continue

La traduction continue est un peu plus complexe que la reconnaissance unique. Pour obtenir les résultats de la reconnaissance, vous devez vous abonner aux événements `recognizing`, `recognized` et `canceled`. Pour arrêter la traduction, vous devez appeler [`stopContinuousRecognitionAsync`](/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?preserve-view=true&view=azure-node-latest#stopcontinuousrecognitionasync). Voici un exemple de traduction continue sur un fichier d’entrée audio.

Commençons par définir l’entrée et initialiser un [`TranslationRecognizer`](/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?preserve-view=true&view=azure-node-latest) :

```javascript
const translator = new TranslationRecognizer(speechTranslationConfig);
```

Nous allons nous abonner aux événements envoyés à partir du [`TranslationRecognizer`](/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?preserve-view=true&view=azure-node-latest).

* [`recognizing`](/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?preserve-view=true&view=azure-node-latest#recognizing) : signal pour les événements contenant des résultats de traduction intermédiaires.
* [`recognized`](/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?preserve-view=true&view=azure-node-latest#recognized) : signal pour les événements contenant des résultats de traduction finaux (indiquant une tentative de traduction réussie).
* [`sessionStopped`](/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?preserve-view=true&view=azure-node-latest#sessionstopped) : signal pour les événements indiquant la fin d’une session (opération) de traduction.
* [`canceled`](/javascript/api/microsoft-cognitiveservices-speech-sdk/translationrecognizer?preserve-view=true&view=azure-node-latest#canceled) : signal pour les événements contenant les résultats d’une traduction annulée (indiquant une tentative de traduction qui a été annulée suite à une demande d’annulation directe ou à un échec de transport ou de protocole).

```javascript
recognizer.recognizing = (s, e) => {
    console.log(`TRANSLATING: Text=${e.result.text}`);
};
recognizer.recognized = (s, e) => {
    if (e.result.reason == ResultReason.RecognizedSpeech) {
        console.log(`TRANSLATED: Text=${e.result.text}`);
    }
    else if (e.result.reason == ResultReason.NoMatch) {
        console.log("NOMATCH: Speech could not be translated.");
    }
};
recognizer.canceled = (s, e) => {
    console.log(`CANCELED: Reason=${e.reason}`);
    if (e.reason == CancellationReason.Error) {
        console.log(`"CANCELED: ErrorCode=${e.errorCode}`);
        console.log(`"CANCELED: ErrorDetails=${e.errorDetails}`);
        console.log("CANCELED: Did you update the subscription info?");
    }
    recognizer.stopContinuousRecognitionAsync();
};
recognizer.sessionStopped = (s, e) => {
    console.log("\n    Session stopped event.");
    recognizer.stopContinuousRecognitionAsync();
};
```

Tout étant configuré, nous pouvons maintenant appeler [`startContinuousRecognitionAsync`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer?preserve-view=true&view=azure-node-latest#startcontinuousrecognitionasync).

```javascript
// Starts continuous recognition. Uses stopContinuousRecognitionAsync() to stop recognition.
recognizer.startContinuousRecognitionAsync();
// Something later can call, stops recognition.
// recognizer.StopContinuousRecognitionAsync();
```

## <a name="choose-a-source-language"></a>Choisir une langue source

Une des tâches courantes de la traduction vocale consiste à spécifier la langue d’entrée (ou source). Examinons comment choisir l’italien comme langue d’entrée. Dans votre code, recherchez votre [`SpeechTranslationConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechtranslationconfig?preserve-view=true&view=azure-node-latest), puis ajoutez la ligne suivante directement en dessous.

```javascript
speechTranslationConfig.speechRecognitionLanguage = "it-IT";
```

La propriété [`speechRecognitionLanguage`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechtranslationconfig?preserve-view=true&view=azure-node-latest#speechrecognitionlanguage) attend une chaîne au format langue-paramètres régionaux. Vous pouvez fournir n’importe quelle valeur figurant dans la colonne **Paramètres régionaux** dans la liste des [paramètres régionaux/langues](../../../language-support.md) pris en charge.

## <a name="choose-one-or-more-target-languages"></a>Choisir une ou plusieurs langues cibles

Le kit SDK Speech peut traduire en parallèle vers plusieurs langues cibles. Les langues cibles disponibles diffèrent légèrement de la liste de langues sources, et vous spécifiez les langues cibles avec un code de langue plutôt qu’avec les paramètres régionaux.
Consultez la liste des codes de langue pour les cibles de texte dans [la table de traduction vocale dans la page de prise en charge des langues](../../../language-support.md#speech-translation). Vous pouvez également y trouver des détails sur la traduction dans des langues synthétisées.

Le code suivant ajoute l’allemand comme langue cible :

```javascript
translationConfig.addTargetLanguage("de");
```

Étant donné que des traductions vers plusieurs langues cibles sont possibles, votre code doit spécifier la langue cible lors de l’examen du résultat. Le code suivant obtient des résultats de traduction pour l’allemand.

```javascript
recognizer.recognized = function (s, e) {
    var str = "\r\n(recognized)  Reason: " +
            sdk.ResultReason[e.result.reason] +
            " Text: " + e.result.text + " Translations:";
    var language = "de";
    str += " [" + language + "] " + e.result.translations.get(language);
    str += "\r\n";
    // show str somewhere
};
```