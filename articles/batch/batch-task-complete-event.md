---
title: Événement de fin de tâche Azure Batch
description: Référence pour l’événement de fin de tâche Batch. Cet événement est émis quand une tâche est terminée, quel que soit le code de sortie.
ms.topic: reference
ms.date: 10/08/2020
ms.openlocfilehash: 11f727b07723f32cd08130b4af17e57ede3d264f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91850889"
---
# <a name="task-complete-event"></a>Événement de fin de tâche

 Cet événement est émis quand une tâche est terminée, quel que soit le code de sortie. Il permet de déterminer la durée d’une tâche, son emplacement d’exécution, et si elle a été tentée plus d’une fois.


 L’exemple suivant montre le corps d’un événement de fin de tâche.

```
{
    "jobId": "myJob",
    "id": "myTask",
    "taskType": "User",
    "systemTaskVersion": 0,
    "requiredSlots": 1,
    "nodeInfo": {
        "poolId": "pool-001",
        "nodeId": "tvm-257509324_1-20160908t162728z"
    },
    "multiInstanceSettings": {
        "numberOfInstances": 1
    },
    "constraints": {
        "maxTaskRetryCount": 2
    },
    "executionInfo": {
        "startTime": "2016-09-08T16:32:23.799Z",
        "endTime": "2016-09-08T16:34:00.666Z",
        "exitCode": 0,
        "retryCount": 0,
        "requeueCount": 0
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
|[`executionInfo`](#executionInfo)|Type complexe|Contient des informations sur l’exécution de la tâche.|

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

###  <a name="executioninfo"></a><a name="executionInfo"></a> executionInfo

|Nom de l'élément|Type|Notes|
|------------------|----------|-----------|
|`startTime`|DateTime|Heure à laquelle l’exécution de la tâche a commencé. « Running » correspond à l’état **en cours d’exécution**. Ainsi, si la tâche spécifie des fichiers de ressources ou des packages d’applications, l’heure de début est l’heure à laquelle la tâche a commencé à télécharger et à déployer ces éléments.  Si la tâche a été redémarrée ou retentée, il s’agit de la dernière heure à laquelle l’exécution de la tâche a commencé.|
|`endTime`|DateTime|Heure à laquelle la tâche s’est terminée.|
|`exitCode`|Int32|Code de sortie de la tâche.|
|`retryCount`|Int32|Nombre de fois que le service Batch a réessayé d’exécuter la tâche. Si la tâche se termine avec un code de sortie autre que zéro, elle est retentée le nombre de fois spécifié par la valeur MaxTaskRetryCount.|
|`requeueCount`|Int32|Nombre de fois que la tâche a été replacée en file d’attente par le service Batch à la suite d’une demande de l’utilisateur.<br /><br /> Lorsque l’utilisateur supprime des nœuds d’un pool (en redimensionnant ou en réduisant le pool), ou quand le travail est désactivé, l’utilisateur peut spécifier que les tâches en cours d’exécution sur les nœuds doivent être replacée en file d’attente pour exécution. Ce nombre reflète le nombre de fois que la tâche a été replacée en file d’attente pour ces raisons.|
