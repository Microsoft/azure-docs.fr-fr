---
title: Planifier l’exécution des indexeurs
titleSuffix: Azure Cognitive Search
description: Planifiez des indexeurs Recherche cognitive Azure pour indexer le contenu à intervalles périodiques ou à des moments donnés.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/06/2020
ms.openlocfilehash: 80c3f9aa02680097276f966ce6aea02acf1e40fb
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358794"
---
# <a name="how-to-schedule-indexers-in-azure-cognitive-search"></a>Comment planifier des indexeurs dans la Recherche cognitive Azure

Un indexeur s’exécute normalement une fois, immédiatement après sa création. Vous pouvez l’exécuter à nouveau à la demande avec le portail, l’API REST ou le kit SDK .NET. Vous pouvez également configurer un indexeur pour qu’il s’exécute à intervalles périodiques.

Certaines situations dans lesquelles la planification de l’indexeur est utilisée :

* Les données sources changent au fil du temps, et vous souhaitez que les indexeurs Recherche cognitive Azure traitent automatiquement les données modifiées.
* L’index est renseigné à partir de plusieurs sources de données et vous souhaitez vous assurer que les indexeurs s’exécutent à des moments différents afin de réduire les conflits.
* Les données sources sont très volumineuses et vous souhaitez répartir le traitement de l’indexeur dans le temps. Pour plus d’informations sur l’indexation de grands volumes de données, consultez [Comment indexer des grands volumes de données dans la Recherche cognitive Azure](search-howto-large-index.md).

Le planificateur est une fonctionnalité intégrée à la Recherche cognitive Azure. Vous ne pouvez pas utiliser un planificateur externe pour contrôler les indexeurs de recherche.

## <a name="define-schedule-properties"></a>Définir les propriétés de la planification

Une planification d’indexeur a deux propriétés :
* **Interval**, qui définit l’intervalle de temps entre les deux exécutions d’indexeur planifiées. Le plus petit intervalle autorisé est de 5 minutes, et le plus grand est de 24 heures.
* **Heure de début (UTC)** , qui indique la première heure à laquelle l’indexeur doit être exécuté.

Vous pouvez spécifier une planification lors de la création de l’indexeur, ou en mettant à jour les propriétés de l’indexeur ultérieurement. Les planifications de l’indexeur peuvent être définies à l’aide du [Portail](#portal), de l’[API REST](#restApi) ou du [SDK .NET](#dotNetSdk).

L’indexeur ne peut s’exécuter qu’une seule fois simultanément. Si un indexeur est déjà en cours d’exécution au moment de sa deuxième exécution planifiée, celle-ci est différée jusqu’à la prochaine date planifiée.

Pour être plus clair, prenons un exemple. Supposons que nous configurons une planification de l’indexeur avec un **intervalle** par heure et une **heure de début** le 1er juin 2019 à 08:00:00 UTC. Voici ce qui peut se produire lorsque l’exécution de l’indexeur prend plus d’une heure :

* La première exécution de l’indexeur commence à ou autour du 1er juin 2019 à 8h00 UTC. Supposons que cette exécution prend 20 minutes (ou en tout cas, moins de 1 heure).
* La deuxième exécution de l’indexeur commence à ou autour du 1er juin 2019 à 9h00 UTC. Supposons que cette exécution dure 70 minutes (plus d’une heure), et qu’elle ne se terminera pas avant 10h10 UTC.
* La troisième exécution est planifiée pour démarrer à 10h00 UTC, mais à ce moment l’exécution précédente est toujours en cours. Cette exécution planifiée est donc ignorée. La prochaine exécution de l’indexeur ne va pas démarrer avant 11h00 UTC.

> [!NOTE]
> Si un indexeur est défini sur une certaine planification, mais échoue à plusieurs reprises sur le même document chaque fois qu’il s’exécute, l’indexeur commence à s’exécuter à un intervalle moins fréquent (jusqu’à un maximum d’au moins une fois toutes les 24 heures) jusqu’à ce qu’il progresse correctement à nouveau.  Si vous pensez avoir résolu le problème qui provoquait le blocage de l’indexeur à un moment donné, vous pouvez effectuer une exécution à la demande de l’indexeur, et en cas de progression, l’indexeur reprend son intervalle de planification défini.

<a name="portal"></a>

## <a name="schedule-in-the-portal"></a>Planification dans le portail

L’Assistant Importation de données dans le portail vous permet de définir la planification pour un indexeur lors de sa création. Le paramètre de planification par défaut est **Par heure**, ce qui signifie que l’indexeur s’exécute une fois qu’il est créé et s’exécute à nouveau toutes les heures par la suite.

Vous pouvez modifier le paramètre de planification sur **Une fois** si vous ne souhaitez pas que l’indexeur s’exécute à nouveau automatiquement, ou sur **Une fois par jour** pour l’exécuter une fois par jour. Définissez-le sur **Personnalisé** si vous souhaitez spécifier un intervalle différent ou une heure de début suivante spécifique.

Lorsque vous définissez la planification sur **Personnalisée**, les champs qui s’affichent vous permettent de spécifier l’**intervalle** et l’**heure de début (UTC)** . Le plus court intervalle de temps autorisé est de 5 minutes, et le plus long est de 1 440 minutes (24 heures).

   ![Définition de la planification de l’indexeur dans l’Assistant Importer des données](media/search-howto-schedule-indexers/schedule-import-data.png "Définition de la planification de l’indexeur dans l’Assistant Importer des données")

Une fois un indexeur créé, vous pouvez modifier les paramètres de planification à l’aide du panneau Modifier de l’indexeur. Les champs de planification sont les mêmes que ceux dans l’Assistant Importation de données.

   ![Définition de la planification dans le panneau de modification de l’indexeur](media/search-howto-schedule-indexers/schedule-edit.png "Définition de la planification dans le panneau de modification de l’indexeur")

<a name="restApi"></a>

## <a name="schedule-using-rest-apis"></a>Planification à l'aide des API REST

Vous pouvez configurer la planification d’un indexeur à l’aide de l’API REST. Pour ce faire, ajoutez la propriété **schedule** lors de la création ou de la mise à jour de l’indexeur. L’exemple ci-dessous montre une requête PUT mettant à jour l’indexeur existant :

```http
    PUT https://myservice.search.windows.net/indexers/myindexer?api-version=2020-06-30
    Content-Type: application/json
    api-key: admin-key

    {
        "dataSourceName" : "myazuresqldatasource",
        "targetIndexName" : "target index name",
        "schedule" : { "interval" : "PT10M", "startTime" : "2015-01-01T00:00:00Z" }
    }
```

Le paramètre **interval** est obligatoire. Il correspond à la durée entre le début de deux exécutions consécutives de l’indexeur. L'intervalle minimal autorisé est de 5 minutes, l'intervalle maximal autorisé est d'une journée. Il doit être formaté en tant que valeur « dayTimeDuration » XSD (un sous-ensemble limité d'une valeur de [durée ISO 8601](https://www.w3.org/TR/xmlschema11-2/#dayTimeDuration) ). Le modèle est le suivant : `P(nD)(T(nH)(nM))`. Exemples : `PT15M` toutes les 15 minutes, `PT2H` toutes les deux heures.

Le paramètre **startTime** facultatif indique quand les exécutions planifiées doivent commencer. S’il est omis, l’heure UTC actuelle est utilisée. Cette heure peut être passée, auquel cas la première exécution est planifiée comme si l’indexeur s’exécutait en continu depuis l’**heure de début** originale.

Vous pouvez également exécuter un indexeur à la demande à tout moment à l’aide de l’appel Exécuter l’indexeur. Pour plus d’informations sur l’exécution des indexeurs et la définition des planifications d’indexeur, consultez [Exécuter l’indexeur](/rest/api/searchservice/run-indexer), [Obtention d’indexeur](/rest/api/searchservice/get-indexer) et [Mise à jour d’un indexeur](/rest/api/searchservice/update-indexer) dans la référence d’API REST.

<a name="dotNetSdk"></a>

## <a name="schedule-using-the-net-sdk"></a>Planification à l’aide du kit de développement logiciel (SDK) REST

Vous pouvez configurer la planification d’un indexeur à l’aide du Kit de développement logiciel (SDK) .NET Recherche cognitive Azure. Pour ce faire, ajoutez la propriété **Schedule** lors de la création ou de la mise à jour d’un indexeur.

L’exemple C# suivant crée un indexeur de base de données Azure SQL à l’aide d’une source de données et d’un index prédéfinis, et définit sa planification de manière à ce qu’il s’exécute une fois par jour à partir de maintenant :

```csharp
var schedule = new IndexingSchedule(TimeSpan.FromDays(1))
{
    StartTime = DateTimeOffset.Now
};

var indexer = new SearchIndexer("hotels-sql-idxr", dataSource.Name, searchIndex.Name)
{
    Description = "Data indexer",
    Schedule = schedule
};

await indexerClient.CreateOrUpdateIndexerAsync(indexer);
```


Si la propriété **Schedule** est omise, l’indexeur s’exécute uniquement une fois immédiatement après sa création.

Le paramètre **StartTime** peut être défini sur une heure passée. Dans ce cas, la première exécution est planifiée comme si l’indexeur s’exécutait en continu depuis la valeur **StartTime** donnée.

La planification est définie à l’aide de la classe [IndexingSchedule](/dotnet/api/azure.search.documents.indexes.models.indexingschedule). Le constructeur **IndexingSchedule** requiert un paramètre **Interval** spécifié à l’aide d’un objet **TimeSpan**. La plus petite valeur d’intervalle autorisée est de 5 minutes, et la plus grande est de 24 heures. Le second paramètre **StartTime**, spécifié comme un objet **DateTimeOffset**, est facultatif.

Le Kit de développement logiciel (SDK) .NET vous permet de contrôler les opérations de l’indexeur à l’aide de [SearchIndexerClient](/dotnet/api/azure.search.documents.indexes.searchindexerclient). 

Vous pouvez exécuter un indexeur à la demande à tout moment en utilisant l’une des méthodes [RunIndexer](/dotnet/api/azure.search.documents.indexes.searchindexerclient.runindexer) ou [RunIndexerAsync](/dotnet/api/azure.search.documents.indexes.searchindexerclient.runindexerasync).

Pour plus d’informations sur la création, la mise à jour et l’exécution des indexeurs, consultez [SearchIndexerClient](/dotnet/api/azure.search.documents.indexes.searchindexerclient).