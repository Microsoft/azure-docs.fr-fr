---
title: Bien démarrer avec les requêtes SQL dans Azure Cosmos DB
description: Découvrez comment utiliser des requêtes SQL pour interroger des données à partir d’Azure Cosmos DB. Vous pouvez charger des exemples de données dans un conteneur dans Azure Cosmos DB et les interroger.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 11/04/2020
ms.author: tisande
ms.openlocfilehash: c78e15a0656abd510bbc241d39ffd026afddfd96
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353661"
---
# <a name="getting-started-with-sql-queries"></a>Bien démarrer avec les requêtes SQL
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Dans les comptes d’API SQL Azure Cosmos DB, il existe deux façons de lire des données :

**Lectures de points** : vous pouvez effectuer une recherche de clé/valeur sur un *ID d’élément* unique et une clé de partition. La combinaison de la clé de partition et de l’*ID d’élément* représente la clé et l’élément proprement dit représente la valeur. Pour un document de 1 Ko, les lectures de points coûtent généralement 1 [unité de requête](request-units.md) avec une latence inférieure à 10 ms. Les lectures de points retournent un seul élément.

Voici quelques exemples montrant comment effectuer des **lectures de points** avec chaque SDK :

- [Kit de développement logiciel (SDK) .NET](/dotnet/api/microsoft.azure.cosmos.container.readitemasync?preserve-view=true&view=azure-dotnet)
- [Kit SDK Java](/java/api/com.azure.cosmos.cosmoscontainer.readitem?preserve-view=true&view=azure-java-stable#com_azure_cosmos_CosmosContainer__T_readItem_java_lang_String_com_azure_cosmos_models_PartitionKey_com_azure_cosmos_models_CosmosItemRequestOptions_java_lang_Class_T__)
- [Kit de développement logiciel (SDK) Node.js](/javascript/api/@azure/cosmos/item?preserve-view=true&view=azure-node-latest#read-requestoptions-)
- [Kit de développement logiciel (SDK) Python](/python/api/azure-cosmos/azure.cosmos.containerproxy?preserve-view=true&view=azure-python#read-item-item--partition-key--populate-query-metrics-none--post-trigger-include-none----kwargs-)

**Requêtes SQL** : vous pouvez interroger des données en écrivant des requêtes avec le langage SQL (Structured Query Language) comme langage de requête JSON. Les requêtes coûtent toujours au moins 2,3 unités de requête et, en général, ont une latence plus élevée et plus variable que les lectures de points. Les requêtes peuvent retourner de nombreux éléments.

La plupart des charges de travail de lecture intensive sur Azure Cosmos DB utilisent une combinaison de lectures de points et de requêtes SQL. Si vous avez simplement besoin de lire un seul élément, les lectures de points sont moins chères et plus rapides que les requêtes. Les lectures de points n’ont pas besoin d’utiliser le moteur d’interrogation pour accéder aux données qu’elles peuvent lire directement. Bien entendu, il n’est pas possible pour toutes les charges de travail de lire exclusivement les données à l’aide des lectures de points. Par conséquent, la prise en charge de SQL comme langage de requête et l’[indexation indépendante du schéma](index-overview.md) offrent un moyen plus souple d’accéder à vos données.

Voici quelques exemples montrant comment effectuer des **requêtes SQL** avec chaque SDK :

- [Kit de développement logiciel (SDK) .NET](./sql-api-dotnet-v3sdk-samples.md#query-examples)
- [Kit SDK Java](./sql-api-java-sdk-samples.md#query-examples)
- [Kit de développement logiciel (SDK) Node.js](./sql-api-nodejs-samples.md#item-examples)
- [Kit de développement logiciel (SDK) Python](./sql-api-python-samples.md#item-examples)

Le reste de cette documentation montre comment commencer à écrire des requêtes SQL dans Azure Cosmos DB. Les requêtes SQL peuvent être exécutées via le SDK ou le portail Azure.

## <a name="upload-sample-data"></a>Charger l’exemple de données

Dans votre compte Cosmos DB de l’API SQL, ouvrez [Explorateur de données](./data-explorer.md) pour créer un conteneur appelé `Families`. Une fois qu’il a été créé, utilisez le navigateur de structures de données pour le trouver et l’ouvrir. Dans votre conteneur `Families`, l’option `Items` se trouve juste en dessous du nom du conteneur. Ouvrez cette option. Un bouton s’affiche dans la barre de menus, au centre de l’écran, pour créer un « nouvel élément ». Vous allez utiliser cette fonctionnalité pour créer les éléments JSON ci-dessous.

### <a name="create-json-items"></a>Créer des éléments JSON

Les deux éléments JSON suivants sont des documents concernant les familles Andersen et Wakefield. Ils comprennent les parents, les enfants, ainsi que leurs animaux, leur adresse et leurs informations d’inscription. 

Le premier élément se compose de chaînes, de nombres, de valeurs booléennes, de tableaux et de propriétés imbriquées :

```json
{
  "id": "AndersenFamily",
  "lastName": "Andersen",
  "parents": [
     { "firstName": "Thomas" },
     { "firstName": "Mary Kay"}
  ],
  "children": [
     {
         "firstName": "Henriette Thaulow",
         "gender": "female",
         "grade": 5,
         "pets": [{ "givenName": "Fluffy" }]
     }
  ],
  "address": { "state": "WA", "county": "King", "city": "Seattle" },
  "creationDate": 1431620472,
  "isRegistered": true
}
```

Le deuxième élément utilise `givenName` et `familyName` au lieu de `firstName` et `lastName` :

```json
{
  "id": "WakefieldFamily",
  "parents": [
      { "familyName": "Wakefield", "givenName": "Robin" },
      { "familyName": "Miller", "givenName": "Ben" }
  ],
  "children": [
      {
        "familyName": "Merriam",
        "givenName": "Jesse",
        "gender": "female",
        "grade": 1,
        "pets": [
            { "givenName": "Goofy" },
            { "givenName": "Shadow" }
        ]
      },
      {
        "familyName": "Miller",
         "givenName": "Lisa",
         "gender": "female",
         "grade": 8 }
  ],
  "address": { "state": "NY", "county": "Manhattan", "city": "NY" },
  "creationDate": 1431620462,
  "isRegistered": false
}
```

### <a name="query-the-json-items"></a>Interroger les éléments JSON

Appliquez quelques requêtes sur les données JSON pour comprendre certains aspects clés du langage de requête SQL d’Azure Cosmos DB.

La requête suivante retourne les éléments dont le champ `id` correspond à `AndersenFamily`. Comme il s’agit d’une requête `SELECT *`, son résultat est l’élément JSON complet. Pour plus d’informations sur la syntaxe SELECT, consultez [Instruction SELECT](sql-query-select.md).

```sql
    SELECT *
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Voici le résultat de la requête :

```json
    [{
        "id": "AndersenFamily",
        "lastName": "Andersen",
        "parents": [
           { "firstName": "Thomas" },
           { "firstName": "Mary Kay"}
        ],
        "children": [
           {
               "firstName": "Henriette Thaulow", "gender": "female", "grade": 5,
               "pets": [{ "givenName": "Fluffy" }]
           }
        ],
        "address": { "state": "WA", "county": "King", "city": "Seattle" },
        "creationDate": 1431620472,
        "isRegistered": true
    }]
```

La requête suivante remet en forme la sortie JSON. La requête projette un nouvel objet JSON `Family` avec deux champs sélectionnés, `Name` et `City`, où la ville de l’adresse est identique à l’État. « NY, NY » correspond à ce cas.

```sql
    SELECT {"Name":f.id, "City":f.address.city} AS Family
    FROM Families f
    WHERE f.address.city = f.address.state
```

Voici le résultat de la requête :

```json
    [{
        "Family": {
            "Name": "WakefieldFamily",
            "City": "NY"
        }
    }]
```

La requête suivante retourne tous les noms donnés des enfants de la famille dont l’`id` correspond à `WakefieldFamily`, classés par ville.

```sql
    SELECT c.givenName
    FROM Families f
    JOIN c IN f.children
    WHERE f.id = 'WakefieldFamily'
    ORDER BY f.address.city ASC
```

Les résultats sont :

```json
    [
      { "givenName": "Jesse" },
      { "givenName": "Lisa"}
    ]
```

## <a name="remarks"></a>Notes

Les exemples précédents illustrent plusieurs aspects du langage de requête Cosmos DB :  

* Comme l’API SQL fonctionne avec les valeurs JSON, il traite des entités d’arborescence au lieu des lignes et des colonnes. Vous pouvez faire référence aux nœuds de l’arborescence quel que soit le niveau auquel ils se trouvent, comme `Node1.Node2.Node3…..Nodem`, à l’image de la référence en deux parties de `<table>.<column>` dans SQL ANSI.

* Étant donné que le langage de requête fonctionne avec des données sans schéma, le système de type doit être lié dynamiquement. La même expression peut produire différents types sur différents éléments. Le résultat d’une requête est une valeur JSON valide, mais n’est pas forcément un schéma fixe.  

* Azure Cosmos DB prend uniquement en charge les éléments JSON stricts. Le système de type et les expressions peuvent uniquement traiter des types JSON. Pour en savoir plus, consultez la [spécification JSON](https://www.json.org/).  

* Un conteneur Cosmos est une collection sans schéma d’éléments JSON. Les relations dans et entre les éléments d’un conteneur sont capturées de façon implicite par l’autonomie, et non par les relations de clé primaire et de clé étrangère. Cette fonctionnalité est importante pour les jointures intra-éléments décrites plus loin dans cet article.

## <a name="next-steps"></a>Étapes suivantes

- [Présentation d’Azure Cosmos DB](introduction.md)
- [Exemples .NET Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Clause SELECT](sql-query-select.md)