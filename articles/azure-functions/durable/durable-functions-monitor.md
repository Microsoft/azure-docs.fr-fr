---
title: Moniteurs dans l’extension Fonctions durables - Azure
description: Découvrez comment implémenter un moniteur d’état à l’aide de l’extension Fonctions durables pour Azure Functions.
ms.topic: conceptual
ms.date: 12/07/2018
ms.author: azfuncdf
ms.openlocfilehash: ed92156df9d8e1e07b56cea4b1e64edee11d68d9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77562120"
---
# <a name="monitor-scenario-in-durable-functions---weather-watcher-sample"></a>Scénario de surveillance dans l’extension Fonctions durables - Exemple d’observateur météo

Le modèle de surveillance fait référence à un processus *récurrent* flexible dans un flux de travail, par exemple l’interrogation jusqu’à ce que certaines conditions soient respectées. Cet article décrit un exemple qui utilise l’extension [Fonctions durables](durable-functions-overview.md) pour implémenter la surveillance.

[!INCLUDE [durable-functions-prerequisites](../../../includes/durable-functions-prerequisites.md)]

## <a name="scenario-overview"></a>Présentation du scénario

Cet exemple surveille les conditions météorologiques actuelles d’un lieu et alerte un utilisateur par SMS quand le ciel est clair. Vous pouvez utiliser une fonction normale déclenchée par un minuteur pour vérifier la météo et envoyer des alertes. Toutefois, cette approche pose le problème de la **gestion de la durée de vie**. Si une seule alerte doit être envoyée, le moniteur doit se désactiver une fois qu’un temps clair a été détecté. Le modèle de surveillance peut, entre autres avantages, mettre fin à sa propre exécution :

* Les moniteurs s’exécutent selon une périodicité, et non selon des planifications : un déclencheur de minuteur *s’exécute* toutes les heures ; un moniteur *attend* une heure entre les actions. Sauf indication contraire, les actions d’un moniteur ne se superposent pas, ce qui peut être important pour les tâches longues.
* Les moniteurs peuvent avoir des intervalles dynamiques : le délai d’attente peut changer en fonction de certaines conditions.
* Les moniteurs peuvent s’arrêter quand une condition donnée est respectée ou être interrompus par un autre processus.
* Les moniteurs peuvent prendre des paramètres. L’exemple montre comment le même processus de surveillance météorologique peut être appliqué à n’importe quels emplacement et numéro de téléphone demandés.
* Les moniteurs sont évolutifs. Étant donné que chaque moniteur est une instance d’orchestration, il est possible de créer plusieurs moniteurs sans avoir à créer de nouvelles fonctions ou à définir davantage de code.
* Les moniteurs s’intègrent facilement à de plus grands flux de travail. Un moniteur peut être une section d’une fonction d’orchestration plus complexe, ou une [sous-orchestration](durable-functions-sub-orchestrations.md).

## <a name="configuration"></a>Configuration

### <a name="configuring-twilio-integration"></a>Configuration de l’intégration de Twilio

[!INCLUDE [functions-twilio-integration](../../../includes/functions-twilio-integration.md)]

### <a name="configuring-weather-underground-integration"></a>Configuration de l’intégration de Weather Underground

Cet exemple implique l’utilisation de l’API Weather Underground pour vérifier les conditions météorologiques actuelles pour un lieu.

La première chose dont vous avez besoin est un compte Weather Underground. Vous pouvez en créer un gratuitement à l’adresse [https://www.wunderground.com/signup](https://www.wunderground.com/signup). Une fois que vous avez un compte, vous devez acquérir une clé d’API. Pour ce faire, visitez le site [https://www.wunderground.com/weather/api](https://www.wunderground.com/weather/api/?MR=1), puis sélectionnez les paramètres de la clé. Le plan Stratus Developer est gratuit et suffisant pour exécuter cet exemple.

Une fois que vous disposez d’une clé API, ajoutez le **paramètre d’application** suivant à votre application de fonction.

| Nom du paramètre d’application | Description de la valeur |
| - | - |
| **WeatherUndergroundApiKey**  | Clé de votre API Weather Underground. |

## <a name="the-functions"></a>Les fonctions

Cet article explique les fonctions suivantes dans l’exemple d’application :

* `E3_Monitor`: [fonction d’orchestrateur](durable-functions-bindings.md#orchestration-trigger) qui appelle régulièrement `E3_GetIsClear`. Elle appelle `E3_SendGoodWeatherAlert` si `E3_GetIsClear` retourne la valeur true.
* `E3_GetIsClear`: [fonction d’activité](durable-functions-bindings.md#activity-trigger) qui vérifie les conditions météorologiques actuelles pour un lieu.
* `E3_SendGoodWeatherAlert`: fonction d’activité qui envoie un SMS par le biais de Twilio.

### <a name="e3_monitor-orchestrator-function"></a>Fonction d'orchestrateur E3_Monitor

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/Monitor.cs?range=41-78,97-115)]

L’orchestrateur nécessite un emplacement à surveiller et un numéro de téléphone auquel envoyer un message lorsque la météo s'améliore à l’emplacement. Ces données sont transmises à l’orchestrateur sous forme d’objet `MonitorRequest` fortement typé.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

La fonction **E3_Monitor** utilise le fichier *function.json* standard pour les fonctions d’orchestrateur.

[!code-json[Main](~/samples-durable-functions/samples/javascript/E3_Monitor/function.json)]

Voici le code qui implémente la fonction :

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E3_Monitor/index.js)]

---

Cette fonction d’orchestrateur effectue les actions suivantes :

1. Obtient la valeur **MonitorRequest** constituée du *lieu* à surveiller et du *numéro de téléphone* auquel il envoie une notification par SMS.
2. Détermine le délai d’expiration du moniteur. Par souci de concision, l’exemple utilise une valeur codée en dur.
3. Appelle **E3_GetIsClear** pour déterminer si le ciel est dégagé au lieu demandé.
4. Si le temps est clair, appelle **E3_SendGoodWeatherAlert** pour envoyer une notification par SMS au numéro de téléphone demandé.
5. Crée un minuteur durable pour reprendre l’orchestration à l’intervalle d’interrogation suivant. Par souci de concision, l’exemple utilise une valeur codée en dur.
6. Continue de s’exécuter jusqu’à ce que l'heure UTC actuelle passe l’heure d’expiration du moniteur, ou une alerte SMS est envoyée.

Plusieurs instances d’orchestrateur peuvent s’exécuter simultanément en appelant la fonction d’orchestrateur plusieurs fois. Le lieu à surveiller et le numéro de téléphone auquel envoyer une alerte SMS peuvent être spécifiés.

### <a name="e3_getisclear-activity-function"></a>Fonction d'activité E3_GetIsClear

Comme avec d’autres exemples, les fonctions d’activité d’assistance sont des fonctions régulières qui utilisent la liaison de déclencheur `activityTrigger`. La fonction **E3_GetIsClear** obtient les conditions météorologiques actuelles à l’aide de l’API Weather Underground et détermine si le ciel est clair.

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/Monitor.cs?range=80-85)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Le fichier *function.json* est défini comme suit :

[!code-json[Main](~/samples-durable-functions/samples/javascript/E3_GetIsClear/function.json)]

Et voici l’implémentation.

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E3_GetIsClear/index.js)]

---

### <a name="e3_sendgoodweatheralert-activity-function"></a>Fonction d'activité E3_SendGoodWeatherAlert

La fonction **E3_SendGoodWeatherAlert** utilise la liaison Twilio pour envoyer un SMS à l’utilisateur final pour l’avertir que c’est le moment d’aller faire une promenade.

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/Monitor.cs?range=87-96,140-205)]

> [!NOTE]
> Vous devez installer le package Nuget `Microsoft.Azure.WebJobs.Extensions.Twilio` pour exécuter l’exemple de code.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Son fichier *function.json* est simple :

[!code-json[Main](~/samples-durable-functions/samples/javascript/E3_SendGoodWeatherAlert/function.json)]

Et voici le code qui envoie le SMS :

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E3_SendGoodWeatherAlert/index.js)]

---

## <a name="run-the-sample"></a>Exécution de l'exemple

En utilisant les fonctions déclenchées via HTTP incluses dans l’exemple, vous pouvez démarrer l’orchestration en envoyant la requête HTTP POST suivante :

```
POST https://{host}/orchestrators/E3_Monitor
Content-Length: 77
Content-Type: application/json

{ "location": { "city": "Redmond", "state": "WA" }, "phone": "+1425XXXXXXX" }
```

```
HTTP/1.1 202 Accepted
Content-Type: application/json; charset=utf-8
Location: https://{host}/runtime/webhooks/durabletask/instances/f6893f25acf64df2ab53a35c09d52635?taskHub=SampleHubVS&connection=Storage&code={SystemKey}
RetryAfter: 10

{"id": "f6893f25acf64df2ab53a35c09d52635", "statusQueryGetUri": "https://{host}/runtime/webhooks/durabletask/instances/f6893f25acf64df2ab53a35c09d52635?taskHub=SampleHubVS&connection=Storage&code={systemKey}", "sendEventPostUri": "https://{host}/runtime/webhooks/durabletask/instances/f6893f25acf64df2ab53a35c09d52635/raiseEvent/{eventName}?taskHub=SampleHubVS&connection=Storage&code={systemKey}", "terminatePostUri": "https://{host}/runtime/webhooks/durabletask/instances/f6893f25acf64df2ab53a35c09d52635/terminate?reason={text}&taskHub=SampleHubVS&connection=Storage&code={systemKey}"}
```

L’instance d’**E3_Monitor** démarre et interroge les conditions météorologiques actuelles pour le lieu demandé. Si le temps est clair, elle appelle une fonction d’activité pour envoyer une alerte ; dans le cas contraire, elle définit un minuteur. Quand le minuteur expire, l’orchestration reprend.

Vous pouvez voir l’activité de l’orchestration en examinant les journaux d’activité de fonction dans le portail Azure Functions.

```
2018-03-01T01:14:41.649 Function started (Id=2d5fcadf-275b-4226-a174-f9f943c90cd1)
2018-03-01T01:14:42.741 Started orchestration with ID = '1608200bb2ce4b7face5fc3b8e674f2e'.
2018-03-01T01:14:42.780 Function completed (Success, Id=2d5fcadf-275b-4226-a174-f9f943c90cd1, Duration=1111ms)
2018-03-01T01:14:52.765 Function started (Id=b1b7eb4a-96d3-4f11-a0ff-893e08dd4cfb)
2018-03-01T01:14:52.890 Received monitor request. Location: Redmond, WA. Phone: +1425XXXXXXX.
2018-03-01T01:14:52.895 Instantiating monitor for Redmond, WA. Expires: 3/1/2018 7:14:52 AM.
2018-03-01T01:14:52.909 Checking current weather conditions for Redmond, WA at 3/1/2018 1:14:52 AM.
2018-03-01T01:14:52.954 Function completed (Success, Id=b1b7eb4a-96d3-4f11-a0ff-893e08dd4cfb, Duration=189ms)
2018-03-01T01:14:53.226 Function started (Id=80a4cb26-c4be-46ba-85c8-ea0c6d07d859)
2018-03-01T01:14:53.808 Function completed (Success, Id=80a4cb26-c4be-46ba-85c8-ea0c6d07d859, Duration=582ms)
2018-03-01T01:14:53.967 Function started (Id=561d0c78-ee6e-46cb-b6db-39ef639c9a2c)
2018-03-01T01:14:53.996 Next check for Redmond, WA at 3/1/2018 1:44:53 AM.
2018-03-01T01:14:54.030 Function completed (Success, Id=561d0c78-ee6e-46cb-b6db-39ef639c9a2c, Duration=62ms)
```

L’orchestration [s’arrête](durable-functions-instance-management.md) une fois que son délai d’attente est atteint ou qu’un ciel clair est détecté. Vous pouvez également utiliser `TerminateAsync` (.NET) ou `terminate` (JavaScript) à l’intérieur d’une autre fonction, ou appeler le Webhook HTTP POST **terminatePostUri** référencé dans la réponse 202 ci-dessus, en remplaçant `{text}` par le motif de l’arrêt :

```
POST https://{host}/runtime/webhooks/durabletask/instances/f6893f25acf64df2ab53a35c09d52635/terminate?reason=Because&taskHub=SampleHubVS&connection=Storage&code={systemKey}
```

## <a name="next-steps"></a>Étapes suivantes

Cet exemple a montré comment utiliser l’extension Fonctions durables pour surveiller l’état d’une source externe à l’aide de [minuteurs durables](durable-functions-timers.md) et d’une logique conditionnelle. L’exemple suivant montre comment utiliser les événements externes et les [minuteurs durables](durable-functions-timers.md) pour gérer l’interaction humaine.

> [!div class="nextstepaction"]
> [Exécuter l’exemple d’interaction humaine](durable-functions-phone-verification.md)
