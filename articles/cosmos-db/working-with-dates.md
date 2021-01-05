---
title: Utilisation des dates dans Azure Cosmos DB
description: Découvrir comment stocker, indexer et interroger des objets DataTime dans Azure Cosmos DB
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
author: SnehaGunda
ms.author: sngun
ms.topic: conceptual
ms.date: 04/03/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: ba4d4e63bdd1e795bc1c599d0eae8a595aa0d643
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359521"
---
# <a name="working-with-dates-in-azure-cosmos-db"></a>Utilisation des dates dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB offre une flexibilité des schémas et une indexation riche par le biais d’un modèle de données [JSON](https://www.json.org) natif. Toutes les ressources Azure Cosmos DB, y compris les bases de données, les conteneurs, les documents et les procédures stockées, sont modélisées et stockées en tant que documents JSON. Pour être portable, JSON (et Azure Cosmos DB) ne prend en charge qu’un petit ensemble de types de base : chaîne, nombre, booléen, tableau, objet et null. Toutefois, JSON est flexible et permet aux développeurs et aux infrastructures de représenter des types plus complexes en utilisant ces primitives et en les composant en tant qu’objets ou tableaux.

Outre les types de base, de nombreuses applications ont besoin du type DateTime pour représenter des dates et des timestamps. Cet article décrit comment des développeurs peuvent stocker, récupérer et interroger des dates dans Azure Cosmos DB à l’aide du kit SDK .NET.

## <a name="storing-datetimes"></a>Stockage de valeurs DateTime

Azure Cosmos DB prend en charge les types JSON tels que les chaînes, nombres, booléens, valeurs Null, tableaux et objets. Il ne prend pas directement en charge le type DateTime. Actuellement, Azure Cosmos DB ne prend pas en charge la localisation des dates. Vous devez donc stocker les valeurs DateTime en tant que chaînes. Le format recommandé pour les chaînes DateTime dans Azure Cosmos DB est le format `yyyy-MM-ddTHH:mm:ss.fffffffZ` conforme à la norme ISO 8601 UTC. Il est recommandé de stocker toutes les dates dans Azure Cosmos DB au format UTC. La conversion des chaînes de dates dans ce format permet de trier les dates de manière lexicographique. Si des dates non-UTC sont stockées, la logique doit être gérée côté client. La conversion d’une valeur DateTime locale au format UTC implique que le décalage soit connu/stocké comme propriété dans le code JSON. Le client peut alors utiliser ce décalage pour calculer la valeur DateTime au format UTC.

Les requêtes de plage avec chaînes DateTime en tant que filtres sont uniquement prises en charge si les chaînes DateTime sont au format UTC et de même longueur. Dans Azure Cosmos DB, la fonction système [GetCurrentDateTime](sql-query-getcurrentdatetime.md) renvoie la valeur de chaîne de date ISO 8601 et d'heure UTC au format : `yyyy-MM-ddTHH:mm:ss.fffffffZ`.

La plupart des applications peuvent utiliser la représentation sous forme de chaîne par défaut pour DateTime pour les raisons suivantes :

* Les chaînes peuvent être comparées et l’ordre relatif des valeurs DateTime est conservé lorsque celles-ci sont transformées en chaînes.
* Cette approche ne nécessite pas de code personnalisé ou d’attributs pour la conversion JSON.
* Les dates stockées au format JSON sont lisibles.
* Cette approche peut tirer parti de l’index d’Azure Cosmos DB pour offrir des performances de requête rapides.

Par exemple, l’extrait de code suivant stocke un objet `Order` contenant deux propriétés DateTime (`ShipDate` et `OrderDate`) en tant que document à l’aide du SDK .NET :

```csharp
    public class Order
    {
        [JsonProperty(PropertyName="id")]
        public string Id { get; set; }
        public DateTime OrderDate { get; set; }
        public DateTime ShipDate { get; set; }
        public double Total { get; set; }
    }

    await container.CreateItemAsync(
        new Order
        {
            Id = "09152014101",
            OrderDate = DateTime.UtcNow.AddDays(-30),
            ShipDate = DateTime.UtcNow.AddDays(-14),
            Total = 113.39
        });
```

Ce document est stocké dans Azure Cosmos DB comme suit :

```json
    {
        "id": "09152014101",
        "OrderDate": "2014-09-15T23:14:25.7251173Z",
        "ShipDate": "2014-09-30T23:14:25.7251173Z",
        "Total": 113.39
    }
```  

Vous pouvez également stocker les valeurs DateTime comme horodateurs Unix, autrement dit, comme un nombre représentant le nombre de secondes écoulées depuis le 1er janvier 1970. La propriété Timestamp interne d’Azure Cosmos DB (`_ts`) suit cette approche. Vous pouvez utiliser la classe [UnixDateTimeConverter](/dotnet/api/microsoft.azure.documents.unixdatetimeconverter) pour sérialiser les valeurs DateTime en tant que nombres.

## <a name="querying-datetimes-in-linq"></a>Interrogation des valeurs DateTime dans LINQ

Le SDK .NET SQL prend automatiquement en charge l’interrogation des données stockées dans Azure Cosmos DB par le biais de LINQ. Par exemple, l’extrait de code suivant montre une requête LINQ qui filtre les commandes qui ont été expédiées au cours des trois derniers jours :

```csharp
    IQueryable<Order> orders = container.GetItemLinqQueryable<Order>(allowSynchronousQueryExecution: true).Where(o => o.ShipDate >= DateTime.UtcNow.AddDays(-3));
```

Traduite par l’instruction SQL suivante et exécutée dans Azure Cosmos DB :

```sql
    SELECT * FROM root WHERE (root["ShipDate"] >= "2014-09-30T23:14:25.7251173Z")
```

Pour plus d’informations sur le langage de requête SQL d’Azure Cosmos DB et le fournisseur LINQ, consultez [Interrogation de Cosmos DB en LINQ](sql-query-linq-to-sql.md).

## <a name="indexing-datetimes-for-range-queries"></a>Indexation des valeurs DateTime pour les requêtes de plage

Les requêtes ont souvent des valeurs DateTime. Pour exécuter ces requêtes efficacement, vous devez disposer d’un index défini sur l’une des propriétés de filtre de la requête.

Pour plus d’informations sur la configuration des stratégies d’indexation, consultez [Stratégies d’indexation d’Azure Cosmos DB](index-policy.md). 

## <a name="next-steps"></a>Étapes suivantes

* Télécharger et exécuter les [exemples de code sur GitHub](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples/code-samples)
* En savoir plus sur les [requêtes SQL](sql-query-getting-started.md)
* En savoir plus sur les [stratégies d’indexation Azure Cosmos DB](index-policy.md)