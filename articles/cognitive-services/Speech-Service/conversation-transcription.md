---
title: Transcription de conversation (préversion) - Service Speech
titleSuffix: Azure Cognitive Services
description: La transcription de conversation est une solution pour les réunions, qui combine la reconnaissance, l’ID de l’orateur et la diarisation pour fournir la transcription de toute conversation.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/16/2020
ms.author: trbye
ms.openlocfilehash: c27493748381a3aced92aab41745f352f228c50e
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024991"
---
# <a name="what-is-conversation-transcription-in-meetings-preview"></a>Qu’est-ce que la transcription de conversation dans les réunions (préversion) ?

La transcription de conversation est une solution de [reconnaissance vocale](speech-to-text.md) qui associe la reconnaissance vocale, l’identification de l’orateur et l’attribution de phrases à chaque orateur (également appelé _diarization_) pour fournir une transcription en temps réel et/ou asynchrone de toute conversation. La transcription de conversation distingue les orateurs d’une conversation pour déterminer qui a dit quoi et quand, et permet aux développeurs d’ajouter facilement la reconnaissance vocale à leurs applications qui effectuent une « diarization » à plusieurs orateurs.

## <a name="key-features"></a>Fonctionnalités clés

- **Timestamps** : chaque énoncé d’orateur inclut un timestamp, ce qui vous permet de retrouver facilement quand une phrase a été prononcée.
- **Transcriptions lisibles** : une mise en forme et une ponctuation sont automatiquement ajoutées aux transcriptions pour s’assurer que le texte correspond exactement à ce qui a été dit.
- **Profils utilisateur** : des profils utilisateur sont générés en collectant des exemples de voix d’utilisateur et en les transmettant pour la génération de signature.
- **Identification de l’orateur** : les orateurs sont identifiés à l’aide de profils utilisateur et un _identificateur d’orateur_ est attribué à chacun d’entre eux.
- **« Diarization » à plusieurs orateurs** : déterminez qui a dit quoi en synthétisant le flux audio avec chaque identificateur d’orateur.
- **Transcription en temps réel** : fournissez des transcriptions en direct sur qui dit quoi et pendant la conversation.
- **Transcription asynchrone** : fournissez des transcriptions d’une précision supérieure à l’aide d’un flux audio multicanal.

> [!NOTE]
> Bien que la transcription de conversation ne limite pas le nombre d’orateurs dans la pièce, elle est optimisée pour 2-10 orateurs par session.

## <a name="get-started"></a>Bien démarrer

Consultez le [Démarrage rapide](how-to-use-conversation-transcription.md) sur la transcription de conversation en temps réel pour commencer.

## <a name="use-cases"></a>Cas d'utilisation

Pour inclure tout le monde dans les réunions, comme les participants sourds et malentendants, il est important de disposer d’une transcription en temps réel. La transcription de conversation en temps réel prend en compte l’audio et détermine qui dit quoi, permettant ainsi à tous les participants à la réunion de suivre la transcription et de participer sans délai à la réunion.

### <a name="improved-efficiency"></a>Meilleure efficacité

Les participants à la réunion peuvent se concentrer sur la réunion et prendre des notes sur la transcription de conversation. Les participants peuvent participer activement à la réunion et suivre rapidement les étapes suivantes grâce à la transcription au lieu de prendre des notes et éventuellement de manquer des éléments pendant la réunion.

## <a name="how-it-works"></a>Fonctionnement

Il s’agit d’une vue d’ensemble détaillée du fonctionnement de la transcription de conversation.

![Schéma d’importation de la transcription de conversation](media/scenarios/conversation-transcription-service.png)

## <a name="expected-inputs"></a>Entrées attendues

- **Flux audio multicanal** - Pour obtenir des détails sur les spécifications et la conception, consultez [Microphones pour le SDK Microsoft Speech Devices](./speech-devices-sdk-microphone.md). Pour obtenir des informations complémentaires ou pour acheter un kit de développement, consultez [Obtenir le SDK Microsoft Speech Devices](./get-speech-devices-sdk.md).
- **Exemples de voix utilisateur** : la transcription de conversation a besoin de profils utilisateur avant la conversation. Vous devrez collecter des enregistrements audio de chaque utilisateur puis envoyer les enregistrements au [service de génération de signatures](https://aka.ms/cts/signaturegenservice) pour valider le contenu audio et générer des profils utilisateur.

> [!NOTE]
> Les échantillons vocaux de l’utilisateur sont facultatifs. Sans cette entrée, la transcription affiche des intervenants différents, mais sous la forme « Speaker1 », « Speaker2 », etc. au lieu de reconnaître les noms de conférenciers spécifiques préinscrits.


## <a name="real-time-vs-asynchronous"></a>Comparaison entre temps réel et asynchrone

La transcription de conversation offre trois modes de transcription :

### <a name="real-time"></a>Temps réel

Les données audio sont traitées en direct pour retourner l’identificateur de l’orateur et la transcription. Sélectionnez ce mode si votre solution de transcription doit fournir aux participants une transcription en direct de leur conversation en cours. Par exemple, la création d’une application pour rendre les réunions plus accessibles aux participants sourds et malentendants est un cas d’usage idéal de la transcription en temps réel.

### <a name="asynchronous"></a>Asynchrone

Les données audio sont traitées par lots pour retourner l’identificateur de l’orateur et la transcription. Sélectionnez ce mode si votre solution de transcription doit fournir une plus grande précision sans affichage de transcription en direct. Par exemple, si vous souhaitez créer une application pour permettre aux participants à des réunions de rattraper facilement les réunions manquées, utilisez le mode de transcription asynchrone pour obtenir des résultats de transcription haute précision.

### <a name="real-time-plus-asynchronous"></a>Temps réel plus asynchrone

Les données audio sont traitées en direct pour retourner l’identificateur de l’orateur et la transcription. De plus, une requête est créée pour obtenir également une transcription haute précision par le biais d’un traitement asynchrone. Sélectionnez ce mode si votre application a besoin d’une transcription en temps réel, mais nécessite également une transcription de précision supérieure pour une utilisation après la conversation ou la réunion.

## <a name="language-support"></a>Support multilingue

La transcription de conversation prend actuellement en charge [toutes les langues de reconnaissance vocale](language-support.md#speech-to-text) dans les régions suivantes :  `centralus`, `eastasia`, `eastus` et `westeurope`. Si vous avez besoin d’une prise en charge de paramètres régionaux supplémentaires, contactez l’[équipe de la fonctionnalité de transcription de conversation](mailto:CTSFeatureCrew@microsoft.com).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Transcrire des conversations en temps réel](how-to-use-conversation-transcription.md)