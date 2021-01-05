---
title: Guide du développeur pour les entités durables dans .NET - Azure Functions
description: Guide pratique pour utiliser les entités durables dans .NET avec l’extension Durable Functions pour Azure Functions.
author: sebastianburckhardt
ms.topic: conceptual
ms.date: 10/06/2019
ms.author: azfuncdf
ms.openlocfilehash: 88d2a23104b67dae8fd480406eb9171e9f3d5652
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92740014"
---
# <a name="developers-guide-to-durable-entities-in-net"></a>Guide des entités durables en .NET pour les développeurs

Dans cet article, nous décrivons en détail les interfaces disponibles pour le développement d’entités durables avec .NET, notamment à l’aide d’exemples et de conseils généraux. 

Les fonctions d’entité fournissent aux développeurs d’applications serverless un moyen pratique d’organiser l’état de l’application en tant que collection d’entités affinées. Pour plus d’informations sur les concepts sous-jacents, consultez l’article [Entités durables : Concepts](durable-functions-entities.md).

Nous proposons deux API pour définir des entités :

- La **syntaxe basée sur les classes** représente les entités et les opérations sous forme de classes et de méthodes. Cette syntaxe produit un code facile à lire et permet aux opérations d’être appelées par contrôle de type via des interfaces. 

- La syntaxe **basée sur les fonctions** est une interface de niveau inférieur qui représente les entités sous forme de fonctions. Elle fournit un contrôle précis sur la façon dont les opérations de l’entité sont réparties ainsi que sur la façon dont l’état de l’entité est géré.  

Cet article porte principalement sur la syntaxe basée sur les classes, car nous pensons qu’elle est mieux adaptée à la plupart des applications. Toutefois, la syntaxe [basée sur les fonctions](#function-based-syntax) peut convenir aux applications qui souhaitent définir ou gérer leurs propres abstractions pour l’état et les opérations de l’entité. De plus, elle peut également être appropriée pour implémenter des bibliothèques qui nécessitent un état générique non pris en charge par la syntaxe basée sur les classes. 

> [!NOTE]
> La syntaxe basée sur les classes est simplement une couche au-dessus de la syntaxe basée sur les fonctions. Ainsi, les deux variantes peuvent être utilisées de manière interchangeable dans la même application. 
 
## <a name="defining-entity-classes"></a>Définition des classes d’entités

L’exemple suivant est une implémentation d’une entité `Counter` qui stocke une seule valeur de type entier et propose quatre opérations `Add`, `Reset`, `Get` et `Delete`.

```csharp
[JsonObject(MemberSerialization.OptIn)]
public class Counter
{
    [JsonProperty("value")]
    public int Value { get; set; }

    public void Add(int amount) 
    {
        this.Value += amount;
    }

    public Task Reset() 
    {
        this.Value = 0;
        return Task.CompletedTask;
    }

    public Task<int> Get() 
    {
        return Task.FromResult(this.Value);
    }

    public void Delete() 
    {
        Entity.Current.DeleteState();
    }

    [FunctionName(nameof(Counter))]
    public static Task Run([EntityTrigger] IDurableEntityContext ctx)
        => ctx.DispatchAsync<Counter>();
}
```

La fonction `Run` contient le texte réutilisable nécessaire à l’utilisation de la syntaxe basée sur les classes. Il doit s’agir d’une fonction Azure *statique* . Elle s’exécute une fois pour chaque message d’opération traité par l’entité. Quand `DispatchAsync<T>` est appelé et que l’entité n’est pas déjà en mémoire, il construit un objet de type `T` et renseigne ses champs à partir du dernier JSON persistant trouvé dans le stockage (le cas échéant). Il appelle ensuite la méthode avec le nom correspondant.

> [!NOTE]
> L’état d’une entité basée sur la classe est **créé implicitement** avant que l’entité ne traite une opération. Il peut être **supprimé explicitement** dans une opération via l’appel de `Entity.Current.DeleteState()`.

### <a name="class-requirements"></a>Exigences des classes
 
Les classes d’entités sont des POCO (objets CLR traditionnels) qui ne nécessitent aucune superclasse, aucune interface ou aucun attribut particulier. Toutefois :

- La classe doit être constructible (consultez [Construction de l’entité](#entity-construction)).
- La classe doit être sérialisable au format JSON (consultez [Sérialisation d’entité](#entity-serialization)).

De plus, toute méthode destinée à être appelée en tant qu’opération doit répondre à des exigences supplémentaires :

- Une opération doit avoir un argument au maximum. De plus, elle ne doit pas avoir de surcharges, ni d’arguments de type générique.
- Une opération destinée à être appelée à partir d’une orchestration à l’aide d’une interface doit retourner `Task` ou `Task<T>`.
- Les arguments et les valeurs retournées doivent être des valeurs ou des objets sérialisables.

### <a name="what-can-operations-do"></a>Que peuvent faire les opérations ?

Toutes les opérations de l’entité peuvent lire et mettre à jour l’état de l’entité. Les changements apportés à l’état sont automatiquement rendus persistants dans le stockage. De plus, les opérations peuvent effectuer des calculs d’E/S externes ou d’autres calculs, dans les limites générales communes à toutes les fonctions Azure Functions.

Les opérations ont également accès aux fonctionnalités fournies par le contexte de `Entity.Current` :

* `EntityName` : nom de l’entité en cours d’exécution.
* `EntityKey` : clé de l’entité en cours d’exécution.
* `EntityId` : ID de l’entité en cours d’exécution (inclut le nom et la clé).
* `SignalEntity` : envoie un message unidirectionnel à une entité.
* `CreateNewOrchestration` : démarre une nouvelle orchestration.
* `DeleteState` : supprime l’état de cette entité.

Par exemple, nous pouvons modifier l’entité Counter pour qu’elle démarre une orchestration quand le compteur atteint 100 et qu’elle passe l’ID d’entité en tant qu’argument d’entrée :

```csharp
    public void Add(int amount) 
    {
        if (this.Value < 100 && this.Value + amount >= 100)
        {
            Entity.Current.StartNewOrchestration("MilestoneReached", Entity.Current.EntityId);
        }
        this.Value += amount;      
    }
```

## <a name="accessing-entities-directly"></a>Accès direct aux entités

Vous pouvez accéder directement aux entités basées sur les classes, à l’aide de noms de chaînes explicites pour l’entité et ses opérations. Nous donnons quelques exemples ci-dessous ; pour une explication plus approfondie des concepts sous-jacents (par exemple, la différence entre les signaux et les appels), voir la discussion dans [Accéder aux entités](durable-functions-entities.md#access-entities). 

> [!NOTE]
> Dans la mesure du possible, nous vous recommandons d’[accéder aux entités via des interfaces](#accessing-entities-through-interfaces), car cela permet un meilleur contrôle de type.

### <a name="example-client-signals-entity"></a>Exemple : le client signale l’entité

La fonction HTTP Azure suivante implémente une opération DELETE à l’aide des conventions REST. Elle envoie un signal de suppression à l’entité Counter dont la clé est passée dans le chemin de l’URL.

```csharp
[FunctionName("DeleteCounter")]
public static async Task<HttpResponseMessage> DeleteCounter(
    [HttpTrigger(AuthorizationLevel.Function, "delete", Route = "Counter/{entityKey}")] HttpRequestMessage req,
    [DurableClient] IDurableEntityClient client,
    string entityKey)
{
    var entityId = new EntityId("Counter", entityKey);
    await client.SignalEntityAsync(entityId, "Delete");    
    return req.CreateResponse(HttpStatusCode.Accepted);
}
```

### <a name="example-client-reads-entity-state"></a>Exemple : le client lit l’état de l’entité

La fonction HTTP Azure suivante implémente une opération GET à l’aide des conventions REST. Elle lit l’état actuel de l’entité Counter dont la clé est passée dans le chemin de l’URL.

```csharp
[FunctionName("GetCounter")]
public static async Task<HttpResponseMessage> GetCounter(
    [HttpTrigger(AuthorizationLevel.Function, "get", Route = "Counter/{entityKey}")] HttpRequestMessage req,
    [DurableClient] IDurableEntityClient client,
    string entityKey)
{
    var entityId = new EntityId("Counter", entityKey);
    var state = await client.ReadEntityStateAsync<Counter>(entityId); 
    return req.CreateResponse(state);
}
```

> [!NOTE]
> L’objet retourné par `ReadEntityStateAsync` est simplement une copie locale, c’est-à-dire une capture instantanée de l’état de l’entité à un moment antérieur. En particulier, il peut être périmé, et la modification de cet objet n’a aucun effet sur l’entité réelle. 

### <a name="example-orchestration-first-signals-then-calls-entity"></a>Exemple : l’orchestration signale, puis appelle l’entité

L’orchestration suivante signale une entité Counter pour l’incrémenter, puis appelle la même entité afin de lire sa dernière valeur.

```csharp
[FunctionName("IncrementThenGet")]
public static async Task<int> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var entityId = new EntityId("Counter", "myCounter");

    // One-way signal to the entity - does not await a response
    context.SignalEntity(entityId, "Add", 1);

    // Two-way call to the entity which returns a value - awaits the response
    int currentValue = await context.CallEntityAsync<int>(entityId, "Get");

    return currentValue;
}
```

## <a name="accessing-entities-through-interfaces"></a>Accès aux entités via des interfaces

Les interfaces permettent d’accéder aux entités via des objets proxy générés. Cette approche garantit que le nom et le type d’argument d’une opération correspondent à ce qui est implémenté. Nous vous recommandons d’utiliser des interfaces pour accéder aux entités chaque fois que cela est possible.

Par exemple, nous pouvons modifier l’exemple de l’entité Counter comme suit :

```csharp
public interface ICounter
{
    void Add(int amount);
    Task Reset();
    Task<int> Get();
    void Delete();
}

public class Counter : ICounter
{
    ...
}
```

Les classes d’entités et les interfaces d’entités sont similaires aux interfaces de graines et aux graines popularisées par [Orleans](https://www.microsoft.com/research/project/orleans-virtual-actors/). Pour plus d’informations sur les similitudes et les différences entre les entités durables et Orleans, consultez [Comparaison avec des acteurs virtuels](durable-functions-entities.md#comparison-with-virtual-actors).

Outre le contrôle de type, les interfaces permettent de mieux séparer les préoccupations au sein de l’application. Par exemple, dans la mesure où une entité peut implémenter plusieurs interfaces, une seule entité peut remplir plusieurs rôles. De plus, dans la mesure où une interface peut être implémentée par plusieurs entités, les modèles de communication généraux peuvent être implémentés sous forme de bibliothèques réutilisables.

### <a name="example-client-signals-entity-through-interface"></a>Exemple : le client signale l’entité via l’interface

Le code du client peut utiliser `SignalEntityAsync<TEntityInterface>` pour envoyer des signaux aux entités qui implémentent `TEntityInterface`. Par exemple :

```csharp
[FunctionName("DeleteCounter")]
public static async Task<HttpResponseMessage> DeleteCounter(
    [HttpTrigger(AuthorizationLevel.Function, "delete", Route = "Counter/{entityKey}")] HttpRequestMessage req,
    [DurableClient] IDurableEntityClient client,
    string entityKey)
{
    var entityId = new EntityId("Counter", entityKey);
    await client.SignalEntityAsync<ICounter>(entityId, proxy => proxy.Delete());    
    return req.CreateResponse(HttpStatusCode.Accepted);
}
```

Dans cet exemple, le paramètre `proxy` est une instance de `ICounter` générée dynamiquement, qui traduit de manière interne l’appel de `Delete` en un signal.

> [!NOTE]
> Les API `SignalEntityAsync` peuvent être utilisées uniquement pour les opérations unidirectionnelles. Même si une opération retourne `Task<T>`, la valeur du paramètre `T` est toujours null ou `default`, mais pas le résultat réel.
Par exemple, il n’est pas judicieux de signaler l’opération `Get`, car aucune valeur n’est retournée. À la place, les clients peuvent utiliser `ReadStateAsync` pour accéder directement à l’état du compteur, ou démarrer une fonction d’orchestrateur qui appelle l’opération `Get`. 

### <a name="example-orchestration-first-signals-then-calls-entity-through-proxy"></a>Exemple : l’orchestration signale, puis appelle l’entité via un proxy

Pour appeler ou signaler une entité à partir d’une orchestration, vous pouvez utiliser `CreateEntityProxy` ainsi que le type d’interface afin de générer un proxy pour l’entité. Ce proxy peut ensuite être utilisé pour appeler ou signaler des opérations :

```csharp
[FunctionName("IncrementThenGet")]
public static async Task<int> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var entityId = new EntityId("Counter", "myCounter");
    var proxy = context.CreateEntityProxy<ICounter>(entityId);

    // One-way signal to the entity - does not await a response
    proxy.Add(1);

    // Two-way call to the entity which returns a value - awaits the response
    int currentValue = await proxy.Get();

    return currentValue;
}
```

Implicitement, toutes les opérations qui retournent `void` sont signalées, et toutes les opérations qui retournent `Task` ou `Task<T>` sont appelées. Vous pouvez changer ce comportement par défaut et signaler les opérations même si elles retournent Task, en utilisant explicitement la méthode `SignalEntity<IInterfaceType>`.

### <a name="shorter-option-for-specifying-the-target"></a>Option plus courte pour spécifier la cible

Durant l’appel ou la signalisation d’une entité à l’aide d’une interface, le premier argument doit spécifier l’entité cible. Vous pouvez spécifier la cible en indiquant l’ID de l’entité ou, dans les cas où une seule classe implémente l’entité, en indiquant simplement la clé de l’entité :

```csharp
context.SignalEntity<ICounter>(new EntityId(nameof(Counter), "myCounter"), ...);
context.SignalEntity<ICounter>("myCounter", ...);
```

Si seule la clé de l’entité est spécifiée et si aucune implémentation unique n’est localisée au moment de l’exécution, `InvalidOperationException` est levé. 

### <a name="restrictions-on-entity-interfaces"></a>Restrictions sur les interfaces d’entités

Comme d’habitude, tous les types de paramètre et de retour doivent être sérialisables au format JSON. Sinon, des exceptions de sérialisation sont levées au moment de l’exécution.

Nous appliquons également certaines règles supplémentaires :
* Les interfaces d’entité doivent uniquement définir des méthodes.
* Les interfaces d’entités ne doivent pas contenir de paramètres génériques.
* Les méthodes d’interface d’entité ne doivent pas avoir plus d’un paramètre.
* Les méthodes d’interface d’entité doivent retourner `void`, `Task` ou `Task<T>` 

Si l’une de ces règles est enfreinte, un `InvalidOperationException` est levé au moment de l’exécution quand l’interface est utilisée en tant qu’argument de type pour `SignalEntity` ou `CreateProxy`. Le message d’exception indique la règle qui a été enfreinte.

> [!NOTE]
> Les méthodes d’interface qui retournent `void` peuvent uniquement être signalées (unidirectionnelles), elles ne peuvent pas être appelées (bidirectionnelles). Les méthodes d’interface qui retournent `Task` ou `Task<T>` peuvent être appelées ou signalées. Si elles sont appelées, elles retournent le résultat de l’opération ou les exceptions levées par l’opération. Toutefois, quand elles sont signalées, elles ne retournent pas à proprement parler le résultat ou l’exception de l’opération, mais simplement la valeur par défaut.

## <a name="entity-serialization"></a>Sérialisation d’entité

Dans la mesure où l’état d’une entité est persistant de manière durable, la classe d’entité doit être sérialisable. Le runtime de l’extension Durable Functions utilise à cet effet la bibliothèque [Json.NET](https://www.newtonsoft.com/json), qui prend en charge un certain nombre de stratégies et d’attributs pour contrôler le processus de sérialisation et de désérialisation. Les types de données C# les plus couramment utilisés (notamment les tableaux et les types de collection) sont déjà sérialisables et peuvent être facilement utilisés pour définir l’état des entités durables.

Par exemple, Json.NET peut facilement sérialiser et désérialiser la classe suivante :

```csharp
[JsonObject(MemberSerialization = MemberSerialization.OptIn)]
public class User
{
    [JsonProperty("name")]
    public string Name { get; set; }

    [JsonProperty("yearOfBirth")]
    public int YearOfBirth { get; set; }

    [JsonProperty("timestamp")]
    public DateTime Timestamp { get; set; }

    [JsonProperty("contacts")]
    public Dictionary<Guid, Contact> Contacts { get; set; } = new Dictionary<Guid, Contact>();

    [JsonObject(MemberSerialization = MemberSerialization.OptOut)]
    public struct Contact
    {
        public string Name;
        public string Number;
    }

    ...
}
```

### <a name="serialization-attributes"></a>Attributs de sérialisation

Dans l’exemple ci-dessus, nous avons choisi d’inclure plusieurs attributs pour rendre la sérialisation sous-jacente plus visible :
- Nous annotons la classe avec `[JsonObject(MemberSerialization.OptIn)]` afin de nous rappeler qu’elle doit être sérialisable ainsi que pour conserver uniquement les membres explicitement marqués en tant que propriétés JSON.
-  Nous annotons les champs à rendre persistants avec `[JsonProperty("name")]` afin de nous rappeler qu’un champ fait partie de l’état de l’entité rendue persistante ainsi que pour spécifier le nom de la propriété à utiliser dans la représentation JSON.

Toutefois, ces attributs ne sont pas obligatoires. D’autres conventions ou attributs sont autorisés tant qu’ils fonctionnent avec Json.NET. Par exemple, vous pouvez utiliser `[DataContract]` attributs ou aucun attribut :

```csharp
[DataContract]
public class Counter
{
    [DataMember]
    public int Value { get; set; }
    ...
}

public class Counter
{
    public int Value;
    ...
}
```

Par défaut, le nom de la classe n’est *pas* stocké dans la représentation JSON : autrement dit, nous utilisons `TypeNameHandling.None` en tant que paramètre par défaut. Ce comportement par défaut peut être remplacé à l’aide des attributs `JsonObject` ou `JsonProperty`.

### <a name="making-changes-to-class-definitions"></a>Apport de changements aux définitions de classe

Vous devez faire preuve d’une attention particulière quand vous apportez des changements à une définition de classe après l’exécution d’une application, car l’objet JSON stocké ne correspond peut-être plus à la nouvelle définition de classe. Toutefois, il est souvent possible de gérer correctement le changement des formats de données à condition de bien comprendre le processus de désérialisation utilisé par `JsonConvert.PopulateObject`.

Par exemple, voici quelques exemples de changements et leurs effets :

1. Si une nouvelle propriété est ajoutée, et si elle n’est pas présente dans le JSON stocké, sa valeur par défaut est supposée.
1. Si une propriété est supprimée, et si elle n’est pas présente dans le JSON stocké, le contenu antérieur est perdu.
1. Si une propriété est renommée, cela revient à supprimer l’ancienne propriété pour en ajouter une nouvelle.
1. Si le type d’une propriété change et s’il ne peut plus être désérialisé à partir du JSON stocké, une exception est levée.
1. Si le type d’une propriété change, mais qu’il peut toujours être désérialisé à partir du JSON stocké, l’opération s’effectue.

De nombreuses options permettent de personnaliser le comportement de Json.NET. Par exemple, pour forcer une exception si le JSON stocké contient un champ qui n’est pas présent dans la classe, spécifiez l’attribut `JsonObject(MissingMemberHandling = MissingMemberHandling.Error)`. Vous pouvez également écrire pour la désérialisation du code personnalisé qui peut lire les JSON stockés dans des formats arbitraires.

## <a name="entity-construction"></a>Construction des entités

Parfois, nous souhaitons exercer davantage de contrôle sur la manière dont les objets d’entités sont construits. Nous décrivons à présent plusieurs options qui permettent de changer le comportement par défaut au moment de la construction d’objets d’entités. 

### <a name="custom-initialization-on-first-access"></a>Initialisation personnalisée au premier accès

Parfois, nous devons effectuer une initialisation spéciale avant de distribuer une opération à une entité qui n’a jamais fait l’objet d’un accès ou qui a été supprimée. Pour spécifier ce comportement, vous pouvez ajouter une condition avant `DispatchAsync` :

```csharp
[FunctionName(nameof(Counter))]
public static Task Run([EntityTrigger] IDurableEntityContext ctx)
{
    if (!ctx.HasState)
    {
        ctx.SetState(...);
    }
    return ctx.DispatchAsync<Counter>();
}
```

### <a name="bindings-in-entity-classes"></a>Liaisons dans les classes d’entités

Contrairement aux fonctions classiques, les méthodes de classe d’entité n’ont pas d’accès direct aux liaisons d’entrée et de sortie. Au lieu de cela, les données de liaison doivent être capturées dans la déclaration de fonction de point d’entrée, puis être passées à la méthode `DispatchAsync<T>`. Tout objet passé à `DispatchAsync<T>` est automatiquement passé dans le constructeur de classe d’entité en tant qu’argument.

L’exemple suivant montre comment une référence `CloudBlobContainer` à partir de la [liaison d’entrée d’objet blob](../functions-bindings-storage-blob-input.md) peut être rendue disponible pour une entité basée sur une classe.

```csharp
public class BlobBackedEntity
{
    [JsonIgnore]
    private readonly CloudBlobContainer container;

    public BlobBackedEntity(CloudBlobContainer container)
    {
        this.container = container;
    }

    // ... entity methods can use this.container in their implementations ...
    
    [FunctionName(nameof(BlobBackedEntity))]
    public static Task Run(
        [EntityTrigger] IDurableEntityContext context,
        [Blob("my-container", FileAccess.Read)] CloudBlobContainer container)
    {
        // passing the binding object as a parameter makes it available to the
        // entity class constructor
        return context.DispatchAsync<BlobBackedEntity>(container);
    }
}
```

Pour plus d’informations sur les liaisons dans Azure Functions, consultez la documentation [Déclencheurs et liaisons Azure Functions](../functions-triggers-bindings.md).

### <a name="dependency-injection-in-entity-classes"></a>Injection de dépendance dans les classes d’entités

Les classes d’entité prennent en charge l’[injection de dépendances Azure Functions](../functions-dotnet-dependency-injection.md). L’exemple suivant montre comment inscrire un service `IHttpClientFactory` dans une entité basée sur une classe.

```csharp
[assembly: FunctionsStartup(typeof(MyNamespace.Startup))]

namespace MyNamespace
{
    public class Startup : FunctionsStartup
    {
        public override void Configure(IFunctionsHostBuilder builder)
        {
            builder.Services.AddHttpClient();
        }
    }
}
```

L’extrait de code suivant montre comment incorporer le service injecté dans votre classe d’entité.

```csharp
public class HttpEntity
{
    [JsonIgnore]
    private readonly HttpClient client;

    public HttpEntity(IHttpClientFactory factory)
    {
        this.client = factory.CreateClient();
    }

    public Task<int> GetAsync(string url)
    {
        using (var response = await this.client.GetAsync(url))
        {
            return (int)response.StatusCode;
        }
    }

    [FunctionName(nameof(HttpEntity))]
    public static Task Run([EntityTrigger] IDurableEntityContext ctx)
        => ctx.DispatchAsync<HttpEntity>();
}
```

> [!NOTE]
> Pour éviter les problèmes de sérialisation, veillez à exclure de la sérialisation les champs destinés à stocker les valeurs injectées.

> [!NOTE]
> Contrairement à l’utilisation de l’injection de constructeur dans .NET Azure Functions standard, la méthode de point d’entrée des fonctions pour les entités basées sur des classes *doit* être déclarée `static`. La déclaration d’un point d’entrée de fonction non statique peut entraîner des conflits entre l’initialiseur d’objet Azure Functions normal et l’initialiseur d’objet Entités durables.

## <a name="function-based-syntax"></a>Syntaxe basée sur les fonctions

Jusqu’ici, nous nous sommes concentrés sur la syntaxe basée sur les classes, car nous pensons qu’elle est mieux adaptée à la plupart des applications. Toutefois, la syntaxe basée sur les fonctions peut convenir aux applications qui souhaitent définir ou gérer leurs propres abstractions pour l’état et les opérations de l’entité. De plus, elle peut également être appropriée pour l’implémentation des bibliothèques qui nécessitent un état générique non pris en charge par la syntaxe basée sur les classes. 

Avec la syntaxe basée sur les fonctions, la fonction d’entité prend en charge explicitement la répartition des opérations et gère explicitement l’état de l’entité. Par exemple, le code suivant montre l’implémentation de l’entité *Counter* à l’aide de la syntaxe basée sur les fonctions.  

```csharp
[FunctionName("Counter")]
public static void Counter([EntityTrigger] IDurableEntityContext ctx)
{
    switch (ctx.OperationName.ToLowerInvariant())
    {
        case "add":
            ctx.SetState(ctx.GetState<int>() + ctx.GetInput<int>());
            break;
        case "reset":
            ctx.SetState(0);
            break;
        case "get":
            ctx.Return(ctx.GetState<int>());
            break;
        case "delete":
            ctx.DeleteState();
            break;
    }
}
```

### <a name="the-entity-context-object"></a>Objet de contexte d’entité

Les fonctionnalités spécifiques aux entités sont accessibles via un objet de contexte de type `IDurableEntityContext`. Cet objet de contexte est disponible en tant que paramètre de la fonction d’entité et via la propriété asynchrone locale `Entity.Current`.

Les membres suivants fournissent des informations sur l’opération en cours et nous permettent de spécifier une valeur retournée. 

* `EntityName` : nom de l’entité en cours d’exécution.
* `EntityKey` : clé de l’entité en cours d’exécution.
* `EntityId` : ID de l’entité en cours d’exécution (inclut le nom et la clé).
* `OperationName` : nom de l’opération en cours.
* `GetInput<TInput>()` : obtient l’entrée de l’opération en cours.
* `Return(arg)` : retourne une valeur à l’orchestration qui a appelé l’opération.

Les membres suivants gèrent l’état de l’entité (créer, lire, mettre à jour, supprimer). 

* `HasState` : indique si l’entité existe, autrement dit, si elle a un état. 
* `GetState<TState>()` : obtient l’état actuel de l’entité. S’il n’existe pas déjà, il est créé.
* `SetState(arg)` : crée ou met à jour l’état de l’entité.
* `DeleteState()` : supprime l’état de l’entité, s’il existe. 

Si l’état retourné par `GetState` est un objet, il peut être modifié directement par le code d’application. Il n’est pas nécessaire de rappeler `SetState` à la fin (mais cela ne pose pas de problème non plus). Si `GetState<TState>` est appelé plusieurs fois, le même type doit être utilisé.

Enfin, les membres suivants sont utilisés pour signaler d’autres entités ou pour démarrer de nouvelles orchestrations :

* `SignalEntity(EntityId, operation, input)` : envoie un message unidirectionnel à une entité.
* `CreateNewOrchestration(orchestratorFunctionName, input)` : démarre une nouvelle orchestration.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur les concepts d’entité](durable-functions-entities.md)
