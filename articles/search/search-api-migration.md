---
title: Mettre à niveau les versions de l’API REST
titleSuffix: Azure Cognitive Search
description: Passez en revue les différences entre les versions des API et découvrez ce que vous devez faire pour migrer le code existant vers la dernière version de l’API REST du service Recherche cognitive Azure.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/09/2020
ms.openlocfilehash: d7734fde529c24e8113ea3b019d343b7223f0122
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91929640"
---
# <a name="upgrade-to-the-latest-rest-api-in-azure-cognitive-search"></a>Effectuer une mise à niveau vers la dernière API REST dans Recherche cognitive Azure

Si vous utilisez une version antérieure de l’[**API REST du service Recherche**](/rest/api/searchservice/), cet article vous aidera à mettre à niveau votre application pour utiliser la toute dernière version de l’API en disponibilité générale, **2020-06-30**.

La version 2020-06-30 inclut une nouvelle fonctionnalité importante ([magasin de connaissances](knowledge-store-concept-intro.md)) et introduit plusieurs modifications de comportement mineures. Par conséquent, cette version est essentiellement à compatibilité descendante. Les modifications du code doivent donc être minimes si vous effectuez une mise à niveau à partir de la version précédente (2019-05-06).

> [!NOTE]
> Un service de recherche prend en charge un éventail de versions de l’API REST, dont des versions plus récentes. Vous pouvez continuer à utiliser ces versions d’API, mais nous vous recommandons de migrer votre code vers la dernière version. Vous pourrez ainsi accéder aux nouvelles fonctionnalités. Au fil du temps, les versions les plus anciennes de l’API REST seront déconseillées et ne seront [plus prises en charge](search-api-versions.md#unsupported-versions).

<a name="UpgradeSteps"></a>

## <a name="how-to-upgrade"></a>Mise à niveau

Lors de la mise à niveau vers une nouvelle version, vous n’aurez probablement pas à modifier beaucoup votre code, en dehors du numéro de version. Les seules situations dans lesquelles vous pouvez avoir à modifier votre code sont les suivantes :

* Lorsque votre code échoue, car des propriétés non reconnues sont renvoyées dans une réponse de l’API. Par défaut, votre application doit ignorer les propriétés qu’elle ne comprend pas.

* Votre code conserve des demandes d’API et tente de les renvoyer à la nouvelle version de l’API. Par exemple, cela peut se produire si votre application conserve les jetons de continuation renvoyés par l’API Recherche (pour plus d’informations, recherchez `@search.nextPageParameters` dans les [références sur l’API Recherche](/rest/api/searchservice/Search-Documents)).

* Votre code fait référence à une version d’API antérieure à la version 2019-05-06 et soumise à un ou plusieurs changements cassants de cette version. La section [Mise à niveau vers la version 2019-05-06](#upgrade-to-2019-05-06) fournit plus de détails. 

Si vous êtes concerné par l’une de ces situations, vous aurez peut-être à modifier votre code en conséquence. Dans le cas contraire, aucune modification n’est nécessaire, sauf si vous souhaitez commencer à utiliser les fonctionnalités ajoutées dans la nouvelle version.

## <a name="upgrade-to-2020-06-30"></a>Mise à niveau vers la version 2020-06-30

La version 2020-06-30 est la nouvelle version de l’API REST en disponibilité générale. Elle comporte un changement cassant et plusieurs différences de comportement. 

Les fonctionnalités désormais en disponibilité générale dans cette version de l’API sont notamment :

* La [base de connaissances](knowledge-store-concept-intro.md), un stockage persistant de contenu enrichi créé via des ensembles de compétences, pour l’analyse et le traitement en aval par le biais d’autres applications. Avec cette fonctionnalité, un pipeline d’enrichissement par IA piloté par un indexeur peut remplir une base de connaissances en plus d’un index de recherche. Si vous avez utilisé la version préliminaire de cette fonctionnalité, elle équivaut à la version en disponibilité générale. La seule modification de code requise est celle de la version de l’API.

### <a name="breaking-change"></a>Modification avec rupture

Le code existant écrit sur des versions d’API antérieures s’arrête sur api-version=2020-06-30 et versions ultérieures si le code contient la fonctionnalité suivante :

* Les littéraux Edm.Date (une date composée de l’année-mois-jour, par exemple `2020-12-12`) dans les expressions de filtre doivent respecter le format Edm.DateTimeOffset : `2020-12-12T00:00:00Z`. Cette modification était nécessaire pour gérer les résultats de requête erronés ou inattendus en raison de différences entre les fuseaux horaires.

### <a name="behavior-changes"></a>Changements de comportement

* L’[algorithme de classement BM25](index-ranking-similarity.md) remplace l’algorithme de classement précédent par une technologie plus récente. Les nouveaux services utiliseront cet algorithme automatiquement. Pour les services existants, vous devez définir des paramètres pour utiliser le nouvel algorithme.

* Les résultats ordonnés pour les valeurs null ont été modifiés dans cette version ; les valeurs null apparaissent en premier si le tri est `asc` et en dernier si le tri est `desc`. Si vous avez écrit du code pour gérer le mode de tri des valeurs null, tenez compte de cette modification.

## <a name="upgrade-to-2019-05-06"></a>Mise à niveau vers la version 2019-05-06

La version 2019-05-06 est la version précédente de l’API REST en disponibilité générale. Les fonctionnalités désormais en disponibilité générale dans cette version de l’API sont notamment :

* [L’autocomplétion](index-add-suggesters.md) est une fonctionnalité prédictive qui complète une entrée de terme partiellement saisie.
* Les [types complexes](search-howto-complex-data-types.md) assurent la prise en charge native des données d’objet structurées dans un index de recherche.
* Les [modes d’analyse JsonLines](search-howto-index-json-blobs.md), inclus dans l’indexation des objets Blob Azure, créent un document de recherche par entité JSON, séparé par un saut de ligne.
* L’[enrichissement par IA](cognitive-search-concept-intro.md) assure une indexation qui tire parti des moteurs d’enrichissement IA de Cognitive Services.

### <a name="breaking-changes"></a>Changements cassants

Le code existant écrit sur des versions d’API antérieures s’arrête sur api-version=2019-05-06 et version ultérieure si le code contient la fonctionnalité suivante :

#### <a name="indexer-for-azure-cosmos-db---datasource-is-now-type-cosmosdb"></a>Indexeur pour Azure Cosmos DB : la source de données est désormais "type": "cosmosdb".

Si vous utilisez un [indexeur Cosmos DB](search-howto-index-cosmosdb.md ), vous devez remplacer `"type": "documentdb"` par `"type": "cosmosdb"`.

#### <a name="indexer-execution-result-errors-no-longer-have-status"></a>Les erreurs de résultat d’exécution de l’indexeur n’ont plus d’état.

La structure d’erreur pour l’exécution de l’indexeur comportait précédemment un élément `status`. Cet élément a été supprimé, car il ne fournissait pas d’informations utiles.

#### <a name="indexer-data-source-api-no-longer-returns-connection-strings"></a>L’API de source de données de l’indexeur ne renvoie plus de chaînes de connexion.

À partir des versions d’API 2019-05-06 et 2019-05-06-Preview, l’API de source de données ne renvoie plus de chaînes de connexion dans la réponse d’une opération REST, quelle qu’elle soit. Dans les versions d’API précédentes, pour les sources de données créées à l’aide de POST, la Recherche cognitive Azure retournait **201**, suivi de la réponse OData, qui contenait la chaîne de connexion en texte brut.

#### <a name="named-entity-recognition-cognitive-skill-is-now-discontinued"></a>La compétence cognitive Reconnaissance d’entité nommée n’est plus proposée.

Si vous avez appelé la compétence [Reconnaissance d’entité nommée](cognitive-search-skill-named-entity-recognition.md) dans votre code, l’appel échoue. La fonctionnalité de remplacement est [Reconnaissance d’entité](cognitive-search-skill-entity-recognition.md). Vous devriez pouvoir remplacer la référence de compétence sans modification supplémentaire. La signature de l’API est la même pour les deux versions. 

### <a name="upgrading-complex-types"></a>Mise à niveau des types complexes

La version d’API 2019-05-06 contient désormais une prise en charge formelle des types complexes. Si votre code implémentait des suggestions précédentes pour l’équivalence de type complexe dans 2017-11-11-Preview ou 2016-09-01-Preview, il existe de nouvelles limites et des limites modifiées à partir de la version 2019-05-06 que vous devez connaître :

+ Les limites relatives à la profondeur des sous-champs et au nombre de collections complexes par index ont été réduites. Si vous avez créé des index qui dépassent ces limites à l’aide de versions d’API en préversion, toute tentative de mise à jour ou de recréation des index à l’aide de la version d’API 2019-05-06 échouera. Si tel est votre cas, vous devrez reconcevoir votre schéma de sorte qu’il respecte les nouvelles limites, puis reconstruire votre index.

+ Il existe une nouvelle limite à partir de la version d’API 2019-05-06 concernant le nombre d’éléments de collections complexes par document. Si vous avez créé des index avec des documents qui dépassent ces limites à l’aide de versions d’API en préversion, toute tentative de réindexation de ces données à l’aide de la version d’API 2019-05-06 échouera. Si tel est votre cas, vous devrez réduire le nombre d’éléments de collections complexes par document avant de réindexer les données.

Pour plus d’informations, consultez [Limites de service de la Recherche cognitive Azure](search-limits-quotas-capacity.md).

#### <a name="how-to-upgrade-an-old-complex-type-structure"></a>Comment mettre à niveau une ancienne structure de type complexe

Si votre code utilise des types complexes avec l’une des anciennes versions d’API en préversion, vous utilisez peut-être un format de définition d’index qui se présente ainsi :

```json
{
  "name": "hotels",  
  "fields": [
    { "name": "HotelId", "type": "Edm.String", "key": true, "filterable": true },
    { "name": "HotelName", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": true, "facetable": false },
    { "name": "Description", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "en.microsoft" },
    { "name": "Description_fr", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "fr.microsoft" },
    { "name": "Category", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
    { "name": "Tags", "type": "Collection(Edm.String)", "searchable": true, "filterable": true, "sortable": false, "facetable": true, "analyzer": "tagsAnalyzer" },
    { "name": "ParkingIncluded", "type": "Edm.Boolean", "filterable": true, "sortable": true, "facetable": true },
    { "name": "LastRenovationDate", "type": "Edm.DateTimeOffset", "filterable": true, "sortable": true, "facetable": true },
    { "name": "Rating", "type": "Edm.Double", "filterable": true, "sortable": true, "facetable": true },
    { "name": "Address", "type": "Edm.ComplexType" },
    { "name": "Address/StreetAddress", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false, "searchable": true },
    { "name": "Address/City", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
    { "name": "Address/StateProvince", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
    { "name": "Address/PostalCode", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
    { "name": "Address/Country", "type": "Edm.String", "searchable": true, "filterable": true, "sortable": true, "facetable": true },
    { "name": "Location", "type": "Edm.GeographyPoint", "filterable": true, "sortable": true },
    { "name": "Rooms", "type": "Collection(Edm.ComplexType)" }, 
    { "name": "Rooms/Description", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "en.lucene" },
    { "name": "Rooms/Description_fr", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false, "analyzer": "fr.lucene" },
    { "name": "Rooms/Type", "type": "Edm.String", "searchable": true },
    { "name": "Rooms/BaseRate", "type": "Edm.Double", "filterable": true, "facetable": true },
    { "name": "Rooms/BedOptions", "type": "Edm.String", "searchable": true },
    { "name": "Rooms/SleepsCount", "type": "Edm.Int32", "filterable": true, "facetable": true },
    { "name": "Rooms/SmokingAllowed", "type": "Edm.Boolean", "filterable": true, "facetable": true },
    { "name": "Rooms/Tags", "type": "Collection(Edm.String)", "searchable": true, "filterable": true, "facetable": true, "analyzer": "tagsAnalyzer" }
  ]
}  
```

Un format plus récent de type arborescence pour la définition de champs d’index a été introduit dans la version d’API 2017-11-11-Preview. Avec le nouveau format, chaque champ complexe a une collection de champs dont les sous-champs sont définis. Dans la version d’API 2019-05-06, ce nouveau format est utilisé exclusivement. Toute tentative de création ou de mise à jour d’un index à l’aide de l’ancien format échouera. Si vous avez créé des index à l’aide de l’ancien format, vous devez utiliser la version d’API 2017-11-11-Preview pour les mettre à jour vers le nouveau format, de sorte qu’ils puissent être gérés avec la version d’API 2019-05-06.

Vous pouvez mettre à jour des index « plats » vers le nouveau format en procédant comme suit avec la version d’API 2017-11-11-Preview :

1. Effectuez une requête GET pour récupérer votre index. S’il est déjà au nouveau format, vous avez terminé.

2. Traduisez l’index à partir du format « plat » vers le nouveau format. Vous devrez écrire du code à cette fin. En effet, aucun exemple de code n’est disponible au moment de la rédaction de ce document.

3. Effectuez une requête PUT pour mettre à jour l’index vers le nouveau format. Veillez à ne modifier aucune autre information de l’index, telle que la possibilité de filtrer ou d’effectuer des recherches dans les champs. En effet, cela n’est pas autorisé par l’API de mise à jour d’index.

> [!NOTE]
> Il n’est pas possible de gérer les index créés avec l’ancien format « plat » à partir du Portail Azure. Mettez à niveau vos index à partir de la représentation « plate » vers la représentation de type « arborescence » dès que possible.

## <a name="next-steps"></a>Étapes suivantes

Consultez la documentation de référence relative à l’API REST du service Recherche. Si vous rencontrez des problèmes, sollicitez notre aide sur [Stack Overflow](https://stackoverflow.com/) ou [contactez le support](https://azure.microsoft.com/support/community/?product=search).

> [!div class="nextstepaction"]
> [Référence de l’API REST du service Recherche](/rest/api/searchservice/)
