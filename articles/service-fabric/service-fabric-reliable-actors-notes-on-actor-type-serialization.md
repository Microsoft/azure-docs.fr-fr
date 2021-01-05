---
title: Remarques sur Reliable Actors pour la sérialisation de type d’acteur
description: Traite des exigences de base pour la définition des classes sérialisables pouvant servir à définir les interfaces et les états Service Fabric Reliable Actors
ms.topic: conceptual
ms.date: 11/02/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: 2a1118e394046b217a288663659a2c910098e992
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576007"
---
# <a name="notes-on-service-fabric-reliable-actors-type-serialization"></a>Remarques sur la sérialisation de type Reliable Actors Service Fabric
Les arguments de toutes les méthodes, les types de résultats des tâches retournées par chaque méthode dans une interface d’acteur et les objets stockés dans le Gestionnaire d’état d’un acteur doivent être [sérialisables en contrat de données](/dotnet/framework/wcf/feature-details/types-supported-by-the-data-contract-serializer). Cela s’applique également aux arguments des méthodes définies dans les [interfaces d’événement d’acteur](service-fabric-reliable-actors-events.md). (Les méthodes d’interface d’événement d’acteur retournent toujours la valeur nulle.)

## <a name="custom-data-types"></a>Types de données personnalisés
Dans cet exemple, l’interface d’acteur suivante définit une méthode qui retourne un type de données personnalisé appelé `VoicemailBox` :

```csharp
public interface IVoiceMailBoxActor : IActor
{
    Task<VoicemailBox> GetMailBoxAsync();
}
```

```Java
public interface VoiceMailBoxActor extends Actor
{
    CompletableFuture<VoicemailBox> getMailBoxAsync();
}
```

L’interface est implémentée par un acteur qui utilise le gestionnaire d’état pour stocker un objet `VoicemailBox` :

```csharp
[StatePersistence(StatePersistence.Persisted)]
public class VoiceMailBoxActor : Actor, IVoicemailBoxActor
{
    public VoiceMailBoxActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task<VoicemailBox> GetMailboxAsync()
    {
        return this.StateManager.GetStateAsync<VoicemailBox>("Mailbox");
    }
}

```

```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
public class VoiceMailBoxActorImpl extends FabricActor implements VoicemailBoxActor
{
    public VoiceMailBoxActorImpl(ActorService actorService, ActorId actorId)
    {
         super(actorService, actorId);
    }

    public CompletableFuture<VoicemailBox> getMailBoxAsync()
    {
         return this.stateManager().getStateAsync("Mailbox");
    }
}

```

Dans cet exemple, l’objet `VoicemailBox` est sérialisé quand :

* L’objet est transmis entre une instance d’acteur et un appelant.
* L’objet est enregistré dans le gestionnaire d’état quand il est enregistré sur disque et répliqué vers d’autres nœuds.

L’infrastructure Reliable Actor utilise la sérialisation DataContract. Par conséquent, les objets de données personnalisés et leurs membres doivent être annotés avec les attributs **DataContract** et **DataMember**, respectivement.

```csharp
[DataContract]
public class Voicemail
{
    [DataMember]
    public Guid Id { get; set; }

    [DataMember]
    public string Message { get; set; }

    [DataMember]
    public DateTime ReceivedAt { get; set; }
}
```
```Java
public class Voicemail implements Serializable
{
    private static final long serialVersionUID = 42L;

    private UUID id;                    //getUUID() and setUUID()

    private String message;             //getMessage() and setMessage()

    private GregorianCalendar receivedAt; //getReceivedAt() and setReceivedAt()
}
```


```csharp
[DataContract]
public class VoicemailBox
{
    public VoicemailBox()
    {
        this.MessageList = new List<Voicemail>();
    }

    [DataMember]
    public List<Voicemail> MessageList { get; set; }

    [DataMember]
    public string Greeting { get; set; }
}
```
```Java
public class VoicemailBox implements Serializable
{
    static final long serialVersionUID = 42L;
    
    public VoicemailBox()
    {
        this.messageList = new ArrayList<Voicemail>();
    }

    private List<Voicemail> messageList;   //getMessageList() and setMessageList()

    private String greeting;               //getGreeting() and setGreeting()
}
```


## <a name="next-steps"></a>Étapes suivantes
* [Cycle de vie des acteurs et Garbage Collection](service-fabric-reliable-actors-lifecycle.md)
* [Minuteries et rappels d’acteur](service-fabric-reliable-actors-timers-reminders.md)
* [Événements d’acteurs](service-fabric-reliable-actors-events.md)
* [Réentrance des acteurs](service-fabric-reliable-actors-reentrancy.md)
* [Polymorphisme des acteurs et modèles de conception orientée objet](service-fabric-reliable-actors-polymorphism.md)
* [Diagnostics et surveillance des performances d’acteur](service-fabric-reliable-actors-diagnostics.md)
