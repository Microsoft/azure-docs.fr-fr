---
title: Gérer la sauvegarde de partage de fichiers Azure avec l’API REST
description: Découvrez comment utiliser l’API REST pour gérer et superviser les partages de fichiers Azure sauvegardés par Sauvegarde Azure.
ms.topic: conceptual
ms.date: 02/17/2020
ms.openlocfilehash: 5e2823472c6a7bdd6b3f9819db3079d7efa78c4e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88892845"
---
# <a name="manage-azure-file-share-backup-with-rest-api"></a>Gérer la sauvegarde de partage de fichiers Azure avec l’API REST

Cet article explique comment effectuer les tâches pour gérer et superviser les partages de fichiers Azure sauvegardés par [Sauvegarde Azure](./backup-overview.md).

## <a name="monitor-jobs"></a>Surveiller des travaux

Le service Sauvegarde Azure déclenche des travaux qui s’exécutent en arrière-plan. Cela comprend des scénarios tels que le déclenchement de la sauvegarde, les opérations de restauration et la désactivation de la sauvegarde. Ces tâches peuvent être suivies à l’aide de leurs ID.

### <a name="fetch-job-information-from-operations"></a>Récupérer des informations sur les travaux à partir des opérations

Une opération telle que le déclenchement d’une sauvegarde renvoie toujours un jobID dans la réponse.

Par exemple, la réponse finale d’une opération [de déclenchement de la sauvegarde par l’API REST](backup-azure-file-share-rest-api.md#trigger-an-on-demand-backup-for-file-share) est comme suit :

```json
{
    "id": "c3a52d1d-0853-4211-8141-477c65740264",
    "name": "c3a52d1d-0853-4211-8141-477c65740264",
    "status": "Succeeded",
    "startTime": "2020-02-03T18:10:48.296012Z",
    "endTime": "2020-02-03T18:10:48.296012Z",
    "properties": {
        "objectType": "OperationStatusJobExtendedInfo",
        "jobId": "e2ca2cf4-2eb9-4d4b-b16a-8e592d2a658b"
    }
}
```

Le travail de sauvegarde de partage de fichiers Azure est identifié par le champ **jobId** et peut être suivi comme indiqué [ici](/rest/api/backup/jobdetails/) en utilisant une requête GET.

### <a name="tracking-the-job"></a>Suivi de la tâche

```http
GET https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupJobs/{jobName}?api-version=2019-05-13
```

{jobName} correspond au « jobId » mentionné ci-dessus. La réponse est toujours « 200 OK » avec le champ **status** indiquant l’état actuel du travail. Lorsque l’état est « Completed » ou « CompletedWithWarnings », la section **extendedInfo** révèle plus de détails sur le travail.

```http
GET https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupJobs/e2ca2cf4-2eb9-4d4b-b16a-8e592d2a658b?api-version=2019-05-13'
```

#### <a name="response"></a>response

Nom  | Type  |  Description
--- | --- | ----
200 OK |  JobResource  | OK

#### <a name="response-example"></a>Exemple de réponse

Une fois l’URI *GET* envoyé, une réponse 200 est retournée.

```http
HTTP/1.1" 200
'Cache-Control': 'no-cache'
'Pragma': 'no-cache'
'Transfer-Encoding': 'chunked'
'Content-Type': 'application/json'
'Content-Encoding': 'gzip'
'Expires': '-1'
'Vary': 'Accept-Encoding'
'Server': 'Microsoft-IIS/10.0, Microsoft-IIS/10.0'
'X-Content-Type-Options': 'nosniff'
'x-ms-request-id': 'dba43f00-5cdb-43b1-a9ec-23e419db67c5'
'x-ms-client-request-id': 'a644712a-4895-11ea-ba57-0a580af42708, a644712a-4895-11ea-ba57-0a580af42708'
'X-Powered-By': 'ASP.NET'
'Strict-Transport-Security': 'max-age=31536000; includeSubDomains'
'x-ms-ratelimit-remaining-subscription-reads': '11999'
'x-ms-correlation-request-id': 'dba43f00-5cdb-43b1-a9ec-23e419db67c5'
'x-ms-routing-request-id': 'WESTEUROPE:20200206T040341Z:dba43f00-5cdb-43b1-a9ec-23e419db67c5'
'Date': 'Thu, 06 Feb 2020 04:03:40 GMT'
{
    "id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupJobs/e2ca2cf4-2eb9-4d4b-b16a-8e592d2a658b",
    "name": "e2ca2cf4-2eb9-4d4b-b16a-8e592d2a658b",
    "type": "Microsoft.RecoveryServices/vaults/backupJobs",
    "properties": {
        "jobType": "AzureStorageJob",
        "duration": "00:00:43.1809140",
        "storageAccountName": "testvault2",
        "storageAccountVersion": "Storage",
        "extendedInfo": {
            "tasksList": [],
            "propertyBag": {
                "File Share Name": "testshare",
                "Storage Account Name": "testvault2",
                "Policy Name": "schedule1"
            }
        },
        "entityFriendlyName": "testshare",
        "backupManagementType": "AzureStorage",
        "operation": "ConfigureBackup",
        "status": "Completed",
        "startTime": "2020-02-03T18:10:48.296012Z",
        "endTime": "2020-02-03T18:11:31.476926Z",
        "activityId": "3677cec0-942d-4eac-921f-8f3c873140d7"
    }
}
```

## <a name="modify-policy"></a>Modifier la stratégie

Pour modifier la stratégie avec laquelle le partage de fichiers est protégé, vous pouvez utiliser le même format que pour l’activation de la protection. Il vous suffit d’indiquer le nouvel ID de stratégie dans le corps de la demande et d’envoyer la demande.

Par exemple : Pour modifier la stratégie de protection de *testshare* de *schedule1* à *schedule2*, indiquez l’ID de *schedule2* dans le corps de la demande.

```json
{
  "properties": {
    "protectedItemType": "AzureFileShareProtectedItem",
    "sourceResourceId": "/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",
    "policyId": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupPolicies/schedule2"
  }
}
```

## <a name="stop-protection-but-retain-existing-data"></a>Arrêter la protection tout en conservant les données existantes

Vous pouvez supprimer la protection sur un partage de fichiers protégé tout en conservant les données déjà sauvegardées. Pour ce faire, supprimez la stratégie dans le corps de la demande que vous avez utilisée pour[activer la sauvegarde](backup-azure-file-share-rest-api.md#enable-backup-for-the-file-share) et envoyez la demande. Une fois l’association à la stratégie supprimée, les sauvegardes ne sont plus déclenchées et aucun point de récupération n’est créé.

```json
{
  "properties": {
    "protectedItemType": "AzureFileShareProtectedItem",
    "sourceResourceId": "/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",
    "policyId": “" ,
“protectionState”:”ProtectionStopped”
  }
}
```

### <a name="sample-response"></a>Exemple de réponse

L’arrêt de la protection d’un partage de fichiers est une opération asynchrone. L’opération crée une autre opération qui doit faire l’objet d’un suivi. Elle retourne deux réponses : 202 (Accepté) lors de la création d’une autre opération et 200 quand cette opération est terminée.

En-tête de réponse lorsque l’opération est acceptée avec succès :

```http
HTTP/1.1" 202
'Cache-Control': 'no-cache'
'Pragma': 'no-cache'
'Expires': '-1'
'Location': 'https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;testvault2/protectedItems/AzureFileShare;testshare/operationResults/b300922a-ad9c-4181-b4cd-d42ea780ad77?api-version=2019-05-13'
'Retry-After': '60'
msrest.http_logger :     'Azure-AsyncOperation': 'https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;testvault2/protectedItems/AzureFileShare;testshare/operationsStatus/b300922a-ad9c-4181-b4cd-d42ea780ad77?api-version=2019-05-13'
'X-Content-Type-Options': 'nosniff'
'x-ms-request-id': '3895e8a1-e4b9-4da5-bec7-2cf0266405f8'
'x-ms-client-request-id': 'd331c15e-48ab-11ea-84c0-0a580af46a50, d331c15e-48ab-11ea-84c0-0a580af46a50'
'Strict-Transport-Security': 'max-age=31536000; includeSubDomains'
'X-Powered-By': 'ASP.NET'
'x-ms-ratelimit-remaining-subscription-writes': '1199'
'x-ms-correlation-request-id': '3895e8a1-e4b9-4da5-bec7-2cf0266405f8'
'x-ms-routing-request-id': 'WESTEUROPE:20200206T064224Z:3895e8a1-e4b9-4da5-bec7-2cf0266405f8'
'Date': 'Thu, 06 Feb 2020 06:42:24 GMT'
'Content-Length': '0'
```

Effectuez ensuite le suivi de l’opération qui en résulte en utilisant l’en-tête d’emplacement ou l’en-tête Azure-AsyncOperation avec une commande GET :

```http
GET https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupoperations/b300922a-ad9c-4181-b4cd-d42ea780ad77?api-version=2016-12-01
```

### <a name="response-body"></a>Response body

```json
{
    "id": "b300922a-ad9c-4181-b4cd-d42ea780ad77",
    "name": "b300922a-ad9c-4181-b4cd-d42ea780ad77",
    "status": "Succeeded",
    "startTime": "2020-02-06T06:42:24.4001299Z",
    "endTime": "2020-02-06T06:42:24.4001299Z",
    "properties": {
        "objectType": "OperationStatusJobExtendedInfo",
        "jobId": "7816fca8-d5be-4c41-b911-1bbd922e5826"
    }
}
```

## <a name="stop-protection-and-delete-data"></a>Arrêter la protection et supprimer les données

Pour supprimer la protection sur un partage de fichiers protégé ainsi que les données de sauvegarde, effectuez une opération de suppression comme indiqué [ici](/rest/api/backup/protecteditems/delete).

```http
DELETE https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}?api-version=2019-05-13
```

Les paramètres {containerName} et {protectedItemName} sont définis [ici](restore-azure-file-share-rest-api.md#fetch-containername-and-protecteditemname).

L’exemple suivant déclenche une opération pour arrêter la protection du partage de fichiers *testshare* protégé par *azurefilesvault*.

```http
DELETE https://management.azure.com/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/protectedItems/azurefileshare;testshare?api-version=2016-12-01
```

### <a name="responses"></a>Réponses

La suppression de la protection est une opération asynchrone. L’opération crée une autre opération qui doit faire l’objet d’un suivi distinct.
Elle retourne deux réponses : 202 (Accepté) lors de la création d’une autre opération et 204 (Pas de contenu) quand cette opération est terminée.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [détecter un problème lors de la configuration de la sauvegarde pour les partages de fichiers Azure](troubleshoot-azure-files.md).
