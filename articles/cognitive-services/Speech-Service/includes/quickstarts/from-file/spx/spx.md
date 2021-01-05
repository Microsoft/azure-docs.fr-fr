---
author: v-demjoh
ms.service: cognitive-services
ms.topic: include
ms.date: 05/13/2020
ms.author: v-demjoh
ms.openlocfilehash: 398dfe36314f86614515168986023146d695e69b
ms.sourcegitcommit: cf7caaf1e42f1420e1491e3616cc989d504f0902
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/22/2020
ms.locfileid: "83806222"
---
## <a name="find-a-file-that-contains-speech"></a>Rechercher un fichier qui contient de la voix

L’interface CLI Speech peut reconnaître la voix dans de nombreux formats de fichier et dans des langages naturels. Pour ce guide de démarrage rapide, vous pouvez utiliser un fichier WAV (16 kHz ou 8 kHz, 16 bits et PCM mono) qui contient de la voix en anglais.

1. Téléchargez le fichier <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/csharp/sharedcontent/console/whatstheweatherlike.wav" download="whatstheweatherlike" target="_blank">whatstheweatherlike.wav <span class="docon docon-download x-hidden-focus"></span></a>.
2. Copiez le fichier `whatstheweatherlike.wav` dans le même répertoire que le fichier binaire de l’interface CLI Speech.

## <a name="run-the-speech-cli"></a>Exécuter l’interface CLI Speech

Vous êtes maintenant prêt à exécuter l’interface CLI Speech pour reconnaître la voix trouvée dans le fichier audio.

À partir de la ligne de commande, accédez au répertoire qui contient le fichier binaire de l’interface CLI Speech, puis tapez :

```bash
spx recognize --file whatstheweatherlike.wav
```

> [!NOTE]
> La valeur par défaut de l’interface CLI Speech est Anglais. Vous pouvez choisir une autre langue [dans la table Reconnaissance vocale](../../../../language-support.md).
> Par exemple, ajoutez `--source de-DE` pour reconnaître de la voix en allemand.

L’interface CLI Speech affiche une transcription textuelle de la voix à l’écran. Ensuite, l’interface CLI Speech se ferme.
