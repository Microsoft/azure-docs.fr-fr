---
title: Tutoriel C# sur l’indexation de plusieurs sources de données Azure
titleSuffix: Azure Cognitive Search
description: Découvrez comment importer des données à partir de plusieurs sources de données dans un seul index Recherche cognitive Azure à l’aide d’indexeurs. Ce didacticiel et cet exemple de code utilisent le langage C#.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 10/13/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: c9d9c43ae1be755ccb30fc377692257a81332ea8
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94593720"
---
# <a name="tutorial-index-from-multiple-data-sources-using-the-net-sdk"></a>Tutoriel : Indexer à partir de plusieurs sources de données à l’aide du SDK .NET

Le service Recherche cognitive Azure peut importer, analyser et indexer des données provenant de plusieurs sources de données dans un même index de recherche fusionné. 

Ce tutoriel utilise C# et la bibliothèque de client [Azure.Search.Documents](/dotnet/api/overview/azure/search) du SDK Azure pour .Net afin d’indexer des exemples de données d’hôtels provenant d’une base de données Azure Cosmos DB et de les fusionner avec les détails concernant les chambres d’hôtel extraits de documents du Stockage Blob Azure. Le résultat est un index de recherche d’hôtel combiné contenant des documents d’hôtel, avec des chambres comme types de données complexes.

Dans ce tutoriel, vous allez effectuer les tâches suivantes :

> [!div class="checklist"]
> * Charger des exemples de données et créer des sources de données
> * Identifier la clé de document
> * Définir et créer l’index
> * Indexer les données d’hôtels issues d’Azure Cosmos DB
> * Fusionner les données des chambres d’hôtel provenant du stockage d’objets blob

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="overview"></a>Vue d’ensemble

Ce tutoriel utilise la nouvelle bibliothèque de client, [Azure.Search.Documents](/dotnet/api/overview/azure/search), version 11.x, pour créer et exécuter plusieurs indexeurs. Dans ce tutoriel, vous allez configurer deux sources de données Azure afin de pouvoir configurer un indexeur qui extrait des informations de ces deux sources pour remplir un index de recherche unique. Les deux jeux de données doivent avoir une valeur commune pour prendre en charge la fusion. Dans cet exemple, ce champ est un ID. Tant qu’il existe un champ en commun pour prendre en charge le mappage, un indexeur peut fusionner des données provenant de ressources disparates : des données structurées à partir d’Azure SQL, des données non structurées à partir du stockage Blob ou toute combinaison de [sources de données prises en charge](search-indexer-overview.md#supported-data-sources) sur Azure.

Une version terminée du code de ce tutoriel se trouve dans le projet suivant :

* [multiple-data-sources/v11 (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/multiple-data-sources/v11)

Ce tutoriel a été mis à jour pour utiliser le package Azure.Search.Documents (version 11). Pour obtenir une version antérieure du SDK .NET, consultez [Exemple de code Microsoft.Azure.Search (version 10)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/multiple-data-sources/v10) sur GitHub.

## <a name="prerequisites"></a>Prérequis

+ [Azure Cosmos DB](../cosmos-db/create-cosmosdb-resources-portal.md)
+ [Stockage Azure](../storage/common/storage-account-create.md)
+ [Visual Studio](https://visualstudio.microsoft.com/)
+ [Package NuGet de la Recherche cognitive Azure (version 11.x)](https://www.nuget.org/packages/Azure.Search.Documents/)
+ [Créer](search-create-service-portal.md) ou [rechercher un service de recherche existant](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) 

> [!Note]
> Vous pouvez utiliser le service gratuit pour ce tutoriel. Avec un service de recherche gratuit, vous êtes limité à trois index, trois indexeurs et trois sources de données. Ce didacticiel crée une occurrence de chaque élément. Avant de commencer, veillez à disposer de l’espace suffisant sur votre service pour accepter les nouvelles ressources.

## <a name="1---create-services"></a>1 - Créer les services

Ce tutoriel utilise Recherche cognitive Azure pour l’indexation et les requêtes, Azure Cosmos DB pour un des jeux de données et Stockage Blob Azure pour le deuxième jeu de données. 

Si possible, créez tous les services dans la même région et le même groupe de ressources pour des raisons de proximité et de facilité de gestion. Dans la pratique, vos services peuvent se trouver dans n’importe quelle région.

Cet exemple utilise deux petits ensembles de données qui décrivent sept hôtels fictifs. Un ensemble décrit les hôtels eux-mêmes et est chargé dans une base de données Azure Cosmos DB. L’autre ensemble contient les détails des chambres d’hôtel et est fourni sous la forme de sept fichiers JSON distincts à charger dans Stockage Blob Azure.

### <a name="start-with-cosmos-db"></a>Démarrer avec Cosmos DB

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis accédez à la page Vue d’ensemble de votre compte Azure Cosmos DB.

1. Sélectionnez **Explorateur de données**, puis sélectionnez **Nouvelle base de données**.

   :::image type="content" source="media/tutorial-multiple-data-sources/cosmos-newdb.png" alt-text="Créer une base de données" border="false":::

1. Entrez le nom **hotel-rooms-db**. Acceptez les valeurs par défaut pour les autres paramètres.

   :::image type="content" source="media/tutorial-multiple-data-sources/cosmos-dbname.png" alt-text="Configurer la base de données" border="false":::

1. Créez un conteneur. Utilisez la base de données existante que vous venez de créer. Entrez **hotels** comme nom de conteneur et utilisez **/HotelId** comme clé de partition.

   :::image type="content" source="media/tutorial-multiple-data-sources/cosmos-add-container.png" alt-text="Ajouter un conteneur" border="false":::

1. Sélectionnez **Éléments** sous **Hôtels**, puis cliquez sur **Charger un élément** dans la barre de commandes. Accédez au fichier **cosmosdb/HotelsDataSubset_CosmosDb.json** dans le dossier du projet, puis sélectionnez-le.

   :::image type="content" source="media/tutorial-multiple-data-sources/cosmos-upload.png" alt-text="Charger dans la collection Azure Cosmos DB" border="false":::

1. Utilisez le bouton Actualiser pour actualiser l’affichage des éléments dans la collection hotels. Sept nouveaux documents doivent être listés.

1. Copier une chaîne de connexion de la page **Clés** dans le Bloc-notes. Vous en aurez besoin pour **appsettings.json** lors d’une étape ultérieure. Si vous n’avez pas utilisé le nom de base de données suggéré « hotel-rooms-db », copiez également le nom de la base de données.

### <a name="azure-blob-storage"></a>Stockage Blob Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com), accédez à votre compte de stockage Azure, cliquez sur **Objets blob**, puis sur **+ Conteneur**.

1. [Créez un conteneur d’objets blob](../storage/blobs/storage-quickstart-blobs-portal.md) nommé **hotel-rooms** pour stocker les exemples de fichiers JSON des chambres d’hôtel. Vous pouvez définir le niveau d’accès public sur l’une de ses valeurs valides.

   Créer un :::image type="content" source="media/tutorial-multiple-data-sources/blob-add-container.png" alt-text="conteneur d’objets blob" border="false":::

1. Une fois le conteneur créé, ouvrez-le et sélectionnez **Charger** dans la barre de commandes. Accédez au dossier contenant les exemples de fichiers. Sélectionnez-les tous, puis cliquez sur **Charger**.

   :::image type="content" source="media/tutorial-multiple-data-sources/blob-upload.png" alt-text="Charger des fichiers" border="false":::

1. Copiez le nom du compte de stockage et une chaîne de connexion de la page **Clés d’accès** dans le Bloc-notes. Vous aurez besoin de ces deux valeurs pour **appsettings.json** lors d’une étape ultérieure.

### <a name="azure-cognitive-search"></a>Recherche cognitive Azure

Le troisième composant est Recherche cognitive Azure, que vous pouvez [créer dans le portail](search-create-service-portal.md). 

### <a name="copy-an-admin-api-key-and-url-for-azure-cognitive-search"></a>Copier une clé API d’administration et une URL pour Recherche cognitive Azure

Pour vous authentifier auprès de votre service de recherche, vous devez disposer de l’URL du service et d’une clé d’accès.

1. [Connectez-vous au portail Azure](https://portal.azure.com/), puis dans la page **Vue d’ensemble** du service de recherche, récupérez l’URL. Voici un exemple de point de terminaison : `https://mydemo.search.windows.net`.

1. Dans **Paramètres** > **Clés**, obtenez une clé d’administration pour avoir des droits d’accès complets sur le service. Il existe deux clés d’administration interchangeables, fournies pour assurer la continuité de l’activité au cas où vous deviez en remplacer une. Vous pouvez utiliser la clé primaire ou secondaire sur les demandes d’ajout, de modification et de suppression d’objets.

   :::image type="content" source="media/search-get-started-javascript/service-name-and-keys.png" alt-text="Obtenir le nom du service, les clés d’administration et les clés de requête" border="false":::

L’utilisation d’une clé valide permet d’établir, en fonction de chaque demande, une relation de confiance entre l’application qui envoie la demande et le service qui en assure le traitement.

## <a name="2---set-up-your-environment"></a>2 - Configurer votre environnement

1. Démarrez Visual Studio. Ensuite, dans le menu **Outils**, sélectionnez **Gestionnaire de package NuGet**, puis **Gérer les packages NuGet pour la solution...** . 

1. Sous l’onglet **Parcourir**, recherchez et installez **Azure.Search.Documents** (version 11.0 ou ultérieure). Vous devrez valider une série de boîtes de dialogue supplémentaires pour terminer l’installation.

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-nuget-azure.png" alt-text="Utilisation de NuGet pour ajouter des bibliothèques Azure" border="false":::

1. Recherchez les packages NuGet **Microsoft.Extensions.Configuration** et **Microsoft.Extensions.Configuration.Json**, puis installez-les également.

1. Ouvrez le fichier solution **/v11/AzureSearchMultipleDataSources.sln**.

1. Dans l’Explorateur de solutions, modifiez le fichier **appsettings.json** en y ajoutant les informations de connexion.  

    ```json
    {
      "SearchServiceUri": "<YourSearchServiceURL>",
      "SearchServiceAdminApiKey": "<YourSearchServiceAdminApiKey>",
      "BlobStorageAccountName": "<YourBlobStorageAccountName>",
      "BlobStorageConnectionString": "<YourBlobStorageConnectionString>",
      "CosmosDBConnectionString": "<YourCosmosDBConnectionString>",
      "CosmosDBDatabaseName": "hotel-rooms-db"
    }
    ```

Les deux premières entrées sont l’URL et les clés d’administration d’un service de recherche. Utilisez le point de terminaison complet ; par exemple : `https://mydemo.search.windows.net`.

Les entrées suivantes spécifient les noms de compte et les informations de chaîne de connexion pour les sources de données Stockage Blob Azure et Azure Cosmos DB.

## <a name="3---map-key-fields"></a>3 - Mapper les champs clés

La fusion de contenu nécessite que les deux flux de données ciblent les mêmes documents dans l’index de recherche. 

Dans Recherche cognitive Azure, le champ de clé identifie chaque document de façon univoque. Chaque index de recherche doit comporter exactement un champ de clé de type `Edm.String`. Ce champ de clé doit être présent pour chaque document d’une source de données qui est ajouté à l’index. (il constitue en fait le seul champ obligatoire).

Quand vous indexez des données provenant de plusieurs sources de données, vérifiez que chaque ligne ou document entrant contient une clé de document commune pour les fusionner à partir de deux documents sources physiquement distincts dans un nouveau document de recherche dans l’index combiné. 

Il est souvent nécessaire de passer par une planification préalable pour identifier une clé de document significative pour votre index et vous assurer qu’elle existe dans les deux sources de données. Dans cette démonstration, la clé `HotelId` de chaque hôtel dans Cosmos DB est également présente dans les objets blob JSON des chambres dans Stockage Blob.

Les indexeurs de la Recherche cognitive Azure peuvent utiliser des mappages de champs pour renommer, voire remettre en forme, les champs de données pendant le processus d’indexation, afin que les données sources puissent être redirigées vers le champ de l’index approprié. Par exemple, dans Cosmos DB, l’identificateur d’hôtel est appelé **`HotelId`** . Mais dans les fichiers d’objets blob JSON pour les chambres d’hôtel, l’identificateur d’hôtel est nommé **`Id`** . Le programme gère cette situation en mappant le champ **`Id`** des objets blob au champ de clé **`HotelId`** dans l’indexeur.

> [!NOTE]
> Dans la plupart des cas, les clés de document générées automatiquement, comme celles créées par défaut par certains indexeurs, ne constituent pas des clés de document appropriées pour les index combinés. En général, il est préférable d’utiliser une valeur de clé unique explicite qui existe déjà dans vos sources de données ou qui peut y être facilement ajoutée.

## <a name="4---explore-the-code"></a>4 - Explorer le code

Une fois que les données et les paramètres de configuration sont en place, l’exemple de programme figurant dans **/v11/AzureSearchMultipleDataSources.sln** doit être opérationnel.

Cette application console C#/.NET simple effectue les tâches suivantes :

* Elle crée un index basé sur la structure de données de la classe Hotel C# (qui référence également les classes Address et Room).
* Elle crée une source de données et un indexeur qui mappe les données Azure Cosmos DB aux champs d’index. Il s’agit de deux objets dans le service Recherche cognitive Azure.
* Elle exécute l’indexeur pour charger les données des hôtels à partir de Cosmos DB.
* Elle crée une seconde source de données et un indexeur qui mappe les données blob JSON aux champs d’index.
* Elle exécute le second indexeur pour charger des données des chambres à partir du stockage d’objets blob.

 Avant d’exécuter le programme, prenez une minute pour étudier le code et les définitions d’index et d’indexeur de cet exemple. Le code qui convient se trouve dans deux fichiers :

  + **Hotel.cs** contient le schéma qui définit l’index
  + **Program.cs** contient des fonctions qui créent l’index, les sources de données et les indexeurs de la Recherche cognitive Azure, et chargent les résultats combinés dans l’index.

### <a name="create-an-index"></a>Création d'un index

Cet exemple de programme utilise [CreateIndexAsync](/dotnet/api/azure.search.documents.indexes.searchindexclient.createindexasync) pour définir et créer un index Recherche cognitive Azure. Il tire parti de la classe [FieldBuilder](/dotnet/api/azure.search.documents.indexes.fieldbuilder) pour générer une structure d’index à partir d’une classe de modèle de données C#.

Le modèle de données est défini par la classe Hotel, qui contient également des références aux classes Address et Room. La classe FieldBuilder explore plusieurs définitions de classe pour générer une structure de données complexes pour l’index. Des étiquettes de métadonnées sont utilisées pour définir les attributs de chaque champ, par exemple s’il peut faire l’objet d’une recherche ou d’un tri.

Le programme supprime tout index existant portant le même nom avant de créer le nouveau, au cas où vous souhaiteriez exécuter cet exemple plusieurs fois.

Les extraits de code suivants tirés du fichier **Hotel.cs** montrent des champs uniques, suivis d’une référence à une autre classe de modèle de données, Room[], qui est à son tour définie dans le fichier **Room.cs** (non affiché).

```csharp
. . .
[SimpleField(IsFilterable = true, IsKey = true)]
public string HotelId { get; set; }

[SearchableField(IsFilterable = true, IsSortable = true)]
public string HotelName { get; set; }
. . .
public Room[] Rooms { get; set; }
. . .
```

Dans le fichier **Program.cs**, un objet [SearchIndex](/dotnet/api/azure.search.documents.indexes.models.searchindex) est défini avec un nom et une collection de champs générés par la méthode `FieldBuilder.Build`, puis créé comme suit :

```csharp
private static async Task CreateIndexAsync(string indexName, SearchIndexClient indexClient)
{
    // Create a new search index structure that matches the properties of the Hotel class.
    // The Address and Room classes are referenced from the Hotel class. The FieldBuilder
    // will enumerate these to create a complex data structure for the index.
    FieldBuilder builder = new FieldBuilder();
    var definition = new SearchIndex(indexName, builder.Build(typeof(Hotel)));

    await indexClient.CreateIndexAsync(definition);
}
```

### <a name="create-azure-cosmos-db-data-source-and-indexer"></a>Créer l’indexeur et la source de données Azure Cosmos DB

Ensuite, le programme principal inclut une logique destinée à créer la source de données Azure Cosmos DB pour les données d’hôtels.

Tout d’abord, il concatène le nom de la base de données Azure Cosmos DB à la chaîne de connexion. Il définit ensuite un objet [SearchIndexerDataSourceConnection](/dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourceconnection).

```csharp
private static async Task CreateAndRunCosmosDbIndexerAsync(string indexName, SearchIndexerClient indexerClient)
{
    // Append the database name to the connection string
    string cosmosConnectString =
        configuration["CosmosDBConnectionString"]
        + ";Database="
        + configuration["CosmosDBDatabaseName"];

    SearchIndexerDataSourceConnection cosmosDbDataSource = new SearchIndexerDataSourceConnection(
        name: configuration["CosmosDBDatabaseName"],
        type: SearchIndexerDataSourceType.CosmosDb,
        connectionString: cosmosConnectString,
        container: new SearchIndexerDataContainer("hotels"));

    // The Cosmos DB data source does not need to be deleted if it already exists,
    // but the connection string might need to be updated if it has changed.
    await indexerClient.CreateOrUpdateDataSourceConnectionAsync(cosmosDbDataSource);
```

Une fois la source de données créée, le programme configure un indexeur Azure Cosmos DB nommé **hotel-rooms-cosmos-indexer**.

Le programme met à jour tous les indexeurs existants portant le même nom, en remplaçant l’indexeur existant par le contenu du code ci-dessus. Il comprend également des actions de réinitialisation et d’exécution, au cas où vous souhaiteriez exécuter cet exemple plusieurs fois.

L’exemple suivant définit une planification pour l’indexeur, afin qu’il s’exécute une fois par jour. Si vous ne souhaitez pas que l’indexeur soit automatiquement réexécuté à l’avenir, vous pouvez supprimer la propriété de planification de cet appel.

```csharp
SearchIndexer cosmosDbIndexer = new SearchIndexer(
    name: "hotel-rooms-cosmos-indexer",
    dataSourceName: cosmosDbDataSource.Name,
    targetIndexName: indexName)
{
    Schedule = new IndexingSchedule(TimeSpan.FromDays(1))
};

// Indexers keep metadata about how much they have already indexed.
// If we already ran the indexer, it "remembers" and does not run again.
// To avoid this, reset the indexer if it exists.
try
{
    await indexerClient.GetIndexerAsync(cosmosDbIndexer.Name);
    // Reset the indexer if it exists.
    await indexerClient.ResetIndexerAsync(cosmosDbIndexer.Name);
}
catch (RequestFailedException ex) when (ex.Status == 404)
{
    // If the indexer does not exist, 404 will be thrown.
}

await indexerClient.CreateOrUpdateIndexerAsync(cosmosDbIndexer);

Console.WriteLine("Running Cosmos DB indexer...\n");

try
{
    // Run the indexer.
    await indexerClient.RunIndexerAsync(cosmosDbIndexer.Name);
}
catch (RequestFailedException ex) when (ex.Status == 429)
{
    Console.WriteLine("Failed to run indexer: {0}", ex.Message);
}
```

Cet exemple inclut un bloc try-catch simple pour signaler les erreurs susceptibles de se produire pendant l’exécution.

Une fois l’indexeur Azure Cosmos DB exécuté, l’index de recherche contient un ensemble complet d’exemples de documents d’hôtel. Toutefois, le champ Rooms (Chambres) pour chaque hôtel est un tableau vide, car la source de données Azure Cosmos DB ne contenait omet les détails sur les chambres. Ensuite, le programme extrait les données des chambres du stockage d’objets blob et les fusionne.

### <a name="create-blob-storage-data-source-and-indexer"></a>Créer l’indexeur et la source de données du stockage blob

Pour obtenir les détails sur les chambres, le programme configure tout d’abord une source de données de stockage d’objets blob pour référencer un ensemble de fichiers d’objets blob JSON individuels.

```csharp
private static async Task CreateAndRunBlobIndexerAsync(string indexName, SearchIndexerClient indexerClient)
{
    SearchIndexerDataSourceConnection blobDataSource = new SearchIndexerDataSourceConnection(
        name: configuration["BlobStorageAccountName"],
        type: SearchIndexerDataSourceType.AzureBlob,
        connectionString: configuration["BlobStorageConnectionString"],
        container: new SearchIndexerDataContainer("hotel-rooms"));

    // The blob data source does not need to be deleted if it already exists,
    // but the connection string might need to be updated if it has changed.
    await indexerClient.CreateOrUpdateDataSourceConnectionAsync(blobDataSource);
```

Une fois la source de données créée, le programme configure un indexeur d’objets blob nommé **hotel-rooms-blob-indexer**, comme indiqué ci-dessous.

Les objets blob JSON contiennent un champ de clé nommé **`Id`** au lieu de **`HotelId`** . Le code utilise la classe `FieldMapping` pour indiquer à l’indexeur de rediriger la valeur du champ **`Id`** vers la clé de document **`HotelId`** dans l’index.

Les indexeurs de stockage Blob peuvent utiliser [IndexingParameters](/dotnet/api/azure.search.documents.indexes.models.indexingparameters) pour spécifier un mode d’analyse. Vous devez définir différents odes d’analyse selon que les objets blob représentent un document unique ou plusieurs documents au sein du même objet blob. Dans cet exemple, comme chaque objet blob représente un document JSON unique, le code utilise le mode d’analyse `json`. Pour plus d’informations sur les paramètres d’analyse d’indexeur pour les objets blob JSON, consultez [Indexer des objets blob JSON](search-howto-index-json-blobs.md).

Cet exemple définit une planification pour l’indexeur, afin qu’il s’exécute une fois par jour. Si vous ne souhaitez pas que l’indexeur soit automatiquement réexécuté à l’avenir, vous pouvez supprimer la propriété de planification de cet appel.

```csharp
// Map the Id field in the Room documents to the HotelId key field in the index
List<FieldMapping> map = new List<FieldMapping> {
    new FieldMapping("Id")
    {
        TargetFieldName =  "HotelId"
    }
};

IndexingParameters parameters = new IndexingParameters();
parameters.Configuration.Add("parsingMode", "json");

SearchIndexer blobIndexer = new SearchIndexer(
    name: "hotel-rooms-blob-indexer",
    dataSourceName: blobDataSource.Name,
    targetIndexName: indexName)
{
    Parameters = parameters,
    Schedule = new IndexingSchedule(TimeSpan.FromDays(1))
};

blobIndexer.FieldMappings.Add(new FieldMapping("Id") { TargetFieldName = "HotelId" });

// Reset the indexer if it already exists
try
{
    await indexerClient.GetIndexerAsync(blobIndexer.Name);
    await indexerClient.ResetIndexerAsync(blobIndexer.Name);
}
catch (RequestFailedException ex) when (ex.Status == 404) { }

await indexerClient.CreateOrUpdateIndexerAsync(blobIndexer);

try
{
    // Run the indexer.
    await searchService.Indexers.RunAsync(blobIndexer.Name);
}
catch (CloudException e) when (e.Response.StatusCode == (HttpStatusCode)429)
{
    Console.WriteLine("Failed to run indexer: {0}", e.Response.Content);
}
```

Étant donné que l’index a déjà été rempli avec les données d’hôtels à partir de la base de données Azure Cosmos DB, l’indexeur d’objets blob met à jour les documents existants dans l’index et ajoute les détails des chambres.

> [!NOTE]
> Si vous avez les mêmes champs non-clé dans vos deux sources de données et que les données au sein de ces champs ne correspondent pas, l’index contient les valeurs de l’indexeur qui a été exécuté le plus récemment. Dans notre exemple, les deux sources de données contiennent un champ **HotelName**. Si, pour une raison quelconque, les données de ce champ sont différentes pour des documents ayant la même valeur de clé, les données **HotelName** de la source de données qui a été indexée le plus récemment sont la valeur stockée dans l’index.

## <a name="5---search"></a>5 - Recherche

Une fois le programme exécuté, vous pouvez explorer l’index de recherche rempli en utilisant l’[**Explorateur de recherche**](search-explorer.md) dans le portail.

Dans le portail Azure, ouvrez la page **Vue d’ensemble** du service de recherche, puis recherchez l’index **hotel-rooms-sample** dans la liste **Index**.

  :::image type="content" source="media/tutorial-multiple-data-sources/index-list.png" alt-text="Liste des index de Recherche cognitive Azure" border="false":::

Cliquez sur l’index hotel-rooms-sample dans la liste. Vous verrez une interface de l’Explorateur de recherche pour l’index. Entrez une requête portant sur un terme, par exemple « Luxury » (Luxe). Vous devriez voir au moins un document dans les résultats, et ce document doit afficher une liste d’objets de chambre dans son tableau de chambres.

## <a name="reset-and-rerun"></a>Réinitialiser et réexécuter

Dans les premières étapes expérimentales du développement, l’approche la plus pratique pour les itérations de conception consiste à supprimer les objets de Recherche cognitive Azure et à autoriser votre code à les reconstruire. Les noms des ressources sont uniques. La suppression d’un objet vous permet de le recréer en utilisant le même nom.

L’exemple de code recherche les objets existants, et les supprime ou les met à jour pour vous permettre de réexécuter le programme.

Vous pouvez également utiliser le portail pour supprimer les index, les indexeurs et les sources de données.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous travaillez dans votre propre abonnement, il est judicieux à la fin d’un projet de supprimer les ressources dont vous n’avez plus besoin. Les ressources laissées en cours d’exécution peuvent vous coûter de l’argent. Vous pouvez supprimer les ressources une par une, ou choisir de supprimer le groupe de ressources afin de supprimer l’ensemble des ressources.

Vous pouvez rechercher et gérer les ressources dans le portail à l’aide des liens Toutes les ressources ou Groupes de ressources situés dans le volet de navigation de gauche.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous êtes familiarisé avec le concept d’investigation de données provenant de plusieurs sources, examinons de plus près la configuration de l’indexeur, en commençant par Cosmos DB.

> [!div class="nextstepaction"]
> [Configurer un indexeur Azure Cosmos DB](search-howto-index-cosmosdb.md)
