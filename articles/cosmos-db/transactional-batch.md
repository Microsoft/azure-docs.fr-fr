---
title: Opérations de lot transactionnel dans Azure Cosmos DB à l’aide du SDK .NET
description: Découvrez comment utiliser TransactionalBatch dans le SDK .NET Azure Cosmos DB pour effectuer un groupe d’opérations de point qui réussissent ou échouent.
author: stefArroyo
ms.author: esarroyo
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 10/27/2020
ms.openlocfilehash: 9f6692db2da3722507136a468d1dcbdc2985e73f
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347555"
---
# <a name="transactional-batch-operations-in-azure-cosmos-db-using-the-net-sdk"></a>Opérations de lot transactionnel dans Azure Cosmos DB à l’aide du SDK .NET
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Le lot transactionnel décrit un groupe d’opérations de point qui doivent réussir ou échouer avec la même clé de partition dans un conteneur. Dans le SDK .NET, la classe `TransactionalBatch` est utilisée pour définir ce lot d’opérations. Si toutes les opérations réussissent dans l’ordre dans lequel elles sont décrites dans l’opération de lot transactionnel, la transaction est validée. Toutefois, si une opération échoue, la totalité de la transaction est restaurée.

## <a name="whats-a-transaction-in-azure-cosmos-db"></a>Définition d’une transaction dans Azure Cosmos DB

Une transaction dans une base de données classique peut être définie comme étant une séquence d'opérations effectuées en tant qu'unité de travail logique unique. Chaque transaction fournit des garanties de propriété ACID (Atomicité, Cohérence, Isolation, Durabilité).

* **L’atomicité** permet de s’assurer que toutes les opérations effectuées au sein d’une transaction sont traitées en tant que simple unité validée dans son intégralité ou aucunement.
* La **cohérence** permet de s’assurer que les données sont toujours dans un état valide d’une transaction à l’autre.
* **L’isolation**, quant à elle, permet de garantir qu’aucune transaction n’interfère avec les autres. De nombreux systèmes commerciaux fournissent plusieurs niveaux d’isolation pouvant être utilisés en fonction des besoins des applications.
* La **durabilité** permet de s’assurer que toute modification validée dans une base de données sera toujours présente.
Azure Cosmos DB prend en charge les [transactions entièrement compatibles avec ACID avec isolement d’instantané](database-transactions-optimistic-concurrency.md) pour les opérations au sein de la même [clé de partition logique](partitioning-overview.md).

## <a name="transactional-batch-operations-vs-stored-procedures"></a>Opérations de lot transactionnel ou procédures stockées

Azure Cosmos DB prend actuellement en charge les procédures stockées, qui fournissent également l’étendue transactionnelle sur les opérations. Toutefois, les opérations de lot transactionnel offrent les avantages suivants :

* **Option de langage** : le lot transactionnel est pris en charge dans le SDK et dans le langage que vous utilisez déjà, tandis que les procédures stockées doivent être écrites en JavaScript.
* **Contrôle de version du code** : le contrôle de version du code d’application et son intégration dans votre pipeline CI/CD est bien plus naturel que l’orchestration de la mise à jour d’une procédure stockée et le fait de s’assurer que la substitution se produit au bon moment. Cela facilite également la restauration des modifications.
* **Performance** : réduction de la latence sur les opérations équivalentes jusqu’à 30 % par rapport à l’exécution de la procédure stockée.
* **Sérialisation de contenu** : chaque opération au sein d’un lot transactionnel peut tirer parti des options de sérialisation personnalisées pour sa charge utile.

## <a name="how-to-create-a-transactional-batch-operation"></a>Procédure de création d’une opération de lot transactionnel

Lors de la création d’une opération de lot transactionnel, vous commencez à partir d’une instance de conteneur et appelez `CreateTransactionalBatch` :

```csharp
string partitionKey = "The Family";
ParentClass parent = new ParentClass(){ Id = "The Parent", PartitionKey = partitionKey, Name = "John", Age = 30 };
ChildClass child = new ChildClass(){ Id = "The Child", ParentId = parent.Id, PartitionKey = partitionKey };
TransactionalBatch batch = container.CreateTransactionalBatch(new PartitionKey(parent.PartitionKey)) 
  .CreateItem<ParentClass>(parent)
  .CreateItem<ChildClass>(child);
```

Ensuite, vous devez appeler `ExecuteAsync` sur le lot :

```csharp
TransactionalBatchResponse batchResponse = await batch.ExecuteAsync();
```

Une fois la réponse reçue, examinez si elle a réussi ou non et extrayez les résultats :

```csharp
using (batchResponse)
{
  if (batchResponse.IsSuccessStatusCode)
  {
    TransactionalBatchOperationResult<ParentClass> parentResult = batchResponse.GetOperationResultAtIndex<ParentClass>(0);
    ParentClass parentClassResult = parentResult.Resource;
    TransactionalBatchOperationResult<ChildClass> childResult = batchResponse.GetOperationResultAtIndex<ChildClass>(1);
    ChildClass childClassResult = childResult.Resource;
  }
}
```

En cas d’échec, l’opération ayant échoué a un code d’état correspondant à l’erreur. Toutes les autres opérations ont un code d’état 424 (échec de la dépendance). Dans l’exemple ci-dessous, l’opération échoue car elle tente de créer un élément qui existe déjà (409 HttpStatusCode.Conflict). Les codes d’état facilitent l’identification de la cause de l’échec de la transaction.

```csharp
// Parent's birthday!
parent.Age = 31;
// Naming two children with the same name, should abort the transaction
ChildClass anotherChild = new ChildClass(){ Id = "The Child", ParentId = parent.Id, PartitionKey = partitionKey };
TransactionalBatchResponse failedBatchResponse = await container.CreateTransactionalBatch(new PartitionKey(partitionKey))
  .ReplaceItem<ParentClass>(parent.Id, parent)
  .CreateItem<ChildClass>(anotherChild)
  .ExecuteAsync();

using (failedBatchResponse)
{
  if (!failedBatchResponse.IsSuccessStatusCode)
  {
    TransactionalBatchOperationResult<ParentClass> parentResult = failedBatchResponse.GetOperationResultAtIndex<ParentClass>(0);
    // parentResult.StatusCode is 424
    TransactionalBatchOperationResult<ChildClass> childResult = failedBatchResponse.GetOperationResultAtIndex<ChildClass>(1);
    // childResult.StatusCode is 409
  }
}
```

## <a name="how-are-transactional-batch-operations-executed"></a>Procédure d’exécution des opérations de lot transactionnel

Lorsque la méthode `ExecuteAsync` est appelée, toutes les opérations de l’objet `TransactionalBatch` sont regroupées, sérialisées en une seule charge utile et envoyées en tant que requête unique au service Azure Cosmos DB.

Le service reçoit la demande et exécute toutes les opérations au sein d’une étendue transactionnelle et retourne une réponse en utilisant le même protocole de sérialisation. Cette réponse est une réussite ou un échec. Elle fournit les réponses aux opérations individuelles par opération.

Le SDK expose la réponse pour vous permettre de vérifier le résultat et, éventuellement, d’extraire chacun des résultats de l’opération interne.

## <a name="limitations"></a>Limites

Il existe actuellement deux limites connues :

* La limite de taille de requête Azure Cosmos DB spécifie que la taille de la charge utile `TransactionalBatch` ne peut pas dépasser 2 Mo et la durée d’exécution maximale est de 5 secondes.
* Il existe une limite actuelle de 100 opérations par `TransactionalBatch` pour garantir que les performances sont telles que prévues et conformes aux contrats SLA.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur ce que [TransactionalBatch vous permet de faire](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/TransactionalBatch)

* Pour plus d’informations sur l’utilisation de notre SDK .NET Cosmos DB, consultez nos [exemples](sql-api-dotnet-v3sdk-samples.md)
