---
title: Gérer les travaux de sauvegarde à l'aide de l'API REST
description: Dans cet article, découvrez comment suivre et gérer les tâches de sauvegarde et de restauration de Sauvegarde Azure avec l’API REST.
ms.topic: conceptual
ms.date: 08/03/2018
ms.assetid: b234533e-ac51-4482-9452-d97444f98b38
ms.openlocfilehash: ced0e0020fe955734bf6cc767480fbadd6eaffc1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88890278"
---
# <a name="track-backup-and-restore-jobs-using-rest-api"></a>Suivi des tâches de sauvegarde et de restauration à l’aide de l’API REST

Le service Sauvegarde Azure déclenche des tâches qui s’exécutent en arrière-plan dans divers scénarios tels que le déclenchement de la sauvegarde, les opérations de restauration et la désactivation de la sauvegarde. Ces tâches peuvent être suivies à l’aide de leurs ID.

## <a name="fetch-job-information-from-operations"></a>Récupérer les informations sur tâches à partir des opérations

Une opération telle que le déclenchement d’une sauvegarde renvoie toujours un ID de tâche (jobID). Par exemple : voici la réponse finale d'une [opération de déclenchement de l'API REST de sauvegarde](backup-azure-arm-userestapi-backupazurevms.md#example-responses-for-on-demand-backup) :

```http
{
  "id": "cd153561-20d3-467a-b911-cc1de47d4763",
  "name": "cd153561-20d3-467a-b911-cc1de47d4763",
  "status": "Succeeded",
  "startTime": "2018-09-12T02:16:56.7399752Z",
  "endTime": "2018-09-12T02:16:56.7399752Z",
  "properties": {
    "objectType": "OperationStatusJobExtendedInfo",
    "jobId": "41f3e94b-ae6b-4a20-b422-65abfcaf03e5"
  }
}
```

La tâche de sauvegarde de machine virtuelle est identifiée par le champ « jobId » et peut être suivie comme indiqué [ici](/rest/api/backup/jobdetails/) en utilisant une simple requête *GET*.

## <a name="tracking-the-job"></a>Suivi de la tâche

```http
GET https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupJobs/{jobName}?api-version=2019-05-13
```

L’`{jobName}` est « jobId » comme indiqué précédemment. La réponse est toujours 200 OK avec le champ « status » indiquant l’état actuel de la tâche. Lorsque l’état est « Completed » ou « CompletedWithWarnings », la section « extendedInfo » révèle plus de détails sur le travail.

### <a name="response"></a>response

|Nom  |Type  |Description  |
|---------|---------|---------|
|200 OK     | [JobResource](/rest/api/backup/jobdetails/get#jobresource)        | OK        |

#### <a name="example-response"></a>Exemple de réponse

Une fois l’URI *GET* envoyé, une réponse 200 (OK) est retournée.

```http
HTTP/1.1 200 OK
Pragma: no-cache
X-Content-Type-Options: nosniff
x-ms-request-id: e9702101-9da2-4681-bdf3-a54e17329a56
x-ms-client-request-id: ba4dff71-1655-4c1d-a71f-c9869371b18b; ba4dff71-1655-4c1d-a71f-c9869371b18b
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-reads: 14989
x-ms-correlation-request-id: e9702101-9da2-4681-bdf3-a54e17329a56
x-ms-routing-request-id: SOUTHINDIA:20180521T102317Z:e9702101-9da2-4681-bdf3-a54e17329a56
Cache-Control: no-cache
Date: Mon, 21 May 2018 10:23:17 GMT
Server: Microsoft-IIS/8.0
X-Powered-By: ASP.NET

{
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Default-RecoveryServices-ResourceGroup-centralindia/providers/microsoft.recoveryservices/vaults/abdemovault/backupJobs/7ddead57-bcb9-4269-ac31-6a1b57588700",
  "name": "7ddead57-bcb9-4269-ac31-6a1b57588700",
  "type": "Microsoft.RecoveryServices/vaults/backupJobs",
  "properties": {
    "jobType": "AzureIaaSVMJob",
    "duration": "00:20:23.0896697",
    "actionsInfo": [
      1
    ],
    "virtualMachineVersion": "Compute",
    "extendedInfo": {
      "tasksList": [
        {
          "taskId": "Take Snapshot",
          "duration": "00:00:00",
          "status": "Completed"
        },
        {
          "taskId": "Transfer data to vault",
          "duration": "00:00:00",
          "status": "Completed"
        }
      ],
      "propertyBag": {
        "VM Name": "uttestvmub1",
        "Backup Size": "2332 MB"
      }
    },
    "entityFriendlyName": "uttestvmub1",
    "backupManagementType": "AzureIaasVM",
    "operation": "Backup",
    "status": "Completed",
    "startTime": "2018-05-21T08:35:40.9488967Z",
    "endTime": "2018-05-21T08:56:04.0385664Z",
    "activityId": "7df8e874-1d66-4f81-8e91-da2fe054811d"
  }
}
}

```
