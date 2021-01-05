---
title: Restaurer des machines virtuelles Azure avec l’API REST
description: Dans cet article, découvrez comment gérer les opérations de restauration de la sauvegarde de machines virtuelles Azure à l’aide de l’API REST.
ms.topic: conceptual
ms.date: 09/12/2018
ms.assetid: b8487516-7ac5-4435-9680-674d9ecf5642
ms.openlocfilehash: 260c78af39c46e493ebb79c26ff1c55153a41c1d
ms.sourcegitcommit: 2989396c328c70832dcadc8f435270522c113229
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92174031"
---
# <a name="restore-azure-virtual-machines-using-rest-api"></a>Restaurer des machines virtuelles avec l’API REST

Après la sauvegarde d’une machine virtuelle Azure à l’aide de Sauvegarde Azure, il est possible de restaurer soit la totalité de cette machine virtuelle, soit des disques ou des fichiers à partir de la même copie de sauvegarde. Cet article explique comment restaurer une machine virtuelle Azure ou des disques avec l’API REST.

Quelle que soit l’opération de restauration, il faut commencer par identifier le point de récupération pertinent.

## <a name="select-recovery-point"></a>Sélectionner un point de récupération

[L’API REST Lister les points de récupération](/rest/api/backup/recoverypoints/list) permet de lister les points de récupération disponibles d’un élément de sauvegarde. Il s’agit d’une simple opération *GET* avec toutes les valeurs nécessaires.

```http
GET https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}/recoveryPoints?api-version=2019-05-13
```

`{containerName}` et `{protectedItemName}` sont tels qu’ils ont été créés [ici](backup-azure-arm-userestapi-backupazurevms.md#example-responses-to-get-operation). `{fabricName}` correspond à « Azure ».

L’URI *GET* contient tous les paramètres obligatoires. Il n’est pas nécessaire d’ajouter un corps de demande.

### <a name="responses"></a>Réponses

|Nom  |Type  |Description  |
|---------|---------|---------|
|200 OK     |   [RecoveryPointResourceList](/rest/api/backup/recoverypoints/list#recoverypointresourcelist)      |       OK  |

#### <a name="example-response"></a>Exemple de réponse

Une fois l’URI *GET* envoyé, une réponse 200 (OK) est retournée.

```http
HTTP/1.1 200 OK
Pragma: no-cache
X-Content-Type-Options: nosniff
x-ms-request-id: 03453538-2f8d-46de-8374-143ccdf60f40
x-ms-client-request-id: c48f4436-ce3f-42da-b537-12710d4d1a24; c48f4436-ce3f-42da-b537-12710d4d1a24
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-reads: 14998
x-ms-correlation-request-id: 03453538-2f8d-46de-8374-143ccdf60f40
x-ms-routing-request-id: SOUTHINDIA:20180604T071851Z:03453538-2f8d-46de-8374-143ccdf60f40
Cache-Control: no-cache
Date: Mon, 04 Jun 2018 07:18:51 GMT
Server: Microsoft-IIS/8.0
X-Powered-By: ASP.NET

{
  "value": [
    {
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/VM;testRG;testVM/recoveryPoints/20982486783671",
      "name": "20982486783671",
      "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints",
      "properties": {
        "objectType": "IaasVMRecoveryPoint",
        "recoveryPointType": "AppConsistent",
        "recoveryPointTime": "2018-06-04T06:06:00.5121087Z",
        "recoveryPointAdditionalInfo": "",
        "sourceVMStorageType": "NormalStorage",
        "isSourceVMEncrypted": false,
        "isInstantIlrSessionActive": false,
        "recoveryPointTierDetails": [
          {
            "type": 1,
            "status": 1
          },
          {
            "type": 2,
            "status": 1
          }
        ],
        "isManagedVirtualMachine": true,
        "virtualMachineSize": "Standard_A1_v2",
        "originalStorageAccountOption": false
      }
    },
    {
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/VM;testRG;testVM/recoveryPoints/23358112038108",
      "name": "23358112038108",
      "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints",
      "properties": {
        "objectType": "IaasVMRecoveryPoint",
        "recoveryPointType": "CrashConsistent",
        "recoveryPointTime": "2018-06-03T06:20:56.3043878Z",
        "recoveryPointAdditionalInfo": "",
        "sourceVMStorageType": "NormalStorage",
        "isSourceVMEncrypted": false,
        "isInstantIlrSessionActive": false,
        "recoveryPointTierDetails": [
          {
            "type": 1,
            "status": 1
          },
          {
            "type": 2,
            "status": 1
          }
        ],
        "isManagedVirtualMachine": true,
        "virtualMachineSize": "Standard_A1_v2",
        "originalStorageAccountOption": false
      }
    },
......
```

Le point de récupération est identifié par le champ `{name}` dans la réponse ci-dessus.

## <a name="restore-operations"></a>Opérations de restauration

Après avoir sélectionné le [point de restauration approprié](#select-recovery-point), procédez au déclenchement de l’opération de restauration.

***Toutes les opérations de restauration sur l’élément de sauvegarde sont effectuées avec la même API *POST*. Seul le corps de la requête change avec les scénarios de restauration.***

> [!IMPORTANT]
> Tous les détails sur les différentes options de restauration et leurs dépendances sont mentionnés [ici](./backup-azure-arm-restore-vms.md#restore-options). Veuillez consulter ces détails avant de commencer à déclencher ces opérations.

Le déclenchement des opérations de restauration est une requête *POST*. Pour en savoir plus sur l’API, consultez l’[API REST « déclencher la restauration »](/rest/api/backup/restores/trigger).

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}/recoveryPoints/{recoveryPointId}/restore?api-version=2019-05-13
```

`{containerName}` et `{protectedItemName}` sont tels qu’ils ont été créés [ici](backup-azure-arm-userestapi-backupazurevms.md#example-responses-to-get-operation). `{fabricName}` correspond à « Azure » et `{recoveryPointId}` est le champ `{name}` du point de récupération mentionné [ci-dessus](#example-response).

Une fois le point de récupération obtenu, nous devons construire le corps de la requête pour le scénario de restauration correspondant. Les sections suivantes décrivent le corps de la requête pour chaque scénario.

- [Restaurer des disques](#restore-disks)
- [Remplacer des disques](#replace-disks-in-a-backed-up-virtual-machine)
- [Restaurer en tant que nouvelle machine virtuelle](#restore-as-another-virtual-machine)

### <a name="restore-response"></a>Réponse de restauration

Le déclenchement de toute opération de restauration est une [opération asynchrone](../azure-resource-manager/management/async-operations.md). ce qui signifie qu’elle crée une autre opération qui doit faire l’objet d’un suivi distinct.

Elle retourne deux réponses : 202 (Accepté) lors de la création d’une autre opération, puis 200 (OK) quand cette opération est terminée.

|Nom  |Type  |Description  |
|---------|---------|---------|
|202 Accepté     |         |     Acceptée    |

#### <a name="example-responses"></a>Exemples de réponses

Une fois l’URI *POST* envoyé pour déclencher la restauration des disques, la réponse initiale est 202 (Accepté) avec un en-tête d’emplacement ou Azure-async-header.

```http
HTTP/1.1 202 Accepted
Pragma: no-cache
Retry-After: 60
Azure-AsyncOperation: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationsStatus/781a0f18-e250-4d73-b059-5e9ffed4069e?api-version=2019-05-13
X-Content-Type-Options: nosniff
x-ms-request-id: 893fe372-8d6c-4c56-b589-45a95eeef95f
x-ms-client-request-id: a15ce064-25bd-4ac6-87e5-e3bc6ec65c0b; a15ce064-25bd-4ac6-87e5-e3bc6ec65c0b
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-writes: 1198
x-ms-correlation-request-id: 893fe372-8d6c-4c56-b589-45a95eeef95f
x-ms-routing-request-id: SOUTHINDIA:20180604T130003Z:893fe372-8d6c-4c56-b589-45a95eeef95f
Cache-Control: no-cache
Date: Mon, 04 Jun 2018 13:00:03 GMT
Location: https://management.azure.com/subscriptions//subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationResults/781a0f18-e250-4d73-b059-5e9ffed4069e?api-version=2019-05-13
X-Powered-By: ASP.NET
```

Effectuez ensuite le suivi de l’opération qui en résulte en utilisant l’en-tête d’emplacement ou Azure-AsyncOperation avec une commande *GET* simple.

```http
GET https://management.azure.com/subscriptions//subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationResults/781a0f18-e250-4d73-b059-5e9ffed4069e?api-version=2019-05-13
```

Une fois terminée, l’opération retourne 200 (OK) avec l’ID de la tâche de restauration obtenue dans le corps de la réponse.

```http
HTTP/1.1 200 OK
Pragma: no-cache
X-Content-Type-Options: nosniff
x-ms-request-id: ea2a8011-eb83-4a4b-9ed2-e694070a966a
x-ms-client-request-id: a7f3a144-ed80-41ee-bffe-ae6a90c35a2f; a7f3a144-ed80-41ee-bffe-ae6a90c35a2f
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-reads: 14973
x-ms-correlation-request-id: ea2a8011-eb83-4a4b-9ed2-e694070a966a
x-ms-routing-request-id: SOUTHINDIA:20180604T130917Z:ea2a8011-eb83-4a4b-9ed2-e694070a966a
Cache-Control: no-cache
Date: Mon, 04 Jun 2018 13:09:17 GMT
Server: Microsoft-IIS/8.0
X-Powered-By: ASP.NET

{
  "id": "781a0f18-e250-4d73-b059-5e9ffed4069e",
  "name": "781a0f18-e250-4d73-b059-5e9ffed4069e",
  "status": "Succeeded",
  "startTime": "2018-06-04T13:00:03.8068176Z",
  "endTime": "2018-06-04T13:00:03.8068176Z",
  "properties": {
    "objectType": "OperationStatusJobExtendedInfo",
    "jobId": "3021262a-fb3a-4538-9b37-e3e97a386093"
  }
}
```

Le travail de restauration étant une opération longue, il doit faire l’objet d’un suivi, qui est détaillé dans le [document sur la supervision des travaux avec l’API REST](backup-azure-arm-userestapi-managejobs.md#tracking-the-job).

### <a name="restore-disks"></a>Restaurer des disques

Pour personnaliser la création d’une machine virtuelle à partir des données de sauvegarde, il vous suffit de restaurer les disques dans un compte de stockage choisi et de créer une machine virtuelle à partir de ces disques en fonction des besoins. Le compte de stockage doit se trouver dans la même région que le coffre Recovery Services et ne pas être redondant interzone. Les disques, ainsi que la configuration de la machine virtuelle sauvegardée (« vmconfig.json »), sont stockés dans le compte de stockage indiqué. Comme expliqué [ci-dessus](#restore-operations), le corps de la requête approprié pour la restauration de disques est fourni ci-dessous.

#### <a name="create-request-body"></a>Créer un corps de demande

Voici les composants du corps de la demande pour déclencher une restauration de disque à partir d’une sauvegarde de machine virtuelle Azure.

|Nom  |Type  |Description  |
|---------|---------|---------|
|properties     | [IaaSVMRestoreRequest](/rest/api/backup/restores/trigger#iaasvmrestorerequest)        |    RestoreRequestResourceProperties     |

Pour obtenir la liste complète des définitions du corps de la demande et d’autres détails, voir le [document API REST Déclencher la restauration](/rest/api/backup/restores/trigger#request-body).

##### <a name="example-request"></a>Exemple de requête

Le corps de demande suivant définit les propriétés requises pour déclencher une restauration de disque.

```json
{
  "properties": {
    "objectType": "IaasVMRestoreRequest",
    "recoveryPointId": "20982486783671",
    "recoveryType": "RestoreDisks",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "storageAccountId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Storage/storageAccounts/testAccount",
    "region": "westus",
    "createNewCloudService": false,
    "originalStorageAccountOption": false,
    "encryptionDetails": {
      "encryptionEnabled": false
    }
  }
}
```

### <a name="restore-disks-selectively"></a>Restaurer des disques de manière sélective

Si vous [sauvegardez des disques de manière sélective](backup-azure-arm-userestapi-backupazurevms.md#excluding-disks-in-azure-vm-backup), la liste de disques sauvegardés est fournie dans le [résumé des points de récupération](#select-recovery-point) et la [réponse détaillée](/rest/api/backup/recoverypoints/get). Vous pouvez également restaurer des disques de manière sélective. Plus de détails sont disponibles [ici](selective-disk-backup-restore.md#selective-disk-restore). Pour restaurer un disque de manière sélective dans la liste des disques sauvegardés, recherchez le numéro d’unité logique du disque à partir de la réponse du point de récupération et ajoutez la propriété **restoreDiskLunList** au [corps de la demande ci-dessus](#example-request) comme indiqué ci-dessous.

```json
{
    "properties": {
        "objectType": "IaasVMRestoreRequest",
        "recoveryPointId": "20982486783671",
        "recoveryType": "RestoreDisks",
        "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
        "storageAccountId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Storage/storageAccounts/testAccount",
        "region": "westus",
        "createNewCloudService": false,
        "originalStorageAccountOption": false,
        "encryptionDetails": {
          "encryptionEnabled": false
        },
        "restoreDiskLunList" : [0]
    }
}

```

Une fois que vous effectuez le suivi de la réponse comme expliqué [ci-dessus](#responses) et que le long travail est terminé, les disques et la configuration de la machine virtuelle sauvegardée (« VMConfig.json ») sont présents dans le compte de stockage indiqué.

### <a name="replace-disks-in-a-backed-up-virtual-machine"></a>Remplacer des disques dans une machine virtuelle sauvegardée

Tandis que l’opération de restauration de disques crée des disques à partir du point de récupération, l’opération de remplacement de disques remplace les disques actuels de la machine virtuelle sauvegardée par ceux du point de récupération. Comme expliqué [ci-dessus](#restore-operations), le corps de la requête approprié pour le remplacement de disques est fourni ci-dessous.

#### <a name="create-request-body"></a>Créer un corps de demande

Voici les composants du corps de la requête pour déclencher un remplacement de disques à partir d’une sauvegarde de machine virtuelle Azure.

|Name  |Type  |Description  |
|---------|---------|---------|
|properties     | [IaaSVMRestoreRequest](/rest/api/backup/restores/trigger#iaasvmrestorerequest)        |    RestoreRequestResourceProperties     |

Pour obtenir la liste complète des définitions du corps de la demande et d’autres détails, voir le [document API REST Déclencher la restauration](/rest/api/backup/restores/trigger#request-body).

#### <a name="example-request"></a>Exemple de requête

Le corps de demande suivant définit les propriétés requises pour déclencher une restauration de disque.

```json
{
    "properties": {
        "objectType": "IaasVMRestoreRequest",
        "recoveryPointId": "20982486783671",
        "recoveryType": "OriginalLocation",
        "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
        "storageAccountId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Storage/storageAccounts/testAccount",  
        "region": "westus",
        "createNewCloudService": false,
        "originalStorageAccountOption": false,
        "affinityGroup": "",
        "diskEncryptionSetId": null,
        "subnetId": null,
        "targetDomainNameId": null,
        "targetResourceGroupId": null,
        "targetVirtualMachineId": null,
        "virtualNetworkId": null
     }
}

```

### <a name="restore-as-another-virtual-machine"></a>Restaurer sous la forme d’une autre machine virtuelle

Comme expliqué [ci-dessus](#restore-operations), le corps de requête suivant définit les propriétés requises pour déclencher une restauration de machine virtuelle.

```json
{
  "parameters": {
        "subscriptionId": "00000000-0000-0000-0000-000000000000",
        "resourceGroupName": "testVaultRG",
        "vaultName": "testVault",
        "fabricName": "Azure",
        "containerName": "IaasVMContainer;iaasvmcontainerv2;testRG;testVM",
        "protectedItemName": "VM;iaasvmcontainerv2;testRG;testVM",
        "recoveryPointId": "348916168024334",
        "api-version": "2019-05-13",
      "parameters": {
        "properties": {
          "objectType":  "IaasVMRestoreRequest",
          "recoveryPointId": "348916168024334",
          "recoveryType": "AlternateLocation",
          "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
          "targetVirtualMachineId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/targetRG/providers/Microsoft.Compute/virtualmachines/targetVM",
          "targetResourceGroupId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/targetRG",
          "storageAccountId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Storage/storageAccounts/testingAccount",
          "virtualNetworkId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/targetRG/providers/Microsoft.Network/virtualNetworks/testNet",
          "subnetId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/targetRG/providers/Microsoft.Network/virtualNetworks/testNet/subnets/default",
          "region": "westus",
          "createNewCloudService": false,
          "originalStorageAccountOption": false,
          "encryptionDetails": {
            "encryptionEnabled": false
          }
        }
      }
    }
}
```

La réponse doit être gérée comme [pour la restauration de disques](#responses).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les API REST de la Sauvegarde Azure, voir les documents suivants :

- [API REST du fournisseur Azure Recovery Services](/rest/api/recoveryservices/)
- [Bien démarrer avec l’API REST Azure](/rest/api/azure/)