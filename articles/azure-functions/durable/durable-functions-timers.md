---
title: Minuteurs dans Fonctions durables - Azure
description: Découvrez comment implémenter des minuteurs durables dans l’extension Fonctions durables pour Azure Functions.
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: azfuncdf
ms.openlocfilehash: bb91f205a9b83b0b4b410644ef6c0fcbbf60876a
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91876445"
---
# <a name="timers-in-durable-functions-azure-functions"></a>Minuteurs dans Fonctions durables (Azure Functions)

[Fonctions durables](durable-functions-overview.md) fournit *des minuteurs durables* à utiliser dans les fonctions de l’orchestrateur pour implémenter des retards ou configurer des délais d’expiration sur des actions asynchrones. Des minuteurs durables doivent être utilisés dans les fonctions de l’orchestrateur à la place de `Thread.Sleep` et `Task.Delay` (C#), `setTimeout()` et `setInterval()` (JavaScript) ou `time.sleep()` (Python).

Vous créez un minuteur durable en appelant la méthode `CreateTimer` (.NET) ou la méthode `createTimer` (JavaScript) de la [liaison de déclencheur d’orchestration](durable-functions-bindings.md#orchestration-trigger). La méthode retourne une tâche qui se termine à une date et une heure spécifiées.

## <a name="timer-limitations"></a>Limitations des minuteurs

Lorsque vous créez un minuteur qui expire à 16:30, l’infrastructure des tâches durables sous-jacent empile un message qui devient uniquement visible à 16:30. Durant l’exécution dans le plan de consommation d’Azure Functions, le message de minuteur nouvellement visible garantit que l’application de fonction est activée sur une machine virtuelle appropriée.

> [!NOTE]
> * À partir de la [version 2.3.0](https://github.com/Azure/azure-functions-durable-extension/releases/tag/v2.3.0) de l’extension durable, les minuteurs durables sont illimités. Dans les versions antérieures de l’extension, les minuteurs durables sont limités à sept jours. Lorsque vous utilisez une version antérieure et que vous avez besoin d’un délai de plus de sept jours, utilisez les API de minuteur dans une boucle `while` pour simuler ce délai.
> * Utilisez toujours `CurrentUtcDateTime` au lieu de `DateTime.UtcNow` en .NET ou `currentUtcDateTime` au lieu de `Date.now` ou de `Date.UTC` en JavaScript lors du calcul de l’heure de déclenchement pour les minuteurs durables. Pour plus d’informations, consultez l’article [Contraintes du code des fonctions d’orchestrateur](durable-functions-code-constraints.md).

## <a name="usage-for-delay"></a>Utilisation des retards

L’exemple suivant montre comment utiliser des minuteurs durables pour retarder une exécution. Dans cet exemple, une notification de facturation est émise tous les jours pendant 10 jours.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("BillingIssuer")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    for (int i = 0; i < 10; i++)
    {
        DateTime deadline = context.CurrentUtcDateTime.Add(TimeSpan.FromDays(1));
        await context.CreateTimer(deadline, CancellationToken.None);
        await context.CallActivityAsync("SendBillingEvent");
    }
}
```

> [!NOTE]
> L’exemple C# précédent cible Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser `DurableOrchestrationContext` au lieu de `IDurableOrchestrationContext`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```js
const df = require("durable-functions");
const moment = require("moment");

module.exports = df.orchestrator(function*(context) {
    for (let i = 0; i < 10; i++) {
        const deadline = moment.utc(context.df.currentUtcDateTime).add(1, 'd');
        yield context.df.createTimer(deadline.toDate());
        yield context.df.callActivity("SendBillingEvent");
    }
});
```
# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df
from datetime import datetime, timedelta

def orchestrator_function(context: df.DurableOrchestrationContext):
    for i in range(0, 9):
        deadline = context.current_utc_datetime + timedelta(days=1)
        yield context.create_timer(deadline)
        yield context.call_activity("SendBillingEvent")

main = df.Orchestrator.create(orchestrator_function)
```
---

> [!WARNING]
> Evitez les boucles infinies dans les fonctions de l’orchestrateur. Pour plus d’informations sur la façon d’implémenter efficacement et en toute sécurité des scénarios de boucles infinies, consultez [Orchestrations externes](durable-functions-eternal-orchestrations.md).

## <a name="usage-for-timeout"></a>Utilisation des délais d’expiration

Cet exemple montre comment utiliser des minuteurs durables pour implémenter des délais d’expiration.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("TryGetQuote")]
public static async Task<bool> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    TimeSpan timeout = TimeSpan.FromSeconds(30);
    DateTime deadline = context.CurrentUtcDateTime.Add(timeout);

    using (var cts = new CancellationTokenSource())
    {
        Task activityTask = context.CallActivityAsync("GetQuote");
        Task timeoutTask = context.CreateTimer(deadline, cts.Token);

        Task winner = await Task.WhenAny(activityTask, timeoutTask);
        if (winner == activityTask)
        {
            // success case
            cts.Cancel();
            return true;
        }
        else
        {
            // timeout case
            return false;
        }
    }
}
```

> [!NOTE]
> L’exemple C# précédent cible Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser `DurableOrchestrationContext` au lieu de `IDurableOrchestrationContext`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```js
const df = require("durable-functions");
const moment = require("moment");

module.exports = df.orchestrator(function*(context) {
    const deadline = moment.utc(context.df.currentUtcDateTime).add(30, "s");

    const activityTask = context.df.callActivity("GetQuote");
    const timeoutTask = context.df.createTimer(deadline.toDate());

    const winner = yield context.df.Task.any([activityTask, timeoutTask]);
    if (winner === activityTask) {
        // success case
        timeoutTask.cancel();
        return true;
    }
    else
    {
        // timeout case
        return false;
    }
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df
from datetime import datetime, timedelta

def orchestrator_function(context: df.DurableOrchestrationContext):
    deadline = context.current_utc_datetime + timedelta(seconds=30)
    activity_task = context.call_activity("GetQuote")
    timeout_task = context.create_timer(deadline)

    winner = yield context.task_any([activity_task, timeout_task])
    if winner == activity_task:
        timeout_task.cancel()
        return True
    elif winner == timeout_task:
        return False

main = df.Orchestrator.create(orchestrator_function)
```

---

> [!WARNING]
> Utilisez un `CancellationTokenSource` (.NET) ou appelez `cancel()` sur le `TimerTask` (JavaScript) retourné pour annuler un minuteur durable si votre code n’attend pas qu’il se termine. Dans le framework des tâches durables, une orchestration ne passe à l’état « terminé » que quand toutes les tâches en attente sont terminées ou annulées.

Ce mécanisme d’annulation ne termine pas la fonction d’activité en cours ni les exécutions de sous-orchestrations. Il permet simplement à la fonction de l’orchestrateur d’ignorer le résultat et de continuer. Si votre application de fonction utilise le plan de consommation, vous serez toujours facturé pour le temps consacré et la mémoire consommée par la fonction d’activité abandonnée. Par défaut, les fonctions exécutées dans le plan de consommation ont un délai d’expiration de cinq minutes. Si cette limite est dépassée, l’hôte d’Azure Functions est recyclé pour arrêter toute exécution en cours et éviter toute facturation supplémentaire. Le [délai d’expiration des fonctions est configurable](../functions-host-json.md#functiontimeout).

Pour obtenir un exemple plus détaillé d’implémentation de délais d’expiration dans les fonctions de l’orchestrateur, consultez l’article [Interaction humaine et délais d’expiration – Vérification par téléphone](durable-functions-phone-verification.md).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Déclenchement et gestion d’événements externes](durable-functions-external-events.md)
