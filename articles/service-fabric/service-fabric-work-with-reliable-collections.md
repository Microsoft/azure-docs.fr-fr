---
title: Utilisation des collections fiables
description: Découvrez les bonnes pratiques liées à l’utilisation des collections fiables dans une application Azure Service Fabric.
ms.topic: conceptual
ms.date: 03/10/2020
ms.openlocfilehash: 2d027dc432d1a0a20888bfca4f59bc41866e358d
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94651904"
---
# <a name="working-with-reliable-collections"></a>Utilisation des collections fiables
Service Fabric propose un modèle de programmation avec état disponible pour les développeurs .NET via les collections fiables. Plus précisément, Service Fabric fournit un dictionnaire fiable et des classes de file d’attente fiables. Lorsque vous utilisez ces classes, votre état est partitionné (pour l’évolutivité) répliqué (pour la disponibilité) et traité dans une partition (pour la sémantique ACID). Examinons l'utilisation type d'un objet de dictionnaire fiable afin de découvrir ses fonctionnalités réelles.

```csharp
try
{
   // Create a new Transaction object for this partition
   using (ITransaction tx = base.StateManager.CreateTransaction())
   {
      // AddAsync takes key's write lock; if >4 secs, TimeoutException
      // Key & value put in temp dictionary (read your own writes),
      // serialized, redo/undo record is logged & sent to secondary replicas
      await m_dic.AddAsync(tx, key, value, cancellationToken);

      // CommitAsync sends Commit record to log & secondary replicas
      // After quorum responds, all locks released
      await tx.CommitAsync();
   }
   // If CommitAsync isn't called, Dispose sends Abort
   // record to log & all locks released
}
catch (TimeoutException)
{
   // choose how to handle the situation where you couldn't get a lock on the file because it was 
   // already in use. You might delay and retry the operation
   await Task.Delay(100);
}
```

Toutes les opérations relatives aux objets de dictionnaires fiables (à l'exception de ClearAsync qui n'est pas annulable) nécessitent un objet ITransaction. Cet objet est associé à toutes les modifications que vous tentez d'apporter à un dictionnaire fiable et/ou à des objets de file d'attente fiable au sein d'une même partition. Vous obtenez un objet ITransaction en appelant la méthode CreateTransaction du StateManager de la partition.

Dans le code ci-dessus, l'objet ITransaction est transféré vers la méthode AddAsync d'un dictionnaire fiable. En interne, les méthodes de dictionnaire qui acceptent une clé prennent un verrou de lecture/écriture associé à la clé. Si la méthode modifie la valeur de la clé, elle accepte un verrou d'écriture sur la clé, et si elle ne lit qu'à partir de la valeur de la clé, un verrou de lecture est appliqué sur la clé. Puisque AddAsync modifie la valeur de la clé en la remplaçant par la nouvelle valeur transmise, le verrou d'écriture de la clé est appliqué. Par conséquent, si 2 threads (ou plus) tentent d'ajouter des valeurs à la même clé au même moment, un thread acquerra le verrou d'écriture et les autres threads se bloqueront. Par défaut, les méthodes se bloquent pendant 4 secondes maximum pour acquérir le verrou. Après 4 secondes, les méthodes lèvent une exception TimeoutException. Il existe des surcharges de méthode qui vous permettent de transmettre une valeur de délai d'attente explicite si vous le souhaitez.

En règle générale, vous écrivez votre code de manière à réagir à une exception TimeoutException en l’interceptant et en recommençant toute l’opération (comme indiqué dans le code ci-dessus). Dans ce code simple, nous appellons simplement Task.Delay en transmettant 100 millisecondes à chaque fois. Mais, en réalité, vous pouvez juger préférable d’utiliser un type de délai de temporisation exponentiel.

Une fois le verrou acquis, AddAsync ajoute les références d’objet de clé et de valeur à un dictionnaire temporaire interne associé à l’objet ITransaction. De cette manière, vous obtenez une sémantique de type « lecture de vos propres écritures ». Autrement dit, après avoir appelé AddAsync, un appel ultérieur à TryGetValueAsync à l’aide du même objet ITransaction renverra la valeur même si vous n’avez pas encore validé la transaction.

> [!NOTE]
> L’appel de TryGetValueAsync avec une nouvelle transaction retourne une référence à la dernière valeur validée. Ne modifiez pas directement cette référence, car cela a pour effet de contourner le mécanisme de persistance et de réplication des modifications. Nous vous recommandons de faire en sorte que les valeurs soient en lecture seule, de sorte que la seule façon de modifier la valeur d’une clé soit d’utiliser des API de dictionnaire fiables.

Ensuite, AddAsync sérialise vos objets de clé et de valeur dans des tableaux d’octets et ajoute ces tableaux d’octets à un fichier journal situé sur le nœud local. Pour finir, AddAsync envoie les tableaux d’octets à tous les réplicas secondaires de manière à leur fournir les mêmes informations de clé/valeur. Même si les informations de clé/valeur ont été écrites dans un fichier journal, les informations ne sont pas considérées comme intégrées au dictionnaire tant que la transaction qui leur est associée n’a pas été validée.

Dans le code ci-dessus, l'appel à CommitAsync permet de valider toutes les opérations de la transaction. Plus précisément, il ajoute des informations de validation au fichier journal situé sur le nœud local et envoie également l’enregistrement de validation à tous les réplicas secondaires. Dès lors qu’un quorum (une majorité) de réplicas a répondu, toutes les modifications apportées aux données sont considérées comme permanentes et tous les verrous associés aux clés qui ont été manipulées à l’aide de l’objet ITransaction sont libérés afin que d’autres threads/transactions puissent manipuler les mêmes clés et les valeurs qui leur sont associées.

Si CommitAsync n’est pas appelée (généralement en raison de la levée d’une exception), l’objet ITransaction est détruit. Lors de la suppression d'un objet ITransaction non validé, Service Fabric ajoute les informations d'abandon au fichier journal situé sur le nœud local et rien n'est envoyé aux réplicas secondaires. Dès lors, tous les verrous associés aux clés qui ont été manipulées à l’aide de la transaction sont libérés.

## <a name="volatile-reliable-collections"></a>Collections fiables volatiles 
Dans certaines charges de travail, comme un cache répliqué, une perte de données occasionnelle peut être tolérée. En évitant la persistance des données sur le disque, vous pouvez améliorer les latences et les débits lors de l'écriture dans les dictionnaires fiables. Mais l'absence de persistance a un inconvénient : en cas de perte du quorum, toutes les données sont également perdues. Cela dit, l'amélioration des performances est telle et les pertes de quorum tellement rares que le risque en vaut souvent la chandelle pour ces charges de travail.

Actuellement, la prise en charge volatile est uniquement disponible pour les dictionnaires fiables et les files d'attente fiables, et non pour ReliableConcurrentQueues. Pour déterminer si vous pouvez utiliser des collections volatiles, consultez la liste des [Mises en garde](service-fabric-reliable-services-reliable-collections-guidelines.md#volatile-reliable-collections).

Pour activer la prise en charge volatile dans votre service, définissez l'indicateur ```HasPersistedState``` de la déclaration de type de service sur ```false```, comme suit :
```xml
<StatefulServiceType ServiceTypeName="MyServiceType" HasPersistedState="false" />
```

>[!NOTE]
>Les services persistants existants ne peuvent pas devenir volatils, et vice versa. Pour bénéficier d'un service volatil, vous devez supprimer le service existant, puis déployer le service avec l'indicateur mis à jour. Autrement dit, si vous souhaitez modifier l'indicateur ```HasPersistedState```, vous devez être prêt à subir une perte totale de données. 

## <a name="common-pitfalls-and-how-to-avoid-them"></a>Pièges courants et comment les éviter
Maintenant que vous avez compris le fonctionnement interne des collections fiables, examinons quelques cas d'erreurs d'utilisation courants. Observez le code ci-dessous :

```csharp
using (ITransaction tx = StateManager.CreateTransaction())
{
   // AddAsync serializes the name/user, logs the bytes,
   // & sends the bytes to the secondary replicas.
   await m_dic.AddAsync(tx, name, user);

   // The line below updates the property's value in memory only; the
   // new value is NOT serialized, logged, & sent to secondary replicas.
   user.LastLogin = DateTime.UtcNow;  // Corruption!

   await tx.CommitAsync();
}
```

Lorsque vous travaillez avec un dictionnaire .NET standard, vous pouvez ajouter une clé/valeur au dictionnaire puis modifier la valeur d’une propriété (par exemple, LastLogin). Toutefois, ce code ne fonctionnera pas correctement avec un dictionnaire fiable. N’oubliez pas que, dans la discussion précédente, l’appel à AddAsync sérialise les objets de clé/valeur dans des tableaux d’octets, puis enregistre ces tableaux dans un fichier local et les envoie également aux réplicas secondaires. Si vous modifiez ultérieurement une propriété, seule la valeur stockée en mémoire est modifiée. La modification n'a aucun impact sur le fichier local ou sur les données envoyées aux réplicas. Si le processus se bloque, le contenu de la mémoire est immédiatement nettoyé. Au démarrage d’un nouveau processus ou si un autre réplica prend le rôle de réplica principal, vous obtiendrez l’ancienne valeur de propriété.

Il est extrêmement facile de commettre le type d’erreur décrit ci-dessus. De plus, vous ne vous rendrez compte que vous avez commis une erreur que si/lorsque le processus se bloque. Pour écrire correctement le code, il suffit d’inverser les deux lignes :


```csharp
using (ITransaction tx = StateManager.CreateTransaction())
{
   user.LastLogin = DateTime.UtcNow;  // Do this BEFORE calling AddAsync
   await m_dic.AddAsync(tx, name, user);
   await tx.CommitAsync();
}
```

Voici un autre exemple d’erreur courante :

```csharp
using (ITransaction tx = StateManager.CreateTransaction())
{
   // Use the user's name to look up their data
   ConditionalValue<User> user = await m_dic.TryGetValueAsync(tx, name);

   // The user exists in the dictionary, update one of their properties.
   if (user.HasValue)
   {
      // The line below updates the property's value in memory only; the
      // new value is NOT serialized, logged, & sent to secondary replicas.
      user.Value.LastLogin = DateTime.UtcNow; // Corruption!
      await tx.CommitAsync();
   }
}
```

Là encore, avec les dictionnaires .NET standard, le code ci-dessus fonctionne correctement et représente un modèle courant : le développeur utilise une clé pour rechercher une valeur. Si la valeur existe, le développeur modifie la valeur d'une propriété. Toutefois, avec les collections fiables, ce code présente le même problème que celui indiqué précédemment : **vous ne devez pas modifier un objet une fois que vous l’avez attribué à une collection fiable**.

Pour mettre à jour une valeur dans une collection fiable, le mieux est d’obtenir une référence à la valeur existante et de considérer comme immuable l’objet référencé par cette référence. Créez ensuite un nouvel objet qui sera la copie exacte de l'objet d'origine. À présent, vous pouvez modifier l’état de ce nouvel objet et écrire le nouvel objet dans la collection afin qu’il soit sérialisé dans des tableaux d’octets, ajouté au fichier local et envoyé aux réplicas. Après avoir validé la ou les modifications, les objets en mémoire, le fichier local et tous les réplicas présenteront exactement le même état. Tout est parfait !

Le code ci-dessous montre comment mettre à jour une valeur dans une collection fiable :

```csharp
using (ITransaction tx = StateManager.CreateTransaction())
{
   // Use the user's name to look up their data
   ConditionalValue<User> currentUser = await m_dic.TryGetValueAsync(tx, name);

   // The user exists in the dictionary, update one of their properties.
   if (currentUser.HasValue)
   {
      // Create new user object with the same state as the current user object.
      // NOTE: This must be a deep copy; not a shallow copy. Specifically, only
      // immutable state can be shared by currentUser & updatedUser object graphs.
      User updatedUser = new User(currentUser);

      // In the new object, modify any properties you desire
      updatedUser.LastLogin = DateTime.UtcNow;

      // Update the key's value to the updateUser info
      await m_dic.SetValue(tx, name, updatedUser);
      await tx.CommitAsync();
   }
}
```

## <a name="define-immutable-data-types-to-prevent-programmer-error"></a>Définir des types de données immuables pour éviter les erreurs de programmation
Dans l'idéal, nous aimerions que le compilateur signale les erreurs lorsque vous produisez accidentellement du code qui modifie l'état d'un objet que vous êtes censé considérer comme immuable. Mais le compilateur C# ne permet pas de le faire. Par conséquent, pour éviter les éventuels bogues de programmation, il est vivement recommandé de définir les types vous utilisez avec les collections fiables comme des types immuables. Plus précisément, cela signifie que vous allez vous en tenir aux types de valeur de base (par exemple, des nombres [Int32, UInt64, etc.], DateTime, Guid, TimeSpan, etc.). Vous pouvez aussi utiliser String. Il est préférable d'éviter les propriétés de collection car la sérialisation et la désérialisation de ces propriétés nuisent souvent aux performances. Toutefois, si vous souhaitez utiliser des propriétés de collection, nous vous recommandons d'utiliser la bibliothèque de collections immuables de .NET ([System.Collections.Immutable](https://www.nuget.org/packages/System.Collections.Immutable/)). Cette bibliothèque est disponible au téléchargement sur https://nuget.org. Nous vous recommandons également de sceller vos classes et de définir les champs en lecture seule chaque fois que cela est possible.

Le type UserInfo ci-dessous montre comment définir un type immuable, en tirant parti des recommandations mentionnées précédemment.

```csharp
[DataContract]
// If you don't seal, you must ensure that any derived classes are also immutable
public sealed class UserInfo
{
   private static readonly IEnumerable<ItemId> NoBids = ImmutableList<ItemId>.Empty;

   public UserInfo(String email, IEnumerable<ItemId> itemsBidding = null) 
   {
      Email = email;
      ItemsBidding = (itemsBidding == null) ? NoBids : itemsBidding.ToImmutableList();
   }

   [OnDeserialized]
   private void OnDeserialized(StreamingContext context)
   {
      // Convert the deserialized collection to an immutable collection
      ItemsBidding = ItemsBidding.ToImmutableList();
   }

   [DataMember]
   public readonly String Email;

   // Ideally, this would be a readonly field but it can't be because OnDeserialized
   // has to set it. So instead, the getter is public and the setter is private.
   [DataMember]
   public IEnumerable<ItemId> ItemsBidding { get; private set; }

   // Since each UserInfo object is immutable, we add a new ItemId to the ItemsBidding
   // collection by creating a new immutable UserInfo object with the added ItemId.
   public UserInfo AddItemBidding(ItemId itemId)
   {
      return new UserInfo(Email, ((ImmutableList<ItemId>)ItemsBidding).Add(itemId));
   }
}
```

Le type ItemId est également un type immuable comme indiqué ici :

```csharp
[DataContract]
public struct ItemId
{
   [DataMember] public readonly String Seller;
   [DataMember] public readonly String ItemName;
   public ItemId(String seller, String itemName)
   {
      Seller = seller;
      ItemName = itemName;
   }
}
```

## <a name="schema-versioning-upgrades"></a>Contrôle de version du schéma (mises à niveau)
En interne, les collections fiables sérialisent vos objets à l'aide du DataContractSerializer de .NET. Les objets sérialisés sont conservés sur le disque local du réplica principal et sont également transmis aux réplicas secondaires. À mesure que votre service évolue, il est probable que vous souhaitiez modifier le type de données (schéma) dont votre service a besoin. Abordez la question du contrôle de version de vos données avec une extrême prudence. Tout d’abord, vous devez toujours être en mesure de désérialiser les anciennes données. Plus précisément, cela signifie que votre code de désérialisation doit avoir une compatibilité descendante à l’infini : la version 333 de votre code de service doit être en mesure de fonctionner sur les données placées dans une collection fiable par la version 1 de votre code de service créé 5 ans plus tôt.

En outre, le code de service est mis à niveau à raison d’un domaine de mise à niveau à la fois. Par conséquent, pendant une mise à niveau, deux versions différentes de votre code de service s’exécutent simultanément. Vous devez éviter que la nouvelle version de votre code de service utilise le nouveau schéma, étant donné que les anciennes versions de votre code de service ne seront peut-être pas en mesure de gérer le nouveau schéma. Dans la mesure du possible, vous devez concevoir chaque version de votre service de manière à ce qu'elle soit compatible avec les versions ultérieures. Plus précisément, cela signifie que la V1 de votre code de service doit pouvoir ignorer tous les éléments de schéma qu'elle ne gère pas explicitement. Toutefois, elle doit être en mesure d'enregistrer toutes les données qu'elle ne connaît pas explicitement et de les réécrire lors de la mise à jour d'une valeur ou d'une clé de dictionnaire.

> [!WARNING]
> Bien que vous puissiez modifier le schéma d'une clé, vous devez vous assurer de la stabilité du code de hachage et des algorithmes d'égalisation de votre clé. Si vous modifiez le mode de fonctionnement de l’un de ces algorithmes, vous ne pourrez plus jamais rechercher la clé dans le dictionnaire fiable.
> Les chaînes .NET peuvent être utilisées comme clés, mais utilisez la chaîne elle-même en tant que clé et n’utilisez pas le résultat de String.GetHashCode comme clé.

Vous pouvez également effectuer ce que l'on appelle communément une mise à niveau en deux phases. Dans le cadre d'une mise à niveau en deux phases, vous mettez à niveau votre service de la V1 vers la V2 : la V2 contient le code capable de prendre en charge les nouvelles modifications du schéma, mais ce code ne s'exécute pas. Lorsque le code V2 lit les données de la V1, il agit sur ces dernières et écrit les données V1. Ensuite, une fois la mise à niveau effectuée sur tous les domaines de mise à niveau, vous pouvez d’une certaine manière signaler aux instances V2 en cours d’exécution que la mise à niveau est terminée (pour ce faire, vous pouvez déployer une mise à niveau de la configuration ; c’est précisément cette opération qui en fait une mise à niveau en deux phases). À présent, les instances V2 peuvent lire les données de V1, les convertir en données V2, les exploiter et les écrire en tant que données V2. Lorsque d’autres instances lisent les données V2, elles n’ont pas besoin de les convertir. Elles les exploitent simplement et écrivent des données V2.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur la création de contrats de données à compatibilité ascendante, consultez [Contrats de données à compatibilité ascendante](/dotnet/framework/wcf/feature-details/forward-compatible-data-contracts).

Pour découvrir les meilleures pratiques relatives au contrôle de version des contrats de données, consultez [Contrôle de version des contrats de données](/dotnet/framework/wcf/feature-details/data-contract-versioning).

Pour savoir comment implémenter des contrats de données à tolérance de version, consultez [Rappels de sérialisation avec tolérance de version](/dotnet/framework/wcf/feature-details/version-tolerant-serialization-callbacks).

Pour savoir comment fournir une structure de données capable d'interagir entre plusieurs versions, consultez [IExtensibleDataObject](/dotnet/api/system.runtime.serialization.iextensibledataobject?view=netcore-3.1).
