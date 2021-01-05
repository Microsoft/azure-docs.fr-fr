---
title: Liaisons pour Fonctions durables - Azure
description: Guide pratique pour utiliser des déclencheurs et des liaisons pour l’extension Durable Functions pour Azure Functions.
ms.topic: conceptual
ms.date: 12/17/2019
ms.author: azfuncdf
ms.openlocfilehash: 899bc3fdc94b8232acd3edf3e0cbab3c481ff8f2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013721"
---
# <a name="bindings-for-durable-functions-azure-functions"></a>Liaisons pour Fonctions durables (Azure Functions)

L’extension [Fonctions durables](durable-functions-overview.md) introduit deux nouvelles liaisons de déclencheur qui contrôlent l’exécution des fonctions d’orchestrateur et d’activité. Elle introduit également une liaison de sortie qui agit comme un client pour l’exécution de Fonctions durables.

## <a name="orchestration-trigger"></a>Déclencheur d'orchestration

Le déclencheur d’orchestration vous permet de créer des [fonctions d’orchestrateur durables](durable-functions-types-features-overview.md#orchestrator-functions). Ce déclencheur prend en charge le démarrage de nouvelles instances de fonctions d’orchestrateur et la reprise d’instances de fonctions d’orchestrateur existantes « en attente » d’une tâche.

Lorsque vous utilisez les outils Visual Studio pour Azure Functions, le déclencheur d’orchestration est configuré à l’aide de l’attribut .NET [OrchestrationTriggerAttribute](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.orchestrationtriggerattribute?view=azure-dotnet).

Lorsque vous écrivez des fonctions d’orchestrateur dans des langages de script (par exemple, JavaScript ou C#), le déclencheur d’orchestration est défini par l’objet JSON suivant dans le tableau `bindings` du fichier *function.json* :

```json
{
    "name": "<Name of input parameter in function signature>",
    "orchestration": "<Optional - name of the orchestration>",
    "type": "orchestrationTrigger",
    "direction": "in"
}
```

* `orchestration` correspond au nom de l’orchestration que les clients doivent utiliser lorsqu’ils souhaitent démarrer de nouvelles instances de cette fonction d’orchestrateur. Cette propriété est facultative. Si cette valeur n’est pas spécifiée, le nom de la fonction est utilisé.

En interne, cette liaison de déclencheur interroge une série de files d’attente dans le compte de stockage par défaut pour l’application de la fonction. Ces files d’attente sont des détails d’implémentation internes de l’extension, et elles ne sont donc pas explicitement configurées dans les propriétés de liaison.

### <a name="trigger-behavior"></a>Comportement du déclencheur

Voici quelques remarques concernant le déclencheur d’orchestration :

* **Thread unique** : un thread de répartiteur unique est utilisé pour toutes les exécutions d’une fonction d’orchestrateur sur une même instance d’hôte. Pour cette raison, il est important de s’assurer que le code de la fonction d’orchestrateur est efficace et n’effectue aucune opération E/S. Il est également important de s’assurer que ce thread n’effectue aucun travail asynchrone, sauf lorsqu’il attend des types de tâches Fonctions durables spécifiques.
* **Gestion des messages incohérents** : les déclencheurs d’orchestration ne prennent en charge aucun message incohérent.
* **Visibilité des messages** : les messages du déclencheur d’orchestration sont supprimés de la file d’attente et restent invisibles pour une durée configurable. La visibilité de ces messages est renouvelée automatiquement tant que l’application de la fonction est en cours d’exécution et saine.
* **Valeurs de retour** : les valeurs de retour sont sérialisées au format JSON et conservées dans la table d’historique d’orchestration du stockage Azure Table. Ces valeurs de retour peuvent être interrogées par la liaison du client d’orchestration, décrite plus loin.

> [!WARNING]
> Les fonctions d’orchestrateur ne doivent jamais utiliser une liaison d’entrée ou de sortie autre que la liaison du déclencheur d’orchestration. Cela pourrait entraîner des problèmes avec l’extension Tâche durable car ces liaisons risquent de ne pas respecter les règles de thread unique et d’E/S. Si vous souhaitez utiliser d’autres liaisons, ajoutez-les à une fonction d’activité appelée à partir de votre fonction d’orchestrateur.

> [!WARNING]
> Les fonctions d’orchestrateur JavaScript ne doivent jamais être déclarées avec la propriété `async`.

### <a name="trigger-usage-net"></a>Utilisation du déclencheur (.NET)

La liaison du déclencheur d’orchestration prend en charge à la fois les entrées et les sorties. Voici quelques éléments à connaître concernant la gestion des entrées et des sorties :

* **entrées** : les fonctions d’orchestration .NET prennent uniquement en charge le type de paramètre `DurableOrchestrationContext`. Les entrées de désérialisation directement dans la signature de la fonction ne sont pas prises en charge. Le code doit utiliser la méthode `GetInput<T>` (.NET) ou `getInput` (JavaScript) pour extraire des entrées de la fonction d’orchestrateur. Ces entrées doivent être de type JSON sérialisable.
* **sorties** : les déclencheurs d’orchestration prennent en charge les valeurs de sortie ainsi que les entrées. La valeur de retour de la fonction sert à affecter la valeur de sortie et doit être de type JSON sérialisable. Si une fonction .NET retourne `Task` ou `void`, une valeur `null` est enregistrée en tant que sortie.

### <a name="trigger-sample"></a>Exemple de déclencheur

L'exemple de code suivant illustre une simple fonction d’orchestrateur « Hello World » :

#### <a name="c"></a>C#

```csharp
[FunctionName("HelloWorld")]
public static string Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string name = context.GetInput<string>();
    return $"Hello {name}!";
}
```
> [!NOTE]
> Le code précédent correspond à Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser `DurableOrchestrationContext` au lieu de `IDurableOrchestrationContext`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

#### <a name="javascript-functions-20-only"></a>JavaScript (Functions 2.0 uniquement)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const name = context.df.getInput();
    return `Hello ${name}!`;
});
```

> [!NOTE]
> L’objet `context` dans JavaScript représente non pas le contexte d’orchestration durable mais le contexte de fonction [ dans son ensemble](../functions-reference-node.md#context-object). Vous pouvez accéder à des méthodes d’orchestration via la propriété `df` de l’objet `context`.

> [!NOTE]
> Les orchestrateurs JavaScript doivent utiliser `return`. La bibliothèque `durable-functions` prend en charge l’appel de la méthode `context.done`.

La plupart des fonctions d’orchestrateur appellent des fonctions d’activité. Voici un exemple « Hello World » qui montre comment appeler une fonction d’activité :

#### <a name="c"></a>C#

```csharp
[FunctionName("HelloWorld")]
public static async Task<string> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string name = context.GetInput<string>();
    string result = await context.CallActivityAsync<string>("SayHello", name);
    return result;
}
```

> [!NOTE]
> Le code précédent correspond à Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser `DurableOrchestrationContext` au lieu de `IDurableOrchestrationContext`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

#### <a name="javascript-functions-20-only"></a>JavaScript (Functions 2.0 uniquement)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const name = context.df.getInput();
    const result = yield context.df.callActivity("SayHello", name);
    return result;
});
```

## <a name="activity-trigger"></a>Déclencheur d'activité

Le déclencheur d’activité vous permet de créer des fonctions appelées par des fonctions d’orchestrateur, appelées [fonctions d'activité](durable-functions-types-features-overview.md#activity-functions).

Si vous utilisez Visual Studio, le déclencheur d’activité est configuré à l’aide de l’attribut .NET `ActivityTriggerAttribute`.

Si vous utilisez VS Code ou le portail Azure pour le développement, le déclencheur d’activité est défini par l’objet JSON suivant dans le tableau `bindings` de *function.json* :

```json
{
    "name": "<Name of input parameter in function signature>",
    "activity": "<Optional - name of the activity>",
    "type": "activityTrigger",
    "direction": "in"
}
```

* `activity` est le nom de l’activité. Cette valeur correspond au nom qu’utilisent les fonctions d’orchestrateur pour appeler cette fonction d’activité. Cette propriété est facultative. Si cette valeur n’est pas spécifiée, le nom de la fonction est utilisé.

En interne, cette liaison de déclencheur interroge une file d’attente dans le compte de stockage par défaut pour l’application de la fonction. Cette file d’attente est un détail d’implémentation interne de l’extension, et elle n’est donc pas explicitement configurée dans les propriétés de liaison.

### <a name="trigger-behavior"></a>Comportement du déclencheur

Voici quelques remarques concernant le déclencheur d’activité :

* **Threading** : contrairement au déclencheur d’orchestration, les déclencheurs d’activité ne sont soumis à aucune restriction au niveau du thread ou des opérations d’E/S. Ils peuvent être traités comme des fonctions normales.
* **Gestion des messages incohérents** : les déclencheurs d’activité ne prennent en charge aucun message incohérent.
* **Visibilité des messages** : les messages du déclencheur d’activité sont supprimés de la file d’attente et restent invisibles pour une durée configurable. La visibilité de ces messages est renouvelée automatiquement tant que l’application de la fonction est en cours d’exécution et saine.
* **Valeurs de retour** : les valeurs de retour sont sérialisées au format JSON et conservées dans la table d’historique d’orchestration du stockage Azure Table.

> [!WARNING]
> Le serveur principal de stockage pour les fonctions de l’activité est un détail d’implémentation, et le code utilisateur ne doit pas interagir directement avec ces entités de stockage.

### <a name="trigger-usage-net"></a>Utilisation du déclencheur (.NET)

La liaison du déclencheur d’activité prend en charge à la fois les entrées et les sorties, exactement comme le déclencheur d’orchestration. Voici quelques éléments à connaître concernant la gestion des entrées et des sorties :

* **entrées** : les fonctions d’activité .NET natives utilisent le type de paramètre `DurableActivityContext`. Une fonction d’activité peut également être déclarée avec n’importe quel type de paramètre sérialisable au format JSON. Lorsque vous utilisez `DurableActivityContext`, vous pouvez appeler `GetInput<T>` pour extraire et désérialiser l’entrée de la fonction d’activité.
* **sorties** : les fonctions d’activité prennent en charge les valeurs de sortie ainsi que les entrées. La valeur de retour de la fonction sert à affecter la valeur de sortie et doit être de type JSON sérialisable. Si une fonction .NET retourne `Task` ou `void`, une valeur `null` est enregistrée en tant que sortie.
* **métadonnées** : les fonctions d’activité .NET peuvent être liées à un paramètre `string instanceId` pour obtenir l’ID d’instance de l’orchestration parente.

### <a name="trigger-sample"></a>Exemple de déclencheur

L'exemple de code suivant illustre une simple fonction d’activité « Hello World » :

#### <a name="c"></a>C#

```csharp
[FunctionName("SayHello")]
public static string SayHello([ActivityTrigger] IDurableActivityContext helloContext)
{
    string name = helloContext.GetInput<string>();
    return $"Hello {name}!";
}
```

> [!NOTE]
> Le code précédent correspond à Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser `DurableActivityContext` au lieu de `IDurableActivityContext`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

Le type de paramètre par défaut pour la liaison `ActivityTriggerAttribute` .NET est `IDurableActivityContext`. Cependant, les déclencheurs d’activité .NET prennent également en charge la liaison directe avec des types sérialisables au format JSON (notamment les types primitifs). La même fonction peut donc être simplifiée comme suit :

```csharp
[FunctionName("SayHello")]
public static string SayHello([ActivityTrigger] string name)
{
    return $"Hello {name}!";
}
```

#### <a name="javascript-functions-20-only"></a>JavaScript (Functions 2.0 uniquement)

```javascript
module.exports = async function(context) {
    return `Hello ${context.bindings.name}!`;
};
```

Les liaisons JavaScript peuvent également être transmises en tant que paramètres supplémentaires. La même fonction peut donc être simplifiée comme suit :

```javascript
module.exports = async function(context, name) {
    return `Hello ${name}!`;
};
```


### <a name="using-input-and-output-bindings"></a>Utilisation des liaisons d’entrée et de sortie

Vous pouvez utiliser des liaisons d’entrée et de sortie standard en plus de la liaison de déclencheur d’activité. Par exemple, vous pouvez prendre l’entrée de votre liaison d’activité et envoyer un message à un EventHub à l’aide de la liaison de sortie EventHub :

```json
{
  "bindings": [
    {
      "name": "message",
      "type": "activityTrigger",
      "direction": "in"
    },
    {
      "type": "eventHub",
      "name": "outputEventHubMessage",
      "connection": "EventhubConnectionSetting",
      "eventHubName": "eh_messages",
      "direction": "out"
  }
  ]
}
```

```javascript
module.exports = async function (context) {
    context.bindings.outputEventHubMessage = context.bindings.message;
};
```

## <a name="orchestration-client"></a>Client d’orchestration

La liaison du client d’orchestration vous permet d’écrire des fonctions qui interagissent avec les fonctions d’orchestrateur. Ces fonctions sont parfois appelées [fonctions clientes](durable-functions-types-features-overview.md#client-functions). Par exemple, vous pouvez agir sur les instances de l’orchestration de l’une des manières suivantes :

* Les démarrer.
* Interroger leur état.
* Les arrêter.
* Leur envoyer des événements pendant qu’elles sont exécutées.
* Purger l’historique d’instance.

Si vous utilisez Visual Studio, vous pouvez établir une liaison avec le client d’orchestration en utilisant l’attribut .NET `OrchestrationClientAttribute` pour Durable Functions 1.0. À partir de Durable Functions 2.0, vous pouvez lier le client d’orchestration à l’aide de l’attribut `DurableClientAttribute` .NET.

Si vous utilisez des scripts de langage (par exemple, des fichiers *.csx* ou *.js*) pour le développement, le déclencheur d’orchestration est défini par l’objet JSON suivant dans le `bindings`tableau *du fichier function.json* :

```json
{
    "name": "<Name of input parameter in function signature>",
    "taskHub": "<Optional - name of the task hub>",
    "connectionName": "<Optional - name of the connection string app setting>",
    "type": "orchestrationClient",
    "direction": "in"
}
```

* `taskHub` : utilisé dans les scénarios où plusieurs applications de fonction partagent le même compte de stockage, mais doivent être isolées les unes des autres. Si ce champ n’est pas spécifié, la valeur `host.json` sera utilisée par défaut. Cette valeur doit correspondre à la valeur utilisée par les fonctions d’orchestrateur cible.
* `connectionName` : nom d’un paramètre d’application contenant une chaîne de connexion du compte de stockage. Le compte de stockage représenté par cette chaîne de connexion doit être le même que celui utilisé par les fonctions d’orchestrateur cible. Si elle n’est pas spécifiée, la chaîne de connexion du compte de stockage par défaut pour l’application de la fonction est utilisée.

> [!NOTE]
> Dans la plupart des cas, nous vous recommandons d’omettre ces propriétés et de vous appuyer sur le comportement par défaut.

### <a name="client-usage"></a>Utilisation du client

Dans les fonctions .NET, vous établissez généralement une liaison avec `IDurableOrchestrationClient`, ce qui vous donne un accès total à toutes les API clientes d’orchestration prises en charge par Durable Functions. Dans les anciennes versions de Durable Functions 2.x, vous pouviez créer une liaison avec la classe `DurableOrchestrationClient`. Dans JavaScript, les mêmes API sont exposées par l’objet renvoyé à partir de `getClient`. Les API sur l’objet client incluent :

* `StartNewAsync`
* `GetStatusAsync`
* `TerminateAsync`
* `RaiseEventAsync`
* `PurgeInstanceHistoryAsync`
* `CreateCheckStatusResponse`
* `CreateHttpManagementPayload`

Des liaisons .NET peuvent également être établies avec `IAsyncCollector<T>`, où `T` est `StartOrchestrationArgs` ou `JObject`.

Pour plus d’informations sur ces opérations, consultez la documentation de l’API `IDurableOrchestrationClient`.

### <a name="client-sample-visual-studio-development"></a>Exemple de client (développement Visual Studio)

Voici l’exemple d’une fonction déclenchée par une file d’attente qui démarre une orchestration « HelloWorld ».

```csharp
[FunctionName("QueueStart")]
public static Task Run(
    [QueueTrigger("durable-function-trigger")] string input,
    [DurableClient] IDurableOrchestrationClient starter)
{
    // Orchestration input comes from the queue message content.
    return starter.StartNewAsync("HelloWorld", input);
}
```

> [!NOTE]
> Le code C# précédent correspond à Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser l’attribut `OrchestrationClient` au lieu de l’attribut `DurableClient`, et vous devez utiliser le type de paramètre `DurableOrchestrationClient` au lieu de `IDurableOrchestrationClient`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

### <a name="client-sample-not-visual-studio"></a>Exemple de client (sans Visual Studio)

Si vous n’utilisez pas Visual Studio pour le développement, vous pouvez créer le fichier *function.json* suivant. Cet exemple montre comment configurer une fonction déclenchée par une file d’attente, qui utilise la liaison du client d’orchestration durable :

```json
{
  "bindings": [
    {
      "name": "input",
      "type": "queueTrigger",
      "queueName": "durable-function-trigger",
      "direction": "in"
    },
    {
      "name": "starter",
      "type": "durableClient",
      "direction": "in"
    }
  ]
}
```

> [!NOTE]
> Le JSON précédent correspond à Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser `orchestrationClient` au lieu de `durableClient` en tant que type de déclencheur. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

Voici des exemples spécifiques à un langage, qui démarrent de nouvelles instances de fonction d’orchestrateur.

#### <a name="c-script-sample"></a>Exemple de script C#

L’exemple suivant montre comment utiliser la liaison de client d’orchestration durable pour démarrer une nouvelle instance de fonction à partir d’une fonction C# déclenchée par une file d’attente :

```csharp
#r "Microsoft.Azure.WebJobs.Extensions.DurableTask"

using Microsoft.Azure.WebJobs.Extensions.DurableTask;

public static Task Run(string input, IDurableOrchestrationClient starter)
{
    return starter.StartNewAsync("HelloWorld", input);
}
```

> [!NOTE]
> Le code précédent correspond à Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser le type de paramètre `DurableOrchestrationClient` au lieu de `IDurableOrchestrationClient`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

#### <a name="javascript-sample"></a>Exemple JavaScript

L’exemple suivant montre comment utiliser la liaison de client d’orchestration durable pour démarrer une nouvelle instance de fonction à partir d’une fonction JavaScript :

```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    return instanceId = await client.startNew("HelloWorld", undefined, context.bindings.input);
};
```

Vous trouverez plus d’informations sur le démarrage des instances dans la section [Gestion d’instance](durable-functions-instance-management.md).

## <a name="entity-trigger"></a>Déclencheur d’entité

Les déclencheurs d’entité vous permettent de créer des [fonctions d'entité](durable-functions-entities.md). Ce déclencheur prend en charge le traitement des événements pour une instance d’entité spécifique.

Lorsque vous utilisez les outils Visual Studio pour Azure Functions, le déclencheur d’entité est configuré à l’aide de l’attribut .NET `EntityTriggerAttribute`.

> [!NOTE]
> Les déclencheurs d’entité sont disponibles à partir de la version Durable Functions 2.x.

En interne, cette liaison de déclencheur interroge une série de files d’attente dans le compte de stockage par défaut pour l’application de la fonction. Ces files d’attente sont des détails d’implémentation internes de l’extension, et elles ne sont donc pas explicitement configurées dans les propriétés de liaison.

### <a name="trigger-behavior"></a>Comportement du déclencheur

Voici quelques remarques concernant le déclencheur d’entité :

* **Thread unique** : un thread de répartiteur unique est utilisé pour traiter les opérations d’une entité donnée. Si plusieurs messages sont envoyés simultanément à une même entité, les opérations sont traitées tour à tour.
* **Gestion des messages incohérents** : les déclencheurs d’entité ne prennent en charge aucun message incohérent.
* **Visibilité des messages** : les messages du déclencheur d’entité sont supprimés de la file d’attente et restent invisibles pour une durée configurable. La visibilité de ces messages est renouvelée automatiquement tant que l’application de la fonction est en cours d’exécution et saine.
* **Valeurs de retour** : les fonctions d’entité ne prennent pas en charge les valeurs de retour. Des API spécifiques peuvent être utilisées pour enregistrer l’état ou transmettre les valeurs aux orchestrations.

Toute modification d’état apportée à une entité lors de son exécution perdure automatiquement au terme de l'exécution.

### <a name="trigger-usage-net"></a>Utilisation du déclencheur (.NET)

Chaque fonction d’entité dispose d'un type de paramètre de `IDurableEntityContext`, qui a les membres suivants :

* **EntityName** : nom de l’entité en cours d’exécution.
* **EntityKey** : clé de l’entité en cours d’exécution.
* **EntityId** : ID de l’entité en cours d’exécution.
* **OperationName** : nom de l’opération en cours.
* **HasState** : indique si l’entité existe, c’est-à-dire si elle a un état. 
* **GetState\<TState>()**  : obtient l’état actuel de l’entité. S’il n’existe pas déjà, il est créé et initialisé pour `default<TState>`. Le paramètre `TState` doit correspondre à un type primitif ou sérialisable JSON. 
* **GetState\<TState>(initfunction)**  : obtient l’état actuel de l’entité. S’il n’existe pas déjà, il est créé en appelant le paramètre `initfunction` fourni. Le paramètre `TState` doit correspondre à un type primitif ou sérialisable JSON. 
* **SetState(arg)**  : crée ou met à jour l’état de l’entité. Le paramètre `arg` doit être un objet ou primitif sérialisable JSON.
* **DeleteState()**  : supprime l’état de l’entité. 
* **GetInput\<TInput>()**  : obtient l’entrée pour l’opération en cours. Le paramètre de type `TInput` doit correspondre à un type primitif ou sérialisable JSON.
* **Return(arg)**  : renvoie une valeur à l'orchestration qui a appelé l’opération. Le paramètre `arg` doit correspondre à un objet primitif ou sérialisable JSON.
* **SignalEntity(EntityId, scheduledTimeUtc, operation, input)**  : envoie un message unidirectionnel à une entité. Le paramètre `operation` doit être une chaîne non null, le `scheduledTimeUtc` facultatif doit être une valeur de date et d’heure UTC pour l’appel de l’opération, et le paramètre `input` doit être un objet primitif ou sérialisable par du code JSON.
* **CreateNewOrchestration(orchestratorFunctionName, input)** : démarre une nouvelle orchestration. Le paramètre `input` doit correspondre à un objet primitif ou sérialisable JSON.

L’objet `IDurableEntityContext` passé à la fonction d’entité est accessible à l’aide de la propriété async-local `Entity.Current`. Cette approche est pratique lorsque vous utilisez le modèle de programmation basé sur une classe.

### <a name="trigger-sample-c-function-based-syntax"></a>Exemple de déclencheur (syntaxe basée sur une fonction C#)

Le code suivant est un exemple d’entité *Counter* simple implémentée en tant que fonction durable. Cette fonction définit trois opérations (`add`, `reset` et `get`), chacune d’elles opérant sur un état d’entier.

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
            ctx.Return(ctx.GetState<int>()));
            break;
    }
}
```

Pour plus d’informations sur la syntaxe basée sur la fonction et sur son utilisation, consultez [Syntaxe basée sur la fonction](durable-functions-dotnet-entities.md#function-based-syntax).

### <a name="trigger-sample-c-class-based-syntax"></a>Exemple de déclencheur (syntaxe basée sur une classe C#)

L’exemple suivant est une implémentation équivalente de l’entité `Counter` à l’aide de classes et de méthodes.

```csharp
[JsonObject(MemberSerialization.OptIn)]
public class Counter
{
    [JsonProperty("value")]
    public int CurrentValue { get; set; }

    public void Add(int amount) => this.CurrentValue += amount;

    public void Reset() => this.CurrentValue = 0;

    public int Get() => this.CurrentValue;

    [FunctionName(nameof(Counter))]
    public static Task Run([EntityTrigger] IDurableEntityContext ctx)
        => ctx.DispatchAsync<Counter>();
}
```

L’état de cette entité est un objet de type `Counter`, qui contient un champ qui stocke la valeur actuelle du compteur. Pour rendre cet objet persistant dans le stockage, il est sérialisé et désérialisé par la bibliothèque [Json.NET](https://www.newtonsoft.com/json). 

Pour plus d’informations sur la syntaxe basée sur la classe et sur son utilisation, consultez [Définition des classes d’entités](durable-functions-dotnet-entities.md#defining-entity-classes).

> [!NOTE]
> La méthode de point d’entrée de la fonction avec l’attribut `[FunctionName]`*doit* être déclarée `static` lors de l’utilisation de classes d’entité. Les méthodes de point d’entrée non statiques peuvent entraîner l’initialisation de plusieurs objets et éventuellement d’autres comportements non définis.

Les classes d’entité disposent des mécanismes spéciaux pour interagir avec les liaisons et l’injection de dépendance .NET. Pour plus d’informations, voir [Construction d’entité](durable-functions-dotnet-entities.md#entity-construction).

### <a name="trigger-sample-javascript"></a>Exemple de déclencheur (JavaScript)

Le code suivant est un exemple d’entité *Counter* simple implémentée en tant que fonction durable écrite en JavaScript. Cette fonction définit trois opérations (`add`, `reset` et `get`), chacune d’elles opérant sur un état d’entier.

**function.json**
```json
{
  "bindings": [
    {
      "name": "context",
      "type": "entityTrigger",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

**index.js**
```javascript
const df = require("durable-functions");

module.exports = df.entity(function(context) {
    const currentValue = context.df.getState(() => 0);
    switch (context.df.operationName) {
        case "add":
            const amount = context.df.getInput();
            context.df.setState(currentValue + amount);
            break;
        case "reset":
            context.df.setState(0);
            break;
        case "get":
            context.df.return(currentValue);
            break;
    }
});
```

> [!NOTE]
> Les entités durables sont disponibles dans JavaScript à partir de la version **1.3.0** du package npm `durable-functions`.

## <a name="entity-client"></a>Client d'entité

La liaison du client d’entité vous permet de déclencher des [fonctions d'entité](#entity-trigger) de manière asynchrone. Ces fonctions sont parfois appelées [fonctions clientes](durable-functions-types-features-overview.md#client-functions).

Si vous utilisez Visual Studio, vous pouvez établir une liaison avec le client d’entité en utilisant l’attribut .NET `DurableClientAttribute`.

> [!NOTE]
> `[DurableClientAttribute]` permet de créer une liaison avec le [client d'orchestration](#orchestration-client).

Si vous utilisez des scripts de langage (par exemple, des fichiers *.csx* ou *.js*) pour le développement, le déclencheur d’entité est défini par l’objet JSON suivant dans le `bindings`tableau *du fichier function.json* :

```json
{
    "name": "<Name of input parameter in function signature>",
    "taskHub": "<Optional - name of the task hub>",
    "connectionName": "<Optional - name of the connection string app setting>",
    "type": "durableClient",
    "direction": "in"
}
```

* `taskHub` : utilisé dans les scénarios où plusieurs applications de fonction partagent le même compte de stockage, mais doivent être isolées les unes des autres. Si ce champ n’est pas spécifié, la valeur `host.json` sera utilisée par défaut. Cette valeur doit correspondre à la valeur utilisée par les fonctions d’entité cible.
* `connectionName` : nom d’un paramètre d’application contenant une chaîne de connexion du compte de stockage. Le compte de stockage représenté par cette chaîne de connexion doit être le même que celui utilisé par les fonctions d’entité cible. Si elle n’est pas spécifiée, la chaîne de connexion du compte de stockage par défaut pour l’application de la fonction est utilisée.

> [!NOTE]
> Dans la plupart des cas, nous vous recommandons d’omettre les propriétés facultatives et de vous appuyer sur le comportement par défaut.

### <a name="entity-client-usage"></a>Utilisation du client d'entité

Dans les fonctions .NET, vous établissez généralement une liaison avec `IDurableEntityClient`, ce qui vous donne un accès total à toutes les API clientes prises en charge par les entités durables. Vous pouvez également créer une liaison avec l'interface`IDurableOrchestrationClient`, qui fournit l’accès aux API clientes pour les entités et les orchestrations. Les API sur l’objet client incluent :

* **ReadEntityStateAsync\<T>**  : lit l’état d’une entité. Ceci retourne une réponse qui indique si l’entité cible existe et, si tel est le cas, son état.
* **SignalEntityAsync** : Envoie un message unidirectionnel à une entité et attend qu’elle soit mise en file d’attente.
* **ListEntitiesAsync** : interroge l’état de plusieurs entités. Les entités peuvent être interrogées selon leur *nom* ou l’*heure de la dernière opération*.

Il n’est pas nécessaire de créer l’entité cible avant d’envoyer un signal ; l’état de l’entité peut être créé à partir de la fonction d’entité qui gère le signal.

> [!NOTE]
> Il est important de comprendre que les « signaux » envoyés à partir du client sont simplement mis en file d’attente pour être traités de façon asynchrone ultérieurement. En particulier, `SignalEntityAsync` retourne généralement avant même que l’entité démarre l’opération, et il n’est pas possible de récupérer la valeur de retour ou d’observer les exceptions. Si des garanties plus fortes sont requises (par exemple, pour les flux de travail), les *fonctions d’orchestrateur* doivent être utilisées, car elles peuvent attendre la fin des opérations d’entité, et traiter des valeurs de retour et observer des exceptions.

### <a name="example-client-signals-entity-directly---c"></a>Exemple : le client signale une entité directement - C#

Voici un exemple de fonction déclenchée par une file d’attente qui appelle une entité « Compteur ».

```csharp
[FunctionName("AddFromQueue")]
public static Task Run(
    [QueueTrigger("durable-function-trigger")] string input,
    [DurableClient] IDurableEntityClient client)
{
    // Entity operation input comes from the queue message content.
    var entityId = new EntityId(nameof(Counter), "myCounter");
    int amount = int.Parse(input);
    return client.SignalEntityAsync(entityId, "Add", amount);
}
```

### <a name="example-client-signals-entity-via-interface---c"></a>Exemple : le client signale une entité via l’interface - C#

Lorsque cela est possible, nous recommandons [d’accéder aux entités par le biais d’interfaces](durable-functions-dotnet-entities.md#accessing-entities-through-interfaces), car cela procure davantage de contrôle de type. Par exemple, supposons que l’entité `Counter` mentionnée précédemment implémentait une interface `ICounter`, définie comme suit :

```csharp
public interface ICounter
{
    void Add(int amount);
    void Reset();
    Task<int> Get();
}

public class Counter : ICounter
{
    // ...
}
```

Le code client peut ensuite utiliser `SignalEntityAsync<ICounter>` pour générer un proxy de type sécurisé :

```csharp
[FunctionName("UserDeleteAvailable")]
public static async Task AddValueClient(
    [QueueTrigger("my-queue")] string message,
    [DurableClient] IDurableEntityClient client)
{
    var target = new EntityId(nameof(Counter), "myCounter");
    int amount = int.Parse(message);
    await client.SignalEntityAsync<ICounter>(target, proxy => proxy.Add(amount));
}
```

Le paramètre `proxy` est une instance générée dynamiquement de `ICounter`, qui traduit en interne l’appel à `Add` en un appel (non typé) équivalent à `SignalEntityAsync`.

> [!NOTE]
> Les API `SignalEntityAsync` représentent des opérations unidirectionnelles. Si une interface d’entité renvoie `Task<T>`, la valeur du paramètre `T` est systématiquement null ou `default`.

En particulier, il n’est pas judicieux de signaler l’opération `Get`, car aucune valeur n’est retournée. À la place, les clients peuvent utiliser `ReadStateAsync` pour accéder directement à l’état du compteur, ou démarrer une fonction d’orchestrateur qui appelle l’opération `Get`.

### <a name="example-client-signals-entity---javascript"></a>Exemple : le client signale une entité directement - JavaScript

Voici un exemple de fonction déclenchée par une file d’attente qui signale une entité « Compteur » dans JavaScript.

**function.json**
```json
{
    "bindings": [
      {
        "name": "input",
        "type": "queueTrigger",
        "queueName": "durable-entity-trigger",
        "direction": "in",
      },
      {
        "name": "starter",
        "type": "durableClient",
        "direction": "in"
      }
    ],
    "disabled": false
  }
```

**index.js**
```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    const entityId = new df.EntityId("Counter", "myCounter");
    await context.df.signalEntity(entityId, "add", 1);
};
```

> [!NOTE]
> Les entités durables sont disponibles dans JavaScript à partir de la version **1.3.0** du package npm `durable-functions`.

<a name="host-json"></a>
## <a name="hostjson-settings"></a>Paramètres host.json

[!INCLUDE [durabletask](../../../includes/functions-host-json-durabletask.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Référence sur l'API HTTP intégrée pour la gestion des instances](durable-functions-http-api.md)
