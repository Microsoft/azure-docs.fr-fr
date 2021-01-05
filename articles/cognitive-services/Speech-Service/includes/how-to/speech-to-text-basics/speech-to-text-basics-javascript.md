---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 04/15/2020
ms.author: trbye
ms.custom: devx-track-js
ms.openlocfilehash: 4e5e23c578d3c8ab72ae4b1483dc14c2161b9451
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96912256"
---
L’une des principales fonctionnalités du service de reconnaissance vocale est la possibilité de reconnaître et de transcrire la voix humaine (souvent appelée « reconnaissance vocale »). Dans ce guide de démarrage rapide, vous allez apprendre à utiliser le SDK de reconnaissance vocale dans vos applications et produits afin d’effectuer une conversion de voix en texte de qualité.

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
> Si vous ciblez un navigateur web et que vous utilisez l’étiquette `<script>`, le préfixe `sdk` n’est pas nécessaire lors du référencement de classes. Le préfixe `sdk` est un alias utilisé pour nommer le module `require`.

# <a name="require"></a>[require](#tab/require)

```javascript
const sdk = require("microsoft-cognitiveservices-speech-sdk");
```

Pour plus d’informations sur `require`, consultez la <a href="https://nodejs.org/en/knowledge/getting-started/what-is-require/" target="_blank">présentation de require <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

---

## <a name="create-a-speech-configuration"></a>Créer une configuration Speech

Pour appeler le service Speech à l’aide du SDK Speech, vous devez créer une classe [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig?preserve-view=true&view=azure-node-latest). Celle-ci comprend des informations sur votre abonnement, telles que votre clé et la région, le point de terminaison, l’hôte ou le jeton d’autorisation associés. Créez une [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig?preserve-view=true&view=azure-node-latest) à l’aide de votre clé et de votre région. Consultez la page [Rechercher les clés et la région](../../../overview.md#find-keys-and-region) pour rechercher votre paire clé-région.

```javascript
const speechConfig = sdk.SpeechConfig.fromSubscription("<paste-your-subscription-key>", "<paste-your-region>");
```

Vous pouvez initialiser une [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig?preserve-view=true&view=azure-node-latest) de plusieurs autres façons :

* Avec un point de terminaison : transmettez un point de terminaison de service Speech. Une clé ou un jeton d’autorisation est facultatif.
* Avec un hôte : transmettez une adresse d’hôte. Une clé ou un jeton d’autorisation est facultatif.
* Avec un jeton d’autorisation : transmettez un jeton d’autorisation et la région associée.

> [!NOTE]
> Quand vous procédez à une reconnaissance vocale, une synthèse vocale, une traduction ou une reconnaissance intentionnelle, vous devez toujours créer une configuration.

## <a name="recognize-from-microphone-browser-only"></a>Reconnaître la voix provenant d’un microphone (navigateur uniquement)

Pour reconnaître la voix à l’aide du microphone de votre appareil, créez une `AudioConfig` en utilisant `fromDefaultMicrophoneInput()`. Initialisez ensuite un [`SpeechRecognizer`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer?preserve-view=true&view=azure-node-latest), en transmettant vos `speechConfig` et `audioConfig`.

```javascript
const sdk = require("microsoft-cognitiveservices-speech-sdk");
const speechConfig = sdk.SpeechConfig.fromSubscription("<paste-your-subscription-key>", "<paste-your-region>");

function fromMic() {
    let audioConfig = sdk.AudioConfig.fromDefaultMicrophoneInput();
    let recognizer = new sdk.SpeechRecognizer(speechConfig, audioConfig);
    
    console.log('Speak into your microphone.');
    recognizer.recognizeOnceAsync(result => {
        console.log(`RECOGNIZED: Text=${result.text}`);
    });
}
fromMic();
```

Si vous souhaitez utiliser un périphérique d’entrée audio *spécifique*, vous devez spécifier son ID dans l’`AudioConfig`. Découvrez [comment obtenir l’ID](../../../how-to-select-audio-input-devices.md) de votre périphérique d’entrée audio.

## <a name="recognize-from-file"></a>Reconnaître la voix à partir d’un fichier 

# <a name="browser"></a>[Browser](#tab/browser)

Pour reconnaître la voix à partir d’un fichier audio dans un environnement JavaScript basé sur un navigateur, vous utilisez la fonction `fromWavFileInput()` afin de créer une [`AudioConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/audioconfig?preserve-view=true&view=azure-node-latest). La fonction `fromWavFileInput()` attend un objet [`File`](https://developer.mozilla.org/en-US/docs/Web/API/File/File) JavaScript en tant que paramètre.

```javascript
const sdk = require("microsoft-cognitiveservices-speech-sdk");
const speechConfig = sdk.SpeechConfig.fromSubscription("<paste-your-subscription-key>", "<paste-your-region>");

function fromFile() {
    // wavByteContent should be a byte array of the raw wav content
    let file = new File([wavByteContent]);
    let audioConfig = sdk.AudioConfig.fromWavFileInput(file);
    let recognizer = new sdk.SpeechRecognizer(speechConfig, audioConfig);
    
    recognizer.recognizeOnceAsync(result => {
        console.log(`RECOGNIZED: Text=${result.text}`);
    });
}
fromFile();
```

# <a name="nodejs"></a>[Node.JS](#tab/node)

Pour reconnaître la voix à partir d’un fichier audio dans Node.js, vous devez utiliser un modèle de conception alternatif à l’aide d’un flux d’envoi (push), car l’objet [`File`](https://developer.mozilla.org/en-US/docs/Web/API/File/File) JavaScript ne peut pas être utilisé dans un runtime Node.js. Le code suivant :

* Crée un flux d’envoi à l’aide de `createPushStream()`
* Ouvre le fichier `.wav` en créant un flux de lecture et l’écrit dans le flux d’envoi
* Crée une configuration audio à l’aide du flux d’envoi

```javascript
const fs = require('fs');
const sdk = require("microsoft-cognitiveservices-speech-sdk");
const speechConfig = sdk.SpeechConfig.fromSubscription("<paste-your-subscription-key>", "<paste-your-region>");

function fromFile() {
    let pushStream = sdk.AudioInputStream.createPushStream();

    fs.createReadStream("YourAudioFile.wav").on('data', function(arrayBuffer) {
        pushStream.write(arrayBuffer.slice());
    }).on('end', function() {
        pushStream.close();
    });
 
    let audioConfig = sdk.AudioConfig.fromStreamInput(pushStream);
    let recognizer = new sdk.SpeechRecognizer(speechConfig, audioConfig);
    recognizer.recognizeOnceAsync(result => {
        console.log(`RECOGNIZED: Text=${result.text}`);
        recognizer.close();
    });
}
fromFile();
```

L’utilisation d’un flux d’envoi comme entrée suppose que les données audio sont au format PCM brut (par exemple, les en-têtes sont ignorés).
L’API fonctionne toujours dans certains cas si l’en-tête n’a pas été ignoré, mais pour obtenir les meilleurs résultats, envisagez d’implémenter une logique permettant de lire les en-têtes afin que `fs` démarre au *début des données audio*.

---

## <a name="error-handling"></a>Gestion des erreurs

Les exemples précédents récupèrent simplement le texte reconnu de `result.text`, mais pour gérer les erreurs et autres réponses, vous devez écrire du code afin de gérer le résultat. Le code suivant évalue la propriété [`result.reason`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognitionresult?preserve-view=true&view=azure-node-latest#reason) et :

* Imprime le résultat de la reconnaissance : `ResultReason.RecognizedSpeech`
* S’il n’existe aucune correspondance de reconnaissance, informez l’utilisateur : `ResultReason.NoMatch`
* Si une erreur se produit, imprimez le message d’erreur : `ResultReason.Canceled`

```javascript
switch (result.reason) {
    case ResultReason.RecognizedSpeech:
        console.log(`RECOGNIZED: Text=${result.text}`);
        break;
    case ResultReason.NoMatch:
        console.log("NOMATCH: Speech could not be recognized.");
        break;
    case ResultReason.Canceled:
        const cancellation = CancellationDetails.fromResult(result);
        console.log(`CANCELED: Reason=${cancellation.reason}`);

        if (cancellation.reason == CancellationReason.Error) {
            console.log(`CANCELED: ErrorCode=${cancellation.ErrorCode}`);
            console.log(`CANCELED: ErrorDetails=${cancellation.errorDetails}`);
            console.log("CANCELED: Did you update the subscription info?");
        }
        break;
    }
```

## <a name="continuous-recognition"></a>Reconnaissance continue

Les exemples précédents utilisent la reconnaissance unique, qui reconnaît un énoncé unique. La fin d’un énoncé unique est déterminée par la détection du silence à la fin, ou après que 15 secondes d’audio ont été traitées.

En revanche, la reconnaissance continue est utilisée quand vous souhaitez **contrôler** à quel moment arrêter la reconnaissance. Pour obtenir les résultats de la reconnaissance, vous devez vous abonner aux événements `Recognizing`, `Recognized` et `Canceled`. Pour arrêter la reconnaissance, vous devez appeler [`stopContinuousRecognitionAsync`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer?preserve-view=true&view=azure-node-latest#stopcontinuousrecognitionasync). Voici un exemple de reconnaissance continue sur un fichier d’entrée audio.

Commencez par définir l’entrée et initialiser un [`SpeechRecognizer`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer?preserve-view=true&view=azure-node-latest) :

```javascript
const recognizer = new sdk.SpeechRecognizer(speechConfig);
```

Ensuite, abonnez-vous aux événements envoyés à partir du [`SpeechRecognizer`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer?preserve-view=true&view=azure-node-latest).

* [`recognizing`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer?preserve-view=true&view=azure-node-latest#recognizing) : signal pour les événements contenant des résultats de reconnaissance intermédiaires.
* [`recognized`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer?preserve-view=true&view=azure-node-latest#recognized) : signal pour les événements contenant des résultats de reconnaissance finaux (indiquant une tentative de reconnaissance réussie).
* [`sessionStopped`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer?preserve-view=true&view=azure-node-latest#sessionstopped) : signal pour les événements indiquant la fin d’une session de reconnaissance (opération).
* [`canceled`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer?preserve-view=true&view=azure-node-latest#canceled) : signal pour les événements contenant des résultats de reconnaissance annulés (indiquant une tentative de reconnaissance qui a été annulée suite une requête d’annulation directe ou à un échec de transport ou de protocole).

```javascript
recognizer.recognizing = (s, e) => {
    console.log(`RECOGNIZING: Text=${e.result.text}`);
};

recognizer.recognized = (s, e) => {
    if (e.result.reason == ResultReason.RecognizedSpeech) {
        console.log(`RECOGNIZED: Text=${e.result.text}`);
    }
    else if (e.result.reason == ResultReason.NoMatch) {
        console.log("NOMATCH: Speech could not be recognized.");
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

Quand tout est configuré, appelez [`startContinuousRecognitionAsync`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer?preserve-view=true&view=azure-node-latest#startcontinuousrecognitionasync) pour commencer la reconnaissance.

```javascript
recognizer.startContinuousRecognitionAsync();

// make the following call at some point to stop recognition.
// recognizer.StopContinuousRecognitionAsync();
```

### <a name="dictation-mode"></a>Mode de dictée

Quand vous utilisez la reconnaissance continue, vous pouvez activer le traitement de la dictée en utilisant la fonction « enable dictation » correspondante. Dans ce mode, l’instance de configuration vocale interprète les descriptions verbales de structures de phrase comme la ponctuation. Par exemple, l’énoncé « Habitez-vous en ville point d’interrogation » donne le texte « Habitez-vous en ville ? ».

Pour activer le mode dictée, utilisez la méthode [`enableDictation`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig?preserve-view=true&view=azure-node-latest#enabledictation--) sur votre [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig?preserve-view=true&view=azure-node-latest).

```javascript
speechConfig.enableDictation();
```

## <a name="change-source-language"></a>Changer la langue source

Une tâche courante pour la reconnaissance vocale consiste à spécifier la langue d’entrée (ou source). Examinons comment choisir l’italien comme langue d’entrée. Dans votre code, recherchez votre [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig?preserve-view=true&view=azure-node-latest), puis ajoutez cette ligne directement en dessous.

```javascript
speechConfig.speechRecognitionLanguage = "it-IT";
```

La propriété [`speechRecognitionLanguage`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig?preserve-view=true&view=azure-node-latest#speechrecognitionlanguage) attend une chaîne au format langue-paramètres régionaux. Vous pouvez fournir n’importe quelle valeur figurant dans la colonne **Paramètres régionaux** dans la liste des [paramètres régionaux/langues](../../../language-support.md) pris en charge.

## <a name="improve-recognition-accuracy"></a>Améliorer la justesse de la reconnaissance

Les listes d’expressions permettent d’identifier des expressions connues dans les données audio, comme le nom d’une personne ou un lieu spécifique. En fournissant une liste d’expressions, vous améliorez le niveau de justesse de la reconnaissance vocale.

Par exemple, si vous disposez d’une commande « Move to » et que parmi les destinations susceptibles d’être prononcées figurent « Ward », vous pouvez ajouter l’entrée « Move to Ward ». Ainsi, quand le contenu audio est reconnu, l’ajout de cette expression augmente la probabilité que « Move to Ward » sera reconnu et non « Move toward ».

Il est possible d’ajouter des mots uniques ou des expressions entières à une liste d’expressions. Pendant le processus de reconnaissance, une entrée de la liste d’expressions est utilisée pour renforcer la reconnaissance des mots et des expressions de la liste, même lorsque les entrées apparaissent au milieu de l’énoncé. 

> [!IMPORTANT]
> La fonctionnalité Liste d’expressions est disponible pour les langues suivantes : en-US, de-DE, en-AU, en-CA, en-GB, es-ES, es-MX, fr-CA, fr-FR, it-IT, ja-JP, ko-KR, pt-BR, zh-CN

Pour utiliser une liste d’expressions, commencez par créer un objet [`PhraseListGrammar`](/javascript/api/microsoft-cognitiveservices-speech-sdk/phraselistgrammar?preserve-view=true&view=azure-node-latest), puis ajoutez des mots et des expressions spécifiques avec [`addPhrase`](/javascript/api/microsoft-cognitiveservices-speech-sdk/phraselistgrammar?preserve-view=true&view=azure-node-latest#addphrase-string-).

Les modifications apportées à [`PhraseListGrammar`](/javascript/api/microsoft-cognitiveservices-speech-sdk/phraselistgrammar?preserve-view=true&view=azure-node-latest) prennent effet lors de la reconnaissance suivante ou après une reconnexion au service Speech.

```javascript
const phraseList = sdk.PhraseListGrammar.fromRecognizer(recognizer);
phraseList.addPhrase("Supercalifragilisticexpialidocious");
```

Si vous devez effacer votre liste d’expressions :

```javascript
phraseList.clear();
```

### <a name="other-options-to-improve-recognition-accuracy"></a>Autres options pour améliorer la justesse de la reconnaissance

Les listes d’expressions ne sont qu’une option parmi d’autres permettant d’améliorer la justesse de la reconnaissance. Vous pouvez également : 

* [Améliorer la justesse avec Custom Speech](../../../custom-speech-overview.md).
* [Améliorer la justesse avec des modèles de locataire](../../../tutorial-tenant-model.md).