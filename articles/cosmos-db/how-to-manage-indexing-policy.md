---
title: Gérer les stratégies d’indexation dans Azure Cosmos DB
description: Découvrir comment gérer les stratégies d’indexation, inclure ou exclure une propriété de l’indexation et définir l’indexation à l’aide de différents Kits de développement logiciel (SDK) Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 11/02/2020
ms.author: tisande
ms.custom: devx-track-python, devx-track-js, devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: cd51210a64223fab5d2d48a91bd3d0a6521a9627
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93341312"
---
# <a name="manage-indexing-policies-in-azure-cosmos-db"></a>Gérer les stratégies d’indexation dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Dans Azure Cosmos DB, les données sont indexées suivant les [stratégies d’indexation](index-policy.md) définies pour chaque conteneur. La stratégie d’indexation par défaut pour les conteneurs nouvellement créés applique des index de plage pour les chaînes ou les nombres. Vous pouvez remplacer cette stratégie par votre propre stratégie d’indexation personnalisée.

> [!NOTE]
> La méthode de mise à jour des stratégies d’indexation décrite dans cet article s’applique uniquement à l’API SQL (principale) de la base de données SQL Azure Cosmos. Apprenez-en davantage sur l’indexation dans [API d’Azure Cosmos DB pour MongoDB](mongodb-indexing.md) et [Indexation secondaire dans l’API Cassandra Azure Cosmos DB](cassandra-secondary-index.md).

## <a name="indexing-policy-examples"></a>Exemples de stratégie d’indexation

Voici quelques exemples de stratégies d’indexation au [format JSON](index-policy.md#include-exclude-paths), comme elles apparaissent sur le Portail Azure. Les mêmes paramètres peuvent être définis par le biais de l’interface Azure CLI ou de n’importe quel SDK.

### <a name="opt-out-policy-to-selectively-exclude-some-property-paths"></a>Stratégie de refus pour exclure de façon sélective certains chemins de propriété

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/path/to/single/excluded/property/?"
            },
            {
                "path": "/path/to/root/of/multiple/excluded/properties/*"
            }
        ]
    }
```

Cette stratégie d’indexation est équivalente à celle ci-dessous, qui permet de définir manuellement les valeurs par défaut de ```kind```, ```dataType``` et ```precision```. Il n’est plus nécessaire de définir explicitement ces propriétés. Vous devez les omettre entièrement dans votre stratégie d’indexation (comme indiqué dans l’exemple ci-dessus).

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/*",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number",
                        "precision": -1
                    },
                    {
                        "kind": "Range",
                        "dataType": "String",
                        "precision": -1
                    }
                ]
            }
        ],
        "excludedPaths": [
            {
                "path": "/path/to/single/excluded/property/?"
            },
            {
                "path": "/path/to/root/of/multiple/excluded/properties/*"
            }
        ]
    }
```

### <a name="opt-in-policy-to-selectively-include-some-property-paths"></a>Stratégie d’acceptation pour inclure de façon sélective certains chemins de propriété

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/path/to/included/property/?"
            },
            {
                "path": "/path/to/root/of/multiple/included/properties/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/*"
            }
        ]
    }
```

Cette stratégie d’indexation est équivalente à celle ci-dessous, qui permet de définir manuellement les valeurs par défaut de ```kind```, ```dataType``` et ```precision```. Il n’est plus nécessaire de définir explicitement ces propriétés. Vous devez les omettre entièrement dans votre stratégie d’indexation (comme indiqué dans l’exemple ci-dessus).

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/path/to/included/property/?",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number"
                    },
                    {
                        "kind": "Range",
                        "dataType": "String"
                    }
                ]
            },
            {
                "path": "/path/to/root/of/multiple/included/properties/*",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number"
                    },
                    {
                        "kind": "Range",
                        "dataType": "String"
                    }
                ]
            }
        ],
        "excludedPaths": [
            {
                "path": "/*"
            }
        ]
    }
```

> [!NOTE]
> Il est généralement recommandé d’utiliser une stratégie d’indexation de **refus** pour permettre à Azure Cosmos DB d’indexer de manière proactive toute nouvelle propriété qui peut être ajoutée à votre modèle de données.

### <a name="using-a-spatial-index-on-a-specific-property-path-only"></a>Utilisation d’un index spatial uniquement sur un chemin de propriété spécifique

```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/*"
        }
    ],
    "excludedPaths": [
        {
            "path": "/_etag/?"
        }
    ],
    "spatialIndexes": [
        {
            "path": "/path/to/geojson/property/?",
            "types": [
                "Point",
                "Polygon",
                "MultiPolygon",
                "LineString"
            ]
        }
    ]
}
```

## <a name="composite-indexing-policy-examples"></a>Exemples de stratégies d’indexation composite

En plus d’inclure ou d’exclure des chemins pour les propriétés individuelles, vous pouvez également spécifier un index composite. Si vous souhaitez effectuer une requête qui a une `ORDER BY` clause pour plusieurs propriétés, un [index composite](index-policy.md#composite-indexes) sur ces propriétés est requis. De plus, les index composites présentent un avantage en matière de performances pour les requêtes qui ont des filtres multiples ou un filtre et une clause ORDER BY à la fois.

> [!NOTE]
> Un chemin composite a un `/?` implicite, car seule la valeur scalaire sur ce chemin est indexée. Le caractère générique `/*` n’est pas pris en charge dans les chemins composites. Vous ne devez pas spécifier `/?` ou `/*` dans un chemin composite.

### <a name="composite-index-defined-for-name-asc-age-desc"></a>Index composite défini pour (name asc, age desc) :

```json
    {  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/name",
                    "order":"ascending"
                },
                {  
                    "path":"/age",
                    "order":"descending"
                }
            ]
        ]
    }
```

L’index composite ci-dessus sur le nom et l’âge est requis pour les requêtes 1 et 2 :

Requête 1 :

```sql
    SELECT *
    FROM c
    ORDER BY c.name ASC, c.age DESC
```

Requête 2 :

```sql
    SELECT *
    FROM c
    ORDER BY c.name DESC, c.age ASC
```

Cet index composite tire parti des requêtes 3 et 4 et optimise les filtres :

Requête 3 :

```sql
SELECT *
FROM c
WHERE c.name = "Tim"
ORDER BY c.name DESC, c.age ASC
```

Requête 4 :

```sql
SELECT *
FROM c
WHERE c.name = "Tim" AND c.age > 18
```

### <a name="composite-index-defined-for-name-asc-age-asc-and-name-asc-age-desc"></a>Index composite défini pour (name ASC, age ASC) et (name ASC, age DESC) :

Vous pouvez définir plusieurs index composites différents au sein de la même stratégie d’indexation.

```json
    {  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/name",
                    "order":"ascending"
                },
                {  
                    "path":"/age",
                    "order":"ascending"
                }
            ],
            [  
                {  
                    "path":"/name",
                    "order":"ascending"
                },
                {  
                    "path":"/age",
                    "order":"descending"
                }
            ]
        ]
    }
```

### <a name="composite-index-defined-for-name-asc-age-asc"></a>Index composite défini pour (name ASC, age ASC) :

Il est facultatif de spécifier l’ordre. S’il n’est pas spécifié, l’ordre est croissant.

```json
{  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/name",
                },
                {  
                    "path":"/age",
                }
            ]
        ]
}
```

### <a name="excluding-all-property-paths-but-keeping-indexing-active"></a>Exclusion de tous les chemins de propriété mais maintien de l’indexation active

Cette stratégie peut être utilisée dans les situations où la [fonctionnalité de durée de vie (TTL)](time-to-live.md) est active, mais où aucun index supplémentaire n’est nécessaire (pour utiliser Azure Cosmos DB comme magasin de clés-valeurs pur).

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [],
        "excludedPaths": [{
            "path": "/*"
        }]
    }
```

### <a name="no-indexing"></a>Aucune indexation

Cette stratégie désactive l’indexation. Si `indexingMode` a la `none`valeur, vous ne pouvez pas définir de TTL sur le conteneur.

```json
    {
        "indexingMode": "none"
    }
```

## <a name="updating-indexing-policy"></a>Mie à jour d’une stratégie d’indexation

Dans Azure Cosmos DB, vous pouvez mettre à jour la stratégie d’indexation à l’aide de l’une des méthodes ci-dessous :

- à partir du portail Azure
- à l’aide de l’interface Azure CLI
- à l’aide de PowerShell
- à l’aide de l’un des SDK

Une [mise à jour de la stratégie d’indexation](index-policy.md#modifying-the-indexing-policy) déclenche une transformation d’index. La progression de cette transformation peut également être suivie à partir des SDK.

> [!NOTE]
> Lors de la mise à jour de la stratégie d’indexation, les écritures dans Azure Cosmos DB sont ininterrompues. En savoir plus sur [l’indexation des transformations](index-policy.md#modifying-the-indexing-policy)

## <a name="use-the-azure-portal"></a>Utilisation du portail Azure

Les conteneurs Azure Cosmos stockent leur stratégie d’indexation sous la forme d’un document JSON que le portail Azure vous permet de modifier directement.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Créez un compte Azure Cosmos ou sélectionnez un compte existant.

1. Ouvrez le volet **Explorateur de données** , puis sélectionnez le conteneur avec lequel vous voulez travailler.

1. Cliquez sur **Mise à l’échelle et paramètres**.

1. Modifiez le document JSON de stratégie d’indexation (voir les exemples [ci-dessous](#indexing-policy-examples)).

1. Cliquez sur **Enregistrer** quand vous avez terminé.

:::image type="content" source="./media/how-to-manage-indexing-policy/indexing-policy-portal.png" alt-text="Gérer l’indexation avec le Portail Azure":::

## <a name="use-the-azure-cli"></a>Utilisation de l’interface de ligne de commande Microsoft Azure

Pour créer un conteneur avec une stratégie d’indexation personnalisée, voir [Créer un conteneur avec une stratégie d’index personnalisée à l’aide d’Azure CLI](manage-with-cli.md#create-a-container-with-a-custom-index-policy)

## <a name="use-powershell"></a>Utiliser PowerShell

Pour créer un conteneur avec une stratégie d’indexation personnalisée, consultez [Créer un conteneur avec une stratégie d’index personnalisée à l’aide de PowerShell](manage-with-powershell.md#create-container-custom-index).

## <a name="use-the-net-sdk"></a><a id="dotnet-sdk"></a>Utiliser le kit de développement logiciel (SDK) .NET

# <a name="net-sdk-v2"></a>[Kit de développement logiciel (SDK) .NET V2](#tab/dotnetv2)

L’objet `DocumentCollection` du [SDK .NET v2](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/) expose une propriété `IndexingPolicy` qui vous permet de changer la valeur `IndexingMode`, et d’ajouter ou de supprimer des `IncludedPaths` et `ExcludedPaths`.

```csharp
// Retrieve the container's details
ResourceResponse<DocumentCollection> containerResponse = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("database", "container"));
// Set the indexing mode to consistent
containerResponse.Resource.IndexingPolicy.IndexingMode = IndexingMode.Consistent;
// Add an included path
containerResponse.Resource.IndexingPolicy.IncludedPaths.Add(new IncludedPath { Path = "/*" });
// Add an excluded path
containerResponse.Resource.IndexingPolicy.ExcludedPaths.Add(new ExcludedPath { Path = "/name/*" });
// Add a spatial index
containerResponse.Resource.IndexingPolicy.SpatialIndexes.Add(new SpatialSpec() { Path = "/locations/*", SpatialTypes = new Collection<SpatialType>() { SpatialType.Point } } );
// Add a composite index
containerResponse.Resource.IndexingPolicy.CompositeIndexes.Add(new Collection<CompositePath> {new CompositePath() { Path = "/name", Order = CompositePathSortOrder.Ascending }, new CompositePath() { Path = "/age", Order = CompositePathSortOrder.Descending }});
// Update container with changes
await client.ReplaceDocumentCollectionAsync(containerResponse.Resource);
```

Pour suivre la progression de la transformation de l’index, vous devez passer un objet `RequestOptions` qui définit la propriété `PopulateQuotaInfo` sur `true`.

```csharp
// retrieve the container's details
ResourceResponse<DocumentCollection> container = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("database", "container"), new RequestOptions { PopulateQuotaInfo = true });
// retrieve the index transformation progress from the result
long indexTransformationProgress = container.IndexTransformationProgress;
```

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET V3](#tab/dotnetv3)

L’objet `ContainerProperties` du [SDK .NET v3](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/) (voir [ce Guide de démarrage rapide](create-sql-api-dotnet.md) concernant son utilisation) expose une propriété `IndexingPolicy` qui vous permet de changer la valeur `IndexingMode`, et d’ajouter ou de supprimer des `IncludedPaths` et `ExcludedPaths`.

```csharp
// Retrieve the container's details
ContainerResponse containerResponse = await client.GetContainer("database", "container").ReadContainerAsync();
// Set the indexing mode to consistent
containerResponse.Resource.IndexingPolicy.IndexingMode = IndexingMode.Consistent;
// Add an included path
containerResponse.Resource.IndexingPolicy.IncludedPaths.Add(new IncludedPath { Path = "/*" });
// Add an excluded path
containerResponse.Resource.IndexingPolicy.ExcludedPaths.Add(new ExcludedPath { Path = "/name/*" });
// Add a spatial index
SpatialPath spatialPath = new SpatialPath
{
    Path = "/locations/*"
};
spatialPath.SpatialTypes.Add(SpatialType.Point);
containerResponse.Resource.IndexingPolicy.SpatialIndexes.Add(spatialPath);
// Add a composite index
containerResponse.Resource.IndexingPolicy.CompositeIndexes.Add(new Collection<CompositePath> { new CompositePath() { Path = "/name", Order = CompositePathSortOrder.Ascending }, new CompositePath() { Path = "/age", Order = CompositePathSortOrder.Descending } });
// Update container with changes
await client.GetContainer("database", "container").ReplaceContainerAsync(containerResponse.Resource);
```

Pour suivre la progression de la transformation de l’index, passez un objet `RequestOptions` qui définit la propriété `PopulateQuotaInfo` sur `true`, puis récupérez la valeur à partir de l’en-tête de réponse `x-ms-documentdb-collection-index-transformation-progress`.

```csharp
// retrieve the container's details
ContainerResponse containerResponse = await client.GetContainer("database", "container").ReadContainerAsync(new ContainerRequestOptions { PopulateQuotaInfo = true });
// retrieve the index transformation progress from the result
long indexTransformationProgress = long.Parse(containerResponse.Headers["x-ms-documentdb-collection-index-transformation-progress"]);
```

Lors de la définition d’une stratégie d’indexation personnalisée pendant la création d’un conteneur, l’API Fluent du SDK V3 vous permet d’écrire cette définition de façon concise et efficace :

```csharp
await client.GetDatabase("database").DefineContainer(name: "container", partitionKeyPath: "/myPartitionKey")
    .WithIndexingPolicy()
        .WithIncludedPaths()
            .Path("/*")
        .Attach()
        .WithExcludedPaths()
            .Path("/name/*")
        .Attach()
        .WithSpatialIndex()
            .Path("/locations/*", SpatialType.Point)
        .Attach()
        .WithCompositeIndex()
            .Path("/name", CompositePathSortOrder.Ascending)
            .Path("/age", CompositePathSortOrder.Descending)
        .Attach()
    .Attach()
    .CreateIfNotExistsAsync();
```
---

## <a name="use-the-java-sdk"></a>Utiliser le SDK Java

L’objet `DocumentCollection` du [SDK Java](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb) (voir [ce Guide de démarrage rapide](create-sql-api-java.md) concernant son utilisation) expose les méthodes `getIndexingPolicy()` et `setIndexingPolicy()`. L’objet `IndexingPolicy` qu’elles manipulent vous permet de changer le mode d’indexation, et d’ajouter ou de supprimer des chemins inclus ou exclus.

```java
// Retrieve the container's details
Observable<ResourceResponse<DocumentCollection>> containerResponse = client.readCollection(String.format("/dbs/%s/colls/%s", "database", "container"), null);
containerResponse.subscribe(result -> {
DocumentCollection container = result.getResource();
IndexingPolicy indexingPolicy = container.getIndexingPolicy();

// Set the indexing mode to consistent
indexingPolicy.setIndexingMode(IndexingMode.Consistent);

// Add an included path

Collection<IncludedPath> includedPaths = new ArrayList<>();
IncludedPath includedPath = new IncludedPath();
includedPath.setPath("/*");
includedPaths.add(includedPath);
indexingPolicy.setIncludedPaths(includedPaths);

// Add an excluded path

Collection<ExcludedPath> excludedPaths = new ArrayList<>();
ExcludedPath excludedPath = new ExcludedPath();
excludedPath.setPath("/name/*");
excludedPaths.add(excludedPath);
indexingPolicy.setExcludedPaths(excludedPaths);

// Add a spatial index

Collection<SpatialSpec> spatialIndexes = new ArrayList<SpatialSpec>();
Collection<SpatialType> collectionOfSpatialTypes = new ArrayList<SpatialType>();

SpatialSpec spec = new SpatialSpec();
spec.setPath("/locations/*");
collectionOfSpatialTypes.add(SpatialType.Point);
spec.setSpatialTypes(collectionOfSpatialTypes);
spatialIndexes.add(spec);

indexingPolicy.setSpatialIndexes(spatialIndexes);

// Add a composite index

Collection<ArrayList<CompositePath>> compositeIndexes = new ArrayList<>();
ArrayList<CompositePath> compositePaths = new ArrayList<>();

CompositePath nameCompositePath = new CompositePath();
nameCompositePath.setPath("/name");
nameCompositePath.setOrder(CompositePathSortOrder.Ascending);

CompositePath ageCompositePath = new CompositePath();
ageCompositePath.setPath("/age");
ageCompositePath.setOrder(CompositePathSortOrder.Descending);

compositePaths.add(ageCompositePath);
compositePaths.add(nameCompositePath);

compositeIndexes.add(compositePaths);
indexingPolicy.setCompositeIndexes(compositeIndexes);

// Update the container with changes

 client.replaceCollection(container, null);
});
```

Pour suivre la progression de la transformation de l’index sur un conteneur, passez un objet `RequestOptions` qui demande les informations de quota à remplir, puis récupérez la valeur à partir de l’en-tête de réponse `x-ms-documentdb-collection-index-transformation-progress`.

```java
// set the RequestOptions object
RequestOptions requestOptions = new RequestOptions();
requestOptions.setPopulateQuotaInfo(true);
// retrieve the container's details
Observable<ResourceResponse<DocumentCollection>> containerResponse = client.readCollection(String.format("/dbs/%s/colls/%s", "database", "container"), requestOptions);
containerResponse.subscribe(result -> {
    // retrieve the index transformation progress from the response headers
    String indexTransformationProgress = result.getResponseHeaders().get("x-ms-documentdb-collection-index-transformation-progress");
});
```

## <a name="use-the-nodejs-sdk"></a>Utiliser le SDK Node.js

L’interface `ContainerDefinition` du [SDK Node.js](https://www.npmjs.com/package/@azure/cosmos) (voir [ce Guide de démarrage rapide](create-sql-api-nodejs.md) concernant son utilisation) expose une propriété `indexingPolicy` qui vous permet de changer la valeur `indexingMode`, et d’ajouter ou de supprimer des `includedPaths` et `excludedPaths`.

Récupérer les détails du conteneur

```javascript
const containerResponse = await client.database('database').container('container').read();
```

Définir le mode d’indexation sur cohérent

```javascript
containerResponse.body.indexingPolicy.indexingMode = "consistent";
```

Ajouter un chemin inclus comprenant un index spatial

```javascript
containerResponse.body.indexingPolicy.includedPaths.push({
    includedPaths: [
      {
        path: "/age/*",
        indexes: [
          {
            kind: cosmos.DocumentBase.IndexKind.Range,
            dataType: cosmos.DocumentBase.DataType.String
          },
          {
            kind: cosmos.DocumentBase.IndexKind.Range,
            dataType: cosmos.DocumentBase.DataType.Number
          }
        ]
      },
      {
        path: "/locations/*",
        indexes: [
          {
            kind: cosmos.DocumentBase.IndexKind.Spatial,
            dataType: cosmos.DocumentBase.DataType.Point
          }
        ]
      }
    ]
  });
```

Ajouter un chemin exclu

```javascript
containerResponse.body.indexingPolicy.excludedPaths.push({ path: '/name/*' });
```

Mettre à jour le conteneur avec les modifications

```javascript
const replaceResponse = await client.database('database').container('container').replace(containerResponse.body);
```

Pour suivre la progression de la transformation de l’index sur un conteneur, passez un objet `RequestOptions` qui définit la propriété `populateQuotaInfo` sur `true`, puis récupérez la valeur à partir de l’en-tête de réponse `x-ms-documentdb-collection-index-transformation-progress`.

```javascript
// retrieve the container's details
const containerResponse = await client.database('database').container('container').read({
    populateQuotaInfo: true
});
// retrieve the index transformation progress from the response headers
const indexTransformationProgress = replaceResponse.headers['x-ms-documentdb-collection-index-transformation-progress'];
```

## <a name="use-the-python-sdk"></a>Utiliser le SDK Python

# <a name="python-sdk-v3"></a>[Python SDK V3](#tab/pythonv3)

Lorsque vous utilisez le [kit de développement logiciel (SDK) Python V3](https://pypi.org/project/azure-cosmos/) (voir [ce Guide de démarrage rapide](create-sql-api-python.md) concernant son utilisation), la configuration du conteneur est gérée comme un dictionnaire. À partir de ce dictionnaire, il est possible d’accéder à la stratégie d’indexation et à tous ses attributs.

Récupérer les détails du conteneur

```python
containerPath = 'dbs/database/colls/collection'
container = client.ReadContainer(containerPath)
```

Définir le mode d’indexation sur cohérent

```python
container['indexingPolicy']['indexingMode'] = 'consistent'
```

Définir une stratégie d’indexation avec un chemin inclus et un index spatial

```python
container["indexingPolicy"] = {

    "indexingMode":"consistent",
    "spatialIndexes":[
                {"path":"/location/*","types":["Point"]}
             ],
    "includedPaths":[{"path":"/age/*","indexes":[]}],
    "excludedPaths":[{"path":"/*"}]
}
```

Définir une stratégie d’indexation avec un chemin exclu

```python
container["indexingPolicy"] = {
    "indexingMode":"consistent",
    "includedPaths":[{"path":"/*","indexes":[]}],
    "excludedPaths":[{"path":"/name/*"}]
}
```

Ajouter un Index composite

```python
container['indexingPolicy']['compositeIndexes'] = [
                [
                    {
                        "path": "/name",
                        "order": "ascending"
                    },
                    {
                        "path": "/age",
                        "order": "descending"
                    }
                ]
                ]
```

Mettre à jour le conteneur avec les modifications

```python
response = client.ReplaceContainer(containerPath, container)
```

# <a name="python-sdk-v4"></a>[Kit SDK Python v4](#tab/pythonv4)

Lorsque vous utilisez le [kit de développement logiciel (SDK) Python V4](https://pypi.org/project/azure-cosmos/), la configuration du conteneur est gérée comme un dictionnaire. À partir de ce dictionnaire, il est possible d’accéder à la stratégie d’indexation et à tous ses attributs.

Récupérer les détails du conteneur

```python
database_client = cosmos_client.get_database_client('database')
container_client = database_client.get_container_client('container')
container = container_client.read()
```

Définir le mode d’indexation sur cohérent

```python
indexingPolicy = {
    'indexingMode': 'consistent'
}
```

Définir une stratégie d’indexation avec un chemin inclus et un index spatial

```python
indexingPolicy = {
    "indexingMode":"consistent",
    "spatialIndexes":[
        {"path":"/location/*","types":["Point"]}
    ],
    "includedPaths":[{"path":"/age/*","indexes":[]}],
    "excludedPaths":[{"path":"/*"}]
}
```

Définir une stratégie d’indexation avec un chemin exclu

```python
indexingPolicy = {
    "indexingMode":"consistent",
    "includedPaths":[{"path":"/*","indexes":[]}],
    "excludedPaths":[{"path":"/name/*"}]
}
```

Ajouter un Index composite

```python
indexingPolicy['compositeIndexes'] = [
    [
        {
            "path": "/name",
            "order": "ascending"
        },
        {
            "path": "/age",
            "order": "descending"
        }
    ]
]
```

Mettre à jour le conteneur avec les modifications

```python
response = database_client.replace_container(container_client, container['partitionKey'], indexingPolicy)
```

Récupérer la progression de la transformation d’index à partir des en-têtes de réponse
```python
container_client.read(populate_quota_info = True,
                      response_hook = lambda h,p: print(h['x-ms-documentdb-collection-index-transformation-progress']))
```

---

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’indexation, consultez les articles suivants :

- [Vue d’ensemble de l’indexation](index-overview.md)
- [Stratégie d’indexation](index-policy.md)
