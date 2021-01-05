---
title: Concepts d’enrichissement de l’IA
titleSuffix: Azure Cognitive Search
description: L’extraction de contenu, le traitement du langage naturel et le traitement des images sont utilisés pour créer du contenu pouvant faire l’objet de recherches dans les index Recherche cognitive Azure à l’aide de compétences cognitives et d’algorithmes d’IA personnalisés prédéfinis.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/18/2020
ms.openlocfilehash: 09e7a39a2d97626dd01a00fdaef9bc4d711d557b
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91828099"
---
# <a name="ai-enrichment-in-azure-cognitive-search"></a>Enrichissement de l’IA dans Recherche cognitive Azure

L’enrichissement par IA est une extension des [indexeurs](search-indexer-overview.md) qui peut être utilisée pour extraire du texte à partir d’images, de blobs et d’autres sources de données non structurées. L’enrichissement et l’extraction facilitent les recherches de votre contenu dans les objets de sortie de l’indexeur, un [index de recherche](search-what-is-an-index.md) ou une [base de connaissances](knowledge-store-concept-intro.md). 

L’extraction et l’enrichissement sont implémentés à l’aide de *compétences cognitives* rattachées au pipeline piloté par l’indexeur. Vous pouvez utiliser les compétences intégrées de Microsoft ou incorporer un traitement externe dans une [*compétence personnalisés*](cognitive-search-create-custom-skill-example.md) que vous créez. Un module d’entité ou un classifieur de documents ciblant un domaine spécifique comme la finance, les publications scientifiques ou la médecine sont des exemples de compétence personnalisée.

Les compétences intégrées se répartissent en fonction des catégories suivantes : 

+ Les compétences de **traitement en langage naturel** incluent la [reconnaissance d’entité](cognitive-search-skill-entity-recognition.md), la [détection de la langue](cognitive-search-skill-language-detection.md), l’[extraction de phrases clés](cognitive-search-skill-keyphrases.md), la manipulation de texte, la [détection de sentiments](cognitive-search-skill-sentiment.md) et la [détection d’informations d’identification personnelle](cognitive-search-skill-pii-detection.md). Grâce à ces compétences, un texte non structuré est mappé sous la forme de champs pouvant être interrogés et filtrés dans un index.

+ Les compétences de **traitement d’image** incluent la [reconnaissance optique de caractères (OCR)](cognitive-search-skill-ocr.md) et l’identification des [caractéristiques visuelles](cognitive-search-skill-image-analysis.md), comme la détection des visages, l’interprétation des images, la reconnaissance des images (monuments et personnes célèbres) ou des attributs tels que l’orientation des images. Ces compétences créent des représentations textuelles du contenu des images, ce qui rend les recherches possibles grâce aux capacités d’interrogation de Recherche cognitive Azure.

![Diagramme de pipeline d’enrichissement](./media/cognitive-search-intro/cogsearch-architecture.png "vue d’ensemble du pipeline d’enrichissement")

Les compétences intégrées de la Recherche cognitive Azure sont basées sur les modèles Machine Learning préentraînés des API Cognitive Services : [Vision par ordinateur](../cognitive-services/computer-vision/index.yml) et [Analyse de texte](../cognitive-services/text-analytics/overview.md). Vous pouvez attacher une ressource Cognitive Services si vous souhaitez tirer parti de ces ressources lors du traitement du contenu.

Le traitement en langage naturel et le traitement des images sont appliqués durant la phase d’ingestion des données. Les résultats sont intégrés à la composition d’un document au sein d’un index pouvant faire l’objet de recherches dans la Recherche cognitive Azure. Les données sont fournies en tant que jeu de données Azure, puis transmises via un pipeline d’indexation à l’aide des [compétences intégrées](cognitive-search-predefined-skills.md) dont vous avez besoin.  

## <a name="when-to-use-ai-enrichment"></a>Quand utiliser l’enrichissement par IA

Vous devez envisager d’utiliser des compétences cognitives intégrées si votre contenu brut est du texte non structuré, du contenu d’image ou du contenu qui nécessite la détection de la langue et la traduction. L’application de l’intelligence artificielle via les compétences cognitives intégrées peut déverrouiller ce contenu, en renforçant sa valeur et son utilité dans vos applications de science des données et de recherche. 

En outre, vous pouvez envisager d’ajouter une compétence personnalisée si vous avez du code open source, tiers ou interne que vous souhaitez intégrer dans le pipeline. Les modèles de classification qui identifient les caractéristiques importantes de différents types de documents appartiennent à cette catégorie, mais tout package qui ajoute de la valeur à votre contenu peut être utilisé.

### <a name="more-about-built-in-skills"></a>En savoir plus sur les compétences intégrées

Un [ensemble de compétences](cognitive-search-defining-skillset.md) assemblé à l’aide de compétences intégrées convient parfaitement aux scénarios d’application suivants :

+ Documents numérisés (JPEG) dans lesquels effectuer une recherche en texte intégral. Vous pouvez associer une compétence de reconnaissance optique de caractères (OCR) pour identifier, extraire et ingérer du texte à partir de fichiers JPEG.

+ PDF avec image et texte combinés. Le texte des fichiers PDF peut être extrait durant l’indexation sans recourir aux étapes d’enrichissement. Toutefois, l’ajout du traitement des images et du traitement en langage naturel permet souvent de produire un résultat supérieur à celui fourni par une indexation standard.

+ Contenu multilingue sur lequel vous souhaitez appliquer la détection de langue et éventuellement la traduction de texte.

+ Documents non structurés ou semi-structurés contenant du contenu qui a une signification ou un contexte intrinsèquement masqué dans le document plus volumineux. 

  En particulier, les blobs contiennent souvent un grand corps de contenu qui est compressé dans un « champ » unique. En associant des compétences en traitement des images et en langage naturel à un indexeur, vous pouvez créer de nouvelles informations qui sont existantes dans le contenu brut, mais qui ne sont pas affichées en tant que champs distincts. Voici quelques compétences cognitives intégrées prêtes à l’emploi qui peuvent aider : l’extraction d’expressions clés, l’analyse des sentiments et la reconnaissance d’entité (personnes, organisations et lieux).

  De plus, les compétences prédéfinies peuvent également être utilisées pour restructurer du contenu via des opérations de découpage, de fusion et de mise en forme du texte.

### <a name="more-about-custom-skills"></a>En savoir plus sur les compétences personnalisées

Les compétences personnalisées peuvent prendre en charge des scénarios plus complexes, tels que la reconnaissance de formulaires ou la détection d’entité personnalisée à l’aide d’un modèle que vous fournissez et encapsulez dans l’[interface web des compétences personnalisées](cognitive-search-custom-skill-interface.md). Plusieurs exemples de compétences personnalisées incluent [Form Recognizer](../cognitive-services/form-recognizer/overview.md), l’intégration de l’[API Recherche d’entités Bing](./cognitive-search-create-custom-skill-example.md) et la [reconnaissance d’entité personnalisée](https://github.com/Microsoft/SkillsExtractorCognitiveSearch).

## <a name="steps-in-an-enrichment-pipeline"></a>Étapes d’un pipeline d’enrichissement <a name="enrichment-steps"></a>

Un pipeline d’enrichissement est basé sur des [*indexeurs*](search-indexer-overview.md). Les indexeurs remplissent un index basé sur des mappages champ à champ entre l’index et votre source de données pour la craquage de documents. Les compétences, désormais rattachées aux indexeurs, interceptent et enrichissent les documents en fonction des ensembles de compétences que vous définissez. Une fois l’indexation effectuée, vous pouvez accéder au contenu via des requêtes de recherche ainsi qu’à l’aide de tous les [types de requête pris en charge par la Recherche cognitive Azure](search-query-overview.md).  Si vous ne connaissez pas les indexeurs, cette section vous guide tout au long des étapes.

### <a name="step-1-connection-and-document-cracking-phase"></a>Étape 1 : Phase de connexion et de décodage du document

Au début du pipeline, vous avez du texte non structuré ou du contenu non textuel (comme des images, des documents numérisés ou des fichiers JPEG). Les données doivent se trouver dans un service de stockage de données Azure accessible par un indexeur. Les indexeurs peuvent décoder les documents sources pour extraire le texte à partir des données sources. Le craquage de document désigne le processus d’extraction ou de création du contenu textuel à partir de sources non textuelles lors de l’indexation.

![Phase de craquage de document](./media/cognitive-search-intro/document-cracking-phase-blowup.png "craquage de document")

 Les sources prises en charge comprennent le stockage Blob Azure, le stockage Table Azure, Azure SQL Database et Azure Cosmos DB. Le contenu textuel peut être extrait des types de fichier suivants : PDF, Word, PowerPoint et CSV. Pour obtenir la liste complète, consultez [Formats de document pris en charge](search-howto-indexing-azure-blob-storage.md#SupportedFormats). Étant donné que l’indexation prend un certain temps, commencez par un petit ensemble de données représentatif, puis augmentez sa taille de façon incrémentielle à mesure que votre solution grandit.

### <a name="step-2-cognitive-skills-and-enrichment-phase"></a>Étape 2 : Phase d’enrichissement et compétences cognitives

L’enrichissement se fait grâce aux *compétences cognitives* effectuant des opérations atomiques. Par exemple, une fois que vous avez craqué un fichier PDF, vous pouvez appliquer la reconnaissance d’entité, la détection de la langue ou l’extraction de phrases clés pour créer dans votre index des champs qui ne sont pas disponibles nativement dans la source. La collection de compétences utilisée dans votre pipeline est appelée *ensemble de compétences*.  

![Phase d’enrichissement](./media/cognitive-search-intro/enrichment-phase-blowup.png "phase d’enrichissement")

Un ensemble de compétences est basé sur des [compétences cognitives intégrées](cognitive-search-predefined-skills.md) ou des [compétences personnalisées](cognitive-search-create-custom-skill-example.md), que vous fournissez et que vous connectez à l’ensemble de compétences. Un ensemble de compétences peut être minimal ou très complexe, et détermine non seulement le type de traitement, mais également l’ordre des opérations. Un ensemble de compétences plus les mappages de champ définis dans un indexeur spécifient entièrement le pipeline d’enrichissement. Pour plus d’informations sur l’extraction de tous ces éléments ensemble, consultez [How to create a skillset in an enrichment pipeline](cognitive-search-defining-skillset.md) (Créer un ensemble de compétences dans un pipeline d’enrichissement).

En interne, le pipeline génère une collection de documents enrichis. Vous pouvez décider quelles parties des documents enrichis doivent être mappées aux champs indexables dans votre index de recherche. Par exemple, si vous avez appliqué les compétences de reconnaissance d’entité et d’extraction de phrases clés, ces nouveaux champs vont faire partie du document enrichi et peuvent donc être mappés aux champs de votre index. Consultez [How to reference annotations in a cognitive search skillset](cognitive-search-concept-annotations-syntax.md) (Référencer des annotations dans un ensemble de compétences de recherche cognitive) pour en savoir plus sur les formations entrée/sortie.

#### <a name="add-a-knowledgestore-element-to-save-enrichments"></a>Ajouter un élément knowledgeStore pour enregistrer des enrichissements

L’[API REST du service Recherche (api-version=2020-06-30)](/rest/api/searchservice/) étend les ensembles de compétences avec une définition `knowledgeStore` qui fournit une connexion de stockage Azure ainsi que des projections décrivant la manière dont les enrichissements sont stockés. Cela s’ajoute à votre index. Dans un pipeline IA standard, les documents enrichis sont temporaires, utilisés uniquement pendant l’indexation, puis ignorés. Avec la base de connaissances, les documents enrichis sont conservés. Pour plus d’informations, consultez [Base de connaissances](knowledge-store-concept-intro.md).

### <a name="step-3-search-index-and-query-based-access"></a>Étape 3 : Accès basé sur des requêtes et index de recherche

À la fin du traitement, vous disposez d’un index de recherche constitué de documents enrichis, qui peut faire l’objet de recherches textuelles dans la Recherche cognitive Azure. [L’interrogation de l’index](search-query-overview.md) est la façon dont les développeurs et les utilisateurs accèdent au contenu enrichi généré par le pipeline. 

![Index avec icône Rechercher](./media/cognitive-search-intro/search-phase-blowup.png "Index avec icône Rechercher")

L’index est semblable à tous ceux que vous pouvez créer pour la Recherche cognitive Azure : vous pouvez ajouter des analyseurs personnalisés, appeler des requêtes de recherche partielle, ajouter une recherche filtrée ou expérimenter des profils de scoring afin de remodeler les résultats de la recherche.

Les index sont générés à partir d’un schéma d’index qui définit les champs, les attributs et d’autres constructions jointes à un index spécifique, telles que les profils de score et les cartes de synonymes. Une fois qu’un index est défini et rempli, vous pouvez effectuer des indexations incrémentielles pour sélectionner de nouveaux documents sources et des documents sources mis à jour. Certaines modifications requièrent une régénération complète. Vous devez utiliser un petit jeu de données tant que la conception du schéma n’est pas stable. Pour plus d’informations, consultez [How to rebuild an Azure Search index](search-howto-reindex.md) (Régénérer un index Recherche Azure).

**Liste de vérification : workflow classique**

1. Créez un sous-ensemble de vos données sources dans un échantillon représentatif. Étant donné que l’indexation prend un certain temps, commencez par un petit ensemble de données représentatif, puis augmentez sa taille de façon incrémentielle à mesure que votre solution grandit.

1. Créez un [objet de source de données](/rest/api/searchservice/create-data-source) dans la Recherche cognitive Azure afin de fournir une chaîne de connexion pour l’extraction de données.

1. Créez un [ensemble de compétences](/rest/api/searchservice/create-skillset) avec les étapes d’enrichissement.

1. Définissez le [schéma d’index](/rest/api/searchservice/create-index). La collection *Champs* inclut des champs issus des données sources. Vous devez également écraser les champs supplémentaires pour stocker des valeurs générées pour le contenu créé au cours de l’enrichissement.

1. Définissez [l’indexeur](/rest/api/searchservice/create-indexer) faisant référence à la source de données, à l’ensemble de compétences et à l’index.

1. Dans l’indexeur, ajoutez *outputFieldMappings*. Cette section mappe la sortie de l’ensemble de compétences (à l’étape 3) aux champs d’entrées dans le schéma d’index (à l’étape 4).

1. Envoyez la requête *Créer un indexeur* que vous venez de créer (requête POST avec une définition d’indexeur dans le corps de la requête) pour exprimer l’indexeur dans la Recherche cognitive Azure. Cette étape correspond à la façon dont vous exécutez l’indexeur, en appelant le pipeline.

1. Exécutez des requêtes pour évaluer les résultats et modifiez le code pour mettre à jour les ensembles de compétences, le schéma ou la configuration de l’indexeur.

1. [Réinitialisez l’indexeur](search-howto-reindex.md) avant de régénérer le pipeline.

## <a name="next-steps"></a>Étapes suivantes

+ [Liens vers la documentation sur l’enrichissement par IA](cognitive-search-resources-documentation.md)
+ [Exemple : Création d’une compétence personnalisée pour l’enrichissement par IA (C#)](cognitive-search-create-custom-skill-example.md)
+ [Démarrage rapide : Procédure pas à pas d’enrichissement par IA à partir du portail](cognitive-search-quickstart-blob.md)
+ [Tutoriel : En savoir plus sur les API d’enrichissement par IA](cognitive-search-tutorial-blob.md)
+ [Base de connaissances](knowledge-store-concept-intro.md)
+ [Créer une base de connaissances avec REST](knowledge-store-create-rest.md)
+ [Conseils de dépannage](cognitive-search-concept-troubleshooting.md)
