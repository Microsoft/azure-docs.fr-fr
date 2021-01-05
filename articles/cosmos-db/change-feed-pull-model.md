---
title: Changer le modèle d’extraction de flux
description: Découvrir comment utiliser le modèle de tirage (pull) du flux de modification Azure Cosmos DB pour lire le flux de modification et les différences entre le modèle de tirage et le processeur de flux de modification
author: timsander1
ms.author: tisande
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 12/04/2020
ms.reviewer: sngun
ms.openlocfilehash: 674bb67018fcbf7df6285a66c2e0aeb37d24f409
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96744880"
---
# <a name="change-feed-pull-model-in-azure-cosmos-db"></a>Modèle de tirage (pull) du flux de modification dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Avec le modèle de tirage du flux de modification, vous pouvez consommer le flux de modification Azure Cosmos DB à votre rythme. Comme le [processeur de flux de modification](change-feed-processor.md) vous le permet déjà, vous pouvez utiliser le modèle de tirage du flux de modification pour paralléliser le traitement des modifications sur plusieurs consommateurs de flux de modification.

> [!NOTE]
> Le modèle de tirage du flux de modification n’est actuellement disponible qu’en [préversion dans le SDK .NET Azure Cosmos DB](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.15.0-preview). La préversion n’est pas encore disponible pour d’autres versions du kit SDK.

## <a name="comparing-with-change-feed-processor"></a>Comparaison avec le processeur de flux de modification

De nombreux scénarios peuvent traiter le flux de modification à l’aide du [processeur de flux de modification](change-feed-processor.md) ou du modèle de tirage (pull). Les jetons de continuation du modèle de tirage, comme le conteneur de bail du processeur de flux de modification, sont des « signets » pour le dernier élément (ou lot d’éléments) traité dans le flux de modification.

Toutefois, vous ne pouvez pas convertir les jetons de continuation en conteneur de bail (ou vice versa).

> [!NOTE]
> Dans la plupart des cas, lorsque vous devez lire à partir du flux de modification, l’option la plus simple consiste à utiliser le [processeur de flux de modification](change-feed-processor.md).

Envisagez plutôt d’utiliser le modèle de tirage dans les scénarios suivants :

- Lire les modifications à partir d’une clé de partition particulière
- Contrôler la vitesse à laquelle votre client reçoit les modifications à traiter
- Effectuer une lecture unique des données existantes dans le flux de modification (par exemple, pour effectuer une migration de données)

Voici quelques différences capitales entre le processeur de flux de modification et le modèle de tirage :

|Fonctionnalité  | Processeur de flux de modification| Modèle de tirage (pull) |
| --- | --- | --- |
| Suivi du point actuel dans le traitement du flux de modification | Bail (stocké dans un conteneur Azure Cosmos DB) | Jeton de continuation (stocké en mémoire ou rendu persistant manuellement) |
| Possibilité de relire les modifications passées | Oui, avec le modèle d’envoi (push) | Oui, avec le modèle de tirage (pull)|
| Interrogation des modifications à venir | Recherche automatiquement les modifications en fonction de `WithPollInterval` spécifié par l’utilisateur | Manuel |
| Comportement dans lequel aucune nouvelle modification n’est apportée | Attente automatique `WithPollInterval` et nouvelle vérification | Interception de l’exception et nouvelle vérification manuelle |
| Traitement des modifications depuis l’ensemble du conteneur | Oui, et automatiquement mis en parallèle sur plusieurs threads/machines consommant à partir du même conteneur| Oui, et mis en parallèle manuellement à l’aide de FeedTokens |
| Traitement des modifications à partir d’une seule clé de partition | Non pris en charge | Oui|
| Niveau de support | Mise à la disposition générale | PRÉVERSION |

> [!NOTE]
> Contrairement à la lecture à l’aide du processeur de flux de modification, vous devez gérer explicitement les cas où aucune nouvelle modification n’est apportée. 

## <a name="consuming-an-entire-containers-changes"></a>Consommation des modifications d’un conteneur entier

Vous pouvez créer un `FeedIterator` pour traiter le flux de modification à l’aide du modèle de tirage. Lorsque vous créez un `FeedIterator`, vous devez spécifier une valeur `ChangeFeedStartFrom` obligatoire qui se compose de la position de départ pour la lecture des modifications et de la valeur `FeedRange` souhaitée. `FeedRange` est une plage de valeurs de clés de partition qui spécifie les éléments qui seront lus à partir du flux de modification en utilisant ce `FeedIterator`spécifique.

Vous pouvez, si vous le souhaitez, spécifier `ChangeFeedRequestOptions` pour définir un `PageSizeHint`. Le `PageSizeHint` est le nombre maximal d’éléments qui seront retournés dans une seule page.

Il existe deux types de `FeedIterator`. En plus des exemples ci-dessous qui retournent des objets d’entité, vous pouvez également obtenir la réponse avec la prise en charge de `Stream`. Les flux vous permettent de lire des données sans avoir à les désérialiser au préalable, en les enregistrant sur les ressources clientes.

Voici un exemple d’obtention d’un `FeedIterator` qui retourne des objets d’entité, dans ce cas un objet `User` :

```csharp
FeedIterator<User> InteratorWithPOCOS = container.GetChangeFeedIterator<User>(ChangeFeedStartFrom.Beginning());
```

Voici un exemple de l’obtention de `FeedIterator` qui retourne `Stream` :

```csharp
FeedIterator iteratorWithStreams = container.GetChangeFeedStreamIterator<User>(ChangeFeedStartFrom.Beginning());
```

Si vous ne fournissez pas de `FeedRange` à un `FeedIterator`, vous pouvez traiter l’intégralité du flux de modification d’un conteneur à votre propre rythme. Voici un exemple qui permet de lire toutes les modifications à partir de l’heure actuelle :

```csharp
FeedIterator iteratorForTheEntireContainer = container.GetChangeFeedStreamIterator<User>(ChangeFeedStartFrom.Now());

while (iteratorForTheEntireContainer.HasMoreResults)
{
    try {
        FeedResponse<User> users = await iteratorForTheEntireContainer.ReadNextAsync();

        foreach (User user in users)
            {
                Console.WriteLine($"Detected change for user with id {user.id}");
            }
    }
    catch {
        Console.WriteLine($"No new changes");
        Thread.Sleep(5000);
    }
}
```

Le flux de modification constituant effectivement une liste infinie d’éléments englobant toutes les écritures et mises à jour ultérieures, la valeur de `HasMoreResults` est toujours true. Lorsque vous essayez de lire le flux de modification et qu’aucune nouvelle modification n’est disponible, vous recevez une exception. Dans l’exemple ci-dessus, l’exception est gérée en attendant 5 secondes avant de revérifier les modifications.

## <a name="consuming-a-partition-keys-changes"></a>Consommation des modifications d’une clé de partition

Dans certains cas, vous souhaiterez peut-être traiter uniquement les modifications d’une clé de partition particulière. Vous pouvez obtenir `FeedIterator` pour une clé de partition spécifique, et traiter les modifications de la même façon que vous le faites pour un conteneur entier.

```csharp
FeedIterator<User> iteratorForPartitionKey = container.GetChangeFeedIterator<User>(ChangeFeedStartFrom.Beginning(FeedRange.FromPartitionKey(new PartitionKey("PartitionKeyValue"))));

while (iteratorForThePartitionKey.HasMoreResults)
{
    try {
        FeedResponse<User> users = await iteratorForThePartitionKey.ReadNextAsync();

        foreach (User user in users)
            {
                Console.WriteLine($"Detected change for user with id {user.id}");
            }
    }
    catch {
        Console.WriteLine($"No new changes");
        Thread.Sleep(5000);
    }
}
```

## <a name="using-feedrange-for-parallelization"></a>Utilisation de FeedRange pour la parallélisation

Dans le [processeur de flux de modification](change-feed-processor.md), le travail est automatiquement réparti sur plusieurs consommateurs. Dans le modèle de tirage du flux de modification, vous pouvez utiliser `FeedRange` pour paralléliser le traitement du flux de modification. `FeedRange` représente une plage de valeurs de clé de partition.

Voici un exemple qui montre comment obtenir une liste de plages pour votre conteneur :

```csharp
IReadOnlyList<FeedRange> ranges = await container.GetFeedRangesAsync();
```

Lorsque vous obtenez la liste des FeedRanges pour votre conteneur, vous récupérez un `FeedRange` par [partition physique](partitioning-overview.md#physical-partitions).

À l’aide d’un `FeedRange`, vous pouvez ensuite créer un `FeedIterator` pour paralléliser le traitement du flux de modification sur plusieurs machines ou threads. Contrairement à l’exemple précédent qui illustrait comment obtenir un `FeedIterator` pour l’ensemble du conteneur ou pour une clé de partition unique, vous pouvez utiliser FeedRanges pour obtenir plusieurs FeedIterators qui peuvent traiter le flux de modification en parallèle.

Si vous souhaitez utiliser FeedRanges, vous devez disposer d’un processus d’orchestrateur qui obtient FeedRanges et les distribue à ces machines. Cette distribution peut être :

* L’utilisation de `FeedRange.ToJsonString` et la distribution de cette valeur de chaîne. Les consommateurs peuvent utiliser cette valeur avec `FeedRange.FromJsonString`
* Le passage de la référence d’objet `FeedRange` si la distribution est en cours.

Voici un exemple montrant comment lire à partir du début du flux de modification du conteneur, au moyen de deux machines hypothétiques distinctes qui lisent en parallèle :

Machine 1 :

```csharp
FeedIterator<User> iteratorA = container.GetChangeFeedIterator<User>(ChangeFeedStartFrom.Beginning(ranges[0]));
while (iteratorA.HasMoreResults)
{
    try {
        FeedResponse<User> users = await iteratorA.ReadNextAsync();

        foreach (User user in users)
            {
                Console.WriteLine($"Detected change for user with id {user.id}");
            }
    }
    catch {
        Console.WriteLine($"No new changes");
        Thread.Sleep(5000);
    }
}
```

Machine 2 :

```csharp
FeedIterator<User> iteratorB = container.GetChangeFeedIterator<User>(ChangeFeedStartFrom.Beginning(ranges[1]));
while (iteratorB.HasMoreResults)
{
    try {
        FeedResponse<User> users = await iteratorA.ReadNextAsync();

        foreach (User user in users)
            {
                Console.WriteLine($"Detected change for user with id {user.id}");
            }
    }
    catch {
        Console.WriteLine($"No new changes");
        Thread.Sleep(5000);
    }
}
```

## <a name="saving-continuation-tokens"></a>Enregistrement des jetons de continuation

Vous pouvez enregistrer la position de votre `FeedIterator` en créant un jeton de continuation. Un jeton de continuation est une valeur de chaîne qui assure le suivi des dernières modifications traitées de votre FeedIterator. Le `FeedIterator` peut ainsi reprendre à cet endroit, ultérieurement. Le code suivant lit le flux de modification depuis la création du conteneur. Lorsque plus aucune modification n’est disponible, un jeton de continuation est conservé pour que la consommation du flux de modification puisse être reprise plus tard.

```csharp
FeedIterator<User> iterator = container.GetChangeFeedIterator<User>(ChangeFeedStartFrom.Beginning());

string continuation = null;

while (iterator.HasMoreResults)
{
   try { 
        FeedResponse<User> users = await iterator.ReadNextAsync();
        continuation = users.ContinuationToken;

        foreach (User user in users)
            {
                Console.WriteLine($"Detected change for user with id {user.id}");
            }
   }
    catch {
        Console.WriteLine($"No new changes");
        Thread.Sleep(5000);
    }   
}

// Some time later
FeedIterator<User> iteratorThatResumesFromLastPoint = container.GetChangeFeedIterator<User>(ChangeFeedStartFrom.ContinuationToken(continuation));
```

Tant que le conteneur Cosmos existe, le jeton de continuation d’un FeedIterator n’expire pas.

## <a name="next-steps"></a>Étapes suivantes

* [Présentation du flux de modification](change-feed.md)
* [Utilisation du processeur de flux de modification](change-feed-processor.md)
* [Déclencher Azure Functions](change-feed-functions.md)