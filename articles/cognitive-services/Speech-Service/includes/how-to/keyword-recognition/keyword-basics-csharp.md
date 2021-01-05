---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 11/03/2020
ms.author: trbye
ms.openlocfilehash: d71a7f6451cda5e2e50b5410140ac88361bf1735
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96509283"
---
Tout d’abord, chargez votre fichier de modèle de mot clé à l’aide de la fonction statique `FromFile()`, qui retourne un élément `KeywordRecognitionModel`. Utilisez le chemin du fichier `.table` que vous avez téléchargé à partir de Speech Studio. De plus, vous créez un fichier `AudioConfig` à l’aide du microphone par défaut, puis instanciez un nouveau `KeywordRecognizer` à l’aide de la configuration audio.

```csharp
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;

var keywordModel = KeywordRecognitionModel.FromFile("your/path/to/Activate_device.table");
using var audioConfig = AudioConfig.FromDefaultMicrophoneInput();
using var keywordRecognizer = new KeywordRecognizer(audioConfig);
```

Ensuite, l’exécution de la reconnaissance de mot clé s’effectue avec un appel à `RecognizeOnceAsync()` en transmettant votre objet de modèle. Cela démarre une session de reconnaissance de mots clés qui dure jusqu’à ce que le mot clé soit reconnu. Ainsi, vous utilisez généralement ce modèle de conception dans les applications multithread ou dans les cas d’utilisation où vous pouvez attendre indéfiniment un mot de mise en éveil.

```csharp
KeywordRecognitionResult result = await keywordRecognizer.RecognizeOnceAsync(keywordModel);
```

> [!NOTE]
> L’exemple indiqué ici utilise la reconnaissance de mot clé locale, car il ne demande pas d’objet `SpeechConfig` pour le contexte d’authentification et ne contacte pas le back-end. Toutefois, vous pouvez exécuter la reconnaissance de mot clé et la vérification [en utilisant une connexion continue au back-end](../../../tutorial-voice-enable-your-bot-speech-sdk.md#view-the-source-code-that-enables-keyword).