---
title: Informations de référence sur le fichier host.json pour Azure Functions 1.x
description: Documentation de référence pour le fichier host.json d’Azure Functions avec le runtime v1.
ms.topic: conceptual
ms.date: 10/19/2018
ms.openlocfilehash: 588ab6723015f34d15e4a46ec4f7324302b13b81
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94832821"
---
# <a name="hostjson-reference-for-azure-functions-1x"></a>Informations de référence sur le fichier host.json pour Azure Functions 1.x

> [!div class="op_single_selector" title1="Sélectionnez la version du runtime Azure Functions que vous utilisez : "]
> * [Version 1](functions-host-json-v1.md)
> * [Version 2](functions-host-json.md)

Le fichier de métadonnées *host.json* contient les options de configuration globale qui affectent l’ensemble des fonctions d’une application de fonction. Cet article répertorie les paramètres qui sont disponibles pour le runtime v1. Le schéma JSON est sur http://json.schemastore.org/host.

> [!NOTE]
> Cet article concerne Azure Functions 1.x.  Pour obtenir des informations de référence sur le fichier host.json dans Functions 2.x et versions ultérieures, consultez [Informations de référence sur le fichier host.json pour Azure Functions 2.x](functions-host-json.md).

Les autres options de configuration d’application de fonction sont managées dans vos [paramètres d’application](functions-app-settings.md).

Certains paramètres host.json sont uniquement utilisés lors de l’exécution locale dans le fichier [local.settings.json](functions-run-local.md#local-settings-file).

## <a name="sample-hostjson-file"></a>Exemple de fichier host.json

L’exemple de fichier *host.json* suivant contient toutes les options possibles spécifiées.


```json
{
    "aggregator": {
        "batchSize": 1000,
        "flushTimeout": "00:00:30"
    },
    "applicationInsights": {
        "sampling": {
          "isEnabled": true,
          "maxTelemetryItemsPerSecond" : 5
        }
    },
    "documentDB": {
        "connectionMode": "Gateway",
        "protocol": "Https",
        "leaseOptions": {
            "leasePrefix": "prefix"
        }
    },
    "eventHub": {
      "maxBatchSize": 64,
      "prefetchCount": 256,
      "batchCheckpointFrequency": 1
    },
    "functions": [ "QueueProcessor", "GitHubWebHook" ],
    "functionTimeout": "00:05:00",
    "healthMonitor": {
        "enabled": true,
        "healthCheckInterval": "00:00:10",
        "healthCheckWindow": "00:02:00",
        "healthCheckThreshold": 6,
        "counterThreshold": 0.80
    },
    "http": {
        "routePrefix": "api",
        "maxOutstandingRequests": 20,
        "maxConcurrentRequests": 10,
        "dynamicThrottlesEnabled": false
    },
    "id": "9f4ea53c5136457d883d685e57164f08",
    "logger": {
        "categoryFilter": {
            "defaultLevel": "Information",
            "categoryLevels": {
                "Host": "Error",
                "Function": "Error",
                "Host.Aggregator": "Information"
            }
        }
    },
    "queues": {
      "maxPollingInterval": 2000,
      "visibilityTimeout" : "00:00:30",
      "batchSize": 16,
      "maxDequeueCount": 5,
      "newBatchThreshold": 8
    },
    "sendGrid": {
        "from": "Contoso Group <admin@contoso.com>"
    },
    "serviceBus": {
      "maxConcurrentCalls": 16,
      "prefetchCount": 100,
      "autoRenewTimeout": "00:05:00",
      "autoComplete": true
    },
    "singleton": {
      "lockPeriod": "00:00:15",
      "listenerLockPeriod": "00:01:00",
      "listenerLockRecoveryPollingInterval": "00:01:00",
      "lockAcquisitionTimeout": "00:01:00",
      "lockAcquisitionPollingInterval": "00:00:03"
    },
    "tracing": {
      "consoleLevel": "verbose",
      "fileLoggingMode": "debugOnly"
    },
    "watchDirectories": [ "Shared" ],
}
```

Les sections suivantes de cet article expliquent chaque propriété de niveau supérieur. Toutes les autres propriétés sont facultatives sauf indication contraire.

## <a name="aggregator"></a>aggregator

[!INCLUDE [aggregator](../../includes/functions-host-json-aggregator.md)]

## <a name="applicationinsights"></a>applicationInsights

[!INCLUDE [applicationInsights](../../includes/functions-host-json-applicationinsights.md)]

## <a name="documentdb"></a>Base de données de documents

Paramètres de configuration pour le [déclencheur et les liaisons Azure Cosmos DB](functions-bindings-cosmosdb.md).

```json
{
    "documentDB": {
        "connectionMode": "Gateway",
        "protocol": "Https",
        "leaseOptions": {
            "leasePrefix": "prefix1"
        }
    }
}
```

|Propriété  |Default | Description |
|---------|---------|---------|
|GatewayMode|Passerelle|Le mode de connexion utilisé par la fonction lors de la connexion au service Azure Cosmos DB. Les options sont `Direct` et `Gateway`.|
|Protocol|Https|Le protocole de connexion utilisé par la fonction lors de la connexion au service Azure Cosmos DB.  Voir [l’explication des deux modes](../cosmos-db/performance-tips.md#networking).|
|leasePrefix|n/a|Préfixe de bail à utiliser dans toutes les fonctions d’une application.|

## <a name="durabletask"></a>durableTask

[!INCLUDE [durabletask](../../includes/functions-host-json-durabletask.md)]

## <a name="eventhub"></a>eventHub

Paramètres de configuration pour les [déclencheurs et liaisons Event Hub](functions-bindings-event-hubs-trigger.md#functions-1x).

## <a name="functions"></a>functions

Liste des fonctions que l’hôte de travail exécute. Un tableau vide désigne l’exécution de toutes les fonctions. Utilisée uniquement pour une [exécution locale](functions-run-local.md). Dans les applications de fonction dans Azure, vous devez plutôt suivre la procédure décrite dans [Guide pratique pour désactiver des fonctions dans Azure Functions](disable-function.md) pour désactiver des fonctions spécifiques au lieu d’utiliser ce paramètre.

```json
{
    "functions": [ "QueueProcessor", "GitHubWebHook" ]
}
```

## <a name="functiontimeout"></a>functionTimeout

Indique la durée avant expiration du délai de toutes les fonctions. Dans les plans de consommation serverless, la plage valide est comprise entre 1 seconde et 10 minutes, et la valeur par défaut est de 5 minutes. Dans un plan App Service, il n’existe aucune limite globale et la valeur par défaut est _null_, ce qui indique qu’il n’y a pas de délai d’expiration.

```json
{
    "functionTimeout": "00:05:00"
}
```

## <a name="healthmonitor"></a>healthMonitor

Paramètres de configuration de l’[analyse d’intégrité d’hôtes](https://github.com/Azure/azure-webjobs-sdk-script/wiki/Host-Health-Monitor).

```
{
    "healthMonitor": {
        "enabled": true,
        "healthCheckInterval": "00:00:10",
        "healthCheckWindow": "00:02:00",
        "healthCheckThreshold": 6,
        "counterThreshold": 0.80
    }
}
```

|Propriété  |Default | Description |
|---------|---------|---------| 
|enabled|true|Indique si la fonctionnalité est activée. | 
|healthCheckInterval|10 secondes|L’intervalle de temps entre les contrôles d’intégrité périodiques en arrière-plan. | 
|healthCheckWindow|2 minutes|Une fenêtre de temps coulissante utilisée conjointement au paramètre `healthCheckThreshold`.| 
|healthCheckThreshold|6|Nombre maximal de fois où le contrôle d’intégrité peut échouer avant le lancement d’un recyclage de l’hôte.| 
|counterThreshold|0.80|Le seuil auquel un compteur de performance est considéré comme non intègre.| 

## <a name="http"></a>http

Paramètre de configuration pour les [déclencheurs et liaisons http](functions-bindings-http-webhook.md).

```json
{
    "http": {
        "routePrefix": "api",
        "maxOutstandingRequests": 200,
        "maxConcurrentRequests": 100,
        "dynamicThrottlesEnabled": true
    }
}
```

|Propriété  |Default | Description |
|---------|---------|---------| 
|dynamicThrottlesEnabled|false|Lorsqu’il est activé, ce paramètre provoque la vérification périodique des compteurs de performance système, comme les connexions/les threads/les processus/la mémoire/le processeur, etc., par le pipeline de traitement des requêtes. Si l’un de ces compteurs dépasse un seuil supérieur intégré (80 %), les requêtes sont rejetées avec une réponse 429 (trop de demandes) jusqu’à ce qu’un niveau normal soit retrouvé.|
|maxConcurrentRequests|illimité (`-1`)|Nombre maximal de fonctions HTTP exécutées en parallèle. Ce paramètre vous permet de contrôler la concurrence, et ainsi facilite la gestion de l’utilisation des ressources. Par exemple, vous pouvez disposer d’une fonction HTTP qui utilise de nombreuses ressources système (mémoire/processeur/sockets) et qui provoque par conséquent des situations où la concurrence est trop élevée. Vous pouvez également avoir une fonction qui effectue des requête vers un service tiers, et qui émet à ce titre des appels dont le débit doit être limité. Dans ces cas, il peut être pertinent d’appliquer une limitation.|
|maxOutstandingRequests|illimité (`-1`)|Nombre maximal de requêtes en attente qui ont lieu à un moment donné. La limite inclut les requêtes qui sont en file d’attente, mais dont l’exécution n’a pas démarré, ainsi que les exécutions en cours. Toutes les requêtes entrantes au-delà de cette limite sont rejetées avec une réponse 429 « Trop occupé ». Ainsi, les appelants peuvent appliquer des stratégies temporelles de nouvelle tentative et vous êtes en mesure de contrôler les latences maximales de requêtes. Ce paramètre contrôle uniquement la mise en file d’attente qui se produit dans le chemin d’accès d’exécution de l’hôte de script. Les autres files d’attente, telles que la file d’attente des requêtes ASP.NET, sont toujours actives et ne sont pas concernées par ce paramètre.|
|routePrefix|api|Préfixe d’itinéraire qui s’applique à tous les itinéraires. Utilisez une chaîne vide pour supprimer le préfixe par défaut. |

## <a name="id"></a>id

ID unique d’un hôte de travail. Il peut s’agir d’un GUID en minuscules dont les tirets ont été supprimés. Requis lors d’une exécution locale. Lors de l’exécution dans Azure, nous vous recommandons de pas définir de valeur d’ID. Un ID est généré automatiquement dans Azure lorsque `id` est omis. 

Si vous partagez un compte de stockage entre plusieurs applications de fonction, assurez-vous que chaque application de fonction a une `id` différente. Vous pouvez omettre la propriété `id` ou définir manuellement chaque `id` de l’application de fonction sur une autre valeur. Le déclencheur du minuteur utilise un verrou de stockage pour vous assurer qu’il n’y aura qu’une seule instance du minuteur lorsqu’une application de fonction augmentera la taille de plusieurs instances. Si deux applications de fonction partagent le même `id` et que chacune utilise un déclencheur de minuteur, un seul minuteur s’exécutera.

```json
{
    "id": "9f4ea53c5136457d883d685e57164f08"
}
```

## <a name="logger"></a>logger

Contrôle le filtrage des journaux d’activité écrits par un [objet ILogger](functions-dotnet-class-library.md#ilogger) ou par [context.log](functions-reference-node.md#contextlog-method).

```json
{
    "logger": {
        "categoryFilter": {
            "defaultLevel": "Information",
            "categoryLevels": {
                "Host": "Error",
                "Function": "Error",
                "Host.Aggregator": "Information"
            }
        }
    }
}
```

|Propriété  |Default | Description |
|---------|---------|---------| 
|categoryFilter|n/a|Spécifie un filtrage par catégorie.| 
|defaultLevel|Information|Pour toutes les catégories non spécifiées dans le tableau `categoryLevels`, envoie les journaux d’activité de ce niveau et des niveaux supérieurs à Application Insights.| 
|categoryLevels|n/a|Tableau des catégories qui spécifie le niveau de journalisation minimal à envoyer à Application Insights pour chaque catégorie. La catégorie indiquée ici contrôle toutes les catégories qui commencent par la même valeur, et les valeurs plus longues sont prioritaires. Dans l’exemple de fichier *host.json* précédent, toutes les catégories qui commencent par « Host.Aggregator » journalisent au niveau `Information`. Toutes les autres catégories commençant par « Host », comme « Host.Executor », journalisent au niveau `Error`.| 

## <a name="queues"></a>queues

Paramètre de configuration pour les [déclencheurs et liaisons de file d’attente de stockage](functions-bindings-storage-queue.md).

```json
{
    "queues": {
      "maxPollingInterval": 2000,
      "visibilityTimeout" : "00:00:30",
      "batchSize": 16,
      "maxDequeueCount": 5,
      "newBatchThreshold": 8
    }
}
```

|Propriété  |Default | Description |
|---------|---------|---------| 
|maxPollingInterval|60000|Intervalle maximal (en millisecondes) entre les interrogations de la file d’attente.| 
|visibilityTimeout|0|Intervalle de temps entre les nouvelles tentatives en cas d’échec du traitement d’un message.| 
|batchSize|16|Le nombre de messages de file d’attente que le runtime Functions récupère simultanément et traite en parallèle. Quand le nombre de messages en cours de traitement descend à `newBatchThreshold`, le runtime obtient un autre lot et commence à traiter ces messages. Par conséquent, le nombre maximal de messages traités simultanément par fonction est `batchSize` plus `newBatchThreshold`. Cette limite s’applique séparément à chaque fonction déclenchée par une file d’attente. <br><br>Si vous souhaitez éviter les exécutions parallèles pour les messages reçus sur une file d’attente, vous pouvez définir `batchSize` sur 1. Toutefois, ce paramètre évite les opérations simultanées uniquement pendant l’exécution de votre application de fonction sur une machine virtuelle unique. Si l’application de fonction augmente la taille des instances sur plusieurs machines virtuelles, chaque machine virtuelle peut exécuter une instance de chaque fonction déclenchée par une file d’attente.<br><br>La valeur `batchSize` maximale est de 32. | 
|maxDequeueCount|5|Nombre de tentatives de traitement d’un message avant de le placer dans la file d’attente de messages incohérents.| 
|newBatchThreshold|batchSize/2|Quand le nombre de messages traités simultanément passe en dessous de cette valeur, le runtime récupère un autre lot.| 

## <a name="sendgrid"></a>SendGrid

Paramètres de configuration pour la [liaison de sortie SendGrind](functions-bindings-sendgrid.md)

```json
{
    "sendGrid": {
        "from": "Contoso Group <admin@contoso.com>"
    }
}    
```

|Propriété  |Default | Description |
|---------|---------|---------| 
|de|n/a|Adresse e-mail de l’expéditeur dans toutes les fonctions.| 

## <a name="servicebus"></a>serviceBus

Paramètre de configuration pour les [déclencheurs et liaisons Service Bus](functions-bindings-service-bus.md).

```json
{
    "serviceBus": {
      "maxConcurrentCalls": 16,
      "prefetchCount": 100,
      "autoRenewTimeout": "00:05:00",
      "autoComplete": true
    }
}
```

|Propriété  |Default | Description |
|---------|---------|---------| 
|maxConcurrentCalls|16|Nombre maximal d’appels simultanés pour le rappel que la pompe de messages doit initier. Par défaut, le runtime Functions traite plusieurs messages simultanément. Pour que le runtime ne traite qu’un message de file d’attente ou de rubrique à la fois, définissez `maxConcurrentCalls` sur 1. | 
|prefetchCount|n/a|Valeur PrefetchCount par défaut qui est utilisée par l’instance MessageReceiver sous-jacente.| 
|autoRenewTimeout|00:05:00|Durée maximale pendant laquelle le verrouillage de message doit être renouvelé automatiquement.|
|autoComplete|true|Lorsque la valeur est true, le déclencheur termine automatiquement le traitement du message en cas d’exécution réussie de l’opération. Lorsque la valeur est false, la fonction est chargée de terminer le message avant de le retourner.|

## <a name="singleton"></a>singleton

Paramètres de configuration du comportement de verrouillage Singleton. Pour plus d’informations, consultez l’article sur le [problème de GitHub relatif à la prise en charge de singleton](https://github.com/Azure/azure-webjobs-sdk-script/issues/912).

```json
{
    "singleton": {
      "lockPeriod": "00:00:15",
      "listenerLockPeriod": "00:01:00",
      "listenerLockRecoveryPollingInterval": "00:01:00",
      "lockAcquisitionTimeout": "00:01:00",
      "lockAcquisitionPollingInterval": "00:00:03"
    }
}
```

|Propriété  |Default | Description |
|---------|---------|---------| 
|lockPeriod|00:00:15|Période pendant laquelle des verrous sont établis au niveau des fonctions. Les verrous se renouvellent automatiquement.| 
|listenerLockPeriod|00:01:00|Période pendant laquelle des verrous sont établis au niveau des écouteurs.| 
|listenerLockRecoveryPollingInterval|00:01:00|Intervalle de temps utilisé pour la récupération des verrous d’écouteurs si un verrou de ce type n’a pas pu être acquis au démarrage.| 
|lockAcquisitionTimeout|00:01:00|Période de temps maximale pendant laquelle le runtime essaie d’acquérir un verrou.| 
|lockAcquisitionPollingInterval|n/a|Intervalle écoulé entre les tentatives d’acquisition de verrou.| 

## <a name="tracing"></a>tracing

*Version 1.x*

Paramètres de configuration des journaux d’activité que vous créez à l’aide d’un objet `TraceWriter`. Pour en savoir plus, consultez [Journalisation C#].

```json
{
    "tracing": {
      "consoleLevel": "verbose",
      "fileLoggingMode": "debugOnly"
    }
}
```

|Propriété  |Default | Description |
|---------|---------|---------| 
|consoleLevel|info|Niveau de suivi pour la journalisation de la console. Options : `off`, `error`, `warning`, `info` et `verbose`.|
|fileLoggingMode|debugOnly|Niveau de suivi pour la journalisation des fichiers. Options : `never`, `always`, `debugOnly`.| 

## <a name="watchdirectories"></a>watchDirectories

Ensemble de [répertoires de code partagé](functions-reference-csharp.md#watched-directories) dont les modifications doivent être surveillées.  Garantit que lorsque le code contenu dans ces répertoires est modifié, les changements sont récupérés par vos fonctions.

```json
{
    "watchDirectories": [ "Shared" ]
}
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrez comment mettre à jour le fichier host.json](functions-reference.md#fileupdate)

> [!div class="nextstepaction"]
> [Consultez les paramètres globaux des variables d’environnement](functions-app-settings.md)
