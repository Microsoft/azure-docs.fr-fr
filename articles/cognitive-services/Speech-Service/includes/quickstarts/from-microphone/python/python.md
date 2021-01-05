---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 04/03/2020
ms.author: trbye
ms.openlocfilehash: f26f0ab6da398dcdee307f89b27cca780d08af85
ms.sourcegitcommit: 374d1533ea2f2d9d3f8b6e6a8e65c6a5cd4aea47
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/01/2020
ms.locfileid: "85838817"
---
## <a name="prerequisites"></a>Prérequis

Avant de commencer :

> [!div class="checklist"]
> * <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices" target="_blank">Créer une ressource Azure Speech<span class="docon docon-navigate-external x-hidden-focus"></span></a>
> * [Configurer votre environnement de développement et créer un projet vide](../../../../quickstarts/setup-platform.md?pivots=programming-language-python)
> * Veiller à avoir accès à un microphone pour la capture audio

## <a name="source-code"></a>Code source

Créez un fichier nommé *quickstart.py* et collez-y le code Python suivant.

[!code-python[Quickstart Code](~/samples-cognitive-services-speech-sdk/quickstart/python/from-microphone/quickstart.py#code)]

[!INCLUDE [replace key and region](../replace-key-and-region.md)]

## <a name="code-explanation"></a>Explication du code

[!INCLUDE [code explanation](../code-explanation.md)]

## <a name="build-and-run-app"></a>Générer et exécuter l’application

Vous êtes maintenant prêt à tester la reconnaissance vocale à l’aide du service Speech. 

Si vous exécutez cette application sur macOS et qu’il s’agit de la première application Python que vous avez créée qui utilise un microphone, vous aurez probablement besoin d’autoriser l’accès du terminal au microphone. Ouvrez **Paramètres système** et sélectionnez **Sécurité et confidentialité**. Sélectionnez ensuite **Confidentialité** et recherchez **Microphone** dans la liste. Enfin, sélectionnez **Terminal** et enregistrez. 

1. **Démarrer votre application** : à la ligne de commande, tapez :
    ```bash
    python quickstart.py
    ```
2. **Démarrer la reconnaissance** : vous êtes invité à prononcer une phrase. Celle-ci est envoyée au service de reconnaissance vocale, transcrite sous forme de texte, puis affichée sur la console.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [Speech recognition basics](../../speech-to-text-next-steps.md)]
