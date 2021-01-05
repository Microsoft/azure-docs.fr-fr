---
author: v-demjoh
ms.service: cognitive-services
ms.topic: include
ms.date: 05/13/2020
ms.author: v-demjoh
ms.openlocfilehash: a9beaea70dd3c8ce852344b11a50c5f20e5f8ca2
ms.sourcegitcommit: cf7caaf1e42f1420e1491e3616cc989d504f0902
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/22/2020
ms.locfileid: "83806369"
---
## <a name="enable-microphone"></a>Activer le microphone

Branchez et allumez le microphone de votre PC, puis désactivez toutes les applications qui peuvent également utiliser le microphone. Certains ordinateurs disposent d’un microphone intégré, tandis que d’autres nécessitent la configuration d’un appareil Bluetooth.

## <a name="run-the-speech-cli"></a>Exécuter l’interface CLI Speech

Vous êtes maintenant prêt à exécuter l’interface CLI Speech pour reconnaître la voix à partir de votre microphone.

1. **Démarrer votre application** - À partir de la ligne de commande, accédez au répertoire qui contient le fichier binaire de l’interface CLI Speech, puis tapez :
    ```bash
    spx recognize --microphone
    ```

    > [!NOTE]
    > La valeur par défaut de l’interface CLI Speech est Anglais. Vous pouvez choisir une autre langue [dans la table Reconnaissance vocale](../../../../language-support.md).
    > Par exemple, ajoutez `--source de-DE` pour reconnaître de la voix en allemand.

2. **Démarrer la reconnaissance** - Parlez dans le microphone. Vous verrez la transcription de vos mots en texte en temps réel. L’interface CLI Speech s’arrête après une période de silence, ou lorsque vous appuyez sur Ctrl-C.
