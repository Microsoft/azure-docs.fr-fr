---
title: Guide pratique pour configurer les écritures multirégions dans Azure Cosmos DB
description: Découvrez comment configurer les fonctionnalités multirégions pour vos applications en utilisant différents SDK dans Azure Cosmos DB.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 09/10/2020
ms.author: mjbrown
ms.custom: devx-track-python, devx-track-js, devx-track-csharp, "seo-nov-2020"
ms.openlocfilehash: 6f71f4c0ec353f36614ea6dcabf4d698b31baacb
ms.sourcegitcommit: 2a8a53e5438596f99537f7279619258e9ecb357a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94336724"
---
# <a name="configure-multi-region-writes-in-your-applications-that-use-azure-cosmos-db"></a>Configurer les fonctionnalités multirégions dans les applications qui utilisent Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Une fois qu’un compte a été créé avec plusieurs régions d’écriture activées, vous devez apporter deux modifications à votre application pour que ConnectionPolicy et DocumentClient activent les écritures multirégions et les fonctionnalités multihébergement dans Azure Cosmos DB. Dans ConnectionPolicy, définissez UseMultipleWriteLocations sur true et passez le nom de la région où l’application est déployée à SetCurrentLocation. Cela remplira la propriété PreferredLocations en fonction de la géo-proximité à partir de l’emplacement transmis. Si une nouvelle région est ultérieurement ajoutée au compte, l’application n’aura pas à être mise à jour ou redéployée ; elle détectera automatiquement la région la plus proche et se repliera automatiquement dessus en cas d’événement régional.

> [!Note]
> Les comptes Cosmos initialement configurés avec une seule région d’écriture peuvent être configurés pour plusieurs régions d’écriture avec un temps de non-disponibilité égal à zéro. Pour en savoir plus, consultez [Configurer plusieurs régions d’écriture](how-to-manage-database-account.md#configure-multiple-write-regions)

## <a name="azure-portal"></a><a id="portal"></a> Portail Azure

Pour activer les écritures multirégions à partir du portail Azure, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Accédez à votre compte Azure Cosmos et, dans le menu, ouvrez le volet **Répliquer les données globalement**.

1. Sous l’option **Écritures multirégions**, choisissez **Activer**. Cela ajoute automatiquement les régions existantes aux régions en lecture et en écriture.

1. Vous pouvez ajouter des régions supplémentaires en sélectionnant les icônes sur la carte ou en sélectionnant le bouton **Ajouter une région**. Toutes les régions que vous ajoutez seront disponibles en lecture et en écriture.

1. Après avoir mis à jour la liste des régions, sélectionnez **Enregistrer** pour appliquer les modifications.

   :::image type="content" source="./media/how-to-multi-master/enable-multi-region-writes.png" alt-text="Capture d’écran pour permettre les écritures multirégions à l’aide du portail Azure" lightbox="./media/how-to-multi-master/enable-multi-region-writes.png":::

## <a name="net-sdk-v2"></a><a id="netv2"></a>Kit SDK .NET v2

Pour activer les écritures multirégions dans votre application, définissez `UseMultipleWriteLocations` sur `true`. Définissez également `SetCurrentLocation` en fonction de la région sur laquelle l’application est déployée et où Azure Cosmos DB est répliquée :

```csharp
ConnectionPolicy policy = new ConnectionPolicy
    {
        ConnectionMode = ConnectionMode.Direct,
        ConnectionProtocol = Protocol.Tcp,
        UseMultipleWriteLocations = true
    };
policy.SetCurrentLocation("West US 2");
```

## <a name="net-sdk-v3"></a><a id="netv3"></a>Kit SDK .NET v3

Pour activer les écritures multirégions dans votre application, définissez `ApplicationRegion` en fonction de la région sur laquelle l’application est déployée et où Cosmos DB est répliquée :

```csharp
CosmosClient cosmosClient = new CosmosClient(
    "<connection-string-from-portal>", 
    new CosmosClientOptions()
    {
        ApplicationRegion = Regions.WestUS2,
    });
```

Si vous le souhaitez, vous pouvez utiliser `CosmosClientBuilder` et `WithApplicationRegion` pour obtenir le même résultat :

```csharp
CosmosClientBuilder cosmosClientBuilder = new CosmosClientBuilder("<connection-string-from-portal>")
            .WithApplicationRegion(Regions.WestUS2);
CosmosClient client = cosmosClientBuilder.Build();
```

## <a name="java-v4-sdk"></a><a id="java4-multi-region-writes"></a> SDK Java V4

Pour activer les écritures multirégions dans votre application, appelez les fonctions `.multipleWriteRegionsEnabled(true)` et `.preferredRegions(preferredRegions)` dans le générateur de clients, où `preferredRegions` est une `List` contenant un élément, c’est-à-dire la région dans laquelle l’application est déployée et où Cosmos DB est répliqué :

# <a name="async"></a>[Async](#tab/api-async)

   API asynchrone du [kit SDK Java V4](sql-api-sdk-java-v4.md) (Maven [com.azure::azure-cosmos](https://mvnrepository.com/artifact/com.azure/azure-cosmos))

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=ConfigureMultimasterAsync)]

# <a name="sync"></a>[Synchronisation](#tab/api-sync)

   API synchrone du [kit SDK Java V4](sql-api-sdk-java-v4.md) (Maven [com.azure::azure-cosmos](https://mvnrepository.com/artifact/com.azure/azure-cosmos))

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=ConfigureMultimasterSync)]

--- 

## <a name="async-java-v2-sdk"></a><a id="java2-multi-region-writes"></a> Kit de développement logiciel (SDK) Async Java v2

Le kit de développement logiciel (SDK) Java v2 utilisait l’outil Maven [com.microsoft.azure::azure-cosmosdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb). Pour activer les écritures multirégions dans votre application, définissez `policy.setUsingMultipleWriteLocations(true)` et `policy.setPreferredLocations` en fonction de la région sur laquelle l’application est déployée et où Cosmos DB est répliquée :

```java
ConnectionPolicy policy = new ConnectionPolicy();
policy.setUsingMultipleWriteLocations(true);
policy.setPreferredLocations(Collections.singletonList(region));

AsyncDocumentClient client =
    new AsyncDocumentClient.Builder()
        .withMasterKeyOrResourceToken(this.accountKey)
        .withServiceEndpoint(this.accountEndpoint)
        .withConsistencyLevel(ConsistencyLevel.Eventual)
        .withConnectionPolicy(policy).build();
```

## <a name="nodejs-javascript-and-typescript-sdks"></a><a id="javascript"></a>Kits SDK Node.js, JavaScript et TypeScript

Pour activer les écritures multirégions dans votre application, définissez `connectionPolicy.UseMultipleWriteLocations` sur `true`. Définissez également `connectionPolicy.PreferredLocations` en fonction de la région sur laquelle l’application est déployée et où Cosmos DB est répliquée :

```javascript
const connectionPolicy: ConnectionPolicy = new ConnectionPolicy();
connectionPolicy.UseMultipleWriteLocations = true;
connectionPolicy.PreferredLocations = [region];

const client = new CosmosClient({
  endpoint: config.endpoint,
  auth: { masterKey: config.key },
  connectionPolicy,
  consistencyLevel: ConsistencyLevel.Eventual
});
```

## <a name="python-sdk"></a><a id="python"></a>Kit SDK Python

Pour activer les écritures multirégions dans votre application, définissez `connection_policy.UseMultipleWriteLocations` sur `true`. Définissez également `connection_policy.PreferredLocations` en fonction de la région sur laquelle l’application est déployée et où Cosmos DB est répliquée.

```python
connection_policy = documents.ConnectionPolicy()
connection_policy.UseMultipleWriteLocations = True
connection_policy.PreferredLocations = [region]

client = cosmos_client.CosmosClient(self.account_endpoint, {
                                    'masterKey': self.account_key}, connection_policy, documents.ConsistencyLevel.Session)
```

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants :

* [Utiliser des jetons de sessions pour gérer la cohérence dans Azure Cosmos DB](how-to-manage-consistency.md#utilize-session-tokens)
* [Types de conflit et stratégies de résolution dans Azure Cosmos DB](conflict-resolution-policies.md)
* [Haute disponibilité dans Azure Cosmos DB](high-availability.md)
* [Niveaux de cohérence dans Azure Cosmos DB](consistency-levels.md)
* [Choisir le niveau de cohérence approprié dans Azure Cosmos DB](./consistency-levels.md)
* [Compromis entre cohérence, disponibilité et niveau de performance dans Azure Cosmos DB](./consistency-levels.md)
* [Compromis entre disponibilité et performance pour différents niveaux de cohérence](./consistency-levels.md)
* [Mise à l’échelle du débit provisionné au niveau global](./request-units.md)
* [Distribution mondiale : Sous le capot](global-dist-under-the-hood.md)