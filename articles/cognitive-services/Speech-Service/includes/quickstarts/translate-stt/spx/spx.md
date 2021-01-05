---
author: v-demjoh
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 05/18/2020
ms.author: v-demjoh
ms.openlocfilehash: 4d548c1b7614503919de2725acb02d7f808ceb93
ms.sourcegitcommit: cf7caaf1e42f1420e1491e3616cc989d504f0902
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/22/2020
ms.locfileid: "83806381"
---
## <a name="run-the-speech-cli"></a>Exécuter l’interface CLI Speech

Vous êtes maintenant prêt à exécuter l’interface CLI Speech pour convertir la voix en texte dans une autre langue.

À partir de la ligne de commande, accédez au répertoire qui contient le fichier binaire de l’interface CLI Speech, puis tapez :

```bash
spx translate --microphone --target de-DE
```

L’interface CLI Speech traduit le langage naturel parlé (anglais) en texte imprimé en allemand.
Appuyez sur Entrée pour arrêter l’outil.

> [!NOTE]
> La valeur par défaut de l’interface CLI Speech est Anglais. Vous pouvez choisir une autre langue [dans la table Reconnaissance vocale](../../../../language-support.md).
> Par exemple, ajoutez `--source ja-JP` pour reconnaître de la voix en japonais.
