---
title: Vue d’ensemble de la synthèse vocale – service Speech
titleSuffix: Azure Cognitive Services
description: La synthèse vocale du service Speech est une fonctionnalité qui permet à vos applications, outils ou appareils de convertir du texte en parole naturelle synthétisée quasi humaine. Cet article est une vue d’ensemble des avantages et des capacités du service de conversion de texte par synthèse vocale.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 09/01/2020
ms.author: trbye
ms.custom: cog-serv-seo-aug-2020
keywords: conversion de texte par synthèse vocale
ms.openlocfilehash: af0c9338cb9661402c3a76bab55f34b01304c7b6
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024515"
---
# <a name="what-is-text-to-speech"></a>Qu’est-ce que la synthèse vocale ?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Dans cette vue d’ensemble, vous allez découvrir les avantages et les capacités du service de conversion de texte par synthèse vocale, qui permet à vos applications, outils ou appareils de convertir du texte en voix synthétisée quasi humaine. Choisissez parmi les voix standard et neuronales, ou créez une voix personnalisée propre à votre produit ou à votre marque. Plus de 75 voix standard sont disponibles dans plus de 45 langues et paramètres régionaux, et 5 voix neuronales sont disponibles dans un nombre limité de langues et de paramètres régionaux. Pour obtenir la liste complète des voix, langues et paramètres régionaux pris en charge, consultez [Langues prises en charge](language-support.md#text-to-speech).

> [!NOTE]
> La reconnaissance vocale Bing a été mise hors service le 15 octobre 2019. Si vos applications, outils ou produits utilisent les API Reconnaissance vocale Bing ou Custom Speech, nous avons créé des guides pour vous aider à effectuer leur migration vers le service Speech.
> - [Effectuer une migration entre la Reconnaissance vocale Bing et le service Speech](how-to-migrate-from-bing-speech.md)

## <a name="core-features"></a>Fonctionnalités de base

* Synthèse vocale : utilisez le [SDK Speech](./get-started-text-to-speech.md) ou l’[API REST](rest-text-to-speech.md) pour convertir du texte en voix avec des voix standard, neuronales ou personnalisées.

* Synthèse asynchrone d’audio long : utilisez l’[API Audio long](long-audio-api.md) pour synthétiser de texte en voix en mode asynchrone des fichiers de plus de 10 minutes (par exemple des livres audio ou des conférences). Contrairement à la synthèse effectuée avec le SDK Speech ou l’API REST de reconnaissance vocale, les réponses ne sont pas retournées en temps réel. Il est prévu que les demandes soient envoyées de façon asynchrone, que les réponses fassent l’objet d’une interrogation et que l’audio synthétisé soit téléchargé quand il est disponible auprès du service. Seules les voix neuronales personnalisées sont prises en charge.

* Voix standard : elles sont créées avec des techniques de synthèse paramétrique statistique et/ou de synthèse par concaténation. Ces voix sont hautement intelligibles et naturelles. Vous pouvez facilement configurer vos applications pour qu’elles parlent dans plus de 45 langues, avec un large éventail d’options vocales. Ces voix sont très précises en matière de prononciation. Elles prennent en charge les abréviations, les développements d’acronyme, l’interprétation des dates/heures, les polyphones et bien plus encore. Vous trouverez une liste complète des voix standard dans [Langues prises en charge](language-support.md#text-to-speech).

* Voix neuronales : les réseaux neuronaux profonds sont utilisés pour surmonter les limites de la synthèse vocale traditionnelle en ce qui concerne les accents toniques et les intonations dans le langage parlé. La prédiction prosodique et la synthèse vocale sont effectuées de façon simultanée, ce qui aboutit à un résultat plus fluide et plus naturel. Les voix neurales peuvent être utilisées pour rendre les interactions avec les chatbots et les assistants vocaux plus naturelles et plus agréables, pour convertir des textes numériques comme les livres électroniques en livres audio et pour améliorer les systèmes de navigation embarqués. Grâce à la prosodie naturelle quasi humaine et à la bonne articulation des mots, les voix neuronales réduisent considérablement la fatigue d’écoute lorsque vous interagissez avec des systèmes d’intelligence artificielle. Vous trouverez une liste complète des voix neuronales dans [Langues prises en charge](language-support.md#text-to-speech).

* SSML (Speech Synthesis Markup Language) : langage de balisage basé sur XML, utilisé pour personnaliser les résultats de la reconnaissance vocale. Avec SSML, vous pouvez ajuster la tonalité, ajouter des pauses, améliorer la prononciation, accélérer ou ralentir le débit des paroles, augmenter ou réduire le volume, et attribuer plusieurs voix à un même document. Consultez [SSML](speech-synthesis-markup.md).

## <a name="get-started"></a>Prise en main

Consultez le [démarrage rapide](get-started-text-to-speech.md) pour découvrir la conversion de texte par synthèse vocale. Le service de synthèse vocale est disponible via le [SDK Speech](speech-sdk.md), l’[API REST](rest-text-to-speech.md) et l’interface [CLI Speech](spx-overview.md)

## <a name="sample-code"></a>Exemple de code

Un exemple de code pour la synthèse vocale est disponible sur GitHub. Ces exemples couvrent la conversion du texte en parole dans les langages de programmation les plus populaires.

- [Exemples de synthèse vocale (SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)
- [Exemples de synthèse vocale (REST)](https://github.com/Azure-Samples/Cognitive-Speech-TTS)

## <a name="customization"></a>Personnalisation

En plus des voix standard et neuronales, vous pouvez créer et affiner des voix personnalisées propres à votre produit ou à votre marque. Pour commencer, vous n’avez besoin que de quelques fichiers audio et des transcriptions associées. Pour plus d’informations, consultez [Bien démarrer avec Custom Voice](how-to-custom-voice.md).

## <a name="pricing-note"></a>Remarque sur la tarification

Lorsque vous utilisez le service de synthèse vocale, vous êtes facturé pour chaque caractère converti en parole, y compris les signes de ponctuation. Alors que le document SSML lui-même n’est pas facturable, les éléments facultatifs utilisés pour ajuster la conversion du texte en parole, tels que les phonèmes et la tonalité, sont comptabilisés comme caractères facturables. Voici une liste des éléments pouvant être facturés :

- Texte transmis au service de synthèse vocale dans le corps SSML de la demande
- Tout balisage dans le champ de texte du corps de la demande au format SSML, à l’exception des balises `<speak>` et `<voice>`
- Lettres, ponctuation, espaces, tabulations, balisage et tous les caractères d’espace blanc
- Chaque point de code défini au format Unicode

Pour plus d’informations, consultez la section [Tarification](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

> [!IMPORTANT]
> Chaque caractère chinois, japonais et coréen compte pour deux en matière de facturation.

## <a name="reference-docs"></a>Documents de référence

- [Kit de développement logiciel (SDK) de reconnaissance vocale](speech-sdk.md)
- [API REST : Synthèse vocale](rest-text-to-speech.md)

## <a name="next-steps"></a>Étapes suivantes

- [Obtenir un abonnement gratuit au service Speech](overview.md#try-the-speech-service-for-free)
- [Obtenir le kit SDK Speech](speech-sdk.md)