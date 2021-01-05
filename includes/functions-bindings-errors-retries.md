---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 10/01/2020
ms.author: glenga
ms.openlocfilehash: 2ccff72be66a88b9bf0a5e9eb9c29ade8397804b
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96356191"
---
Les erreurs signalées dans Azure Functions peuvent avoir l’une des origines suivantes :

- Utilisation des [déclencheurs et liaisons intégrés](..\articles\azure-functions\functions-triggers-bindings.md) Azure Functions
- Appels aux API des services Azure sous-jacents
- Appels aux points de terminaison REST
- Appels aux bibliothèques clientes, aux packages ou aux API tierces

Il est important de suivre les bonnes pratiques de gestion des erreurs afin d’éviter de perdre des données ou de manquer des messages. Les pratiques recommandées concernant la gestion des erreurs impliquent les actions suivantes :

- [Activer Application Insights](../articles/azure-functions/functions-monitoring.md)
- [Utiliser la gestion structurée des erreurs](#use-structured-error-handling)
- [Concevoir en ayant en vue l’idempotence](../articles/azure-functions/functions-idempotent.md)
- [Implémenter des stratégies relatives aux nouvelles tentatives](#retry-policies-preview) (si nécessaire)

### <a name="use-structured-error-handling"></a>Utiliser la gestion structurée des erreurs

La capture et la journalisation des erreurs sont essentielles pour superviser l’intégrité de votre application. Le niveau le plus élevé de tout code de fonction doit inclure un bloc try/catch. Dans le bloc catch, vous pouvez capturer et journaliser les erreurs.

## <a name="retry-policies-preview"></a>Stratégies de nouvelle tentative (préversion)

Une stratégie de nouvelles tentatives peut être définie sur n’importe quelle fonction pour n’importe quel type de déclencheur dans votre application de fonction.  La stratégie de nouvelles tentatives réexécute une fonction jusqu’à ce que celle-ci aboutisse ou jusqu’à ce que le nombre maximal de nouvelles tentatives soit atteint.  Vous pouvez définir des stratégies de nouvelles tentatives pour toutes les fonctions d’une application ou pour des fonctions individuelles.  Par défaut, une application de fonction n’effectue pas de nouvelles tentatives pour les messages (à part les [déclencheurs spécifiques qui ont une stratégie de nouvelles tentatives sur la source du déclencheur](#using-retry-support-on-top-of-trigger-resilience)).  Une stratégie de nouvelles tentatives est évaluée chaque fois qu’une exécution entraîne une exception non interceptée.  Il est recommandé d’intercepter toutes les exceptions dans votre code et de regénérer toute erreur susceptible d’entraîner une nouvelle tentative.  Les points de contrôle Event Hubs et Azure Cosmos DB ne sont pas écrits tant que la stratégie de nouvelles tentatives pour l’exécution n’est pas terminée, ce qui signifie que la progression sur cette partition est suspendue jusqu’à ce que le traitement du lot actuel soit terminé.

### <a name="retry-policy-options"></a>Options de stratégie de nouvelles tentatives

Les options suivantes permettent de définir une stratégie de nouvelles tentatives.

**Nombre maximal de nouvelles tentatives** correspond au nombre maximal de nouvelles tentatives d’exécution avant un éventuel échec. Une valeur de `-1` signifie qu’il faut effectuer ces nouvelles tentatives indéfiniment. Le nombre actuel de nouvelles tentatives est stocké dans la mémoire de l’instance. Il est possible qu’un échec d’instance se produise entre deux nouvelles tentatives.  Quand une instance échoue durant une stratégie de nouvelles tentatives, le nombre de nouvelles tentatives est perdu.  En cas d’échec d’instance, les déclencheurs comme Event Hubs, Azure Cosmos DB et Queue Storage peuvent reprendre le traitement et retenter le lot sur une nouvelle instance, le nombre de nouvelles tentatives étant réinitialisé à zéro.  D’autres déclencheurs, tels que ceux basés sur HTTP ou un minuteur, ne reprennent pas le traitement sur une nouvelle instance.  Cela signifie que le nombre maximal de nouvelles tentatives est régi par le principe du « meilleur effort ». Dans certains cas rares, le nombre maximal de nouvelles tentatives d’exécution peut être supérieur au maximum ou, pour des déclencheurs basés sur HTTP ou un minuteur, inférieur au maximum.

La **stratégie de nouvelles tentatives** contrôle le comportement des nouvelles tentatives.  Voici deux options de nouvelle tentative prises en charge :

| Option | Description|
|---|---|
|**`fixedDelay`**| Un laps de temps spécifié est autorisé à s’écouler entre chaque nouvelle tentative.|
| **`exponentialBackoff`**| La première nouvelle tentative attend le délai minimal. Lors des nouvelles tentatives suivantes, le laps de temps est ajouté de manière exponentielle à la durée initiale de chaque nouvelle tentative jusqu’à ce que le délai maximal soit atteint.  Le back-off exponentiel ajoute une faible randomisation aux délais pour échelonner les nouvelles tentatives dans les scénarios à débit élevé.|

#### <a name="app-level-configuration"></a>Configuration au niveau d’une l’application

Une stratégie de nouvelles tentatives peut être définie pour toutes les fonctions d’une application [à l’aide du fichier `host.json`](../articles/azure-functions/functions-host-json.md#retry). 

#### <a name="function-level-configuration"></a>Configuration au niveau d’une fonction

Vous pouvez définir une stratégie de nouvelles tentatives pour une fonction spécifique.  Une configuration spécifique à une fonction a priorité sur une configuration au niveau d’une application.

#### <a name="fixed-delay-retry"></a>Nouvelle tentative avec délai fixe

# <a name="c"></a>[C#](#tab/csharp)

Les nouvelles tentatives nécessitent un package NuGet [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs) >= 3.0.23

```csharp
[FunctionName("EventHubTrigger")]
[FixedDelayRetry(5, "00:00:10")]
public static async Task Run([EventHubTrigger("myHub", Connection = "EventHubConnection")] EventData[] events, ILogger log)
{
// ...
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Voici la stratégie de nouvelles tentatives dans le fichier *function.json* :

```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "fixedDelay",
        "maxRetryCount": 4,
        "delayInterval": "00:00:10"
    }
}
```
# <a name="javascript"></a>[JavaScript](#tab/javascript)

Voici la stratégie de nouvelles tentatives dans le fichier *function.json* :


```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "fixedDelay",
        "maxRetryCount": 4,
        "delayInterval": "00:00:10"
    }
}
```

# <a name="python"></a>[Python](#tab/python)

Voici la stratégie de nouvelles tentatives dans le fichier *function.json* :

```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "fixedDelay",
        "maxRetryCount": 4,
        "delayInterval": "00:00:10"
    }
}
```

# <a name="java"></a>[Java](#tab/java)

Voici la stratégie de nouvelles tentatives dans le fichier *function.json* :


```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "fixedDelay",
        "maxRetryCount": 4,
        "delayInterval": "00:00:10"
    }
}
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Voici la stratégie de nouvelles tentatives dans le fichier *function.json* :


```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "fixedDelay",
        "maxRetryCount": 4,
        "delayInterval": "00:00:10"
    }
}
```
---

#### <a name="exponential-backoff-retry"></a>Nouvelle tentative avec backoff exponentiel

# <a name="c"></a>[C#](#tab/csharp)

Les nouvelles tentatives nécessitent un package NuGet [Microsoft.Azure.WebJobs](https://www.nuget.org/packages/Microsoft.Azure.WebJobs) >= 3.0.23

```csharp
[FunctionName("EventHubTrigger")]
[ExponentialBackoffRetry(5, "00:00:04", "00:15:00")]
public static async Task Run([EventHubTrigger("myHub", Connection = "EventHubConnection")] EventData[] events, ILogger log)
{
// ...
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Voici la stratégie de nouvelles tentatives dans le fichier *function.json* :

```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "exponentialBackoff",
        "maxRetryCount": 5,
        "minimumInterval": "00:00:10",
        "maximumInterval": "00:15:00"
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Voici la stratégie de nouvelles tentatives dans le fichier *function.json* :

```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "exponentialBackoff",
        "maxRetryCount": 5,
        "minimumInterval": "00:00:10",
        "maximumInterval": "00:15:00"
    }
}
```

# <a name="python"></a>[Python](#tab/python)

Voici la stratégie de nouvelles tentatives dans le fichier *function.json* :

```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "exponentialBackoff",
        "maxRetryCount": 5,
        "minimumInterval": "00:00:10",
        "maximumInterval": "00:15:00"
    }
}
```

# <a name="java"></a>[Java](#tab/java)

Voici la stratégie de nouvelles tentatives dans le fichier *function.json* :

```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "exponentialBackoff",
        "maxRetryCount": 5,
        "minimumInterval": "00:00:10",
        "maximumInterval": "00:15:00"
    }
}
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Voici la stratégie de nouvelles tentatives dans le fichier *function.json* :

```json
{
    "disabled": false,
    "bindings": [
        {
            ....
        }
    ],
    "retry": {
        "strategy": "exponentialBackoff",
        "maxRetryCount": 5,
        "minimumInterval": "00:00:10",
        "maximumInterval": "00:15:00"
    }
}
```
---

|Propriété function.json  |Propriété d'attribut | Description |
|---------|---------|---------| 
|stratégie|Non applicable|Obligatoire. Stratégie de nouvelle tentative à utiliser. Les valeurs valides sont `fixedDelay` ou `exponentialBackoff`.|
|maxRetryCount|Non applicable|Obligatoire. Nombre maximal de nouvelles tentatives autorisées par exécution de fonction. `-1` signifie qu’il faut effectuer ces nouvelles tentatives indéfiniment.|
|delayInterval|Non applicable|Délai utilisé entre chaque nouvelle tentative lors de l’utilisation d’une stratégie `fixedDelay`.|
|minimumInterval|Non applicable|Délai minimal de nouvelle tentative lors de l’utilisation de la stratégie `exponentialBackoff`.|
|maximumInterval|Non applicable|Délai maximal de nouvelle tentative lors de l’utilisation de la stratégie `exponentialBackoff`.| 

### <a name="retry-limitations-during-preview"></a>Limitations relatives aux nouvelles tentatives pendant la préversion

- Pour les projets .NET, vous devrez peut-être extraire manuellement une version de [GitHub AE](https://www.nuget.org/packages/Microsoft.Azure.WebJobs) >= 3.0.23.
- Dans le plan consommation, il est possible d’effectuer un scale-down de l’application jusqu’à zéro tout en renouvelant les messages finaux dans une file d’attente.
- Dans le plan consommation, il est possible d’effectuer un scale-down de l’application en effectuant des nouvelles tentatives.  Pour de meilleurs résultats, choisissez un intervalle avant nouvelle tentative <= 00:01:00 et <= 5 nouvelles tentatives.

## <a name="using-retry-support-on-top-of-trigger-resilience"></a>Utilisation de la prise en charge des nouvelles tentatives en plus de la résilience de déclencheur

La stratégie de nouvelles tentatives d’application de fonction est indépendante des nouvelles tentatives ou de la résilience fournies par le déclencheur.  La stratégie de nouvelles tentatives de fonction ne fera que s’ajouter à une nouvelle tentative résiliente du déclencheur.  Par exemple, si vous utilisez Azure Service Bus, les files d’attente par défaut ont un nombre de remises de messages de 10.  Le nombre de remises par défaut signifie qu’au bout de 10 tentatives de remise d’un message de file d’attente, Service Bus mettra le message en file d’attente de lettres mortes.  Vous pouvez définir une stratégie de nouvelles tentatives pour une fonction qui a un déclencheur Service Bus, mais les nouvelles tentatives se superposeront aux tentatives de remise Service Bus.  

Par exemple, si vous avez utilisé le nombre par défaut de 10 remises Service Bus, et que vous avez défini une stratégie de nouvelles tentatives de fonction de 5.  Le message est tout d’abord sorti de la file d’attente, ce qui porte le nombre de remise Service Bus à 1.  Si chaque exécution a échoué, après cinq tentatives de déclenchement du même message, ce message est marqué comme abandonné.  Service Bus remet immédiatement en file d’attente le message, déclenche la fonction et porte le nombre de remises à 2.  Enfin, après 50 tentatives éventuelles (10 remises Service Bus * 5 tentatives de fonction par remise), le message est abandonné et déclenche une lettre morte sur Service Bus.

> [!WARNING]
> Il n’est pas recommandé de définir le nombre de remises pour un déclencheur comme Files d’attente Service Bus sur 1, où le message serait immédiatement mis en file d’attente de lettres mortes après un seul cycle de nouvelles tentatives de fonction.  Cela s’explique par le fait que les déclencheurs offrent une certaine résilience lors des nouvelles tentatives, tandis que la stratégie de nouvelle tentative de la fonction est plus efficace et peut aboutir à un nombre de tentatives inférieur au nombre total souhaité.

### <a name="triggers-with-additional-resiliency-or-retries"></a>Déclencheurs avec résilience ou nouvelles tentatives supplémentaires

Les déclencheurs suivants prennent en charge les nouvelles tentatives au niveau de la source du déclencheur :

* [stockage d’objets blob Azure](../articles/azure-functions/functions-bindings-storage-blob.md)
* [Stockage File d’attente Azure](../articles/azure-functions/functions-bindings-storage-queue.md)
* [Azure Service Bus (file d’attente/rubrique)](../articles/azure-functions/functions-bindings-service-bus.md)

Par défaut, la plupart des déclencheurs retentent les requêtes jusqu’à cinq fois. Après la cinquième tentative, le stockage File d’attente Azure écrit un message dans une [file d’attente de messages incohérents](../articles/azure-functions/functions-bindings-storage-queue-trigger.md#poison-messages).  La stratégie par défaut en matière de file d’attente et de rubrique Service Bus consiste à écrire un message dans une [file d’attente de lettres mortes](../articles/service-bus-messaging/service-bus-dead-letter-queues.md) après 10 tentatives.
