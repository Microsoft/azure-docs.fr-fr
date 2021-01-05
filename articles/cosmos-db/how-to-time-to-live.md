---
title: Configurer et gérer la durée de vie dans Azure Cosmos DB
description: Découvrir comment configurer et gérer la durée de vie sur un conteneur et un élément dans Azure Cosmos DB
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/11/2020
ms.author: anfeldma
ms.custom: devx-track-js, devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 2ddba95f9ccc25d536638dbc68c41027d26e71c7
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93341006"
---
# <a name="configure-time-to-live-in-azure-cosmos-db"></a>Configurer la durée de vie dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Dans Azure Cosmos DB, vous pouvez choisir de configurer la durée de vie (TTL) au niveau du conteneur, mais vous pouvez aussi la définir au niveau de l’élément après avoir configuré le conteneur. Vous pouvez configurer la TTL d’un conteneur en utilisant le portail Azure ou les kits SDK propres à chaque langage. Les remplacements de la TTL au niveau de l’élément peuvent être configurés avec les kits SDK.

> Ce contenu concerne la TTL des magasins transactionnels d’Azure Cosmos DB. Si vous recherchez la TTL des magasins analytiques, qui permet des scénarios de HTAP NoETL via [Azure Synapse Link](./synapse-link.md), cliquez [ici](./analytical-store-introduction.md#analytical-ttl).

## <a name="enable-time-to-live-on-a-container-using-azure-portal"></a>Activer la durée de vie sur un conteneur avec le portail Azure

Utilisez les étapes suivantes pour activer la durée de vie sur un conteneur sans date d’expiration. Activez cette option pour autoriser le remplacement de la TTL au niveau de l’élément. Vous pouvez également définir la TTL en entrant une valeur différente de zéro pour les secondes.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Créez un compte Azure Cosmos ou sélectionnez un compte existant.

3. Ouvrez le volet **Explorateur de données**.

4. Sélectionnez un conteneur existant, développez-le et modifiez les valeurs suivantes :

   * Ouvrez la fenêtre **Mise à l’échelle et paramètres**.
   * Sous **Paramètres** , recherchez **Durée de vie**.
   * Sélectionnez **Activée (pas par défaut)** ou sélectionnez **Activée** et définissez une valeur de TTL.
   * Cliquez sur **Enregistrer** pour enregistrer les modifications.

   :::image type="content" source="./media/how-to-time-to-live/how-to-time-to-live-portal.png" alt-text="Configurer de la durée de vie dans le portail Azure":::

* Quand DefaultTimeToLive est null, votre Durée de vie est Désactivée
* Lorsque DefaultTimeToLive est -1, votre Durée de vie est Activée (pas par défaut)
* Quand DefaultTimeToLive a une autre valeur Int (à l’exception de 0), votre Durée de vie est Activée

## <a name="enable-time-to-live-on-a-container-using-azure-cli-or-powershell"></a>Activer la durée de vie sur un conteneur avec l’interface Azure CLI ou PowerShell

Pour créer ou activer la TTL sur un conteneur, consultez :

* [Créer un conteneur avec TTL à l’aide d’Azure CLI](manage-with-cli.md#create-a-container-with-ttl)
* [Créer un conteneur avec TTL à l’aide de PowerShell](manage-with-powershell.md#create-container-unique-key-ttl)

## <a name="enable-time-to-live-on-a-container-using-sdk"></a>Activer la durée de vie sur un conteneur à l’aide du kit SDK

### <a name="net-sdk"></a><a id="dotnet-enable-noexpiry"></a> Kit de développement logiciel (SDK) .NET

# <a name="net-sdk-v2"></a>[Kit de développement logiciel (SDK) .NET V2](#tab/dotnetv2)

.NET SDK V2 (Microsoft.Azure.DocumentDB)

```csharp
// Create a new container with TTL enabled and without any expiration value
DocumentCollection collectionDefinition = new DocumentCollection();
collectionDefinition.Id = "myContainer";
collectionDefinition.PartitionKey.Paths.Add("/myPartitionKey");
collectionDefinition.DefaultTimeToLive = -1; //(never expire by default)

DocumentCollection ttlEnabledCollection = await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("myDatabaseName"),
    collectionDefinition);
```

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET V3](#tab/dotnetv3)

.NET SDK V3 (Microsoft.Azure.Cosmos)

```csharp
// Create a new container with TTL enabled and without any expiration value
await client.GetDatabase("database").CreateContainerAsync(new ContainerProperties
{
    Id = "container",
    PartitionKeyPath = "/myPartitionKey",
    DefaultTimeToLive = -1 //(never expire by default)
});
```
---

### <a name="java-sdk"></a><a id="java-enable-noexpiry"></a> Kit de développement logiciel (SDK) Java

# <a name="java-sdk-v4"></a>[Kit SDK Java V4](#tab/javav4)

Kit de développement logiciel (SDK) Java V4 (Maven com.azure::azure-cosmos)

```java
CosmosAsyncContainer container;

// Create a new container with TTL enabled and without any expiration value
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
containerProperties.setDefaultTimeToLiveInSeconds(-1);
container = database.createContainerIfNotExists(containerProperties, 400).block().getContainer();
```

# <a name="java-sdk-v3"></a>[Kit SDK Java V3](#tab/javav3)

Kit de développement logiciel (SDK) Java V3 (Maven com.microsoft.azure::azure-cosmos)

```java
CosmosContainer container;

// Create a new container with TTL enabled and without any expiration value
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
containerProperties.defaultTimeToLive(-1);
container = database.createContainerIfNotExists(containerProperties, 400).block().container();
```
---

## <a name="set-time-to-live-on-a-container-using-sdk"></a>Définir la durée de vie sur un conteneur à l’aide du kit SDK

Pour définir la durée de vie sur un conteneur, vous devez fournir un nombre positif non nul qui indique la durée en secondes. Selon la valeur de la TTL configurée, tous les éléments présents dans le conteneur sont supprimés après l’horodatage de la dernière modification de l’élément `_ts`. Vous pouvez éventuellement définir `TimeToLivePropertyPath`, ce qui permet d’utiliser une autre propriété que la propriété `_ts` générée par le système pour déterminer les éléments à supprimer en fonction de la durée de vie.

### <a name="net-sdk"></a><a id="dotnet-enable-withexpiry"></a> Kit de développement logiciel (SDK) .NET

# <a name="net-sdk-v2"></a>[Kit de développement logiciel (SDK) .NET V2](#tab/dotnetv2)

.NET SDK V2 (Microsoft.Azure.DocumentDB)

```csharp
// Create a new container with TTL enabled and a 90 day expiration
DocumentCollection collectionDefinition = new DocumentCollection();
collectionDefinition.Id = "myContainer";
collectionDefinition.PartitionKey.Paths.Add("/myPartitionKey");
collectionDefinition.DefaultTimeToLive = 90 * 60 * 60 * 24 // expire all documents after 90 days

DocumentCollection ttlEnabledCollection = await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("myDatabaseName"),
    collectionDefinition;
```

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET V3](#tab/dotnetv3)

.NET SDK V3 (Microsoft.Azure.Cosmos)

```csharp
// Create a new container with TTL enabled and a 90 day expiration
await client.GetDatabase("database").CreateContainerAsync(new ContainerProperties
{
    Id = "container",
    PartitionKeyPath = "/myPartitionKey",
    DefaultTimeToLive = 90 * 60 * 60 * 24 // expire all documents after 90 days
});
```
---

### <a name="java-sdk"></a><a id="java-enable-defaultexpiry"></a> Kit de développement logiciel (SDK) Java

# <a name="java-sdk-v4"></a>[Kit SDK Java V4](#tab/javav4)

Kit de développement logiciel (SDK) Java V4 (Maven com.azure::azure-cosmos)

```java
CosmosAsyncContainer container;

// Create a new container with TTL enabled with default expiration value
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
containerProperties.setDefaultTimeToLiveInSeconds(90 * 60 * 60 * 24);
container = database.createContainerIfNotExists(containerProperties, 400).block().getContainer();
```

# <a name="java-sdk-v3"></a>[Kit SDK Java V3](#tab/javav3)

Kit de développement logiciel (SDK) Java V3 (Maven com.microsoft.azure::azure-cosmos)

```java
CosmosContainer container;

// Create a new container with TTL enabled with default expiration value
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
containerProperties.defaultTimeToLive(90 * 60 * 60 * 24);
container = database.createContainerIfNotExists(containerProperties, 400).block().container();
```
---

### <a name="nodejs-sdk"></a><a id="nodejs-enable-withexpiry"></a>Kit de développement logiciel (SDK) NodeJS

```javascript
const containerDefinition = {
          id: "sample container1",
        };

async function createcontainerWithTTL(db: Database, containerDefinition: ContainerDefinition, collId: any, defaultTtl: number) {
      containerDefinition.id = collId;
      containerDefinition.defaultTtl = defaultTtl;
      await db.containers.create(containerDefinition);
}
```

## <a name="set-time-to-live-on-an-item"></a>Définir la durée de vie sur un élément

Non seulement vous pouvez définir une durée de vie par défaut sur un conteneur, mais vous pouvez aussi définir une durée de vie pour un élément. La définition de la durée de vie au niveau d’un élément remplace la valeur TTL par défaut de l’élément dans le conteneur donné.

* Pour définir la TTL sur un élément, vous devez fournir un nombre positif non nul qui indique, en secondes, le délai d’expiration de l’élément après l’horodatage de la dernière modification de l’élément `_ts`.

* Si l’élément ne présente aucun champ TTL, la durée de vie définie par défaut dans le conteneur s’applique à l’élément.

* Si la TTL est désactivée au niveau du conteneur, le champ TTL de l’élément est ignoré jusqu’à ce que la TTL soit de nouveau activée sur le conteneur.

### <a name="azure-portal"></a><a id="portal-set-ttl-item"></a>Portail Azure

Utilisez les étapes suivantes pour activer la durée de vie sur un élément :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Créez un compte Azure Cosmos ou sélectionnez un compte existant.

3. Ouvrez le volet **Explorateur de données**.

4. Sélectionnez un conteneur existant, développez-le et modifiez les valeurs suivantes :

   * Ouvrez la fenêtre **Mise à l’échelle et paramètres**.
   * Sous **Paramètres** , recherchez **Durée de vie**.
   * Sélectionnez **Activée (pas par défaut)** ou sélectionnez **Activée** et définissez une valeur de TTL. 
   * Cliquez sur **Enregistrer** pour enregistrer les modifications.

5. Accédez ensuite à l’élément pour lequel vous souhaitez définir une durée de vie, ajoutez la propriété `ttl`, puis sélectionnez **Mettre à jour**. 

   ```json
   {
    "id": "1",
    "_rid": "Jic9ANWdO-EFAAAAAAAAAA==",
    "_self": "dbs/Jic9AA==/colls/Jic9ANWdO-E=/docs/Jic9ANWdO-EFAAAAAAAAAA==/",
    "_etag": "\"0d00b23f-0000-0000-0000-5c7712e80000\"",
    "_attachments": "attachments/",
    "ttl": 10,
    "_ts": 1551307496
   }
   ```

### <a name="net-sdk-any"></a><a id="dotnet-set-ttl-item"></a>.NET SDK (n’importe quelle version)

```csharp
// Include a property that serializes to "ttl" in JSON
public class SalesOrder
{
    [JsonProperty(PropertyName = "id")]
    public string Id { get; set; }
    [JsonProperty(PropertyName="cid")]
    public string CustomerId { get; set; }
    // used to set expiration policy
    [JsonProperty(PropertyName = "ttl", NullValueHandling = NullValueHandling.Ignore)]
    public int? ttl { get; set; }

    //...
}
// Set the value to the expiration in seconds
SalesOrder salesOrder = new SalesOrder
{
    Id = "SO05",
    CustomerId = "CO18009186470",
    ttl = 60 * 60 * 24 * 30;  // Expire sales orders in 30 days
};
```

### <a name="nodejs-sdk"></a><a id="nodejs-set-ttl-item"></a>Kit de développement logiciel (SDK) NodeJS

```javascript
const itemDefinition = {
          id: "doc",
          name: "sample Item",
          key: "value",
          ttl: 2
        };
```

### <a name="java-sdk"></a><a id="java-set-ttl-item"></a> Kit de développement logiciel (SDK) Java

# <a name="java-sdk-v4"></a>[Kit SDK Java V4](#tab/javav4)

Kit de développement logiciel (SDK) Java V4 (Maven com.azure::azure-cosmos)

```java
// Include a property that serializes to "ttl" in JSON
public class SalesOrder
{
    private String id;
    private String customerId;
    private Integer ttl;

    public SalesOrder(String id, String customerId, Integer ttl) {
        this.id = id;
        this.customerId = customerId;
        this.ttl = ttl;
    }

    public String getId() {return this.id;}
    public void setId(String new_id) {this.id = new_id;}
    public String getCustomerId() {return this.customerId;}
    public void setCustomerId(String new_cid) {this.customerId = new_cid;}
    public Integer getTtl() {return this.ttl;}
    public void setTtl(Integer new_ttl) {this.ttl = new_ttl;}

    //...
}

// Set the value to the expiration in seconds
SalesOrder salesOrder = new SalesOrder(
    "SO05",
    "CO18009186470",
    60 * 60 * 24 * 30  // Expire sales orders in 30 days
);

```

# <a name="java-sdk-v3"></a>[Kit SDK Java V3](#tab/javav3)

Kit de développement logiciel (SDK) Java V3 (Maven com.microsoft.azure::azure-cosmos)

```java
// Include a property that serializes to "ttl" in JSON
public class SalesOrder
{
    private String id;
    private String customerId;
    private Integer ttl;

    public SalesOrder(String id, String customerId, Integer ttl) {
        this.id = id;
        this.customerId = customerId;
        this.ttl = ttl;
    }

    public String id() {return this.id;}
    public void id(String new_id) {this.id = new_id;}
    public String customerId() {return this.customerId;}
    public void customerId(String new_cid) {this.customerId = new_cid;}
    public Integer ttl() {return this.ttl;}
    public void ttl(Integer new_ttl) {this.ttl = new_ttl;}

    //...
}

// Set the value to the expiration in seconds
SalesOrder salesOrder = new SalesOrder(
    "SO05",
    "CO18009186470",
    60 * 60 * 24 * 30  // Expire sales orders in 30 days
);

```
---

## <a name="reset-time-to-live"></a>Réinitialiser la durée de vie

Vous pouvez réinitialiser la durée de vie sur un élément en effectuant une opération d’écriture ou de mise à jour sur celui-ci. L’opération d’écriture ou de mise à jour définit l’élément `_ts` sur l’heure actuelle, et la durée de vie déclenchant l’expiration de l’élément recommence son décompte. Si vous souhaitez modifier la durée de vie d’un élément, vous pouvez mettre à jour ce champ de la même manière que vous mettriez à jour n’importe quel autre champ.

### <a name="net-sdk"></a><a id="dotnet-extend-ttl-item"></a> Kit de développement logiciel (SDK) .NET

# <a name="net-sdk-v2"></a>[Kit de développement logiciel (SDK) .NET V2](#tab/dotnetv2)

.NET SDK V2 (Microsoft.Azure.DocumentDB)

```csharp
// This examples leverages the Sales Order class above.
// Read a document, update its TTL, save it.
response = await client.ReadDocumentAsync(
    "/dbs/salesdb/colls/orders/docs/SO05"),
    new RequestOptions { PartitionKey = new PartitionKey("CO18009186470") });

Document readDocument = response.Resource;
readDocument.ttl = 60 * 30 * 30; // update time to live
response = await client.ReplaceDocumentAsync(readDocument);
```

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET V3](#tab/dotnetv3)

.NET SDK V3 (Microsoft.Azure.Cosmos)

```csharp
// This examples leverages the Sales Order class above.
// Read a document, update its TTL, save it.
ItemResponse<SalesOrder> itemResponse = await client.GetContainer("database", "container").ReadItemAsync<SalesOrder>("SO05", new PartitionKey("CO18009186470"));

itemResponse.Resource.ttl = 60 * 30 * 30; // update time to live
await client.GetContainer("database", "container").ReplaceItemAsync(itemResponse.Resource, "SO05");
```
---

### <a name="java-sdk"></a><a id="java-enable-modifyitemexpiry"></a> Kit de développement logiciel (SDK) Java

# <a name="java-sdk-v4"></a>[Kit SDK Java V4](#tab/javav4)

Kit de développement logiciel (SDK) Java V4 (Maven com.azure::azure-cosmos)

```java
// This examples leverages the Sales Order class above.
// Read a document, update its TTL, save it.
CosmosAsyncItemResponse<SalesOrder> itemResponse = container.readItem("SO05", new PartitionKey("CO18009186470"), SalesOrder.class)
        .flatMap(readResponse -> {
            SalesOrder salesOrder = readResponse.getItem();
            salesOrder.setTtl(60 * 30 * 30);
            return container.createItem(salesOrder);
}).block();
```

# <a name="java-sdk-v3"></a>[Kit SDK Java V3](#tab/javav3)

Kit de développement logiciel (SDK) V3 (Maven com.microsoft.azure::azure-cosmos)

```java
// This examples leverages the Sales Order class above.
// Read a document, update its TTL, save it.
container.getItem("SO05", new PartitionKey("CO18009186470")).read()
        .flatMap(readResponse -> {
            SalesOrder salesOrder = null;
            try {
                salesOrder = readResponse.properties().getObject(SalesOrder.class);
            } catch (Exception err) {

            }
            salesOrder.ttl(60 * 30 * 30);
            return container.createItem(salesOrder);
}).block();
```
---

## <a name="turn-off-time-to-live"></a>Suspendre la durée de vie

Si la durée de vie a été définie sur un élément et que vous ne souhaitez plus faire expirer cet élément, vous pouvez récupérer l’élément, supprimer le champ TTL et replacer l’élément sur le serveur. Lorsque le champ TTL est supprimé de l’élément, la valeur de durée de vie par défaut affectée au conteneur est appliquée à l’élément. Définissez la valeur de durée de vie sur -1, pour empêcher l’expiration d’un élément et pour ne pas hériter de la valeur de durée de vie du conteneur.

### <a name="net-sdk"></a><a id="dotnet-turn-off-ttl-item"></a> Kit de développement logiciel (SDK) .NET

# <a name="net-sdk-v2"></a>[Kit de développement logiciel (SDK) .NET V2](#tab/dotnetv2)

.NET SDK V2 (Microsoft.Azure.DocumentDB)

```csharp
// This examples leverages the Sales Order class above.
// Read a document, turn off its override TTL, save it.
response = await client.ReadDocumentAsync(
    "/dbs/salesdb/colls/orders/docs/SO05"),
    new RequestOptions { PartitionKey = new PartitionKey("CO18009186470") });

Document readDocument = response.Resource;
readDocument.ttl = null; // inherit the default TTL of the container

response = await client.ReplaceDocumentAsync(readDocument);
```

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET V3](#tab/dotnetv3)

.NET SDK V3 (Microsoft.Azure.Cosmos)

```csharp
// This examples leverages the Sales Order class above.
// Read a document, turn off its override TTL, save it.
ItemResponse<SalesOrder> itemResponse = await client.GetContainer("database", "container").ReadItemAsync<SalesOrder>("SO05", new PartitionKey("CO18009186470"));

itemResponse.Resource.ttl = null; // inherit the default TTL of the container
await client.GetContainer("database", "container").ReplaceItemAsync(itemResponse.Resource, "SO05");
```
---

### <a name="java-sdk"></a><a id="java-enable-itemdefaultexpiry"></a> Kit de développement logiciel (SDK) Java

# <a name="java-sdk-v4"></a>[Kit SDK Java V4](#tab/javav4)

Kit de développement logiciel (SDK) Java V4 (Maven com.azure::azure-cosmos)

```java
// This examples leverages the Sales Order class above.
// Read a document, update its TTL, save it.
CosmosAsyncItemResponse<SalesOrder> itemResponse = container.readItem("SO05", new PartitionKey("CO18009186470"), SalesOrder.class)
        .flatMap(readResponse -> {
            SalesOrder salesOrder = readResponse.getItem();
            salesOrder.setTtl(null);
            return container.createItem(salesOrder);
}).block();
```

# <a name="java-sdk-v3"></a>[Kit SDK Java V3](#tab/javav3)

Kit de développement logiciel (SDK) Java V3 (Maven com.microsoft.azure::azure-cosmos)

```java
// This examples leverages the Sales Order class above.
// Read a document, update its TTL, save it.
container.getItem("SO05", new PartitionKey("CO18009186470")).read()
        .flatMap(readResponse -> {
            SalesOrder salesOrder = null;
            try {
                salesOrder = readResponse.properties().getObject(SalesOrder.class);
            } catch (Exception err) {

            }
            salesOrder.ttl(null);
            return container.createItem(salesOrder);
}).block();
```
---

## <a name="disable-time-to-live"></a>Désactiver la durée de vie

Pour désactiver la durée de vie sur un conteneur et arrêter le processus en arrière-plan qui recherche les éléments ayant expiré, la propriété `DefaultTimeToLive` sur le conteneur doit être supprimée. Supprimer cette propriété ne revient pas à la définir sur -1. Lorsque vous la définissez sur -1, les nouveaux éléments ajoutés au conteneur continuent de vivre indéfiniment, mais vous pouvez remplacer cette valeur sur des éléments spécifiques du conteneur. Lorsque vous supprimez du conteneur la propriété de durée de vie, les éléments n’expirent jamais, même si la précédente valeur de durée de vie par défaut a été explicitement remplacée.

### <a name="net-sdk"></a><a id="dotnet-disable-ttl"></a> Kit de développement logiciel (SDK) .NET

# <a name="net-sdk-v2"></a>[Kit de développement logiciel (SDK) .NET V2](#tab/dotnetv2)

.NET SDK V2 (Microsoft.Azure.DocumentDB)

```csharp
// Get the container, update DefaultTimeToLive to null
DocumentCollection collection = await client.ReadDocumentCollectionAsync("/dbs/salesdb/colls/orders");
// Disable TTL
collection.DefaultTimeToLive = null;
await client.ReplaceDocumentCollectionAsync(collection);
```

# <a name="net-sdk-v3"></a>[Kit de développement logiciel (SDK) .NET V3](#tab/dotnetv3)

.NET SDK V3 (Microsoft.Azure.Cosmos)

```csharp
// Get the container, update DefaultTimeToLive to null
ContainerResponse containerResponse = await client.GetContainer("database", "container").ReadContainerAsync();
// Disable TTL
containerResponse.Resource.DefaultTimeToLive = null;
await client.GetContainer("database", "container").ReplaceContainerAsync(containerResponse.Resource);
```
---

### <a name="java-sdk"></a><a id="java-enable-disableexpiry"></a> Kit de développement logiciel (SDK) Java

# <a name="java-sdk-v4"></a>[Kit SDK Java V4](#tab/javav4)

Kit de développement logiciel (SDK) Java V4 (Maven com.azure::azure-cosmos)

```java
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
// Disable TTL
containerProperties.setDefaultTimeToLiveInSeconds(null);
// Update container settings
container.replace(containerProperties).block();
```

# <a name="java-sdk-v3"></a>[Kit SDK Java V3](#tab/javav3)

Kit de développement logiciel (SDK) Java V3 (Maven com.microsoft.azure::azure-cosmos)

```java
CosmosContainer container;

// Create a new container with TTL enabled and without any expiration value
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
// Disable TTL
containerProperties.defaultTimeToLive(null);
// Update container settings
container = database.createContainerIfNotExists(containerProperties, 400).block().container();
```
---

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur la durée de vie dans l’article suivant :

* [Durée de vie](time-to-live.md)