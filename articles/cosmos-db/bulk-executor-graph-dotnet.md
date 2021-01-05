---
title: Utiliser la bibliothèque graphique de l’exécuteur en bloc .NET avec l’API Gremlin Azure Cosmos DB
description: Découvrez comment utiliser la bibliothèque de l’exécuteur en bloc pour importer massivement des données de graphique dans un conteneur d’API Gremlin Azure Cosmos DB.
author: christopheranderson
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 05/28/2019
ms.author: chrande
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: b31cb33e09158de5912132d0fb7bd31a62131181
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93360511"
---
# <a name="using-the-graph-bulk-executor-net-library-to-perform-bulk-operations-in-azure-cosmos-db-gremlin-api"></a>Utilisation de la bibliothèque graphique de l’exécuteur en bloc .NET pour effectuer des opérations en bloc dans l’API Gremlin Azure Cosmos DB
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

Ce tutoriel fournit des instructions sur l’utilisation de la bibliothèque de l’exécuteur en bloc .NET d’Azure Cosmos DB pour importer et mettre à jour des objets graphiques au sein d’un conteneur d’API Gremlin Azure Cosmos DB. Ce processus se sert de la classe Graph dans la [bibliothèque de l’exécuteur en bloc](./bulk-executor-overview.md) pour créer des objets Arête et Sommet par programmation puis en insérer plusieurs par requête réseau. Ce comportement est configurable par le biais de la bibliothèque de l’exécuteur en bloc pour une utilisation optimale des ressources de la base de données et de la mémoire locale.

Par opposition à l’envoi de requêtes Gremlin à une base de données, où les commandes sont évaluées puis exécutées une par une, l’utilisation de la bibliothèque de l’exécuteur en bloc nécessite la création et la validation des objets localement. Une fois les objets créés, la bibliothèque vous permet d’envoyer des objets graphiques séquentiellement pour le service de base de données. À l’aide de cette méthode, les vitesses d’ingestion de données peuvent être multipliées jusqu'à 100 fois, ce qui en fait la méthode idéale pour les migrations de données initiales ou les opérations périodiques de déplacement de données. Pour en savoir plus, rendez-vous sur la page GitHub de l’[exemple d’application Graph de l’exécuteur en bloc Azure Cosmos DB](https://github.com/Azure-Samples/azure-cosmosdb-graph-bulkexecutor-dotnet-getting-started).

## <a name="bulk-operations-with-graph-data"></a>Opérations en bloc avec des données graphiques

La [bibliothèque de l’exécuteur en bloc](/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.graph?preserve-view=true&view=azure-dotnet) contient un espace de noms `Microsoft.Azure.CosmosDB.BulkExecutor.Graph` pour fournir des fonctionnalités pour la création et l’importation d’objets graphiques. 

Le processus suivant décrit l’utilisation de la migration des données pour un conteneur de l’API Gremlin :
1. Récupérez les enregistrements de la source de données.
2. Construisez des objets `GremlinVertex` et `GremlinEdge` à partir des enregistrements obtenus et ajoutez-les à une structure de données `IEnumerable`. Dans cette partie de l’application, la logique de détection et d’ajout des relations doit être implémentée au cas où la source de données n’est pas une base de données de graphiques.
3. Utilisez la [méthode Graph BulkImportAsync](/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.graph.graphbulkexecutor.bulkimportasync?preserve-view=true&view=azure-dotnet) pour insérer les objets graphiques dans la collection.

Ce mécanisme améliorera l’efficacité de la migration de données par rapport à l’utilisation d’un client Gremlin. Cette amélioration est expérimentée car l’insertion de données avec Gremlin demandera l’envoi d’une requête à la fois par l’application, requête qui devra être validée, évaluée, puis exécutée pour créer les données. La bibliothèque de l’exécuteur en bloc gérera la validation dans l’application et enverra plusieurs objets graphiques à la fois pour chaque requête réseau.

### <a name="creating-vertices-and-edges"></a>Création de vertex et de bords

`GraphBulkExecutor` fournit la méthode `BulkImportAsync` nécessitant une liste `IEnumerable` des objets `GremlinVertex` ou `GremlinEdge`, tous deux définis dans l’espace de noms `Microsoft.Azure.CosmosDB.BulkExecutor.Graph.Element`. Dans l’exemple, nous avons séparé les bords et les vertex dans deux tâches d’importation BulkExecutor. Prenons l'exemple suivant :

```csharp

IBulkExecutor graphbulkExecutor = new GraphBulkExecutor(documentClient, targetCollection);

BulkImportResponse vResponse = null;
BulkImportResponse eResponse = null;

try
{
    // Import a list of GremlinVertex objects
    vResponse = await graphbulkExecutor.BulkImportAsync(
            Utils.GenerateVertices(numberOfDocumentsToGenerate),
            enableUpsert: true,
            disableAutomaticIdGeneration: true,
            maxConcurrencyPerPartitionKeyRange: null,
            maxInMemorySortingBatchSize: null,
            cancellationToken: token);

    // Import a list of GremlinEdge objects
    eResponse = await graphbulkExecutor.BulkImportAsync(
            Utils.GenerateEdges(numberOfDocumentsToGenerate),
            enableUpsert: true,
            disableAutomaticIdGeneration: true,
            maxConcurrencyPerPartitionKeyRange: null,
            maxInMemorySortingBatchSize: null,
            cancellationToken: token);
}
catch (DocumentClientException de)
{
    Trace.TraceError("Document client exception: {0}", de);
}
catch (Exception e)
{
    Trace.TraceError("Exception: {0}", e);
}
```

Pour plus d’informations sur les paramètres de la bibliothèque de l’exécuteur en bloc, consultez la [rubrique BulkImportData vers Azure Cosmos DB](bulk-executor-dot-net.md#bulk-import-data-to-an-azure-cosmos-account).

La charge utile doit être instanciée dans les objets `GremlinVertex` et `GremlinEdge`. Voici comment ces objets peuvent être créés :

**Vertex** :
```csharp
// Creating a vertex
GremlinVertex v = new GremlinVertex(
    "vertexId",
    "vertexLabel");

// Adding custom properties to the vertex
v.AddProperty("customProperty", "value");

// Partitioning keys must be specified for all vertices
v.AddProperty("partitioningKey", "value");
```

**Bords** :
```csharp
// Creating an edge
GremlinEdge e = new GremlinEdge(
    "edgeId",
    "edgeLabel",
    "targetVertexId",
    "sourceVertexId",
    "targetVertexLabel",
    "sourceVertexLabel",
    "targetVertexPartitioningKey",
    "sourceVertexPartitioningKey");

// Adding custom properties to the edge
e.AddProperty("customProperty", "value");
```

> [!NOTE]
> L’utilitaire exécuteur en bloc ne vérifie pas automatiquement la présence d’arêtes existantes avant d’ajouter des sommets. Cette opération doit être validée dans l’application avant d’exécuter les tâches BulkImport.

## <a name="sample-application"></a>Exemple d’application

### <a name="prerequisites"></a>Prérequis
* Visual Studio 2019 avec la charge de travail de développement Azure. Vous pouvez commencer gratuitement avec [Visual Studio 2019 Community Edition](https://visualstudio.microsoft.com/downloads/).
* Un abonnement Azure. Vous pouvez créer [un compte Azure gratuit ici](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=cosmos-db). Vous pouvez également créer un compte de base de données Cosmos [en essayant Azure Cosmos DB gratuitement](https://azure.microsoft.com/try/cosmosdb/) sans abonnement Azure.
* Une base de données d’API Gremlin Azure Cosmos DB avec une **collection illimitée**. Ce guide montre comment commencer avec [l’API Gremlin Azure Cosmos DB dans .NET](./create-graph-dotnet.md).
* Git. Pour plus d’informations, consultez la [page des téléchargements Git](https://git-scm.com/downloads).

### <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application
Dans ce tutoriel, nous allons suivre les étapes de mise en route en utilisant l’[exemple d’exécuteur en bloc Graph Azure Cosmos DB](https://github.com/Azure-Samples/azure-cosmosdb-graph-bulkexecutor-dotnet-getting-started) hébergé sur GitHub. Cette application se compose d’une solution .NET qui génère des objets de vertex et de bords de manière aléatoire, puis exécutez des insertions en bloc pour le compte spécifié de base de données de graphiques. Pour obtenir l’application, exécutez la commande `git clone` ci-dessous :

```bash
git clone https://github.com/Azure-Samples/azure-cosmosdb-graph-bulkexecutor-dotnet-getting-started.git
```

Ce référentiel contient l’exemple GraphBulkExecutor avec les fichiers suivants :

Fichier|Description
---|---
`App.config`|Il s’agit de l’emplacement où les paramètres spécifiques à l’application et à la base de données sont spécifiés. Ce fichier doit d’abord être modifié pour vous connecter à la base de données et aux collections de destination.
`Program.cs`| Ce fichier contient la logique sous-jacente à la création de la collection `DocumentClient`, à la gestion des nettoyages et à l’envoi des requêtes de l’exécuteur en bloc.
`Util.cs`| Ce fichier contient une classe d’assistance qui contient la logique derrière la génération de données de test et la vérification de l’existence de la base de données et des collections.

Dans le fichier `App.config`, les éléments suivants sont les valeurs de configuration pouvant être fournies :

Paramètre|Description
---|---
`EndPointUrl`|Il s’agit de **votre point de terminaison .NET SDK** trouvé dans le panneau Vue d’ensemble de votre compte de base de données d’API Gremlin Azure Cosmos DB. Il a le format `https://your-graph-database-account.documents.azure.com:443/`
`AuthorizationKey`|Il s’agit de la clé primaire ou secondaire répertoriée sous votre compte Azure Cosmos DB. Pour en savoir plus sur la [sécurisation de l’accès aux données Azure Cosmos DB](./secure-access-to-data.md#primary-keys)
`DatabaseName`, `CollectionName`|Il s’agit des **noms de la base de données et de la collection cible**. Lorsque `ShouldCleanupOnStart` a la valeur `true`, ces valeurs, ainsi que `CollectionThroughput`, seront utilisées pour les supprimer et créer une nouvelle base de données et une nouvelle collection. De même, si `ShouldCleanupOnFinish` a la valeur `true`, ils seront utilisés pour supprimer la base de données une fois l’ingestion terminée. Notez que la collection cible doit être **une collection illimitée**.
`CollectionThroughput`|Cela permet de créer une nouvelle collection si l’option `ShouldCleanupOnStart` est définie sur `true`.
`ShouldCleanupOnStart`|Cela supprime le compte de base de données et les collections avant l’exécution du programme, puis en créer de nouveaux avec les valeurs `DatabaseName`, `CollectionName` et `CollectionThroughput`.
`ShouldCleanupOnFinish`|Cela supprimera le compte de base de données et les collections avec les valeurs `DatabaseName` et `CollectionName` une fois le programme exécuté.
`NumberOfDocumentsToImport`|Cela détermine le nombre de vertex et de bords de test devant être générés dans l’exemple. Ce nombre s’appliquent aux vertex et aux bords.
`NumberOfBatches`|Cela détermine le nombre de vertex et de bords de test devant être générés dans l’exemple. Ce nombre s’appliquent aux vertex et aux bords.
`CollectionPartitionKey`|Cela sera utilisé pour créer des vertex et des bords de test, où cette propriété sera affectée automatiquement. Il sera également utilisé lorsque vous recréez la base de données et les collections si l’option `ShouldCleanupOnStart` est définie sur `true`.

### <a name="run-the-sample-application"></a>Exécuter l’exemple d’application

1. Ajoutez vos paramètres spécifiques de configuration de base de données dans `App.config`. Ils seront utilisés pour créer une instance DocumentClient. Si la base de données et le conteneur n’ont pas encore été créés, ils le seront automatiquement.
2. Exécutez l'application. Cela permet d’appeler `BulkImportAsync` deux fois, une fois pour importer les es vertex et l’autre fois pour importer les bords. Si un objet génère une erreur lorsqu’il est inséré, il est ajouté à un `.\BadVertices.txt` ou `.\BadEdges.txt`.
3. Évaluez les résultats en interrogeant la base de données de graphiques. Si l’option `ShouldCleanupOnFinish` est définie sur True, la base de données sera automatiquement supprimée.

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les packages NuGet et les notes de publication de la bibliothèque .NET de l’exécuteur en bloc, consultez les [détails relatifs au Kit de développement logiciel (SDK) de l’exécuteur en bloc](sql-api-sdk-bulk-executor-dot-net.md). 
* Découvrez les [conseils relatifs aux performances](./bulk-executor-dot-net.md#performance-tips) pour optimiser davantage l’utilisation de l’exécuteur en bloc.
* Examinez l’[article de référence sur BulkExecutor.Graph](/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.graph?preserve-view=true&view=azure-dotnet) pour plus d’informations sur les classes et les méthodes définies dans cet espace de noms.