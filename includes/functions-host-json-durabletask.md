---
title: Fichier Include
description: Fichier Include
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/14/2019
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: 6a862a051d0040ac99746d81f10ae63d5af7545f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013720"
---
Paramètres de configuration de [Fonctions durables](../articles/azure-functions/durable/durable-functions-overview.md).

> [!NOTE]
> Toutes les versions principales de Durable Functions sont prises en charge sur toutes les versions du runtime Azure Functions. Toutefois, le schéma de la configuration host.json est légèrement différent selon la version du runtime Azure Functions et la version d’extension Durable Functions que vous utilisez. Les exemples suivants sont destinés à être utilisés avec Azure Functions 2.0 et 3.0. Dans les deux exemples, si vous utilisez Azure Functions 1.0, les paramètres disponibles sont les mêmes, mais la section « durableTask » de host.json doit être dans la racine de la configuration host.json plutôt que sous la forme d’un champ sous « extensions ».

### <a name="durable-functions-2x"></a><a name="durable-functions-2-0-host-json"></a>Durable Functions 2.x

```json
{
 "extensions": {
  "durableTask": {
    "hubName": "MyTaskHub",
    "storageProvider": {
      "connectionStringName": "AzureWebJobsStorage",
      "controlQueueBatchSize": 32,
      "controlQueueBufferThreshold": 256,
      "controlQueueVisibilityTimeout": "00:05:00",
      "maxQueuePollingInterval": "00:00:30",
      "partitionCount": 4,
      "trackingStoreConnectionStringName": "TrackingStorage",
      "trackingStoreNamePrefix": "DurableTask",
      "useLegacyPartitionManagement": true,
      "workItemQueueVisibilityTimeout": "00:05:00",
    },
    "tracing": {
      "traceInputsAndOutputs": false,
      "traceReplayEvents": false,
    },
    "notifications": {
      "eventGrid": {
        "topicEndpoint": "https://topic_name.westus2-1.eventgrid.azure.net/api/events",
        "keySettingName": "EventGridKey",
        "publishRetryCount": 3,
        "publishRetryInterval": "00:00:30",
        "publishEventTypes": [
          "Started",
          "Pending",
          "Failed",
          "Terminated"
        ]
      }
    },
    "maxConcurrentActivityFunctions": 10,
    "maxConcurrentOrchestratorFunctions": 10,
    "extendedSessionsEnabled": false,
    "extendedSessionIdleTimeoutInSeconds": 30,
    "useAppLease": true,
    "useGracefulShutdown": false
  }
 }
}

```

### <a name="durable-functions-1x"></a>Durable Functions 1.x

```json
{
  "extensions": {
    "durableTask": {
      "hubName": "MyTaskHub",
      "controlQueueBatchSize": 32,
      "partitionCount": 4,
      "controlQueueVisibilityTimeout": "00:05:00",
      "workItemQueueVisibilityTimeout": "00:05:00",
      "maxConcurrentActivityFunctions": 10,
      "maxConcurrentOrchestratorFunctions": 10,
      "maxQueuePollingInterval": "00:00:30",
      "azureStorageConnectionStringName": "AzureWebJobsStorage",
      "trackingStoreConnectionStringName": "TrackingStorage",
      "trackingStoreNamePrefix": "DurableTask",
      "traceInputsAndOutputs": false,
      "logReplayEvents": false,
      "eventGridTopicEndpoint": "https://topic_name.westus2-1.eventgrid.azure.net/api/events",
      "eventGridKeySettingName":  "EventGridKey",
      "eventGridPublishRetryCount": 3,
      "eventGridPublishRetryInterval": "00:00:30",
      "eventGridPublishEventTypes": ["Started", "Completed", "Failed", "Terminated"]
    }
  }
}
```

Les noms de hubs de tâches doivent commencer par une lettre et contenir uniquement des lettres et des chiffres. S’il n’est pas spécifié, le nom du hub de tâches par défaut d’une application de fonction est **DurableFunctionsHub**. Pour en savoir plus, consultez la section relative aux [hubs de tâches](../articles/azure-functions/durable/durable-functions-task-hubs.md).

|Propriété  |Default | Description |
|---------|---------|---------|
|hubName|DurableFunctionsHub|D'autres noms de [hub de tâches](../articles/azure-functions/durable/durable-functions-task-hubs.md) peuvent être utilisés pour isoler plusieurs applications Durable Functions les unes des autres, même si elles s'appuient sur le même principal de stockage.|
|controlQueueBatchSize|32|Nombre de messages à extraire de la file d’attente de contrôle en une seule fois.|
|controlQueueBufferThreshold|256|Nombre de messages de file d’attente de contrôle qui peuvent être mis en mémoire tampon à la fois, auquel cas le répartiteur attend avant de retirer des messages supplémentaires de la file d’attente.|
|partitionCount |4|Nombre de partitions pour la file d’attente de contrôle. Doit être un entier positif compris entre 1 et 16.|
|controlQueueVisibilityTimeout |5 minutes|Délai d’expiration de la visibilité des messages supprimés de la file d’attente de contrôle.|
|workItemQueueVisibilityTimeout |5 minutes|Délai d’expiration de la visibilité des messages supprimés de la file d’attente des éléments de travail.|
|maxConcurrentActivityFunctions |10 fois le nombre de processeurs sur l’ordinateur actuel|Nombre maximal de fonctions d’activité pouvant être traitées simultanément sur une seule instance d’hôte.|
|maxConcurrentOrchestratorFunctions |10 fois le nombre de processeurs sur l’ordinateur actuel|Nombre maximal de fonctions d’orchestrateur pouvant être traitées simultanément sur une seule instance d’hôte.|
|maxQueuePollingInterval|30 secondes|Intervalle d’interrogation de la file d’attente des éléments de travail et de contrôle maximum, indiqué au format *hh:mm:ss*. Des valeurs plus élevées peuvent entraîner une plus grande latence lors du traitement des messages. Des valeurs plus faibles peuvent entraîner des coûts de stockage plus importants, suite à un nombre plus important de transactions de stockage.|
|azureStorageConnectionStringName |AzureWebJobsStorage|Nom du paramètre d’application qui contient la chaîne de connexion de stockage Azure utilisée pour gérer les ressources de stockage Azure sous-jacentes.|
|trackingStoreConnectionStringName||Nom d’une chaîne de connexion à utiliser pour les tables d’historique et d’instances. S’il n’est pas spécifié, la connexion `connectionStringName` (Durable 2.x) ou `azureStorageConnectionStringName` (Durable 1.x) est utilisée.|
|trackingStoreNamePrefix||Préfixe à utiliser pour les tables d’historique et d’instances lorsque le paramètre `trackingStoreConnectionStringName` est spécifié. S’il n’est pas défini, la valeur du préfixe par défaut correspond à `DurableTask`. Si le paramètre `trackingStoreConnectionStringName` n’est pas spécifié, les tables d’historique et d’instances utilisent la valeur `hubName` en tant que préfixe, et tous les paramètres relatifs à `trackingStoreNamePrefix` sont ignorés.|
|traceInputsAndOutputs |false|Valeur indiquant s’il faut tracer les entrées et sorties des appels de fonction. Lors du traçage d’événements d’exécution de fonctions, le nombre d’octets est inclus par défaut dans les entrées et les sorties sérialisées pour les appels de fonction. Ce comportement fournit un minimum d’informations sur l’apparence des entrées et sorties, sans encombrer les journaux d’activité ni exposer accidentellement des informations sensibles. La définition de cette propriété sur True amène la journalisation de la fonction par défaut à consigner la totalité du contenu des entrées et sorties de fonction.|
|logReplayEvents|false|Valeur indiquant s’il faut écrire les événements de relecture d’orchestration à Application Insights.|
|eventGridTopicEndpoint ||URL du point de terminaison de la rubrique personnalisée Azure Event Grid. Lorsque cette propriété est définie, les événements de notification du cycle de vie de l’orchestration sont publiés sur ce point de terminaison. Cette propriété prend en charge la résolution de paramètres de l’application.|
|eventGridKeySettingName ||Nom du paramètre d’application contenant la clé utilisée pour l’authentification à l’aide de la rubrique personnalisée Azure Event Grid sur `EventGridTopicEndpoint`.|
|eventGridPublishRetryCount|0|Nombre de nouvelles tentatives à effectuer en cas d’échec de la publication de la rubrique Event Grid.|
|eventGridPublishRetryInterval|5 minutes|L’intervalle avant nouvelle tentative de publication Event Grid au format *hh:mm:ss*.|
|eventGridPublishEventTypes||Liste des types d’événement à publier sur Event Grid. Si elle n’est pas spécifiée, cela signifie que tous les types d’événement seront publiés. Les valeurs autorisées sont `Started`, `Completed`, `Failed` et `Terminated`.|
|useAppLease|true|Quand la valeur est `true`, les applications nécessitent l’acquisition d’un bail d’objet blob au niveau de l’application pour traiter les messages du hub de tâches. Pour plus d’informations, consultez la documentation [Récupération d’urgence et géodistribution](../articles/azure-functions/durable/durable-functions-disaster-recovery-geo-distribution.md). Disponible à partir de la version 2.3.0.
|useLegacyPartitionManagement|true|Quand sa valeur est `false`, utilise un algorithme de gestion des partitions qui réduit le risque d’exécuter une fonction en double lors du scale-out.  Disponible à partir de la version 2.3.0. La valeur par défaut sera remplacée par `false` dans une version ultérieure.|
|useGracefulShutdown|false|(Préversion) Activez l’arrêt approprié pour réduire le risque d’échecs d’arrêt de l’hôte dans les exécutions de fonctions in-process.|

La plupart de ces paramètres sont destinés à l’optimisation des performances. Pour plus d’informations, consultez [Performances et échelle](../articles/azure-functions/durable/durable-functions-perf-and-scale.md).