---
title: Vue d’ensemble de Fonctions durables - Azure
description: Introduction à l’extension Fonctions durables pour Azure Functions.
author: cgillum
ms.topic: overview
ms.date: 03/12/2020
ms.author: cgillum
ms.reviewer: azfuncdf
ms.openlocfilehash: 28c494bf2867ec5d2d3ee99ef7ee45f8181cfd90
ms.sourcegitcommit: 5d7f8c57eaae91f7d9cf1f4da059006521ed4f9f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/10/2020
ms.locfileid: "89669257"
---
# <a name="what-are-durable-functions"></a>Présentation de Durable Functions

*Durable Functions* est une extension d’[Azure Functions](../functions-overview.md) qui vous permet d’écrire des fonctions avec état dans un environnement de calcul serverless. L’extension vous permet de définir des workflows avec état en écrivant des [*fonctions orchestrator*](durable-functions-orchestrations.md) et des entités avec état en écrivant des [*fonctions d’entité*](durable-functions-entities.md) à l’aide du modèle de programmation Azure Functions. En arrière-plan, l’extension gère l’état, les points de contrôle et les redémarrages à votre place, ce qui vous permet de vous concentrer sur votre logique métier.

## <a name="supported-languages"></a><a name="language-support"></a>Langages pris en charge

Durable Functions prend actuellement en charge les langages suivants :

* **C#**  : les [bibliothèques de classes précompilées](../functions-dotnet-class-library.md) et le [script C#](../functions-reference-csharp.md).
* **JavaScript** : pris en charge uniquement pour la version 2.x du runtime Azure Functions. Nécessite la version 1.7.0 ou ultérieure de l’extension Durable Functions. 
* **Python** : nécessite la version 1.8.5 ou une version ultérieure de l’extension Durable Functions. La prise en charge de Durable Functions est actuellement en préversion publique.
* **F#**  : les bibliothèques de classes précompilées et le script F#. Le script F# est pris en charge uniquement pour la version 1.x du runtime Azure Functions.
* **PowerShell** : la prise en charge de Durable Functions est actuellement en préversion publique. Uniquement pris en charge pour la version 3.x du runtime Azure Functions et PowerShell 7. Nécessite la version 2.2.2 ou ultérieure de l'extension Durable Functions. Actuellement, seuls les modèles suivants sont pris en charge : [Chaînage de fonctions](#chaining), [Fan-out/fan-in](#fan-in-out), [API HTTP Async](#async-http).

À l’avenir, Durable Functions est censé prendre en charge tous les [langages pris en charge dans Azure Functions](../supported-languages.md). Consultez la [liste des problèmes avec Durable Functions](https://github.com/Azure/azure-functions-durable-extension/issues) pour connaître l’état actuel de la prise en charge de langages supplémentaires.

Comme dans Azure Functions, des modèles sont disponibles pour vous aider à développer des fonctions durables en utilisant [Visual Studio 2019](durable-functions-create-first-csharp.md), [Visual Studio Code](quickstart-js-vscode.md) et le [portail Azure](durable-functions-create-portal.md).

## <a name="application-patterns"></a>Modèles d’application

Le principal cas d’usage de Durable Functions est la simplification d’exigences complexes de coordination avec état dans des applications serverless. Les sections suivantes décrivent des modèles d’application standard qui peuvent tirer parti de Durable Functions :

* [Chaînage de fonctions](#chaining)
* [Fan-out/fan-in](#fan-in-out)
* [API HTTP Async](#async-http)
* [Surveillance](#monitoring)
* [Interaction humaine](#human)
* [Agrégateur (entités avec état)](#aggregator)

### <a name="pattern-1-function-chaining"></a>Modèle 1 : chaînage de fonctions

Dans le modèle de chaînage de fonctions, une séquence de fonctions s’exécute dans un ordre spécifique. Dans ce modèle, la sortie d’une fonction est appliquée à l’entrée d’une autre fonction.

![Schéma de modèle de chaînage de fonctions](./media/durable-functions-concepts/function-chaining.png)

Vous pouvez utiliser Durable Functions pour implémenter le modèle de chaînage de fonctions de façon concise, comme indiqué dans l’exemple ci-dessous.

Dans cet exemple, les valeurs `F1`, `F2`, `F3` et `F4` représentent les noms d’autres fonctions dans la même application de fonction. Vous pouvez implémenter le flux de contrôle à l’aide de constructions de codage impératives normales. Le code s’exécute du haut vers le bas. Le code peut impliquer une sémantique de flux contrôle de langage existante, notamment des instructions conditionnelles et des boucles. Vous pouvez inclure une logique de gestion des erreurs dans des blocs `try`/`catch`/`finally`.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("Chaining")]
public static async Task<object> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    try
    {
        var x = await context.CallActivityAsync<object>("F1", null);
        var y = await context.CallActivityAsync<object>("F2", x);
        var z = await context.CallActivityAsync<object>("F3", y);
        return  await context.CallActivityAsync<object>("F4", z);
    }
    catch (Exception)
    {
        // Error handling or compensation goes here.
    }
}
```

Vous pouvez utiliser le paramètre `context` pour appeler d’autres fonctions par nom, passer des paramètres et retourner la sortie d’une fonction. Chaque fois que le code appelle `await`, l’infrastructure Durable Functions crée des points de contrôle de la progression de l’instance de la fonction actuelle. En cas de recyclage du processus ou de la machine virtuelle au milieu de l’exécution, l’instance de la fonction reprend à partir de l’appel `await` précédent. Pour en savoir plus, consultez la section suivante, Modèle 2 : Fan out/fan in.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    try {
        const x = yield context.df.callActivity("F1");
        const y = yield context.df.callActivity("F2", x);
        const z = yield context.df.callActivity("F3", y);
        return    yield context.df.callActivity("F4", z);
    } catch (error) {
        // Error handling or compensation goes here.
    }
});
```

Vous pouvez utiliser l’objet `context.df` pour appeler d’autres fonctions par nom, passer des paramètres et retourner la sortie d’une fonction. Chaque fois que le code appelle `yield`, l’infrastructure Durable Functions crée des points de contrôle de la progression de l’instance de la fonction actuelle. En cas de recyclage du processus ou de la machine virtuelle au milieu de l’exécution, l’instance de la fonction reprend à partir de l’appel `yield` précédent. Pour en savoir plus, consultez la section suivante, Modèle 2 : Fan out/fan in.

> [!NOTE]
> L’objet `context` dans JavaScript représente l’intégralité du [contexte de la fonction](../functions-reference-node.md#context-object). Accédez au contexte Durable Functions à l’aide de la propriété `df` du contexte principal.

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df


def orchestrator_function(context: df.DurableOrchestrationContext):
    x = yield context.call_activity("F1", None)
    y = yield context.call_activity("F2", x)
    z = yield context.call_activity("F3", y)
    result = yield context.call_activity("F4", z)
    return result


main = df.Orchestrator.create(orchestrator_function)
```

Vous pouvez utiliser l’objet `context` pour appeler d’autres fonctions par nom, passer des paramètres et retourner la sortie d’une fonction. Chaque fois que le code appelle `yield`, l’infrastructure Durable Functions crée des points de contrôle de la progression de l’instance de la fonction actuelle. En cas de recyclage du processus ou de la machine virtuelle au milieu de l’exécution, l’instance de la fonction reprend à partir de l’appel `yield` précédent. Pour en savoir plus, consultez la section suivante, Modèle 2 : Fan out/fan in.

> [!NOTE]
> L’objet `context` dans Python représente le contexte d’orchestration. Accédez au contexte Azure Functions principal à l’aide de la propriété `function_context` du contexte d’orchestration.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```PowerShell
param($Context)

$X = Invoke-ActivityFunction -FunctionName 'F1'
$Y = Invoke-ActivityFunction -FunctionName 'F2' -Input $X
$Z = Invoke-ActivityFunction -FunctionName 'F3' -Input $Y
Invoke-ActivityFunction -FunctionName 'F4' -Input $Z
```

Vous pouvez utiliser la commande `Invoke-ActivityFunction` pour appeler d'autres fonctions par nom, passer des paramètres et renvoyer la sortie d'une fonction. Chaque fois que le code appelle `Invoke-ActivityFunction` sans le commutateur `NoWait`, l'infrastructure Durable Functions crée des points de contrôle de la progression de l'instance de la fonction actuelle. En cas de recyclage du processus ou de la machine virtuelle au milieu de l’exécution, l’instance de la fonction reprend à partir de l’appel `Invoke-ActivityFunction` précédent. Pour en savoir plus, consultez la section suivante, Modèle 2 : Fan out/fan in.

---

### <a name="pattern-2-fan-outfan-in"></a>Modèle 2 : fan-out/fan-in

Dans le modèle fan out/fan in, vous exécutez plusieurs fonctions en parallèle, puis attendez que toutes ces fonctions se terminent. Un travail d’agrégation est souvent effectué sur les résultats retournés par les fonctions.

![Schéma du modèle fan out/fan in](./media/durable-functions-concepts/fan-out-fan-in.png)

Avec des fonctions normales, vous pouvez effectuer un processus fan out en configurant la fonction afin qu’elle envoie plusieurs messages vers une file d’attente. Mais le processus fan-in est beaucoup plus difficile. Dans ce cas, dans une fonction normale, vous écrivez du code pour surveiller l’achèvement des fonctions déclenchées en file d’attente et stocker les sorties des fonctions.

L’extension Durable Functions gère ce modèle avec un code relativement simple :

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("FanOutFanIn")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var parallelTasks = new List<Task<int>>();

    // Get a list of N work items to process in parallel.
    object[] workBatch = await context.CallActivityAsync<object[]>("F1", null);
    for (int i = 0; i < workBatch.Length; i++)
    {
        Task<int> task = context.CallActivityAsync<int>("F2", workBatch[i]);
        parallelTasks.Add(task);
    }

    await Task.WhenAll(parallelTasks);

    // Aggregate all N outputs and send the result to F3.
    int sum = parallelTasks.Sum(t => t.Result);
    await context.CallActivityAsync("F3", sum);
}
```

Dans le cadre du processus fan-out, la distribution s’effectue vers plusieurs instances de la fonction `F2`. Le travail est suivi à l’aide d’une liste de tâches dynamique. `Task.WhenAll` est appelée pour attendre la fin de toutes les fonctions appelées. Les sorties de la fonction `F2` sont ensuite agrégées à partir de la liste de tâches dynamique puis transmises à la fonction `F3`.

La création automatique de points de contrôle qui a lieu lors de l’appel `await` sur `Task.WhenAll` garantit qu’un incident ou qu’un redémarrage potentiel survenu au milieu du processus ne nécessite pas le redémarrage d’une quelconque tâche déjà terminée.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const parallelTasks = [];

    // Get a list of N work items to process in parallel.
    const workBatch = yield context.df.callActivity("F1");
    for (let i = 0; i < workBatch.length; i++) {
        parallelTasks.push(context.df.callActivity("F2", workBatch[i]));
    }

    yield context.df.Task.all(parallelTasks);

    // Aggregate all N outputs and send the result to F3.
    const sum = parallelTasks.reduce((prev, curr) => prev + curr, 0);
    yield context.df.callActivity("F3", sum);
});
```

Dans le cadre du processus fan-out, la distribution s’effectue vers plusieurs instances de la fonction `F2`. Le travail est suivi à l’aide d’une liste de tâches dynamique. L’API `context.df.Task.all` est appelée pour attendre la fin de toutes les fonctions appelées. Les sorties de la fonction `F2` sont ensuite agrégées à partir de la liste de tâches dynamique puis transmises à la fonction `F3`.

La création automatique de points de contrôle qui a lieu lors de l’appel `yield` sur `context.df.Task.all` garantit qu’un incident ou qu’un redémarrage potentiel survenu au milieu du processus ne nécessite pas le redémarrage d’une quelconque tâche déjà terminée.

# <a name="python"></a>[Python](#tab/python)

```python
import azure.durable_functions as df


def orchestrator_function(context: df.DurableOrchestrationContext):
    # Get a list of N work items to process in parallel.
    work_batch = yield context.call_activity("F1", None)

    parallel_tasks = [ context.call_activity("F2", b) for b in work_batch ]
    
    outputs = yield context.task_all(parallel_tasks)

    # Aggregate all N outputs and send the result to F3.
    total = sum(outputs)
    yield context.call_activity("F3", total)


main = df.Orchestrator.create(orchestrator_function)
```

Dans le cadre du processus fan-out, la distribution s’effectue vers plusieurs instances de la fonction `F2`. Le travail est suivi à l’aide d’une liste de tâches dynamique. L’API `context.task_all` est appelée pour attendre la fin de toutes les fonctions appelées. Les sorties de la fonction `F2` sont ensuite agrégées à partir de la liste de tâches dynamique puis transmises à la fonction `F3`.

La création automatique de points de contrôle qui a lieu lors de l’appel `yield` sur `context.task_all` garantit qu’un incident ou qu’un redémarrage potentiel survenu au milieu du processus ne nécessite pas le redémarrage d’une quelconque tâche déjà terminée.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```PowerShell
param($Context)

# Get a list of work items to process in parallel.
$WorkBatch = Invoke-ActivityFunction -FunctionName 'F1'

$ParallelTasks =
    foreach ($WorkItem in $WorkBatch) {
        Invoke-ActivityFunction -FunctionName 'F2' -Input $WorkItem -NoWait
    }

$Outputs = Wait-ActivityFunction -Task $ParallelTasks

# Aggregate all outputs and send the result to F3.
$Total = ($Outputs | Measure-Object -Sum).Sum
Invoke-ActivityFunction -FunctionName 'F3' -Input $Total
```

Dans le cadre du processus fan-out, la distribution s’effectue vers plusieurs instances de la fonction `F2`. Notez l'utilisation du commutateur `NoWait` lors de l'appel de la fonction `F2` : ce commutateur permet à l'orchestrateur de continuer à appeler `F2` sans pour autant que l'activité soit terminée. Le travail est suivi à l’aide d’une liste de tâches dynamique. La commande `Wait-ActivityFunction` est appelée pour attendre la fin de toutes les fonctions appelées. Les sorties de la fonction `F2` sont ensuite agrégées à partir de la liste de tâches dynamique puis transmises à la fonction `F3`.

La création automatique de points de contrôle qui a lieu lors de l'appel `Wait-ActivityFunction` garantit qu'un incident ou qu'un redémarrage potentiel survenu au milieu du processus ne nécessite pas le redémarrage d'une quelconque tâche déjà terminée.

---

> [!NOTE]
> Dans de rares circonstances, un plantage peut se produire dans la fenêtre après l’exécution d’une fonction d’activité, mais avant que son achèvement ne soit enregistré dans l’historique d’orchestration. Si cela se produit, la fonction d’activité s’exécute à nouveau depuis le début après la reprise du processus.

### <a name="pattern-3-async-http-apis"></a>Modèle 3 : API HTTP Async

Le modèle d’API HTTP asynchrone traite le problème de coordination de l’état des opérations à exécution longue avec des clients externes. Pour implémenter ce modèle, une méthode courante consiste à faire déclencher l’action à exécution longue par un point de terminaison HTTP. Le client est ensuite redirigé vers un point de terminaison d’état que le client interroge pour savoir quand l’opération est terminée.

![Schéma du modèle d’API HTTP](./media/durable-functions-concepts/async-http-api.png)

Durable Functions fournit une **prise en charge intégrée** pour ce modèle, en simplifiant ou même en supprimant le code que vous devez écrire pour interagir avec des exécutions de fonctions de longue durée. Ainsi, les exemples de démarrage rapide de Durable Functions ([C#](durable-functions-create-first-csharp.md) et [JavaScript](quickstart-js-vscode.md)) montrent une commande REST simple qui vous permet de démarrer de nouvelles instances de fonctions orchestrator. Lorsqu’une instance démarre, l’extension expose des API HTTP webhook qui interrogent l’état de la fonction d’orchestrateur. 

L’exemple suivant montre les commandes REST permettant de démarrer un orchestrateur et d’interroger son état. Par souci de clarté, certains détails du protocole ont été retirés de l’exemple.

```
> curl -X POST https://myfunc.azurewebsites.net/api/orchestrators/DoWork -H "Content-Length: 0" -i
HTTP/1.1 202 Accepted
Content-Type: application/json
Location: https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/instances/b79baf67f717453ca9e86c5da21e03ec

{"id":"b79baf67f717453ca9e86c5da21e03ec", ...}

> curl https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/instances/b79baf67f717453ca9e86c5da21e03ec -i
HTTP/1.1 202 Accepted
Content-Type: application/json
Location: https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/instances/b79baf67f717453ca9e86c5da21e03ec

{"runtimeStatus":"Running","lastUpdatedTime":"2019-03-16T21:20:47Z", ...}

> curl https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/instances/b79baf67f717453ca9e86c5da21e03ec -i
HTTP/1.1 200 OK
Content-Length: 175
Content-Type: application/json

{"runtimeStatus":"Completed","lastUpdatedTime":"2019-03-16T21:20:57Z", ...}
```

Étant donné que le runtime de Durable Functions gère l’état à votre place, vous n’avez pas besoin d’implémenter votre propre mécanisme de suivi de l’état.

L’extension Durable Functions expose des API HTTP intégrées qui gèrent les orchestrations à exécution longue. Vous pouvez également implémenter ce modèle vous-même à l’aide de vos propres déclencheurs de fonction (comme un déclencheur HTTP, une file d’attente ou Azure Event Hubs) et de la [liaison du client d’orchestration](durable-functions-bindings.md#orchestration-client). Ainsi, vous pouvez utiliser un message de file d’attente pour déclencher l’arrêt. Sinon, vous pouvez vous servir d’un déclencheur HTTP protégé par une stratégie d’authentification Azure Active Directory à la place des API HTTP intégrées qui utilisent une clé générée pour l’authentification.

Pour plus d’informations, consultez l’article [Fonctionnalités HTTP](durable-functions-http-features.md), qui explique comment vous pouvez exposer des processus asynchrones à exécution longue sur HTTP à l’aide de l’extension Durable Functions.

### <a name="pattern-4-monitor"></a><a name="monitoring"></a>Modèle 4 : Superviser

Le modèle de surveillance fait référence à un processus souple et récurrent dans un flux de travail. Il peut s’agir, par exemple, d’une interrogation se poursuivant jusqu’à ce que certaines conditions soient remplies. Vous pouvez utiliser un [déclencheur de minuteur](../functions-bindings-timer.md) standard pour un scénario simple, comme une tâche de nettoyage périodique, mais son intervalle est statique et la gestion de la durée de vie des instances devient complexe. Vous pouvez utiliser Durable Functions pour créer des intervalles de récurrence flexibles, gérer la durée de vie des tâches et créer plusieurs processus de surveillance à partir d’une seule orchestration.

L’inversion du scénario d’API HTTP asynchrone antérieur représente un exemple du modèle de surveillance. Au lieu d’exposer un point de terminaison d’un client externe pour surveiller une opération longue, l’analyse de longue durée consomme un point de terminaison externe, puis attend un changement d’état.

![Schéma du modèle de surveillance](./media/durable-functions-concepts/monitor.png)

Avec quelques lignes de code, vous pouvez utiliser Durable Functions pour créer plusieurs moniteurs qui observent des points de terminaison arbitraires. Les moniteurs peuvent mettre fin à une exécution lorsqu’une condition est remplie, ou une autre fonction peut utiliser le client d’orchestration durable pour arrêter les moniteurs. Vous pouvez modifier l’intervalle `wait` d’un moniteur selon une condition spécifique (par exemple, avec un backoff exponentiel). 

Le code suivant implémente un moniteur de base :

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("MonitorJobStatus")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    int jobId = context.GetInput<int>();
    int pollingInterval = GetPollingInterval();
    DateTime expiryTime = GetExpiryTime();

    while (context.CurrentUtcDateTime < expiryTime)
    {
        var jobStatus = await context.CallActivityAsync<string>("GetJobStatus", jobId);
        if (jobStatus == "Completed")
        {
            // Perform an action when a condition is met.
            await context.CallActivityAsync("SendAlert", machineId);
            break;
        }

        // Orchestration sleeps until this time.
        var nextCheck = context.CurrentUtcDateTime.AddSeconds(pollingInterval);
        await context.CreateTimer(nextCheck, CancellationToken.None);
    }

    // Perform more work here, or let the orchestration end.
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");
const moment = require("moment");

module.exports = df.orchestrator(function*(context) {
    const jobId = context.df.getInput();
    const pollingInterval = getPollingInterval();
    const expiryTime = getExpiryTime();

    while (moment.utc(context.df.currentUtcDateTime).isBefore(expiryTime)) {
        const jobStatus = yield context.df.callActivity("GetJobStatus", jobId);
        if (jobStatus === "Completed") {
            // Perform an action when a condition is met.
            yield context.df.callActivity("SendAlert", machineId);
            break;
        }

        // Orchestration sleeps until this time.
        const nextCheck = moment.utc(context.df.currentUtcDateTime).add(pollingInterval, 's');
        yield context.df.createTimer(nextCheck.toDate());
    }

    // Perform more work here, or let the orchestration end.
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.durable_functions as df
import json
from datetime import timedelta 


def orchestrator_function(context: df.DurableOrchestrationContext):
    job = json.loads(context.get_input())
    job_id = job["jobId"]
    polling_interval = job["pollingInterval"]
    expiry_time = job["expiryTime"]

    while context.current_utc_datetime < expiry_time:
        job_status = yield context.call_activity("GetJobStatus", job_id)
        if job_status == "Completed":
            # Perform an action when a condition is met.
            yield context.call_activity("SendAlert", job_id)
            break

        # Orchestration sleeps until this time.
        next_check = context.current_utc_datetime + timedelta(seconds=polling_interval)
        yield context.create_timer(next_check)

    # Perform more work here, or let the orchestration end.


main = df.Orchestrator.create(orchestrator_function)
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

L'analyse n'est actuellement pas prise en charge dans PowerShell.

---

Quand une requête est reçue, une nouvelle instance d’orchestration est créée pour cet ID de tâche. L’instance interroge un état jusqu’à ce qu’une condition soit respectée et que vous quittiez la boucle. Un minuteur durable contrôle la fréquence d’interrogation. Des opérations supplémentaires peuvent ensuite être exécutées, ou l’orchestration peut prendre fin. Quand `nextCheck` dépasse `expiryTime`, le moniteur s’arrête.

### <a name="pattern-5-human-interaction"></a>Modèle 5 : interaction humaine

De nombreux processus automatisés impliquent un certain type d’interaction humaine. L’implication de personnes humaines dans un processus automatisé est complexe, car elles ne sont pas toujours aussi disponibles et réactives que les services cloud. Un processus automatisé peut rendre cette interaction possible en utilisant des délais d’expiration et une logique de compensation.

Un processus d’approbation est un exemple de processus d’entreprise impliquant une interaction humaine. L’approbation d’un manager peut être requise si une note de frais dépasse un certain montant. Si le manager n’approuve pas cette note de frais sous 72 heures (par exemple, s’il est en vacances), un processus d’escalade est déclenché pour obtenir l’approbation d’une autre personne (par exemple, le supérieur hiérarchique de ce manager).

![Schéma du modèle d’interaction humaine](./media/durable-functions-concepts/approval.png)

Vous pouvez implémenter le modèle utilisé dans cet exemple à l’aide d’une fonction d’orchestrateur. L’orchestrateur utilise un [minuteur durable](durable-functions-timers.md) pour demander l’approbation. L’orchestrateur procède à l’escalade si le délai d’expiration est atteint. L’orchestrateur attend un [événement externe](durable-functions-external-events.md), par exemple une notification générée par une interaction humaine.

Les exemples suivants créent un processus d’approbation illustrant le modèle d’interaction humaine :

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("ApprovalWorkflow")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    await context.CallActivityAsync("RequestApproval", null);
    using (var timeoutCts = new CancellationTokenSource())
    {
        DateTime dueTime = context.CurrentUtcDateTime.AddHours(72);
        Task durableTimeout = context.CreateTimer(dueTime, timeoutCts.Token);

        Task<bool> approvalEvent = context.WaitForExternalEvent<bool>("ApprovalEvent");
        if (approvalEvent == await Task.WhenAny(approvalEvent, durableTimeout))
        {
            timeoutCts.Cancel();
            await context.CallActivityAsync("ProcessApproval", approvalEvent.Result);
        }
        else
        {
            await context.CallActivityAsync("Escalate", null);
        }
    }
}
```

Pour créer le minuteur durable, appelez `context.CreateTimer`. La notification est reçue par `context.WaitForExternalEvent`. Ensuite, `Task.WhenAny` est appelée pour déterminer s’il faut procéder à l’escalade (le délai d’expiration est atteint en premier) ou traiter l’approbation (l’approbation est reçue avant l’expiration du délai).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");
const moment = require('moment');

module.exports = df.orchestrator(function*(context) {
    yield context.df.callActivity("RequestApproval");

    const dueTime = moment.utc(context.df.currentUtcDateTime).add(72, 'h');
    const durableTimeout = context.df.createTimer(dueTime.toDate());

    const approvalEvent = context.df.waitForExternalEvent("ApprovalEvent");
    if (approvalEvent === yield context.df.Task.any([approvalEvent, durableTimeout])) {
        durableTimeout.cancel();
        yield context.df.callActivity("ProcessApproval", approvalEvent.result);
    } else {
        yield context.df.callActivity("Escalate");
    }
});
```

Pour créer le minuteur durable, appelez `context.df.createTimer`. La notification est reçue par `context.df.waitForExternalEvent`. Ensuite, `context.df.Task.any` est appelée pour déterminer s’il faut procéder à l’escalade (le délai d’expiration est atteint en premier) ou traiter l’approbation (l’approbation est reçue avant l’expiration du délai).

# <a name="python"></a>[Python](#tab/python)

```python
import azure.durable_functions as df
import json
from datetime import timedelta 


def orchestrator_function(context: df.DurableOrchestrationContext):
    yield context.call_activity("RequestApproval", None)

    due_time = context.current_utc_datetime + timedelta(hours=72)
    durable_timeout_task = context.create_timer(due_time)
    approval_event_task = context.wait_for_external_event("ApprovalEvent")

    winning_task = yield context.task_any([approval_event_task, durable_timeout_task])

    if approval_event_task == winning_task:
        durable_timeout_task.cancel()
        yield context.call_activity("ProcessApproval", approval_event_task.result)
    else:
        yield context.call_activity("Escalate", None)


main = df.Orchestrator.create(orchestrator_function)
```

Pour créer le minuteur durable, appelez `context.create_timer`. La notification est reçue par `context.wait_for_external_event`. Ensuite, `context.task_any` est appelée pour déterminer s’il faut procéder à l’escalade (le délai d’expiration est atteint en premier) ou traiter l’approbation (l’approbation est reçue avant l’expiration du délai).

# <a name="powershell"></a>[PowerShell](#tab/powershell)

L'interaction humaine n'est actuellement pas prise en charge dans PowerShell.

---

Un client externe peut remettre la notification d’événement à une fonction d’orchestrateur en attente au moyen d’[API HTTP intégrées](durable-functions-http-api.md#raise-event) :

```bash
curl -d "true" http://localhost:7071/runtime/webhooks/durabletask/instances/{instanceId}/raiseEvent/ApprovalEvent -H "Content-Type: application/json"
```

Un événement peut également être déclenché à l’aide du client d’orchestration durable à partir d’une autre fonction dans la même application de fonction :

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("RaiseEventToOrchestration")]
public static async Task Run(
    [HttpTrigger] string instanceId,
    [DurableClient] IDurableOrchestrationClient client)
{
    bool isApproved = true;
    await client.RaiseEventAsync(instanceId, "ApprovalEvent", isApproved);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    const isApproved = true;
    await client.raiseEvent(instanceId, "ApprovalEvent", isApproved);
};
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.durable_functions as df


async def main(client: str):
    durable_client = df.DurableOrchestrationClient(client)
    is_approved = True
    await durable_client.raise_event(instance_id, "ApprovalEvent", is_approved)
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

L'interaction humaine n'est actuellement pas prise en charge dans PowerShell.

---

### <a name="pattern-6-aggregator-stateful-entities"></a><a name="aggregator"></a>Modèle 6 : Agrégateur (entités avec état)

Le sixième modèle repose sur l’agrégation de données d’événement sur une période au sein d’une seule *entité* adressable. Dans ce modèle, les données agrégées peuvent provenir de plusieurs sources, être transmises par lots ou être dispersées sur de longues périodes. L’agrégateur devra peut-être effectuer une action sur les données d’événement à leur arrivée, et des clients externes devront peut-être interroger les données agrégées.

![Schéma de l’agrégateur](./media/durable-functions-concepts/aggregator.png)

L’implémentation de ce modèle avec des fonctions normales sans état fait naître un défi de taille : le contrôle d’accès concurrentiel. Vous devrez non seulement vous soucier du risque de modification des mêmes données par plusieurs threads au même moment, mais également veiller à ce que l’agrégateur s’exécute sur une seule machine virtuelle à la fois.

Vous pouvez utiliser des [entités durables](durable-functions-entities.md) pour implémenter facilement ce modèle en tant que fonction unique.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("Counter")]
public static void Counter([EntityTrigger] IDurableEntityContext ctx)
{
    int currentValue = ctx.GetState<int>();
    switch (ctx.OperationName.ToLowerInvariant())
    {
        case "add":
            int amount = ctx.GetInput<int>();
            ctx.SetState(currentValue + amount);
            break;
        case "reset":
            ctx.SetState(0);
            break;
        case "get":
            ctx.Return(currentValue);
            break;
    }
}
```

Les entités durables peuvent également être modélisées en tant que classes dans .NET. Ce modèle peut être utile si la liste des opérations est fixe et devient volumineuse. L’exemple suivant est une implémentation équivalente de l’entité `Counter` à l’aide de classes et de méthodes .NET.

```csharp
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

# <a name="javascript"></a>[JavaScript](#tab/javascript)

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

# <a name="python"></a>[Python](#tab/python)

Les entités durables ne sont actuellement pas prises en charge dans Python.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Les entités durables ne sont actuellement pas prises en charge dans PowerShell.

---

Les clients peuvent empiler les *opérations* dans le cadre d’une fonction d’entité (processus également appelé « signalisation ») à l’aide de la [liaison du client d’entité](durable-functions-bindings.md#entity-client).

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("EventHubTriggerCSharp")]
public static async Task Run(
    [EventHubTrigger("device-sensor-events")] EventData eventData,
    [DurableClient] IDurableOrchestrationClient entityClient)
{
    var metricType = (string)eventData.Properties["metric"];
    var delta = BitConverter.ToInt32(eventData.Body, eventData.Body.Offset);

    // The "Counter/{metricType}" entity is created on-demand.
    var entityId = new EntityId("Counter", metricType);
    await entityClient.SignalEntityAsync(entityId, "add", delta);
}
```

> [!NOTE]
> Les proxies générés dynamiquement sont également disponibles dans .NET pour signaler les entités de type sécurisé. En plus de la signalisation, les clients peuvent aussi interroger l’état d’une fonction d’entité à l’aide de [méthodes de type sécurisé](durable-functions-bindings.md#entity-client-usage) sur la liaison du client d’orchestration.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    const entityId = new df.EntityId("Counter", "myCounter");
    await context.df.signalEntity(entityId, "add", 1);
};
```

# <a name="python"></a>[Python](#tab/python)

Les entités durables ne sont actuellement pas prises en charge dans Python.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Les entités durables ne sont actuellement pas prises en charge dans PowerShell.

---

Les fonctions d’entité sont disponibles dans [Durable Functions 2.0](durable-functions-versions.md) et les versions ultérieures pour C# et JavaScript.

## <a name="the-technology"></a>La technologie

En arrière-plan, l’extension Durable Functions repose sur le [Framework Durable Task](https://github.com/Azure/durabletask), une bibliothèque open source sur GitHub utilisée pour la génération de workflows dans le code. Tout comme Azure Functions est l’évolution serverless d’Azure WebJobs, Durable Functions est l’évolution serverless de l’infrastructure Durable Task Framework. Microsoft et d’autres organisations utilisent couramment l’infrastructure Durable Task Framework pour automatiser les processus critiques. Il convient parfaitement à l’environnement Azure Functions sans serveur.

## <a name="code-constraints"></a>Contraintes du code

Pour pouvoir fournir des garanties d’exécution longue et fiable, les fonctions orchestrator possèdent un ensemble de règles de codage qui doivent être suivies. Pour plus d’informations, consultez l’article [Contraintes du code des fonctions orchestrator](durable-functions-code-constraints.md).

## <a name="billing"></a>Facturation

Durable Functions suit le même modèle de facturation qu’Azure Functions. Pour plus d’informations, consultez [Tarification d’Azure Functions](https://azure.microsoft.com/pricing/details/functions/). Lors de l’exécution de fonctions orchestrator dans le [plan de consommation](../functions-scale.md#consumption-plan) Azure Functions, il est nécessaire de connaître certains comportements de facturation. Pour plus d’informations sur ces comportements, consultez l’article [Facturation Durable Functions](durable-functions-billing.md).

## <a name="jump-right-in"></a>Démarrage rapide

Vous pouvez démarrer avec Durable Functions en moins de dix minutes en suivant l’un de ces tutoriels de démarrage rapide propres à chaque langage :

* [C# avec Visual Studio 2019](durable-functions-create-first-csharp.md)
* [JavaScript avec Visual Studio Code](quickstart-js-vscode.md)
* [Python avec Visual Studio Code](quickstart-python-vscode.md)
* [PowerShell avec Visual Studio Code](quickstart-powershell-vscode.md)

Dans ces guides de démarrage rapide, vous créez et testez localement une fonction durable « hello world ». Vous allez ensuite publier le code de la fonction dans Azure. La fonction que vous créez orchestre et chaîne des appels à d’autres fonctions.

## <a name="learn-more"></a>En savoir plus

La vidéo suivante présente les avantages d’utiliser Durable Functions :

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Durable-Functions-in-Azure-Functions/player] 

Pour obtenir une discussion plus approfondie sur Durable Functions et la technologie sous-jacente, regardez la vidéo suivante (elle est consacrée à .NET, mais les concepts s’appliquent également à d’autres langages prises en charge) :

> [!VIDEO https://channel9.msdn.com/Events/dotnetConf/2018/S204/player]

Durable Functions est une extension avancée d’[Azure Functions](../functions-overview.md) et ne convient donc pas à toutes les applications. Pour une comparaison avec d’autres technologies d’orchestration Azure, consultez [Comparer Azure Functions et Azure Logic Apps](../functions-compare-logic-apps-ms-flow-webjobs.md#compare-azure-functions-and-azure-logic-apps).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Fonctionnalités et types de fonctions Durable Functions](durable-functions-types-features-overview.md)
