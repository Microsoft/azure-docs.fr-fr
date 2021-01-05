---
title: Ajouter des analyseurs personnalisés à des champs de chaîne
titleSuffix: Azure Cognitive Search
description: Configurer les générateurs de jetons de texte et les filtres de caractères utilisés dans les requêtes de recherche en texte intégral de Recherche cognitive Azure.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/05/2020
ms.openlocfilehash: fef73a9b98fef40aaceeacca43836d4b2f3c5de0
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97630205"
---
# <a name="add-custom-analyzers-to-string-fields-in-an-azure-cognitive-search-index"></a>Ajouter des analyseurs personnalisés à des champs de chaîne dans l’index de Recherche cognitive Azure

Un *analyseur personnalisé* est un type spécifique d’[analyseur de texte](search-analyzers.md) qui se compose d’une combinaison définie par l’utilisateur du générateur de jetons existant et des filtres facultatifs. En combinant des générateurs de jetons et des filtres de manière innovante, vous pouvez personnaliser le traitement du texte dans le moteur de recherche pour obtenir des résultats spécifiques. Par exemple, vous pouvez créer un analyseur personnalisé avec un *filtre de caractères* pour supprimer le balisage HTML avant que les entrées de texte soient tokenisées.

 Vous pouvez définir plusieurs analyseurs personnalisés pour varier la combinaison de filtres, mais chaque champ ne peut utiliser qu’un analyseur pour l’analyse de l’indexation et qu’un analyseur pour l’analyse de la recherche. Pour savoir à quoi ressemble un analyseur de client, consultez [Exemple d’analyseur personnalisé](search-analyzers.md#Custom-analyzer-example).

## <a name="overview"></a>Vue d’ensemble

 En termes simples, le rôle d’un [moteur de recherche en texte intégral](search-lucene-query-architecture.md) est de traiter et de stocker des documents afin de permettre une interrogation et une récupération efficaces. À un niveau élevé, tout cela se résume à extraire les mots importants des documents, à les placer dans un index, puis à utiliser ce dernier pour rechercher les documents qui correspondent aux mots d’une requête donnée. Le processus d’extraction de mots des documents et de requêtes de recherche est appelé *analyse lexicale*. Les composants qui effectuent une analyse lexicale sont appelés des *analyseurs*.

 Dans Recherche cognitive Azure, vous pouvez choisir parmi un ensemble prédéfini d’analyseurs indépendants des langues dans le tableau [Analyseurs](#AnalyzerTable) ou dans des analyseurs spécifiques à des langues listés dans [Analyseurs linguistiques &#40;API REST Recherche cognitive Azure&#41;](index-add-language-analyzers.md). Vous pouvez également définir vos propres analyseurs personnalisés.  

 Un analyseur personnalisé vous permet de contrôler le processus de conversion du texte en jetons indexables et utilisables pour la recherche. Il s’agit d’une configuration définie par l’utilisateur constituée d’un seul générateur de jetons prédéfini, d’un ou plusieurs filtres de jetons et d’un ou plusieurs filtres de caractères. Le générateur de jetons est chargé de segmenter le texte en jetons tandis que les filtres de jeton modifient les jetons émis par le générateur de jetons. Les filtres de caractères sont appliqués pour préparer le texte en entrée avant son traitement par le générateur de jetons. Par exemple, le filtre de caractères peut remplacer certains caractères ou certains symboles.

 Les scénarios courants permis par les analyseurs personnalisés sont les suivants :  

- Recherche phonétique. Ajoutez un filtre phonétique pour permettre une recherche basée sur la façon dont un mot est prononcé et non pas sur la façon dont il s’écrit.  

- Analyse lexicale désactivée. Utilisez l’analyseur de mots clés pour créer des champs utilisables pour la recherche qui ne sont pas analysés.  

- Recherche rapide de préfixe/suffixe. Ajoutez le filtre de jetons Edge N-gram pour indexer les préfixes des mots afin de permettre la mise en correspondance rapide des préfixes. Combinez-le avec le filtre de jeton Reverse pour effectuer une correspondance de suffixes.  

- Segmentation personnalisée du texte en unités lexicales. Par exemple, utilisez le générateur de jetons Whitespace pour segmenter les phrases en jetons en utilisant l’espace comme délimiteur.  

- Conversion ASCII. Ajoutez le filtre de conversion ASCII Standard pour normaliser les signes diacritiques, comme ö ou ê, dans les termes de recherche.  

  Cette page contient la liste des analyseurs, des générateurs de jetons, des filtres de jetons et des filtres de caractères pris en charge. Vous trouverez également la description des modifications apportées à la définition d’index avec un exemple d’utilisation. Pour plus d’informations sur la technologie sous-jacente exploitée dans l’implémentation de Recherche cognitive Azure, consultez l’article [Analysis package summary (Lucene)](https://lucene.apache.org/core/6_0_0/core/org/apache/lucene/codecs/lucene60/package-summary.html)(Résumé du package d’analyse [Lucene]). Pour obtenir des exemples de configurations d’analyseurs, consultez [Ajouter des analyseurs dans Recherche cognitive Azure](search-analyzers.md#examples).

## <a name="validation-rules"></a>Règles de validation  
 Les noms des analyseurs, des générateurs de jetons, des filtres de jetons et des filtres de caractères doivent être uniques, et ils ne peuvent pas être identiques à ceux des analyseurs, générateurs de jetons, filtres de jetons et filtres de caractères prédéfinis. Consultez les [Informations de référence sur les propriétés](#PropertyReference) pour les noms déjà utilisés.

## <a name="create-custom-analyzers"></a>Créer des analyseurs personnalisés
 Vous pouvez définir des analyseurs personnalisés au moment de la création d’index. La syntaxe de spécification d’un analyseur personnalisé est décrite dans cette section. Vous pouvez également vous familiariser avec la syntaxe en examinant les exemples de définitions dans [Ajouter des analyseurs dans Recherche cognitive Azure](search-analyzers.md#examples).  

 Une définition d’analyseur inclut un nom, un type, un ou plusieurs filtres de caractères, un générateur de jetons au maximum, et un ou plusieurs filtres de jetons pour le traitement venant après la segmentation du texte en unités lexicales. Les filtres de caractères sont appliqués avant la segmentation du texte en unités lexicales. Les filtres de jetons et les filtres de caractères sont appliqués de gauche à droite.

 Le `tokenizer_name` est le nom d’un générateur de jetons, `token_filter_name_1` et `token_filter_name_2` sont les noms des filtres de jetons, et `char_filter_name_1` et `char_filter_name_2` sont les noms des filtres de caractères (consultez les tableaux [Générateurs de jetons](#Tokenizers), [Filtres de jetons](#TokenFilters) et Filtres de caractères pour accéder aux valeurs valides).

La définition de l’analyseur est une partie de l’index. Consultez [API de création d’index](/rest/api/searchservice/create-index) pour plus d’informations sur le reste de l’index.

```
"analyzers":(optional)[
   {
      "name":"name of analyzer",
      "@odata.type":"#Microsoft.Azure.Search.CustomAnalyzer",
      "charFilters":[
         "char_filter_name_1",
         "char_filter_name_2"
      ],
      "tokenizer":"tokenizer_name",
      "tokenFilters":[
         "token_filter_name_1",
         "token_filter_name_2"
      ]
   },
   {
      "name":"name of analyzer",
      "@odata.type":"#analyzer_type",
      "option1":value1,
      "option2":value2,
      ...
   }
],
"charFilters":(optional)[
   {
      "name":"char_filter_name",
      "@odata.type":"#char_filter_type",
      "option1":value1,
      "option2":value2,
      ...
   }
],
"tokenizers":(optional)[
   {
      "name":"tokenizer_name",
      "@odata.type":"#tokenizer_type",
      "option1":value1,
      "option2":value2,
      ...
   }
],
"tokenFilters":(optional)[
   {
      "name":"token_filter_name",
      "@odata.type":"#token_filter_type",
      "option1":value1,
      "option2":value2,
      ...
   }
]
```

> [!NOTE]  
>  Les analyseurs personnalisés que vous créez ne sont pas exposés dans le portail Azure. La seule façon d’ajouter un analyseur personnalisé est de le faire dans du code qui effectue des appels à l’API lors de la définition d’un index.  

 Au sein d’une définition d’index, vous pouvez placer cette section n’importe où dans le corps d’une demande de création d’index, mais elle est généralement placée à la fin :  

```
{
  "name": "name_of_index",
  "fields": [ ],
  "suggesters": [ ],
  "scoringProfiles": [ ],
  "defaultScoringProfile": (optional) "...",
  "corsOptions": (optional) { },
  "analyzers":(optional)[ ],
  "charFilters":(optional)[ ],
  "tokenizers":(optional)[ ],
  "tokenFilters":(optional)[ ]
}
```

Les définitions des filtres de caractères, des générateurs de jetons et des filtres de jetons sont ajoutées à l’index seulement si vous définissez des options personnalisées. Pour utiliser tel quel un filtre ou un générateur de jetons existant, spécifiez son nom dans la définition de l’analyseur.

<a name="Testing custom analyzers"></a>

## <a name="test-custom-analyzers"></a>Tester des analyseurs personnalisés

Vous pouvez utiliser l’**opération de test d’analyseur** dans l’[API REST](/rest/api/searchservice/test-analyzer) pour voir comment un analyseur décompose le texte donné en jetons.

**Requête**
```
  POST https://[search service name].search.windows.net/indexes/[index name]/analyze?api-version=[api-version]
  Content-Type: application/json
    api-key: [admin key]

  {
     "analyzer":"my_analyzer",
     "text": "Vis-à-vis means Opposite"
  }
```
**Réponse**
```
  {
    "tokens": [
      {
        "token": "vis_a_vis",
        "startOffset": 0,
        "endOffset": 9,
        "position": 0
      },
      {
        "token": "vis_à_vis",
        "startOffset": 0,
        "endOffset": 9,
        "position": 0
      },
      {
        "token": "means",
        "startOffset": 10,
        "endOffset": 15,
        "position": 1
      },
      {
        "token": "opposite",
        "startOffset": 16,
        "endOffset": 24,
        "position": 2
      }
    ]
  }
```

## <a name="update-custom-analyzers"></a>Mettre à jour des analyseurs personnalisés

Une fois qu’un analyseur, un générateur de jetons, un filtre de jetons ou un filtre de caractères est défini, il ne peut pas être modifié. D’autres peuvent être ajoutés à un index existant à condition que l’indicateur `allowIndexDowntime` soit défini comme true dans la demande de mise à jour de l’index :

```
PUT https://[search service name].search.windows.net/indexes/[index name]?api-version=[api-version]&allowIndexDowntime=true
```

Cette opération place votre index hors connexion pendant au moins quelques secondes, ce qui entraîne l’échec de vos demandes d’indexation et de requête. Les performances et la disponibilité en écriture de l’index peuvent être altérées pendant plusieurs minutes après la mise à jour de l’index, ou plus longtemps pour les très grands index. Ces effets sont cependant temporaires et finissent par se résoudre d’eux-mêmes.

 <a name="ReferenceIndexAttributes"></a>

## <a name="analyzer-reference"></a>Référence d’analyseur

Les tableaux ci-dessous listent les propriétés de configuration de la section des analyseurs, des générateurs de jetons, des filtres de jetons et des filtres de caractères d’une définition d’index. La structure d’un analyseur, d’un générateur de jetons ou d’un filtre dans votre index est composée de ces attributs. Pour plus d’informations sur l’affectation d’une valeur, consultez les [Informations de référence sur les propriétés](#PropertyReference).

### <a name="analyzers"></a>Analyseurs

Pour les analyseurs, les attributs d’index varient selon que vous utilisez des analyseurs prédéfinis ou des analyseurs personnalisés.

#### <a name="predefined-analyzers"></a>Analyseurs prédéfinis

| Type | Description |
| ---- | ----------- |  
|Nom|Il doit contenir uniquement des lettres, des chiffres, des espaces, des tirets ou des traits de soulignement. Il doit commencer et se terminer uniquement par des caractères alphanumériques, et ne doit pas dépasser 128 caractères.|  
|Type|Type de l’analyseur provenant de la liste des analyseurs pris en charge. Consultez la colonne **type_analyseur** dans le tableau [Analyseurs](#AnalyzerTable) ci-dessous.|  
|Options|Il doit s’agir des options valides d’un analyseur prédéfini listées dans le tableau [Analyseurs](#AnalyzerTable) ci-dessous.|  

#### <a name="custom-analyzers"></a>Analyseurs personnalisés

| Type | Description |
| ---- | ----------- |  
|Nom|Il doit contenir uniquement des lettres, des chiffres, des espaces, des tirets ou des traits de soulignement. Il doit commencer et se terminer uniquement par des caractères alphanumériques, et ne doit pas dépasser 128 caractères.|  
|Type|Doit être « #Microsoft.Azure.Search.CustomAnalyzer ».|  
|CharFilters|Défini sur un des filtres de caractères prédéfinis listés dans le tableau [Filtres de caractères](#char-filters-reference) ou sur un filtre de caractères personnalisé spécifié dans la définition d’index.|  
|Générateur de jetons|Obligatoire. Défini sur un des générateurs de jetons prédéfinis listés dans le tableau [Générateur de jetons](#Tokenizers) ou sur un générateur de jetons personnalisé spécifié dans la définition d’index.|  
|TokenFilters|Défini sur un des filtres de jetons prédéfinis listés dans le tableau [Filtres de jetons](#TokenFilters) ou sur un filtre de jetons personnalisé spécifié dans la définition d’index.|  

> [!NOTE]
> Il est nécessaire de configurer votre analyseur personnalisé de façon à ne pas produire des jetons d’une longueur supérieure à 300 caractères. L’indexation échoue pour les documents avec de tels jetons. Pour les tronquer ou les ignorer, utilisez respectivement **TruncateTokenFilter** et **LengthTokenFilter**.  Voir [**Filtres de jeton**](#TokenFilters) pour en savoir plus.

<a name="CharFilter"></a>

### <a name="char-filters"></a>Filtres de caractères

 Un filtre de caractères est utilisé pour préparer le texte en entrée avant son traitement par le générateur de jetons. Par exemple, il peut remplacer certains caractères ou certains symboles. Vous pouvez avoir plusieurs filtres de caractères dans un analyseur personnalisé. Les filtres de caractères s’exécutent dans l’ordre où ils sont listés.  

| Type | Description |
| ---- | ----------- | 
|Nom|Il doit contenir uniquement des lettres, des chiffres, des espaces, des tirets ou des traits de soulignement. Il doit commencer et se terminer uniquement par des caractères alphanumériques, et ne doit pas dépasser 128 caractères.|  
|Type|Type de filtre de caractères provenant de la liste des filtres de caractères pris en charge. Consultez la colonne **type_filtre_caractères** dans le tableau [Filtres de caractères](#char-filters-reference) ci-dessous.|  
|Options|Il doit s’agir des options valides d’un type de [Filtre de caractères](#char-filters-reference) donné.|  

### <a name="tokenizers"></a>Générateurs de jetons

 Un générateur de jetons divise un texte continu en une séquence de jetons, par exemple en divisant une phrase en mots.  

 Vous pouvez spécifier un seul générateur de jetons par analyseur personnalisé. Si vous avez besoin de plusieurs générateurs de jetons, vous pouvez créer plusieurs analyseurs personnalisés et les affecter champ par champ dans votre schéma d’index.  
Un analyseur personnalisé peut utiliser un générateur de jetons prédéfini avec des options personnalisées ou par défaut.  

| Type | Description |
| ---- | ----------- | 
|Nom|Il doit contenir uniquement des lettres, des chiffres, des espaces, des tirets ou des traits de soulignement. Il doit commencer et se terminer uniquement par des caractères alphanumériques, et ne doit pas dépasser 128 caractères.|  
|Type|Nom du générateur de jetons provenant de la liste des générateurs de jetons pris en charge. Consultez la colonne **type_générateur_jetons** dans le tableau [Générateurs de jetons](#Tokenizers) ci-dessous.|  
|Options|Il doit s’agir des options valides d’un type de générateur de jetons donné listé dans le tableau [Générateurs de jetons](#Tokenizers) ci-dessous.|  

### <a name="token-filters"></a>Filtres de jeton

 Un filtre de jetons est utilisé pour filtrer ou modifier les jetons générés par un générateur de jetons. Par exemple, vous pouvez spécifier un filtre lowercase qui convertit tous les caractères en minuscules.   
Vous pouvez avoir plusieurs filtres de jetons dans un analyseur personnalisé. Les filtres de jetons s’exécutent dans l’ordre où ils sont listés.  

| Type | Description |
| ---- | ----------- |  
|Nom|Il doit contenir uniquement des lettres, des chiffres, des espaces, des tirets ou des traits de soulignement. Il doit commencer et se terminer uniquement par des caractères alphanumériques, et ne doit pas dépasser 128 caractères.|  
|Type|Nom du filtre de jetons provenant de la liste des filtres de jetons pris en charge. Consultez la colonne **type_filtre_jetons** dans le tableau [Filtres de jetons](#TokenFilters) ci-dessous.|  
|Options|Il doit s’agir de [Filtres de jeton](#TokenFilters) d’un type de filtre de jetons donné.|  

<a name="PropertyReference"></a>  

## <a name="property-reference"></a>Informations de référence sur les propriétés

Cette section fournit les valeurs valides pour les attributs spécifiés dans la définition d’un analyseur, d’un générateur de jetons, d’un filtre de caractères ou d’un filtre de jetons personnalisé dans votre index. Les analyseurs, les générateurs et les filtres qui sont implémentés avec Apache Lucene ont des liens vers la documentation de l’API Lucene.

<a name="AnalyzerTable"></a>

###  <a name="predefined-analyzers-reference"></a>Informations de référence sur les analyseurs prédéfinis

|**nom_analyseur**|**analyzer_type**  <sup>1</sup>|**Description et options**|  
|-|-|-|  
|[keyword](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordAnalyzer.html)| (le type s’applique seulement quand des options sont disponibles) |Traite l’intégralité du contenu d’un champ comme un seul jeton. Ceci est utile pour les données comme les codes postaux, les ID et certains noms de produit.|  
|[pattern](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/PatternAnalyzer.html)|PatternAnalyzer|Sépare le texte de façon flexible en termes via un modèle d’expression régulière.<br /><br /> **Options**<br /><br /> lowercase (type : booléen) : détermine si les termes sont en minuscules. La valeur par défaut est true.<br /><br /> [pattern](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html?is-external=true) (type : chaîne) : un modèle d’expression régulière pour mettre en correspondance les séparateurs de jetons. La valeur par défaut est `\W+`, qui correspond aux caractères non alphabétiques.<br /><br /> [flags](https://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html#field_summary) (type : chaîne) : indicateurs d’expression régulière. La valeur par défaut est une chaîne vide. Valeurs autorisées : CANON_EQ, CASE_INSENSITIVE, COMMENTS, DOTALL, LITERAL, MULTILINE, UNICODE_CASE, UNIX_LINES<br /><br /> stopwords (type : tableau de chaînes) : une liste de mots vides. La valeur par défaut est une liste vide.|  
|[simple](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/SimpleAnalyzer.html)|(le type s’applique seulement quand des options sont disponibles) |Divise le texte à l’endroit des caractères qui ne sont pas des lettres et le convertit en minuscules. |  
|[standard](https://lucene.apache.org/core/6_6_1/core/org/apache/lucene/analysis/standard/StandardAnalyzer.html) <br />(Également appelé standard.lucene)|StandardAnalyzer|Analyseur Lucene Standard, composé du générateur de jetons standard, du filtre lowercase et du filtre stop.<br /><br /> **Options**<br /><br /> maxTokenLength (type : entier) : la longueur maximale des jetons. La valeur par défaut est 255. Les jetons dépassant la longueur maximale sont fractionnés. La longueur maximale des jetons qui peut être utilisée est de 300 caractères.<br /><br /> stopwords (type : tableau de chaînes) : une liste de mots vides. La valeur par défaut est une liste vide.|  
|standardasciifolding.lucene|(le type s’applique seulement quand des options sont disponibles) |Analyseur standard avec filtre de conversion ASCII. |  
|[stop](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/StopAnalyzer.html)|StopAnalyzer|Divise un texte à l’endroit des caractères qui ne sont pas des lettres, applique les filtres de jetons lowercase et stopword.<br /><br /> **Options**<br /><br /> stopwords (type : tableau de chaînes) : une liste de mots vides. La valeur par défaut est une liste prédéfinie pour l’anglais. |  
|[whitespace](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/WhitespaceAnalyzer.html)|(le type s’applique seulement quand des options sont disponibles) |Un analyseur qui utilise le générateur de jetons whitespace. Les jetons d’une longueur supérieure à 255 caractères sont fractionnés.|  

 <sup>1</sup> Les types d’analyseurs sont toujours préfixés dans le code par « #Microsoft.Azure.Search » ; ainsi, « PatternAnalyzer » serait spécifié sous la forme « #Microsoft.Azure.Search.PatternAnalyzer ». Nous avons supprimé le préfixe par souci de concision, mais ce préfixe est obligatoire dans votre code. 
 
Le type d’analyseur (type_analyseur) est fourni seulement pour les analyseurs qui peuvent être personnalisés. S’il n’existe pas d’options, comme c’est le cas avec l’analyseur keyword, aucun type #Microsoft.Azure.Search n’est associé.


<a name="CharFilter"></a>

###  <a name="char-filters-reference"></a>Informations de référence sur les filtres de caractères

Dans le tableau ci-dessous, les filtres de caractères qui sont implémentés avec Apache Lucene sont liés à la documentation de l’API Lucene.

|**nom_filtre_caractères**|**char_filter_type** <sup>1</sup>|**Description et options**|  
|-|-|-|
|[html_strip](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/charfilter/HTMLStripCharFilter.html)|(le type s’applique seulement quand des options sont disponibles)  |Un filtre de caractères qui tente d’enlever retirer les constructions HTML.|  
|[mapping](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/charfilter/MappingCharFilter.html)|MappingCharFilter|Un filtre de caractères applique des mappages définis avec l’option mappings. La mise en correspondance est gourmande en ressources (la correspondance du modèle le plus long à un point donné l’emporte). La chaîne vide est autorisée comme remplacement.<br /><br /> **Options**<br /><br /> Mappings (type : tableau de chaînes) : une liste de mappages au format suivant : « a=>b » (toutes les occurrences du caractère « a » sont remplacées par le caractère « b »). Obligatoire.|  
|[pattern_replace](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/pattern/PatternReplaceCharFilter.html)|PatternReplaceCharFilter|Un filtre de caractères qui remplace des caractères dans la chaîne d’entrée. Il utilise une expression régulière pour identifier les séquences de caractères à conserver et un modèle de remplacement pour identifier les caractères à remplacer. Par exemple, texte d’entrée = "aa  bb aa bb", modèle = "(aa)\\\s+(bb)" remplacement = "$1#$2", résultat = "aa#bb aa#bb".<br /><br /> **Options**<br /><br /> pattern (type : chaîne) : obligatoire.<br /><br /> replacement (type : chaîne) : obligatoire.|  

 <sup>1</sup> Les types de filtres de caractères sont toujours préfixés dans le code par « #Microsoft.Azure.Search » ; ainsi, « MappingCharFilter » serait spécifié sous la forme « #Microsoft.Azure.Search.MappingCharFilter ». Nous avons supprimé le préfixe pour réduire la largeur du tableau, mais n’oubliez pas de l’inclure dans votre code. Notez que le type de filtre de caractères (type_filtre_caractères) est fourni seulement pour les filtres qui peuvent être personnalisés. S’il n’existe pas d’options, comme c’est le cas avec html_strip, aucun type #Microsoft.Azure.Search n’est associé.

<a name="Tokenizers"></a>

###  <a name="tokenizers-reference"></a>Informations de référence sur les générateurs de jetons

Dans le tableau ci-dessous, les générateurs de jetons qui sont implémentés avec Apache Lucene sont liés à la documentation de l’API Lucene.

|**nom_générateur_jetons**|**tokenizer_type** <sup>1</sup>|**Description et options**|  
|-|-|-|  
|[classique](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/standard/ClassicTokenizer.html)|ClassicTokenizer|Générateur de jetons basé sur la grammaire qui convient pour le traitement des documents dans la plupart des langues européennes.<br /><br /> **Options**<br /><br /> maxTokenLength (type : entier) : la longueur maximale des jetons. Valeur par défaut : 255, maximum : 300. Les jetons dépassant la longueur maximale sont fractionnés.|  
|[edgeNGram](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ngram/EdgeNGramTokenizer.html)|EdgeNGramTokenizer|Génère des jetons à partir de l’entrée depuis une délimitation en n-grammes d’une ou plusieurs tailles données.<br /><br /> **Options**<br /><br /> minGram (type : entier) : par défaut : 1, maximum : 300.<br /><br /> maxGram (type : entier) : par défaut : 2, maximum : 300. Doit être supérieur à minGram.<br /><br /> tokenChars (type : tableau de chaînes) : classes de caractères à conserver dans les jetons. Valeurs autorisées : <br />"letter", "digit", "whitespace", "punctuation", "symbol". La valeur par défaut est un tableau vide - conserve tous les caractères. |  
|[keyword_v2](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordTokenizer.html)|KeywordTokenizerV2|Génère la totalité de l’entrée sous la forme d’un unique jeton.<br /><br /> **Options**<br /><br /> maxTokenLength (type : entier) : la longueur maximale des jetons. Valeur par défaut : 256, maximum : 300. Les jetons dépassant la longueur maximale sont fractionnés.|  
|[letter](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/LetterTokenizer.html)|(le type s’applique seulement quand des options sont disponibles)  |Divise un texte à l’endroit des caractères qui ne sont pas des lettres. Les jetons d’une longueur supérieure à 255 caractères sont fractionnés.|  
|[lowercase](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/LowerCaseTokenizer.html)|(le type s’applique seulement quand des options sont disponibles)  |Divise le texte à l’endroit des caractères qui ne sont pas des lettres et le convertit en minuscules. Les jetons d’une longueur supérieure à 255 caractères sont fractionnés.|  
| microsoft_language_tokenizer| MicrosoftLanguageTokenizer| Divise le texte en utilisant des règles spécifiques à la langue.<br /><br /> **Options**<br /><br /> maxTokenLength (type : entier) : la longueur maximale des jetons. Par défaut : 255, maximum : 300. Les jetons dépassant la longueur maximale sont fractionnés. Les jetons de plus de 300 caractères sont d’abord fractionnés en jetons d’une longueur de 300 caractères, puis chacun de ces jetons est ensuite fractionné selon la valeur définie pour maxTokenLength.<br /><br />isSearchTokenizer (type : booléen) : défini sur true s’il est utilisé comme générateur de jetons de recherche, défini sur false s’il est utilisé comme générateur de jetons d’indexation. <br /><br /> language (type : chaîne) : langue à utiliser, par défaut « english ». Les valeurs autorisées sont les suivantes :<br />"bangla", "bulgarian", "catalan", "chineseSimplified",  "chineseTraditional", "croatian", "czech", "danish", "dutch", "english",  "french", "german", "greek", "gujarati", "hindi", "icelandic", "indonesian", "italian", "japanese", "kannada", "korean", "malay", "malayalam", "marathi", "norwegianBokmaal", "polish", "portuguese", "portugueseBrazilian", "punjabi", "romanian", "russian", "serbianCyrillic", "serbianLatin", "slovenian", "spanish", "swedish", "tamil", "telugu", "thai", "ukrainian", "urdu", "vietnamese" |
| microsoft_language_stemming_tokenizer | MicrosoftLanguageStemmingTokenizer| Divise le texte en utilisant des règles spécifiques à la langue et réduit les mots à leurs formes de base<br /><br /> **Options**<br /><br />maxTokenLength (type : entier) : la longueur maximale des jetons. Par défaut : 255, maximum : 300. Les jetons dépassant la longueur maximale sont fractionnés. Les jetons de plus de 300 caractères sont d’abord fractionnés en jetons d’une longueur de 300 caractères, puis chacun de ces jetons est ensuite fractionné selon la valeur définie pour maxTokenLength.<br /><br /> isSearchTokenizer (type : booléen) : défini sur true s’il est utilisé comme générateur de jetons de recherche, défini sur false s’il est utilisé comme générateur de jetons d’indexation.<br /><br /> language (type : chaîne) : langue à utiliser, par défaut « english ». Les valeurs autorisées sont les suivantes :<br />"arabic", "bangla", "bulgarian", "catalan", "croatian", "czech", "danish", "dutch", "english", "estonian", "finnish", "french", "german", "greek", "gujarati", "hebrew", "hindi", "hungarian", "icelandic", "indonesian", "italian", "kannada", "latvian", "lithuanian", "malay", "malayalam", "marathi", "norwegianBokmaal", "polish", "portuguese", "portugueseBrazilian", "punjabi", "romanian", "russian", "serbianCyrillic", "serbianLatin", "slovak", "slovenian", "spanish", "swedish", "tamil", "telugu", "turkish", "ukrainian", "urdu" |
|[nGram](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ngram/NGramTokenizer.html)|NGramTokenizer|Génère des jetons à partir de l’entrée en n-grammes d’une ou plusieurs tailles données.<br /><br /> **Options**<br /><br /> minGram (type : entier) : par défaut : 1, maximum : 300.<br /><br /> maxGram (type : entier) : par défaut : 2, maximum : 300. Doit être supérieur à minGram. <br /><br /> tokenChars (type : tableau de chaînes) : classes de caractères à conserver dans les jetons. Valeurs autorisées : "letter", "digit", "whitespace", "punctuation", "symbol". La valeur par défaut est un tableau vide - conserve tous les caractères. |  
|[path_hierarchy_v2](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/path/PathHierarchyTokenizer.html)|PathHierarchyTokenizerV2|Générateur de jetons pour les hiérarchies de type chemin.<br /><br /> **Options**<br /><br /> delimiter (type : chaîne) : par défaut : '/.<br /><br /> replacement (type : chaîne) : s’il est défini, remplace le caractère délimiteur. Par défaut, identique à la valeur du délimiteur.<br /><br /> maxTokenLength (type : entier) : la longueur maximale des jetons. Valeur par défaut : 300, maximum : 300. Les chemins plus longs que maxTokenLength sont ignorés.<br /><br /> reverse (type : booléen) : si la valeur est true, génère le jeton dans l’ordre inverse. Valeur par défaut : false.<br /><br /> skip (type : booléen) : jetons initiaux à ignorer. La valeur par défaut est 0.|  
|[pattern](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/pattern/PatternTokenizer.html)|PatternTokenizer|Ce générateur de jetons utilise la correspondance de modèle d’expression régulière pour construire des jetons distincts.<br /><br /> **Options**<br /><br /> [pattern](https://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html) (type : chaîne) : un modèle d’expression régulière pour mettre en correspondance les séparateurs de jetons. La valeur par défaut est `\W+`, qui correspond aux caractères non alphabétiques. <br /><br /> [flags](https://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html#field_summary) (type : chaîne) : indicateurs d’expression régulière. La valeur par défaut est une chaîne vide. Valeurs autorisées : CANON_EQ, CASE_INSENSITIVE, COMMENTS, DOTALL, LITERAL, MULTILINE, UNICODE_CASE, UNIX_LINES<br /><br /> group (type : entier) : groupe à extraire dans les jetons. La valeur par défaut est -1 (diviser).|
|[standard_v2](https://lucene.apache.org/core/6_6_1/core/org/apache/lucene/analysis/standard/StandardTokenizer.html)|StandardTokenizerV2|Décompose le texte en suivant les [règles de segmentation du texte Unicode](https://unicode.org/reports/tr29/).<br /><br /> **Options**<br /><br /> maxTokenLength (type : entier) : la longueur maximale des jetons. Valeur par défaut : 255, maximum : 300. Les jetons dépassant la longueur maximale sont fractionnés.|  
|[uax_url_email](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/standard/UAX29URLEmailTokenizer.html)|UaxUrlEmailTokenizer|Génère des jetons pour des URL et des e-mails sous la forme d’un seul jeton.<br /><br /> **Options**<br /><br /> maxTokenLength (type : entier) : la longueur maximale des jetons. Valeur par défaut : 255, maximum : 300. Les jetons dépassant la longueur maximale sont fractionnés.|  
|[whitespace](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/WhitespaceTokenizer.html)|(le type s’applique seulement quand des options sont disponibles) |Divise le texte au niveau des espaces. Les jetons d’une longueur supérieure à 255 caractères sont fractionnés.|  

 <sup>1</sup> Les types de générateurs de jetons sont toujours préfixés dans le code par « #Microsoft.Azure.Search » ; ainsi, « ClassicTokenizer » serait spécifié sous la forme « #Microsoft.Azure.Search.ClassicTokenizer ». Nous avons supprimé le préfixe pour réduire la largeur du tableau, mais n’oubliez pas de l’inclure dans votre code. Notez que le type de générateur de jetons (type_générateur_jetons) est fourni seulement pour les générateurs de jetons qui peuvent être personnalisés. S’il n’existe pas d’options, comme c’est le cas avec le générateur de jetons letter, aucun type #Microsoft.Azure.Search n’est associé.

<a name="TokenFilters"></a>

###  <a name="token-filters-reference"></a>Informations de référence sur les filtres de jetons

Dans le tableau ci-dessous, les filtres de jetons qui sont implémentés avec Apache Lucene sont liés à la documentation de l’API Lucene.

|**nom_filtre_jetons**|**token_filter_type** <sup>1</sup>|**Description et options**|  
|-|-|-|  
|[arabic_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ar/ArabicNormalizationFilter.html)|(le type s’applique seulement quand des options sont disponibles)  |Un filtre de jetons qui applique le normaliseur arabe pour normaliser l’orthographe.|  
|[apostrophe](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/tr/ApostropheFilter.html)|(le type s’applique seulement quand des options sont disponibles)  |Supprime tous les caractères suivant une apostrophe (y compris l’apostrophe elle-même). |  
|[asciifolding](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/ASCIIFoldingFilter.html)|AsciiFoldingTokenFilter|Convertit les caractères Unicode alphabétiques, numériques et symboliques qui ne sont pas dans les 127 premiers caractères ASCII (le bloc Unicode « Basic Latin ») en leur équivalent ASCII, s’il existe.<br /><br /> **Options**<br /><br /> preserveOriginal (type : booléen) : si la valeur est true, le jeton d’origine est conservé. La valeur par défaut est false.|  
|[cjk_bigram](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/cjk/CJKBigramFilter.html)|CjkBigramTokenFilter|Forme des digrammes de termes CJC qui sont générés à partir de StandardTokenizer.<br /><br /> **Options**<br /><br /> ignoreScripts (type : tableau de chaînes) : scripts à ignorer. Les valeurs autorisées sont : "han", "hiragana", "katakana", "hangul". La valeur par défaut est une liste vide.<br /><br /> outputUnigrams (type : booléen) : à définir sur true si vous voulez toujours obtenir en sortie des unigrammes et des digrammes. La valeur par défaut est false.|  
|[cjk_width](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/cjk/CJKWidthFilter.html)|(le type s’applique seulement quand des options sont disponibles)  |Normalise les différences de largeur de CJC. Convertit les variantes ASCII pleine chasse en équivalent Latin de base, et les variantes Katakana demi-chasse en équivalent kana. |  
|[classique](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/standard/ClassicFilter.html)|(le type s’applique seulement quand des options sont disponibles)  |Supprime les possessifs anglais et les points des acronymes. |  
|[common_grams](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/commongrams/CommonGramsFilter.html)|CommonGramTokenFilter|Construit des digrammes pour les termes d’occurrence fréquente lors de l’indexation. Les termes uniques sont néanmoins aussi indexés, avec des digrammes superposés.<br /><br /> **Options**<br /><br /> commonWords (type : tableau de chaînes) : l’ensemble des mots courants. La valeur par défaut est une liste vide. Obligatoire.<br /><br /> ignoreCase (type : booléen) : Si la valeur est true, la mise en correspondance ne respecte pas la casse. La valeur par défaut est false.<br /><br /> queryMode (type : booléen) : génère des digrammes, puis supprime les mots courants et les termes uniques suivis d’un mot courant. La valeur par défaut est false.|  
|[dictionary_decompounder](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/compound/DictionaryCompoundWordTokenFilter.html)|DictionaryDecompounderTokenFilter|Décompose les mots composés trouvés dans beaucoup de langues germaniques.<br /><br /> **Options**<br /><br /> wordList (type : tableau de chaînes) : la liste de mots dans laquelle rechercher des correspondances. La valeur par défaut est une liste vide. Obligatoire.<br /><br /> minWordSize (type : entier) : seuls les mots d’une longueur supérieure à cette taille sont traités. La valeur par défaut est 5.<br /><br /> minSubwordSize (type : entier) : seuls les sous-mots d’une longueur supérieure à cette taille figurent dans les résultats. La valeur par défaut est 2.<br /><br /> maxSubwordSize (type : entier) : seuls les sous-mots d’une longueur inférieure à cette taille figurent dans les résultats. La valeur par défaut est 15.<br /><br /> onlyLongestMatch (type : booléen) : ajoute seulement le sous-mot correspondant le plus long à la sortie. La valeur par défaut est false.|  
|[edgeNGram_v2](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ngram/EdgeNGramTokenFilter.html)|EdgeNGramTokenFilterV2|Génère des n-grammes de la ou des tailles données en démarrant de l’avant ou de l’arrière d’un jeton d’entrée.<br /><br /> **Options**<br /><br /> minGram (type : entier) : par défaut : 1, maximum : 300.<br /><br /> maxGram (type : entier) : par défaut : 2, maximum 300. Doit être supérieur à minGram.<br /><br /> side (type : chaîne) : spécifie le côté de l’entrée à partir duquel le n-gramme doit être généré. Valeurs autorisées : "front", "back" |  
|[elision](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/util/ElisionFilter.html)|ElisionTokenFilter|Supprime les élisions. Par exemple, « l’avion » est converti en « avion ».<br /><br /> **Options**<br /><br /> articles (type : tableau de chaînes) : un ensemble d’articles à supprimer. La valeur par défaut est une liste vide. Si aucune liste d’articles n’est définie, par défaut, tous les articles du français sont supprimés.|  
|[german_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/de/GermanNormalizationFilter.html)|(le type s’applique seulement quand des options sont disponibles)  |Normalise les caractères allemands en fonction de l’heuristique de l’[algorithme Snowball German2](https://snowballstem.org/algorithms/german2/stemmer.html).|  
|[hindi_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/hi/HindiNormalizationFilter.html)|(le type s’applique seulement quand des options sont disponibles)  |Normalise le texte dans Hindi de façon à supprimer des différences dans les variations orthographiques. |  
|[indic_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/in/IndicNormalizationFilter.html)|IndicNormalizationTokenFilter|Normalise la représentation Unicode du texte dans les langues indiennes.
|[keep](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/KeepWordFilter.html)|KeepTokenFilter|Filtre de jetons qui conserve seulement les jetons avec du texte contenu dans la liste de mots spécifiée.<br /><br /> **Options**<br /><br /> keepWords (type : tableau de chaînes) : une liste de mots à conserver. La valeur par défaut est une liste vide. Obligatoire.<br /><br /> keepWordsCase (type : booléen) : si la valeur est true, convertit d’abord tous les mots en minuscules. La valeur par défaut est false.|  
|[keyword_marker](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/KeywordMarkerFilter.html)|KeywordMarkerTokenFilter|Marque les termes comme mots clés.<br /><br /> **Options**<br /><br /> keywords (type : tableau de chaînes) : une liste de mots à marque comme mots clés. La valeur par défaut est une liste vide. Obligatoire.<br /><br /> ignoreCase (type : booléen) : si la valeur est true, convertit d’abord tous les mots en minuscules. La valeur par défaut est false.|  
|[keyword_repeat](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/KeywordRepeatFilter.html)|(le type s’applique seulement quand des options sont disponibles)  |Génère deux fois chaque jeton entrant : une fois comme mot clé et une fois comme non-mot clé. |  
|[kstem](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/en/KStemFilter.html)|(le type s’applique seulement quand des options sont disponibles)  |Un filtre kstem à hautes performances pour l’anglais. |  
|[length](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/LengthFilter.html)|LengthTokenFilter|Supprime les mots qui sont trop longs ou trop courts.<br /><br /> **Options**<br /><br /> min (type : entier) : le nombre minimal. Valeur par défaut : 0, maximum : 300.<br /><br /> max (type : entier) : le nombre maximal. Valeur par défaut : 300, maximum : 300.|  
|[limit](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/LimitTokenCountFilter.html)|Microsoft.Azure.Search.LimitTokenFilter|Limite le nombre de jetons lors de l’indexation.<br /><br /> **Options**<br /><br /> maxTokenCount (type : entier) : nombre maximal de jetons à produire. La valeur par défaut est 1.<br /><br /> consumeAllTokens (type : booléen) : indique si tous les jetons de l’entrée doivent être utilisés même si maxTokenCount est atteint. La valeur par défaut est false.|  
|[lowercase](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/LowerCaseFilter.html)|(le type s’applique seulement quand des options sont disponibles)  |Normalise le texte des jetons en minuscules. |  
|[nGram_v2](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ngram/NGramTokenFilter.html)|NGramTokenFilterV2|Génère des n-grammes de la taille donnée.<br /><br /> **Options**<br /><br /> minGram (type : entier) : par défaut : 1, maximum : 300.<br /><br /> maxGram (type : entier) : par défaut : 2, maximum 300. Doit être supérieur à minGram.|  
|[pattern_capture](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/pattern/PatternCaptureGroupTokenFilter.html)|PatternCaptureTokenFilter|Utilise des expressions régulières Java pour générer plusieurs jetons, un pour chaque groupe de capture dans un ou plusieurs modèles.<br /><br /> **Options**<br /><br /> patterns (type : tableau de chaînes) : une liste de modèles à mettre en correspondance avec chaque jeton. Obligatoire.<br /><br /> preserveOriginal (type : booléen) : à définir sur true pour retourner le jeton d’origine, même si un des modèles est en correspondance. Valeur par défaut : true |  
|[pattern_replace](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/pattern/PatternReplaceFilter.html)|PatternReplaceTokenFilter|Un filtre de jeton qui applique un modèle à chaque jeton du flux, en remplaçant les occurrences en correspondance par la chaîne de remplacement spécifiée.<br /><br /> **Options**<br /><br /> pattern (type : chaîne) : obligatoire.<br /><br /> replacement (type : chaîne) : obligatoire.|  
|[persian_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/fa/PersianNormalizationFilter.html)|(le type s’applique seulement quand des options sont disponibles) |Applique la normalisation pour le persan. |  
|[phonetic](https://lucene.apache.org/core/6_6_1/analyzers-phonetic/org/apache/lucene/analysis/phonetic/package-tree.html)|PhoneticTokenFilter|Crée des jetons pour les correspondances phonétiques.<br /><br /> **Options**<br /><br /> encoder (type : chaîne) : Encodeur phonétique à utiliser. Les valeurs autorisées sont : "metaphone", "doubleMetaphone", "soundex", "refinedSoundex", "caverphone1", "caverphone2", "cologne", "nysiis", "koelnerPhonetik", "haasePhonetik", "beiderMorse". Par défaut : "metaphone". La valeur par défaut est metaphone.<br /><br /> Pour plus d’informations, consultez [encoder](https://commons.apache.org/proper/commons-codec/archives/1.10/apidocs/org/apache/commons/codec/language/package-frame.html).<br /><br /> replace (type : booléen) : true si les jetons encodés doivent remplacer les jetons d’origine ; false s’ils doivent être ajoutés comme synonymes. La valeur par défaut est true.|  
|[porter_stem](https://lucene.apache.org/core/6_6_1/analyzers-common/org/tartarus/snowball/ext/PorterStemmer.html)|(le type s’applique seulement quand des options sont disponibles)  |Transforme le flux de jetons selon l’[algorithme de recherche de radical de Porter](https://tartarus.org/~martin/PorterStemmer/). |  
|[reverse](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/reverse/ReverseStringFilter.html)|(le type s’applique seulement quand des options sont disponibles)  |Inverse la chaîne des jetons. |  
|[scandinavian_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/ScandinavianNormalizationFilter.html)|(le type s’applique seulement quand des options sont disponibles)  |Normalise l’utilisation des caractères scandinaves interchangeables. |  
|[scandinavian_folding](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/ScandinavianFoldingFilter.html)|(le type s’applique seulement quand des options sont disponibles)  |Convertit les caractères scandinaves åÅäæÄÆ->a et öÖøØ->o. Il identifie aussi l’utilisation des voyelles doubles aa, ae, ao, oe et oo, et conserve seulement la première voyelle. |  
|[shingle](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/shingle/ShingleFilter.html)|ShingleTokenFilter|Crée des combinaisons de jetons sous la forme d’un unique jeton.<br /><br /> **Options**<br /><br /> maxShingleSize (type : entier) : la valeur par défaut est 2.<br /><br /> minShingleSize (type : entier) : la valeur par défaut est 2.<br /><br /> outputUnigrams (type : booléen) : si la valeur est true, le flux de sortie contient les jetons d’entrée (unigrammes) ainsi que les n-grammes composés de mots. La valeur par défaut est true.<br /><br /> outputUnigramsIfNoShingles (type : booléen) : si la valeur est true, remplace le comportement de outputUnigrams==false quand aucun n-gramme composé de mots n’est disponible. La valeur par défaut est false.<br /><br /> tokenSeparator (type : chaîne) : la chaîne à utiliser lors de la jonction de jetons adjacents pour former un n-gramme composé de mots. La valeur par défaut est " ".<br /><br /> filterToken (type : chaîne) : la chaîne à insérer pour chaque position à laquelle il n’y a pas de jeton. La valeur par défaut est "_".|  
|[snowball](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/snowball/SnowballFilter.html)|SnowballTokenFilter|Filtre de jetons Snowball.<br /><br /> **Options**<br /><br /> language (type : chaîne) : les valeurs autorisées sont : "armenian", "basque", "catalan", "danish", "dutch", "english", "finnish", "french", "german", "german2", "hungarian", "italian", "kp", "lovins", "norwegian", "porter", "portuguese", "romanian", "russian", "spanish", "swedish", "turkish"|  
|[sorani_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ckb/SoraniNormalizationFilter.html)|SoraniNormalizationTokenFilter|Normalise la représentation Unicode du texte en sorani.<br /><br /> **Options**<br /><br /> Aucun.|  
|stemmer|StemmerTokenFilter|Filtre de recherche de radical spécifique à la langue.<br /><br /> **Options**<br /><br /> language (type : chaîne) : les valeurs autorisées sont : <br /> -   ["arabic"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ar/ArabicStemmer.html)<br />-   ["armenian"](https://snowballstem.org/algorithms/armenian/stemmer.html)<br />-   ["basque"](https://snowballstem.org/algorithms/basque/stemmer.html)<br />-   ["brazilian"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/br/BrazilianStemmer.html)<br />-   "bulgarian"<br />-   ["catalan"](https://snowballstem.org/algorithms/catalan/stemmer.html)<br />-   ["czech"](https://portal.acm.org/citation.cfm?id=1598600)<br />-   ["danish"](https://snowballstem.org/algorithms/danish/stemmer.html)<br />-   ["dutch"](https://snowballstem.org/algorithms/dutch/stemmer.html)<br />-   ["dutchKp"](https://snowballstem.org/algorithms/kraaij_pohlmann/stemmer.html)<br />-   ["english"](https://snowballstem.org/algorithms/porter/stemmer.html)<br />-   ["lightEnglish"](https://ciir.cs.umass.edu/pubfiles/ir-35.pdf)<br />-   ["minimalEnglish"](https://www.researchgate.net/publication/220433848_How_effective_is_suffixing)<br />-   ["possessiveEnglish"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/en/EnglishPossessiveFilter.html)<br />-   ["porter2"](https://snowballstem.org/algorithms/english/stemmer.html)<br />-   ["lovins"](https://snowballstem.org/algorithms/lovins/stemmer.html)<br />-   ["finnish"](https://snowballstem.org/algorithms/finnish/stemmer.html)<br />-   "lightFinnish"<br />-   ["french"](https://snowballstem.org/algorithms/french/stemmer.html)<br />-   ["lightFrench"](https://dl.acm.org/citation.cfm?id=1141523)<br />-   ["minimalFrench"](https://dl.acm.org/citation.cfm?id=318984)<br />-   "galician"<br />-   "minimalGalician"<br />-   ["german"](https://snowballstem.org/algorithms/german/stemmer.html)<br />-   ["german2"](https://snowballstem.org/algorithms/german2/stemmer.html)<br />-   ["lightGerman"](https://dl.acm.org/citation.cfm?id=1141523)<br />-   "minimalGerman"<br />-   ["greek"](https://sais.se/mthprize/2007/ntais2007.pdf)<br />-   "hindi"<br />-   ["hungarian"](https://snowballstem.org/algorithms/hungarian/stemmer.html)<br />-   ["lightHungarian"](https://dl.acm.org/citation.cfm?id=1141523&dl=ACM&coll=DL&CFID=179095584&CFTOKEN=80067181)<br />-   ["indonesian"](https://eprints.illc.uva.nl/741/2/MoL-2003-03.text.pdf)<br />-   ["irish"](https://snowballstem.org/otherapps/oregan/)<br />-   ["italian"](https://snowballstem.org/algorithms/italian/stemmer.html)<br />-   ["lightItalian"](https://www.ercim.eu/publication/ws-proceedings/CLEF2/savoy.pdf)<br />-   ["sorani"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ckb/SoraniStemmer.html)<br />-   ["latvian"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/lv/LatvianStemmer.html)<br />-   ["norwegian"](https://snowballstem.org/algorithms/norwegian/stemmer.html)<br />-   ["lightNorwegian"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianLightStemmer.html)<br />-   ["minimalNorwegian"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianMinimalStemmer.html)<br />-   ["lightNynorsk"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianLightStemmer.html)<br />-   ["minimalNynorsk"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianMinimalStemmer.html)<br />-   ["portuguese"](https://snowballstem.org/algorithms/portuguese/stemmer.html)<br />-   ["lightPortuguese"](https://dl.acm.org/citation.cfm?id=1141523&dl=ACM&coll=DL&CFID=179095584&CFTOKEN=80067181)<br />-   ["minimalPortuguese"](https://www.inf.ufrgs.br/~buriol/papers/Orengo_CLEF07.pdf)<br />-   ["portugueseRslp"](https://www.inf.ufrgs.br//~viviane/rslp/index.htm)<br />-   ["romanian"](https://snowballstem.org/otherapps/romanian/)<br />-   ["russian"](https://snowballstem.org/algorithms/russian/stemmer.html)<br />-   ["lightRussian"](https://doc.rero.ch/lm.php?url=1000%2C43%2C4%2C20091209094227-CA%2FDolamic_Ljiljana_-_Indexing_and_Searching_Strategies_for_the_Russian_20091209.pdf)<br />-   ["spanish"](https://snowballstem.org/algorithms/spanish/stemmer.html)<br />-   ["lightSpanish"](https://www.ercim.eu/publication/ws-proceedings/CLEF2/savoy.pdf)<br />-   ["swedish"](https://snowballstem.org/algorithms/swedish/stemmer.html)<br />-   "lightSwedish"<br />-   ["turkish"](https://snowballstem.org/algorithms/turkish/stemmer.html)|  
|[stemmer_override](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/StemmerOverrideFilter.html)|StemmerOverrideTokenFilter|Les termes dont le radical est trouvé dans un dictionnaire sont marqués comme mots clés, ce qui empêche la recherche de radical plus avant dans la chaîne. Doit être placé avant les filtres de recherche de radical.<br /><br /> **Options**<br /><br /> rules (type : tableau de chaînes) : règles de recherche de radical au format suivant "mot => radical", par exemple "ran => run". La valeur par défaut est une liste vide.  Obligatoire.|  
|[stopwords](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/StopFilter.html)|StopwordsTokenFilter|Supprime les mots vides d’un flux de jetons. Par défaut, le filtre utilise une liste de mots vides prédéfinie pour l’anglais.<br /><br /> **Options**<br /><br /> stopwords (type : tableau de chaînes) : une liste de mots vides. Ne peut pas être spécifié si une liste stopwordsList est spécifiée.<br /><br /> stopwordsList (type : chaîne) : une liste prédéfinie de mots vides. Ne peut pas être spécifié si stopwords est spécifié. Les valeurs autorisées sont :"arabic", "armenian", "basque", "brazilian", "bulgarian", "catalan", "czech", "danish", "dutch", "english", "finnish", "french", "galician", "german", "greek", "hindi", "hungarian", "indonesian", "irish", "italian", "latvian", "norwegian", "persian", "portuguese", "romanian", "russian", "sorani", "spanish", "swedish", "thai", "turkish" ; par défaut : "english". Ne peut pas être spécifié si stopwords est spécifié. <br /><br /> ignoreCase (type : booléen) : si la valeur est true, tous les mots sont d’abord convertis en minuscules. La valeur par défaut est false.<br /><br /> removeTrailing (type : booléen) : si la valeur est true, ignore le dernier terme de recherche s’il s’agit d’un mot vide. La valeur par défaut est true.
|[synonym](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/synonym/SynonymFilter.html)|SynonymTokenFilter|Met en correspondance des synonymes constitués d’un ou plusieurs mots dans un flux de jetons.<br /><br /> **Options**<br /><br /> synonyms (type : tableau de chaînes) : obligatoire. Liste des synonymes dans l’un des deux formats suivants :<br /><br /> \- incroyable, inouï, fabuleux => étonnant : tous les termes du côté gauche du symbole => sont remplacés par tous les termes du côté droit.<br /><br /> \- incroyable, inouï, fabuleux, étonnant : une liste séparée par des virgules de mots équivalents. Définissez l’option expand pour changer la façon dont cette liste est interprétée.<br /><br /> ignoreCase (type : booléen) : convertit la casse de l’entrée pour la mise en correspondance. La valeur par défaut est false.<br /><br /> expand (type : booléen) : si la valeur est true, tous les mots de la liste des synonymes (si la notation « => » n’est pas utilisée) sont mappés les uns aux autres. <br />La liste suivante : incroyable, inouï, fabuleux, étonnant équivaut à : incroyable, inouï, fabuleux, étonnant => incroyable, inouï, fabuleux, étonnant<br /><br />- Si la valeur est false, la liste suivante : incroyable, inouï, fabuleux, étonnant équivaut à : incroyable, inouï, fabuleux, étonnant => incroyable.|  
|[trim](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/TrimFilter.html)|(le type s’applique seulement quand des options sont disponibles)  |Supprime les espaces de début et de fin des jetons. |  
|[truncate](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/TruncateTokenFilter.html)|TruncateTokenFilter|Tronque les termes à une longueur spécifique.<br /><br /> **Options**<br /><br /> length (type : entier) : par défaut : 300, maximum : 300. Obligatoire.|  
|[unique](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/RemoveDuplicatesTokenFilter.html)|UniqueTokenFilter|Élimine les jetons avec le même texte que le jeton précédent.<br /><br /> **Options**<br /><br /> onlyOnSamePosition (type : booléen) : s’il est défini, supprime les doublons seulement à la même position. La valeur par défaut est true.|  
|[uppercase](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/UpperCaseFilter.html)|(le type s’applique seulement quand des options sont disponibles)  |Normalise le texte des jetons en majuscules. |  
|[word_delimiter](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/WordDelimiterFilter.html)|WordDelimiterTokenFilter|Divise les mots en sous-mots et effectue des transformations facultatives sur les groupes de sous-mots.<br /><br /> **Options**<br /><br /> generateWordParts (type : booléen) : provoque la génération de parties de mots ; par exemple « AzureSearch » devient « Azure » « Search ». La valeur par défaut est true.<br /><br /> generateNumberParts (type : booléen) : provoque la génération de sous-mots constitués de nombres. La valeur par défaut est true.<br /><br /> catenateWords (type : booléen) : provoque la concaténation maximale des parties de mots ; par exemple « Azure-Search » devient « AzureSearch ». La valeur par défaut est false.<br /><br /> catenateNumbers (type : booléen) : provoque la concaténation maximale des parties numériques ; par exemple « 1-2 » devient « 12 ». La valeur par défaut est false.<br /><br /> catenateAll (type : booléen) : provoque la concaténation de tous les éléments qui sont des sous-mots ; par exemple « Azure-Search-1 » devient « AzureSearch1 ». La valeur par défaut est false.<br /><br /> splitOnCaseChange (type : booléen) : si la valeur est true, fractionne les mots au niveau des changements de casse ; par exemple « AzureSearch » devient « Azure » « Search ». La valeur par défaut est true.<br /><br /> preserveOriginal : fait que les mots d’origine sont conservés et ajoutés à la liste des sous-mots. La valeur par défaut est false.<br /><br /> splitOnNumerics (type : booléen) : si la valeur est true, fractionne au niveau des nombres ; par exemple « Azure1Search » devient « Azure », « 1 » « Search ». La valeur par défaut est true.<br /><br /> stemEnglishPossessive (type : booléen) : provoque la suppression du « s » final pour chaque sous-mot. La valeur par défaut est true.<br /><br /> protectedWords (type : tableau de chaînes) : jetons à protéger de la délimitation. La valeur par défaut est une liste vide.|  

 <sup>1</sup> Les types de filtres de jetons sont toujours préfixés dans le code par « #Microsoft.Azure.Search » ; ainsi, « ArabicNormalizationTokenFilter » serait spécifié sous la forme « #Microsoft.Azure.Search.ArabicNormalizationTokenFilter ».  Nous avons supprimé le préfixe pour réduire la largeur du tableau, mais n’oubliez pas de l’inclure dans votre code.  


## <a name="see-also"></a>Voir aussi  
 [API REST Recherche cognitive Azure](/rest/api/searchservice/)   
 [Analyseurs dans Recherche cognitive Azure > Exemples](search-analyzers.md#examples)    
 [Créer un index &#40;API REST Recherche cognitive Azure&#41;](/rest/api/searchservice/create-index)