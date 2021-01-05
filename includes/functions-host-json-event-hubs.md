---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/04/2018
ms.author: glenga
ms.openlocfilehash: 8f3a58d3a7470867ab23249bbd645289e010ad89
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95997298"
---
### <a name="functions-2x-and-higher"></a>Functions 2.x et versions ultérieures

```json
{
    "version": "2.0",
    "extensions": {
        "eventHubs": {
            "batchCheckpointFrequency": 5,
            "eventProcessorOptions": {
                "maxBatchSize": 256,
                "prefetchCount": 512
            }
        }
    }
}  
```

|Propriété  |Default | Description |
|---------|---------|---------|
|maxBatchSize|10|Nombre d’événements maximal reçu par boucle de réception.|
|prefetchCount|300|Nombre de prérécupérations par défaut utilisé par le `EventProcessorHost` sous-jacent. La valeur minimale autorisée est de 10.|
|batchCheckpointFrequency|1|Nombre de lots d’événements à traiter avant de créer un point de contrôle de curseur EventHub.|

> [!NOTE]
> Pour obtenir des informations de référence sur le fichier host.json dans Azure Functions 2.x et ultérieur, consultez [Informations de référence sur le fichier host.json pour Azure Functions](../articles/azure-functions/functions-host-json.md).

### <a name="functions-1x"></a>Functions 1.x

```json
{
    "eventHub": {
      "maxBatchSize": 64,
      "prefetchCount": 256,
      "batchCheckpointFrequency": 1
    }
}
```

|Propriété  |Default | Description |
|---------|---------|---------| 
|maxBatchSize|64|Nombre d’événements maximal reçu par boucle de réception.|
|prefetchCount|n/a|Prérécupérations par défaut utilisées par le `EventProcessorHost` sous-jacent.| 
|batchCheckpointFrequency|1|Nombre de lots d’événements à traiter avant de créer un point de contrôle de curseur EventHub.| 

> [!NOTE]
> Pour obtenir des informations de référence sur le fichier host.json dans Azure Functions 1.x, consultez [Informations de référence sur le fichier host.json pour Azure Functions 1.x](../articles/azure-functions/functions-host-json-v1.md).
