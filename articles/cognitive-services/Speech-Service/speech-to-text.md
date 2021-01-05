---
title: Vue d’ensemble de la reconnaissance vocale – service Speech
titleSuffix: Azure Cognitive Services
description: Un logiciel de reconnaissance vocale permet la transcription en temps réel de flux audio en texte. Vos applications, outils ou appareils peuvent consommer, afficher et agir sur cette entrée de texte. Cet article est une vue d’ensemble des avantages et des capacités du service de reconnaissance vocale.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 09/01/2020
ms.author: trbye
ms.custom: cog-serv-seo-aug-2020
keywords: reconnaissance vocale, logiciel de reconnaissance vocale
ms.openlocfilehash: 102289fd233a744952f37568abf77a753953637e
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499643"
---
# <a name="what-is-speech-to-text"></a>Qu’est-ce que la reconnaissance vocale ?

Dans cette vue d’ensemble, vous découvrez les avantages et les fonctionnalités du service de reconnaissance vocale.
La conversion de parole en texte, également appelée reconnaissance vocale, permet la transcription en temps réel de flux audio en texte. Vos applications, outils ou appareils peuvent consommer, afficher et agir sur ce texte sous la forme d’une entrée de commande. Ce service s’appuie sur la même technologie de reconnaissance que celle utilisée par Microsoft pour Cortana et les produits Office. Il fonctionne de façon homogène avec les offres de service de <a href="./speech-translation.md" target="_blank">traduction<span class="docon docon-navigate-external x-hidden-focus"></span></a> et de <a href="./text-to-speech.md" target="_blank">synthèse vocale <span class="docon docon-navigate-external x-hidden-focus"></span></a>. Vous trouverez une liste complète des langues compatibles avec la reconnaissance vocale dans [Langues prises en charge](language-support.md#speech-to-text).

Par défaut, le service de reconnaissance vocale utilise le modèle de langage universel. Ce modèle a été entraîné avec des données qui sont la propriété de Microsoft et est déployé dans le cloud. Il est particulièrement adapté aux scénarios de conversation et de dictée. Si vous utilisez la reconnaissance vocale pour la reconnaissance et la transcription dans un environnement unique, vous pouvez créer et entraîner des modèles de prononciation, de langue et d’acoustique personnalisés. La personnalisation permet de prendre en compte un bruit ambiant ou le vocabulaire spécifique d’un secteur.

Avec une entrée de texte de référence supplémentaire, le service de reconnaissance vocale permet également [l’évaluation de la prononciation](rest-speech-to-text.md#pronunciation-assessment-parameters) afin d’évaluer la prononciation orale en fournissant aux orateurs des commentaires concertant l’exactitude et la fluidité de leur façon de parler. Grâce à l’évaluation de la prononciation, les élèves qui apprennent des langues peuvent pratiquer, obtenir des commentaires instantanés et améliorer leur prononciation pour pouvoir parler et se présenter en toute confiance. Les enseignants peuvent utiliser la fonctionnalité pour évaluer la prononciation de plusieurs intervenants en temps réel. La fonctionnalité prend actuellement en charge l’anglais américain et met en corrélation les évaluations orales menées par des experts.

> [!NOTE]
> La reconnaissance vocale Bing a été mise hors service le 15 octobre 2019. Si vos applications, outils ou produits utilisent les API Reconnaissance vocale Bing, nous avons créé des guides pour vous aider à effectuer leur migration vers le service Speech.
> - [Effectuer une migration entre la Reconnaissance vocale Bing et le service Speech](how-to-migrate-from-bing-speech.md)

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

## <a name="get-started"></a>Bien démarrer

Consultez le [guide de démarrage rapide](get-started-speech-to-text.md) pour démarrer avec la reconnaissance vocale. Ce service est disponible via le [kit SDK Speech](speech-sdk.md), l’[API REST](rest-speech-to-text.md#pronunciation-assessment-parameters) et l’[interface CLI Speech](spx-overview.md).

## <a name="sample-code"></a>Exemple de code

Un exemple de code pour le SDK Speech est disponible sur GitHub. Ces exemples couvrent des scénarios courants tels que la lecture du signal audio d’un fichier ou d’un flux, la reconnaissance continue et ponctuelle, et l’utilisation de modèles personnalisés.

- [Exemples de reconnaissance vocale (SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)
- [Exemples de transcription par lot (REST)](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/batch)
- [Échantillons d’évaluation de la prononciation (REST)](rest-speech-to-text.md#pronunciation-assessment-parameters)

## <a name="customization"></a>Personnalisation

En plus du modèle du service Speech standard, vous pouvez créer des modèles personnalisés. La personnalisation permet d’éliminer les obstacles à la reconnaissance vocale, comme le style d’élocution, le bruit de fond et le vocabulaire. Consultez [Custom Speech](./custom-speech-overview.md). Les options de personnalisation varient selon la langue et les paramètres régionaux. Consultez [Langues prises en charge](./language-support.md) pour vérifier ce qui est pris en charge.

## <a name="batch-transcription"></a>Transcription Batch

La transcription par lots est un ensemble d’opérations d’API REST qui vous permettent de transcrire une grande quantité de données audio dans un stockage. Vous pouvez pointer vers des fichiers audio à l’aide d’un URI de signature d’accès partagé (SAP) et recevoir les résultats de la transcription de manière asynchrone. Pour plus d’informations sur l’utilisation de l’API de transcription par lot, consultez la [procédure](batch-transcription.md).

[!INCLUDE [speech-reference-doc-links](includes/speech-reference-doc-links.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Obtenir gratuitement une clé d’abonnement au service Speech](overview.md#try-the-speech-service-for-free)
- [Obtenir le kit SDK Speech](speech-sdk.md)