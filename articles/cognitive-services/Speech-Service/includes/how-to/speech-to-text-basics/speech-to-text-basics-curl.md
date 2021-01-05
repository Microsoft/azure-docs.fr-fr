---
author: v-jaswel
ms.service: cognitive-services
ms.topic: include
ms.date: 10/09/2020
ms.author: v-jawe
ms.openlocfilehash: 0964872d03d6e321d25d51a18edbb4a6f0be8a4f
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94425087"
---
Dans ce guide de démarrage rapide, vous allez apprendre à convertir de la parole en texte à l’aide du service Speech et de cURL.

Pour obtenir une vue d’ensemble des concepts de reconnaissance vocale, consultez l’article de [présentation](../../../speech-to-text.md).

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous disposez d’un compte Azure et d’un abonnement au service Speech. Si vous n’avez pas de compte et d’abonnement, [essayez le service Speech gratuitement](../../../overview.md#try-the-speech-service-for-free).

## <a name="convert-speech-to-text"></a>Convertir la parole en texte

Depuis une invite de commandes, exécutez la commande suivante. Vous devrez insérer les valeurs suivantes dans la commande.
- Votre clé d’abonnement du service Speech.
- Votre région du service Speech.
- Le chemin du fichier audio d’entrée. Vous pouvez générer des fichiers audio en utilisant la [synthèse vocale](../../../get-started-text-to-speech.md).

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/speech-to-text.sh" id="request":::

Vous devez recevoir une réponse semblable à la suivante.

:::code language="curl" source="~/cognitive-services-quickstart-code/curl/speech/speech-to-text.sh" id="response":::

Pour plus d’informations, consultez les [informations de référence sur l’API REST de reconnaissance vocale](../../../rest-speech-to-text.md).