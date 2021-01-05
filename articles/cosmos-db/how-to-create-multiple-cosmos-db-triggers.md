---
title: Créer plusieurs déclencheurs Azure Functions indépendants pour Cosmos DB
description: Découvrez comment configurer plusieurs déclencheurs Azure Functions indépendants pour Cosmos DB afin de créer des architectures basées sur les événements.
author: ealsur
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 07/17/2019
ms.author: maquaran
ms.custom: devx-track-csharp
ms.openlocfilehash: 78fff48a97965f0b80456cd3e56ed1507bc784fc
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93336671"
---
# <a name="create-multiple-azure-functions-triggers-for-cosmos-db"></a>Créer plusieurs déclencheurs Azure Functions pour Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article explique comment configurer plusieurs déclencheurs Azure Functions pour Cosmos DB fonctionnant en parallèle et réagissant de façon indépendante aux modifications.

:::image type="content" source="./media/change-feed-functions/multi-trigger.png" alt-text="Fonctions basées sur les événements serverless fonctionnant avec le déclencheur Azure Functions pour Cosmos DB et partageant un conteneur de baux" border="false":::

## <a name="event-based-architecture-requirements"></a>Exigences de l’architecture basée sur les événements

Quand il s’agit de créer des architectures serverless avec [Azure Functions](../azure-functions/functions-overview.md), il est [recommandé](../azure-functions/functions-best-practices.md#avoid-long-running-functions) de créer un ensemble de petites fonctions qui fonctionnent bien ensemble plutôt que des fonctions volumineuses et durables.

À mesure que vous créerez des flux serverless basés sur les événements à l’aide du [déclencheur Azure Functions pour Cosmos DB](./change-feed-functions.md), le cas se présentera où vous voudrez faire plusieurs choses chaque fois qu’un nouvel événement se produit dans un [conteneur Azure Cosmos](./account-databases-containers-items.md#azure-cosmos-containers) déterminé. Si les actions que vous souhaitez déclencher sont indépendantes les unes des autres, la solution idéale consiste à **créer un déclencheur Azure Functions pour Cosmos DB par action** , chacun étant à l’écoute des modifications se produisant dans le même conteneur Azure Cosmos.

## <a name="optimizing-containers-for-multiple-triggers"></a>Optimisation des conteneurs pour plusieurs déclencheurs

Compte tenu des *exigences* du déclencheur Azure Functions pour Cosmos DB, nous avons besoin d’un second conteneur pour stocker l’état, également appelé *conteneur de baux*. Cela veut-il dire que vous avez besoin d’un conteneur de baux distinct pour chaque fonction Azure ?

Ici, vous avez le choix entre deux options :

* Créer **un conteneur de baux par fonction**  : cette approche peut entraîner des coûts supplémentaires, à moins que vous utilisiez une [base de données à débit partagé](./set-throughput.md#set-throughput-on-a-database). Ne perdez pas de vue que le débit minimal au niveau du conteneur est de 400 [unités de requête](./request-units.md), et dans le cas du conteneur de baux, il sert uniquement à vérifier la progression et à maintenir l’état.
* Prévoir **un conteneur de baux et le partager** pour toutes vos fonctions : cette deuxième option fait un meilleur usage des unités de requête provisionnées dans le conteneur, car elle permet à plusieurs fonctions Azure de partager et d’utiliser le même débit provisionné.

Le but de cet article est de vous guider jusqu’à parvenir à la deuxième option.

## <a name="configuring-a-shared-leases-container"></a>Configuration d’un conteneur de baux partagé

Pour configurer le conteneur de baux partagé, la seule tâche de configuration supplémentaire que vous devez effectuer au niveau de vos déclencheurs consiste à ajouter l’[attribut](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md#attributes-and-annotations) `LeaseCollectionPrefix` si vous utilisez C# ou l’[attribut](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md) `leaseCollectionPrefix` si vous utilisez JavaScript. La valeur de l’attribut doit être un descripteur logique de l’action effectuée par le déclencheur en question.

Par exemple, supposez que vous disposez de trois déclencheurs : un qui envoie des e-mails, un qui effectue une agrégation pour créer une vue matérialisée et un autre qui envoie les modifications à un autre espace de stockage pour une analyse ultérieure. Vous pouvez affecter à l’attribut `LeaseCollectionPrefix` la valeur « e-mails » au premier, « matérialisé » au deuxième et « analytique » au troisième.

Le point essentiel est que les trois déclencheurs **puissent utiliser la même configuration de conteneur de baux** (compte, base de données et nom de conteneur).

Voici un exemple de code très simple avec l’attribut `LeaseCollectionPrefix` en C# :

```cs
using Microsoft.Azure.Documents;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;
using Microsoft.Extensions.Logging;

[FunctionName("SendEmails")]
public static void SendEmails([CosmosDBTrigger(
    databaseName: "ToDoItems",
    collectionName: "Items",
    ConnectionStringSetting = "CosmosDBConnection",
    LeaseCollectionName = "leases",
    LeaseCollectionPrefix = "emails")]IReadOnlyList<Document> documents,
    ILogger log)
{
    ...
}

[FunctionName("MaterializedViews")]
public static void MaterializedViews([CosmosDBTrigger(
    databaseName: "ToDoItems",
    collectionName: "Items",
    ConnectionStringSetting = "CosmosDBConnection",
    LeaseCollectionName = "leases",
    LeaseCollectionPrefix = "materialized")]IReadOnlyList<Document> documents,
    ILogger log)
{
    ...
}
```

Et pour JavaScript, vous pouvez appliquer la configuration au fichier `function.json` avec l’attribut `leaseCollectionPrefix` :

```json
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "CosmosDBConnection",
    "databaseName": "ToDoItems",
    "collectionName": "Items",
    "leaseCollectionPrefix": "emails"
},
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "CosmosDBConnection",
    "databaseName": "ToDoItems",
    "collectionName": "Items",
    "leaseCollectionPrefix": "materialized"
}
```

> [!NOTE]
> Surveillez toujours les unités de requête provisionnées dans votre conteneur de baux partagé. Sachant que chaque déclencheur qui le partage augmentera la consommation moyenne de débit, vous serez peut-être amené à augmenter le débit provisionné à mesure qu’augmentera le nombre de fonctions Azure qui l’utilisent.

## <a name="next-steps"></a>Étapes suivantes

* Consultez la configuration complète du [déclencheur Azure Functions pour Cosmos DB](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md#configuration).
* Vérifiez la [liste d’exemples](../azure-functions/functions-bindings-cosmosdb-v2-trigger.md) étendue pour tous les langages.
* Pour obtenir d’autres exemple, visitez les recettes serverless avec Azure Cosmos DB et Azure Functions sur le [dépôt GitHub](https://github.com/ealsur/serverless-recipes/tree/master/cosmosdbtriggerscenarios).
