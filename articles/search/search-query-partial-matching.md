---
title: Termes partiels, modèles et caractères spéciaux
titleSuffix: Azure Cognitive Search
description: Utilisez des requêtes de caractères génériques, d’expression régulière et de préfixes pour faire correspondre des termes entiers ou partiels dans une demande de requête de Recherche cognitive Azure. Les critères difficiles à mettre en correspondance qui incluent des caractères spéciaux peuvent être résolus à l’aide de la syntaxe complète de la requête et des analyseurs personnalisés.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 12/03/2020
ms.openlocfilehash: 79ba186351cc145e012658abc30572e99b123dbb
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96573984"
---
# <a name="partial-term-search-and-patterns-with-special-characters-hyphens-wildcard-regex-patterns"></a>Recherche de termes partiels et modèles avec des caractères spéciaux (traits d’union, caractères génériques, expressions régulières, modèles)

Une *recherche de terme partiel* fait référence à des requêtes composées de fragments de termes où, au lieu d’un terme entier, vous pouvez avoir juste le début, le milieu ou la fin du terme (elle est parfois appelée requête de préfixe, d’infixe ou de suffixe). Une recherche de terme partiel peut inclure une combinaison de fragments, souvent avec des caractères spéciaux comme des traits d’union, des tirets ou des barres obliques qui font partie de la chaîne de requête. Les cas d’usage courants incluent les parties d’un numéro de téléphone, une URL, des codes ou des mots composés avec trait d’union.

La recherche de terme partiel et les chaînes de requête qui comprennent des caractères spéciaux peuvent être problématiques si l’index n’a pas de jetons au format attendu. Pendant la [phase d’analyse lexicale](search-lucene-query-architecture.md#stage-2-lexical-analysis) de l’indexation (en supposant une utilisation de l’analyseur standard par défaut), les caractères spéciaux sont ignorés, les mots composées sont fractionnés et l’espace blanc est supprimé ; ceci peut provoquer l’échec des requêtes quand aucune correspondance n’est trouvée. Par exemple, un numéro de téléphone comme `+1 (425) 703-6214` (segmenté en unités `"1"`, `"425"`, `"703"`, `"6214"`) n’apparaît pas dans une requête `"3-62"`, car ce contenu n’existe pas dans l’index en réalité. 

La solution consiste à appeler pendant l’indexation un analyseur qui conserve une chaîne complète, y compris les espaces et les caractères spéciaux si nécessaire, afin que vous puissiez inclure les espaces et les caractères dans votre chaîne de requête. De même, le fait d’avoir une chaîne complète qui n’est pas tokenisée en parties plus petites autorise des critères spéciaux pour les requêtes « commence par » ou « se termine par », où le modèle que vous fournissez peut être évalué par rapport à un terme qui n’est pas transformé par l’analyse lexicale. La création d’un champ supplémentaire pour une chaîne intacte, ainsi que l’utilisation d’un analyseur à conservation du contenu qui émet des jetons de terme entier, est la solution pour les critères spéciaux et pour la correspondance sur les chaînes de requête qui incluent des caractères spéciaux.

> [!TIP]
> Si vous connaissez Postman et les API REST, [téléchargez la collection d’exemples de requêtes](https://github.com/Azure-Samples/azure-search-postman-samples/) pour interroger les termes partiels et les caractères spéciaux décrits dans cet article.

## <a name="about-partial-term-search"></a>À propos de la recherche de terme partiel

Recherche cognitive Azure recherche des termes tokenisés entiers dans l’index, et ne trouvera pas de correspondance sur un terme partiel, sauf si vous incluez des opérateurs génériques d’espace réservé (`*` et `?`) ou que vous mettez en forme la requête en tant qu’expression régulière. Les termes partiels sont spécifiés à l’aide des techniques suivantes :

+ Les [requêtes d’expression régulière](query-lucene-syntax.md#bkmk_regex) peuvent être n’importe quelle expression régulière valide sous Apache Lucene. 

+ Les [opérateurs génériques avec correspondance de préfixe](query-simple-syntax.md#prefix-search) font référence à un modèle généralement reconnu qui comprend le début d’un terme, suivi d’opérateurs de suffixe `*` ou `?`, comme pour la mise en correspondance de `search=cap*` sur « Cap’n Jack’s Waterfront Inn » ou « Gacc Capital ». La mise en correspondance de préfixe est prise en charge dans la syntaxe de requête Lucene simple et complète.

+ La [mise en correspondance générique avec infixe et suffixe](query-lucene-syntax.md#bkmk_wildcard) place les opérateurs `*` et `?` à l’intérieur ou au début d’un terme, et nécessite une syntaxe d’expression régulière (où l’expression est entourée de barres obliques). Par exemple, la chaîne de requête (`search=/.*numeric*./`) retourne les résultats sur « alphanumeric » et « alphanumerical » en tant que correspondances de suffixe et d’infixe.

Pour la recherche de terme partiel ou de modèle, et quelques autres formes de requête telles que la recherche approximative, les analyseurs ne sont pas utilisés au moment de la requête. Pour ces formes de requête, que l’analyseur détecte par la présence d’opérateurs et de délimiteurs, la chaîne de requête est passée au moteur sans analyse lexicale. Pour ces formes de requête, l’analyseur spécifié sur le champ est ignoré.

> [!NOTE]
> Quand une chaîne de requête partielle inclut des caractères, tels que des barres obliques dans un fragment d’URL, vous devrez peut-être ajouter des caractères d’échappement. Dans JSON, une barre oblique `/` est placée dans une séquence d’échappement avec une barre oblique inverse `\`. Par conséquent, `search=/.*microsoft.com\/azure\/.*/` est la syntaxe pour le fragment d’URL « microsoft.com/azure/ ».

## <a name="solving-partialpattern-search-problems"></a>Résolution des problèmes liés à la recherche partielle/de modèles

Quand vous devez effectuer une recherche sur des fragments, modèles ou caractères spéciaux, vous pouvez remplacer l’analyseur par défaut par un analyseur personnalisé qui fonctionne selon des règles de tokenisation plus simples, préservant la chaîne entière dans l’index. En prenant un peu de recul, l’approche ressemble à ceci :

1. Définir un champ pour stocker une version intacte de la chaîne (en supposant que vous voulez du texte analysé et non analysé au moment de la requête)
1. Évaluer et choisir parmi les différents analyseurs qui émettent des jetons au niveau de précision approprié
1. Attribuer l’analyseur au champ
1. Générer et tester l’index

> [!TIP]
> L’évaluation des analyseurs est un processus itératif qui nécessite de fréquentes regénérations de l’index. Vous pouvez faciliter cette étape en utilisant Postman, les API REST pour [créer un index](/rest/api/searchservice/create-index), [supprimer un index](/rest/api/searchservice/delete-index), [charger des documents](/rest/api/searchservice/addupdate-or-delete-documents) et [rechercher des documents](/rest/api/searchservice/search-documents). Pour charger des documents, le corps de la requête doit contenir un petit jeu de données représentatif que vous souhaitez tester (par exemple, un champ avec des numéros de téléphone ou des codes de produit). Ces API étant dans la même collection Postman, vous pouvez effectuer rapidement ces étapes.

## <a name="1---create-a-dedicated-field"></a>1 - Créer un champ dédié

Les analyseurs déterminent comment les termes sont tokenisés dans un index. Les analyseurs étant affectés par champ, vous pouvez créer des champs dans votre index pour optimiser les différents scénarios. Par exemple, vous pouvez définir « featureCode » et « featureCodeRegex » pour prendre en charge une recherche en texte intégral normale sur le premier et des critères spéciaux avancés sur le second. Les analyseurs affectés à chaque champ déterminent comment le contenu de chaque champ est tokenisé dans l’index.  

```json
{
  "name": "featureCode",
  "type": "Edm.String",
  "retrievable": true,
  "searchable": true,
  "analyzer": null
},
{
  "name": "featureCodeRegex",
  "type": "Edm.String",
  "retrievable": true,
  "searchable": true,
  "analyzer": "my_custom_analyzer"
},
```

<a name="set-an-analyzer"></a>

## <a name="2---set-an-analyzer"></a>2 - Définir un analyseur

Lors du choix d’un analyseur qui produit des jetons à terme entier, les analyseurs suivants sont des choix courants :

| Analyseur | Comportements |
|----------|-----------|
| [Analyseurs de langage](index-add-language-analyzers.md) | Conserve les traits d’Union dans les mots composés, les chaînes, les mutations de voyelles et les formes verbales. Si les modèles de requête incluent des tirets, l’utilisation d’un analyseur de langage peut suffire. |
| [keyword](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordAnalyzer.html) | Le contenu du champ entier est segmenté en jetons en tant que terme unique. |
| [whitespace](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/WhitespaceAnalyzer.html) | Sépare sur les espaces blancs seulement. Les termes qui incluent des tirets ou d’autres caractères sont traités comme un jeton unique. |
| [analyseur personnalisé](index-add-custom-analyzers.md) | (recommandé) La création d’un analyseur personnalisé vous permet de spécifier à la fois le générateur de jetons et le filtre de jeton. Les analyseurs précédents doivent être utilisés tels quels. Un analyseur personnalisé vous permet de choisir les générateurs et les filtres de jetons à utiliser. <br><br>Une combinaison recommandée est le [générateur de jetons keyword](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordTokenizer.html) avec un [filtre de jeton lowercase](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/LowerCaseFilter.html). En soi, l’[analyseur keyword](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordAnalyzer.html) prédéfini ne met pas en minuscules les caractères majuscules, ce qui peut entraîner l’échec des requêtes. Un analyseur personnalisé vous donne un mécanisme pour ajouter le filtre de jeton lowercase. |

Si vous utilisez un outil de test de l’API Web tel que Postman, vous pouvez ajouter l’[appel REST de l’analyseur de test](/rest/api/searchservice/test-analyzer) pour inspecter la sortie segmentée en jetons.

Vous devez avoir un index rempli avec lequel travailler. À partir d’un index existant et d’un champ contenant des tirets ou des termes partiels, vous pouvez essayer différents analyseurs sur des termes spécifiques pour voir quels jetons sont émis.  

1. Vérifiez d’abord l’analyseur Standard pour voir comment les termes sont tokenisés par défaut.

   ```json
   {
   "text": "SVP10-NOR-00",
   "analyzer": "standard"
   }
    ```

1. Évaluez la réponse pour voir comment le texte est segmenté en jetons dans l’index. Notez que chaque terme est en minuscules, que les traits d’union ont été supprimés et les sous-chaînes divisées en jetons individuels. Seules les requêtes qui correspondent à ces jetons retourneront ce document dans les résultats. Une requête qui comprend « 10-NOR » échouera.

    ```json
    {
        "tokens": [
            {
                "token": "svp10",
                "startOffset": 0,
                "endOffset": 5,
                "position": 0
            },
            {
                "token": "nor",
                "startOffset": 6,
                "endOffset": 9,
                "position": 1
            },
            {
                "token": "00",
                "startOffset": 10,
                "endOffset": 12,
                "position": 2
            }
        ]
    }
    ```
1. Maintenant, modifiez la requête pour utiliser l’analyseur `whitespace` ou `keyword` :

    ```json
    {
    "text": "SVP10-NOR-00",
    "analyzer": "keyword"
    }
    ```

1. À présent, la réponse se compose d’un jeton unique, en majuscules, avec des tirets conservés en tant que partie de la chaîne. Si vous devez effectuer une recherche sur un modèle ou un terme partiel tel que « 10-NOR », le moteur de requête dispose maintenant de la base pour trouver une correspondance.


    ```json
    {

        "tokens": [
            {
                "token": "SVP10-NOR-00",
                "startOffset": 0,
                "endOffset": 12,
                "position": 0
            }
        ]
    }
    ```
> [!Important]
> N’oubliez pas que les analyseurs de requêtes mettent souvent les termes en minuscules dans une expression de recherche lors de la création de l’arborescence de requêtes. Si vous utilisez un analyseur qui ne met pas les entrées de texte en minuscules durant l’indexation, et que vous n’obtenez pas les résultats attendus, cela peut être la raison. La solution consiste à ajouter un filtre de jeton lowercase, comme décrit dans la section « Utiliser des analyseurs personnalisés », ci-dessous.

## <a name="3---configure-an-analyzer"></a>3 - Configurer un analyseur
 
Que vous évaluiez des analyseurs ou que vous avanciez avec une configuration spécifique, vous devrez spécifier l’analyseur sur la définition de champ et éventuellement configurer l’analyseur lui-même si vous n’utilisez pas d’analyseur intégré. Lorsque vous changez d’analyseur, vous devez généralement reconstruire l’index (supprimer, recréer et recharger). 

### <a name="use-built-in-analyzers"></a>Utiliser des analyseurs intégrés

Les analyseurs intégrés ou prédéfinis peuvent être spécifiés par leur nom dans la propriété `analyzer` d’une définition de champ, sans qu’une configuration supplémentaire ne soit requise dans l’index. L’exemple suivant montre comment définir l’analyseur `whitespace` sur un champ. 

Pour d’autres scénarios et pour en savoir plus sur les autres analyseurs intégrés, consultez [Liste des analyseurs prédéfinis](./index-add-custom-analyzers.md#predefined-analyzers-reference). 

```json
    {
      "name": "phoneNumber",
      "type": "Edm.String",
      "key": false,
      "retrievable": true,
      "searchable": true,
      "analyzer": "whitespace"
    }
```

### <a name="use-custom-analyzers"></a>Utiliser des analyseurs personnalisés

Si vous utilisez un [analyseur personnalisé](index-add-custom-analyzers.md), définissez-le dans l’index avec une combinaison définie par l’utilisateur d’un générateur de jetons et d’un filtre de jeton, avec les paramètres de configuration possibles. Ensuite, référencez-le sur une définition de champ, comme vous le feriez pour un analyseur intégré.

Lorsque l’objectif est la segmentation du texte en termes entiers, un analyseur personnalisé qui se compose d’un **générateur de jetons keyword** et d’un **filtre de jeton lowercase** est recommandé.

+ Le générateur de jetons keyword crée un jeton unique pour tout le contenu d’un champ.
+ Le filtre de jeton lowercase transforme les lettres majuscules en caractères minuscules. Les analyseurs de requêtes mettent généralement en minuscules toutes les entrées de texte qui sont en majuscules. La mise en minuscules homogénéise les entrées avec les termes tokenisés.

L’exemple suivant illustre un analyseur personnalisé qui fournit le générateur de jetons keyword et un filtre de jetons lowercase.

```json
{
"fields": [
  {
  "name": "accountNumber",
  "analyzer":"myCustomAnalyzer",
  "type": "Edm.String",
  "searchable": true,
  "filterable": true,
  "retrievable": true,
  "sortable": false,
  "facetable": false
  }
],

"analyzers": [
  {
  "@odata.type":"#Microsoft.Azure.Search.CustomAnalyzer",
  "name":"myCustomAnalyzer",
  "charFilters":[],
  "tokenizer":"keyword_v2",
  "tokenFilters":["lowercase"]
  }
],
"tokenizers":[],
"charFilters": [],
"tokenFilters": []
```

> [!NOTE]
> Le générateur de jetons `keyword_v2` et le filtre de jeton `lowercase` sont connus du système et utilisent leurs configurations par défaut, c’est pourquoi vous pouvez les référencer par leur nom sans avoir à les définir au préalable.

## <a name="4---build-and-test"></a>4 - Créer et tester

Une fois que vous avez défini un index avec des analyseurs et des définitions de champ qui prennent en charge votre scénario, chargez des documents contenant des chaînes représentatives afin de pouvoir tester des requêtes de chaînes partielles. 

Les sections précédentes ont expliqué la logique. Cette section parcourt chaque API que vous devez appeler lors du test de votre solution. Comme indiqué précédemment, si vous utilisez un outil de test web interactif comme Postman, vous pouvez parcourir ces tâches rapidement.

+ [Supprimer l’index](/rest/api/searchservice/delete-index) supprime un index existant du même nom afin que vous puissiez le recréer.

+ [Créer un index](/rest/api/searchservice/create-index) crée la structure d’index sur votre service de recherche, notamment des définitions et des champs d’analyseur avec la spécification d’un analyseur.

+ [Charger des documents](/rest/api/searchservice/addupdate-or-delete-documents) importe des documents ayant la même structure que votre index, ainsi que du contenu pouvant faire l’objet d’une recherche. Après cette étape, votre index est prêt à être interrogé ou testé.

+ L’[analyseur de test](/rest/api/searchservice/test-analyzer) a été introduit dans [Définir un analyseur](#set-an-analyzer). Testez certaines chaînes de votre index à l’aide de divers analyseurs pour comprendre comment les termes sont tokenisés.

+ [Rechercher dans les documents](/rest/api/searchservice/search-documents) explique comment construire une demande de requête, en utilisant une [syntaxe simple](query-simple-syntax.md) ou la [syntaxe Lucene complète](query-lucene-syntax.md) pour les expressions génériques et les expressions régulières.

  Pour les requêtes de termes partiels, telles que l’interrogation « 3-6214 » pour trouver une correspondance sur « + 1 (425) 703-6214 », vous pouvez utiliser la syntaxe simple : `search=3-6214&queryType=simple`.

  Pour les requêtes d’infixe et de suffixe, telles que l’interrogation « num » ou « numérique pour trouver une correspondance sur «alphanumérique », utilisez la syntaxe Lucene complète et une expression régulière : `search=/.*num.*/&queryType=full`

## <a name="tune-query-performance"></a>Régler les performances des requêtes

Si vous implémentez la configuration recommandée qui comprend le générateur de jetons keyword_v2 et le filtre de jeton lowercase, vous remarquerez peut-être une baisse des performances de requêtes en raison du traitement du filtre de jeton supplémentaire sur les jetons existants dans votre index. 

L’exemple suivant ajoute un [EdgeNGramTokenFilter](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ngram/EdgeNGramTokenizer.html) pour que les correspondances de préfixe soient plus rapides. Des jetons supplémentaires sont générés pour les combinaisons de 2 à 25 caractères qui incluent des caractères : (pas seulement MS, MSF, MSFT, MSFT/, MSFT/S, MSFT/SQ, MSFT/SQL). 

Comme vous pouvez l’imaginer, les résultats supplémentaires de segmentation du texte en unités lexicales entraîne un plus grand index. Si vous disposez d’une capacité suffisante pour accueillir l’index plus volumineux, cette approche offrant un temps de réponse plus rapide peut constituer une meilleure solution.

```json
{
"fields": [
  {
  "name": "accountNumber",
  "analyzer":"myCustomAnalyzer",
  "type": "Edm.String",
  "searchable": true,
  "filterable": true,
  "retrievable": true,
  "sortable": false,
  "facetable": false
  }
],

"analyzers": [
  {
  "@odata.type":"#Microsoft.Azure.Search.CustomAnalyzer",
  "name":"myCustomAnalyzer",
  "charFilters":[],
  "tokenizer":"keyword_v2",
  "tokenFilters":["lowercase", "my_edgeNGram"]
  }
],
"tokenizers":[],
"charFilters": [],
"tokenFilters": [
  {
  "@odata.type":"#Microsoft.Azure.Search.EdgeNGramTokenFilterV2",
  "name":"my_edgeNGram",
  "minGram": 2,
  "maxGram": 25,
  "side": "front"
  }
]
```

## <a name="next-steps"></a>Étapes suivantes

Cet article explique comment les analyseurs contribuent aux problèmes de requête et à les résoudre. À l’étape suivante, examinez de plus près l’impact de l’analyseur sur l’indexation et le traitement des requêtes. En particulier, envisagez d’utiliser l’API Analyze Text pour retourner une sortie segmentée en jetons afin que vous puissiez voir exactement ce qu’un analyseur crée pour votre index.

+ [Analyseurs de langage](search-language-support.md)
+ [Analyseurs pour le traitement de texte dans Recherche cognitive Azure](search-analyzers.md)
+ [API (REST) Analyze Text](/rest/api/searchservice/test-analyzer)
+ [Fonctionnement de la recherche en texte intégral (architecture de requête)](search-lucene-query-architecture.md)