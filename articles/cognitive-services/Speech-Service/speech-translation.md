---
title: Vue d’ensemble de la traduction vocale – service Speech
titleSuffix: Azure Cognitive Services
description: La traduction vocale vous permet d’ajouter à vos applications, outils et appareils la traduction entière, en temps réel et en plusieurs langues d’un contenu oral. La même API peut être utilisée pour la traduction de parole en parole et de parole en texte. Cet article est une vue d’ensemble des avantages et des capacités du service de traduction vocale.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 09/01/2020
ms.author: erhopf
ms.custom: devx-track-csharp, cog-serv-seo-aug-2020
keywords: traduction vocale
ms.openlocfilehash: 36c2943d7cd5ce2c5c7c4a8501dc0033aa8c9cbc
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95015203"
---
# <a name="what-is-speech-translation"></a>Qu’est-ce que la traduction vocale ?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Dans cette vue d’ensemble, vous découvrez les avantages et les fonctionnalités du service de traduction vocale, qui permet de traduire en temps réel les flux audio multilingues de parole en parole et de parole en texte. Avec le SDK Speech, vos applications, outils et appareils ont accès à une transcription de la source et à une traduction en sortie pour l’audio fourni. Des résultats de transcription et de traduction intermédiaires sont retournés dès que de la parole est détectée, et les résultats finaux peuvent être convertis en synthèse vocale.

Le moteur de traduction de Microsoft repose sur deux approches différentes : la traduction automatique statistique (SMT) et la traduction automatique neuronale (NMT). La SMT s’appuie sur une analyse statistique avancée pour estimer la meilleure traduction possible compte tenu du contexte fourni par quelques mots. Avec la NMT, le recours aux réseaux neuronaux vise à offrir une traduction plus précise et naturelle en traduisant les mots à partir du contexte complet des phrases.

Aujourd’hui, Microsoft utilise la NMT pour la traduction dans les langues les plus courantes. Tous les [langues disponibles pour la traduction de parole en parole](language-support.md#speech-translation) sont alimentées par la NMT. La traduction de parole en texte peut utiliser la SMT ou la NMT en fonction de la paire de langues. Quand la langue cible est prise en charge par la NMT, la traduction complète s’appuie sur la NMT. Quand la langue cible n’est pas prise en charge par la NMT, la traduction combine la NMT et la SMT en utilisant l’anglais comme « pivot » entre les deux langues.

## <a name="core-features"></a>Fonctionnalités de base

* Traduction de la parole en texte avec les résultats de la reconnaissance.
* Traduction de la parole en parole.
* Prise en charge de la traduction dans plusieurs langues cibles.
* Résultats de reconnaissance et de traduction intermédiaires.

## <a name="get-started"></a>Bien démarrer 

Consultez le [démarrage rapide](get-started-speech-translation.md) pour découvrir la traduction vocale. La traduction vocale est disponible via le [kit de développement logiciel (SDK) Speech](speech-sdk.md) et l’[interface CLI](spx-overview.md).

## <a name="sample-code"></a>Exemple de code

Un exemple de code pour le SDK Speech est disponible sur GitHub. Ces exemples couvrent des scénarios courants tels que la lecture du contenu audio d’un fichier ou d’un flux, la reconnaissance/traduction continue et ponctuelle, et l’utilisation de modèles personnalisés.

* [Exemples de reconnaissance vocale et de traduction (SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)

## <a name="migration-guides"></a>Guides de migration

Si vos applications, outils ou produits utilisent l’[API Traduction de conversation Translator Speech](./how-to-migrate-from-translator-speech-api.md), nous avons créé des guides pour vous aider à effectuer une migration vers le service Speech.

* [Effectuer une migration de l’API de traduction de conversation Translator Speech vers le service Speech](how-to-migrate-from-translator-speech-api.md)

## <a name="reference-docs"></a>Documents de référence

* [Kit de développement logiciel (SDK) de reconnaissance vocale](./speech-sdk.md)
* [SDK Speech Devices](speech-devices-sdk.md)
* [API REST : Reconnaissance vocale](rest-speech-to-text.md)
* [API REST : Synthèse vocale](rest-text-to-speech.md)
* [API REST : Transcription et personnalisation par lot](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0)

## <a name="next-steps"></a>Étapes suivantes

* Effectuer le [démarrage rapide](get-started-speech-translation.md) de la traduction vocale
* [Obtenir gratuitement une clé d’abonnement au service Speech](overview.md#try-the-speech-service-for-free)
* [Obtenir le kit SDK Speech](speech-sdk.md)