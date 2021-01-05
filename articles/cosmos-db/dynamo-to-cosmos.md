---
title: Migrer votre application d’Amazon DynamoDB vers Azure Cosmos DB
description: Découvrez comment migrer votre application .NET d’Amazon DynamoDB vers Azure Cosmos DB
author: manishmsfte
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 04/29/2020
ms.author: mansha
ms.openlocfilehash: 9b4b5fca8017a906fa44b02edcf5f0bdcf6166b3
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93334223"
---
# <a name="migrate-your-application-from-amazon-dynamodb-to-azure-cosmos-db"></a>Migrer votre application d’Amazon DynamoDB vers Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB est une base de données scalable, distribuée à l’échelle mondiale et complètement managée. Elle fournit un accès à faible latence garanti à vos données. Pour en savoir plus sur Azure Cosmos DB, consultez l’article de [présentation](introduction.md). Cet article explique comment migrer votre application .NET à partir de DynamoDB vers Azure Cosmos DB avec des modifications de code minimes.

## <a name="conceptual-differences"></a>Différences conceptuelles

Voici les principales différences conceptuelles entre Azure Cosmos DB et DynamoDB :

|  DynamoDB | Azure Cosmos DB  |
|---|---|
|Non applicable|  Base de données |
|Table de charge de travail      |  Collection |
|  Élément |  Document |
|Attribut|Champ|
|Index secondaire|Index secondaire|
|Clé primaire - clé de partition|Partition Key|
|Clé primaire - clé de tri| Facultatif |
|STREAM|ChangeFeed|
|Écrire l’unité Compute|Unité de requête (flexible, pour être utilisée pour les lectures et les écritures)|
|Lire l’unité Compute    |Unité de requête (flexible, pour être utilisée pour les lectures et les écritures)|
|Tables globales| Non exigé. Vous pouvez sélectionner directement la région lors de l’approvisionnement du compte Azure Cosmos (vous pouvez modifier la région ultérieurement)|

## <a name="structural-differences"></a>Différences structurelles

La structure JSON d’Azure Cosmos DB est plus simple que celle de DynamoDB. L’exemple suivant répertorie les différences

**DynamoDB**  :

L’objet JSON suivant représente le format de données dans DynamoDB

```json
{
TableName: "Music",
KeySchema: [
{ 
  AttributeName: "Artist",
  KeyType: "HASH", //Partition key
},
{ 
  AttributeName: "SongTitle",
  KeyType: "RANGE" //Sort key
}
],
AttributeDefinitions: [
{ 
  AttributeName: "Artist",
  AttributeType: "S"
},
{ 
  AttributeName: "SongTitle",
  AttributeType: "S"
}
],
ProvisionedThroughput: {
  ReadCapacityUnits: 1,
  WriteCapacityUnits: 1
 }
}
 ```

**Azure Cosmos DB**  :

L’objet JSON suivant représente le format de données dans Azure Cosmos DB

```json
{
"Artist": "",
"SongTitle": "",
"AlbumTitle": "",
"Year": 9999,
"Price": 0.0,
"Genre": "",
"Tags": ""
}
```

## <a name="migrate-your-data"></a>Migration de vos données

Vous disposez de différentes options pour migrer vos données vers Azure Cosmos DB. Pour en savoir plus, consultez l’article [Options pour migrer vos données locales ou cloud vers Azure Cosmos DB](cosmosdb-migrationchoices.md).

## <a name="migrate-your-code"></a>Migration de votre code

Cet article est étendu à la migration du code d’une application vers Azure Cosmos DB, qui est l’aspect essentiel de la migration de bases de données. Pour vous aider à réduire la courbe d’apprentissage, les sections suivantes incluent une comparaison de code côte à côte entre Amazon DynamoDB et l’extrait de code équivalent d’Azure Cosmos DB.

Pour télécharger le code source, clonez le référentiel suivant :

```bash
git clone https://github.com/Azure-Samples/DynamoDB-to-CosmosDB
```

### <a name="pre-requisites"></a>Conditions préalables

- .NET Framework 4.7.2
- Visual Studio 2019
- Accéder à un compte d’API SQL Azure Cosmos DB
- Installation locale d’Amazon DynamoDB
- Java 8
- Exécutez la version téléchargeable d’Amazon DynamoDB sur le port 8000 (vous pouvez modifier et configurer le code)

### <a name="set-up-your-code"></a>Configurez votre code

Ajoutez le « package NuGet » suivant dans votre projet :

```bash
Install-Package Microsoft.Azure.Cosmos 
```

### <a name="establish-connection"></a>Établissez la connexion

**DynamoDB**  :

Dans Amazon DynamoDB, le code suivant est utilisé pour se connecter :

```csharp
    AmazonDynamoDBConfig addbConfig = new AmazonDynamoDBConfig() ;
        addbConfig.ServiceURL = "endpoint";
        try { aws_dynamodbclient = new AmazonDynamoDBClient( addbConfig ); }
```

**Azure Cosmos DB**  :

Pour vous connecter à Azure Cosmos DB, mettez à jour votre code pour :

```csharp
client_documentDB = new CosmosClient("your connectionstring from the Azure portal");
```

**Optimiser la connexion dans Azure Cosmos DB**

Avec Azure Cosmos DB, vous pouvez utiliser les options suivantes pour optimiser votre connexion :

* **ConnectionMode**  : utilise le mode de connexion directe pour la connexion aux nœuds de données dans le service Azure Cosmos DB. Utilisez le mode passerelle uniquement pour initialiser et mettre en cache les adresses logiques et les actualiser sur les mises à jour. Pour plus d’informations, consultez l’article [modes de connectivité](sql-sdk-connection-modes.md).

* **ApplicationRegion**  : cette option permet de définir la région répliquée géographiquement qui est utilisée pour interagir avec Azure Cosmos DB. Pour en savoir plus, consultez l’article sur la [distribution mondiale](distribute-data-globally.md).

* **ConsistencyLevel**  : cette option est utilisée pour remplacer le niveau de cohérence par défaut. Pour en savoir plus, consultez l’article [Niveaux de cohérence](consistency-levels.md).

* **BulkExecutionMode**  : cette option permet d’exécuter des opérations en bloc en définissant la propriété *AllowBulkExecution* sur true. Pour en savoir plus, consultez l’article [Importer en bloc](tutorial-sql-api-dotnet-bulk-import.md).

   ```csharp
   client_cosmosDB = new CosmosClient(" Your connection string ",new CosmosClientOptions()
   { 
    ConnectionMode=ConnectionMode.Direct,
    ApplicationRegion=Regions.EastUS2,
    ConsistencyLevel=ConsistencyLevel.Session,
    AllowBulkExecution=true  
   });
   ```

### <a name="provision-the-container"></a>Approvisionner le conteneur

**DynamoDB**  :

Pour stocker les données dans Amazon DynamoDB, vous devez d’abord créer la table. Dans ce processus, vous définissez le schéma, le type de clé et les attributs comme indiqué dans le code suivant :

```csharp
// movies_key_schema
public static List<KeySchemaElement> movies_key_schema
  = new List<KeySchemaElement>
{
  new KeySchemaElement
  {
    AttributeName = partition_key_name,
    KeyType = "HASH"
  },
  new KeySchemaElement
  {
    AttributeName = sort_key_name,
    KeyType = "RANGE"
  }
};

// key names for the Movies table
public const string partition_key_name = "year";
public const string sort_key_name      = "title";
  public const int readUnits=1, writeUnits=1; 

    // movie_items_attributes
    public static List<AttributeDefinition> movie_items_attributes
  = new List<AttributeDefinition>
{
  new AttributeDefinition
  {
    AttributeName = partition_key_name,
    AttributeType = "N"
  },
  new AttributeDefinition
  {
    AttributeName = sort_key_name,
    AttributeType = "S"
  }

CreateTableRequest  request;
CreateTableResponse response;

// Build the 'CreateTableRequest' structure for the new table
request = new CreateTableRequest
{
  TableName             = table_name,
  AttributeDefinitions  = table_attributes,
  KeySchema             = table_key_schema,
  // Provisioned-throughput settings are always required,
  // although the local test version of DynamoDB ignores them.
  ProvisionedThroughput = new ProvisionedThroughput( readUnits, writeUnits );
};
```

**Azure Cosmos DB**  :

Dans Amazon DynamoDB, vous devez approvisionner les unités Compute de lecture et d’écriture. Dans Azure Cosmos DB, spécifiez le débit comme [Unités de requête (RU/s)](request-units.md), qui peuvent être utilisées pour toutes les opérations de manière dynamique. Les données sont organisées sous forme de base de données --> conteneur --> élément. Vous pouvez spécifier le débit au niveau de la base de données ou au niveau de la collection ou les deux.

Pour créer une base de données :

```csharp
await client_cosmosDB.CreateDatabaseIfNotExistsAsync(movies_table_name);
```

Pour créer le conteneur :

```csharp
await cosmosDatabase.CreateContainerIfNotExistsAsync(new ContainerProperties() { PartitionKeyPath = "/" + partitionKey, Id = new_collection_name }, provisionedThroughput);
```

### <a name="load-the-data"></a>Chargement des données

**DynamoDB**  :

Le code suivant montre comment charger les données dans Amazon DynamoDB. MoviesArray se compose d’une liste de documents JSON, vous devez ensuite effectuer une itération et charger le document JSON dans Amazon DynamoDB :

```csharp
int n = moviesArray.Count;
for( int i = 0, j = 99; i < n; i++ )
    {
  try
  {
    string itemJson = moviesArray[i].ToString();
    Document doc = Document.FromJson(itemJson);
    Task putItem = moviesTable.PutItemAsync(doc);
    if( i >= j )
    {
      j++;
      Console.Write( "{0,5:#,##0}, ", j );
      if( j % 1000 == 0 )
        Console.Write( "\n " );
      j += 99;
    }
    await putItem;
```

**Azure Cosmos DB**  :

Dans Azure Cosmos DB, vous pouvez choisir un flux et écrire avec `moviesContainer.CreateItemStreamAsync()`. Toutefois, dans cet exemple, le code JSON sera désérialisé dans le type de *MovieModel* pour illustrer la fonctionnalité de diffusion du type. Le code est multithread et utilisera l’architecture distribuée d’Azure Cosmos DB et accélérera le chargement :

```csharp
List<Task> concurrentTasks = new List<Task>();
for (int i = 0, j = 99; i < n; i++)
{
  try
  {
      MovieModel doc= JsonConvert.DeserializeObject<MovieModel>(moviesArray[i].ToString());
      doc.Id = Guid.NewGuid().ToString();
      concurrentTasks.Add(moviesContainer.CreateItemAsync(doc,new PartitionKey(doc.Year)));
      {
          j++;
          Console.Write("{0,5:#,##0}, ", j);
          if (j % 1000 == 0)
              Console.Write("\n               ");
          j += 99;
      }
      
  }
  catch (Exception ex)
  {
      Console.WriteLine("\n     ERROR: Could not write the movie record #{0:#,##0}, because:\n       {1}",
                          i, ex.Message);
      operationFailed = true;
      break;
  }
}
await Task.WhenAll(concurrentTasks);
```

### <a name="create-a-document"></a>Créer un document

**DynamoDB**  :

L’écriture d’un nouveau document dans Amazon DynamoDB n’est pas du type sécurisé, l’exemple suivant utilise newItem comme type de document :

```csharp
Task<Document> writeNew = moviesTable.PutItemAsync(newItem, token);
await writeNew;
```

**Azure Cosmos DB**  :

Azure Cosmos DB fournit la cohérence des types via le modèle de données. Nous utilisons un modèle de données nommé « MovieModel » :

```csharp
public class MovieModel
{
    [JsonProperty("id")]
    public string Id { get; set; }
    [JsonProperty("title")]
    public string Title{ get; set; }
    [JsonProperty("year")]
    public int Year { get; set; }
    public MovieModel(string title, int year)
    {
        this.Title = title;
        this.Year = year;
    }
    public MovieModel()
    {

    }
    [JsonProperty("info")]
    public   MovieInfo MovieInfo { get; set; }

    internal string PrintInfo()
    {
        if(this.MovieInfo!=null)
        return            string.Format("\nMovie with title:{1}\n Year: {2}, Actors: {3}\n Directors:{4}\n Rating:{5}\n", this.Id, this.Title, this.Year, String.Join(",",this.MovieInfo.Actors), this.MovieInfo, this.MovieInfo.Rating);
        else
            return string.Format("\nMovie with  title:{0}\n Year: {1}\n",  this.Title, this.Year);
    }
}
```

Dans Azure Cosmos DB newItem sera MovieModel :

```csharp
 MovieModel movieModel = new MovieModel()
            {
                Id = Guid.NewGuid().ToString(),
                Title = "The Big New Movie",
                Year = 2018,
                MovieInfo = new MovieInfo() { Plot = "Nothing happens at all.", Rating = 0 }
            };
    var writeNew= moviesContainer.CreateItemAsync(movieModel, new Microsoft.Azure.Cosmos.PartitionKey(movieModel.Year));
    await writeNew;
```

### <a name="read-a-document"></a>Lire un document

**DynamoDB**  :

Pour lire dans Amazon DynamoDB, vous devez définir des primitives :

```csharp
// Create Primitives for the HASH and RANGE portions of the primary key
Primitive hash = new Primitive(year.ToString(), true);
Primitive range = new Primitive(title, false);

  Task<Document> readMovie = moviesTable.GetItemAsync(hash, range, token);
  movie_record = await readMovie;
```

**Azure Cosmos DB**  :

Toutefois, avec Azure Cosmos DB la requête est naturelle (LINQ) :

```csharp
IQueryable<MovieModel> movieQuery = moviesContainer.GetItemLinqQueryable<MovieModel>(true)
                        .Where(f => f.Year == year && f.Title == title);
// The query is executed synchronously here, but can also be executed asynchronously via the IDocumentQuery<T> interface
    foreach (MovieModel movie in movieQuery)
    {
      movie_record_cosmosdb = movie;
    }
```

La collection de documents dans l’exemple ci-dessus est :

- type sécurisé
- fournir une option de requête naturelle.

### <a name="update-an-item"></a>Mettre à jour un élément

**DynamoDB**  : Pour mettre à jour l’élément dans Amazon DynamoDB :

```csharp
updateResponse = await client.UpdateItemAsync( updateRequest );
````

**Azure Cosmos DB**  :

Dans Azure Cosmos DB, la mise à jour est traitée comme une opération Upsert, ce qui signifie qu’il faut insérer le document s’il n’existe pas :

```csharp
await moviesContainer.UpsertItemAsync<MovieModel>(updatedMovieModel);
```

### <a name="delete-a-document"></a>Supprimer un document

**DynamoDB**  :

Pour supprimer un élément dans Amazon DynamoDB, vous devez à nouveau tomber sur les primitives :

```csharp
Primitive hash = new Primitive(year.ToString(), true);
      Primitive range = new Primitive(title, false);
      DeleteItemOperationConfig deleteConfig = new DeleteItemOperationConfig( );
      deleteConfig.ConditionalExpression = condition;
      deleteConfig.ReturnValues = ReturnValues.AllOldAttributes;
      
  Task<Document> delItem = table.DeleteItemAsync( hash, range, deleteConfig );
        deletedItem = await delItem;
```

**Azure Cosmos DB**  :

Dans Azure Cosmos DB, nous pouvons récupérer le document et les supprimer de manière asynchrone :

```csharp
var result= ReadingMovieItem_async_List_CosmosDB("select * from c where c.info.rating>7 AND c.year=2018 AND c.title='The Big New Movie'");
while (result.HasMoreResults)
{
  var resultModel = await result.ReadNextAsync();
  foreach (var movie in resultModel.ToList<MovieModel>())
  {
    await moviesContainer.DeleteItemAsync<MovieModel>(movie.Id, new PartitionKey(movie.Year));
  }
  }
```

### <a name="query-documents"></a>Interroger des documents

**DynamoDB**  :

Dans Amazon DynamoDB, les fonctions d’API sont requises pour interroger les données :

```csharp
QueryOperationConfig config = new QueryOperationConfig( );
  config.Filter = new QueryFilter( );
  config.Filter.AddCondition( "year", QueryOperator.Equal, new DynamoDBEntry[ ] { 1992 } );
  config.Filter.AddCondition( "title", QueryOperator.Between, new DynamoDBEntry[ ] { "B", "Hzz" } );
  config.AttributesToGet = new List<string> { "year", "title", "info" };
  config.Select = SelectValues.SpecificAttributes;
  search = moviesTable.Query( config ); 
```

**Azure Cosmos DB**  :

Dans Azure Cosmos DB, vous pouvez effectuer une projection et un filtre à l’intérieur d’une requête SQL simple :

```csharp
var result = moviesContainer.GetItemQueryIterator<MovieModel>( 
  "select c.Year, c.Title, c.info from c where Year=1998 AND (CONTAINS(Title,'B') OR CONTAINS(Title,'Hzz'))");
```

Pour les opérations de plage, par exemple, « between », vous devez effectuer une analyse dans Amazon DynamoDB :

```csharp
ScanRequest sRequest = new ScanRequest
{
  TableName = "Movies",
  ExpressionAttributeNames = new Dictionary<string, string>
  {
    { "#yr", "year" }
  },
  ExpressionAttributeValues = new Dictionary<string, AttributeValue>
  {
      { ":y_a", new AttributeValue { N = "1960" } },
      { ":y_z", new AttributeValue { N = "1969" } },
  },
  FilterExpression = "#yr between :y_a and :y_z",
  ProjectionExpression = "#yr, title, info.actors[0], info.directors, info.running_time_secs"
};

ClientScanning_async( sRequest ).Wait( );
```

Dans Azure Cosmos DB, vous pouvez utiliser une requête SQL et une instruction sur une seule ligne :

```csharp
var result = moviesContainer.GetItemQueryIterator<MovieModel>( 
  "select c.title, c.info.actors[0], c.info.directors,c.info.running_time_secs from c where BETWEEN year 1960 AND 1969");
```

### <a name="delete-a-container"></a>Supprimer un conteneur

**DynamoDB**  :

Pour supprimer la table dans Amazon DynamoDB, vous pouvez spécifier :

```csharp
client.DeleteTableAsync( tableName );
```

**Azure Cosmos DB**  :

Pour supprimer la collection dans Azure Cosmos DB, vous pouvez spécifier :

```csharp
await moviesContainer.DeleteContainerAsync();
```
Ensuite, supprimez également la base de données si nécessaire :

```csharp
await cosmosDatabase.DeleteAsync();
```

Comme vous pouvez le voir, Azure Cosmos DB prend en charge des requêtes naturelles (SQL), les opérations sont asynchrones et beaucoup plus faciles. Vous pouvez facilement migrer votre code complexe vers Azure Cosmos DB, ce qui devient plus simple après la migration.

### <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [l’optimisation des performances](performance-tips.md).
- En savoir plus sur [l’optimisation des lectures et des écritures](key-value-store-cost.md)
- En savoir plus sur [l’analyse dans Cosmos DB](monitor-cosmos-db.md)

