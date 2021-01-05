---
title: Interroger des bases de données partitionnées
description: Exécutez des requêtes en utilisant la bibliothèque cliente des bases de données élastiques.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.topic: how-to
ms.custom: sqldbrb=1
author: stevestein
ms.author: sstein
ms.date: 01/25/2019
ms.openlocfilehash: 5a0dd12efb9d94bda264b3bd04b05cdc3df917e5
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92786630"
---
# <a name="multi-shard-querying-using-elastic-database-tools"></a>Requête sur plusieurs partitions à l’aide d’outils de base de données élastique
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

## <a name="overview"></a>Vue d’ensemble

Avec les [outils des bases de données élastiques](elastic-scale-introduction.md), vous pouvez créer des solutions de base de données partitionnée. **Requête sur plusieurs partitions** est utilisée pour des tâches telles que la collecte de données / la création de rapports qui nécessitent l'exécution d'une requête qui s'étend sur plusieurs partitions. (Comparez cela au [routage dépendant des données](elastic-scale-data-dependent-routing.md)qui effectue tout le travail sur une partition unique.)

1. Obtenez un **RangeShardMap** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) ou **ListShardMap** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.map.listshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.listshardmap-1)) à l’aide de la méthode **TryGetRangeShardMap** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.trygetrangeshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetrangeshardmap)), **TryGetListShardMap** ( [ Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.trygetlistshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetlistshardmap)), ou **GetShardMap** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.getshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.getshardmap)). Consultez les rubriques [Construction d’un objet ShardMapManager](elastic-scale-shard-map-management.md#constructing-a-shardmapmanager) et [Obtenir un objet RangeShardMap ou ListShardMap](elastic-scale-shard-map-management.md#get-a-rangeshardmap-or-listshardmap).
2. Créez un objet **MultiShardConnection** ( [Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardconnection), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardconnection)).
3. Créez **MultiShardStatement ou MultiShardCommand** ( [Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardstatement), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand)).
4. Définissez la **propriété CommandText** ( [Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardstatement), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand)) sur une commande T-SQL.
5. Exécutez la commande en appelant la méthode **ExecuteQueryAsync ou ExecuteReader** ( [Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardstatement.executeQueryAsync), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand)).
6. Affichez les résultats à l’aide de la classe **MultiShardResultSet ou MultiShardDataReader** ( [Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardresultset), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multisharddatareader)).

## <a name="example"></a>Exemple

Le code suivant illustre l'utilisation de la requête sur plusieurs partitions à l'aide d'une **ShardMap** donnée nommée *myShardMap*.

```csharp
using (MultiShardConnection conn = new MultiShardConnection(myShardMap.GetShards(), myShardConnectionString))
{
    using (MultiShardCommand cmd = conn.CreateCommand())
    {
        cmd.CommandText = "SELECT c1, c2, c3 FROM ShardedTable";
        cmd.CommandType = CommandType.Text;
        cmd.ExecutionOptions = MultiShardExecutionOptions.IncludeShardNameColumn;
        cmd.ExecutionPolicy = MultiShardExecutionPolicy.PartialResults;

        using (MultiShardDataReader sdr = cmd.ExecuteReader())
        {
            while (sdr.Read())
            {
                var c1Field = sdr.GetString(0);
                var c2Field = sdr.GetFieldValue<int>(1);
                var c3Field = sdr.GetFieldValue<Int64>(2);
            }
        }
    }
}
```

La principale différence est la construction de connexions à plusieurs partitions. Lorsque **SqlConnection** fonctionne sur une base de données individuelle, l’objet **MultiShardConnection** utilise une **_collection de partitions_ *_ comme entrée. Remplissez la collection de partitions à partir d'une carte de partitions. La requête est ensuite exécutée sur la collection de partitions à l'aide de la sémantique _* UNION ALL** pour assembler un seul résultat global. Le nom de la partition d'où provient la ligne peut éventuellement être ajouté à la sortie à l'aide de la propriété sur la commande **ExecutionOptions** .

Notez l'appel à **myShardMap.GetShards()** . Cette méthode récupère toutes les partitions de la carte de partitions et offre un moyen facile d’exécuter une requête sur toutes les bases de données concernées. La collection de partitions pour une requête sur plusieurs partitions peut être affinée davantage en effectuant une requête LINQ sur la collection retournée par l'appel à **myShardMap.GetShards()** . En combinaison avec la stratégie de résultats partiels, la capacité actuelle de l'interrogation de plusieurs partitions a été conçue pour fonctionner correctement avec des dizaines, voire des centaines, de partitions.

Une limitation de l'interrogation de plusieurs partitions est actuellement le manque de validation des partitions et shardlets interrogés. Tandis que le routage dépendant des données vérifie qu'une partition donnée fait partie de la carte de partitions au moment de l'interrogation, les requêtes sur plusieurs partitions n'effectuent pas cette vérification. De ce fait, les requêtes sur plusieurs partitions peuvent s’exécuter sur des bases de données qui ont été supprimées de la carte de partitions.

## <a name="multi-shard-queries-and-split-merge-operations"></a>Requêtes sur plusieurs partitions et opérations de fractionnement et de fusion

Les requêtes sur plusieurs partitions ne vérifient pas si les shardlets de la base de données interrogée participent à des opérations de fractionnement et de fusion en cours. (Consultez [Mise à l’échelle utilisant l’outil de fractionnement et de fusion de bases de données élastiques](elastic-scale-overview-split-and-merge.md).) Cela peut entraîner des incohérences avec des lignes du même shardlet qui s’affichent pour plusieurs bases de données dans la même requête sur plusieurs partitions. Tenez compte de ces limitations et envisagez de purger les opérations de fractionnement et de fusion en cours et les modifications apportées à la carte de partitions lors de l’exécution de requêtes sur plusieurs partitions.

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]