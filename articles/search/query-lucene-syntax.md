---
title: Syntaxe de requête Lucene
titleSuffix: Azure Cognitive Search
description: Référence pour la syntaxe de requête Lucene complète, telle qu’elle est utilisée dans Recherche cognitive Azure pour les caractères génériques, la recherche approximative, RegEx et d’autres constructions de requête avancées.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 12/14/2020
ms.openlocfilehash: 0dbf418d0a673dd0799f0f638e454c484f837fd7
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516604"
---
# <a name="lucene-query-syntax-in-azure-cognitive-search"></a>Syntaxe de requête Lucene dans la Recherche cognitive Azure

Lorsque vous créez des alertes, vous pouvez choisir la syntaxe [Analyseur de requêtes Lucene](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/classic/package-summary.html) pour des formes de requêtes spécialisées : caractère générique, recherche approximative, recherche de proximité et expressions régulières. La plus grande partie de la syntaxe de l’analyseur de requêtes Lucene est [implémentée telle quelle dans Recherche cognitive Azure](search-lucene-query-architecture.md), à l’exception des *recherches de plage*, qui sont construites via des expressions **`$filter`** . 

La syntaxe Lucene complète est utilisée pour des expressions de requête transmises dans le paramètre **`search`** d’une requête [Rechercher des documents (API REST)](/rest/api/searchservice/search-documents), à ne pas confondre avec la [syntaxe OData](query-odata-filter-orderby-syntax.md) utilisée pour les expressions [ **`$filter`**](search-filters.md) et [ **`$orderby`** ](search-query-odata-orderby.md) dans la même demande. Les paramètres OData ont une syntaxe et des règles différente pour la construction de requêtes, l’échappement de chaînes, et ainsi de suite.

## <a name="example-full-syntax"></a>Exemple (syntaxe complète)

Définissez le paramètre **`queryType`** pour spécifier la syntaxe Lucene complète. L’exemple suivant appelle une recherche dans le champ et un promotion de terme. Cette requête recherche des hôtels où le champ de catégorie contient le terme « budget ». Tout document contenant l’expression « recently renovated » est classé plus haut en raison de la valeur de promotion de terme (3).  

```http
POST /indexes/hotels-sample-index/docs/search?api-version=2020-06-30
{
  "queryType": "full",
  "search": "category:budget AND \"recently renovated\"^3",
  "searchMode": "all"
}
```

Le paramètre **`searchMode`** est approprié dans cet exemple. Quand des opérateurs se trouvent sur la requête, vous devez généralement définir `searchMode=all` pour garantir que *tous* les critères sont satisfaits.  

Pour obtenir des exemples supplémentaires, consultez [Exemples de syntaxe de requête Lucene](search-query-lucene-examples.md). Pour plus d’informations sur la demande et les paramètres de requête, consultez [Rechercher des documents (API REST)](/rest/api/searchservice/Search-Documents).

## <a name="syntax-fundamentals"></a><a name="bkmk_syntax"></a> Principes de base de la syntaxe  

Les principes de base suivants de la syntaxe s’appliquent à toutes les requêtes qui utilisent la syntaxe Lucene.  

### <a name="operator-evaluation-in-context"></a>Évaluation des opérateurs en contexte

L’emplacement détermine si un symbole est interprété comme un opérateur ou simplement comme un autre caractère dans une chaîne.

Par exemple, dans la syntaxe complète Lucene, le tilde (~) est utilisé pour la recherche approximative et la recherche de proximité. Quand il est placé après une expression entre guillemets, « ~ » appelle la recherche de proximité. Quand il est placé à la fin d’un terme, « ~ » appelle la recherche approximative.

Au sein d’un terme, comme « business~analyst », le caractère n’est pas évalué comme opérateur. Dans ce cas, en supposant que la requête est une requête de terme ou d’expression, la [recherche en texte intégral](search-lucene-query-architecture.md) avec [analyse lexicale](search-lucene-query-architecture.md#stage-2-lexical-analysis) supprime le ~ et décompose le terme « business~analyst » en deux : business OR analyst.

L’exemple ci-dessus concerne le tilde (~), mais le même principe s’applique à chaque opérateur.

### <a name="escaping-special-characters"></a>Échappement des caractères spéciaux

Pour utiliser un opérateur de recherche dans le texte de recherche, placez le caractère dans une séquence d'échappement en le faisant précéder d'une barre oblique inverse (`\`). Par exemple, pour une recherche avec caractères génériques sur `https://`, sachant que `://` fait partie de la chaîne de requête, vous devez spécifier `search=https\:\/\/*`. De même, un modèle de numéro de téléphone placé dans une séquence d'échappement peut se présenter comme suit : `\+1 \(800\) 642\-7676`.

Les caractères spéciaux qui doivent être placés dans une séquence d'échappement sont les suivants :  
`+ - & | ! ( ) { } [ ] ^ " ~ * ? : \ /`  

> [!NOTE]  
> Bien que l'échappement maintienne les jetons ensemble, l'[analyse lexicale](search-lucene-query-architecture.md#stage-2-lexical-analysis) pendant l'indexation peut les supprimer. Par exemple, l'analyseur Lucene standard coupe les mots sur les traits d'union, les espaces et autres caractères. Si vous avez besoin de caractères spéciaux dans la chaîne de requête, vous aurez peut-être également besoin d'un analyseur qui les conserve dans l'index. Parmi les choix possibles figurent les [analyseurs de langage](index-add-language-analyzers.md) naturel de Microsoft, qui conservent les mots avec trait d'union, ou un analyseur personnalisé pour les modèles plus complexes. Pour plus d'informations, consultez [Termes partiels, modèles et caractères spéciaux](search-query-partial-matching.md).

### <a name="encoding-unsafe-and-reserved-characters-in-urls"></a>Encodage de caractères dangereux et réservés dans les URL

Vérifiez que tous les caractères dangereux et réservés dans une URL sont encodés. Par exemple, « # » est un caractère non sécurisé, car il s'agit d'un identificateur de fragment/d'ancrage au sein d'une URL. Le caractère doit être encodé en `%23` s’il est utilisé dans une URL. « & » et « = » sont des exemples de caractères réservés, car ils délimitent les paramètres et spécifient des valeurs dans la Recherche cognitive Azure. Pour plus d’informations, consultez [RFC1738 : Uniform Resource Locators (URL)](https://www.ietf.org/rfc/rfc1738.txt).

Les caractères dangereux sont ``" ` < > # % { } | \ ^ ~ [ ]``. Les caractères réservés sont `; / ? : @ = + &`.

## <a name="boolean-operators"></a><a name="bkmk_boolean"></a> Opérateurs booléens

Vous pouvez incorporer des opérateurs booléens dans une chaîne de requête pour améliorer la précision d’une correspondance. La syntaxe complète prend en charge des opérateurs de texte en plus des opérateurs de caractères. Spécifiez toujours les opérateurs booléens de texte (AND, OR, NOT) tout en majuscules.

|Opérateur de texte | Caractère | Exemple | Usage |
|--------------|----------- |--------|-------|
| AND | `&`, `+` | `wifi + luxury` | Spécifie les termes qu’une correspondance doit contenir. Dans l’exemple, le moteur de requête recherche les documents contenant à la fois `wifi` et `luxury`. Le caractère plus (`+`) est utilisé pour les termes obligatoirement présents. Par exemple, `+wifi +luxury` stipule que les deux termes doivent apparaître quelque part dans le champ d’un même document.|
| OR | `|` | `wifi | luxury` | Trouve une correspondance quand un terme est trouvé. Dans l’exemple, le moteur de requête renvoie des correspondances sur les documents contenant le terme `wifi` ou `luxury`, ou les deux termes. Comme OR est l’opérateur de conjonction par défaut, vous pouvez aussi l’omettre : ainsi, `wifi luxury` est l’équivalent de `wifi | luxury`.|
| NOT | `!`, `-` | `wifi –luxury` | Retourne des correspondances sur des documents qui excluent le terme. Par exemple, `wifi –luxury` recherche les documents qui contiennent le terme `wifi`, mais pas le terme `luxury`. <br/><br/>Dans une requête, le paramètre `searchMode` détermine si un terme accompagné de l'opérateur NOT est associé à d'autres termes de la requête par les opérateurs AND ou OR (en supposant qu'il n'y ait pas d'opérateur `+` ou `|` sur les autres termes). Les valeurs valides sont `any` ou `all`.  <br/><br/>`searchMode=any` augmente le rappel des requêtes en incluant plus de résultats, et par défaut `-` est interprété comme « OR NOT ». Par exemple, `wifi -luxury` établit une correspondance avec les documents qui contiennent le terme `wifi` ou avec ceux qui ne contiennent pas le terme `luxury`.  <br/><br/>`searchMode=all` augmente la précision des requêtes en incluant moins de résultats et, par défaut, « - » est interprété comme « AND NOT ». Par exemple, `wifi -luxury` établit une correspondance avec les documents qui contiennent le terme `wifi` et ne contiennent pas le terme « luxury ». Il s’agit sans doute d’un comportement plus intuitif pour l’opérateur `-`. Ainsi, préférez `searchMode=all` à `searchMode=any` si vous souhaitez optimiser la précision des recherches plutôt que le rappel, *et* si vos utilisateurs utilisent fréquemment l'opérateur `-` dans les recherches.<br/><br/>Lorsque vous choisissez un paramètre `searchMode`, tenez compte des modèles d'interaction utilisateur pour les requêtes liées à différentes applications. Les utilisateurs qui recherchent des informations sont plus enclins à inclure un opérateur dans une requête, contrairement aux sites de commerce électronique qui disposent de structures de navigation intégrées. |

##  <a name="fielded-search"></a><a name="bkmk_fields"></a> Recherche par champ

Vous pouvez définir une opération de recherche par champ avec la syntaxe `fieldName:searchExpression`, où l’expression de recherche peut être un mot ou une phrase, ou une expression plus complexe entre parenthèses, éventuellement avec des opérateurs booléens. Voici quelques exemples :  

- genre:jazz NOT history  

- artists:("Miles Davis" "John Coltrane")

Veillez à placer les chaînes multiples entre guillemets si vous voulez que les deux chaînes soient évaluées comme une seule entité, comme ici où deux artistes distincts sont recherchés dans le champ `artists`.  

Le champ spécifié dans `fieldName:searchExpression` doit être un champ `searchable`.  Pour plus d’informations sur l’utilisation des attributs d’index dans les définitions de champs, consultez [Créer un index ](/rest/api/searchservice/create-index).  

> [!NOTE]
> Lorsque vous utilisez des expressions de recherche par champ, il est inutile d’utiliser le paramètre `searchFields`, car chaque expression de recherche par champ a un nom de champ spécifié explicitement. Cependant, vous pouvez toujours utiliser le paramètre `searchFields` si vous voulez exécuter une requête où certaines parties sont limitées à un champ spécifique, et le reste peut s’appliquer à plusieurs champs. Par exemple, la requête `search=genre:jazz NOT history&searchFields=description` ne correspondrait à `jazz` qu’au niveau du champ `genre`, alors qu’elle correspondrait au champ `NOT history` avec le champ `description`. Le nom du champ fourni dans `fieldName:searchExpression` a toujours priorité sur le paramètre `searchFields`, c’est pourquoi dans cet exemple, nous n’avons pas besoin d’inclure `genre` dans le paramètre `searchFields`.

##  <a name="fuzzy-search"></a><a name="bkmk_fuzzy"></a> Recherche approximative

Une recherche approximative recherche des correspondances dans des termes à la construction similaire, en développant un terme jusqu’au maximum de 50 termes qui répondent aux critères de distance de deux ou moins. Pour plus d’informations, consultez [Recherche approximative](search-query-fuzzy.md).

Pour effectuer une recherche approximative, utilisez le symbole « ~ » (tilde) à la fin d’un mot avec un paramètre facultatif, un nombre compris entre 0 et 2 (la valeur par défaut), qui spécifie la distance de modification. Par exemple, « blue~ » ou « blue~1 » retournent « blue », « blues » et « glue ».

La recherche partielle est applicable uniquement pour les termes, et non les expressions, mais vous pouvez ajouter le tilde à chaque terme individuel dans un nom en plusieurs parties ou une expression. Par exemple, « Unviersté~ de~ « Wshington~ » correspondrait à « Université de Washington ».
 
##  <a name="proximity-search"></a><a name="bkmk_proximity"></a> Recherche de proximité

Les recherches de proximité servent à rechercher des termes qui sont proches les uns des autres dans un document. Insérez un signe tilde « ~ » à la fin d’une expression, suivi du nombre de mots qui créent la limite de proximité. Par exemple, `"hotel airport"~5` recherche les termes « hotel » et « airport » distants de 5 mots ou moins dans un document.  

##  <a name="term-boosting"></a><a name="bkmk_termboost"></a> Promotion de termes

La promotion de termes signifie que vous pouvez accorder à un document un rang plus élevé s’il contient le terme promu, par rapport aux documents qui ne contiennent pas ce terme. À ne pas confondre avec les profils de score qui promeuvent certains champs, plutôt que des termes spécifiques.  

L’exemple suivant permet d’illustrer les différences entre les deux. Supposons qu’un profil de score promeuve les correspondances dans un certain champ, disons *genre* dans l’[exemple musicstoreindex](index-add-scoring-profiles.md#bkmk_ex). En utilisant la promotion de termes, vous pouvez promouvoir encore plus certains termes de recherche. Par exemple, `rock^2 electronic` promeut les documents qui contiennent les termes de recherche dans le champ genre en les classant mieux que d’autres champs de recherche de l’index. Par ailleurs, les documents qui contiennent le terme de recherche *rock* bénéficient d’un meilleur classement que ceux qui contiennent le terme de recherche *electronic* en raison de la valeur de promotion du terme (2).  

 Pour promouvoir un terme, utilisez le signe « ^ » (caret) avec un facteur de promotion (un nombre) à la fin du terme recherché. Vous pouvez également promouvoir des expressions. Plus le facteur de promotion est élevé, plus le terme est pertinent par rapport aux autres termes de recherche. Par défaut, le facteur de promotion est égal à 1. Ce facteur doit être positif, mais il peut être inférieur à 1 (par exemple 0,20).  

##  <a name="regular-expression-search"></a><a name="bkmk_regex"></a> Recherche d’expression régulière  
 Une recherche d’expression régulière trouve une correspondance en fonction de modèles qui sont valides sous Apache Lucene, comme le décrit la [classe RegExp](https://lucene.apache.org/core/6_6_1/core/org/apache/lucene/util/automaton/RegExp.html). Dans Recherche cognitive Azure, une expression régulière est placée entre des barres obliques `/`.

 Par exemple, pour rechercher des documents contenant « motel » ou « hotel », spécifiez `/[mh]otel/`. Les recherches d’expression régulière se font par comparaison avec des mots individuels.

Certains outils et certaines langues imposent des exigences supplémentaires en matière de caractères d’échappement. Pour JSON, les chaînes qui incluent une barre oblique sont placées dans une séquence d’échappement avec une barre oblique inverse : « microsoft.com/azure/ » devient `search=/.*microsoft.com\/azure\/.*/` où `search=/.* <string-placeholder>.*/` configure l’expression régulière et `microsoft.com\/azure\/` est la chaîne avec une barre oblique d’échappement.

##  <a name="wildcard-search"></a><a name="bkmk_wildcard"></a> Recherche par caractères génériques

Vous pouvez utiliser la syntaxe généralement reconnue pour effectuer des recherches avec plusieurs caractères génériques (`*`) ou un caractère générique unique (`?`). Par exemple, une expression de requête `search=alpha*` retourne « alphanumérique » ou « alphabétique ». Notez que l’Analyseur de requêtes Lucene prend en charge l’utilisation de ces symboles avec un terme unique, et non une expression.

La syntaxe Lucene complète prend en charge la correspondance de préfixe, d’infixe et de suffixe. Toutefois, si tout ce dont vous avez besoin est la correspondance de préfixe, vous pouvez utiliser la syntaxe simple (la correspondance de préfixe est prise en charge dans les deux).

La correspondance de suffixe, où `*` ou `?` précède la chaîne (comme dans `search=/.*numeric./`), ou la correspondance d’infixe nécessite une syntaxe Lucene complète ainsi que les barres obliques `/` délimiteurs d’expressions régulières. Vous ne pouvez pas utiliser un signe * ou ? comme premier caractère d’un terme, ou dans un terme, sans `/`. 

> [!NOTE]  
> En règle générale, les critères spéciaux sont lents ; vous préférerez donc peut-être explorer d’autres méthodes, telles que la tokenisation Edge n-Gram qui crée des jetons pour les séquences de caractères d’un terme. L’index sera plus grand, mais les requêtes pourront s’exécuter plus rapidement, en fonction de la construction du modèle et de la longueur des chaînes que vous indexez.
>

### <a name="impact-of-an-analyzer-on-wildcard-queries"></a>Impact d’un analyseur sur les requêtes génériques

Pendant l’analyse des requêtes, les requêtes qui sont formulées sous forme de préfixe, de suffixe, de caractère générique ou d’expression régulière sont transmises telles quelles à l’arborescence de requête, en ignorant [l’analyse lexicale](search-lucene-query-architecture.md#stage-2-lexical-analysis). Les correspondances ne seront trouvées que si l’index contient les chaînes au format spécifié par votre requête. Dans la plupart des cas, vous aurez besoin d’un analyseur pendant l’indexation qui préserve l’intégrité de la chaîne pour que le terme partiel et les critères spéciaux soient respectés. Pour plus d’informations, consultez [Recherche de termes partiels dans les requêtes Recherche cognitive Azure](search-query-partial-matching.md).

Supposons que vous souhaitez que la requête de recherche « terminat* » retourne des résultats qui contiennent des termes tels que « terminate », « termination » et « terminates ».

Si vous deviez utiliser l’analyseur en.lucene (Lucene anglais), il appliquerait une recherche de radical agressive pour chaque terme. Par exemple, « terminate », « termination », « terminates » seraient tous réduits au jeton « termi » dans votre index. D’un côté, comme les termes dans les requêtes utilisant des caractères génériques ou une recherche approximative ne sont pas analysés du tout, la requête « terminat* » ne donnerait aucun résultat.

D’un autre côté, les analyseurs Microsoft (dans ce cas, l’analyseur en.microsoft) sont un peu plus avancés et utilisent la lemmatisation et non la recherche de radical. Cela signifie que tous les jetons générés doivent être des mots anglais valides. Par exemple, « terminate », « terminates » et « termination » seront généralement conservés dans leur forme entière dans l’index, ce qui constitue un choix préférable pour les scénarios qui dépendent beaucoup des caractères génériques et de la recherche approximative.

## <a name="scoring-wildcard-and-regex-queries"></a> Scoring des requêtes avec des caractères génériques et des expressions régulières

La Recherche cognitive Azure utilise un scoring basé sur la fréquence ([TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf)) pour les requêtes de texte. Cependant, pour les requêtes avec des caractères génériques et des expressions régulières où l’étendue des termes est potentiellement vaste, le facteur de fréquence est ignoré pour empêcher que le classement soit faussé par les correspondances avec des termes plus rares. Toutes les correspondances sont traitées de façon égale pour les recherches avec des caractères génériques et avec des expressions régulières.

## <a name="special-characters"></a>Caractères spéciaux

Dans certains cas, vous rechercherez un caractère spécial, tel que l’émoji « ❤ » ou le signe « € ». Le cas échéant, assurez-vous que l’analyseur que vous utilisez n’ignore pas ces caractères. L’analyseur standard ignore de nombreux caractères spéciaux, en les excluant de votre index.

Les analyseur qui segmentent du texte en unités lexicales incluent l’analyseur « whitespace », qui prend en compte toutes les séquences de caractères séparées par des espaces blancs comme des jetons (donc, la chaîne « ❤ » est considérée comme un jeton). En outre, un analyseur de langage comme l’analyseur Microsoft English (« en.microsoft ») considère la chaîne « € » comme un jeton. Vous pouvez [tester un analyseur](/rest/api/searchservice/test-analyzer) pour voir quels jetons il génère pour une requête donnée.

Lorsque vous utilisez des caractères Unicode, assurez-vous que les symboles sont correctement placés dans une séquence d’échappement dans l’URL de la requête (par exemple, pour « ❤ » utilisez la séquence d’échappement `%E2%9D%A4+`). Postman effectue cette traduction automatiquement.  

## <a name="precedence-grouping"></a>Précédence (regroupement)

Vous pouvez utiliser des parenthèses pour créer des sous-requêtes, en incluant des opérateurs au sein de l’instruction entre parenthèses. Par exemple, `motel+(wifi|luxury)` recherche les documents contenant le terme « motel », et « wifi » ou « luxury » (ou les deux).

Le regroupement de champs est similaire, mais il délimite le regroupement à un seul champ. Par exemple, `hotelAmenities:(gym+(wifi|pool))` recherche « gym » et « wifi », ou « gym » et « pool », dans le champ « hotelAmenities ».  

## <a name="query-size-limits"></a> Limites de taille des requêtes

Il existe une limite à la taille des requêtes que vous pouvez envoyer à la Recherche cognitive Azure. Plus précisément, vous pouvez avoir au maximum 1 024 clauses (des expressions séparées par AND, OR, etc.). Il existe également une limite d’environ 32 Ko pour la taille d’un terme individuel dans une requête. Si votre application génère des requêtes de recherche par programmation, nous vous recommandons de la concevoir de façon à ce qu’elle ne génère pas des requêtes d’une taille illimitée.  

## <a name="see-also"></a>Voir aussi

+ [Exemples de requêtes pour une recherche simple](search-query-simple-examples.md)
+ [Exemples de requêtes pour une recherche Lucene complète](search-query-lucene-examples.md)
+ [Recherche dans des documents](/rest/api/searchservice/Search-Documents)
+ [Syntaxe des expressions OData pour les filtres et le tri](query-odata-filter-orderby-syntax.md)   
+ [Syntaxe de requête simple dans la Recherche cognitive Azure](query-simple-syntax.md)
