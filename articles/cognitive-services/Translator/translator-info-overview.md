---
title: Service Microsoft Translator
titlesuffix: Azure Cognitive Services
description: Intégrez Translator à vos applications, sites web, outils et autres solutions pour offrir une expérience utilisateur multilingue.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: overview
ms.date: 09/11/2020
ms.author: swmachan
ms.custom: cog-serv-seo-aug-2020
keywords: traducteur, traduction de texte, traduction automatique, service de traduction
ms.openlocfilehash: 8016c79cbe7f78bdaa18274f8873548e3982df2f
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95016467"
---
# <a name="what-is-the-translator-service"></a>Qu’est-ce que le service Translator ?

Translator est un service de traduction automatique basé sur le cloud qui fait partie de la famille [Azure Cognitive Services](../../index.yml?panel=ai&pivot=products) des API cognitives utilisées pour créer des applications intelligentes. Il est facile d’intégrer Translator à vos applications, sites web, outils et solutions. Cette interface vous permet d’ajouter des expériences utilisateur multilingues dans [plus de 70 langues](./language-support.md). Elle peut être utilisée sur tout type de plateforme matérielle possédant un système d’exploitation adapté à la traduction de texte.

## <a name="about-microsoft-translator"></a>À propos de Microsoft Translator

Translator alimente un certain nombre de produits et services Microsoft. Il est utilisé par des milliers d’entreprises à travers le monde, dans leurs applications et workflows qui proposent leur contenu à un public international.

La traduction vocale, technologie de Translator, est également disponible par le biais du [service Azure Speech](../speech-service/index.yml). Il combine les fonctionnalités de l’API Traduction de conversation Translator Speech et du Custom Speech Service en un service unifié et entièrement personnalisable. 

## <a name="language-support"></a>Support multilingue

Translator fournit une prise en charge multilingue pour la traduction de texte, la translittération, la détection de la langue et les dictionnaires. Consultez [Prise en charge de la langue](language-support.md) pour obtenir une liste complète, ou accéder à la liste par programmation avec l’[API REST](./reference/v3-0-languages.md).  

## <a name="microsoft-translator-neural-machine-translation"></a>Traduction automatique neuronale Microsoft Translator

La traduction automatique neuronale (NMT) constitue le nouveau standard en matière de traductions automatiques de grande qualité reposant sur l’intelligence artificielle. Elle remplace la technologie de traduction automatique statistique (SMT) héritée dont la qualité a plafonné à la moitié des années 2010.

La NMT fournit de meilleures traductions que la SMT, non seulement en termes de qualité par rapport à une traduction brute, mais également parce qu’elle donne des résultats plus fluides et plus naturels. La cause principale de cette fluidité est que la NMT utilise le contexte entier d’une phrase pour traduire des mots. La SMT ne prend en compte que le contexte immédiat de quelques mots avant et après chaque mot.

Les modèles NMT sont au cœur de l’API et ne sont pas visibles aux utilisateurs finaux. La seule différence notable réside dans l’amélioration de la qualité des traductions, en particulier pour les langues telles que le chinois, le japonais et l’arabe.

Découvrez-en plus sur le [fonctionnement de la traduction automatique neuronale](https://www.microsoft.com/en-us/translator/mt.aspx#nnt).

## <a name="improve-translations-with-custom-translator"></a>Améliorer les traductions avec Custom Translator

En tant qu’extension du service Translator, Custom Translator est utilisable conjointement avec Translator pour personnaliser le système de traduction neuronal et améliorer la traduction de la terminologie et du style qui vous sont propres.

Avec Custom Translator, vous pouvez créer des systèmes de traduction qui gèrent la terminologie utilisée dans votre entreprise ou secteur d’activité. Votre système de traduction personnalisé s’intégrera alors facilement à vos applications, flux de travail et sites web existants, sur plusieurs types d’appareils, par le biais de Translator standard, à l’aide du paramètre de catégorie.

Découvrez-en plus sur [Custom Translator](customization.md).

## <a name="next-steps"></a>Étapes suivantes

- [Inscrivez-vous](./translator-how-to-signup.md) pour obtenir une clé d’accès.
- Essayez notre [guide de démarrage rapide](quickstart-translator.md) pour appeler rapidement le service Translator.
- Les [informations de référence sur les API](./reference/v3-0-reference.md) constituent leur documentation technique.
- [Détails de la tarification](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/)