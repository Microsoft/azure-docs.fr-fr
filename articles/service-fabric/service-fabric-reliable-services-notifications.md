---
title: Notifications Reliable Services
description: Documentation conceptuelle sur les notifications de Service Fabric Reliable Services pour le gestionnaire d’état et le dictionnaire fiables
author: mcoskun
ms.topic: conceptual
ms.date: 6/29/2017
ms.author: mcoskun
ms.custom: devx-track-csharp
ms.openlocfilehash: 4a336daf9bd7400d049233a22a04d64d561b42c9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89021950"
---
# <a name="reliable-services-notifications"></a>Notifications Reliable Services
Les notifications permettent aux clients de suivre les modifications apportées à un objet qui les intéresse. Deux types d’objets prennent en charge les notifications : le *Gestionnaire d’état fiable* et le *Dictionnaire fiable*.

Raisons courantes d’utiliser les notifications :

* Construction de vues matérialisées, comme des index secondaires ou des vues filtrées agrégées de l’état du réplica. Par exemple, un index trié de toutes les clés d’un dictionnaire fiable.
* Envoi de données de surveillance comme le nombre d’utilisateurs ajoutés durant la dernière heure.

Les notifications sont déclenchées dans le cadre de l’application d’opérations. Pour cette raison, les notifications doivent être traitées le plus rapidement possible, et les événements synchrones ne doivent pas inclure d’opérations coûteuses.

## <a name="reliable-state-manager-notifications"></a>Notifications du Gestionnaire d’état fiable
Le Gestionnaire d’état fiable fournit des notifications pour les événements suivants :

* Transaction
  * Commit
* Gestionnaire d’état
  * Reconstruire
  * Ajout d’un état fiable
  * Suppression d’un état fiable

Le Gestionnaire d’état fiable effectue le suivi à la volée des transactions en cours. Le seul changement d’état de transaction qui déclenche une notification est une transaction en cours de validation.

Le Gestionnaire d’état fiable tient à jour une collection d’états fiables comme Dictionnaire fiable et File d’attente fiable. Le Gestionnaire d’état fiable déclenche des notifications quand cette collection est modifiée : un état fiable est ajouté, supprimé, ou la collection entière est reconstruite.
La collection du Gestionnaire d’état fiable est reconstruite dans trois cas :

* Récupération : quand un réplica démarre, il récupère son état précédent à partir du disque. À la fin de la récupération, il utilise **NotifyStateManagerChangedEventArgs** pour déclencher un événement qui contient l’ensemble des états fiables récupérés.
* Copie complète : avant qu’un réplica puisse rejoindre le jeu de configuration, il doit être créé. Dans certains cas, il est nécessaire d’appliquer une copie complète de l’état du Gestionnaire d’état fiable du réplica principal au réplica secondaire inactif. Le Gestionnaire d’état fiable sur le réplica secondaire utilise **NotifyStateManagerChangedEventArgs** pour déclencher un événement qui contient l’ensemble des états fiables qu’il a acquis à partir du réplica principal.
* Restauration : dans les scénarios de récupération d’urgence, vous pouvez restaurer l’état du réplica à partir d’une sauvegarde par le biais de **RestoreAsync**. Dans ces cas-là, le Gestionnaire d’état fiable sur le réplica secondaire utilise **NotifyStateManagerChangedEventArgs** pour déclencher un événement qui contient l’ensemble des états fiables qu’il a restauré à partir de la sauvegarde.

Pour vous inscrire aux notifications de transactions et/ou notifications du Gestionnaire d’état, vous devez vous inscrire auprès de l’événement **TransactionChanged** ou **StateManagerChanged** sur le Gestionnaire d’état fiable. Le constructeur de votre élément StatefulService est un endroit courant où s’inscrire auprès de ces gestionnaires d’événements. Si vous êtes inscrit dans le constructeur, vous ne manquerez aucune notification due à une modification pendant la durée de vie de **IReliableStateManager**.

```csharp
public MyService(StatefulServiceContext context)
    : base(MyService.EndpointName, context, CreateReliableStateManager(context))
{
    this.StateManager.TransactionChanged += this.OnTransactionChangedHandler;
    this.StateManager.StateManagerChanged += this.OnStateManagerChangedHandler;
}
```

Le gestionnaire d’événements **TransactionChanged** utilise **NotifyTransactionChangedEventArgs** pour fournir des détails sur l’événement. Il contient la propriété d’action (par exemple, **NotifyTransactionChangedAction.Commit**) qui spécifie le type de modification. Il contient également la propriété de transaction qui fournit une référence à la transaction qui a changé.

> [!NOTE]
> Aujourd’hui, les événements **TransactionChanged** sont déclenchés uniquement si la transaction est validée. L’action est alors égale à **NotifyTransactionChangedAction.Commit**. Mais à l’avenir, des événements pourront être déclenchés pour d’autres types de modifications d’état de transaction. Nous vous recommandons de vérifier l’action et de traiter l’événement uniquement s’il correspond à ce que vous attendez.
> 
> 

Voici un exemple de gestionnaire d’événements **TransactionChanged** .

```csharp
private void OnTransactionChangedHandler(object sender, NotifyTransactionChangedEventArgs e)
{
    if (e.Action == NotifyTransactionChangedAction.Commit)
    {
        this.lastCommitLsn = e.Transaction.CommitSequenceNumber;
        this.lastTransactionId = e.Transaction.TransactionId;

        this.lastCommittedTransactionList.Add(e.Transaction.TransactionId);
    }
}
```

Le gestionnaire d’événements **StateManagerChanged** utilise **NotifyStateManagerChangedEventArgs** pour fournir des détails sur l’événement.
**NotifyStateManagerChangedEventArgs** comporte deux sous-classes : **NotifyStateManagerRebuildEventArgs** et **NotifyStateManagerSingleEntityChangedEventArgs**.
Vous utilisez la propriété d’action dans **NotifyStateManagerChangedEventArgs** pour convertir **NotifyStateManagerChangedEventArgs** en la sous-classe correcte :

* **NotifyStateManagerChangedAction.Rebuild** : **NotifyStateManagerRebuildEventArgs**
* **NotifyStateManagerChangedAction.Add** et **NotifyStateManagerChangedAction.Remove** : **NotifyStateManagerSingleEntityChangedEventArgs**

Voici un exemple de gestionnaire de notification **StateManagerChanged** .

```csharp
public void OnStateManagerChangedHandler(object sender, NotifyStateManagerChangedEventArgs e)
{
    if (e.Action == NotifyStateManagerChangedAction.Rebuild)
    {
        this.ProcessStateManagerRebuildNotification(e);

        return;
    }

    this.ProcessStateManagerSingleEntityNotification(e);
}
```

## <a name="reliable-dictionary-notifications"></a>Notifications Dictionnaire fiable
Dictionnaire fiable fournit des notifications pour les événements suivants :

* Rebuild : appelée quand **ReliableDictionary** a récupéré son état à partir d’une sauvegarde ou d’un état local récupéré ou copié.
* Effacement : appelée quand l’état de **ReliableDictionary** a été effacé par le biais de la méthode **ClearAsync**.
* Ajoutez : appelée quand un élément a été ajouté à **ReliableDictionary**.
* Mise à jour : appelée quand un élément de **ReliableDictionary** a été mis à jour.
* Suppression : appelée quand un élément de **ReliableDictionary** a été supprimé.

Pour obtenir des notifications Dictionnaire fiable, vous devez vous inscrire auprès du gestionnaire d’événements **DictionaryChanged** sur **IReliableDictionary**. La notification d’ajout **ReliableStateManager.StateManagerChanged** est une situation courante au cours de laquelle l’utilisateur s’inscrit auprès de ces gestionnaires d’événements.
L’inscription quand **IReliableDictionary** est ajouté à **IReliableStateManager** permet de s’assurer que vous ne manquerez aucune notification.

```csharp
private void ProcessStateManagerSingleEntityNotification(NotifyStateManagerChangedEventArgs e)
{
    var operation = e as NotifyStateManagerSingleEntityChangedEventArgs;

    if (operation.Action == NotifyStateManagerChangedAction.Add)
    {
        if (operation.ReliableState is IReliableDictionary<TKey, TValue>)
        {
            var dictionary = (IReliableDictionary<TKey, TValue>)operation.ReliableState;
            dictionary.RebuildNotificationAsyncCallback = this.OnDictionaryRebuildNotificationHandlerAsync;
            dictionary.DictionaryChanged += this.OnDictionaryChangedHandler;
        }
    }
}
```

> [!NOTE]
> **ProcessStateManagerSingleEntityNotification** est l’exemple de méthode appelée par l’exemple **OnStateManagerChangedHandler** précédent.
> 
> 

Le code précédent définit l’interface **IReliableNotificationAsyncCallback**, ainsi que **DictionaryChanged**. Étant donné que **NotifyDictionaryRebuildEventArgs** contient une interface **IAsyncEnumerable** qui doit être énumérée de façon asynchrone, des notifications de reconstruction sont déclenchées par le biais de **RebuildNotificationAsyncCallback** au lieu de **OnDictionaryChangedHandler**.

```csharp
public async Task OnDictionaryRebuildNotificationHandlerAsync(
    IReliableDictionary<TKey, TValue> origin,
    NotifyDictionaryRebuildEventArgs<TKey, TValue> rebuildNotification)
{
    this.secondaryIndex.Clear();

    var enumerator = e.State.GetAsyncEnumerator();
    while (await enumerator.MoveNextAsync(CancellationToken.None))
    {
        this.secondaryIndex.Add(enumerator.Current.Key, enumerator.Current.Value);
    }
}
```

> [!NOTE]
> Dans le code précédent, dans le cadre du traitement de la notification de reconstruction, l’état agrégé est d’abord effacé. La collection fiable étant en cours de reconstruction avec un nouvel état, toutes les notifications précédentes sont sans intérêt.
> 
> 

Le gestionnaire d’événements **DictionaryChanged** utilise **NotifyDictionaryChangedEventArgs** pour fournir des détails sur l’événement.
**NotifyDictionaryChangedEventArgs** a cinq sous-classes. Utilisez la propriété d’action dans **NotifyDictionaryChangedEventArgs** pour convertir **NotifyDictionaryChangedEventArgs** en la sous-classe correcte :

* **NotifyDictionaryChangedAction.Rebuild** : **NotifyDictionaryRebuildEventArgs**
* **NotifyDictionaryChangedAction.Clear** : **NotifyDictionaryClearEventArgs**
* **NotifyDictionaryChangedAction.Add** : **NotifyDictionaryItemAddedEventArgs**
* **NotifyDictionaryChangedAction.Update** : **NotifyDictionaryItemUpdatedEventArgs**
* **NotifyDictionaryChangedAction.Remove** : **NotifyDictionaryItemRemovedEventArgs**

```csharp
public void OnDictionaryChangedHandler(object sender, NotifyDictionaryChangedEventArgs<TKey, TValue> e)
{
    switch (e.Action)
    {
        case NotifyDictionaryChangedAction.Clear:
            var clearEvent = e as NotifyDictionaryClearEventArgs<TKey, TValue>;
            this.ProcessClearNotification(clearEvent);
            return;

        case NotifyDictionaryChangedAction.Add:
            var addEvent = e as NotifyDictionaryItemAddedEventArgs<TKey, TValue>;
            this.ProcessAddNotification(addEvent);
            return;

        case NotifyDictionaryChangedAction.Update:
            var updateEvent = e as NotifyDictionaryItemUpdatedEventArgs<TKey, TValue>;
            this.ProcessUpdateNotification(updateEvent);
            return;

        case NotifyDictionaryChangedAction.Remove:
            var deleteEvent = e as NotifyDictionaryItemRemovedEventArgs<TKey, TValue>;
            this.ProcessRemoveNotification(deleteEvent);
            return;

        default:
            break;
    }
}
```

## <a name="recommendations"></a>Recommandations
* *Complétez* les événements de notification le plus rapidement possible.
* *N’exécutez aucune* opération coûteuse (par exemple des opérations d’E/S) dans le cadre d’événements synchrones.
* *Vérifiez* le type d’action avant de traiter l’événement. De nouveaux types d’actions pourront être ajoutés à l’avenir.

Voici quelques points à retenir :

* Les notifications sont déclenchées dans le cadre de l’exécution d’une opération. Par exemple, une notification de restauration est déclenchée comme dernière étape d’une opération de restauration. Une restauration ne se termine pas tant que l’événement de notification n’a pas été traité.
* Les notifications étant déclenchées dans le cadre des opérations d’application, les clients ne voient que les notifications des opérations validées localement. De plus, dans la mesure où les opérations sont garanties uniquement pour être validées localement (en d’autres termes consignées), elles risquent de ne pas pouvoir être annulées par la suite.
* Lors d’une opération de rétablissement, une seule notification est déclenchée pour chaque opération appliquée. Cela signifie que si la transaction T1 inclut Create(X), Delete(X) et Create(X), vous obtiendrez une notification pour la création de X, une autre pour sa suppression et une autre pour sa création, dans cet ordre.
* Pour les transactions qui contiennent plusieurs opérations, les opérations sont appliquées dans l’ordre dans lequel elles ont été reçues sur le réplica principal.
* Lors du traitement d’une mauvaise progression, certaines opérations peuvent être annulées. Des notifications sont déclenchées pour ces opérations d’annulation afin de ramener le réplica à un état stable. Les notifications d’annulation présentent une importante différence. En effet, les événements avec des clés en double sont agrégés. Par exemple, si la transaction T1 est annulée, vous ne verrez qu’une seule notification Delete(X).

## <a name="next-steps"></a>Étapes suivantes
* [Collections fiables](service-fabric-work-with-reliable-collections.md)
* [Démarrage rapide de Reliable Services](service-fabric-reliable-services-quick-start.md)
* [Sauvegarde et restauration de Reliable Services (récupération d’urgence)](service-fabric-reliable-services-backup-restore.md)
* [Référence du développeur pour les Collections fiables](/dotnet/api/microsoft.servicefabric.data.collections?view=azure-dotnet#microsoft_servicefabric_data_collections)
