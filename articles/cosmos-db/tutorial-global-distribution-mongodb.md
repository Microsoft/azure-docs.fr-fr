---
title: Tutoriel pour configurer la distribution mondiale avec l’API Azure Cosmos DB pour MongoDB
description: Découvrez comment configurer la distribution mondiale à l’aide de l’API Azure Cosmos DB pour MongoDB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: tutorial
ms.date: 12/26/2018
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: 89826eab7b1686ae695a2716a03b2f5d03da277f
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93099248"
---
# <a name="set-up-global-distributed-database-using-azure-cosmos-dbs-api-for-mongodb"></a>Configurer une base de données distribuée à l’échelle mondiale à l’aide de l’API Azure Cosmos DB pour MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Dans cet article, nous vous montrons comment utiliser le portail Azure pour configurer une base de données distribuée à l’échelle mondiale et vous y connecter à l’aide de l’API Azure Cosmos DB pour MongoDB.

Cet article décrit les tâches suivantes : 

> [!div class="checklist"]
> * Configurer la diffusion mondiale à l’aide du portail Azure
> * Configurer la distribution mondiale à l’aide de l’[API Azure Cosmos DB pour MongoDB](mongodb-introduction.md)

[!INCLUDE [cosmos-db-tutorial-global-distribution-portal](../../includes/cosmos-db-tutorial-global-distribution-portal.md)]

## <a name="verifying-your-regional-setup"></a>Vérification de votre configuration régionale 
La façon la plus simple de vérifier votre configuration mondiale avec l’API Cosmos DB pour MongoDB consiste à exécuter la commande *isMaster()* à partir de l’interpréteur de commandes Mongo.

À partir de votre interpréteur de commandes Mongo :

   ```
      db.isMaster()
   ```
   
Résultats de l'exemple :

   ```JSON
      {
         "_t": "IsMasterResponse",
         "ok": 1,
         "ismaster": true,
         "maxMessageSizeBytes": 4194304,
         "maxWriteBatchSize": 1000,
         "minWireVersion": 0,
         "maxWireVersion": 2,
         "tags": {
            "region": "South India"
         },
         "hosts": [
            "vishi-api-for-mongodb-southcentralus.documents.azure.com:10255",
            "vishi-api-for-mongodb-westeurope.documents.azure.com:10255",
            "vishi-api-for-mongodb-southindia.documents.azure.com:10255"
         ],
         "setName": "globaldb",
         "setVersion": 1,
         "primary": "vishi-api-for-mongodb-southindia.documents.azure.com:10255",
         "me": "vishi-api-for-mongodb-southindia.documents.azure.com:10255"
      }
   ```

## <a name="connecting-to-a-preferred-region"></a>Connexion à une région de prédilection 

L’API Azure Cosmos DB pour MongoDB vous permet de spécifier les préférences de lecture de votre collection pour une base de données mondialement distribuée. Pour les lectures à faible latence et la haute disponibilité globale, nous vous recommandons de définir les préférences de lecture de votre collection sur *La plus proche*. Une préférence de lecture définie sur *La plus proche* est configurée pour lire à partir de la région la plus proche.

```csharp
var collection = database.GetCollection<BsonDocument>(collectionName);
collection = collection.WithReadPreference(new ReadPreference(ReadPreferenceMode.Nearest));
```

Pour les applications avec une région primaire en lecture/écriture et une région secondaire pour les scénarios de récupération d’urgence, nous vous recommandons de définir les préférences de lecture de votre collection sur *Secondary preferred* (Secondaire par défaut). Une préférence de lecture définie sur *Secondary preferred* (Secondaire par défaut) est configurée pour lire à partir de la région secondaire lorsque la région primaire n’est pas disponible.

```csharp
var collection = database.GetCollection<BsonDocument>(collectionName);
collection = collection.WithReadPreference(new ReadPreference(ReadPreferenceMode.SecondaryPreferred));
```

Enfin, si vous souhaitez spécifier manuellement vos régions de lecture, vous pouvez définir la balise de la région dans vos préférences de lecture.

```csharp
var collection = database.GetCollection<BsonDocument>(collectionName);
var tag = new Tag("region", "Southeast Asia");
collection = collection.WithReadPreference(new ReadPreference(ReadPreferenceMode.Secondary, new[] { new TagSet(new[] { tag }) }));
```

C’est ici que s’achève ce didacticiel. Découvrez comment gérer la cohérence de votre compte répliqué à l’échelle mondiale en lisant l’article [Niveaux de cohérence dans Azure Cosmos DB](consistency-levels.md). Pour plus d’informations sur le fonctionnement de la réplication de base de données à l’échelle mondiale dans Azure Cosmos DB, voir [Diffuser des données à l’échelle mondiale avec Azure Cosmos DB](distribute-data-globally.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez :

> [!div class="checklist"]
> * Configurer la diffusion mondiale à l’aide du portail Azure
> * Configuré la distribution mondiale à l’aide de l’API de Cosmos DB pour MongoDB.

Vous pouvez maintenant passer au didacticiel suivant pour apprendre à développer en local à l’aide de l’émulateur local Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Développer localement avec l’émulateur Azure Cosmos DB](local-emulator.md)
