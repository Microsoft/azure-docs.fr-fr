---
title: Comment interroger des données de table dans Azure Cosmos DB ?
description: Découvrez comment interroger des données stockées dans le compte d’API Table Azure Cosmos DB à l’aide de filtres OData et de requêtes LINQ
author: sakash279
ms.author: akshanka
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: tutorial
ms.date: 06/05/2020
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: e184d85e3daee41f530334aa0034fc98f40a8766
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93099217"
---
# <a name="tutorial-query-azure-cosmos-db-by-using-the-table-api"></a>Tutoriel : Interroger Azure Cosmos DB à l’aide de l’API Table
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

[L’API Table](table-introduction.md) d’Azure Cosmos DB prend en charge les requêtes OData et [LINQ](/rest/api/storageservices/fileservices/writing-linq-queries-against-the-table-service) effectuées sur des données de clé/valeur (table).  

Cet article décrit les tâches suivantes :

> [!div class="checklist"]
> * Interrogation des données avec l’API de table

L’exemple de table `People` suivant est utilisé pour les interrogations de cet article :

| PartitionKey | RowKey | E-mail | PhoneNumber |
| --- | --- | --- | --- |
| Harp | Walter | Walter@contoso.com| 425-555-0101 |
| Smith | Ben | Ben@contoso.com| 425-555-0102 |
| Smith | Jeff | Jeff@contoso.com| 425-555-0104 |

Consultez [Interrogation des tables et des entités](/rest/api/storageservices/fileservices/querying-tables-and-entities) pour plus d’informations sur la manière d’effectuer des requêtes à l’aide de l’API Table.

Pour plus d’informations sur les fonctionnalités étendues offertes par Azure Cosmos DB, consultez [Azure Cosmos DB : API Table](table-introduction.md) et [Développer avec l’API Table en utilisant .NET](tutorial-develop-table-dotnet.md).

## <a name="prerequisites"></a>Prérequis

Pour le bon fonctionnement de ces requêtes, vous devez disposer d’un compte Azure Cosmos DB et de données d’entité dans le conteneur. Cela n’est pas le cas ? Lancez le [démarrage rapide en 5 minutes](create-table-dotnet.md) ou le [didacticiel destiné aux développeurs](tutorial-develop-table-dotnet.md) pour créer un compte et alimenter votre base de données.

## <a name="query-on-partitionkey-and-rowkey"></a>Interroger sur PartitionKey et RowKey

Les propriétés PartitionKey et RowKey formant une clé primaire d’une entité, vous pouvez utiliser la syntaxe spéciale suivante pour identifier l’entité :

**Requête**

```
https://<mytableendpoint>/People(PartitionKey='Harp',RowKey='Walter')  
```

**Résultats**

| PartitionKey | RowKey | E-mail | PhoneNumber |
| --- | --- | --- | --- |
| Harp | Walter | Walter@contoso.com| 425-555-0104 |

Vous pouvez également spécifier ces propriétés dans le cadre de l’option `$filter`, comme indiqué dans la section suivante. Notez que les noms de propriété de clé et les valeurs de constante respectent la casse. Les propriétés PartitionKey et RowKey sont de type chaîne.

## <a name="query-by-using-an-odata-filter"></a>Interroger en utilisant un filtre OData

Quand vous construisez une chaîne de filtrage, prenez en compte les règles suivantes :

* Utilisez les opérateurs logiques définis par la spécification du protocole OData Protocol pour comparer une propriété par rapport à une valeur. Notez que vous ne pouvez pas comparer une propriété à une valeur dynamique. Un côté de l’expression doit être une constante.
* Le nom de la propriété, l’opérateur et la valeur de constante doivent être séparés par des espaces codés URL. Un espace est codé URL sous la forme `%20`.
* Toutes les parties de la chaîne de filtrage respectent la casse.
* Pour que le filtre retourne des résultats valides, la valeur constante doit être du même type de données que la propriété. Pour plus d’informations sur les types de propriétés pris en charge, consultez [Présentation du modèle de données du service de Table](/rest/api/storageservices/understanding-the-table-service-data-model).

Voici un exemple de requête qui montre comment filtrer sur les propriétés PartitionKey et Email en utilisant un `$filter` OData.

**Requête**

```
https://<mytableapi-endpoint>/People()?$filter=PartitionKey%20eq%20'Smith'%20and%20Email%20eq%20'Ben@contoso.com'
```

Pour plus d’informations sur la construction d’expressions de filtre pour différents types de données, consultez [Querying Tables and Entities](/rest/api/storageservices/querying-tables-and-entities).

**Résultats**

| PartitionKey | RowKey | E-mail | PhoneNumber |
| --- | --- | --- | --- |
| Smith |Ben | Ben@contoso.com| 425-555-0102 |

Les requêtes sur les propriétés DateHeure ne retournent pas de données quand elles sont exécutées dans l’API Table d’Azure Cosmos DB. Même si le stockage Table Azure stocke les valeurs de date avec une précision temporelle de cycles, l’API Table d’Azure Cosmos DB utilise la propriété `_ts`. La propriété `_ts` se situe à un deuxième niveau de précision, ce qui ne correspond pas à un filtre OData. Par conséquent, les requêtes sur les propriétés timestamp sont bloquées par Azure Cosmos DB. Pour contourner ce problème, vous pouvez définir une propriété de type de données DateHeure ou Long personnalisée et définir la valeur de date à partir du client.

## <a name="query-by-using-linq"></a>Interroger en utilisant LINQ 
Vous pouvez également interroger en utilisant LINQ, qui est traduit en expressions de requête OData correspondantes. Voici un exemple montrant comment créer des requêtes en utilisant le SDK .NET :

```csharp
IQueryable<CustomerEntity> linqQuery = table.CreateQuery<CustomerEntity>()
            .Where(x => x.PartitionKey == "4")
            .Select(x => new CustomerEntity() { PartitionKey = x.PartitionKey, RowKey = x.RowKey, Email = x.Email });
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez :

> [!div class="checklist"]
> * Appris à interroger des données avec l’API Table

Vous pouvez maintenant poursuivre avec le didacticiel suivant montrant comment distribuer vos données globalement.

> [!div class="nextstepaction"]
> [Distribuer vos données globalement](tutorial-global-distribution-table.md)
