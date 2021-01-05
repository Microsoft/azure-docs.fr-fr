---
title: Écrire des procédures stockées et des déclencheurs à l’aide de l’API de requête JavaScript dans Azure Cosmos DB
description: Découvrez comment écrire des procédures stockées et des déclencheurs à l’aide de l’API de requête JavaScript dans Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/07/2020
ms.author: tisande
ms.custom: devx-track-js
ms.openlocfilehash: 449af63ef3409b8c8173edef7a5d77074cca02b9
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93335838"
---
# <a name="how-to-write-stored-procedures-and-triggers-in-azure-cosmos-db-by-using-the-javascript-query-api"></a>Comment écrire des procédures stockées et des déclencheurs dans Azure Cosmos DB à l’aide de l’API de requête JavaScript
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB vous permet d’envoyer des requêtes optimisées à l’aide d’une interface JavaScript fluide, sans avoir besoin de connaissances particulières en matière de langage SQL. Vous pouvez utiliser cette interface pour écrire des procédures stockées ou des déclencheurs. Pour en savoir plus sur la prise en charge de l’API de requête JavaScript dans Azure Cosmos DB, consultez l’article [Working with JavaScript language-integrated query API with Azure Cosmos DB](javascript-query-api.md) (Utilisation de l’API de requête avec langage intégré JavaScript avec Azure Cosmos DB).

## <a name="stored-procedure-using-the-javascript-query-api"></a><a id="stored-procedures"></a>Procédure stockée utilisant l’API de requête JavaScript

L’exemple de code suivant illustre comment l’API de requête JavaScript est utilisée dans le contexte d’une procédure stockée. La procédure stockée insère un élément Azure Cosmos, spécifié par un paramètre d’entrée, et met à jour un document de métadonnées à l’aide de la méthode `__.filter()`, avec minSize, maxSize et totalSize basés sur la propriété de taille de l’élément d’entrée.

> [!NOTE]
> `__` (trait de soulignement double) est un alias pour `getContext().getCollection()` lors de l’utilisation de l’API de requête JavaScript.

```javascript
/**
 * Insert an item and update metadata doc: minSize, maxSize, totalSize based on item.size.
 */
function insertDocumentAndUpdateMetadata(item) {
  // HTTP error codes sent to our callback function by CosmosDB server.
  var ErrorCode = {
    RETRY_WITH: 449,
  }

  var isAccepted = __.createDocument(__.getSelfLink(), item, {}, function(err, item, options) {
    if (err) throw err;

    // Check the item (ignore items with invalid/zero size and metadata itself) and call updateMetadata.
    if (!item.isMetadata && item.size > 0) {
      // Get the metadata. We keep it in the same container. it's the only item that has .isMetadata = true.
      var result = __.filter(function(x) {
        return x.isMetadata === true
      }, function(err, feed, options) {
        if (err) throw err;

        // We assume that metadata item was pre-created and must exist when this script is called.
        if (!feed || !feed.length) throw new Error("Failed to find the metadata item.");

        // The metadata item.
        var metaItem = feed[0];

        // Update metaDoc.minSize:
        // for 1st document use doc.Size, for all the rest see if it's less than last min.
        if (metaItem.minSize == 0) metaItem.minSize = item.size;
        else metaItem.minSize = Math.min(metaItem.minSize, item.size);

        // Update metaItem.maxSize.
        metaItem.maxSize = Math.max(metaItem.maxSize, item.size);

        // Update metaItem.totalSize.
        metaItem.totalSize += item.size;

        // Update/replace the metadata item in the store.
        var isAccepted = __.replaceDocument(metaItem._self, metaItem, function(err) {
          if (err) throw err;
          // Note: in case concurrent updates causes conflict with ErrorCode.RETRY_WITH, we can't read the meta again
          //       and update again because due to Snapshot isolation we will read same exact version (we are in same transaction).
          //       We have to take care of that on the client side.
        });
        if (!isAccepted) throw new Error("replaceDocument(metaItem) returned false.");
      });
      if (!result.isAccepted) throw new Error("filter for metaItem returned false.");
    }
  });
  if (!isAccepted) throw new Error("createDocument(actual item) returned false.");
}
```

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour en savoir plus sur les procédures stockées, les déclencheurs et fonctions définies par l’utilisateur dans Azure Cosmos DB :

* [How to register and use stored procedures, triggers, and user-defined functions in Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md) (Comment inscrire et utiliser des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur dans Azure Cosmos DB)

* [How to run stored procedures](how-to-use-stored-procedures-triggers-udfs.md#stored-procedures) (Comment exécuter des procédures stockées)

* Comment inscrire et utiliser des [prédéclencheurs](how-to-use-stored-procedures-triggers-udfs.md#pre-triggers) et des [post-déclencheurs](how-to-use-stored-procedures-triggers-udfs.md#post-triggers) dans Azure Cosmos DB

* [How to work with user-defined functions](how-to-use-stored-procedures-triggers-udfs.md#udfs) (Comment utiliser des fonctions définies par l’utilisateur)

* [Create a synthetic partition key](synthetic-partition-keys.md) (Créer une clé de partition synthétique)
