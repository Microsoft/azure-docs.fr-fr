---
title: Rechercher les frais d’unités de requête (RU) pour une les requêtes d’une API Gremlin dans Azure Cosmos DB
description: Découvrez comment trouver les frais d’unités de requête pour les requêtes de Gremlin exécutées sur un conteneur Azure Cosmos. Vous pouvez utiliser les pilotes du portail Azure, .NET et Java pour rechercher les frais de RU.
author: ThomasWeiss
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 10/14/2020
ms.author: thweiss
ms.custom: devx-track-js
ms.openlocfilehash: 8393fb431c5148d3f4885135c90fe4d0b8970d52
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93082009"
---
# <a name="find-the-request-unit-charge-for-operations-executed-in-azure-cosmos-db-gremlin-api"></a>Rechercher les frais des unités de requête pour les opérations exécutées dans l’API Gremlin d’Azure Cosmos DB
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

Azure Cosmos DB prend en charge de nombreuses API, notamment SQL, MongoDB, Cassandra, Gremlin et Table. Chaque API possède son propre ensemble d’opérations de base de données, qui vont de simples opérations ponctuelles de lecture et d'écriture à des requêtes complexes. Chaque opération de base de données consomme des ressources système en fonction de sa complexité.

Le coût de toutes les opérations de base de données, normalisé par Azure Cosmos DB, est exprimé en unités de requête (RU). Les RU correspondent en quelque sorte à une devise de performances, faisant abstraction des ressources système (UC, IOPS, mémoire, etc.) requises pour effectuer les opérations de base de données prises en charge par Azure Cosmos DB. Quelle que soit l’API utilisée pour interagir avec le conteneur Azure Cosmos, les coûts sont toujours mesurés en unités de requête, Que l’opération de base de données soit une opération d’écriture, de lecture de point ou de requête, les coûts sont toujours mesurés en unités de requête. Pour plus d’informations, consultez l’article [Unités de requête et considérations](request-units.md).

Cet article présente les différentes façons de rechercher la consommation d’[unités de requête](request-units.md) pour toute opération exécutée sur un conteneur dans l’API Gremlin d’Azure Cosmos DB. Si vous utilisez une autre API, consultez les articles [API pour MongoDB](find-request-unit-charge-mongodb.md), [API Cassandra](find-request-unit-charge-cassandra.md), [API SQL](find-request-unit-charge.md) et [API Table](find-request-unit-charge-table.md) pour trouver les frais de RU/s.

Les en-têtes retournés par l’API Gremlin sont mappés aux attributs d’état personnalisés qui sont actuellement exposés par les Kits de développement logiciel (SDK) Java et .NET Gremlin. Les frais de demande sont disponibles sous la clé `x-ms-request-charge`. Lorsque vous utilisez l’API Gremlin, vous disposez de plusieurs options pour rechercher la consommation d’unités de requête pour une opération sur un conteneur Azure Cosmos.

## <a name="use-the-azure-portal"></a>Utilisation du portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. [Créez un compte Azure Cosmos](create-graph-gremlin-console.md#create-a-database-account) et remplissez-le avec des données, ou sélectionnez un compte existant qui contient déjà des données.

1. Accédez au volet **Explorateur de données** , puis sélectionnez le conteneur sur lequel vous voulez travailler.

1. Entrez une requête valide, puis sélectionnez **Exécuter la requête Gremlin**.

1. Sélectionnez **Statistiques des requêtes** pour afficher les frais réels de la demande que vous avez exécutée.

:::image type="content" source="./media/find-request-unit-charge/portal-gremlin-query.png" alt-text="Capture d’écran d’obtention des frais de requête Gremlin sur le portail Azure":::

## <a name="use-the-net-sdk-driver"></a>Utiliser le pilote SDK .NET

Lorsque vous utilisez le [Kit de développement logiciel (SDK) .NET Gremlin](https://www.nuget.org/packages/Gremlin.Net/), les attributs d’état sont disponibles sous la propriété `StatusAttributes` de l’objet `ResultSet<>` :

```csharp
ResultSet<dynamic> results = client.SubmitAsync<dynamic>("g.V().count()").Result;
double requestCharge = (double)results.StatusAttributes["x-ms-request-charge"];
```

Pour plus d’informations, consultez [Démarrage rapide : Générer une application .NET Framework ou Core à l’aide d’un compte d’API Gremlin Azure Cosmos DB](create-graph-dotnet.md).

## <a name="use-the-java-sdk-driver"></a>Utiliser le pilote de SDK Java

Lorsque vous utilisez le [Kit de développement logiciel (SDK) Java Gremlin ](https://mvnrepository.com/artifact/org.apache.tinkerpop/gremlin-driver), vous pouvez récupérer les attributs d’état en appelant la méthode `statusAttributes()` sur l’objet `ResultSet` :

```java
ResultSet results = client.submit("g.V().count()");
Double requestCharge = (Double)results.statusAttributes().get().get("x-ms-request-charge");
```

Pour plus d’informations, consultez [Démarrage rapide : Générer une base de données de graphe dans Azure Cosmos DB à l’aide du Kit de développement logiciel (SDK) Java](create-graph-java.md).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’optimisation de votre consommation d’unités de requête, consultez les articles suivants :

* [Unités de requête et débit dans Azure Cosmos DB](request-units.md)
* [Optimiser le coût du débit approvisionné dans Azure Cosmos DB](optimize-cost-throughput.md)
* [Optimiser le coût de requête dans Azure Cosmos DB](./optimize-cost-reads-writes.md)