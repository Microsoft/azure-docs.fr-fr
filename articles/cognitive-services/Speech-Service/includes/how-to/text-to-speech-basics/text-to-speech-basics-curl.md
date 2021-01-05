---
author: v-jaswel
ms.service: cognitive-services
ms.topic: include
ms.date: 10/09/2020
ms.author: v-jawe
ms.openlocfilehash: 8a877e1773431053c5ad7344209076cb868a0ee3
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94424758"
---
Dans ce guide de démarrage rapide, vous allez apprendre à convertir du texte en parole à l’aide du service Speech et de cURL.

Pour obtenir une vue d’ensemble des concepts de synthèse vocale, consultez l’article de [présentation](../../../text-to-speech.md).

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous disposez d’un compte Azure et d’un abonnement au service Speech. Si vous n’avez pas de compte et d’abonnement, [essayez le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free).

## <a name="convert-text-to-speech"></a>Convertir du texte en parole

Depuis une invite de commandes, exécutez la commande suivante. Vous devrez insérer les valeurs suivantes dans la commande.
- Votre clé d’abonnement du service Speech.
- Votre région du service Speech.

Vous pouvez également changer les valeurs suivantes.
- La valeur d’en-tête `X-Microsoft-OutputFormat`, qui contrôle le format de sortie audio. Vous trouverez la liste des formats de sortie audio pris en charge dans les [informations de référence sur l’API REST de synthèse vocale](../../../rest-text-to-speech.md#audio-outputs).
- La voix de sortie. Pour obtenir la liste des voix disponibles pour votre point de terminaison Speech, consultez la section suivante.
- Fichier de sortie. Dans cet exemple, nous dirigeons la réponse du serveur vers un fichier nommé `output.wav`.

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/text-to-speech.sh":::

## <a name="list-available-voices-for-your-speech-endpoint"></a>Lister les voix disponibles pour votre point de terminaison Speech

Pour lister les voix disponibles pour votre point de terminaison Speech, exécutez la commande suivante.

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/get-voices.sh" id="request":::

Vous devez recevoir une réponse semblable à la suivante.

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/get-voices.sh" id="response":::