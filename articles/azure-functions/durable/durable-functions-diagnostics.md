---
title: Diagnostics dans Fonctions durables - Azure
description: Découvrez comment gérer diagnostiquer les problèmes avec l’extension Fonctions durables pour Azure Functions.
author: cgillum
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: azfuncdf
ms.openlocfilehash: 4714b9330c4a9d9cd390a58f814e3cdb4b591038
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92168139"
---
# <a name="diagnostics-in-durable-functions-in-azure"></a>Diagnostics de Durable Functions dans Azure

Il existe plusieurs options permettant de diagnostiquer les problèmes avec [Fonctions durables](durable-functions-overview.md). Certaines de ces options sont les mêmes pour les fonctions standard, et d’autres sont uniques à Fonctions durables.

## <a name="application-insights"></a>Application Insights

[Application Insights](../../azure-monitor/app/app-insights-overview.md) est la méthode recommandée pour les diagnostics et la surveillance dans Azure Functions. Il en va de même pour Fonctions durables. Pour obtenir une vue d’ensemble montrant comment tirer parti d’Application Insights dans votre application de fonction, consultez [Surveiller l’exécution des fonctions Azure](../functions-monitoring.md).

L’extension Fonctions durables d’Azure émet également des *événements de suivi* vous permettant de tracer l’exécution de bout en bout d’une orchestration. Ces événements de suivi sont accessibles et interrogés à l’aide de l’outil [Application Insights Analytics](../../azure-monitor/log-query/log-query-overview.md) dans le portail Azure.

### <a name="tracking-data"></a>Suivi des données

Chaque événement du cycle de vie d’une instance d’orchestration entraîne l’inscription d’un événement de suivi dans la collection des **suivis** d’Application Insights. Cet événement contient une charge utile **customDimensions** avec plusieurs champs.  Les noms de champs contiennent tous le préfixe `prop__`.

* **hubName** : Nom du hub de tâches dans lequel vos orchestrations sont en cours d’exécution.
* **appName** : Le nom de l’application de fonction. Ce champ est utile si plusieurs de vos applications de fonction partagent la même instance Application Insights.
* **slotName** : [emplacement de déploiement](../functions-deployment-slots.md) dans lequel s’exécute l’application de fonction actuelle. Ce champ est utile pour tirer parti des emplacements de déploiement pour la version de vos orchestrations.
* **functionName** : Nom de la fonction d’orchestrateur ou d’activité.
* **functionType** : Type de la fonction, par exemple **Orchestrateur** ou **Activité**.
* **instanceId** : ID unique de l’instance d’orchestration.
* **state** : État d’exécution du cycle de vie de l’instance. Les valeurs valides sont les suivantes :
  * **Scheduled** : La fonction a été planifiée pour être exécutée, mais elle n’a pas encore démarré.
  * **Started** : La fonction a démarré, mais elle n’a pas encore été attendue ou s’est terminée.
  * **Awaited** : L’orchestrateur a planifié des tâches et attend qu’elles se terminent.
  * **Listening** : L’orchestrateur écoute une notification d’événement externe.
  * **Completed** : La fonction s’est terminée avec succès.
  * **Échec** : La fonction a échoué avec une erreur.
* **reason** : données supplémentaires associées à l’événement de suivi. Par exemple, si une instance attend une notification d’événement externe, ce champ indique le nom de l’événement attendu. Si une fonction a échoué, ce champ contient les détails de l’erreur.
* **isReplay** : Valeur booléenne qui indique si l’événement de suivi est destiné à une réexécution.
* **extensionVersion** : Version de l’extension Tâche durable. Ces informations de version sont particulièrement importantes pour signaler d’éventuels bogues dans l’extension. Des instances à long terme peuvent signaler plusieurs versions si une mise à jour se produit pendant leur exécution.
* **sequenceNumber** : Numéro séquentiel d’extension pour un événement. Combiné avec le timestamp, il permet de classer les événements par durée d’exécution. *Notez que ce numéro sera réinitialisé à zéro si l’hôte redémarre alors que l’instance est en cours d’exécution, il est donc important de toujours d’abord trier par timestamp, puis par sequenceNumber.*

Le niveau de détail des données de suivi transmises à Application Insights peut être configuré dans la section `logger` (Functions 1.x) ou `logging` (Functions 2.0) du fichier `host.json`.

#### <a name="functions-10"></a>Functions 1.0

```json
{
    "logger": {
        "categoryFilter": {
            "categoryLevels": {
                "Host.Triggers.DurableTask": "Information"
            }
        }
    }
}
```

#### <a name="functions-20"></a>Functions 2.0

```json
{
    "logging": {
        "logLevel": {
          "Host.Triggers.DurableTask": "Information",
        },
    }
}
```

Par défaut, tous les événements de suivi non rejoués sont transmis. Le volume de données peut être réduit en définissant `Host.Triggers.DurableTask` sur `"Warning"` ou `"Error"`. Dans ce cas, les événements de suivi seront uniquement transmis en cas de situation exceptionnelle.

Pour activer l’émission d’événements de relecture d’orchestration détaillée, `LogReplayEvents` peut être défini sur `true` dans le fichier `host.json` sous `durableTask` comme indiqué :

#### <a name="functions-10"></a>Functions 1.0

```json
{
    "durableTask": {
        "logReplayEvents": true
    }
}
```

#### <a name="functions-20"></a>Functions 2.0

```json
{
    "extensions": {
        "durableTask": {
            "logReplayEvents": true
        }
    }
}
```

> [!NOTE]
> Par défaut, les données de télémétrie Application Insights sont échantillonnées par le runtime Azure Functions pour éviter un transfert trop fréquent de données. Cela peut entraîner une perte des informations de suivi si de nombreux événements de cycle de vie se produisent sur une courte période. L’article sur [la surveillance d’Azure Functions](../configure-monitoring.md#configure-sampling) explique comment configurer ce comportement.

### <a name="single-instance-query"></a>Requête d’instance unique

La requête suivante affiche les données de suivi historiques d’une seule instance de la fonction d’orchestration [Séquence Hello](durable-functions-sequence.md). Ces informations sont écrites à l’aide du [langage de requête Kusto](/azure/data-explorer/kusto/query/). Elle exclut la réexécution afin que seul le chemin d’exécution *logique* s’affiche. Les événements peuvent être classés en les triant par `timestamp` et `sequenceNumber` comme indiqué dans la requête ci-dessous :

```kusto
let targetInstanceId = "ddd1aaa685034059b545eb004b15d4eb";
let start = datetime(2018-03-25T09:20:00);
traces
| where timestamp > start and timestamp < start + 30m
| where customDimensions.Category == "Host.Triggers.DurableTask"
| extend functionName = customDimensions["prop__functionName"]
| extend instanceId = customDimensions["prop__instanceId"]
| extend state = customDimensions["prop__state"]
| extend isReplay = tobool(tolower(customDimensions["prop__isReplay"]))
| extend sequenceNumber = tolong(customDimensions["prop__sequenceNumber"])
| where isReplay != true
| where instanceId == targetInstanceId
| sort by timestamp asc, sequenceNumber asc
| project timestamp, functionName, state, instanceId, sequenceNumber, appName = cloud_RoleName
```

Le résultat est une liste d’événements de suivi indiquant le chemin d’exécution de l’orchestration, y compris toutes les fonctions d’activité triées par durée d’exécution par ordre ascendant.

![Requête triée sur une seule instance Application Insights](./media/durable-functions-diagnostics/app-insights-single-instance-ordered-query.png)

### <a name="instance-summary-query"></a>Requête de résumé de l’instance

La requête suivante affiche l’état de toutes les instances d’orchestration qui ont été exécutées dans un intervalle de temps spécifié.

```kusto
let start = datetime(2017-09-30T04:30:00);
traces
| where timestamp > start and timestamp < start + 1h
| where customDimensions.Category == "Host.Triggers.DurableTask"
| extend functionName = tostring(customDimensions["prop__functionName"])
| extend instanceId = tostring(customDimensions["prop__instanceId"])
| extend state = tostring(customDimensions["prop__state"])
| extend isReplay = tobool(tolower(customDimensions["prop__isReplay"]))
| extend output = tostring(customDimensions["prop__output"])
| where isReplay != true
| summarize arg_max(timestamp, *) by instanceId
| project timestamp, instanceId, functionName, state, output, appName = cloud_RoleName
| order by timestamp asc
```

Le résultat est une liste d’ID d’instances et leur actuel état d’exécution.

![Requête sur une seule instance Application Insights](./media/durable-functions-diagnostics/app-insights-single-summary-query.png)

## <a name="durable-task-framework-logging"></a>Journalisation durable de l’infrastructure des tâches

Les journaux d’extension Durable sont utiles pour comprendre le comportement de votre logique d’orchestration. Toutefois, ces journaux ne contiennent pas toujours assez d’informations pour déboguer les problèmes de performances et de fiabilité au niveau de l’infrastructure. À partir de la version **2.3.0** de l’extension Durable, les journaux émis par l’infrastructure Durable Task Framework (DTFx) sous-jacentes sont également disponibles pour la collecte.

Lorsque vous examinez les journaux émis par l’infrastructure DTFx, il est important de comprendre que le moteur DTFx est doté de deux composants : le moteur de distribution principal (`DurableTask.Core`) et l’un des nombreux fournisseurs de stockage pris en charge (Durable Functions utilise `DurableTask.AzureStorage` par défaut).

* **DurableTask.Core** : contient des informations sur l’exécution de l’orchestration et la planification de bas niveau.
* **DurableTask.AzureStorage** : contient des informations relatives aux interactions avec les artefacts de Stockage Azure, y compris les files d’attente internes, les objets blob et les tables de stockage utilisées pour stocker et extraire l’état de l’orchestration interne.

Vous pouvez activer ces journaux en mettant à jour la section `logging/logLevel` du fichier**host.json** de votre application de fonction. L’exemple suivant montre comment activer les journaux d’avertissement et d’erreur depuis `DurableTask.Core` et `DurableTask.AzureStorage` :

```json
{
  "version": "2.0",
  "logging": {
    "logLevel": {
      "DurableTask.AzureStorage": "Warning",
      "DurableTask.Core": "Warning"
    }
  }
}
```

Si Application Insights est activé, ces journaux sont automatiquement ajoutés à la collecte `trace`. Vous pouvez les rechercher de la même façon que vous recherchez d’autres journaux `trace` à l’aide de requêtes Kusto.

> [!NOTE]
> Pour les applications de production, il est recommandé d’activer les journaux `DurableTask.Core` et `DurableTask.AzureStorage` à l’aide du filtre `"Warning"`. Des filtres plus détaillés, tels que `"Information"` , sont très utiles pour le débogage des problèmes de performances. Toutefois, ces événements de journaux sont volumineux et peuvent considérablement augmenter les coûts de stockage de données Application Insights.

La requête Kusto suivante montre comment interroger les journaux DTFx. La partie la plus importante de la requête est `where customerDimensions.Category startswith "DurableTask"` dans la mesure où celle-ci filtre les résultats dans les journaux des catégories `DurableTask.Core` et `DurableTask.AzureStorage`.

```kusto
traces
| where customDimensions.Category startswith "DurableTask"
| project
    timestamp,
    severityLevel,
    Category = customDimensions.Category,
    EventId = customDimensions.EventId,
    message,
    customDimensions
| order by timestamp asc 
```
Le résultat est un ensemble de journaux écrits par les fournisseurs de journaux Durable Task Framework.

![Résultats de requête DTFx dans Application Insights](./media/durable-functions-diagnostics/app-insights-dtfx.png)

Pour plus d’informations sur les événements de journalisation disponibles, consultez la [documentation sur Durable Task Framework dans le cadre de la journalisation structurée sur GitHub](https://github.com/Azure/durabletask/tree/master/src/DurableTask.Core/Logging#durabletaskcore-logging).

## <a name="app-logging"></a>Journalisation des applications

Il est important de garder à l’esprit le comportement de réexécution de l’orchestrateur lors de l’écriture des journaux d’activité directement à partir d’une fonction d’orchestrateur. Par exemple, considérez la fonction d’orchestrateur suivante :

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("FunctionChain")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context,
    ILogger log)
{
    log.LogInformation("Calling F1.");
    await context.CallActivityAsync("F1");
    log.LogInformation("Calling F2.");
    await context.CallActivityAsync("F2");
    log.LogInformation("Calling F3");
    await context.CallActivityAsync("F3");
    log.LogInformation("Done!");
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context){
    context.log("Calling F1.");
    yield context.df.callActivity("F1");
    context.log("Calling F2.");
    yield context.df.callActivity("F2");
    context.log("Calling F3.");
    yield context.df.callActivity("F3");
    context.log("Done!");
});
```

# <a name="python"></a>[Python](#tab/python)
```python
import logging
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    logging.info("Calling F1.")
    yield context.call_activity("F1")
    logging.info("Calling F2.")
    yield context.call_activity("F2")
    logging.info("Calling F3.")
    yield context.call_activity("F3")
    return None

main = df.Orchestrator.create(orchestrator_function)
```

---

Les données de journal obtenues se présentent comme dans l’exemple suivant :

```txt
Calling F1.
Calling F1.
Calling F2.
Calling F1.
Calling F2.
Calling F3.
Calling F1.
Calling F2.
Calling F3.
Done!
```

> [!NOTE]
> N’oubliez pas que même si les journaux d’activité annoncent appeler les fonctions F1, F2 et F3, ces fonctions sont uniquement *réellement* appelées la première fois qu’elles sont rencontrées. Les prochains appels qui se produisent lors de la réexécution sont ignorés et les sorties sont réexécutées à la logique d’orchestrateur.

Si vous souhaitez uniquement consigner une non réexécution, vous pouvez écrire une expression conditionnelle qui écrit dans le journal uniquement si l’indicateur « is replaying » est `false`. Considérez l’exemple ci-dessus, mais cette fois avec des vérifications de réexécution.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("FunctionChain")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context,
    ILogger log)
{
    if (!context.IsReplaying) log.LogInformation("Calling F1.");
    await context.CallActivityAsync("F1");
    if (!context.IsReplaying) log.LogInformation("Calling F2.");
    await context.CallActivityAsync("F2");
    if (!context.IsReplaying) log.LogInformation("Calling F3");
    await context.CallActivityAsync("F3");
    log.LogInformation("Done!");
}
```

À partir de Durable Functions 2.0, les fonctions de l’orchestrateur .NET ont également la possibilité de créer un `ILogger` qui filtre automatiquement les instructions de journal lors de la relecture. Ce filtrage automatique s’effectue à l’aide de l’API [IDurableOrchestrationContext. CreateReplaySafeLogger(ILogger)](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.durablecontextextensions.createreplaysafelogger).

```csharp
[FunctionName("FunctionChain")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context,
    ILogger log)
{
    log = context.CreateReplaySafeLogger(log);
    log.LogInformation("Calling F1.");
    await context.CallActivityAsync("F1");
    log.LogInformation("Calling F2.");
    await context.CallActivityAsync("F2");
    log.LogInformation("Calling F3");
    await context.CallActivityAsync("F3");
    log.LogInformation("Done!");
}
```

> [!NOTE]
> Les exemples C# précédents portent sur Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser `DurableOrchestrationContext` au lieu de `IDurableOrchestrationContext`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context){
    if (!context.df.isReplaying) context.log("Calling F1.");
    yield context.df.callActivity("F1");
    if (!context.df.isReplaying) context.log("Calling F2.");
    yield context.df.callActivity("F2");
    if (!context.df.isReplaying) context.log("Calling F3.");
    yield context.df.callActivity("F3");
    context.log("Done!");
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import logging
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    if not context.is_replaying:
        logging.info("Calling F1.")
    yield context.call_activity("F1")
    if not context.is_replaying:
        logging.info("Calling F2.")
    yield context.call_activity("F2")
    if not context.is_replaying:
        logging.info("Calling F3.")
    yield context.call_activity("F3")
    return None

main = df.Orchestrator.create(orchestrator_function)
```

---

Avec les modifications mentionnées précédemment, la sortie du journal est la suivante :

```txt
Calling F1.
Calling F2.
Calling F3.
Done!
```

## <a name="custom-status"></a>État personnalisé

L’état d’orchestration personnalisé vous permet de définir une valeur d’état personnalisée pour votre fonction d’orchestrateur. Cet état personnalisé est ensuite visible par les clients externes via l’[API de requête d’état HTTP](durable-functions-http-api.md#get-instance-status) ou via des appels d’API spécifiques à une langue. L’état d’une orchestration personnalisée permet de surveiller plus précisément les fonctions d’orchestrateur. Par exemple, le code de fonction d’orchestrateur peut appeler l’API « définir le statut personnalisé » afin de mettre à jour la progression d’une opération de longue durée. Un client, comme une page web ou un autre système externe, peut ensuite interroger régulièrement les API de requête d’état HTTP pour en savoir plus sur l’état d’avancement. Vous trouverez ci-dessous un exemple de code pour la définition d’une valeur d’état personnalisée dans une fonction d’orchestrateur :

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("SetStatusTest")]
public static async Task SetStatusTest([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    // ...do work...

    // update the status of the orchestration with some arbitrary data
    var customStatus = new { completionPercentage = 90.0, status = "Updating database records" };
    context.SetCustomStatus(customStatus);

    // ...do more work...
}
```

> [!NOTE]
> L’exemple C# précédent porte sur Durable Functions 2.x. Pour Durable Functions 1.x, vous devez utiliser `DurableOrchestrationContext` au lieu de `IDurableOrchestrationContext`. Pour en savoir plus sur les différences entre les versions, consultez l’article [Versions de Durable Functions](durable-functions-versions.md).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    // ...do work...

    // update the status of the orchestration with some arbitrary data
    const customStatus = { completionPercentage: 90.0, status: "Updating database records", };
    context.df.setCustomStatus(customStatus);

    // ...do more work...
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import logging
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    # ...do work...

    # update the status of the orchestration with some arbitrary data
    custom_status = {'completionPercentage': 90.0, 'status': 'Updating database records'}
    context.set_custom_status(custom_status)
    # ...do more work...

    return None

main = df.Orchestrator.create(orchestrator_function)
```

---

Pendant l’exécution de l’orchestration, les clients externes peuvent récupérer cet état personnalisé :

```http
GET /runtime/webhooks/durabletask/instances/instance123?code=XYZ

```

Les clients obtiennent la réponse suivante :

```json
{
  "runtimeStatus": "Running",
  "input": null,
  "customStatus": { "completionPercentage": 90.0, "status": "Updating database records" },
  "output": null,
  "createdTime": "2017-10-06T18:30:24Z",
  "lastUpdatedTime": "2017-10-06T19:40:30Z"
}
```

> [!WARNING]
> La charge utile de l’état personnalisé est limitée à 16 Ko de texte JSON UTF-16, car elle doit pouvoir tenir dans une colonne de Stockage Table Azure. Vous pouvez utiliser un stockage externe si vous avez besoin d’une charge utile plus importante.

## <a name="debugging"></a>Débogage

Azure Functions prend directement en charge un code de fonction de débogage, et cette prise en charge s’applique également à Fonctions durables, que ce soit avec une exécution dans Azure ou localement. Toutefois, vous devez tenir compte de certains comportements lors du débogage :

* **Réexécution** : Les fonctions d’orchestrateur sont régulièrement [réexécutées](durable-functions-orchestrations.md#reliability) lors de la réception de nouvelles entrées. Ce comportement signifie qu’une seule exécution *logique* d’une fonction d’orchestrateur peut atteindre le même point d’arrêt plusieurs fois, en particulier si elle est définie très tôt dans le code de la fonction.
* **Await** : Chaque fois qu’un élément `await` est rencontré dans une fonction d’orchestrateur, le contrôle repasse au répartiteur de l’infrastructure Durable Task Framework. Si c’est la première fois qu’un événement `await` particulier est détecté, la tâche associée n’est *jamais* reprise. Puisque la tâche ne reprend jamais, vous ne pouvez pas *parcourir* l’événement await (F10 dans Visual Studio). Parcourez uniquement des travaux lorsqu’une tâche est réexécutée.
* **Délais d’expiration des messages** : Durable Functions utilise en interne des files d’attente de messages pour gérer l’exécution des fonctions d’orchestrateur, d’activité et d’entité. Dans un environnement à plusieurs machines virtuelles, si vous arrêtez le débogage pendant de longues périodes, une autre machine virtuelle risque de récupérer le message, ce qui entraîne une double exécution. Ce comportement s’applique également aux fonctions déclenchées par une file d’attente standard, mais il est important de le souligner car les files d’attente constituent un détail d’implémentation.
* **Arrêt et démarrage** : Les messages dans Durable Functions persistent entre les sessions de débogage. Si vous arrêtez le débogage et mettez fin au processus hôte local pendant l’exécution d’une fonction durable, cette fonction peut être réexécutée automatiquement dans une session de débogage ultérieure. Ce comportement peut prêter à confusion lorsque ce n’est pas attendu. L’effacement de tous les messages des [files d’attente de stockage interne](durable-functions-perf-and-scale.md#internal-queue-triggers) entre les sessions de débogage est une technique qui permet d’éviter ce comportement.

> [!TIP]
> Lors de la définition de points d’arrêt dans les fonctions d’orchestrateur, si vous voulez uniquement arrêter en cas de non-réexécution, vous pouvez définir un point d’arrêt conditionnel qui s’arrête uniquement si la valeur « is replaying » est `false`.

## <a name="storage"></a>Stockage

Par défaut, Durable Functions stocke l’état dans Stockage Azure. Ce comportement signifie que vous pouvez examiner l’état de vos orchestrations à l’aide d’outils tels que l’[Explorateur Stockage Azure Microsoft](../../vs-azure-tools-storage-manage-with-storage-explorer.md).

![Capture d'écran de l'Explorateur Stockage Azure](./media/durable-functions-diagnostics/storage-explorer.png)

Cet outil est utile pour le débogage car il vous permet de visualiser exactement l’état d’une orchestration. Vous pouvez également examiner les messages de files d’attente pour identifier les tâches en attente (ou bloquées dans certains cas).

> [!WARNING]
> Même s’il est pratique d’afficher l’historique d’exécution dans le stockage de table, évitez de créer une dépendance sur cette table. Elle peut changer à mesure que l’extension Fonctions durables évolue.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur la supervision dans Azure Functions](../functions-monitoring.md)
