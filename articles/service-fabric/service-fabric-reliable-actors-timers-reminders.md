---
title: Minuteurs et rappels dans Reliable Actors
description: Présentation des minuteurs et rappels pour Service Fabric Reliable Actors, avec notamment des conseils sur l’utilisation de chacun d’eux.
ms.topic: conceptual
ms.date: 11/02/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: 2b97b15ca4eb287f8d8f2c1af932f22acafae546
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89016544"
---
# <a name="actor-timers-and-reminders"></a>Minuteries et rappels d’acteur
Les acteurs peuvent planifier un travail régulier par eux-mêmes en inscrivant des minuteries ou des rappels. Cet article montre comment utiliser des minuteries et des rappels, puis explique les différences entre les deux.

## <a name="actor-timers"></a>Minuteries des acteurs
Les minuteries des acteurs fournissent un wrapper simple autour de minuteries .NET ou Java pour veiller à ce que les méthodes de rappel respectent les garanties d’un accès concurrentiel en alternance fournies par le runtime Actors.

Les acteurs peuvent utiliser les méthodes `RegisterTimer` (C#) ou `registerTimer` (Java) et `UnregisterTimer` (C#) ou `unregisterTimer` (Java) sur leur classe de base pour inscrire et désinscrire leurs minuteries. L'exemple ci-dessous montre l'utilisation des API de minuterie. Les API sont très similaires à la minuterie .NET ou Java. Dans cet exemple, quand la minuterie arrive à son terme, le runtime Actors appelle la méthode `MoveObject` (C#) ou `moveObject` (Java). Le respect par la méthode de l’accès concurrentiel en alternance est garanti. Aucune autre méthode d’acteur ou aucun autre rappel de minuterie n’est donc en cours d’exécution avant la fin de ce rappel.

```csharp
class VisualObjectActor : Actor, IVisualObject
{
    private IActorTimer _updateTimer;

    public VisualObjectActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    protected override Task OnActivateAsync()
    {
        ...

        _updateTimer = RegisterTimer(
            MoveObject,                     // Callback method
            null,                           // Parameter to pass to the callback method
            TimeSpan.FromMilliseconds(15),  // Amount of time to delay before the callback is invoked
            TimeSpan.FromMilliseconds(15)); // Time interval between invocations of the callback method

        return base.OnActivateAsync();
    }

    protected override Task OnDeactivateAsync()
    {
        if (_updateTimer != null)
        {
            UnregisterTimer(_updateTimer);
        }

        return base.OnDeactivateAsync();
    }

    private Task MoveObject(object state)
    {
        ...
        return Task.FromResult(true);
    }
}
```
```Java
public class VisualObjectActorImpl extends FabricActor implements VisualObjectActor
{
    private ActorTimer updateTimer;

    public VisualObjectActorImpl(FabricActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    @Override
    protected CompletableFuture onActivateAsync()
    {
        ...

        return this.stateManager()
                .getOrAddStateAsync(
                        stateName,
                        VisualObject.createRandom(
                                this.getId().toString(),
                                new Random(this.getId().toString().hashCode())))
                .thenApply((r) -> {
                    this.registerTimer(
                            (o) -> this.moveObject(o),                        // Callback method
                            "moveObject",
                            null,                                             // Parameter to pass to the callback method
                            Duration.ofMillis(10),                            // Amount of time to delay before the callback is invoked
                            Duration.ofMillis(timerIntervalInMilliSeconds));  // Time interval between invocations of the callback method
                    return null;
                });
    }

    @Override
    protected CompletableFuture onDeactivateAsync()
    {
        if (updateTimer != null)
        {
            unregisterTimer(updateTimer);
        }

        return super.onDeactivateAsync();
    }

    private CompletableFuture moveObject(Object state)
    {
        ...
        return this.stateManager().getStateAsync(this.stateName).thenCompose(v -> {
            VisualObject v1 = (VisualObject)v;
            v1.move();
            return (CompletableFuture<?>)this.stateManager().setStateAsync(stateName, v1).
                    thenApply(r -> {
                      ...
                      return null;});
        });
    }
}
```

La période suivante de la minuterie démarre après la fin du rappel. Cela implique que la minuterie s’arrête pendant l’exécution du rappel, puis démarre quand le rappel se termine.

Le runtime Actors enregistre les modifications apportées au Gestionnaire d’état de l’acteur à la fin du rappel. Si une erreur se produit lors de l'enregistrement de l'état, cet objet acteur sera désactivé et une nouvelle instance sera activée.

Contrairement aux [rappels](#actor-reminders), les minuteurs ne peuvent pas être mis à jour. En cas de nouvel appel de `RegisterTimer`, un nouveau minuteur sera inscrit.

Toutes les minuteries sont arrêtées quand l’acteur est désactivé dans le cadre du nettoyage de la mémoire. Aucun rappel de minuterie n’est appelé après cela. En outre, le runtime Actors ne conserve aucune information concernant les minuteries qui étaient exécutées avant la désactivation. C'est à l'acteur d'inscrire toutes les minuteries dont il a besoin lors d'une réactivation ultérieure. Pour plus d’informations, consultez la section sur le [nettoyage de la mémoire d’acteur](service-fabric-reliable-actors-lifecycle.md).

## <a name="actor-reminders"></a>Rappels d’acteur
Les rappels sont un mécanisme permettant de déclencher des rappels persistants sur un acteur à certaines heures. Leur fonctionnalité est semblable à celle des minuteries. Toutefois, contrairement aux minuteries, les rappels sont déclenchés en toutes circonstances jusqu’à ce que l’acteur les désinscrive ou qu’il soit supprimé explicitement. Plus précisément, les rappels sont déclenchés lors des désactivations d'acteur et des basculements car le runtime Actors conserve des informations sur les rappels de l'acteur à l’aide du fournisseur d’état d’acteur. Contrairement aux minuteurs, les rappels existants peuvent être mis à jour en appelant à nouveau la méthode d’inscription (`RegisterReminderAsync`) à l’aide du même *reminderName*.

> [!NOTE]
> La fiabilité des rappels est liée aux garanties de fiabilité d’état fournies par le fournisseur d’état d’acteur. Cela signifie que pour les acteurs dont la persistance de l’état est définie sur *Aucune*, les rappels ne sont pas déclenchés après un basculement.

Pour inscrire un rappel, un acteur appelle la méthode [`RegisterReminderAsync`](/dotnet/api/microsoft.servicefabric.actors.runtime.actorbase.registerreminderasync?view=azure-dotnet#remarks) fournie dans la classe de base, comme illustré dans l’exemple suivant :

```csharp
protected override async Task OnActivateAsync()
{
    string reminderName = "Pay cell phone bill";
    int amountInDollars = 100;

    IActorReminder reminderRegistration = await this.RegisterReminderAsync(
        reminderName,
        BitConverter.GetBytes(amountInDollars),
        TimeSpan.FromDays(3),    //The amount of time to delay before firing the reminder
        TimeSpan.FromDays(1));    //The time interval between firing of reminders
}
```

```Java
@Override
protected CompletableFuture onActivateAsync()
{
    String reminderName = "Pay cell phone bill";
    int amountInDollars = 100;

    ActorReminder reminderRegistration = this.registerReminderAsync(
            reminderName,
            state,
            dueTime,    //The amount of time to delay before firing the reminder
            period);    //The time interval between firing of reminders
}
```

Dans cet exemple, `"Pay cell phone bill"` est le nom du rappel. Il s’agit d’une chaîne que l’acteur utilise pour identifier de façon unique un rappel. `BitConverter.GetBytes(amountInDollars)` (C#) est le contexte associé au rappel. Il sera retourné à l’acteur en tant qu’argument pour le rappel de la relance, soit `IRemindable.ReceiveReminderAsync`(C#) ou `Remindable.receiveReminderAsync`(Java).

Les acteurs qui utilisent des rappels doivent implémenter l’interface `IRemindable` , comme illustré dans l’exemple ci-dessous.

```csharp
public class ToDoListActor : Actor, IToDoListActor, IRemindable
{
    public ToDoListActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task ReceiveReminderAsync(string reminderName, byte[] context, TimeSpan dueTime, TimeSpan period)
    {
        if (reminderName.Equals("Pay cell phone bill"))
        {
            int amountToPay = BitConverter.ToInt32(context, 0);
            System.Console.WriteLine("Please pay your cell phone bill of ${0}!", amountToPay);
        }
        return Task.FromResult(true);
    }
}
```
```Java
public class ToDoListActorImpl extends FabricActor implements ToDoListActor, Remindable
{
    public ToDoListActor(FabricActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture receiveReminderAsync(String reminderName, byte[] context, Duration dueTime, Duration period)
    {
        if (reminderName.equals("Pay cell phone bill"))
        {
            int amountToPay = ByteBuffer.wrap(context).getInt();
            System.out.println("Please pay your cell phone bill of " + amountToPay);
        }
        return CompletableFuture.completedFuture(true);
    }

```

Quand un rappel est déclenché, le runtime Reliable Actors appelle la méthode `ReceiveReminderAsync` (C#) ou `receiveReminderAsync` (Java) sur l’acteur. Un acteur peut inscrire plusieurs rappels, et la méthode `ReceiveReminderAsync` (C#) ou `receiveReminderAsync` (Java) est appelée chaque fois qu’un de ces rappels est déclenché. L’acteur peut ensuite utiliser le nom du rappel qui est envoyé à la méthode `ReceiveReminderAsync` (C#) ou `receiveReminderAsync` (Java) pour identifier le rappel qui a été déclenché.

Le runtime Actors enregistre l’état de l’acteur à la fin de l’appel `ReceiveReminderAsync` (C#) ou `receiveReminderAsync` (Java). Si une erreur se produit lors de l'enregistrement de l'état, cet objet acteur sera désactivé et une nouvelle instance sera activée.

Pour désinscrire un rappel, un acteur appelle la méthode `UnregisterReminderAsync` (C#) ou `unregisterReminderAsync` (Java), comme l’illustre l’exemple ci-dessous.

```csharp
IActorReminder reminder = GetReminder("Pay cell phone bill");
Task reminderUnregistration = UnregisterReminderAsync(reminder);
```
```Java
ActorReminder reminder = getReminder("Pay cell phone bill");
CompletableFuture reminderUnregistration = unregisterReminderAsync(reminder);
```

Comme indiqué ci-dessus, la méthode `UnregisterReminderAsync`(C#) ou `unregisterReminderAsync`(Java) accepte une interface `IActorReminder` (C#) ou `ActorReminder` (Java). La classe de base de l’acteur prend en charge une méthode `GetReminder` (C#) ou `getReminder` (Java) qui peut être utilisée pour récupérer l’interface `IActorReminder` (C#) ou `ActorReminder` (Java) en passant le nom du rappel. C’est pratique, car l’acteur n’a pas besoin de conserver l’interface `IActorReminder` (C#) ou `ActorReminder` (Java) renvoyée par l’appel de la méthode `RegisterReminder` (C#) ou `registerReminder` (Java).

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur les événements acteurs fiables et la réentrance :
* [Événements d’acteurs](service-fabric-reliable-actors-events.md)
* [Réentrance des acteurs](service-fabric-reliable-actors-reentrancy.md)
