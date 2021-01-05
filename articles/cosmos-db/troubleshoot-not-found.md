---
title: Résoudre les problèmes liés à des exceptions introuvables Azure Cosmos DB
description: Découvrez comment diagnostiquer et résoudre des exceptions introuvables.
author: j82w
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.date: 07/13/2020
ms.author: jawilley
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: 2df401f7871d631ba317fb670783cad086b9a351
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "96017557"
---
# <a name="diagnose-and-troubleshoot-azure-cosmos-db-not-found-exceptions"></a>Diagnostiquer et résoudre les problèmes liés à des exceptions introuvables Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Le code d’état HTTP 404 indique que la ressource n’existe plus.

## <a name="expected-behavior"></a>Comportement attendu
Il existe de nombreux scénarios valides où une application attend un code 404 et gère correctement le scénario.

## <a name="a-not-found-exception-was-returned-for-an-item-that-should-exist-or-does-exist"></a>Une exception introuvable a été retournée pour un élément qui doit exister ou existe
Voici les raisons possibles pour qu’un code d’état 404 soit retourné si l’élément doit exister ou existe.

### <a name="the-read-session-is-not-available-for-the-input-session-token"></a>The read session is not available for the input session token (La session de lecture n’est pas disponible pour le jeton de session d’entrée)

#### <a name="solution"></a>Solution :
1. Mettez à jour votre Kit de développement logiciel (SDK) actuel vers la dernière version disponible. Les causes les plus courantes de cette erreur particulière ont été corrigées dans les versions les plus récentes du SDK.

### <a name="race-condition"></a>Condition de concurrence
Il existe plusieurs instances de client SDK et la lecture a eu lieu avant l’écriture.

#### <a name="solution"></a>Solution :
1. La cohérence de compte par défaut pour Azure Cosmos DB est la cohérence de session. Lorsqu’un élément est créé ou mis à jour, la réponse retourne un jeton de session qui peut être transmis entre les instances de SDK pour garantir que la demande de lecture lit à partir d’un réplica avec cette modification.
1. Affectez au [niveau de cohérence](./consistency-levels.md) un [niveau plus fort](./consistency-levels.md).

### <a name="invalid-partition-key-and-id-combination"></a>Combinaison de la clé de partition et de l’ID non valide
La combinaison de la clé de partition et de l’ID n’est pas valide.

#### <a name="solution"></a>Solution :
Corrigez la logique d’application qui provoque la combinaison incorrecte. 

### <a name="invalid-character-in-an-item-id"></a>Caractère non valide dans un ID d’élément
Un élément est inséré dans Azure Cosmos DB avec un [caractère non valide](/dotnet/api/microsoft.azure.documents.resource.id?preserve-view=true&view=azure-dotnet#remarks) dans l’ID d’élément.

#### <a name="solution"></a>Solution :
Remplacez l’ID par une valeur différente qui ne contient pas les caractères spéciaux. S’il n’est pas possible de modifier l’ID, vous pouvez l’encoder au format Base64 pour placer les caractères spéciaux dans une séquence d’échappement. Base64 peut toujours produire un nom avec un caractère non valide « / » qui doit être remplacé.

Pour les éléments déjà insérés dans le conteneur, l’ID peut être remplacé avec des valeurs RID plutôt que des références basées sur un nom.
```c#
// Get a container reference that uses RID values.
ContainerProperties containerProperties = await this.Container.ReadContainerAsync();
string[] selfLinkSegments = containerProperties.SelfLink.Split('/');
string databaseRid = selfLinkSegments[1];
string containerRid = selfLinkSegments[3];
Container containerByRid = this.cosmosClient.GetContainer(databaseRid, containerRid);

// Invalid characters are listed here.
//https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id?view=azure-dotnet&preserve-view=true#remarks
FeedIterator<JObject> invalidItemsIterator = this.Container.GetItemQueryIterator<JObject>(
    @"select * from t where CONTAINS(t.id, ""/"") or CONTAINS(t.id, ""#"") or CONTAINS(t.id, ""?"") or CONTAINS(t.id, ""\\"") ");
while (invalidItemsIterator.HasMoreResults)
{
    foreach (JObject itemWithInvalidId in await invalidItemsIterator.ReadNextAsync())
    {
        // Choose a new ID that doesn't contain special characters.
        // If that isn't possible, then Base64 encode the ID to escape the special characters.
        byte[] plainTextBytes = Encoding.UTF8.GetBytes(itemWithInvalidId["id"].ToString());
        itemWithInvalidId["id"] = Convert.ToBase64String(plainTextBytes).Replace('/', '!');

        // Update the item with the new ID value by using the RID-based container reference.
        JObject item = await containerByRid.ReplaceItemAsync<JObject>(
            item: itemWithInvalidId,
            ID: itemWithInvalidId["_rid"].ToString(),
            partitionKey: new Cosmos.PartitionKey(itemWithInvalidId["status"].ToString()));

        // Validating the new ID can be read by using the original name-based container reference.
        await this.Container.ReadItemAsync<ToDoActivity>(
            item["id"].ToString(),
            new Cosmos.PartitionKey(item["status"].ToString())); ;
    }
}
```

### <a name="time-to-live-purge"></a>Suppression définitive de la durée de vie
La propriété [Durée de vie](./time-to-live.md) a été définie pour l’élément. L’élément a été supprimé définitivement, car la propriété Durée de vie a expiré.

#### <a name="solution"></a>Solution :
Modifiez la propriété Durée de vie pour empêcher la suppression définitive de l’élément.

### <a name="lazy-indexing"></a>Indexation différée
L’[indexation différée](index-policy.md#indexing-mode) n’est pas à jour.

#### <a name="solution"></a>Solution :
Attendez que l’indexation soit à jour ou modifiez la stratégie d’indexation.

### <a name="parent-resource-deleted"></a>Ressource parente supprimée
La base de données ou le conteneur dans lequel se trouve l’élément a été supprimé.

#### <a name="solution"></a>Solution :
1. [Restaurez](./online-backup-and-restore.md#request-data-restore-from-a-backup) la ressource parente ou recréez les ressources.
1. Créez une ressource pour remplacer la ressource supprimée.

### <a name="7-containercollection-names-are-case-sensitive"></a>7. Les noms de conteneur/collection sont sensibles à la casse
Les noms de conteneur/collection respectent la casse dans Cosmos DB.

#### <a name="solution"></a>Solution :
Veillez à utiliser le nom exact lors de la connexion à Cosmos DB.

## <a name="next-steps"></a>Étapes suivantes
* [Diagnostiquer et résoudre](troubleshoot-dot-net-sdk.md) des problèmes lors de l’utilisation du kit de développement logiciel (SDK) .NET Azure Cosmos DB.
* Découvrez les recommandations relatives aux performances pour [.NET V3](performance-tips-dotnet-sdk-v3-sql.md) et [.NET V2](performance-tips.md).
* [Diagnostiquer et résoudre](troubleshoot-java-sdk-v4-sql.md) des problèmes lors de l'utilisation du SDK Java v4 Azure Cosmos DB.
* Découvrez les recommandations relatives aux performances pour le [SDK Java v4](performance-tips-java-sdk-v4-sql.md).
