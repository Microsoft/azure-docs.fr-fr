---
title: Événement d’échec de planification de tâches Azure Batch
description: Référence pour l’événement d’échec de planification de tâches Batch. Cet événement est émis lorsqu’une tâche n’a pas pu être planifiée et qu’une nouvelle tentative aura lieu.
ms.topic: reference
ms.date: 09/20/2020
ms.openlocfilehash: 549281d2b2c371e8f09c584e771cf44f7abc8a00
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91852060"
---
# <a name="task-schedule-fail-event"></a>Événement d’échec de planification de tâches

 Cet événement est émis lorsqu’une tâche n’a pas pu être planifiée et qu’une nouvelle tentative aura lieu. Il s’agit d’une défaillance temporaire au moment de la planification des tâches, due à une limitation des ressources, telle qu’un nombre insuffisant d’emplacements disponibles sur les nœuds pour exécuter une tâche avec la valeur `requiredSlots` spécifiée.

 L’exemple suivant montre le corps d’un événement d’échec de planification de tâches.

```
{
    "jobId": "job-01",
    "id": "task-01",
    "taskType": "User",
    "systemTaskVersion": 665378862,
    "requiredSlots": 1,
    "nodeInfo": {
        "poolId": "pool-01",
        "nodeId": " "
    },
    "multiInstanceSettings": {
        "numberOfInstances": 1
    },
    "constraints": {
        "maxTaskRetryCount": 0
    },
    "schedulingError": {
        "category": "UserError",
        "code": "JobPreparationTaskFailed",
        "message": "Task cannot run because the job preparation task failed on node"
    }
}
```

|Nom de l'élément|Type|Notes|
|------------------|----------|-----------|
|`jobId`|String|ID du travail contenant la tâche.|
|`id`|String|ID de la tâche.|
|`taskType`|String|Type de la tâche. Ce peut être « JobManager », indiquant qu’il s’agit une tâche du gestionnaire, ou « User », indiquant qu’il ne s’agit pas d’une tâche du gestionnaire. Cet événement n’est pas émis pour des tâches de préparation du travail, des tâches de fin de travail ou des tâches de démarrage.|
|`systemTaskVersion`|Int32|Compteur de tentatives internes d’exécution d’une tâche. En interne, le service Batch peut recommencer une tâche pour prendre en compte des problèmes temporaires. Ces problèmes peuvent être des erreurs de planification internes ou des tentatives de récupération à partir de nœuds de calcul en mauvais état.|
|`requiredSlots`|Int32|Emplacements requis pour exécuter la tâche.|
|[`nodeInfo`](#nodeInfo)|Type complexe|Contient des informations sur le nœud de calcul sur lequel la tâche a été exécutée.|
|[`multiInstanceSettings`](#multiInstanceSettings)|Type complexe|Spécifie que la tâche est une tâche multi-instance nécessitant plusieurs nœuds de calcul.  Consultez [`multiInstanceSettings`](/rest/api/batchservice/get-information-about-a-task) pour plus de détails.|
|[`constraints`](#constraints)|Type complexe|Contraintes d’exécution qui s’appliquent à cette tâche.|
|[`schedulingError`](#schedulingError)|Type complexe|Contient des informations sur l’erreur de planification de la tâche.|

###  <a name="nodeinfo"></a><a name="nodeInfo"></a> nodeInfo

|Nom de l'élément|Type|Notes|
|------------------|----------|-----------|
|`poolId`|String|ID du pool sur lequel la tâche a été exécutée.|
|`nodeId`|String|ID du nœud sur lequel la tâche a été exécutée.|

###  <a name="multiinstancesettings"></a><a name="multiInstanceSettings"></a> multiInstanceSettings

|Nom de l'élément|Type|Notes|
|------------------|----------|-----------|
|`numberOfInstances`|Int32|Nombre de nœuds de calcul requis pour la tâche.|

###  <a name="constraints"></a><a name="constraints"></a> constraints

|Nom de l'élément|Type|Notes|
|------------------|----------|-----------|
|`maxTaskRetryCount`|Int32|Nombre maximal de fois que la tâche peut être retentée. Le service Batch retente une tâche si le code de sortie de celle-ci diffère de zéro.<br /><br /> Notez que cette valeur contrôle spécifiquement le nombre de nouvelles tentatives. Le service Batch tente la tâche une fois et peut réessayer jusqu’à cette limite. Par exemple, si le nombre maximal de nouvelles tentatives est 3, le service Batch peut tenter d’exécuter la tâche jusqu’à 4 fois (tentative initiale + 3 tentatives supplémentaires).<br /><br /> Si le nombre maximal de tentatives est 0, le service Batch ne réessaye pas d’exécuter des tâches.<br /><br /> Si le nombre maximal de nouvelles tentatives est -1, le service Batch réessaie d’exécuter les tâches sans limite.<br /><br /> La valeur par défaut est 0 (aucune nouvelle tentative).|


###  <a name="schedulingerror"></a><a name="schedulingError"></a> schedulingError

|Nom de l'élément|Type|Notes|
|------------------|----------|-----------|
|`category`|String|Catégorie de l’erreur.|
|`code`|String|Identificateur de l’erreur de planification des tâches. Les codes sont invariants et sont destinés à être consommés par programmation.|
|`message`|String|Message décrivant l’erreur de planification des tâches, conçu pour pouvoir être affiché dans une interface utilisateur.|
