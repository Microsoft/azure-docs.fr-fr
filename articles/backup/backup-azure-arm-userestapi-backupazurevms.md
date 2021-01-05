---
title: Sauvegarder des machines virtuelles Azure avec l’API REST
description: Dans cet article, découvrez comment configurer, lancer et gérer les opérations de la sauvegarde de machines virtuelles Azure avec l’API REST.
ms.topic: conceptual
ms.date: 08/03/2018
ms.assetid: b80b3a41-87bf-49ca-8ef2-68e43c04c1a3
ms.openlocfilehash: 9ba22c51c7a6c26a232ed20aec21fc83d2c54b37
ms.sourcegitcommit: 2989396c328c70832dcadc8f435270522c113229
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92171456"
---
# <a name="back-up-an-azure-vm-using-azure-backup-via-rest-api"></a>Sauvegarder une machine virtuelle Azure à l’aide de la sauvegarde Azure via une API REST

Cet article décrit comment gérer des sauvegardes pour une machine virtuelle Azure à l’aide de la sauvegarde Azure via une API REST. Configurez la protection pour la première fois pour une machine virtuelle Azure précédemment non protégée, déclenchez une sauvegarde à la demande pour une machine virtuelle Azure protégée et modifiez les propriétés de sauvegarde d’une machine virtuelle sauvegardée par le biais d’une API REST, comme expliqué ici.

Reportez-vous aux tutoriels sur les API REST [créer un coffre](backup-azure-arm-userestapi-createorupdatevault.md) et [créer une stratégie](backup-azure-arm-userestapi-createorupdatepolicy.md) pour la création de coffres et de stratégies.

Supposons que vous souhaitez protéger une machine virtuelle « testVM » sous un groupe de ressources « testRG » dans un coffre Recovery Services « testVault », présent dans le groupe de ressources « testVaultRG », avec la stratégie par défaut (nommée « DefaultPolicy »).

## <a name="configure-backup-for-an-unprotected-azure-vm-using-rest-api"></a>Configurer la sauvegarde pour une machine virtuelle Azure non protégée avec une API REST

### <a name="discover-unprotected-azure-vms"></a>Découvrir les machines virtuelles Azure non protégées

Tout d’abord, le coffre doit être en mesure d’identifier la machine virtuelle Azure. Cette action est déclenchée à l’aide de l’[opération d’actualisation](/rest/api/backup/protectioncontainers/refresh). Il s’agit d’une opération *POST* asynchrone qui garantit que le coffre obtient la liste la plus récente de toutes les machines virtuelles non protégées dans l’abonnement actuel et les « met en cache ». Une fois que la machine virtuelle est « mise en cache », Recovery Services est en mesure d’y accéder et de la protéger.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{vaultresourceGroupname}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/refreshContainers?api-version=2016-12-01
```

L’URI POST contient les paramètres `{subscriptionId}`, `{vaultName}`, `{vaultresourceGroupName}`, `{fabricName}`. `{fabricName}` est « Azure ». Selon notre exemple, `{vaultName}` correspond à « testVault » et `{vaultresourceGroupName}` à « testVaultRG ». Étant donné que tous les paramètres nécessaires sont fournis dans l’URI, un corps de demande distinct est inutile.

```http
POST https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/refreshContainers?api-version=2016-12-01
```

#### <a name="responses-to-refresh-operation"></a>Réponses à l’opération d’actualisation

L’opération « Actualiser » est une [opération asynchrone](../azure-resource-manager/management/async-operations.md). ce qui signifie qu’elle crée une autre opération qui doit faire l’objet d’un suivi distinct.

Elle retourne deux réponses : 202 (Accepté) lors de la création d’une autre opération, puis 200 (OK) quand cette opération est terminée.

|Nom  |Type  |Description  |
|---------|---------|---------|
|204 Pas de contenu     |         |  OK sans contenu retourné      |
|202 Accepté     |         |     Acceptée    |

##### <a name="example-responses-to-refresh-operation"></a>Exemples de réponses à l’opération d’actualisation

Une fois la demande *POST* envoyée, une réponse 202 (Accepté) est retournée.

```http
HTTP/1.1 202 Accepted
Pragma: no-cache
Retry-After: 60
X-Content-Type-Options: nosniff
x-ms-request-id: 43cf550d-e463-421c-8922-37e4766db27d
x-ms-client-request-id: 4910609f-bb9b-4c23-8527-eb6fa2d3253f; 4910609f-bb9b-4c23-8527-eb6fa2d3253f
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-writes: 1199
x-ms-correlation-request-id: 43cf550d-e463-421c-8922-37e4766db27d
x-ms-routing-request-id: SOUTHINDIA:20180521T105701Z:43cf550d-e463-421c-8922-37e4766db27d
Cache-Control: no-cache
Date: Mon, 21 May 2018 10:57:00 GMT
Location: https://management.azure.com/subscriptions//00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/operationResults/aad204aa-a5cf-4be2-a7db-a224819e5890?api-version=2019-05-13
X-Powered-By: ASP.NET
```

Effectuez le suivi de l’opération qui en résulte à l’aide de l’en-tête « Location » avec une commande *GET* simple

```http
GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/operationResults/aad204aa-a5cf-4be2-a7db-a224819e5890?api-version=2019-05-13
```

Une fois toutes les machines virtuelles Azure découvertes, la commande GET retourne une réponse 204 (Pas de contenu). Le coffre est désormais capable de découvrir toutes les machines virtuelles au sein de l’abonnement.

```http
HTTP/1.1 204 NoContent
Pragma: no-cache
X-Content-Type-Options: nosniff
x-ms-request-id: cf6cd73b-9189-4942-a61d-878fcf76b1c1
x-ms-client-request-id: 25bb6345-f9fc-4406-be1a-dc6db0eefafe; 25bb6345-f9fc-4406-be1a-dc6db0eefafe
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-reads: 14997
x-ms-correlation-request-id: cf6cd73b-9189-4942-a61d-878fcf76b1c1
x-ms-routing-request-id: SOUTHINDIA:20180521T105825Z:cf6cd73b-9189-4942-a61d-878fcf76b1c1
Cache-Control: no-cache
Date: Mon, 21 May 2018 10:58:25 GMT
X-Powered-By: ASP.NET
```

### <a name="selecting-the-relevant-azure-vm"></a>Sélection de la machine virtuelle Azure appropriée

 Vous pouvez vérifier que la « mise en cache » est effectuée en [listant tous les éléments pouvant être protégés](/rest/api/backup/backupprotectableitems/list) sous l’abonnement et rechercher la machine virtuelle souhaitée dans la réponse. [La réponse de cette opération](#example-responses-to-get-operation) vous fournit également des informations sur la façon dont Recovery Services identifie une machine virtuelle.  Une fois que vous êtes familiarisé avec le modèle, vous pouvez ignorer cette étape et passer directement à l’[activation de la protection](#enabling-protection-for-the-azure-vm).

Cette opération est une opération *GET*.

```http
GET https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{vaultresourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupProtectableItems?api-version=2016-12-01&$filter=backupManagementType eq 'AzureIaasVM'
```

L’URI *GET* contient tous les paramètres obligatoires. Aucun corps de demande supplémentaire n’est nécessaire.

#### <a name="responses-to-get-operation"></a>Réponses à l’opération de récupération

|Nom  |Type  |Description  |
|---------|---------|---------|
|200 OK     | [WorkloadProtectableItemResourceList](/rest/api/backup/backupprotectableitems/list#workloadprotectableitemresourcelist)        |       OK |

#### <a name="example-responses-to-get-operation"></a>Exemples de réponses à l’opération de récupération

Une fois la demande *GET* envoyée, une réponse 200 (OK) est retournée.

```http
HTTP/1.1 200 OK
Pragma: no-cache
X-Content-Type-Options: nosniff
x-ms-request-id: 7c2cf56a-e6be-4345-96df-c27ed849fe36
x-ms-client-request-id: 40c8601a-c217-4c68-87da-01db8dac93dd; 40c8601a-c217-4c68-87da-01db8dac93dd
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-reads: 14979
x-ms-correlation-request-id: 7c2cf56a-e6be-4345-96df-c27ed849fe36
x-ms-routing-request-id: SOUTHINDIA:20180521T071408Z:7c2cf56a-e6be-4345-96df-c27ed849fe36
Cache-Control: no-cache
Date: Mon, 21 May 2018 07:14:08 GMT
Server: Microsoft-IIS/8.0
X-Powered-By: ASP.NET

{
  "value": [
    {
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/IaasVMContainer;iaasvmcontainerv2;testRG;testVM/protectableItems/vm;iaasvmcontainerv2;testRG;testVM",
      "name": "iaasvmcontainerv2;testRG;testVM",
      "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectableItems",
      "properties": {
        "virtualMachineId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
        "virtualMachineVersion": "Compute",
        "resourceGroup": "testRG",
        "backupManagementType": "AzureIaasVM",
        "protectableItemType": "Microsoft.Compute/virtualMachines",
        "friendlyName": "testVM",
        "protectionState": "NotProtected"
      }
    },……………..

```

> [!TIP]
> Le nombre de valeurs dans une réponse *GET* est limité à 200 pour une « page ». Utilisez le champ « nextLink » pour obtenir l’URL de l’ensemble suivant de réponses.

La réponse contient la liste de toutes les machines virtuelles Azure non protégées et chaque `{value}` contient toutes les informations requises par Azure Recovery Services pour configurer la sauvegarde. Pour configurer la sauvegarde, notez le champ `{name}` et le champ `{virtualMachineId}` de la section `{properties}`. Créez deux variables à partir de ces valeurs de champ, comme indiqué ci-dessous.

- containerName = "iaasvmcontainer;"+`{name}`
- protectedItemName = "vm;"+ `{name}`
- `{virtualMachineId}` est utilisé ultérieurement dans [le corps de la demande](#example-request-body)

Dans l’exemple, les valeurs ci-dessus sont traduites en :

- containerName = "iaasvmcontainer;iaasvmcontainerv2;testRG;testVM"
- protectedItemName = "vm;iaasvmcontainerv2;testRG;testVM"

### <a name="enabling-protection-for-the-azure-vm"></a>Activation de la protection pour la machine virtuelle Azure

Une fois la machine virtuelle appropriée « mise en cache » et « identifiée », sélectionnez la stratégie de protection. Pour en savoir plus sur les stratégies existantes dans le coffre, reportez-vous à l’[API lister les stratégies](/rest/api/backup/backuppolicies/list). Sélectionnez ensuite la [stratégie appropriée](/rest/api/backup/protectionpolicies/get) en faisant référence au nom de la stratégie. Pour créer des stratégies, reportez-vous au [tutoriel de création de stratégies](backup-azure-arm-userestapi-createorupdatepolicy.md). La stratégie « DefaultPolicy » est sélectionnée dans l’exemple ci-dessous.

L’activation de la protection est une opération *PUT* asynchrone qui crée un « élément protégé ».

```http
https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{vaultresourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}?api-version=2019-05-13
```

`{containerName}` et `{protectedItemName}` sont tels que créés ci-dessus. `{fabricName}` est « Azure ». Dans notre exemple, cela se traduit par :

```http
PUT https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;iaasvmcontainerv2;testRG;testVM?api-version=2019-05-13
```

#### <a name="create-the-request-body"></a>Créer le corps de la demande

Pour créer un élément protégé, voici les composants du corps de la demande.

|Nom  |Type  |Description  |
|---------|---------|---------|
|properties     | AzureIaaSVMProtectedItem        |Propriétés de ressource ProtectedItem         |

Pour obtenir la liste complète des définitions du corps de la demande et d’autres détails, reportez-vous au [document sur l’API REST créer un élément protégé](/rest/api/backup/protecteditems/createorupdate#request-body).

##### <a name="example-request-body"></a>Exemple de corps de demande

Le corps de la demande suivant définit les propriétés requises pour créer un élément protégé.

```json
{
  "properties": {
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "policyId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupPolicies/DefaultPolicy"
  }
}
```

`{sourceResourceId}` correspond au `{virtualMachineId}` mentionné ci-dessus à partir de la [réponse de la liste des éléments pouvant être protégés](#example-responses-to-get-operation).

#### <a name="responses-to-create-protected-item-operation"></a>Réponses à l’opération de création d’élément protégé

La création d’un élément protégé est une [opération asynchrone](../azure-resource-manager/management/async-operations.md). ce qui signifie qu’elle crée une autre opération qui doit faire l’objet d’un suivi distinct.

Elle retourne deux réponses : 202 (Accepté) lors de la création d’une autre opération, puis 200 (OK) quand cette opération est terminée.

|Nom  |Type  |Description  |
|---------|---------|---------|
|200 OK     |    [ProtectedItemResource](/rest/api/backup/protecteditemoperationresults/get#protecteditemresource)     |  OK       |
|202 Accepté     |         |     Acceptée    |

##### <a name="example-responses-to-create-protected-item-operation"></a>Exemples de réponses à l’opération de création d’élément protégé

Une fois que vous envoyez la demande *PUT* de création ou de mise à jour d’un élément protégé, la réponse initiale est 202 (Accepté) avec un en-tête d’emplacement ou Azure-async-header.

```http
HTTP/1.1 202 Accepted
Pragma: no-cache
Retry-After: 60
Azure-AsyncOperation: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationsStatus/a0866047-6fc7-4ac3-ba38-fb0ae8aa550f?api-version=2019-05-13
X-Content-Type-Options: nosniff
x-ms-request-id: db785be0-bb20-4598-bc9f-70c9428b170b
x-ms-client-request-id: e1f94eef-9b2d-45c4-85b8-151e12b07d03; e1f94eef-9b2d-45c4-85b8-151e12b07d03
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-writes: 1199
x-ms-correlation-request-id: db785be0-bb20-4598-bc9f-70c9428b170b
x-ms-routing-request-id: SOUTHINDIA:20180521T073907Z:db785be0-bb20-4598-bc9f-70c9428b170b
Cache-Control: no-cache
Date: Mon, 21 May 2018 07:39:06 GMT
Location: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationResults/a0866047-6fc7-4ac3-ba38-fb0ae8aa550f?api-version=2019-05-13
X-Powered-By: ASP.NET
```

Effectuez ensuite le suivi de l’opération qui en résulte en utilisant l’en-tête d’emplacement ou Azure-AsyncOperation avec une commande *GET* simple.

```http
GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationsStatus/a0866047-6fc7-4ac3-ba38-fb0ae8aa550f?api-version=2019-05-13
```

Une fois l’opération terminée, elle retourne 200 (OK) avec le contenu de l’élément protégé dans le corps de la réponse.

```json
{
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM",
  "name": "VM;testRG;testVM",
  "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectedItems",
  "properties": {
    "friendlyName": "testVM",
    "virtualMachineId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "protectionStatus": "Healthy",
    "protectionState": "IRPending",
    "healthStatus": "Passed",
    "lastBackupStatus": "",
    "lastBackupTime": "2001-01-01T00:00:00Z",
    "protectedItemDataId": "17592691116891",
    "extendedInfo": {
      "recoveryPointCount": 0,
      "policyInconsistent": false
    },
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "backupManagementType": "AzureIaasVM",
    "workloadType": "VM",
    "containerName": "iaasvmcontainerv2;testRG;testVM",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "policyId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupPolicies/DefaultPolicy",
    "policyName": "DefaultPolicy"
  }
}
```

Cela confirme que la protection est activée pour la machine virtuelle et que la première sauvegarde est déclenchée conformément à la planification de la stratégie.

### <a name="excluding-disks-in-azure-vm-backup"></a>Exclusion de disques dans la sauvegarde de la machine virtuelle Azure

Sauvegarde Azure offre également un moyen de sauvegarder de manière sélective un sous-ensemble de disques dans une machine virtuelle Azure. Des informations supplémentaires sont fournies [ici](selective-disk-backup-restore.md). Si vous souhaitez sauvegarder de manière sélective quelques disques pendant l’activation de la protection, l’extrait de code suivant doit être le [corps de la demande lors de l’activation de la protection](#example-request-body).

```json
{
"properties": {
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "policyId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupPolicies/DefaultPolicy",
    "extendedProperties":  {
      "diskExclusionProperties":{
          "diskLunList":[0,1],
          "isInclusionList":true
        }
    }
}
}
```

Dans le corps de la demande ci-dessus, la liste des disques à sauvegarder est fournie dans la section Propriétés étendues.

|Propriété  |Valeur  |
|---------|---------|
|diskLunList     | La liste LUN de disques est une liste de *numéros d’unités logiques de disques de données*. **Le disque du système d’exploitation est toujours sauvegardé et n’a pas besoin d’être mentionné**.        |
|IsInclusionList     | Doit être **true** pour que les numéros d’unités logiques soient inclus lors de la sauvegarde. Si la valeur est **false**, les numéros d’unités logiques susmentionnés seront exclus.         |

Par conséquent, si la nécessité est de sauvegarder uniquement le disque du système d’exploitation, _tous_ les disques de données doivent être exclus. Un moyen plus simple consiste à indiquer qu’aucun disque de données ne doit être inclus. Par conséquent, la liste des numéros d’unités logiques de disques est vide et la propriété **IsInclusionList** est **true**. De même, réfléchissez à la manière la plus simple de sélectionner un sous-ensemble : certains disques doivent être toujours exclus ou d’autres doivent toujours être inclus. Choisissez la liste de numéros d’unités logiques et la valeur de la variable booléenne en conséquence.

## <a name="trigger-an-on-demand-backup-for-a-protected-azure-vm"></a>Déclencher une sauvegarde à la demande pour une machine virtuelle Azure protégée

Après configuration d'une machine virtuelle Azure pour la sauvegarde, les sauvegardes sont effectuées conformément à la planification de la stratégie. Vous pouvez attendre la première sauvegarde planifiée ou déclencher une sauvegarde à la demande à tout moment. La conservation des sauvegardes de la demande se distingue de la conservation de la stratégie de sauvegarde et peut être spécifiée à une date/heure précise. Si elle n’est pas indiquée, elle est supposée être de 30 jours à partir du jour du déclenchement de la sauvegarde à la demande.

Le déclenchement d’une sauvegarde à la demande est une opération *POST*.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}/backup?api-version=2016-12-01
```

`{containerName}` et `{protectedItemName}` sont tels que créés [ci-dessus](#responses-to-get-operation). `{fabricName}` est « Azure ». Dans notre exemple, cela se traduit par :

```http
POST https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;iaasvmcontainerv2;testRG;testVM/backup?api-version=2016-12-01
```

### <a name="create-the-request-body-for-on-demand-backup"></a>Créer le corps de la demande pour une sauvegarde à la demande

Pour déclencher une sauvegarde à la demande, voici les composants du corps de la demande.

|Nom  |Type  |Description  |
|---------|---------|---------|
|properties     | [IaaSVMBackupRequest](/rest/api/backup/backups/trigger#iaasvmbackuprequest)        |Propriétés de BackupRequestResource         |

Pour obtenir la liste complète des définitions du corps de la demande et d’autres détails, reportez-vous au [document sur l’API REST déclencher des sauvegardes pour les éléments protégés](/rest/api/backup/backups/trigger#request-body).

#### <a name="example-request-body-for-on-demand-backup"></a>Exemple de corps de la demande pour une sauvegarde à la demande

Le corps de la demande suivant définit les propriétés requises pour déclencher une sauvegarde pour un élément protégé. Si la conservation n’est pas indiquée, elle est de 30 jours à partir du moment du déclenchement du travail de sauvegarde.

```json
{
   "properties": {
    "objectType": "IaasVMBackupRequest",
    "recoveryPointExpiryTimeInUTC": "2018-12-01T02:16:20.3156909Z"
  }
}
```

### <a name="responses-for-on-demand-backup"></a>Réponses à une sauvegarde à la demande

Le déclenchement d’une sauvegarde à la demande est une [opération asynchrone](../azure-resource-manager/management/async-operations.md). ce qui signifie qu’elle crée une autre opération qui doit faire l’objet d’un suivi distinct.

Elle retourne deux réponses : 202 (Accepté) lors de la création d’une autre opération, puis 200 (OK) quand cette opération est terminée.

|Nom  |Type  |Description  |
|---------|---------|---------|
|202 Accepté     |         |     Acceptée    |

#### <a name="example-responses-for-on-demand-backup"></a>Exemples de réponses à une sauvegarde à la demande

Une fois que vous envoyez la demande *POST* pour une sauvegarde à la demande, la réponse initiale est 202 (Accepté) avec un en-tête d’emplacement ou Azure-async-header.

```http
HTTP/1.1 202 Accepted
Pragma: no-cache
Retry-After: 60
Azure-AsyncOperation: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testVaultRG;testVM/protectedItems/vm;testRG;testVM/operationsStatus/b8daecaa-f8f5-44ed-9f18-491a9e9ba01f?api-version=2019-05-13
X-Content-Type-Options: nosniff
x-ms-request-id: 7885ca75-c7c6-43fb-a38c-c0cc437d8810
x-ms-client-request-id: 7df8e874-1d66-4f81-8e91-da2fe054811d; 7df8e874-1d66-4f81-8e91-da2fe054811d
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-writes: 1199
x-ms-correlation-request-id: 7885ca75-c7c6-43fb-a38c-c0cc437d8810
x-ms-routing-request-id: SOUTHINDIA:20180521T083541Z:7885ca75-c7c6-43fb-a38c-c0cc437d8810
Cache-Control: no-cache
Date: Mon, 21 May 2018 08:35:41 GMT
Location: https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testVaultRG;testVM/protectedItems/vm;testRG;testVM/operationResults/b8daecaa-f8f5-44ed-9f18-491a9e9ba01f?api-version=2019-05-13
X-Powered-By: ASP.NET
```

Effectuez ensuite le suivi de l’opération qui en résulte en utilisant l’en-tête d’emplacement ou Azure-AsyncOperation avec une commande *GET* simple.

```http
GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;testRG;testVM/operationsStatus/a0866047-6fc7-4ac3-ba38-fb0ae8aa550f?api-version=2019-05-13
```

Une fois l’opération terminée, elle retourne 200 (OK) avec l’ID du travail de sauvegarde obtenu dans le corps de la réponse.

```json
HTTP/1.1 200 OK
Pragma: no-cache
X-Content-Type-Options: nosniff
x-ms-request-id: a8b13524-2c95-445f-b107-920806f385c1
x-ms-client-request-id: 5a63209d-3708-4e69-a75f-9499f4c8977c; 5a63209d-3708-4e69-a75f-9499f4c8977c
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-reads: 14995
x-ms-correlation-request-id: a8b13524-2c95-445f-b107-920806f385c1
x-ms-routing-request-id: SOUTHINDIA:20180521T083723Z:a8b13524-2c95-445f-b107-920806f385c1
Cache-Control: no-cache
Date: Mon, 21 May 2018 08:37:22 GMT
Server: Microsoft-IIS/8.0
X-Powered-By: ASP.NET

{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "00000000-0000-0000-0000-000000000000",
  "status": "Succeeded",
  "startTime": "2018-05-21T08:35:40.9488967Z",
  "endTime": "2018-05-21T08:35:40.9488967Z",
  "properties": {
    "objectType": "OperationStatusJobExtendedInfo",
    "jobId": "7ddead57-bcb9-4269-ac31-6a1b57588700"
  }
}
```

Comme le travail de sauvegarde est une opération longue, il doit être suivi comme expliqué dans le [document surveiller les travaux avec une API REST](backup-azure-arm-userestapi-managejobs.md#tracking-the-job).

## <a name="modify-the-backup-configuration-for-a-protected-azure-vm"></a>Modifier la configuration de sauvegarde pour une machine virtuelle Azure protégée

### <a name="changing-the-policy-of-protection"></a>Changement de la stratégie de protection

Pour changer la stratégie avec laquelle la machine virtuelle est protégée, vous pouvez utiliser le même format que pour l’[activation de la protection](#enabling-protection-for-the-azure-vm). Il vous suffit d’indiquer le nouvel ID de stratégie dans [le corps de la demande](#example-request-body) et d’envoyer la demande. Par exemple : Pour remplacer la stratégie « DefaultPolicy » de testVM par « ProdPolicy », indiquez l’ID « ProdPolicy » dans le corps de la requête.

```json
{
  "properties": {
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "policyId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/microsoft.recoveryservices/vaults/testVault/backupPolicies/ProdPolicy"
  }
}
```

La réponse suivra le même format que celui mentionné [pour l’activation de la protection](#responses-to-create-protected-item-operation)

#### <a name="excluding-disks-during-azure-vm-protection"></a>Exclusion de disques pendant la protection de la machine virtuelle Azure

Si la machine virtuelle Azure est déjà sauvegardée, vous pouvez spécifier la liste des disques à sauvegarder ou à exclure en modifiant la stratégie de protection. Préparez simplement la demande dans le même format que [l’exclusion de disques pendant l’activation de la protection](#excluding-disks-in-azure-vm-backup).

> [!IMPORTANT]
> Le corps de la demande ci-dessus est toujours la copie finale des disques de données à exclure ou à inclure. Cela n’*ajoute* rien à la configuration précédente. Par exemple : si vous mettez d’abord à jour la protection avec « Exclure le disque de données 1 », puis répétez l’opération avec « Exclure le disque de données 2 », *seul le disque de données 2 est exclu* dans les sauvegardes suivantes, et le disque de données 1 sera inclus. C’est toujours la dernière liste qui sera incluse/exclue dans les sauvegardes ultérieures.

Pour récupérer la liste actuelle des disques qui sont exclus ou inclus, récupérez les informations relatives aux éléments protégés comme indiqué [ici](/rest/api/backup/protecteditems/get). La réponse fournira la liste des numéros d’unités logiques de disques de données et indiquera s’ils sont inclus ou exclus.

### <a name="stop-protection-but-retain-existing-data"></a>Arrêter la protection tout en conservant les données existantes

Pour supprimer la protection sur une machine virtuelle protégée tout en conservant les données déjà sauvegardées, supprimez la stratégie dans le corps de la demande et envoyez la demande. Une fois l’association à la stratégie supprimée, les sauvegardes ne sont plus déclenchées et aucun point de récupération n’est créé.

```http
{
  "properties": {
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "policyId": ""
  }
}
```

La réponse suivra le même format que celui mentionné [pour le déclenchement d’une sauvegarde à la demande](#example-responses-for-on-demand-backup). Le travail résultant doit être suivi, comme expliqué dans le [document surveiller les travaux avec une API REST](backup-azure-arm-userestapi-managejobs.md#tracking-the-job).

### <a name="stop-protection-and-delete-data"></a>Arrêter la protection et supprimer les données

Pour supprimer la protection sur une machine virtuelle protégée ainsi que les données de sauvegarde, effectuez une opération de suppression comme indiqué [ici](/rest/api/backup/protecteditems/delete).

L’arrêt de la protection avec suppression des données est une opération *DELETE*.

```http
DELETE https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}?api-version=2019-05-13
```

`{containerName}` et `{protectedItemName}` sont tels que créés [ci-dessus](#responses-to-get-operation). `{fabricName}` correspond à « Azure ». Dans notre exemple, cela se traduit par :

```http
DELETE https://management.azure.com//Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testVaultRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/iaasvmcontainer;iaasvmcontainerv2;testRG;testVM/protectedItems/vm;iaasvmcontainerv2;testRG;testVM?api-version=2019-05-13
```

#### <a name="responses-for-delete-protection"></a>Réponses à la suppression de la protection

L’opération *DELETE* sur la protection est une [opération asynchrone](../azure-resource-manager/management/async-operations.md). ce qui signifie qu’elle crée une autre opération qui doit faire l’objet d’un suivi distinct.

Elle retourne deux réponses : 202 (Accepté) lors de la création d’une autre opération, puis 204 (Pas de contenu) quand cette opération est terminée.

|Nom  |Type  |Description  |
|---------|---------|---------|
|204 Pas de contenu     |         |  Pas de contenu       |
|202 Accepté     |         |     Acceptée    |

> [!IMPORTANT]
> Pour vous protéger contre les scénarios de suppression accidentelle, une [fonctionnalité de suppression réversible est disponible](use-restapi-update-vault-properties.md#soft-delete-state) pour le coffre Recovery Services. Si l’état de suppression réversible du coffre est activé, l’opération de suppression ne supprime pas immédiatement les données. Elles seront conservées pendant 14 jours, puis purgées définitivement. Vous n’êtes pas facturé pour le stockage pendant cette période de 14 jours. Pour annuler l’opération de suppression, reportez-vous à la section [Annuler la suppression](#undo-the-deletion).

### <a name="undo-the-deletion"></a>Annuler la suppression

L’annulation de la suppression accidentelle est semblable à la création de l’élément de sauvegarde. Une fois la suppression annulée, l’élément est conservé, mais aucune sauvegarde ultérieure n’est déclenchée.

L’annulation de la suppression est une opération *PUT* qui est très similaire à [la modification de la stratégie](#changing-the-policy-of-protection) et/ou [l’activation de la protection](#enabling-protection-for-the-azure-vm). Il vous suffit de fournir l’intention d’annuler la suppression avec la variable *isRehydrate* dans [le corps de la demande](#example-request-body) et de soumettre la demande. Par exemple : Pour annuler la suppression de testVM, le corps de la demande suivant doit être utilisé.

```http
{
  "properties": {
    "protectedItemType": "Microsoft.Compute/virtualMachines",
    "protectionState": "ProtectionStopped",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Compute/virtualMachines/testVM",
    "isRehydrate": true
  }
}
```

La réponse suivra le même format que celui mentionné [pour le déclenchement d’une sauvegarde à la demande](#example-responses-for-on-demand-backup). Le travail résultant doit être suivi, comme expliqué dans le [document surveiller les travaux avec une API REST](backup-azure-arm-userestapi-managejobs.md#tracking-the-job).

## <a name="next-steps"></a>Étapes suivantes

[Restaurer des données à partir d’une sauvegarde de machine virtuelle Azure](backup-azure-arm-userestapi-restoreazurevms.md).

Pour plus d’informations sur les API REST de la Sauvegarde Azure, voir les documents suivants :

- [API REST du fournisseur Azure Recovery Services](/rest/api/recoveryservices/)
- [Bien démarrer avec l’API REST Azure](/rest/api/azure/)