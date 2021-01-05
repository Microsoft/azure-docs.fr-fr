---
title: Informations de référence sur le fichier host.json pour Azure Functions 2.x
description: Documentation de référence pour le fichier host.json d’Azure Functions avec le runtime v2.
ms.topic: conceptual
ms.date: 04/28/2020
ms.openlocfilehash: 96d6b884e9e2c835316af01140c6fc7208ee5ab9
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96746078"
---
# <a name="hostjson-reference-for-azure-functions-2x-and-later"></a>Informations de référence sur le fichier host.json pour Azure Functions 2.x et ultérieur 

> [!div class="op_single_selector" title1="Sélectionnez la version du runtime Azure Functions que vous utilisez : "]
> * [Version 1](functions-host-json-v1.md)
> * [Version 2+](functions-host-json.md)

Le fichier de métadonnées *host.json* contient les options de configuration globale qui affectent l’ensemble des fonctions d’une application de fonction. Cet article liste les paramètres disponibles à compter de la version 2.x du runtime Azure Functions.  

> [!NOTE]
> Cet article concerne Azure Functions versions 2.x et ultérieures.  Pour obtenir une référence de host.json dans Functions 1.x, consultez [Informations de référence sur le fichier host.json pour Azure Functions 1.x](functions-host-json-v1.md).

D’autres options de configuration d’application de fonction sont gérées dans vos [paramètres d’application](functions-app-settings.md) (pour les applications déployées) ou dans votre fichier [local.settings.json](functions-run-local.md#local-settings-file) (pour un développement local).

Les configurations dans host.json relatives aux liaisons sont appliquées de façon égale à chaque fonction de l’application de fonction. 

Vous pouvez également [substituer ou appliquer des paramètres en fonction de l’environnement](#override-hostjson-values) à l’aide des paramètres d’application.

## <a name="sample-hostjson-file"></a>Exemple de fichier host.json

L’exemple de fichier *host.json* suivant pour la version 2.x+ contient toutes les options possibles spécifiées (sauf celles qui sont destinées à un usage interne uniquement).

```json
{
    "version": "2.0",
    "aggregator": {
        "batchSize": 1000,
        "flushTimeout": "00:00:30"
    },
    "extensions": {
        "cosmosDb": {},
        "durableTask": {},
        "eventHubs": {},
        "http": {},
        "queues": {},
        "sendGrid": {},
        "serviceBus": {}
    },
    "extensionBundle": {
        "id": "Microsoft.Azure.Functions.ExtensionBundle",
        "version": "[1.*, 2.0.0)"
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
    "logging": {
        "fileLoggingMode": "debugOnly",
        "logLevel": {
          "Function.MyFunction": "Information",
          "default": "None"
        },
        "applicationInsights": {
            "samplingSettings": {
              "isEnabled": true,
              "maxTelemetryItemsPerSecond" : 20,
              "evaluationInterval": "01:00:00",
              "initialSamplingPercentage": 100.0, 
              "samplingPercentageIncreaseTimeout" : "00:00:01",
              "samplingPercentageDecreaseTimeout" : "00:00:01",
              "minSamplingPercentage": 0.1,
              "maxSamplingPercentage": 100.0,
              "movingAverageRatio": 1.0,
              "excludedTypes" : "Dependency;Event",
              "includedTypes" : "PageView;Trace"
            },
            "enableLiveMetrics": true,
            "enableDependencyTracking": true,
            "enablePerformanceCountersCollection": true,            
            "httpAutoCollectionOptions": {
                "enableHttpTriggerExtendedInfoCollection": true,
                "enableW3CDistributedTracing": true,
                "enableResponseHeaderInjection": true
            },
            "snapshotConfiguration": {
                "agentEndpoint": null,
                "captureSnapshotMemoryWeight": 0.5,
                "failedRequestLimit": 3,
                "handleUntrackedExceptions": true,
                "isEnabled": true,
                "isEnabledInDeveloperMode": false,
                "isEnabledWhenProfiling": true,
                "isExceptionSnappointsEnabled": false,
                "isLowPrioritySnapshotUploader": true,
                "maximumCollectionPlanSize": 50,
                "maximumSnapshotsRequired": 3,
                "problemCounterResetInterval": "24:00:00",
                "provideAnonymousTelemetry": true,
                "reconnectInterval": "00:15:00",
                "shadowCopyFolder": null,
                "shareUploaderProcess": true,
                "snapshotInLowPriorityThread": true,
                "snapshotsPerDayLimit": 30,
                "snapshotsPerTenMinutesLimit": 1,
                "tempFolder": null,
                "thresholdForSnapshotting": 1,
                "uploaderProxy": null
            }
        }
    },
    "managedDependency": {
        "enabled": true
    },
    "retry": {
      "strategy": "fixedDelay",
      "maxRetryCount": 5,
      "delayInterval": "00:00:05"
    },
    "singleton": {
      "lockPeriod": "00:00:15",
      "listenerLockPeriod": "00:01:00",
      "listenerLockRecoveryPollingInterval": "00:01:00",
      "lockAcquisitionTimeout": "00:01:00",
      "lockAcquisitionPollingInterval": "00:00:03"
    },
    "watchDirectories": [ "Shared", "Test" ]
}
```

Les sections suivantes de cet article expliquent chaque propriété de niveau supérieur. Toutes les autres propriétés sont facultatives sauf indication contraire.

## <a name="aggregator"></a>aggregator

[!INCLUDE [aggregator](../../includes/functions-host-json-aggregator.md)]

## <a name="applicationinsights"></a>applicationInsights

Ce paramètre est un enfant de la [journalisation](#logging).

Contrôle les options disponibles dans Application Insights, notamment les [options d’échantillonnage](./configure-monitoring.md#configure-sampling).

Pour obtenir la structure JSON complète, reportez-vous à l’[exemple de fichier host.json](#sample-hostjson-file) précédent.

> [!NOTE]
> L’échantillonnage de journal peut faire que certaines exécutions n’apparaissent pas dans le panneau de surveillance Application Insights. Pour éviter l’échantillonnage des journaux, ajoutez `excludedTypes: "Request"` à la valeur `samplingSettings`.

| Propriété | Default | Description |
| --------- | --------- | --------- | 
| samplingSettings | n/a | Consultez [applicationInsights.samplingSettings](#applicationinsightssamplingsettings). |
| enableLiveMetrics | true | Active la collecte des métriques temps réel. |
| enableDependencyTracking | true | Active le suivi des dépendances. |
| enablePerformanceCountersCollection | true | Active la collecte des compteurs de performances Kudu. |
| liveMetricsInitializationDelay | 00:00:15 | À usage interne uniquement. |
| httpAutoCollectionOptions | n/a | Consultez [applicationInsights.httpAutoCollectionOptions](#applicationinsightshttpautocollectionoptions). |
| snapshotConfiguration | n/a | Consultez [applicationInsights.snapshotConfiguration](#applicationinsightssnapshotconfiguration). |

### <a name="applicationinsightssamplingsettings"></a>applicationInsights.samplingSettings

Pour plus d'informations sur ces paramètres, consultez [Échantillonnage dans Application Insights](../azure-monitor/app/sampling.md). 

|Propriété | Default | Description |
| --------- | --------- | --------- | 
| isEnabled | true | Active ou désactive l’échantillonnage. | 
| maxTelemetryItemsPerSecond | 20 | Nombre cible d’éléments de télémétrie enregistrés par seconde sur chaque hôte de serveur. Si votre application s’exécute sur de nombreux ordinateurs hôtes, réduisez cette valeur pour rester dans votre taux de trafic cible global. | 
| evaluationInterval | 01:00:00 | Intervalle auquel le taux actuel de télémétrie est réévalué. L’évaluation est effectuée sous forme de moyenne mobile. Vous souhaiterez peut-être raccourcir cet intervalle si vos données de télémétrie sont soumises à des pics soudains. |
| initialSamplingPercentage| 100.0 | Pourcentage d’échantillonnage initial appliqué au début du processus d’échantillonnage pour faire varier dynamiquement le pourcentage. Ne diminuez pas la valeur pendant le débogage. |
| samplingPercentageIncreaseTimeout | 00:00:01 | Lorsque la valeur de pourcentage d’échantillonnage change, cette propriété détermine le moment où Application Insights est autorisé à augmenter à nouveau le pourcentage d’échantillonnage pour capturer plus de données. |
| samplingPercentageDecreaseTimeout | 00:00:01 | Lorsque la valeur de pourcentage d’échantillonnage change, cette propriété détermine le moment où Application Insights est autorisé à diminuer à nouveau le pourcentage d’échantillonnage pour capturer moins de données. |
| minSamplingPercentage | 0.1 | Comme le pourcentage d’échantillonnage varie, cette propriété détermine le pourcentage d’échantillonnage autorisé minimal. |
| minSamplingPercentage | 100.0 | Comme le pourcentage d’échantillonnage varie, cette propriété détermine le pourcentage d’échantillonnage autorisé maximal. |
| movingAverageRatio | 1.0 | Lors du calcul de la moyenne mobile, poids affecté à la valeur la plus récente. Utilisez une valeur inférieure ou égale à 1. Plus les valeurs sont petites, moins l’algorithme est réactif en cas de modifications brusques. |
| excludedTypes | null | Une liste délimitée par des points-virgules des types que vous ne souhaitez pas voir échantillonnés. Les types reconnus sont les suivants : `Dependency`, `Event`, `Exception`, `PageView`, `Request` et `Trace`. Toutes les instances des types spécifiés sont transmises ; les types qui ne sont pas spécifiés sont échantillonnés. |
| includedTypes | null | Liste délimitée par des points-virgules des types que vous souhaitez échantillonner ; une liste vide signifie que tous les types sont impliqués. Le type répertorié dans `excludedTypes` remplace les types répertoriés ici. Les types reconnus sont les suivants : `Dependency`, `Event`, `Exception`, `PageView`, `Request` et `Trace`. Les instances des types spécifiés sont échantillonnées ; les types non spécifiés ou implicites sont transmis sans échantillonnage. |

### <a name="applicationinsightshttpautocollectionoptions"></a>applicationInsights.httpAutoCollectionOptions

|Propriété | Default | Description |
| --------- | --------- | --------- | 
| enableHttpTriggerExtendedInfoCollection | true | Active ou désactive les informations de requête HTTP étendues pour les déclencheurs HTTP : en-têtes de corrélation de requête entrante, prise en charge des clés d’instrumentation multiples, méthode HTTP, chemin d’accès et réponse. |
| enableW3CDistributedTracing | true | Active ou désactive la prise en charge du protocole W3C Distributed Tracing Protocol (et active le schéma de corrélation hérité). Activé par défaut si `enableHttpTriggerExtendedInfoCollection` a la valeur true. Si `enableHttpTriggerExtendedInfoCollection` a la valeur false, cet indicateur s’applique uniquement aux demandes sortantes, et non aux demandes entrantes. |
| enableResponseHeaderInjection | true | Active ou désactive l’injection d’en-têtes de corrélation à composants multiples dans les réponses. L’activation de l’injection permet à Application Insights de construire un mappage d’application en cas d’utilisation de plusieurs clés d’instrumentation. Activé par défaut si `enableHttpTriggerExtendedInfoCollection` a la valeur true. Ce paramètre ne s’applique pas si `enableHttpTriggerExtendedInfoCollection` a la valeur false. |

### <a name="applicationinsightssnapshotconfiguration"></a>applicationInsights.snapshotConfiguration

Pour plus d’informations sur les instantanés, consultez [Captures instantanées de débogage sur exceptions levées dans des applications .NET](../azure-monitor/app/snapshot-debugger.md) et [Résoudre les problèmes d’activation du Débogueur de capture instantanée Application Insights ou d’affichage d’instantanés](../azure-monitor/app/snapshot-debugger-troubleshoot.md).

|Propriété | Default | Description |
| --------- | --------- | --------- | 
| agentEndpoint | null | Point de terminaison utilisé pour se connecter au service de Débogueur de capture instantanée Application Insights. Si la valeur est null, un point de terminaison par défaut est utilisé. |
| captureSnapshotMemoryWeight | 0.5 | Poids donné à la taille actuelle de la mémoire du processus lors de la vérification s’il y a assez de mémoire pour prendre un instantané. La valeur attendue est une fraction supérieure à 0 (0 < CaptureSnapshotMemoryWeight < 1). |
| failedRequestLimit | 3 | Limite du nombre de demandes ayant échoué pour demander des captures instantanées avant que le processeur de télémétrie soit désactivé.|
| handleUntrackedExceptions | true | Active ou désactive le suivi des exceptions qui ne sont pas suivies par la télémétrie Application Insights. |
| isEnabled | true | Active ou désactive la collecte de captures instantanées | 
| isEnabledInDeveloperMode | false | Active ou désactive la collecte de captures instantanées activée en mode développeur. |
| isEnabledWhenProfiling | true | Active ou désactive la création de captures instantanées même si Application Insights Profiler collecte une session de profilage détaillée. |
| isExceptionSnappointsEnabled | false | Active ou désactive le filtrage des exceptions. |
| isLowPrioritySnapshotUploader | true | Détermine s’il faut exécuter le processus SnapshotUploader en dessous de la priorité normale. |
| maximumCollectionPlanSize | 50 | Nombre maximal de problèmes que nous pouvons suivre à tout moment dans une plage comprise entre 1 et 9999. |
| maximumSnapshotsRequired | 3 | Nombre maximal de captures instantanées collectées pour un seul problème dans une plage comprise entre 1 et 999. Un problème peut être assimilé à une instruction throw individuelle dans votre application. Une fois que le nombre de captures instantanées collectées pour un problème atteint cette valeur, aucune capture instantanée n’est collectée pour ce problème jusqu’à ce que les compteurs de problèmes soient réinitialisés (voir `problemCounterResetInterval`) et que la limite de `thresholdForSnapshotting` soit de nouveau atteinte. |
| problemCounterResetInterval | 24:00:00 | Fréquence de réinitialisation des compteurs de problèmes dans une plage comprise entre une minute et sept jours. Lorsque cet intervalle est atteint, tous les compteurs de problèmes sont remis à zéro. Les problèmes existants qui ont déjà atteint le seuil pour les captures instantanées, mais qui n’ont pas encore généré le nombre de captures instantanées dans `maximumSnapshotsRequired`, subsistent. |
| provideAnonymousTelemetry | true | Détermine s’il faut envoyer une télémétrie d’utilisation et d’erreur anonymes à Microsoft. Cette télémétrie peut être utilisée si vous contactez Microsoft pour vous aider à résoudre les problèmes liés au Débogueur de capture instantanée. Elle permet également d’analyser les modèles d’utilisation. |
| reconnectInterval | 00:15:00 | Fréquence de reconnexion au point de terminaison Débogueur de capture instantanée. La plage autorisée est comprise entre une minute et un jour. |
| shadowCopyFolder | null | Spécifie le dossier à utiliser pour les fichiers binaires de clichés instantanés. Si la valeur n’est pas définie, les dossiers spécifiés par les variables d’environnement suivantes sont essayées dans l’ordre : Fabric_Folder_App_Temp, LOCALAPPDATA, APPDATA, TEMP. |
| shareUploaderProcess | true | Si la valeur est true, une seule instance de SnapshotUploader collecte et charge des captures instantanées pour plusieurs applications qui partagent la clé InstrumentationKey. Si la valeur est false, SnapshotUploader est unique pour chaque tuple (ProcessName, InstrumentationKey). |
| snapshotInLowPriorityThread | true | Détermine si les instantanés doivent être traités dans un thread à faible priorité d’E/S. La création d’une capture instantanée est une opération rapide, mais, pour charger une capture instantanée vers le service Débogueur de capture instantanée, celle doit d’abord être écrite sur le disque en tant que minidump. Cela se produit dans le processus SnapshotUploader. La définition de cette valeur sur true utilise des E/S de priorité basse pour écrire le minidump, qui n’est pas en concurrence avec votre application pour les ressources. Le réglage de cette valeur sur false accélère la création de minidumps au prix d'un ralentissement de votre application. |
| snapshotsPerDayLimit | 30 | Nombre maximal de captures instantanées autorisées dans une journée (24 heures). Cette limite est également appliquée du côté du service Application Insights. Les chargements sont limités à 50 par jour par application (autrement dit, par clé d’Instrumentation). Cette valeur permet d’empêcher la création de captures instantanées supplémentaires qui seront finalement rejetées pendant le chargement. La valeur zéro supprime entièrement la limite, ce qui n’est pas recommandé. |
| snapshotsPerTenMinutesLimit | 1 | Nombre maximal de captures instantanées autorisées en 10 minutes. Bien qu'il n'y ait pas de limite supérieure à cette valeur, faites preuve de prudence en l’augmentant sur les charges de production car elle pourrait avoir un impact sur les performances de votre application. La création d’une capture instantanée est rapide, mais la création d’un minidump de la capture instantanée et son chargement vers le service Débogueur de capture instantanée est une opération beaucoup plus lente qui entrera en concurrence avec votre application pour l’utilisation des ressources (à la fois UC et E-S). |
| tempFolder | null | Spécifie le dossier dans lequel écrire les fichiers journaux des minidumps et du programme de chargement. S’il n’est pas défini, *%TEMP%\Dumps* est utilisé. |
| thresholdForSnapshotting | 1 | Nombre de fois où Application Insights doit voir une exception avant de demander des captures instantanées. |
| uploaderProxy | null | Remplace le serveur proxy utilisé dans le processus du chargeur de captures instantanées. Vous devrez peut-être utiliser ce paramètre si votre application se connecte à Internet via un serveur proxy. Le Snapshot Collector s’exécute dans le processus de votre application et utilise les mêmes paramètres de proxy. Toutefois, le chargeur de captures instantanées s’exécute en tant que processus distinct et vous devrez peut-être configurer le serveur proxy manuellement. Si cette valeur est null, Snapshot Collector tentera de détecter automatiquement l’adresse du proxy en examinant System.Net.WebRequest.DefaultWebProxy et en transmettant la valeur au téléchargeur de captures instantanées. Dans le cas contraire, la détection automatique n’est pas utilisée et le serveur proxy spécifié ici sera utilisé dans le téléchargeur de captures instantanées. |

## <a name="cosmosdb"></a>cosmosDb

Le paramètre de configuration se trouve dans les [déclencheurs et liaisons Cosmos DB](functions-bindings-cosmosdb-v2-output.md#host-json).

## <a name="customhandler"></a>customHandler

Paramètres de configuration d’un gestionnaire personnalisé. Pour plus d’informations, consultez [Gestionnaires personnalisés Azure Functions](functions-custom-handlers.md#configuration).

```json
"customHandler": {
  "description": {
    "defaultExecutablePath": "server",
    "workingDirectory": "handler",
    "arguments": [ "--port", "%FUNCTIONS_CUSTOMHANDLER_PORT%" ]
  },
  "enableForwardingHttpRequest": false
}
```

|Propriété | Default | Description |
| --------- | --------- | --------- |
| defaultExecutablePath | n/a | Exécutable à démarrer en tant que processus de gestionnaire personnalisé. Il s’agit d’un paramètre obligatoire lorsque vous utilisez des gestionnaires personnalisés et dont la valeur est relative à la racine de l’application de fonction. |
| workingDirectory | *racine de l’application de fonction* | Répertoire de travail dans lequel démarrer le processus de gestionnaire personnalisé. Il s’agit d’un paramètre facultatif dont la valeur est relative à la racine de l’application de fonction. |
| arguments | n/a | Tableau d’arguments de ligne de commande à transmettre au processus de gestionnaire personnalisé. |
| enableForwardingHttpRequest | false | Si cette valeur est définie, toutes les fonctions qui se composent uniquement d’un déclencheur HTTP et d’une sortie HTTP sont transférées à la requête HTTP d’origine au lieu de la [charge utile de demande](functions-custom-handlers.md#request-payload) du gestionnaire personnalisé. |

## <a name="durabletask"></a>durableTask

Le paramètre de configuration se trouve dans les [liaisons de Durable Functions](durable/durable-functions-bindings.md#host-json).

## <a name="eventhub"></a>eventHub

Les paramètres de configuration se trouvent dans les [déclencheurs et liaisons Event Hub](functions-bindings-event-hubs-trigger.md#host-json). 

## <a name="extensions"></a>extensions

Propriété qui retourne un objet qui contient tous les paramètres spécifiques d’une liaison, tel que [http](#http) et [eventHub](#eventhub).

## <a name="extensionbundle"></a>extensionBundle 

Les offres groupées d’extensions vous permettent d’ajouter un jeu d’extensions de liaison Functions compatible avec votre application de fonction. Pour plus d’informations, voir [Offres groupées d’extension pour développement local](functions-bindings-register.md#extension-bundles).

[!INCLUDE [functions-extension-bundles-json](../../includes/functions-extension-bundles-json.md)]

## <a name="functions"></a>functions

Liste des fonctions que l’hôte de travail exécute. Un tableau vide désigne l’exécution de toutes les fonctions. Utilisée uniquement pour une [exécution locale](functions-run-local.md). Dans les applications de fonction dans Azure, vous devez plutôt suivre la procédure décrite dans [Guide pratique pour désactiver des fonctions dans Azure Functions](disable-function.md) pour désactiver des fonctions spécifiques au lieu d’utiliser ce paramètre.

```json
{
    "functions": [ "QueueProcessor", "GitHubWebHook" ]
}
```

## <a name="functiontimeout"></a>functionTimeout

Indique la durée avant expiration du délai de toutes les fonctions. Il suit le format de chaîne TimeSpan. 

| Type de plan | Par défaut (min) | Maximum (min) |
| -- | -- | -- |
| Consommation | 5 | 10 |
| Premium<sup>1</sup> | 30 | -1 (non lié)<sup>2</sup> |
| Dédié (App Service) | 30 | -1 (non lié)<sup>2</sup> |

<sup>1</sup> L’exécution du plan Premium n’est garantie que pendant 60 minutes, mais techniquement illimitée.   
<sup>2</sup> La valeur `-1` indique une exécution illimitée, mais il est recommandé de conserver une limite supérieure fixe.

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

Les paramètres de configuration se trouvent dans les [déclencheurs et liaisons HTTP](functions-bindings-http-webhook-output.md#hostjson-settings).

## <a name="logging"></a>journalisation

Contrôle les comportements de journalisation de l’application de fonction, y compris Application Insights.

```json
"logging": {
    "fileLoggingMode": "debugOnly",
    "logLevel": {
      "Function.MyFunction": "Information",
      "default": "None"
    },
    "console": {
        ...
    },
    "applicationInsights": {
        ...
    }
}
```

|Propriété  |Default | Description |
|---------|---------|---------|
|fileLoggingMode|debugOnly|Définit le niveau de journalisation de fichiers activé.  Options : `never`, `always`, `debugOnly`. |
|logLevel|n/a|Objet qui définit le filtrage par catégorie du journal pour les fonctions de l’application. Les versions 2.x et ultérieures suivent la disposition d’ASP.NET Core pour le filtrage des catégories du journal. Ce paramètre vous permet de filtrer la journalisation pour des fonctions spécifiques. Pour plus d’informations, consultez [Filtrage de journal](/aspnet/core/fundamentals/logging/?view=aspnetcore-2.1&preserve-view=true#log-filtering) dans la documentation ASP.NET Core. |
|console|n/a| Le paramètre de journalisation de la [console](#console). |
|applicationInsights|n/a| Le paramètre [applicationInsights](#applicationinsights). |

## <a name="console"></a>console

Ce paramètre est un enfant de la [journalisation](#logging). Il contrôle la journalisation de la console lorsque le mode débogage n’est pas activé.

```json
{
    "logging": {
    ...
        "console": {
          "isEnabled": "false"
        },
    ...
    }
}
```

|Propriété  |Default | Description |
|---------|---------|---------| 
|isEnabled|false|Active ou désactive la journalisation de la console.| 

## <a name="manageddependency"></a>managedDependency

La dépendance managée est une fonctionnalité qui est actuellement prise en charge uniquement par les fonctions basées sur PowerShell. Elle permet au service de gérer automatiquement les dépendances. Lorsque la propriété `enabled` est définie sur `true`, le fichier `requirements.psd1` est traité. Les dépendances sont mises à jour lorsque des versions mineures sont publiées. Pour plus d’informations, consultez la section [Dépendance managée](functions-reference-powershell.md#dependency-management) de l’article PowerShell.

```json
{
    "managedDependency": {
        "enabled": true
    }
}
```

## <a name="queues"></a>queues

Les paramètres de configuration se trouvent dans les [déclencheurs et liaisons de la file d'attente de stockage](functions-bindings-storage-queue-output.md#host-json).  

## <a name="retry"></a>retry

Contrôle les options de [stratégie de nouvelles tentatives](./functions-bindings-error-pages.md#retry-policies-preview) pour toutes les exécutions effectuées dans l’application.

```json
{
    "retry": {
        "strategy": "fixedDelay",
        "maxRetryCount": 2,
        "delayInterval": "00:00:03"  
    }
}
```

|Propriété  |Default | Description |
|---------|---------|---------| 
|strategy|null|Obligatoire. Stratégie de nouvelle tentative à utiliser. Les valeurs valides sont `fixedDelay` ou `exponentialBackoff`.|
|maxRetryCount|null|Obligatoire. Nombre maximal de nouvelles tentatives autorisées par exécution de fonction. `-1` signifie qu’il faut effectuer ces nouvelles tentatives indéfiniment.|
|delayInterval|null|Délai qui est utilisé entre les nouvelles tentatives avec une stratégie `fixedDelay`.|
|minimumInterval|null|Délai minimal de nouvelle tentative lors de l’utilisation de la stratégie `exponentialBackoff`.|
|maximumInterval|null|Délai maximal de nouvelle tentative lors de l’utilisation de la stratégie `exponentialBackoff`.| 

## <a name="sendgrid"></a>sendGrid

Le paramètre de configuration se trouve dans les [déclencheurs et liaisons SendGrid](functions-bindings-sendgrid.md#host-json).

## <a name="servicebus"></a>serviceBus

Le paramètre de configuration se trouve dans les [déclencheurs et liaisons Service Bus](functions-bindings-service-bus-output.md#host-json).

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

## <a name="version"></a>version

Cette valeur indique la version de schéma de host.json. La chaîne de version `"version": "2.0"` est nécessaire pour une application de fonction qui cible le runtime v2 ou une version ultérieure. Il n’y a aucune modification du schéma host.json entre v2 et v3.

## <a name="watchdirectories"></a>watchDirectories

Ensemble de [répertoires de code partagé](functions-reference-csharp.md#watched-directories) dont les modifications doivent être surveillées.  Garantit que lorsque le code contenu dans ces répertoires est modifié, les changements sont récupérés par vos fonctions.

```json
{
    "watchDirectories": [ "Shared" ]
}
```

## <a name="override-hostjson-values"></a>Substituer les valeurs host.json

Dans certains cas, vous pouvez être amené à configurer ou modifier des paramètres spécifiques dans un fichier host.json pour un environnement spécifique, sans changer le fichier host.json lui-même.  Vous pouvez substituer des valeurs host.json spécifiques en créant une valeur équivalente en tant que paramètre d’application. Quand le runtime trouve un paramètre d’application au format `AzureFunctionsJobHost__path__to__setting`, il remplace le paramètre host.json équivalent situé sur `path.to.setting` dans le fichier JSON. Quand il est exprimé sous la forme d’un paramètre d’application, le point (`.`) utilisé pour indiquer la hiérarchie JSON est remplacé par un trait de soulignement double (`__`). 

Par exemple, vous pouvez être amené à désactiver l’échantillonnage Application Insight en cas d’exécution locale. Si vous avez changé le fichier local host.json pour désactiver Application Insights, ce changement peut être envoyé (push) à votre application de production durant le déploiement. La méthode la plus fiable consiste à créer un paramètre d’application sous la forme `"AzureFunctionsJobHost__logging__applicationInsights__samplingSettings__isEnabled":"false"` dans le fichier `local.settings.json`. Vous pouvez le voir dans le fichier `local.settings.json` suivant, qui n’est pas publié :

```json
{
    "IsEncrypted": false,
    "Values": {
        "AzureWebJobsStorage": "{storage-account-connection-string}",
        "FUNCTIONS_WORKER_RUNTIME": "{language-runtime}",
        "AzureFunctionsJobHost__logging__applicationInsights__samplingSettings__isEnabled":"false"
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Découvrez comment mettre à jour le fichier host.json](functions-reference.md#fileupdate)

> [!div class="nextstepaction"]
> [Consultez les paramètres globaux des variables d’environnement](functions-app-settings.md)
