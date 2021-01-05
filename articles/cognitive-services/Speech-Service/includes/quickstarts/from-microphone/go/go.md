---
author: trrwilson
ms.service: cognitive-services
ms.topic: include
ms.date: 05/25/2020
ms.author: trrwilson
ms.openlocfilehash: a18ce7b7c28b99967668bc33c5a94cbb58bfbc34
ms.sourcegitcommit: d95cab0514dd0956c13b9d64d98fdae2bc3569a0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91376692"
---
## <a name="prerequisites"></a>Prérequis

Avant de commencer :

> [!div class="checklist"]
> * [Créer une ressource Azure Speech](../../../../overview.md#try-the-speech-service-for-free)
> * [Configurer votre environnement de développement et créer un projet vide](../../../../quickstarts/setup-platform.md)
> * Veiller à avoir accès à un microphone pour la capture audio

## <a name="setup-your-environment"></a>Configurer votre environnement

Mettez à jour le fichier go.mod avec la dernière version du Kit de développement logiciel (SDK) en ajoutant cette ligne
```sh
require (
    github.com/Microsoft/cognitive-services-speech-sdk-go v1.13.0
)
```

## <a name="start-with-some-boilerplate-code"></a>Commencer avec du code réutilisable
1. Remplacez le contenu de votre fichier source (par exemple `sr-quickstart.go`) par le contenu ci-dessous, qui comprend :

- définition du package « main »
- importation des modules nécessaires à partir du Kit de développement logiciel (SDK) Speech
- variables pour le stockage des informations de l’abonnement qui seront remplacées ultérieurement dans ce guide de démarrage rapide
- implémentation simple utilisant le microphone pour l’entrée audio
- gestionnaires d’événements pour différents événements qui se produisent pendant une reconnaissance vocale

```sh
package recognizer

import (
    "bufio"
    "fmt"
    "os"

    "github.com/Microsoft/cognitive-services-speech-sdk-go/audio"
    "github.com/Microsoft/cognitive-services-speech-sdk-go/speech"
)

func recognizingHandler(event speech.SpeechRecognitionEventArgs) {
    defer event.Close()
    fmt.Println("Recognizing:", event.Result.Text)
}

func recognizedHandler(event speech.SpeechRecognitionEventArgs) {
    defer event.Close()
    fmt.Println("Recognized:", event.Result.Text)
}

func cancelledHandler(event speech.SpeechRecognitionCanceledEventArgs) {
    defer event.Close()
    fmt.Println("Received a cancellation: ", event.ErrorDetails)
}

func main() {
    subscription :=  "YOUR_SUBSCRIPTION_KEY"
    region := "YOUR_SUBSCRIPTIONKEY_REGION"

    audioConfig, err := audio.NewAudioConfigFromDefaultMicrophoneInput()
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer audioConfig.Close()
    config, err := speech.NewSpeechConfigFromSubscription(subscription, region)
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer config.Close()
    speechRecognizer, err := speech.NewSpeechRecognizerFromConfig(config, audioConfig)
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer speechRecognizer.Close()
    speechRecognizer.Recognizing(recognizingHandler)
    speechRecognizer.Recognized(recognizedHandler)
    speechRecognizer.Canceled(cancelledHandler)
    speechRecognizer.StartContinuousRecognitionAsync()
    defer speechRecognizer.StopContinuousRecognitionAsync()
    bufio.NewReader(os.Stdin).ReadBytes('\n')
}
```

Remplacez les valeurs `YOUR_SUBSCRIPTION_KEY` et `YOUR_SUBSCRIPTIONKEY_REGION` par les valeurs réelles provenant de la ressource Speech.

- Accédez au portail Azure et ouvrez la ressource Speech
- Sous **Clés** à gauche, se trouvent deux clés d’abonnement disponibles
    - Utilisez l’une ou l’autre en guise de remplacement de la valeur `YOUR_SUBSCRIPTION_KEY`
- Sous **Vue d’ensemble** à gauche, notez la région et mappez-la à l’identificateur de région
- Utilisez l’Identificateur de région en guise de remplacement de la valeur `YOUR_SUBSCRIPTIONKEY_REGION`, par exemple : `"westus"` pour **USA Ouest**

## <a name="code-explanation"></a>Explication du code
La région et la clé de l’abonnement Speech sont nécessaires pour créer un objet de configuration Speech. L’objet de configuration est nécessaire pour instancier un objet de reconnaissance vocale.

Le module de reconnaissance vocale expose plusieurs façons de reconnaître la parole. Dans cet exemple, la parole est reconnue de façon continue. Cette fonctionnalité permet au service Speech de savoir que vous envoyez plusieurs phrases pour reconnaissance, et quand le programme arrête la reconnaissance. À mesure que les résultats sont générés, le code les écrit dans la console.

## <a name="build-and-run"></a>Créer et exécuter
Vous êtes maintenant prêt à créer votre projet et à tester votre reconnaissance vocale à l’aide du service Speech.
1. Créez votre projet, par exemple. **« go build »**
2. Exécutez le module et prononcez une phrase ou quelques mots dans le micro de votre appareil. Votre production orale est transmise au service Speech, et transcrite en texte qui apparaît dans la sortie.


> [!NOTE]
> Le SDK Speech reconnaît par défaut l’utilisation de la langue en-US. Consultez [Spécifier la langue source pour la reconnaissance vocale](../../../../how-to-specify-source-language.md) pour plus d’informations sur le choix de la langue source.


## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [footer](./footer.md)]
