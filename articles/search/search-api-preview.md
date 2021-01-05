---
title: Liste des fonctionnalités en préversion
titleSuffix: Azure Cognitive Search
description: Les fonctionnalités d’évaluation sont publiées afin que les clients puissent fournir des commentaires sur leur conception et leur utilité. Cet article est une liste complète de toutes les fonctionnalités d’évaluation actuelles.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 983322800198246bcfaecec92fe0b2b74e587756
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91952095"
---
# <a name="preview-features-in-azure-cognitive-search"></a>Fonctionnalités d’évaluation dans Recherche cognitive Azure

Cet article est une liste complète de toutes les fonctionnalités disponibles en préversion publique. Les fonctionnalités d’évaluation sont fournies sans contrat de niveau de service et ne sont pas recommandées pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Les fonctionnalités d’évaluation qui passent en disponibilité générale sont supprimées de cette liste. Si une fonctionnalité n’est pas listée ci-dessous, vous pouvez supposer qu’elle est en disponibilité générale. Pour accéder aux annonces concernant la disponibilité générale, consultez [Mises à jour de service](https://azure.microsoft.com/updates/?product=search) ou [Nouveautés](whats-new.md).

|Fonctionnalité&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  | Category | Description | Disponibilité  |
|---------|------------------|-------------|---------------|
| [**Compétence Azure Machine Learning (AML)** ](cognitive-search-aml-skill.md) | Enrichissement de l’IA| Nouveau type de compétence permettant d'intégrer un point de terminaison d'inférence à partir d'Azure Machine Learning. Pour bien démarrer, suivez [ce tutoriel](cognitive-search-tutorial-aml-custom-skill.md). | Utilisez l'[API REST Recherche 2020-06-30-Preview](/rest/api/searchservice/) ou 2019-05-06-Preview. Également disponible sur le portail, dans la conception de compétences, si les services Recherche cognitive et Azure ML sont déployés sur le même abonnement. |
| [**Paramètre featuresMode**](/rest/api/searchservice/search-documents#featuresmode) | Pertinence (scoring) | Expansion du score de pertinence pour inclure des détails : score de similarité par champ, fréquence de terme par champ, et nombre de jetons uniques correspondants par champ. Vous pouvez consommer ces points de données dans des [solutions de scoring personnalisées](https://github.com/Azure-Samples/search-ranking-tutorial). | Ajoutez ce paramètre de requête à l’aide de [Recherche dans des documents (REST)](/rest/api/searchservice/search-documents) avec api-version=2020-06-30-Preview ou 2019-05-06-Preview. |
| [**Sessions de débogage**](cognitive-search-debug-session.md) | Portail, enrichissement par IA (ensemble de compétences) | Éditeur d’ensemble de compétences dans la session utilisé pour examiner et résoudre les problèmes liés aux ensembles de compétences. Les correctifs appliqués au cours d’une session de débogage peuvent être enregistrés dans un ensemble de compétences dans le service. | Portail uniquement, à l’aide des liens en milieu de page Vue d’ensemble pour ouvrir une session de débogage. |
| [**Suppression réversible native d’objets blob**](search-howto-index-changed-deleted-blobs.md) | Indexeurs, objets blob Azure| L’indexeur Stockage Blob Azure dans Recherche cognitive Azure reconnaît les objets blob qui sont dans un état de suppression réversible, et supprime le document de recherche correspondant durant l’indexation. | Ajoutez ce paramètre de configuration à l’aide de [Créer un indexeur (REST)](/rest/api/searchservice/create-indexer) avec api-version=2020-06-30-Preview ou api-version=2019-05-06-Preview. |
| [**Compétence de recherche d’entité personnalisée**](cognitive-search-skill-custom-entity-lookup.md ) | Enrichissement par IA (ensemble de compétences) | Compétence cognitive qui recherche du texte dans une liste personnalisée de mots et d'expressions définie par l'utilisateur. À l’aide de cette liste, elle étiquète tous les documents contenant des entités correspondantes. La compétence prend également en charge un degré de correspondance approximative qui peut être appliqué pour rechercher des correspondances similaires sans être rigoureusement exactes. | Référencez cette qualification en préversion à l’aide de l’éditeur d’ensemble de compétences dans le portail ou [créez un ensemble de compétences (REST)](/rest/api/searchservice/create-skillset) avec api-version=2020-06-30-Preview or api-version=2019-05-06-Preview. |
| [**Compétence de détection d’informations d’identification personnelle**](cognitive-search-skill-pii-detection.md) | Enrichissement par IA (ensemble de compétences) | Compétence cognitive utilisée lors de l’indexation, qui extrait les informations personnelles d’un texte d’entrée et vous donne la possibilité de les y masquer de différentes façons. | Référencez cette qualification en préversion à l’aide de l’éditeur d’ensemble de compétences dans le portail ou [créez un ensemble de compétences (REST)](/rest/api/searchservice/create-skillset) avec api-version=2020-06-30-Preview or api-version=2019-05-06-Preview. |
| [**Enrichissement incrémentiel**](cognitive-search-incremental-indexing-conceptual.md) | Configuration de l’indexeur| Ajoute la mise en cache à un pipeline d'enrichissement, ce qui vous permet de réutiliser la sortie existante si une modification ciblée, telle que la mise à jour d'un ensemble de compétences ou d'un autre objet, ne change pas le contenu. La mise en cache s’applique uniquement aux documents enrichis générés par un ensemble de compétences.| Ajoutez ce paramètre de configuration à l’aide de [Créer un indexeur (REST)](/rest/api/searchservice/create-indexer) avec api-version=2020-06-30-Preview ou api-version=2019-05-06-Preview. |
| [**Indexeur Cosmos DB : API MongoDB, API Gremlin, API Cassandra**](search-howto-index-cosmosdb.md) | Source de données d’indexeur | Pour Cosmos DB, l’API SQL est en disponibilité générale, mais les API MongoDB, Gremlin et Cassandra sont en préversion. | Pour Gremlin et Cassandra uniquement, [inscrivez-vous au préalable](https://aka.ms/azure-cognitive-search/indexer-preview) afin que la prise en charge puisse être activée pour votre abonnement sur le back-end. Les sources de données MongoDB peuvent être configurées dans le portail. Autrement, la configuration de la source de données pour les trois API est prise en charge à l’aide de [Créer une source de données (REST)](/rest/api/searchservice/create-data-source) avec api-version=2020-06-30-Preview ou api-version=2019-05-06-Preview. |
|  [**Indexeur Azure Data Lake Storage Gen2**](search-howto-index-azure-data-lake-storage.md) | Source de données d’indexeur | Indexe le contenu et les métadonnées de Data Lake Storage Gen2.| L’[inscription](https://aka.ms/azure-cognitive-search/indexer-preview) est nécessaire pour que la prise en charge puisse être activée pour votre abonnement sur le back-end. Accédez à cette source de données à l’aide de [Créer une source de données (REST)](/rest/api/searchservice/create-data-source) avec api-version=2020-06-30-Preview ou api-version=2019-05-06-Preview. |
| [**moreLikeThis**](search-more-like-this.md) | Requête | Recherche les documents correspondant à un document spécifique. Cette fonctionnalité existait dans les préversions antérieures. | Ajoutez ce paramètre de requête dans les appels à [Recherche dans des documents (REST)](/rest/api/searchservice/search-documents) avec api-version=2020-06-30-Preview, 2019-05-06-Preview, 2016-09-01-Preview ou 2017-11-11-Preview. |

## <a name="how-to-call-a-preview-rest-api"></a>Comment appeler une API REST en préversion

Le service Recherche cognitive Azure publie toujours les fonctionnalités expérimentales par le biais de l’API REST, puis par le biais des préversions du SDK .NET.

Les fonctionnalités en préversion sont disponibles pour le test et l’expérimentation en vue de recueillir des commentaires sur la conception et la mise en œuvre de la fonctionnalité. Ainsi, les fonctionnalités en préversion peuvent changer au fil du temps, voire de manière à empêcher la compatibilité descendante. Ce comportement diffère de celui des fonctionnalités dans la version à disposition générale, qui sont stables et peu susceptibles de changer à l’exception d’améliorations et de correctifs peu importants pour la compatibilité descendante. En outre, les fonctionnalités en préversion ne sont pas toujours intégrées à une version à disposition générale.

Alors que certaines fonctionnalités d’évaluation peuvent être disponibles dans le portail et le SDK .NET, l’API REST dispose toujours de fonctionnalités d’évaluation.

+ Pour les opérations de recherche, [ **`2020-06-30-Preview`** ](/rest/api/searchservice/index-preview) correspond à la préversion actuelle.

+ Pour les opérations de gestion, [ **`2019-10-01-Preview`** ](/rest/api/searchmanagement/index-2019-10-01-preview) correspond à la préversion actuelle.

Les préversions plus anciennes sont toujours opérationnelles mais deviennent obsolètes au fil du temps. Si votre code appelle `api-version=2019-05-06-Preview`, `api-version=2016-09-01-Preview` ou `api-version=2017-11-11-Preview`, ces appels sont toujours valides. Toutefois, seule la dernière préversion est actualisée avec des améliorations.

La syntaxe de l’exemple suivant illustre un appel à l’API en préversion.

```HTTP
POST https://[service name].search.windows.net/indexes/hotels-idx/docs/search?api-version=2020-06-30-Preview  
  Content-Type: application/json  
  api-key: [admin key]
```

Le service Recherche cognitive Azure est disponible sous plusieurs versions. Pour plus d’informations, consultez [Versions d’API](search-api-versions.md).

## <a name="next-steps"></a>Étapes suivantes

Consultez la documentation de référence relative à l’API REST en préversion du service de recherche. Si vous rencontrez des problèmes, sollicitez notre aide sur [Stack Overflow](https://stackoverflow.com/) ou [contactez le support](https://azure.microsoft.com/support/community/?product=search).

> [!div class="nextstepaction"]
> [Informations de référence sur l’API REST du service de recherche (préversion)](/rest/api/searchservice/index-preview)