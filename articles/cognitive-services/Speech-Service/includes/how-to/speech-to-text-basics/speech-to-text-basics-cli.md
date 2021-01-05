---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 09/08/2020
ms.author: trbye
ms.openlocfilehash: 905eacc3751b3d5d6c66a2fdb0e1391a747ab895
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91327123"
---
L’une des principales fonctionnalités du service de reconnaissance vocale est la possibilité de reconnaître et de transcrire la voix humaine (souvent appelée « reconnaissance vocale »). Dans ce guide de démarrage rapide, vous allez apprendre à utiliser l’interface CLI Speech dans vos applications et produits afin d’effectuer une conversion de voix en texte de qualité.

[!INCLUDE [SPX Setup](../../spx-setup.md)]

## <a name="speech-to-text-from-microphone"></a>Reconnaissance vocale à partir du microphone

Branchez et allumez le microphone de votre PC, puis désactivez toutes les applications qui peuvent également utiliser le microphone. Certains ordinateurs disposent d’un microphone intégré, tandis que d’autres nécessitent la configuration d’un appareil Bluetooth.

Vous êtes maintenant prêt à exécuter l’interface CLI Speech pour reconnaître la voix à partir de votre microphone. À partir de la ligne de commande, accédez au répertoire qui contient le fichier binaire de l’interface CLI Speech et exécutez la commande suivante.

```bash
spx recognize --microphone
```

> [!NOTE]
> La valeur par défaut de l’interface CLI Speech est Anglais. Vous pouvez choisir une autre langue [dans la table Reconnaissance vocale](../../../../language-support.md).
> Par exemple, ajoutez `--source de-DE` pour reconnaître de la voix en allemand.

Parlez dans le microphone et vous verrez la transcription de vos mots en texte en temps réel. L’interface CLI Speech s’arrête après une période de silence, ou lorsque vous appuyez sur Ctrl-C.

## <a name="speech-to-text-from-audio-file"></a>Reconnaissance vocale à partir d’un fichier audio

L’interface CLI Speech peut reconnaître la voix dans de nombreux formats de fichier et dans des langages naturels. Dans cet exemple, vous pouvez utiliser un fichier WAV (16 kHz ou 8 kHz, 16 bits et PCM mono) qui contient de la voix en anglais. Sinon, si vous souhaitez un exemple rapide, téléchargez le fichier <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/csharp/sharedcontent/console/whatstheweatherlike.wav" download="whatstheweatherlike" target="_blank">whatstheweatherlike.wav <span class="docon docon-download x-hidden-focus"></span></a> et copiez-le dans le même répertoire que le fichier binaire de l’interface CLI Speech.

Vous êtes maintenant prêt à exécuter l’interface CLI Speech pour reconnaître la voix trouvée dans le fichier audio en exécutant la commande suivante.

```bash
spx recognize --file whatstheweatherlike.wav
```

> [!NOTE]
> La valeur par défaut de l’interface CLI Speech est Anglais. Vous pouvez choisir une autre langue [dans la table Reconnaissance vocale](../../../../language-support.md).
> Par exemple, ajoutez `--source de-DE` pour reconnaître de la voix en allemand.

L’interface CLI Speech affiche une transcription textuelle de la voix à l’écran.