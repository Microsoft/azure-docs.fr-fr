---
title: Requêtes paramétrables dans Azure Cosmos DB
description: Découvrez dans quelle mesure les requêtes SQL paramétrées fournissent une gestion et un échappement robustes de l’entrée utilisateur et empêchent l’exposition accidentelle des données via l’injection SQL.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 07/29/2020
ms.author: tisande
ms.openlocfilehash: dc32aab89e50b500001fd2267f62e3031154be62
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96549155"
---
# <a name="parameterized-queries-in-azure-cosmos-db"></a>Requêtes paramétrables dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB prend en charge les requêtes avec des paramètres exprimées par la notation @ classique. SQL paramétré fournit une gestion et un échappement robustes de l’entrée utilisateur et empêche l’exposition accidentelle des données via l’injection SQL.

## <a name="examples"></a>Exemples

Par exemple, vous pouvez écrire une requête qui accepte `lastName` et `address.state` comme paramètres, et l’exécuter pour différentes valeurs de `lastName` et `address.state` basées sur l’entrée d’utilisateur.

```sql
    SELECT *
    FROM Families f
    WHERE f.lastName = @lastName AND f.address.state = @addressState
```

Vous pouvez ensuite envoyer cette demande à Azure Cosmos DB sous la forme d’une requête JSON paramétrable comme suit :

```sql
    {
        "query": "SELECT * FROM Families f WHERE f.lastName = @lastName AND f.address.state = @addressState",
        "parameters": [
            {"name": "@lastName", "value": "Wakefield"},
            {"name": "@addressState", "value": "NY"},
        ]
    }
```

L’exemple suivant définit l’argument TOP avec une requête paramétrable :

```sql
    {
        "query": "SELECT TOP @n * FROM Families",
        "parameters": [
            {"name": "@n", "value": 10},
        ]
    }
```

Les valeurs de paramètres peuvent être n’importe quel format JSON valide : chaînes, nombres, booléens, Null, même des tableaux ou des valeurs JSON imbriquées. Azure Cosmos DB étant sans schéma, les paramètres ne sont pas validés par rapport à un type.

Voici des exemples pour des requêtes paramétrables dans chaque kit de développement logiciel (SDK) Azure Cosmos DB :

- [Kit de développement logiciel (SDK) .NET](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/Queries/Program.cs#L195)
- [Java](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L392-L421)
- [Node.JS](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ItemManagement.ts#L58-L79)
- [Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/cosmos/azure-cosmos/samples/document_management.py#L66-L78)

## <a name="next-steps"></a>Étapes suivantes

- [Exemples .NET Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Modéliser des données de document](modeling-data.md)
