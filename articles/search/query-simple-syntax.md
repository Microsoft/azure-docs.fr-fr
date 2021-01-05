---
title: Syntaxe de requête simple
titleSuffix: Azure Cognitive Search
description: Référence pour la syntaxe des requêtes simples utilisée pour les requêtes de recherche en texte intégral dans la recherche cognitive Azure.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 12/14/2020
ms.openlocfilehash: f679d6fbab57bcbcccc09b722f6b2f670df49eb2
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516590"
---
# <a name="simple-query-syntax-in-azure-cognitive-search"></a>Syntaxe de requête simple dans la recherche cognitive Azure

La recherche cognitive Azure implémente deux langages de requête basés sur Lucene : L’[analyseur de requêtes simples](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/simple/SimpleQueryParser.html) et l’[analyseur de requêtes Lucene](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/classic/package-summary.html). L’analyseur simple est plus flexible, et tente d’interpréter une requête même si elle n’est pas parfaitement composée. Pour cette raison, il s’agit de l’analyseur par défaut pour les requêtes dans Recherche cognitive Azure.

La syntaxe simple est utilisée pour les expressions de requête transmises dans le paramètre **`search`** d’une demande de [recherche dans des documents (API REST)](/rest/api/searchservice/search-documents), à ne pas confondre avec la [syntaxe OData](query-odata-filter-orderby-syntax.md) utilisée pour les expressions [ **`$filter`**](search-filters.md) et [ **`$orderby`** ](search-query-odata-orderby.md) des expressions dans la même demande. Les paramètres OData ont une syntaxe et des règles différentes pour la construction de requêtes, l’échappement de chaînes, et ainsi de suite.

Bien que l’analyseur simple soit basé sur la classe de l’[analyseur de requête simple Apache Lucene](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/simple/SimpleQueryParser.html), l’implémentation dans Recherche cognitive exclut la recherche approximative. Si vous avez besoin de la [recherche approximative](search-query-fuzzy.md), utilisez plutôt la [syntaxe de requête Lucene complète](query-lucene-syntax.md).

## <a name="example-simple-syntax"></a>Exemple (syntaxe simple)

Bien que **`queryType`** soit défini ci-dessous, il s’agit de la valeur par défaut, qui peut être omise, sauf si vous effectuez une restauration à partir d’un autre type. L’exemple suivant est une recherche sur des termes indépendants, avec une exigence stipulant que tous les documents correspondants incluent « pool ».

```http
POST https://{{service-name}}.search.windows.net/indexes/hotel-rooms-sample/docs/search?api-version=2020-06-30
{
  "queryType": "simple",
  "search": "budget hotel +pool",
  "searchMode": "all"
}
```

Le paramètre **`searchMode`** est approprié dans cet exemple. Quand des opérateurs booléens se trouvent sur la requête, vous devez généralement définir `searchMode=all` pour garantir que *tous* les critères sont satisfaits. Dans le cas contraire, vous pouvez utiliser `searchMode=any` (par défaut) qui préfère le rappel à la précision.

Pour voir des exemples supplémentaires, consultez [Exemples de syntaxe de requête simple](search-query-simple-examples.md). Pour plus d’informations sur les paramètres et la demande de requête, consultez [Rechercher dans des documents (API REST)](/rest/api/searchservice/Search-Documents).

## <a name="keyword-search-on-terms-and-phrases"></a>Recherche par mot clé sur des termes et des expressions

Les chaînes transmises au paramètre **`search`** peuvent inclure des termes ou des expressions dans les différents langages, opérateurs booléens, opérateurs de précédence, caractères génériques ou de préfixe pour les requêtes « commence par », caractères d’échappement et caractères d’encodage d’URL pris en charge. Le paramètre **`search`** est facultatif. S’il n’est pas spécifié, la recherche (`search=*` ou `search=" "`) retourne les 50 premiers documents dans un ordre arbitraire (non classé).

+ Une *recherche de termes* est une requête portant sur un ou plusieurs termes et dans laquelle un des termes est considéré comme une correspondance.

+ Une *recherche d’expression* est une expression exacte placée entre guillemets `" "`. Par exemple, si ```Roach Motel``` (sans guillemets) recherche les documents contenant ```Roach``` et/ou ```Motel``` n’importe où dans n’importe quel ordre, ```"Roach Motel"``` (avec des guillemets) établit une correspondance seulement avec les documents qui contiennent cette expression entière, avec les mots dans cet ordre (l’analyse lexicale s’applique néanmoins toujours). 

  Selon votre client de recherche, vous devrez peut-être faire précéder les guillemets de caractères d’échappement dans une recherche d’expression. Par exemple, dans une demande Postman ou POST, une recherche d’expression sur `"Roach Motel"` dans le corps de la demande est spécifiée au format suivant : `"\"Roach Motel\""`.

Par défaut, tous les termes ou expressions transmis dans le paramètre **`search`** subissent une analyse lexicale. Assurez-vous de bien comprendre le comportement de segmentation du texte en unités lexicales de l’analyseur que vous utilisez. Souvent, lorsque les résultats de la requête sont inattendus, cela peut être dû à la façon dont les termes sont segmentés en unités lexicales au moment de la requête.

Tout texte avec un ou plusieurs termes est considéré comme un point de départ valide pour l’exécution des requêtes. La recherche cognitive Azure établit une correspondance avec les documents contenant tout ou partie des conditions, y compris d’éventuelles variations détectées lors de l’analyse du texte.

Aussi simple que cela puisse paraître, un aspect de l’exécution des requêtes dans la recherche cognitive Azure est qu’elles *peuvent* produire des résultats inattendus, en augmentant le nombre des résultats de la recherche au lieu de les diminuer quand vous ajoutez plus de conditions et d’opérateurs à la chaîne d’entrée. Le fait que cette expansion se produit ou non dépend de l’inclusion d’un opérateur NOT, combiné avec une valeur pour le paramètre **`searchMode`** qui détermine comment NOT est interprété en termes de comportements de AND et de OR. Pour plus d’informations, consultez la partie sur l’opérateur NOT sous [Opérateurs booléens](#boolean-operators).

## <a name="boolean-operators"></a>Opérateurs booléens

Vous pouvez incorporer des opérateurs booléens dans une chaîne de requête pour améliorer la précision d’une correspondance. Dans la syntaxe simple, les opérateurs booléens sont basés sur des caractères. Les opérateurs de texte, tels que le mot AND, ne sont pas pris en charge.

| Caractère | Exemple | Usage |
|----------- |--------|-------|
| `+` | `pool + ocean` | Opération AND. Par exemple, `pool + ocean` stipule qu’un document doit contenir les deux termes.|
| `|` | `pool | ocean` | Une opération OR trouve une correspondance lorsque l’un des termes est trouvé. Dans l’exemple, le moteur de requête retourne des correspondances sur les documents contenant `pool` ou `ocean` ou les deux. Comme OR est l’opérateur de conjonction par défaut, vous pouvez aussi l’omettre : ainsi, `pool ocean` est l’équivalent de `pool | ocean`.|
| `-` | `pool – ocean` | Une opération NOT retourne des correspondances sur les documents qui excluent le terme. <br/><br/>Pour obtenir le comportement attendu sur une expression NOT, envisagez de définir **`searchMode=all`** sur la demande. Dans le cas contraire, sous la valeur par défaut **`searchMode=any`** , vous obtenez des correspondances sur `pool`, et des correspondances sur tous les documents qui ne contiennent pas `ocean`, ce qui peut représenter un grand nombre de documents. Dans une requête, le paramètre **`searchMode`** détermine si un terme accompagné de l’opérateur NOT est associé à d’autres termes de la requête par les opérateurs AND ou OR (en supposant qu’il n’y ait pas d’opérateur `+` ou `|` sur les autres termes). L’utilisation de **`searchMode=all`** augmente la précision des requêtes en incluant moins de résultats, et par défaut, « - » est interprété comme « AND NOT ». <br/><br/>Lorsque vous choisissez un paramètre **`searchMode`** , tenez compte des modèles d’interaction utilisateur pour les requêtes liées à différentes applications. Les utilisateurs qui recherchent des informations sont plus enclins à inclure un opérateur dans une requête, contrairement aux sites de commerce électronique qui disposent de structures de navigation intégrées. |

<a name="prefix-search"></a>

## <a name="prefix-queries"></a>Requête de préfixe

Pour les requêtes « commence par », ajoutez un opérateur de suffixe (`*`) en tant qu’espace réservé pour le reste d’un terme. Une requête de préfixe doit commencer par au moins un caractère alphanumérique avant que vous puissiez ajouter l’opérateur de suffixe.

| Caractère | Exemple | Usage |
|----------- |--------|-------|
| `*` | `lingui*` correspond à « linguistique » ou à « linguini » | L’astérisque (`*`) représente un ou plusieurs caractères de longueur arbitraire, sans tenir compte de la casse.  |

Comme pour les filtres, une requête de préfixe recherche une correspondance exacte. Par conséquent, il n’existe aucun scoring de pertinence (tous les résultats reçoivent un score de recherche de 1.0). Sachez que les requêtes de préfixe peuvent être lentes, en particulier si l’index est volumineux et le préfixe constitué d’un petit nombre de caractères. Une autre méthodologie, telle que la segmentation du texte en unités lexicales de type edge n-gram, peut s’effectuer plus rapidement.

La syntaxe simple ne prend en charge que la correspondance de préfixes. Pour la correspondance de suffixes ou d’infixes à la fin ou au milieu d’un terme, utilisez la [syntaxe Lucene complète pour la recherche de caractères génériques](query-lucene-syntax.md#bkmk_wildcard).

## <a name="escaping-search-operators"></a>Échappement des opérateurs de recherche  

Dans la syntaxe simple, les opérateurs de recherche incluent les caractères suivants : `+ | " ( ) ' \`  

Si l’un de ces caractères fait partie d’un jeton dans l’index, échappez-le en le faisant précéder d’une barre oblique inverse (`\`) dans la requête. Par exemple, supposez que vous avez utilisé un analyseur personnalisé pour la création de jetons de termes entiers, et que votre index contient la chaîne « Luxury+Hotel ». Pour obtenir une correspondance exacte sur ce jeton, insérez un caractère d’échappement : `search=luxury\+hotel`.

Pour simplifier les choses pour les cas les plus classiques, il existe deux exceptions à cette règle où l’échappement n’est pas nécessaire :  

+ L’opérateur NOT `-` doit être placé en échappement seulement s’il s’agit du premier caractère après un espace. Si le `-` apparaît au milieu (par exemple `3352CDD0-EF30-4A2E-A512-3B30AF40F3FD`), aucun échappement n’est nécessaire.

+ L’opérateur de suffixe `*` doit être placé en échappement seulement s’il s’agit du dernier caractère avant un espace. Si le `*` apparaît au milieu (par exemple `4*4=16`), aucun échappement n’est nécessaire.

> [!NOTE]  
> Par défaut, l’analyseur standard supprime et coupe les mots sur les traits d’union, les espaces, les esperluettes et autres caractères pendant l’[analyse lexicale](search-lucene-query-architecture.md#stage-2-lexical-analysis). Si vous avez besoin que les caractères spéciaux restent dans la chaîne de requête, vous aurez peut-être besoin d’un analyseur qui les conserve dans l’index. Parmi les choix possibles figurent les [analyseurs de langage](index-add-language-analyzers.md) naturel de Microsoft, qui conservent les mots avec trait d'union, ou un analyseur personnalisé pour les modèles plus complexes. Pour plus d'informations, consultez [Termes partiels, modèles et caractères spéciaux](search-query-partial-matching.md).

## <a name="encoding-unsafe-and-reserved-characters-in-urls"></a>Encodage de caractères dangereux et réservés dans les URL

Vérifiez que tous les caractères non sécurisés et réservés sont encodés dans une URL. Par exemple, « # » est un caractère non sécurisé, car il s'agit d'un identificateur de fragment/d'ancrage au sein d'une URL. Le caractère doit être encodé en `%23` s’il est utilisé dans une URL. « & » et « = » sont des exemples de caractères réservés, car ils délimitent les paramètres et spécifient des valeurs dans la Recherche cognitive Azure. Pour plus d’informations, consultez la page [RFC1738 : Uniform Resource Locator (URL)](https://www.ietf.org/rfc/rfc1738.txt).

Les caractères dangereux sont ``" ` < > # % { } | \ ^ ~ [ ]``. Les caractères réservés sont `; / ? : @ = + &`.

## <a name="special-characters"></a>Caractères spéciaux

Dans certains cas, vous rechercherez un caractère spécial, tel que l’émoji « ❤ » ou le signe « € ». Le cas échéant, assurez-vous que l’analyseur que vous utilisez n’ignore pas ces caractères. L’analyseur standard ignore de nombreux caractères spéciaux, en les excluant de votre index.

Les analyseurs qui marquent les caractères spéciaux incluent l’analyseur d’« espace blanc », qui prend en compte toutes les séquences de caractères séparées par des espaces blancs comme des jetons, donc la chaîne « ❤ » est considérée comme un jeton. En outre, un analyseur de langage comme l’analyseur Microsoft English (« en.microsoft ») considère la chaîne « € » comme un jeton. Vous pouvez [tester un analyseur](/rest/api/searchservice/test-analyzer) pour voir quels jetons il génère pour une requête donnée.

Lorsque vous utilisez des caractères Unicode, assurez-vous que les symboles sont correctement placés dans une séquence d’échappement dans l’URL de la requête (par exemple, pour « ❤ » utilisez la séquence d’échappement `%E2%9D%A4+`). Postman effectue cette traduction automatiquement.  

## <a name="precedence-grouping"></a>Calcul prioritaire

Vous pouvez utiliser des parenthèses pour créer des sous-requêtes, en incluant des opérateurs au sein de l’instruction entre parenthèses. Par exemple, `motel+(wifi|luxury)` recherche les documents contenant le terme « motel », et « wifi » ou « luxury » (ou les deux).

## <a name="query-size-limits"></a> Limites de taille des requêtes

Si votre application génère des requêtes de recherche par programmation, nous vous recommandons de la concevoir de façon à ce qu’elle ne génère pas des requêtes d’une taille illimitée. 

+ Pour GET, la longueur de l’URL ne peut pas dépasser 8 Ko.

+ Pour POST (et toute autre demande), où le corps de la demande inclut `search` et d’autres paramètres tels que `filter` et `orderby`, la taille maximale est de 16 Mo, où le nombre maximal de clauses dans `search` (expressions séparées par AND, OR, etc.) est 1024. Il existe également une limite d’environ 32 Ko pour la taille d’un terme individuel dans une requête. Pour plus d’informations, consultez les [limites associées aux demandes d’API](search-limits-quotas-capacity.md#api-request-limits).

## <a name="next-steps"></a>Étapes suivantes

Si vous devez construire des requêtes par programme, consultez [Recherche en texte intégral dans Recherche cognitive Azure](search-lucene-query-architecture.md) pour comprendre les étapes du traitement des requêtes et les implications de l’analyse de texte.

Vous pouvez également consulter les articles suivants pour en savoir plus sur la construction des requêtes :

+ [Exemples de requêtes pour une recherche simple](search-query-simple-examples.md)
+ [Exemples de requêtes pour une recherche Lucene complète](search-query-lucene-examples.md)
+ [API REST de recherche de documents](/rest/api/searchservice/Search-Documents)
+ [Syntaxe de requête Lucene](query-lucene-syntax.md)
+ [Syntaxe des expressions de filtre et de sélection (OData)](query-odata-filter-orderby-syntax.md)