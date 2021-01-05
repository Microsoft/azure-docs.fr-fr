---
title: Limites d’Azure Cosmos DB Gremlin
description: Documentation de référence pour les limitations du runtime du moteur de graphe
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: reference
ms.date: 10/04/2019
ms.author: sngun
ms.openlocfilehash: 4e638fdff67ad2d0bc6f191cdfd46867ab847923
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93080105"
---
# <a name="azure-cosmos-db-gremlin-limits"></a>Limites d’Azure Cosmos DB Gremlin
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

Cet article aborde les limites du moteur Azure Cosmos DB Gremlin et explique comment elles peuvent avoir un impact sur les traversées des clients.

Cosmos DB Gremlin est basé sur l’infrastructure Cosmos DB. Pour cette raison, toutes les limites expliquées dans [Limites du service Azure Cosmos DB](./concepts-limits.md) continuent à s’appliquer.

## <a name="limits"></a>limites

Quand une limite Gremlin est atteinte, la traversée est annulée avec un code **x-ms-status-code** de 429, ce qui indique une erreur de limitation. Consultez [En-têtes de réponse du serveur Gremlin](gremlin-limits.md) pour plus d’informations.

**Ressource**    | **Limite par défaut** | **Explication**
--- | --- | ---
*Longueur du script* | **64 Ko** | Longueur maximale d’un script de traversée Gremlin par demande.
*Profondeur d’opérateurs* | **400** |  Nombre total d’étapes uniques dans une traversée. Par exemple, ```g.V().out()``` a un nombre d’opérateurs de 2 : V() et out(), ```g.V('label').repeat(out()).times(100)``` a une profondeur d’opérateurs de 3 : V(), repeat() et out(), car ```.times(100)``` est un paramètre pour l’opérateur ```.repeat()```.
*Degré de parallélisme* | **32** | Nombre maximal de partitions de stockage interrogées dans une demande unique au niveau de la couche de stockage. Les graphiques avec des centaines de partitions seront affectés par cette limite.
*Limite de répétition* | **32** | Nombre maximal d’itérations qu’un opérateur ```.repeat()``` peut exécuter. Dans la plupart des cas, chaque itération de l’étape ```.repeat()``` effectue une traversée en largeur, ce qui signifie que toute traversée est limitée au maximum à 32 tronçons entre des sommets.
*Délai d’expiration de la traversée* | **30 secondes** | La traversée est annulée quand il dépasse cette durée. Cosmos DB Graph est une base de données OLTP dont la grande majorité des traversées s’effectue en quelques millisecondes. Pour exécuter des requêtes OLAP sur Cosmos DB Graph, utilisez [Apache Spark](https://azure.microsoft.com/services/cosmos-db/) avec des [dataframes de graphe](https://spark.apache.org/docs/latest/sql-programming-guide.html#datasets-and-dataframes) et le [connecteur Spark pour Cosmos DB](https://github.com/Azure/azure-cosmosdb-spark).
*Délai d’expiration de connexion inactive* | **1 heure** | Durée pendant laquelle le service Gremlin garde ouvertes les connexions WebSocket inactives. Les paquets de persistance TCP et les demandes de persistance HTTP n’étendent pas la durée de connexion au-delà de cette limite. Le moteur Cosmos DB Graph considère les connexions WebSocket comme inactives si aucune demande Gremlin active n’y est en cours d’exécution.
*Jetons de ressource par heure* | **100** | Nombre de jetons de ressource uniques utilisés par les clients Gremlin pour se connecter au compte Gremlin dans une région. Lorsque l’application dépasse la limite de jetons uniques horaire, `"Exceeded allowed resource token limit of 100 that can be used concurrently"` est renvoyé lors de la prochaine demande d’authentification.

## <a name="next-steps"></a>Étapes suivantes
* [En-têtes de réponse d’Azure Cosmos DB Gremlin](gremlin-headers.md)
* [Jetons de ressource Azure Cosmos DB avec Gremlin](how-to-use-resource-tokens-gremlin.md)