---
title: Conversion de données - LUIS
titleSuffix: Azure Cognitive Services
description: Découvrez comment les énoncés peuvent être modifiés avant les prédictions de Language Understanding (LUIS)
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 07/29/2019
ms.openlocfilehash: 42a9caff0433808734ee853cbad90a2088bf4e1e
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95019243"
---
# <a name="convert-data-format-of-utterances"></a>Convertir le format de données des énoncés
LUIS fournit les conversions suivantes d’un énoncé utilisateur avant prédiction

* Reconnaissance vocale à l’aide du service [Cognitive Services Speech](../Speech-Service/overview.md).

## <a name="speech-to-text"></a>Reconnaissance vocale

La reconnaissance vocale est fournie en tant qu’intégration à LUIS.

### <a name="intent-conversion-concepts"></a>Concepts de conversion d’intention
La conversion de la reconnaissance vocale dans LUIS vous permet d’envoyer des énoncés parlés d’envoi à un point de terminaison et de recevoir une réponse de prédiction LUIS. Le processus est une intégration du service de [reconnaissance vocale](/azure/cognitive-services/Speech) avec LUIS. En savoir plus sur la conversion de sortie orale en intention avec un [tutoriel](../speech-service/how-to-recognize-intents-from-speech-csharp.md).

### <a name="key-requirements"></a>Conditions clés
Vous n’avez pas besoin de créer une clé d’**API Reconnaissance vocale Bing** pour cette intégration. Une clé **Language Understanding** créée dans le portail Azure fonctionne pour cette intégration. N’utilisez pas la clé de démarrage LUIS.

### <a name="pricing-tier"></a>Niveau de tarification
Cette intégration utilise un modèle [tarifaire](luis-limits.md#key-limits) différent de celui des niveaux tarifaires habituels de Language Understanding.

### <a name="quota-usage"></a>Rapport d’utilisation des quotas
Voir les [limites de clés](luis-limits.md#key-limits) pour plus d’informations.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Extraction des données](luis-concept-data-extraction.md)