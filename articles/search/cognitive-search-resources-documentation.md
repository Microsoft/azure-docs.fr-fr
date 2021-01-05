---
title: Liens vers la documentation pour l’enrichissement de l’IA
titleSuffix: Azure Cognitive Search
description: Liste annotée d’articles, de tutoriels, d’exemples et de billets de blog sur les charges de travail d’enrichissement de l’IA dans la recherche cognitive Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/12/2020
ms.openlocfilehash: eaafb9536a8667417c0abf4ab570609f2db290bc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88935345"
---
# <a name="documentation-resources-for-ai-enrichment-in-azure-cognitive-search"></a>Ressources de documentation pour l’enrichissement de l’IA dans la recherche cognitive Azure

L’enrichissement de l’IA est un module complémentaire associé à l’indexation basée sur un indexeur qui trouve des informations latentes dans des sources non textuelles et le texte indifférencié, et les transforme en contenu avec possibilité de recherche en texte intégral dans la recherche cognitive Azure. 

Pour le traitement intégré, les modèles AI préentraînés dans Cognitive Services sont appelés en interne pour effectuer les analyses. Vous pouvez également intégrer des modèles personnalisés à l’aide d’Azure Machine Learning, Azure Functions ou d’autres approches.

Vous trouverez ci-dessous une liste consolidée de la documentation sur l’enrichissement par IA.

## <a name="concepts"></a>Concepts

+ [Enrichissements par IA](cognitive-search-concept-intro.md)
+ [Ensemble de compétences](cognitive-search-working-with-skillsets.md)
+ [Sessions de débogage](cognitive-search-debug-session.md)
+ [Bases de connaissances](knowledge-store-concept-intro.md)
+ [Projections](knowledge-store-projection-overview.md)
+ [Enrichissement incrémentiel (réutilisation d’un document enrichi mis en cache)](cognitive-search-incremental-indexing-conceptual.md)

## <a name="hands-on-walkthroughs"></a>Procédures pratiques

+ [Démarrage rapide : Créer un ensemble de compétences cognitives dans le Portail Azure](cognitive-search-quickstart-blob.md)
+ [Tutoriel : Indexation enrichie avec l’IA](cognitive-search-tutorial-blob.md)
+ [Tutoriel : Diagnostiquer, réparer et valider les changements apportés à votre ensemble de compétences avec des sessions de débogage ](cognitive-search-tutorial-debug-sessions.md)

## <a name="knowledge-stores"></a>Bases de connaissances

+ [Démarrage rapide : Créer une base de connaissances dans le portail Azure](knowledge-store-create-portal.md)
+ [Créer une base de connaissances à l’aide de REST et Postman](knowledge-store-create-rest.md)
+ [Visualiser une base de connaissances avec l’Explorateur Stockage](knowledge-store-view-storage-explorer.md)
+ [Connecter une base de connaissances à Power BI](knowledge-store-connect-power-bi.md)
+ [Exemples de projection (comment mettre en forme et exporter des enrichissements)](knowledge-store-projections-examples.md)

## <a name="custom-skills-advanced"></a>Compétences personnalisées (avancé)

+ [Guide pratique pour définir une interface de compétences personnalisées](cognitive-search-custom-skill-interface.md)
+ [Exemple : Créer une qualification personnalisée à l’aide d’Azure Functions (et des API Recherche d'entités Bing)](cognitive-search-create-custom-skill-example.md)
+ [Exemple : Créer une compétence personnalisée avec Python](cognitive-search-custom-skill-python.md)
+ [Exemple : Créer une compétence personnalisée à l’aide de Form Recognizer](cognitive-search-custom-skill-form.md) 
+ [Exemple : Créer une compétence personnalisée avec Azure Machine Learning](cognitive-search-tutorial-aml-custom-skill.md) 

## <a name="how-to-guidance"></a>Guides pratiques

+ [Attacher une ressource Cognitive Services](cognitive-search-attach-cognitive-services.md)
+ [How to create a skillset in an enrichment pipeline](cognitive-search-defining-skillset.md) (Créer un ensemble de compétences dans un pipeline d’enrichissement)
+ [référencer des annotations dans un ensemble de compétences](cognitive-search-concept-annotations-syntax.md)
+ [Mapper des champs sur un index](cognitive-search-output-field-mapping.md)
+ [Traiter et extraire des informations à partir d’images](cognitive-search-concept-image-scenarios.md)
+ [Configurer la mise en cache pour l’enrichissement incrémentiel](search-howto-incremental-index.md)
+ [Guide pratique pour régénérer un index de recherche cognitive Azure](search-howto-reindex.md)
+ [Conseils de conception](cognitive-search-concept-troubleshooting.md)
+ [Erreurs et avertissements courants](cognitive-search-common-errors-warnings.md)

## <a name="skills-reference"></a>Informations de référence sur les compétences

+ [Compétences prédéfinies](cognitive-search-predefined-skills.md)
  + [Microsoft.Skills.Text.KeyPhraseExtractionSkill](cognitive-search-skill-keyphrases.md)
  + [Microsoft.Skills.Text.LanguageDetectionSkill](cognitive-search-skill-language-detection.md)
  + [Microsoft.Skills.Text.EntityRecognitionSkill](cognitive-search-skill-entity-recognition.md)
  + [Microsoft.Skills.Text.MergeSkill](cognitive-search-skill-textmerger.md)
  + [Microsoft.Skills.Text.PIIDetectionSkill](cognitive-search-skill-pii-detection.md)
  + [Microsoft.Skills.Text.SplitSkill](cognitive-search-skill-textsplit.md)
  + [Microsoft.Skills.Text.SentimentSkill](cognitive-search-skill-sentiment.md)
  + [Microsoft.Skills.Text.TranslationSkill](cognitive-search-skill-text-translation.md)
  + [Microsoft.Skills.Vision.ImageAnalysisSkill](cognitive-search-skill-image-analysis.md)
  + [Microsoft.Skills.Vision.OcrSkill](cognitive-search-skill-ocr.md)
  + [Microsoft.Skills.Util.ConditionalSkill](cognitive-search-skill-conditional.md)
  + [Microsoft.Skills.Util.DocumentExtractionSkill](cognitive-search-skill-document-extraction.md)
  + [Microsoft.Skills.Util.ShaperSkill](cognitive-search-skill-shaper.md)

+ Compétences personnalisées
  + [Microsoft.Skills.Custom.AmlSkill](cognitive-search-aml-skill.md)
  + [Microsoft.Skills.Custom.WebApiSkill](cognitive-search-custom-skill-web-api.md)

+ [Compétences dépréciées](cognitive-search-skill-deprecated.md)
  + [Microsoft.Skills.Text.NamedEntityRecognitionSkill](cognitive-search-skill-named-entity-recognition.md)

## <a name="apis"></a>API

+ [REST API](/rest/api/searchservice/)
  + [Créer un ensemble de compétences (api-version=2020-06-30)](/rest/api/searchservice/create-skillset)
  + [Créer un indexeur (api-version=2020-06-30)](/rest/api/searchservice/create-indexer)

## <a name="see-also"></a>Voir aussi

+ [API REST Recherche cognitive Azure](/rest/api/searchservice/)
+ [Indexeurs dans Recherche cognitive Azure](search-indexer-overview.md)
+ [Qu’est-ce que la recherche cognitive Azure ?](search-what-is-azure-search.md)