---
author: v-jaswel
ms.service: cognitive-services
ms.topic: include
ms.date: 10/07/2020
ms.author: v-jawe
ms.custom: references_regions
ms.openlocfilehash: a8a34a483390e12ba49123e26d2071d3aa667c89
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94425467"
---
Dans ce guide de démarrage rapide, vous allez découvrir les modèles de conception de base pour la reconnaissance de l’orateur à l’aide du SDK de reconnaissance vocale, notamment :

* Vérification dépendante ou indépendante du texte
* Identification de l’orateur pour identifier un échantillon vocal parmi un groupe de voix
* Suppression de profils vocaux

Pour obtenir une vue d’ensemble des concepts de reconnaissance vocale, consultez l’article de [présentation](../../../speaker-recognition-overview.md).

## <a name="skip-to-samples-on-github"></a>Passer aux exemples sur GitHub

Si vous souhaitez passer directement à l’exemple de code, consultez les [exemples de démarrage rapide JavaScript](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/fa6428a0837779cbeae172688e0286625e340942/quickstart/javascript/node/speaker-recognition) sur GitHub.

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous disposez d’un compte Azure et d’un abonnement au service Speech. Si vous n’avez pas de compte et d’abonnement, [essayez le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free).

> [!IMPORTANT]
> La fonctionnalité Reconnaissance de l’orateur est actuellement *uniquement* prise en charge dans les ressources Azure Speech créées dans la région `westus`.

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

## <a name="import-dependencies"></a>Importer les dépendances

Pour exécuter les exemples de cet article, ajoutez les instructions suivantes en haut de votre fichier .js.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="dependencies":::

Ces instructions importent les bibliothèques requises et récupèrent la clé et la région de votre abonnement au service Speech à partir de vos variables d’environnement. Elles spécifient également les chemins des fichiers audio que vous allez utiliser dans les tâches suivantes.

## <a name="create-helper-function"></a>Créer la fonction d’assistance

Ajoutez la fonction d’assistance suivante pour lire les fichiers audio dans les flux en vue d’une utilisation par le service Speech.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="helpers":::

Dans cette fonction, vous utilisez les méthodes [AudioInputStream.createPushStream](/javascript/api/microsoft-cognitiveservices-speech-sdk/audioinputstream?preserve-view=true&view=azure-node-latest#createpushstream-audiostreamformat-) et [AudioConfig.fromStreamInput](/javascript/api/microsoft-cognitiveservices-speech-sdk/audioconfig?preserve-view=true&view=azure-node-latest#fromstreaminput-audioinputstream---pullaudioinputstreamcallback-) pour créer un objet [AudioConfig](/javascript/api/microsoft-cognitiveservices-speech-sdk/audioconfig?preserve-view=true&view=azure-node-latest). Cet objet `AudioConfig` représente un flux audio. Vous utiliserez plusieurs de ces objets `AudioConfig` lors des tâches suivantes.

## <a name="text-dependent-verification"></a>Vérification dépendante du texte

La vérification de l’orateur est l’acte qui consiste à confirmer qu’un orateur correspond à une voix **inscrite** (connue). La première étape consiste à **inscrire** un profil vocal qui permet ensuite au service de comparer les futurs exemples de voix. Dans cet exemple, vous inscrivez le profil à l’aide d’une stratégie **dépendante du texte** , qui nécessite une phrase secrète spécifique à utiliser à la fois pour l’inscription et la vérification. Pour obtenir la liste des phrases secrètes prises en charge, consultez la [documentation de référence](/rest/api/speakerrecognition/).

### <a name="textdependentverification-function"></a>Fonction TextDependentVerification

Commencez par créer la fonction `TextDependentVerification`.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="text_dependent_verification":::

Cette fonction crée un objet [VoiceProfile](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofile?preserve-view=true&view=azure-node-latest) à l’aide de la méthode [VoiceProfileClient.createProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient?preserve-view=true&view=azure-node-latest#createprofileasync-voiceprofiletype--string---e--voiceprofile-----void---e--string-----void-). Notez qu’il existe trois [types](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofiletype?preserve-view=true&view=azure-node-latest) d’objets `VoiceProfile` :

- TextIndependentIdentification
- TextDependentVerification
- TextIndependentVerification

En l’occurrence, vous transmettez `VoiceProfileType.TextDependentVerification` à `VoiceProfileClient.createProfileAsync`.

Vous appelez ensuite deux fonctions d’assistance que vous allez définir ci-après, `AddEnrollmentsToTextDependentProfile` et `SpeakerVerify`. Enfin, appelez [VoiceProfileClient.deleteProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient?preserve-view=true&view=azure-node-latest#deleteprofileasync-voiceprofile---response--voiceprofileresult-----void---e--string-----void-) pour supprimer le profil.

### <a name="addenrollmentstotextdependentprofile-function"></a>Fonction AddEnrollmentsToTextDependentProfile

Définissez la fonction suivante pour inscrire un profil vocal.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="add_enrollments_dependent":::

Dans cette fonction, vous appelez la fonction `GetAudioConfigFromFile` que vous avez définie, pour créer des objets `AudioConfig` à partir d’échantillons audio. Ces échantillons audio contiennent une phrase secrète telle que « Ma voix est mon passeport, vérifiez-moi ». Vous inscrivez ensuite ces échantillons audio à l’aide de la méthode [VoiceProfileClient.enrollProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient?preserve-view=true&view=azure-node-latest#enrollprofileasync-voiceprofile--audioconfig---e--voiceprofileenrollmentresult-----void---e--string-----void-).

### <a name="speakerverify-function"></a>Fonction SpeakerVerify

Définissez `SpeakerVerify` comme suit.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="speaker_verify":::

Dans cette fonction, vous créez un objet [SpeakerVerificationModel](/javascript/api/microsoft-cognitiveservices-speech-sdk/speakerverificationmodel?preserve-view=true&view=azure-node-latest) avec la méthode [SpeakerVerificationModel.FromProfile](/javascript/api/microsoft-cognitiveservices-speech-sdk/speakerverificationmodel?preserve-view=true&view=azure-node-latest#fromprofile-voiceprofile-), en passant l’objet [VoiceProfile](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofile?preserve-view=true&view=azure-node-latest) que vous avez créé.

Ensuite, vous appelez la méthode [SpeechRecognizer.recognizeOnceAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer?preserve-view=true&view=azure-node-latest#recognizeonceasync--e--speechrecognitionresult-----void---e--string-----void-) pour valider un échantillon audio qui contient la même phrase secrète que les échantillons audio que vous avez inscrits. `SpeechRecognizer.recognizeOnceAsync` retourne un objet [SpeakerRecognitionResult](/javascript/api/microsoft-cognitiveservices-speech-sdk/speakerrecognitionresult?preserve-view=true&view=azure-node-latest), dont la propriété `score` contient un score de similarité compris entre 0,0 et 1,0. L’objet `SpeakerRecognitionResult` contient également une propriété `reason` de type [ResultReason](/javascript/api/microsoft-cognitiveservices-speech-sdk/resultreason?preserve-view=true&view=azure-node-latest). Si la vérification a réussi, la propriété `reason` doit avoir la valeur `RecognizedSpeaker`.

## <a name="text-independent-verification"></a>Vérification indépendante du texte

Contrairement à la vérification **dépendante du texte** , la vérification **indépendante du texte**  :

* Ne nécessite pas la lecture d’une phrase secrète. Le texte à prononcer est choisi par l’utilisateur.
* Ne nécessite pas trois échantillons audio, mais *nécessite* 20 secondes d’audio en tout.

### <a name="textindependentverification-function"></a>Fonction TextIndependentVerification

Commencez par créer la fonction `TextIndependentVerification`.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="text_independent_verification":::

À l’instar de la fonction `TextDependentVerification`, cette fonction crée un objet [VoiceProfile](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofile?preserve-view=true&view=azure-node-latest) avec la méthode [VoiceProfileClient.createProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient?preserve-view=true&view=azure-node-latest#createprofileasync-voiceprofiletype--string---e--voiceprofile-----void---e--string-----void-).

En l’occurrence, vous transmettez `VoiceProfileType.TextIndependentVerification` à `createProfileAsync`.

Vous appelez ensuite deux fonctions d’assistance : `AddEnrollmentsToTextIndependentProfile`, que vous allez définir ci-après et `SpeakerVerify`, que vous avez déjà définie. Enfin, appelez [VoiceProfileClient.deleteProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient?preserve-view=true&view=azure-node-latest#deleteprofileasync-voiceprofile---response--voiceprofileresult-----void---e--string-----void-) pour supprimer le profil.

### <a name="addenrollmentstotextindependentprofile"></a>AddEnrollmentsToTextIndependentProfile

Définissez la fonction suivante pour inscrire un profil vocal.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="add_enrollments_independent":::

Dans cette fonction, vous appelez la fonction `GetAudioConfigFromFile` que vous avez définie, pour créer des objets `AudioConfig` à partir d’échantillons audio. Vous inscrivez ensuite ces échantillons audio à l’aide de la méthode [VoiceProfileClient.enrollProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient?preserve-view=true&view=azure-node-latest#enrollprofileasync-voiceprofile--audioconfig---e--voiceprofileenrollmentresult-----void---e--string-----void-).

## <a name="speaker-identification"></a>Identification de l’orateur

L’identification de l’orateur est utilisée pour déterminer **qui** parle dans un groupe donné de voix inscrites. Le processus est similaire à la **vérification indépendante du texte**. La principale différence est la possibilité de vérifier plusieurs profils vocaux à la fois, au lieu d’un seul profil.

### <a name="textindependentidentification-function"></a>Fonction TextIndependentIdentification

Commencez par créer la fonction `TextIndependentIdentification`.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="text_independent_indentification":::

À l’instar des fonctions `TextDependentVerification` et `TextIndependentVerification`, cette fonction crée un objet [VoiceProfile](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofile?preserve-view=true&view=azure-node-latest) avec la méthode [VoiceProfileClient.createProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient?preserve-view=true&view=azure-node-latest#createprofileasync-voiceprofiletype--string---e--voiceprofile-----void---e--string-----void-).

En l’occurrence, vous transmettez `VoiceProfileType.TextIndependentIdentification` à `VoiceProfileClient.createProfileAsync`.

Vous appelez ensuite deux fonctions d’assistance : `AddEnrollmentsToTextIndependentProfile`, que vous avez déjà définie, et `SpeakerIdentify`, que vous allez définir ci-après. Enfin, appelez [VoiceProfileClient.deleteProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient?preserve-view=true&view=azure-node-latest#deleteprofileasync-voiceprofile---response--voiceprofileresult-----void---e--string-----void-) pour supprimer le profil.

### <a name="speakeridentify-function"></a>Fonction SpeakerIdentify

Définissez la fonction `SpeakerIdentify` comme suit.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="speaker_identify":::

Dans cette fonction, vous créez un objet [SpeakerIdentificationModel](/javascript/api/microsoft-cognitiveservices-speech-sdk/speakeridentificationmodel?preserve-view=true&view=azure-node-latest) avec la méthode [SpeakerIdentificationModel.fromProfiles](/javascript/api/microsoft-cognitiveservices-speech-sdk/speakeridentificationmodel?preserve-view=true&view=azure-node-latest#fromprofiles-voiceprofile---), en passant l’objet [VoiceProfile](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofile?preserve-view=true&view=azure-node-latest) que vous avez créé.

Ensuite, vous appelez la méthode [SpeechRecognizer.recognizeOnceAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer?preserve-view=true&view=azure-node-latest#recognizeonceasync--e--speechrecognitionresult-----void---e--string-----void-) et transmettez un échantillon audio.
`SpeechRecognizer.recognizeOnceAsync` tente d’identifier la voix pour cet échantillon audio en fonction des objets `VoiceProfile` que vous avez utilisés pour créer le `SpeakerIdentificationModel`. Elle retourne un objet [SpeakerRecognitionResult](/javascript/api/microsoft-cognitiveservices-speech-sdk/speakerrecognitionresult?preserve-view=true&view=azure-node-latest), dont la propriété `profileId` identifie le `VoiceProfile` correspondant éventuel, tandis que la propriété `score` contient un score de similarité compris entre 0,0 et 1,0.

## <a name="main-function"></a>Fonction main

Enfin, définissez la fonction `main` comme suit.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="main":::

Cette fonction crée un objet [VoiceProfileClient](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient?preserve-view=true&view=azure-node-latest), qui permet de créer, d’inscrire et de supprimer des profils vocaux. Elle appelle ensuite les fonctions que vous avez définies.