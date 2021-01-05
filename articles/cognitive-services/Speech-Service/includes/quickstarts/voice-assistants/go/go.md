---
author: trrwilson
ms.service: cognitive-services
ms.topic: include
ms.date: 05/25/2020
ms.author: trrwilson
ms.openlocfilehash: c37637ba1cb5154bbd56f7fd22bbbca62b8c3438
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96187628"
---
## <a name="prerequisites"></a>Prérequis

Avant de commencer :

> [!div class="checklist"]
> * [Créer une ressource Azure Speech](../../../../overview.md#try-the-speech-service-for-free)
> * [Configurer votre environnement de développement et créer un projet vide](../../../../quickstarts/setup-platform.md)
> * Créer un bot connecté au [canal Direct Line Speech](/azure/bot-service/bot-service-channel-connect-directlinespeech)
> * Veiller à avoir accès à un microphone pour la capture audio
>
  > [!NOTE]
  > Reportez-vous à [la liste des régions prises en charge pour les assistants vocaux](~/articles/cognitive-services/speech-service/regions.md#voice-assistants) et vérifiez que vos ressources sont déployées dans une de ces régions.

## <a name="setup-your-environment"></a>Configurer votre environnement

Mettez à jour le fichier go.mod avec la dernière version du Kit de développement logiciel (SDK) en ajoutant cette ligne
```sh
require (
    github.com/Microsoft/cognitive-services-speech-sdk-go v1.14.0
)
```

## <a name="start-with-some-boilerplate-code"></a>Commencer avec du code réutilisable
Remplacez le contenu de votre fichier source (par exemple `quickstart.go`) par le contenu ci-dessous, qui comprend :

- définition du package « main »
- importation des modules nécessaires à partir du Kit de développement logiciel (SDK) Speech
- variables pour le stockage des informations du bot qui seront remplacées ultérieurement dans ce guide de démarrage rapide
- implémentation simple utilisant le microphone pour l’entrée audio
- gestionnaires d’événements pour différents événements qui se produisent pendant une interaction vocale

```sh
package main

import (
    "fmt"
    "time"

    "github.com/Microsoft/cognitive-services-speech-sdk-go/audio"
    "github.com/Microsoft/cognitive-services-speech-sdk-go/dialog"
    "github.com/Microsoft/cognitive-services-speech-sdk-go/speech"
)

func main() {
    subscription :=  "YOUR_SUBSCRIPTION_KEY"
    region := "YOUR_BOT_REGION"

    audioConfig, err := audio.NewAudioConfigFromDefaultMicrophoneInput()
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer audioConfig.Close()
    config, err := dialog.NewBotFrameworkConfigFromSubscription(subscription, region)
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer config.Close()
    connector, err := dialog.NewDialogServiceConnectorFromConfig(config, audioConfig)
    if err != nil {
        fmt.Println("Got an error: ", err)
        return
    }
    defer connector.Close()
    activityReceivedHandler := func(event dialog.ActivityReceivedEventArgs) {
        defer event.Close()
        fmt.Println("Received an activity.")
    }
    connector.ActivityReceived(activityReceivedHandler)
    recognizedHandle := func(event speech.SpeechRecognitionEventArgs) {
        defer event.Close()
        fmt.Println("Recognized ", event.Result.Text)
    }
    connector.Recognized(recognizedHandle)
    recognizingHandler := func(event speech.SpeechRecognitionEventArgs) {
        defer event.Close()
        fmt.Println("Recognizing ", event.Result.Text)
    }
    connector.Recognizing(recognizingHandler)
    connector.ListenOnceAsync()
    <-time.After(10 * time.Second)
}
```

Remplacez les valeurs `YOUR_SUBSCRIPTION_KEY` et `YOUR_BOT_REGION` par les valeurs réelles provenant de la ressource Speech.

- Accédez au portail Azure et ouvrez votre ressource Speech
- Sous **Clés et point de terminaison** à gauche, se trouvent deux clés d’abonnement disponibles
    - Utilisez l’une ou l’autre en guise de remplacement de la valeur `YOUR_SUBSCRIPTION_KEY`
- Sous **Vue d’ensemble** à gauche, notez la région et mappez-la à l’identificateur de région
    - Utilisez l’Identificateur de région en guise de remplacement de la valeur `YOUR_BOT_REGION`, par exemple : `"westus"` pour **USA Ouest**

   > [!NOTE]
   > Reportez-vous à [la liste des régions prises en charge pour les assistants vocaux](~/articles/cognitive-services/speech-service/regions.md#voice-assistants) et vérifiez que vos ressources sont déployées dans une de ces régions.

   > [!NOTE]
   > Pour plus d’informations sur la configuration de votre bot, consultez la documentation de Bot Framework relative au [canal Direct Line Speech](/azure/bot-service/bot-service-channel-connect-directlinespeech).

## <a name="code-explanation"></a>Explication du code
La région et la clé de l’abonnement Speech sont nécessaires pour créer un objet de configuration Speech. L’objet de configuration est nécessaire pour instancier un objet de reconnaissance vocale.

Le module de reconnaissance vocale expose plusieurs façons de reconnaître la parole. Dans cet exemple, la parole est reconnue de façon continue. Cette fonctionnalité permet au service Speech de savoir que vous envoyez plusieurs phrases pour reconnaissance, et quand le programme arrête la reconnaissance. À mesure que les résultats sont générés, le code les écrit dans la console.

## <a name="build-and-run"></a>Créer et exécuter
Vous êtes maintenant prêt à créer votre projet et à tester votre assistant vocal personnalisé à l’aide du service Speech.
1. Créez votre projet, par exemple. **« go build »**
2. Exécutez le module et prononcez une phrase ou quelques mots dans le micro de votre appareil. Votre production orale est transmise au canal Direct Line Speech, puis transcrite en texte qui apparaît comme sortie.


> [!NOTE]
> Le SDK Speech reconnaît par défaut l’utilisation de la langue en-US. Consultez [Spécifier la langue source pour la reconnaissance vocale](../../../../how-to-specify-source-language.md) pour plus d’informations sur le choix de la langue source.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [footer](./footer.md)]