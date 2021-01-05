---
title: Déclencheur de minuteur pour Azure Functions
description: Découvrez comment utiliser des déclencheurs de minuteur dans Azure Functions.
author: craigshoemaker
ms.assetid: d2f013d1-f458-42ae-baf8-1810138118ac
ms.topic: reference
ms.date: 11/18/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 0d9852659801040d64fe4143f024fd52ffec16ee
ms.sourcegitcommit: 642988f1ac17cfd7a72ad38ce38ed7a5c2926b6c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94874081"
---
# <a name="timer-trigger-for-azure-functions"></a>Déclencheur de minuteur pour Azure Functions

Cet article explique comment utiliser des déclencheurs de minuteur dans Azure Functions. Un déclencheur de minuteur vous permet d’exécuter une fonction de manière planifiée.

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

Pour savoir comment exécuter manuellement une fonction déclenchée par un minuteur, consultez [Exécuter manuellement une fonction non déclenchée via HTTP](./functions-manually-run-non-http.md).

## <a name="packages---functions-2x-and-higher"></a>Packages – Functions 2.x et versions ultérieures

Le déclencheur de minuteur est fourni dans le package NuGet [Microsoft.Azure.WebJobs.Extensions](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions) version 3.x. Le code source du package se trouve dans le référentiel GitHub [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/).

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

## <a name="packages---functions-1x"></a>Packages - Functions 1.x

Le déclencheur de minuteur est fourni dans le package NuGet [Microsoft.Azure.WebJobs.Extensions](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions) version 2.x. Le code source du package se trouve dans le référentiel GitHub [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions/Extensions/Timers/).

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

## <a name="example"></a>Exemple

# <a name="c"></a>[C#](#tab/csharp)

L’exemple suivant montre une [fonction C#](functions-dotnet-class-library.md) qui est exécutée chaque fois que les minutes ont une valeur divisible par 5 (par exemple, si la fonction commence à 18:57:00, l’exécution suivante aura lieu à 19:00:00). L’objet [`TimerInfo`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/TimerInfo.cs) est transmis à la fonction.

```cs
[FunctionName("TimerTriggerCSharp")]
public static void Run([TimerTrigger("0 */5 * * * *")]TimerInfo myTimer, ILogger log)
{
    if (myTimer.IsPastDue)
    {
        log.LogInformation("Timer is running late!");
    }
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

L’exemple suivant montre une liaison de déclencheur de minuteur dans un fichier *function.json* et une [fonction de script C#](functions-reference-csharp.md) qui utilise la liaison. La fonction écrit un journal indiquant si cet appel de fonction est dû à une occurrence de planification manquée. L’objet [`TimerInfo`](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/TimerInfo.cs) est transmis à la fonction.

Voici les données de liaison dans le fichier *function.json* :

```json
{
    "schedule": "0 */5 * * * *",
    "name": "myTimer",
    "type": "timerTrigger",
    "direction": "in"
}
```

Voici le code Script C# :

```csharp
public static void Run(TimerInfo myTimer, ILogger log)
{
    if (myTimer.IsPastDue)
    {
        log.LogInformation("Timer is running late!");
    }
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}" );  
}
```

# <a name="java"></a>[Java](#tab/java)

L’exemple suivant lance et exécute la fonction toutes les cinq minutes. L’annotation `@TimerTrigger` sur la fonction définit le programme avec le même format de chaîne de caractères que les [expressions CRON](https://en.wikipedia.org/wiki/Cron#CRON_expression).

```java
@FunctionName("keepAlive")
public void keepAlive(
  @TimerTrigger(name = "keepAliveTrigger", schedule = "0 */5 * * * *") String timerInfo,
      ExecutionContext context
 ) {
     // timeInfo is a JSON string, you can deserialize it to an object using your favorite JSON library
     context.getLogger().info("Timer is triggered: " + timerInfo);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant montre une liaison de déclencheur de minuteur dans un fichier *function.json* et une [fonction JavaScript](functions-reference-node.md) qui utilise la liaison. La fonction écrit un journal indiquant si cet appel de fonction est dû à une occurrence de planification manquée. L’[objet de minuteur](#usage) est transmis à la fonction.

Voici les données de liaison dans le fichier *function.json* :

```json
{
    "schedule": "0 */5 * * * *",
    "name": "myTimer",
    "type": "timerTrigger",
    "direction": "in"
}
```

Voici le code JavaScript :

```JavaScript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();

    if (myTimer.IsPastDue)
    {
        context.log('Node is running late!');
    }
    context.log('Node timer trigger function ran!', timeStamp);   

    context.done();
};
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

L’exemple suivant montre comment configurer le fichier *function.json* et *run.ps1* pour un déclencheur de minuteur dans [PowerShell](./functions-reference-powershell.md).

```json
{
  "bindings": [
    {
      "name": "Timer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "0 */5 * * * *"
    }
  ]
}
```

```powershell
# Input bindings are passed in via param block.
param($Timer)

# Get the current universal time in the default string format.
$currentUTCtime = (Get-Date).ToUniversalTime()

# The 'IsPastDue' property is 'true' when the current function invocation is later than scheduled.
if ($Timer.IsPastDue) {
    Write-Host "PowerShell timer is running late!"
}

# Write an information log with the current time.
Write-Host "PowerShell timer trigger function ran! TIME: $currentUTCtime"
```

Une instance de l’[objet de minuteur](#usage) est transmise comme premier argument à la fonction.

# <a name="python"></a>[Python](#tab/python)

L’exemple suivant utilise une liaison de déclencheur de minuteur dont la configuration est décrite dans le fichier *function.json*. La véritable [fonction Python](functions-reference-python.md) qui utilise la liaison est décrite dans le fichier *__init__.py*. L’objet transmis à la fonction est de type [Objet azure.functions.TimerRequest](/python/api/azure-functions/azure.functions.timerrequest). La logique de fonction écrit dans les journaux indiquant si l’appel est dû à une occurrence de planification manquée.

Voici les données de liaison dans le fichier *function.json* :

```json
{
    "name": "mytimer",
    "type": "timerTrigger",
    "direction": "in",
    "schedule": "0 */5 * * * *"
}
```

Voici le code Python :

```python
import datetime
import logging

import azure.functions as func


def main(mytimer: func.TimerRequest) -> None:
    utc_timestamp = datetime.datetime.utcnow().replace(
        tzinfo=datetime.timezone.utc).isoformat()

    if mytimer.past_due:
        logging.info('The timer is past due!')

    logging.info('Python timer trigger function ran at %s', utc_timestamp)
```

---

## <a name="attributes-and-annotations"></a>Attributs et annotations

# <a name="c"></a>[C#](#tab/csharp)

Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md), utilisez l’attribut [TimerTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/TimerTriggerAttribute.cs).

Le constructeur de l’attribut prend une expression CRON ou `TimeSpan`. Vous pouvez utiliser `TimeSpan` uniquement si l’application de fonction s’exécute sur un plan App Service. `TimeSpan` n’est pas pris en charge pour la consommation ou les fonctions Premium élastiques.

L’exemple suivant illustre une expression CRON :

```csharp
[FunctionName("TimerTriggerCSharp")]
public static void Run([TimerTrigger("0 */5 * * * *")]TimerInfo myTimer, ILogger log)
{
    if (myTimer.IsPastDue)
    {
        log.LogInformation("Timer is running late!");
    }
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
}
```

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="java"></a>[Java](#tab/java)

L’annotation `@TimerTrigger` sur la fonction définit le programme avec le même format de chaîne de caractères que les [expressions CRON](https://en.wikipedia.org/wiki/Cron#CRON_expression).

```java
@FunctionName("keepAlive")
public void keepAlive(
  @TimerTrigger(name = "keepAliveTrigger", schedule = "0 */5 * * * *") String timerInfo,
      ExecutionContext context
 ) {
     // timeInfo is a JSON string, you can deserialize it to an object using your favorite JSON library
     context.getLogger().info("Timer is triggered: " + timerInfo);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Les attributs ne sont pas pris en charge par PowerShell.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

---

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `TimerTrigger`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
|**type** | n/a | Doit avoir la valeur « timerTrigger ». Cette propriété est définie automatiquement lorsque vous créez le déclencheur dans le portail Azure.|
|**direction** | n/a | Doit être défini sur « in ». Cette propriété est définie automatiquement lorsque vous créez le déclencheur dans le portail Azure. |
|**name** | n/a | Nom de la variable qui représente l’objet de minuteur dans le code de la fonction. | 
|**schedule**|**ScheduleExpression**|Une [expression CRON](#ncrontab-expressions) ou une valeur [TimeSpan](#timespan). `TimeSpan` peut être utilisé uniquement pour une application de fonction qui s’exécute sur un plan App Service. Vous pouvez placer l’expression de planification dans un paramètre d’application et définir cette propriété selon le nom du paramètre d’application encapsulé dans les signes **%** , comme sur cet exemple : « %ScheduleAppSetting% ». |
|**runOnStartup**|**RunOnStartup**|Si la valeur est `true`, la fonction est appelée au démarrage du runtime. Par exemple, le runtime démarre lorsque l’application de fonction sort de veille après une période d’inactivité. Lorsque l’application de fonction redémarre en raison de modifications apportées à la fonction, et lorsque l’application de fonction augmente la taille des instances. Par conséquent, la propriété **runOnStartup** doit être rarement, voire jamais, définie sur `true`, notamment en production. |
|**useMonitor**|**UseMonitor**|Peut-être défini sur la valeur `true` ou `false` pour indiquer si la planification doit être surveillée ou non. La surveillance de planification conserve les occurrences de planification pour garantir la maintenance correcte de cette dernière même en cas de redémarrage des instances de l’application de fonction. Si elle n’est pas définie explicitement, la valeur par défaut est `true` pour les planifications dont l’intervalle de récurrence est supérieur ou égal à 1 minute. Pour les planifications qui se déclenchent plusieurs fois par minute, la valeur par défaut est `false`.

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

> [!CAUTION]
> Nous vous déconseillons de définir la propriété **runOnStartup** sur `true` en production. Si vous utilisez cette définition, le code s’exécute à des moments extrêmement imprévisibles. Dans certains paramètres de production, ces exécutions supplémentaires peuvent entraîner des coûts sensiblement plus élevés pour les applications hébergées dans les plans Consommation. Par exemple, avec la propriété **runOnStartup** activée, le déclencheur est appelé à chaque fois que votre Function App est mise à l’échelle. Veillez à bien comprendre le comportement en production de vos fonctions avant d’activer la propriété **runOnStartup** en production.

## <a name="usage"></a>Usage

Lorsqu’une fonction de déclencheur de minuteur est appelée, l’objet minuteur est transmis à la fonction. Le code JSON suivant est un exemple de représentation de l’objet minuteur.

```json
{
    "schedule":{
    },
    "scheduleStatus": {
        "last":"2016-10-04T10:15:00+00:00",
        "lastUpdated":"2016-10-04T10:16:00+00:00",
        "next":"2016-10-04T10:20:00+00:00"
    },
    "isPastDue":false
}
```

La propriété `isPastDue` est `true` lorsque l’appel de fonction en cours arrive plus tard que prévu. Par exemple, un redémarrage de la fonction d’application peut entraîner l’échec d’un appel.

## <a name="ncrontab-expressions"></a>Expressions NCRONTAB

Azure Functions utilise la bibliothèque [NCronTab](https://github.com/atifaziz/NCrontab) pour interpréter les expressions NCRONTAB. Une expression NCRONTAB est semblable à une expression CRON, à ceci près qu’elle comprend un sixième champ supplémentaire au début pour utiliser la précision de temps en secondes :

`{second} {minute} {hour} {day} {month} {day-of-week}`

Chaque champ peut être associé aux types de valeurs suivants :

|Type  |Exemple  |En cas de déclenchement  |
|---------|---------|---------|
|Une valeur spécifique |<nobr>`0 5 * * * *`</nobr>| Une fois par heure du jour, à la 5e minute de chaque heure |
|Toutes les valeurs (`*`)|<nobr>`0 * 5 * * *`</nobr>| Une fois par minute, à partir de la 5e heure |
|Une plage (opérateur `-`)|<nobr>`5-7 * * * * *`</nobr>| Trois fois par minute - aux secondes 5 à 7 de chaque minute de chaque heure de chaque jour |
|Un ensemble de valeurs (opérateur `,`)|<nobr>`5,8,10 * * * * *`</nobr>| Trois fois par minute - aux secondes 5, 8 et 10 de chaque minute de chaque heure de chaque jour |
|Une valeur d’intervalle (opérateur `/`)|<nobr>`0 */5 * * * *`</nobr>| 12 fois par heure - à la seconde 0 de la 5e minute de chaque heure de chaque jour |

[!INCLUDE [functions-cron-expressions-months-days](../../includes/functions-cron-expressions-months-days.md)]

### <a name="ncrontab-examples"></a>Exemples NCRONTAB

Voici quelques exemples d’expressions NCRONTAB que vous pouvez utiliser pour le déclencheur de minuteur dans Azure Functions.

| Exemple            | En cas de déclenchement                     |
|--------------------|------------------------------------|
| `0 */5 * * * *`    | une fois toutes les cinq minutes            |
| `0 0 * * * *`      | une fois toutes les heures      |
| `0 0 */2 * * *`    | une fois toutes les deux heures               |
| `0 0 9-17 * * *`   | une fois toutes les heures entre 9h et 17h  |
| `0 30 9 * * *`     | à 9h30 tous les jours               |
| `0 30 9 * * 1-5`   | à 9h30 tous les jours de la semaine           |
| `0 30 9 * Jan Mon` | à 9h30 tous les lundis en janvier |

> [!NOTE]
> L'expression NCRONTAB nécessite le format à **six champs**. Le sixième champ comprend une valeur correspondant aux secondes qui est placée au début de l'expression. Les expressions cron à cinq champs ne sont pas prises en charge dans Azure.

### <a name="ncrontab-time-zones"></a>Fuseaux horaires NCRONTAB

Les nombres d’une expression CRON font référence à une heure et à une date, pas à un intervalle de temps. Par exemple, un 5 dans le champ `hour` correspond à 17h, pas à toutes les 5 heures.

[!INCLUDE [functions-timezone](../../includes/functions-timezone.md)]

## <a name="timespan"></a>TimeSpan

 `TimeSpan` peut être utilisé uniquement pour une application de fonction qui s’exécute sur un plan App Service.

Contrairement à une expression CRON, une valeur `TimeSpan` spécifie l’intervalle de temps entre chaque appel de fonction. Quand une fonction se termine après une exécution plus longue que l’intervalle spécifié, le minuteur rappelle immédiatement la fonction.

Exprimé sous forme de chaîne, le format `TimeSpan` est `hh:mm:ss` lorsque la valeur de `hh` est inférieure à 24. Lorsque les deux premiers chiffres sont 24 ou plus, le format est `dd:hh:mm`. Voici quelques exemples :

| Exemple      | En cas de déclenchement |
|--------------|----------------|
| "01:00:00"   | toutes les heures     |
| "00:01:00"   | chaque minute   |
| "24:00:00"   | tous les 24 jours  |
| "1.00:00:00" | chaque jour      |

## <a name="scale-out"></a>Montée en charge

Si une application de fonction augmente la taille de plusieurs instances, une seule instance de fonction déclenchée par minuteur est exécutée sur toutes les instances.

## <a name="function-apps-sharing-storage"></a>Applications de fonction qui partagent du stockage

Si vous partagez des comptes de stockage entre des applications de fonction qui ne sont pas déployées sur App Service, vous devrez peut-être affecter explicitement l’ID d’hôte à chaque application.

| Version de Functions | Paramètre                                              |
| ----------------- | ---------------------------------------------------- |
| 2.x (et ultérieures)  | `AzureFunctionsWebHost__hostid` variable d’environnement |
| 1.x               | `id` dans *host.json*                                  |

Vous pouvez omettre la valeur d’identification ou définir manuellement la configuration d’identification de chaque application de fonction sur une valeur différente.

Le déclencheur du minuteur utilise un verrou de stockage pour s’assurer qu’il n’y a qu’une seule instance du minuteur lorsqu’une application de fonction augmente la taille de plusieurs instances. Si deux applications de fonction partagent la même configuration d’identification et que chacune utilise un déclencheur de minuteur, un seul minuteur s’exécute.

## <a name="retry-behavior"></a>Comportement pour les nouvelles tentatives

À la différence du déclencheur de file d’attente, le déclencheur de minuteur n’effectue pas de nouvelle tentative après l’échec d’une fonction. En cas d’échec d’une fonction, elle n’est pas rappelée avant la prochaine période planifiée.

## <a name="troubleshooting"></a>Dépannage

Pour plus d’informations sur la procédure à suivre lorsque le déclencheur du minuteur ne fonctionne pas comme prévu, consultez [Investigating and reporting issues with timer triggered functions not firing](https://github.com/Azure/azure-functions-host/wiki/Investigating-and-reporting-issues-with-timer-triggered-functions-not-firing) (Examen et rapport des problèmes concernant l’absence de déclenchement des fonctions déclenchées par minuteur).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Accéder à un guide de démarrage rapide qui utilise un déclencheur de minuteur](functions-create-scheduled-function.md)

> [!div class="nextstepaction"]
> [En savoir plus sur les déclencheurs et les liaisons Azure Functions](functions-triggers-bindings.md)
