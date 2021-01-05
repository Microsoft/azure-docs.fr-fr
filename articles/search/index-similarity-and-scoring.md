---
title: Vue d’ensemble de la similarité et du scoring
titleSuffix: Azure Cognitive Search
description: Explique les concepts de similarité et de scoring, et ce qu’un développeur peut faire pour personnaliser le résultat du scoring.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/08/2020
ms.openlocfilehash: 5bd1a9111528146224561995feaecf54612a1c78
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91535659"
---
# <a name="similarity-and-scoring-in-azure-cognitive-search"></a>Similarité et scoring dans Recherche cognitive Azure

Le scoring consiste à calculer un score de recherche pour chaque élément retourné dans des résultats de recherche pour des requêtes de texte intégral. Le score est un indicateur de la pertinence d'un élément dans le contexte de l'opération de recherche en cours. Plus le score est élevé, plus l'élément est pertinent. Dans des résultats de recherche, les éléments sont classés par ordre décroissant de pertinence, sur la base des résultats de recherche calculés pour chacun d'eux. 

Par défaut, les 50 premiers sont retournés dans la réponse, mais vous pouvez utiliser le paramètre **$top** pour retourner un nombre inférieur ou supérieur d’éléments (jusqu’à 1000 par réponse) et le paramètre **$skip** pour obtenir le jeu de résultats suivant.

Le score de recherche est calculé sur la base de propriétés statistiques des données et de la requête. Recherche cognitive Azure trouve les documents qui contiennent des correspondances de recherche (en totalité ou en partie, selon [searchMode](/rest/api/searchservice/search-documents#searchmodeany--all-optional)), en favorisant les documents qui contiennent de nombreuses occurrences du terme recherché. Le score de recherche augmente davantage si le terme est rare dans l’index de données, mais courant au sein du document. La base de cette approche de la pertinence du calcul est appelée *TF-IDF* ou Term Frequency-Inverse Document Frequency (fréquence de terme-fréquence inverse de document).

Des valeurs de score de recherche peuvent être répétées dans un jeu de résultats. Quand plusieurs correspondances ont le même score de recherche, le classement des éléments ayant le même score n’est ni défini ni stable. Réexécutez la requête, et vous constaterez peut-être que des éléments changent de position, en particulier si vous utilisez le service gratuit ou un service facturable avec plusieurs réplicas. Si deux éléments ont un score identique, il est impossible de prédire celui qui apparaîtra en première position.

Si vous souhaitez départager des scores identiques, vous pouvez ajouter une clause **$orderby** afin de trier d’abord par score, puis par un autre champ pouvant être trié (par exemple `$orderby=search.score() desc,Rating desc`). Pour plus d’informations, consultez [$orderby](./search-query-odata-orderby.md).

> [!NOTE]
> Un `@search.score = 1.00` indique un jeu de résultats sans score ou non classé. Le score est uniforme parmi tous les résultats. Des résultats sans score se produisent quand le formulaire de requête est une recherche approximative, des requêtes Regex ou de caractères génériques, ou une expression **$filter**. 

## <a name="scoring-profiles"></a>Profils de score

Vous pouvez personnaliser la façon dont les différents champs sont classés en définissant un *profil de scoring* personnalisé. Les profils de score vous permettent de mieux contrôler le classement d'éléments dans des résultats de recherche. Par exemple, vous pouvez privilégier des éléments en fonction de leur revenu potentiel, promouvoir des éléments plus récents, voire en favoriser d'autres restés trop longtemps en stock. 

Un profil de score fait partie de la définition d'index, composée de champs, fonctions et paramètres pondérés. Pour plus d’informations sur la définition d’un profil de scoring, consultez [Profils de scoring](index-add-scoring-profiles.md).

<a name="scoring-statistics"></a>

## <a name="scoring-statistics-and-sticky-sessions"></a>Statistiques de scoring et sessions rémanentes

À des fins de scalabilité, Recherche cognitive Azure distribue chaque index horizontalement par le biais d’un processus de sharding, ce qui signifie que les [portions d’un index sont physiquement séparées](search-capacity-planning.md#concepts-search-units-replicas-partitions-shards).

Par défaut, le score d’un document est calculé en fonction de propriétés statistiques des données *au sein d’une partition*. Cette approche n’est généralement pas un problème pour un corpus important de données, et elle offre de meilleures performances que le calcul du score à partir des informations de l’ensemble des partitions. Cela dit, avec cette optimisation des performances, deux documents très similaires (ou même des documents identiques) risquent de se retrouver avec des scores de pertinence différents s’ils figurent dans des partitions différentes.

Si vous préférez calculer le score à partir des propriétés statistiques sur l’ensemble des partitions, vous pouvez le faire en ajoutant *scoringStatistics=global* en tant que [paramètre de requête](/rest/api/searchservice/search-documents) (ou en ajoutant *"scoringStatistics": "global"* en tant que paramètre de corps de la [demande de requête](/rest/api/searchservice/search-documents)).

```http
GET https://[service name].search.windows.net/indexes/[index name]/docs?scoringStatistics=global&api-version=2020-06-30&search=[search term]
  Content-Type: application/json
  api-key: [admin or query key]  
```
L’utilisation de scoringStatistics garantit que toutes les partitions dans le même réplica fournissent les mêmes résultats. Cela dit, plusieurs réplicas peuvent être légèrement différents les uns des autres, car ils sont toujours mis à jour avec les dernières modifications apportées à votre index. Dans certains scénarios, vous souhaiterez peut-être que vos utilisateurs obtiennent des résultats plus cohérents pendant une « session de requête ». Dans de tels scénarios, vous pouvez fournir un `sessionId` dans le cadre de vos requêtes. Le `sessionId` est une chaîne unique que vous créez pour faire référence à une session utilisateur unique.

```http
GET https://[service name].search.windows.net/indexes/[index name]/docs?sessionId=[string]&api-version=2020-06-30&search=[search term]
  Content-Type: application/json
  api-key: [admin or query key]  
```
Tant que le même `sessionId` est utilisé, la méthode de la meilleure tentative possible est utilisée pour cibler le même réplica, ce qui améliore la cohérence des résultats présentés à vos utilisateurs. 

> [!NOTE]
> La réutilisation des mêmes valeurs `sessionId` à plusieurs reprises peut interférer avec l’équilibrage de la charge des demandes sur les réplicas et nuire aux performances du service de recherche. La valeur utilisée comme sessionId ne peut pas commencer par un caractère « _ ».

## <a name="similarity-ranking-algorithms"></a>Algorithmes de classement de similarité

Recherche cognitive Azure prend en charge deux algorithmes de classement de similarité différents : un algorithme de *similarité classique* et l’implémentation officielle de l’algorithme *Okapi BM25* (actuellement en préversion). L’algorithme de similarité classique est l’algorithme par défaut, mais à compter du 15 juillet tous les nouveaux services créés utiliseront le nouvel algorithme BM25. Il s’agira du seul algorithme disponible sur les nouveaux services.

Pour le moment, vous pouvez spécifier l’algorithme de classement de similarité que vous souhaitez utiliser. Pour plus d’informations, consultez [Algorithme de classement](index-ranking-similarity.md).

Le segment vidéo suivant permet d’accéder rapidement à une explication des algorithmes de classement utilisés dans Recherche cognitive Azure. Vous pouvez regarder la vidéo complète pour plus d’informations.

> [!VIDEO https://www.youtube.com/embed/Y_X6USgvB1g?version=3&start=322&end=643]

<a name="featuresMode-param"></a>

## <a name="featuresmode-parameter-preview"></a>Paramètre featuresMode (préversion)

Les requêtes [Rechercher des documents](/rest/api/searchservice/preview-api/search-documents) ont un nouveau paramètre [featuresMode](/rest/api/searchservice/preview-api/search-documents#featuresmode) qui peut fournir des détails supplémentaires sur la pertinence au niveau du champ. Tandis que `@searchScore` est calculé pour l’ensemble du document (quelle est la pertinence de ce document dans le contexte de cette requête), par le biais de featuresMode, vous pouvez obtenir des informations sur des champs individuels, tels qu’ils sont exprimés dans une structure `@search.features`. La structure contient tous les champs utilisés dans la requête (soit des champs spécifiques par le biais de **searchFields** dans une requête, soit tous les champs attribués comme **interrogeables** dans un index). Pour chaque champ, vous pouvez obtenir les valeurs suivantes :

+ Nombre de jetons uniques trouvés dans le champ
+ Score de similarité, ou mesure de la similitude entre le contenu du champ et le terme recherché
+ Fréquence du terme, ou nombre de fois où le terme recherché a été trouvé dans le champ

Pour une requête qui cible les champs « Description » et « Titre », une réponse incluant `@search.features` peut se présenter comme suit :

```json
"value": [
 {
    "@search.score": 5.1958685,
    "@search.features": {
        "description": {
            "uniqueTokenMatches": 1.0,
            "similarityScore": 0.29541412,
            "termFrequency" : 2
        },
        "title": {
            "uniqueTokenMatches": 3.0,
            "similarityScore": 1.75451557,
            "termFrequency" : 6
        }
```

Vous pouvez utiliser ces points de données dans [des solutions de scoring personnalisées](https://github.com/Azure-Samples/search-ranking-tutorial) ou utiliser les informations pour déboguer des problèmes de pertinence des recherches.


## <a name="see-also"></a>Voir aussi

 [Profils de calcul de score](index-add-scoring-profiles.md) [Informations de référence sur les API REST](/rest/api/searchservice/) [API de recherche dans des documents](/rest/api/searchservice/search-documents) [Kit de développement logiciel (SDK) .NET Recherche cognitive Azure](/dotnet/api/overview/azure/search)