---
title: 'Tutoriel : Tutoriel de distribution mondiale Azure Cosmos DB pour l’API SQL'
description: 'Tutoriel : Découvrez comment configurer la diffusion mondiale Azure Cosmos DB à l’aide de l’API SQL avec .NET, Java, Python et divers autres kits SDK'
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: tutorial
ms.date: 11/05/2019
ms.reviewer: sngun
ms.custom: devx-track-python, devx-track-js, devx-track-csharp
ms.openlocfilehash: aacb8d4ffb98d553b17aa52e4c4b11a4837dc1c6
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93337862"
---
# <a name="tutorial-set-up-azure-cosmos-db-global-distribution-using-the-sql-api"></a>Tutoriel : Configuration de la distribution mondiale Azure Cosmos DB à l’aide de l’API SQL
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article montre comment utiliser le portail Azure pour configurer la diffusion mondiale d’Azure Cosmos DB, puis établir une connexion à l’aide de l’API SQL.

Cet article décrit les tâches suivantes : 

> [!div class="checklist"]
> * Configurer la diffusion mondiale à l’aide du portail Azure
> * Configurer la distribution mondiale à l’aide des [API SQL](./introduction.md)

<a id="portal"></a>
[!INCLUDE [cosmos-db-tutorial-global-distribution-portal](../../includes/cosmos-db-tutorial-global-distribution-portal.md)]

## <a name="connecting-to-a-preferred-region-using-the-sql-api"></a><a id="preferred-locations"></a> Se connecter à une région de prédilection avec l’API SQL

Pour tirer parti de la [distribution mondiale](distribute-data-globally.md), les applications clientes peuvent spécifier la liste ordonnée de préférences de régions à utiliser pour effectuer des opérations sur les documents. Selon la configuration du compte Azure Cosmos DB, la disponibilité régionale actuelle et la liste de préférences spécifiée, le Kit de développement logiciel (SDK) SQL choisit le point de terminaison optimal pour les opérations de lecture et d’écriture.

Cette liste de préférences est spécifiée lors de l’initialisation d’une connexion à l’aide des SDK SQL. Les SDK acceptent un paramètre facultatif `PreferredLocations` qui est une liste ordonnée des régions Azure.

Le SDK envoie automatiquement toutes les écritures vers la région d’écriture en cours. Toutes les lectures sont envoyées vers la première région disponible dans la liste des emplacements préférés. Si la demande échoue, le client passe à la région suivante dans la liste.

Le SDK tente des opérations de lecture uniquement à partir des régions spécifiées dans les emplacements préférés. Ainsi, par exemple, si le compte Azure Cosmos est disponible dans quatre régions, mais que le client spécifie uniquement deux régions de lecture (sans écriture) dans `PreferredLocations`, aucune lecture n’est prise en charge en dehors de la région de lecture si elle n’est pas spécifiée dans `PreferredLocations`. Si les régions de lecture spécifiées dans la liste `PreferredLocations` ne sont pas disponibles, les lectures sont prises en charge en dehors de la région d’écriture.

L’application peut vérifier le point de terminaison d’écriture et le point de terminaison de lecture actuels choisis par le SDK en vérifiant deux propriétés, `WriteEndpoint` et `ReadEndpoint`, disponibles dans le SDK version 1.8 et ultérieure. Si la propriété `PreferredLocations` n’est pas définie, toutes les demandes seront traitées par la zone d’écriture en cours.

Si vous ne spécifiez pas les emplacements préférés, mais que vous avez utilisé la méthode `setCurrentLocation`, le SDK remplit automatiquement les emplacements préférés en fonction de la région actuelle dans laquelle le client se trouve. Le SDK ordonne les régions en fonction de la proximité d’une région par rapport à la région actuelle.

## <a name="net-sdk"></a>Kit de développement logiciel (SDK) .NET

Le SDK peut être utilisé sans aucune modification du code. Dans ce cas, le SDK dirige automatiquement les lectures et les écritures vers la région d’écriture en cours.

Dans la version 1.8 et ultérieure du SDK .NET, le paramètre ConnectionPolicy du constructeur DocumentClient comporte une propriété appelée Microsoft.Azure.Documents.ConnectionPolicy.PreferredLocations. Cette propriété est de type Collection `<string>` et doit contenir une liste des noms de région. Les valeurs de chaîne sont mises en forme en fonction de la colonne Nom de la région, dans la page [Régions Azure][regions], sans espaces avant ou après le premier et le dernier caractère, respectivement.

Les points de terminaison d’écriture et de lecture en cours sont disponibles dans DocumentClient.WriteEndpoint et DocumentClient.ReadEndpoint, respectivement.

> [!NOTE]
> Les URL des points de terminaison ne doivent pas être considérées comme des constantes à long terme. Le service peut les mettre à jour à tout moment. Le SDK gère ce changement automatiquement.
>

# <a name="net-sdk-v2"></a>[Kit de développement logiciel (SDK) .NET V2](#tab/dotnetv2)

Si vous utilisez le SDK .NET v2, utilisez la propriété `PreferredLocations` pour définir la région préférée.

```csharp
// Getting endpoints from application settings or other configuration location
Uri accountEndPoint = new Uri(Properties.Settings.Default.GlobalDatabaseUri);
string accountKey = Properties.Settings.Default.GlobalDatabaseKey;
  
ConnectionPolicy connectionPolicy = new ConnectionPolicy();

//Setting read region selection preference
connectionPolicy.PreferredLocations.Add(LocationNames.WestUS); // first preference
connectionPolicy.PreferredLocations.Add(LocationNames.EastUS); // second preference
connectionPolicy.PreferredLocations.Add(LocationNames.NorthEurope); // third preference

// initialize connection
DocumentClient docClient = new DocumentClient(
    accountEndPoint,
    accountKey,
    connectionPolicy);

// connect to DocDB
await docClient.OpenAsync().ConfigureAwait(false);
```

Vous pouvez également utiliser la propriété `SetCurrentLocation` et laisser le SDK choisir l’emplacement préféré en fonction de la proximité.

```csharp
// Getting endpoints from application settings or other configuration location
Uri accountEndPoint = new Uri(Properties.Settings.Default.GlobalDatabaseUri);
string accountKey = Properties.Settings.Default.GlobalDatabaseKey;
  
ConnectionPolicy connectionPolicy = new ConnectionPolicy();

connectionPolicy.SetCurrentLocation("West US 2"); /

// initialize connection
DocumentClient docClient = new DocumentClient(
    accountEndPoint,
    accountKey,
    connectionPolicy);

// connect to DocDB
await docClient.OpenAsync().ConfigureAwait(false);
```

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET V3](#tab/dotnetv3)

Si vous utilisez le SDK .NET v3, utilisez la propriété `ApplicationPreferredRegions` pour définir la région préférée.

```csharp

CosmosClientOptions options = new CosmosClientOptions();
options.ApplicationName = "MyApp";
options.ApplicationPreferredRegions = new List<string> {Regions.WestUS, Regions.WestUS2};

CosmosClient client = new CosmosClient(connectionString, options);

```

Vous pouvez également utiliser la propriété `ApplicationRegion` et laisser le SDK choisir l’emplacement préféré en fonction de la proximité.

```csharp
CosmosClientOptions options = new CosmosClientOptions();
options.ApplicationName = "MyApp";
// If the application is running in West US
options.ApplicationRegion = Regions.WestUS;

CosmosClient client = new CosmosClient(connectionString, options);
```

---

## <a name="nodejsjavascript"></a>Node.js/JavaScript

> [!NOTE]
> Les URL des points de terminaison ne doivent pas être considérées comme des constantes à long terme. Le service peut les mettre à jour à tout moment. Le SDK gère ce changement automatiquement.
>
>

Voici un exemple de code pour Node.js/Javascript.

```JavaScript
// Setting read region selection preference, in the following order -
// 1 - West US
// 2 - East US
// 3 - North Europe
const preferredLocations = ['West US', 'East US', 'North Europe'];

// initialize the connection
const client = new CosmosClient{ endpoint, key, connectionPolicy: { preferredLocations } });
```

## <a name="python-sdk"></a>Kit de développement logiciel (SDK) Python

Le code suivant montre comment définir des emplacements préférés à l’aide du SDK Python :

```python
connectionPolicy = documents.ConnectionPolicy()
connectionPolicy.PreferredLocations = ['West US', 'East US', 'North Europe']
client = cosmos_client.CosmosClient(ENDPOINT, {'masterKey': MASTER_KEY}, connectionPolicy)

```

## <a name="java-v4-sdk"></a><a id="java4-preferred-locations"></a> SDK Java V4

Le code suivant montre comment définir des emplacements préférés à l’aide du SDK Java :

# <a name="async"></a>[Asynchrone](#tab/api-async)

   API asynchrone du [kit SDK Java V4](sql-api-sdk-java-v4.md) (Maven [com.azure::azure-cosmos](https://mvnrepository.com/artifact/com.azure/azure-cosmos))

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=TutorialGlobalDistributionPreferredLocationAsync)]

# <a name="sync"></a>[Synchronisation](#tab/api-sync)

   API synchrone du [kit SDK Java V4](sql-api-sdk-java-v4.md) (Maven [com.azure::azure-cosmos](https://mvnrepository.com/artifact/com.azure/azure-cosmos))

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=TutorialGlobalDistributionPreferredLocationSync)]

--- 

## <a name="rest"></a>REST

Une fois qu’un compte de base de données est mis à disposition dans plusieurs régions, les clients peuvent interroger sa disponibilité en exécutant une requête GET sur l’URI `https://{databaseaccount}.documents.azure.com/`

Le service renvoie une liste des régions et leurs URI de points de terminaison Azure Cosmos DB correspondants pour les réplicas. La région d’écriture en cours est indiquée dans la réponse. Le client peut ensuite sélectionner le point de terminaison approprié pour toutes les autres requêtes d’API REST, comme suit.

Exemple de réponse

```json
{
    "_dbs": "//dbs/",
    "media": "//media/",
    "writableLocations": [
        {
            "Name": "West US",
            "DatabaseAccountEndpoint": "https://globaldbexample-westus.documents.azure.com:443/"
        }
    ],
    "readableLocations": [
        {
            "Name": "East US",
            "DatabaseAccountEndpoint": "https://globaldbexample-eastus.documents.azure.com:443/"
        }
    ],
    "MaxMediaStorageUsageInMB": 2048,
    "MediaStorageUsageInMB": 0,
    "ConsistencyPolicy": {
        "defaultConsistencyLevel": "Session",
        "maxStalenessPrefix": 100,
        "maxIntervalInSeconds": 5
    },
    "addresses": "//addresses/",
    "id": "globaldbexample",
    "_rid": "globaldbexample.documents.azure.com",
    "_self": "",
    "_ts": 0,
    "_etag": null
}
```

* Les requêtes PUT, POST et DELETE doivent accéder à l’URI d’écriture indiqué
* Toutes les requêtes GET et autres demandes en lecture seule (par ex., Requêtes) peuvent accéder à n’importe quel point de terminaison choisi par le client

L’écriture de demandes dans les régions en lecture seule échoue avec le code d’erreur HTTP 403 (« Interdit »).

Si la région d’écriture change après la phase de découverte initiale du client, les écritures suivantes dans la région d’écriture précédente échouent avec le code d’erreur HTTP 403 (« Interdit »). Le client doit alors exécuter à nouveau la requête GET sur la liste des régions pour obtenir la région d’écriture mise à jour.

C’est ici que s’achève ce didacticiel. Découvrez comment gérer la cohérence de votre compte répliqué à l’échelle mondiale en lisant l’article [Niveaux de cohérence dans Azure Cosmos DB](consistency-levels.md). Pour plus d’informations sur le fonctionnement de la réplication de base de données à l’échelle mondiale dans Azure Cosmos DB, voir [Diffuser des données à l’échelle mondiale avec Azure Cosmos DB](distribute-data-globally.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez :

> [!div class="checklist"]
> * Configurer la diffusion mondiale à l’aide du portail Azure
> * Configurer la distribution mondiale à l’aide des API SQL

Vous pouvez maintenant passer au didacticiel suivant pour apprendre à développer en local à l’aide de l’émulateur local Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Développer en local avec l’émulateur](local-emulator.md)

[regions]: https://azure.microsoft.com/regions/