---
title: Limites de requête – Translator
titleSuffix: Azure Cognitive Services
description: Cet article liste les limites de requête pour Translator. Des frais sont facturés sur la base du nombre de caractères, et non pas de la fréquence des requêtes, avec une limite de 5 000 caractères par requêtes. Les limites de caractères sont fonction de l’abonnement, les abonnements F0 étant limités à 2 millions de caractères par heure.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 08/06/2020
ms.author: swmachan
ms.openlocfilehash: 8841c55d8f276f048db53a531bd2e9218e498b34
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95016494"
---
# <a name="request-limits-for-translator"></a>Limites de requête pour Translator

Cet article fournit des seuils de limitation de requêtes pour Translator. Les services incluent la traduction, la translittération, la détection de longueur de phrase, la détection de la langue et les traductions alternatives.

## <a name="character-and-array-limits-per-request"></a>Limites de caractères et de tableaux par requête

Chaque demande de traduction est limitée à 10 000 caractères, dans toutes les langues cibles de traduction. Par exemple, l’envoi d’une demande de traduction de 3 000 caractères à traduire en 3 langues différentes génère une demande d’une taille de 3 000 x 3 = 9 000 caractères, qui satisfait la limite de demande. Vous êtes facturé au caractère, et non pas au nombre de requêtes. Il est recommandé d’envoyer des requêtes plus courtes.

Les tableaux suivants répertorient les limites d’éléments de tableau et de caractères pour chaque opération de Translator.

| Opération | Taille maximale d’un élément de tableau |    Nombre maximal d’éléments de tableau |    Taille de requête maximale (caractères) |
|:----|:----|:----|:----|
| Translate | 10 000    | 100   | 10 000 |
| Transliterate | 5 000 | 10    | 5 000 |
| Detect | 50 000 | 100 |   50 000 |
| BreakSentence | 50 000    | 100 | 50 000 |
| Recherche dans le dictionnaire| 100 |  10  | 1 000 |
| Exemples de dictionnaire | 100 pour le texte et 100 pour la traduction (200 au total)| 10|   2 000 |

## <a name="character-limits-per-hour"></a>Limites de caractères par heure

Votre limite de caractère par heure est basée sur votre niveau d’abonnement à Translator. 

Le quota horaire doit être utilisé uniformément pendant la période d’une heure. Par exemple, pour la limite de 2 millions de caractères par heure du niveau F0, les caractères doivent être utilisés à une vitesse maximale de 33 300 caractères par fenêtre glissante de 1 minute (2 millions de caractères divisés par 60 minutes).

Si vous atteignez ou dépassez ces limites, ou si vous envoyez une partie trop importante du quota dans un court laps de temps, vous recevrez probablement une réponse de dépassement de quota. Il n’existe aucune limite concernant les requêtes simultanées.

| Niveau | Limite de caractères |
|------|-----------------|
| F0 | 2 millions de caractères par heure |
| S1 | 40 millions de caractères par heure |
| S2 / C2 | 40 millions de caractères par heure |
| S3 / C3 | 120 millions de caractères par heure |
| S4 / C4 | 200 millions de caractères par heure |

Les limites pour les [abonnements multiservice](./reference/v3-0-reference.md#authentication) sont les mêmes que pour le niveau S1.

Ces limites s’appliquent uniquement aux modèles de traduction standard de Microsoft. Les systèmes de traduction personnalisés qui utilisent Custom Translator sont limités à 1 800 caractères par seconde.

## <a name="latency"></a>Latence

Translator a une latence maximale de 15 secondes avec les modèles standards et de 120 secondes avec les modèles personnalisés. En général, les réponses *pour un texte de moins de 100 caractères* sont retournées dans un délai de 150 à 300 millisecondes. Les modèles de traducteurs personnalisés ont des caractéristiques de latence similaires sur le taux de requêtes soutenu et peuvent avoir une latence plus élevée lorsque le taux de requêtes est intermittent. Les temps de réponse peuvent varier en fonction de la taille de la requête et de la paire de langues. Si vous ne recevez pas de traduction ou de [réponse d’erreur](./reference/v3-0-reference.md#errors) pendant ce laps de temps, vérifiez votre code et votre connexion réseau, puis réessayez. 

## <a name="sentence-length-limits"></a>Limites de longueur de phrase

Quand vous utilisez la fonction [BreakSentence](./reference/v3-0-break-sentence.md), la longueur des phrases est limitée à 275 caractères. Il existe des exceptions pour les langues suivantes :

| Langage | Code | Limite de caractères |
|----------|------|-----------------|
| Chinois | zh | 166 |
| Allemand | de | 800 |
| Italien | it | 800 |
| Japonais | ja | 166 |
| Portugais | pt | 800 |
| Espagnol | es | 800 |
| Thaï | th | 180 |

> [!NOTE]
> Cette limite ne s’applique pas aux traductions.

## <a name="next-steps"></a>Étapes suivantes

* [Tarification](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/)
* [Disponibilité régionale](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services)
* [Référence de Translator v3](./reference/v3-0-reference.md)