---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 10/20/2020
ms.author: trbye
ms.openlocfilehash: 7ab4d7708dd75ff2e07e77f3be4f996068704797
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92486777"
---
## <a name="install-the-speech-sdk"></a>Installer le Kit de développement logiciel (SDK) Speech

Avant de pouvoir faire quoi que ce soit, vous devez installer le <a href="https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk" target="_blank">SDK Speech pour JavaScript<span class="docon docon-navigate-external x-hidden-focus"></span></a>. Suivez les instructions ci-dessous, en fonction de votre plateforme :

- <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk?tabs=nodejs#get-the-speech-sdk" target="_blank">Node.js <span 
class="docon docon-navigate-external x-hidden-focus"></span></a>
- <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk?tabs=browser#get-the-speech-sdk" target="_blank">Navigateur web <span class="docon docon-navigate-external x-hidden-focus"></span></a>

## <a name="create-voice-signatures"></a>Créer des signatures vocales

La première étape consiste à créer des signatures vocales pour les orateurs à la conversation afin qu’ils puissent être identifiés en tant qu’orateurs uniques. Le fichier son `.wav` d’entrée permettant de créer des signatures vocales doit correspondre à 16 bits, à un taux d’échantillonnage de 16 kHz et à un format à canal unique (mono). La longueur recommandée de chaque échantillon audio se situe entre trente secondes et deux minutes. Le fichier `.wav` doit être un échantillon de voix **d’une personne** afin qu’un profil vocal unique puisse être créé.

L’exemple suivant montre comment créer une signature vocale à l’aide de l’[API REST](https://aka.ms/cts/signaturegenservice) en JavaScript. Notez que vous devez remplacer `subscriptionKey`, `region` et le chemin d’un fichier d’échantillon `.wav` par des valeurs réelles.

```javascript
const fs = require('fs');
const axios = require('axios');
const formData = require('form-data');
 
const subscriptionKey = 'your-subscription-key';
const region = 'your-region';
 
async function createProfile() {
    let form = new formData();
    form.append('file', fs.createReadStream('path-to-voice-sample.wav'));
    let headers = form.getHeaders();
    headers['Ocp-Apim-Subscription-Key'] = subscriptionKey;
 
    let url = `https://signature.${region}.cts.speech.microsoft.com/api/v1/Signature/GenerateVoiceSignatureFromFormData`;
    let response = await axios.post(url, form, { headers: headers });
    
    // get signature from response, serialize to json string
    return JSON.stringify(response.data.Signature);
}
 
async function main() {
    // use this voiceSignature string with conversation transcription calls below
    let voiceSignatureString = await createProfile();
    console.log(voiceSignatureString);
}
main();
```

L’exécution de ce script retourne une chaîne de signature vocale dans la variable `voiceSignatureString`. Exécutez la fonction à deux reprises afin d’utiliser deux chaînes comme entrée pour les variables `voiceSignatureStringUser1` et `voiceSignatureStringUser2` ci-dessous.

> [!NOTE]
> Les signatures vocales peuvent **uniquement** être créées à l’aide de l’API REST.

## <a name="transcribe-conversations"></a>Transcrire des conversations

L’exemple de code suivant montre comment transcrire les conversations en temps réel pour deux intervenants. Il part du principe que vous avez déjà créé des chaînes de signature vocale pour chaque intervenant, comme indiqué ci-dessus. Remplacez `subscriptionKey`, `region` et le chemin `filepath` pour le son que vous souhaitez transcrire par des informations réelles.

Cet exemple de code effectue les opérations suivantes :

* Crée un flux de transmission de type push à utiliser pour la transcription et écrit l’exemple de fichier `.wav` dans celui-ci.
* Crée une `Conversation` à l’aide de `createConversationAsync()`.
* Crée un `ConversationTranscriber` à l’aide du constructeur.
* Ajoute des participants à la conversation. Les chaînes `voiceSignatureStringUser1` et `voiceSignatureStringUser2` doivent être générées à partir des étapes ci-dessus.
* S’inscrit auprès des événements et commence la transcription.

```javascript
(function() {
    "use strict";
    var sdk = require("microsoft-cognitiveservices-speech-sdk");
    var fs = require("fs");
    
    var subscriptionKey = "your-subscription-key";
    var region = "your-region";
    var filepath = "audio-file-to-transcribe.wav"; // 8-channel audio
    
    // create the push stream and write file to it
    var pushStream = sdk.AudioInputStream.createPushStream();
    fs.createReadStream(filepath).on('data', function(arrayBuffer) {
        pushStream.write(arrayBuffer.slice());
    }).on('end', function() {
        pushStream.close();
    });
    
    var speechTranslationConfig = sdk.SpeechTranslationConfig.fromSubscription(subscriptionKey, region);
    var audioConfig = sdk.AudioConfig.fromStreamInput(pushStream);
    speechTranslationConfig.speechRecognitionLanguage = "en-US";
    
    // create conversation and transcriber
    var conversation = sdk.Conversation.createConversationAsync(speechTranslationConfig, "myConversation");
    var transcriber = new sdk.ConversationTranscriber(audioConfig);
    
    // attach the transcriber to the conversation
    transcriber.joinConversationAsync(conversation,
    function () {
        // add first participant using voiceSignature created in enrollment step
        var user1 = sdk.Participant.From("user1@example.com", "en-us", voiceSignatureStringUser1);
        conversation.addParticipantAsync(user1,
        function () {
            // add second participant using voiceSignature created in enrollment step
            var user2 = sdk.Participant.From("user2@example.com", "en-us", voiceSignatureStringUser2);
            conversation.addParticipantAsync(user2,
            function () {
                transcriber.sessionStarted = function(s, e) {
                console.log("(sessionStarted)");
                };
                transcriber.sessionStopped = function(s, e) {
                console.log("(sessionStopped)");
                };
                transcriber.canceled = function(s, e) {
                console.log("(canceled)");
                };
                transcriber.transcribed = function(s, e) {
                console.log("(transcribed) text: " + e.result.text);
                console.log("(transcribed) speakerId: " + e.result.speakerId);
                };
    
                // begin conversation transcription
                transcriber.startTranscribingAsync(
                function () { },
                function (err) {
                    console.trace("err - starting transcription: " + err);
                });
        },
        function (err) {
            console.trace("err - adding user1: " + err);
        });
    },
    function (err) {
        console.trace("err - adding user2: " + err);
    });
    },
    function (err) {
    console.trace("err - " + err);
    });
}()); 
```