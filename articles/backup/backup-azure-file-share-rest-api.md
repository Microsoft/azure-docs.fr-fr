---
title: Sauvegarder des partages de fichiers Azure avec l’API REST
description: Apprenez à utiliser l’API REST pour sauvegarder des partages de fichiers Azure dans le coffre Recovery Services
ms.topic: conceptual
ms.date: 02/16/2020
ms.openlocfilehash: 8d2d8ed88da133986540a293185c8e37000ab87b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88824863"
---
# <a name="backup-azure-file-share-using-azure-backup-via-rest-api"></a>Sauvegarder un partage de fichiers Azure à l’aide de Sauvegarde Azure via l’API REST

Cet article explique comment sauvegarder un partage de fichiers Azure à l’aide de Sauvegarde Azure via l’API REST.

Cet article suppose que vous avez déjà créé un coffre Recovery Services et une stratégie de configuration de la sauvegarde pour votre partage de fichiers. Si ce n’est pas le cas, reportez-vous aux tutoriels relatifs à l’API REST [Créer un coffre](./backup-azure-arm-userestapi-createorupdatevault.md) et [Créer une stratégie](./backup-azure-arm-userestapi-createorupdatepolicy.md) sur la création de coffres et de stratégies.

Pour cet article, nous allons utiliser les ressources suivantes :

- **RecoveryServicesVault** : *azurefilesvault*

- **Stratégie :** *schedule1*

- **Groupe de ressources** : *azurefiles*

- **Compte de stockage** : *testvault2*

- **Partage de fichiers** : *testshare*

## <a name="configure-backup-for-an-unprotected-azure-file-share-using-rest-api"></a>Configurer la sauvegarde pour un partage de fichiers Azure non protégé à l’aide de l’API REST

### <a name="discover-storage-accounts-with-unprotected-azure-file-shares"></a>Découvrir des comptes de stockage avec des partages de fichiers Azure non protégés

Le coffre doit découvrir tous les comptes de stockage Azure dans l’abonnement avec des partages de fichiers qui peuvent être sauvegardés dans le coffre Recovery Services. Cette action est déclenchée à l’aide de l’[opération d’actualisation](/rest/api/backup/protectioncontainers/refresh). Il s’agit d’une opération *POST* asynchrone qui garantit que le coffre obtient la liste la plus récente de tous les partages de fichiers Azure non protégés dans l’abonnement actuel et les « met en cache ». Une fois que le partage de fichiers est mis en cache, Recovery Services peut accéder au partage de fichiers et le protéger.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{vaultresourceGroupname}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/refreshContainers?api-version=2016-12-01&$filter={$filter}
```

L’URI POST contient les paramètres `{subscriptionId}`, `{vaultName}`, `{vaultresourceGroupName}` et `{fabricName}`. Dans notre exemple, la valeur des différents paramètres est la suivante :

- `{fabricName}` est *Azure*

- `{vaultName}` est *azurefilesvault*

- `{vaultresourceGroupName}` est *azurefiles*

- $filter=backupManagementType eq 'AzureStorage'

Étant donné que tous les paramètres nécessaires sont fournis dans l’URI, un corps de demande distinct est inutile.

```http
POST https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/refreshContainers?api-version=2016-12-01&$filter=backupManagementType eq 'AzureStorage'
```

#### <a name="responses-to-the-refresh-operation"></a>Réponses à l’opération d’actualisation

L’opération « Actualiser » est une [opération asynchrone](../azure-resource-manager/management/async-operations.md). ce qui signifie qu’elle crée une autre opération qui doit faire l’objet d’un suivi distinct.

Elle retourne deux réponses : 202 (Accepté) lors de la création d’une autre opération, et 200 (OK) quand cette opération est terminée.

##### <a name="example-responses-to-the-refresh-operation"></a>Exemples de réponses à l’opération d’actualisation

Une fois la demande *POST* envoyée, une réponse 202 (Accepté) est retournée.

```http
HTTP/1.1 202 Accepted
'Pragma': 'no-cache'
'Expires': '-1'
'Location': ‘https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/ResourceGroups
/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/operationResults/
cca47745-12d2-42f9-b3a4-75335f18fdf6?api-version=2016-12-01’
'Retry-After': '60'
'X-Content-Type-Options': 'nosniff'
'x-ms-request-id': '6cc12ceb-90a2-430d-a1ec-9b6b6fdea92b'
'x-ms-client-request-id': ‘3da383a5-d66d-4b7c-982a-bc8d94798d61,3da383a5-d66d-4b7c-982a-bc8d94798d61’
'Strict-Transport-Security': 'max-age=31536000; includeSubDomains'
'X-Powered-By': 'ASP.NET'
'x-ms-ratelimit-remaining-subscription-reads': '11996'
'x-ms-correlation-request-id': '6cc12ceb-90a2-430d-a1ec-9b6b6fdea92b'
'x-ms-routing-request-id': CENTRALUSEUAP:20200203T091326Z:6cc12ceb-90a2-430d-a1ec-9b6b6fdea92b'
'Date': 'Mon, 03 Feb 2020 09:13:25 GMT'
```

Effectuez le suivi de l’opération qui en résulte à l’aide de l’en-tête « Location » avec une commande *GET* simple

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/operationResults/cca47745-12d2-42f9-b3a4-75335f18fdf6?api-version=2016-12-01
```

Une fois tous les comptes de stockage Azure découverts, la commande GET retourne une réponse 200 (Pas de contenu). Le coffre est maintenant en mesure de découvrir tout compte de stockage avec des partages de fichiers pouvant être sauvegardés au sein de l’abonnement.

```http
HTTP/1.1 200 NoContent
Cache-Control  : no-cache
Pragma   : no-cache
X-Content-Type-Options  : nosniff
x-ms-request-id    : d9bdb266-8349-4dbd-9688-de52f07648b2
x-ms-client-request-id  : 3da383a5-d66d-4b7c-982a-bc8d94798d61,3da383a5-d66d-4b7c-982a-bc8d94798d61
Strict-Transport-Security  : max-age=31536000; includeSubDomains
X-Powered-By    : ASP.NET
x-ms-ratelimit-remaining-subscription-reads: 11933
x-ms-correlation-request-id   : d9bdb266-8349-4dbd-9688-de52f07648b2
x-ms-routing-request-id  : CENTRALUSEUAP:20200127T105304Z:d9bdb266-8349-4dbd-9688-de52f07648b2
Date   : Mon, 27 Jan 2020 10:53:04 GMT
```

### <a name="get-list-of-storage-accounts-with-file-shares-that-can-be-backed-up-with-recovery-services-vault"></a>Obtenir la liste des comptes de stockage avec des partages de fichiers qui peuvent être sauvegardés l’aide du coffre Recovery Services

Pour confirmer qu’une « mise en cache » a bien été réalisée, répertoriez tous les comptes de stockage Azure dans l’abonnement avec des partages de fichiers qui peuvent être sauvegardés à l’aide du coffre Recovery Services. Recherchez ensuite le compte de stockage souhaité dans la réponse. Pour ce faire, utilisez l’opération [GET ProtectableContainers](/rest/api/backup/protectablecontainers/list).

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectableContainers?api-version=2016-12-01&$filter=backupManagementType eq 'AzureStorage'
```

L’URI *GET* contient tous les paramètres obligatoires. Aucun corps de demande supplémentaire n’est nécessaire.

Exemple de corps de réponse :

```json
{

  "value": [

    {

      "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers
 /Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/

protectableContainers/StorageContainer;Storage;AzureFiles;testvault2",

      "name": "StorageContainer;Storage;AzureFiles;testvault2",

      "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectableContainers",

      "properties": {

        "friendlyName": "testvault2",

        "backupManagementType": "AzureStorage",

        "protectableContainerType": "StorageContainer",

        "healthStatus": "Healthy",

        "containerId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/
 AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2"

      }

    }

  ]

}
```

Étant donné que nous pouvons localiser le compte de stockage *testvault2* dans le corps de la réponse grâce au nom convivial, l’opération d’actualisation effectuée ci-dessus a réussi. Le coffre Recovery Services peut désormais découvrir correctement les comptes de stockage ayant des partages de fichiers non protégés dans le même abonnement.

### <a name="register-storage-account-with-recovery-services-vault"></a>Inscrire un compte de stockage auprès du coffre Recovery Services

Cette étape est nécessaire uniquement si vous n’avez pas inscrit le compte de stockage dans le coffre précédemment. Vous pouvez inscrire le coffre à l’aide de l’opération [ProtectionContainers-Register](/rest/api/backup/protectioncontainers/register).

```http
PUT https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}?api-version=2016-12-01
```

Définissez les variables pour l’URI comme suit :

- {resourceGroupName} : *azurefiles*
- {fabricName} : *Azure*
- {vaultName} : *azurefilesvault*
- {containerName} : il s’agit de l’attribut de nom dans le corps de la réponse de l’opération GET ProtectableContainers.
   Dans notre exemple, il s’agit de *StorageContainer;Storage;AzureFiles;testvault2*

>[!NOTE]
> Prenez toujours l’attribut de nom dans la réponse et renseignez-le dans cette requête. Ne codez pas en dur ni ne créez le format container-name. Si vous le créez ou le codez en dur, l’appel d’API échoue si le format container-name change à l’avenir.

<br>

```http
PUT https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2?api-version=2016-12-01
```

Le corps de demande de création est le suivant :

```json
{

 "properties": {


  "containerType": "StorageContainer",


  "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",


  "resourceGroup": "AzureFiles",


  "friendlyName": "testvault2",


  "backupManagementType": "AzureStorage"


 }
```

Pour obtenir la liste complète des définitions du corps de la demande et d’autres détails, reportez-vous à [ProtectionContainers-Register](/rest/api/backup/protectioncontainers/register#azurestoragecontainer).

Il s’agit d’une opération asynchrone qui retourne deux réponses : « 202 Accepté » lorsque l’opération est acceptée et « 200 OK » lorsque l’opération est terminée.  Pour suivre l’état de l’opération, utilisez l’en-tête d’emplacement pour récupérer l’état le plus récent de l’opération.

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/operationresults/1a3c8ee7-e0e5-43ed-b8b3-73cc992b6db9?api-version=2016-12-01
```

Exemple de corps de réponse lorsque l’opération est terminée :

```json
{
    "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/
protectionContainers/StorageContainer;Storage;AzureFiles;testvault2",
    "name": "StorageContainer;Storage;AzureFiles;testvault2",
    "properties": {
        "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",
        "protectedItemCount": 0,
        "friendlyName": "testvault2",
        "backupManagementType": "AzureStorage",
        "registrationStatus": "Registered",
        "healthStatus": "Healthy",
        "containerType": "StorageContainer",
        "protectableObjectType": "StorageContainer"
    }
}
```

Vous pouvez vérifier si l’inscription a réussi à partir de la valeur du paramètre *registrationstatus* dans le corps de la réponse. Dans notre cas, elle indique l’état enregistré pour *testvault2*, ce qui signifie que l’opération d’inscription a réussi.

### <a name="inquire-all-unprotected-files-shares-under-a-storage-account"></a>Rechercher tous les partages de fichiers non protégés sous un compte de stockage

Vous pouvez vous renseigner sur les éléments protégeables dans un compte de stockage à l’aide de l’opération [Protection Containers-Inquire](/rest/api/backup/protectioncontainers/inquire). Il s’agit d’une opération asynchrone et les résultats doivent être suivis à l’aide de l’en-tête d’emplacement.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/inquire?api-version=2016-12-01
```

Définissez les variables pour l’URI ci-dessus comme suit :

- {vaultName} : *azurefilesvault*
- {fabricName} : *Azure*
- {containerName} : fait référence à l’attribut de nom dans le corps de la réponse de l’opération GET ProtectableContainers. Dans notre exemple, il s’agit de *StorageContainer;Storage;AzureFiles;testvault2*

```http
https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/inquire?api-version=2016-12-01
```

Une fois la requête réussie, elle retourne le code d’état « OK »

```http
Cache-Control : no-cache
Pragma   : no-cache
X-Content-Type-Options: nosniff
x-ms-request-id  : 68727f1e-b8cf-4bf1-bf92-8e03a9d96c46
x-ms-client-request-id  : 3da383a5-d66d-4b7c-982a-bc8d94798d61,3da383a5-d66d-4b7c-982a-bc8d94798d61
Strict-Transport-Security: max-age=31536000; includeSubDomains
Server  : Microsoft-IIS/10.0
X-Powered-B : ASP.NET
x-ms-ratelimit-remaining-subscription-reads: 11932
x-ms-correlation-request-id  : 68727f1e-b8cf-4bf1-bf92-8e03a9d96c46
x-ms-routing-request-id   : CENTRALUSEUAP:20200127T105305Z:68727f1e-b8cf-4bf1-bf92-8e03a9d96c46
Date  : Mon, 27 Jan 2020 10:53:05 GMT
```

### <a name="select-the-file-share-you-want-to-back-up"></a>Sélectionner le partage de fichiers à sauvegarder

Vous pouvez répertorier tous les éléments protégeables sous l’abonnement et rechercher le partage de fichiers à sauvegarder à l’aide de l’opération [GET backupprotectableItems](/rest/api/backup/backupprotectableitems/list).

```http
GET https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupProtectableItems?api-version=2016-12-01&$filter={$filter}
```

Construisez l’URI comme suit :

- {vaultName} : *azurefilesvault*
- {$filter} : *backupManagementType eq 'AzureStorage'*

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupProtectableItems?$filter=backupManagementType eq 'AzureStorage'&api-version=2016-12-01
```

Exemple de réponse :

```json
Status Code:200

{
    "value": [
        {
            "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/storagecontainer;storage;azurefiles;afaccount1/protectableItems/azurefileshare;azurefiles1",
            "name": "azurefileshare;azurefiles1",
            "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectableItems",
            "properties": {
                "parentContainerFabricId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/afaccount1",
                "parentContainerFriendlyName": "afaccount1",
                "azureFileShareType": "XSMB",
                "backupManagementType": "AzureStorage",
                "workloadType": "AzureFileShare",
                "protectableItemType": "AzureFileShare",
                "friendlyName": "azurefiles1",
                "protectionState": "NotProtected"
            }
        },
        {
            "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/storagecontainer;storage;azurefiles;afsaccount/protectableItems/azurefileshare;afsresource",
            "name": "azurefileshare;afsresource",
            "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectableItems",
            "properties": {
                "parentContainerFabricId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/afsaccount",
                "parentContainerFriendlyName": "afsaccount",
                "azureFileShareType": "XSMB",
                "backupManagementType": "AzureStorage",
                "workloadType": "AzureFileShare",
                "protectableItemType": "AzureFileShare",
                "friendlyName": "afsresource",
                "protectionState": "NotProtected"
            }
        },
        {
            "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/storagecontainer;storage;azurefiles;testvault2/protectableItems/azurefileshare;testshare",
            "name": "azurefileshare;testshare",
            "type": "Microsoft.RecoveryServices/vaults/backupFabrics/protectionContainers/protectableItems",
            "properties": {
                "parentContainerFabricId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",
                "parentContainerFriendlyName": "testvault2",
                "azureFileShareType": "XSMB",
                "backupManagementType": "AzureStorage",
                "workloadType": "AzureFileShare",
                "protectableItemType": "AzureFileShare",
                "friendlyName": "testshare",
                "protectionState": "NotProtected"
            }
        }
    ]
}
```

La réponse contient la liste de tous les partages de fichiers non protégés et toutes les informations requises par Azure Recovery Services pour configurer la sauvegarde.

### <a name="enable-backup-for-the-file-share"></a>Activer la sauvegarde pour le partage de fichiers

Une fois que le partage de fichiers approprié est identifié par le nom convivial, sélectionnez la stratégie pour le protéger. Pour en savoir plus sur les stratégies existantes dans le coffre, reportez-vous à l’[API de liste des stratégies](/rest/api/backup/backuppolicies/list). Sélectionnez ensuite la [stratégie appropriée](/rest/api/backup/protectionpolicies/get) en faisant référence au nom de la stratégie. Pour créer des stratégies, reportez-vous au [tutoriel de création de stratégies](./backup-azure-arm-userestapi-createorupdatepolicy.md).

L’activation de la protection est une opération *PUT* asynchrone qui crée un « élément protégé ».

```http
PUT https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{vaultresourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}?api-version=2019-05-13
```

Définissez les variables **containername** et **protecteditemname** à l’aide de l’attribut ID dans le corps de la réponse de l’opération GET backupprotectableitems.

Dans notre exemple, l’ID du partage de fichiers que vous souhaitez protéger est le suivant :

```output
"/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/storagecontainer;storage;azurefiles;testvault2/protectableItems/azurefileshare;testshare
```

- {containername} : *storagecontainer;storage;azurefiles;testvault2*
- {protectedItemName} : *azurefileshare;testshare*

Ou bien, vous pouvez faire référence à l’attribut **name** du conteneur de protection et aux réponses d’éléments protégeables.

>[!NOTE]
>Prenez toujours l’attribut de nom dans la réponse et renseignez-le dans cette requête. Ne codez pas en dur ni ne créez le format container-name ou celui du nom de l’élément protégé. Si vous le créez ou le codez en dur, l’appel d’API échoue si le format container-name ou celui du nom de l’élément protégé change à l’avenir.

<br>

```http
PUT https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/protectedItems/azurefileshare;testshare?api-version=2016-12-01
```

Créez un corps de demande :

Le corps de la demande suivant définit les propriétés requises pour créer un élément protégé.

```json
{
  "properties": {
    "protectedItemType": "AzureFileShareProtectedItem",
    "sourceResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/AzureFiles/providers/Microsoft.Storage/storageAccounts/testvault2",
    "policyId": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupPolicies/schedule1"
  }
}
```

Le **sourceResourceId** est le **parentcontainerFabricID** en réponse à GET backupprotectableItems.

Exemple de réponse

La création d’un élément protégé est une opération asynchrone qui crée une autre opération devant faire l’objet d’un suivi. Elle retourne deux réponses : 202 (Accepté) lors de la création d’une autre opération, et 200 (OK) quand cette opération est terminée.

Une fois que vous envoyez la requête *PUT* de création ou de mise à jour d’un élément protégé, la réponse initiale est 202 (Accepté) avec un en-tête d’emplacement.

```http
HTTP/1.1 202 Accepted
Cache-Control  : no-cache
Pragma  : no-cache
Location : https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/protectedItems/azurefileshare;testshare/operationResults/c3a52d1d-0853-4211-8141-477c65740264?api-version=2016-12-01
Retry-Afte  : 60
Azure-AsyncOperation  : https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;Storage;AzureFiles;testvault2/protectedItems/azurefileshare;testshare/operationResults/c3a52d1d-0853-4211-8141-477c65740264?api-version=2016-12-01
X-Content-Type-Options : nosniff
x-ms-request-id : b55527fa-f473-4f09-b169-9cc3a7a39065
x-ms-client-request-id: 3da383a5-d66d-4b7c-982a-bc8d94798d61,3da383a5-d66d-4b7c-982a-bc8d94798d61
Strict-Transport-Security : max-age=31536000; includeSubDomains
X-Powered-By  : ASP.NET
x-ms-ratelimit-remaining-subscription-writes: 1198
x-ms-correlation-request-id : b55527fa-f473-4f09-b169-9cc3a7a39065
x-ms-routing-request-id  : CENTRALUSEUAP:20200127T105412Z:b55527fa-f473-4f09-b169-9cc3a7a39065
Date : Mon, 27 Jan 2020 10:54:12 GMT
```

Effectuez ensuite le suivi de l’opération qui en résulte en utilisant l’en-tête d’emplacement ou l’en-tête Azure-AsyncOperation avec une commande *GET*.

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupOperations/c3a52d1d-0853-4211-8141-477c65740264?api-version=2016-12-01
```

Une fois l’opération terminée, elle retourne 200 (OK) avec le contenu de l’élément protégé dans le corps de la réponse.

Exemple de corps de réponse :

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

Cela confirme que la protection est activée pour le partage de fichiers et que la première sauvegarde est déclenchée conformément à la planification de la stratégie.

## <a name="trigger-an-on-demand-backup-for-file-share"></a>Déclencher une sauvegarde à la demande des partages de fichiers

Une fois qu’un partage de fichiers Azure est configuré pour la sauvegarde, les sauvegardes s’exécutent conformément à la planification de la stratégie. Vous pouvez attendre la première sauvegarde planifiée ou déclencher une sauvegarde à la demande à tout moment.

Le déclenchement d’une sauvegarde à la demande est une opération POST.

```http
POST https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupFabrics/{fabricName}/protectionContainers/{containerName}/protectedItems/{protectedItemName}/backup?api-version=2016-12-01
```

{containerName} et {protectedItemName} sont construits comme indiqué ci-dessus lors de l’activation de la sauvegarde. Dans notre exemple, cela se traduit par :

```http
POST https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;testvault2/protectedItems/AzureFileShare;testshare/backup?api-version=2017-07-01
```

### <a name="create-request-body"></a>Créer un corps de demande

Pour déclencher une sauvegarde à la demande, voici les composants du corps de la demande.

| Nom       | Type                       | Description                       |
| ---------- | -------------------------- | --------------------------------- |
| Propriétés | AzurefilesharebackupReques | Propriétés BackupRequestResource |

Pour obtenir la liste complète des définitions du corps de la demande et d’autres détails, reportez-vous au [document sur l’API REST déclencher des sauvegardes pour les éléments protégés](/rest/api/backup/backups/trigger#request-body).

Exemple de corps de la demande

```json
{

  "properties":{

   "objectType":"AzureFileShareBackupRequest",
    "recoveryPointExpiryTimeInUTC":"2020-03-07T18:29:59.000Z"
}

}
```

### <a name="responses-to-the-on-demand-backup-operation"></a>Réponses à l’opération de sauvegarde à la demande

Le déclenchement d’une sauvegarde à la demande est une [opération asynchrone](../azure-resource-manager/management/async-operations.md). ce qui signifie qu’elle crée une autre opération qui doit faire l’objet d’un suivi distinct.

Elle retourne deux réponses : 202 (Accepté) lors de la création d’une autre opération, et 200 (OK) quand cette opération est terminée.

### <a name="example-responses-to-the-on-demand-backup-operation"></a>Exemples de réponses à l’opération de sauvegarde à la demande

Une fois que vous envoyez la demande *POST* pour une sauvegarde à la demande, la réponse initiale est 202 (Accepté) avec un en-tête d’emplacement ou Azure-async-header.

```http
'Cache-Control': 'no-cache'
'Pragma': 'no-cache'
'Expires': '-1'
'Location': https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;testvault2/protectedItems/AzureFileShare;testshare/operationResults/dc62d524-427a-4093-968d-e951c0a0726e?api-version=2017-07-01
'Retry-After': '60'
'Azure-AsyncOperation': https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupFabrics/Azure/protectionContainers/StorageContainer;storage;azurefiles;testvault2/protectedItems/AzureFileShare;testshare/operationsStatus/dc62d524-427a-4093-968d-e951c0a0726e?api-version=2017-07-01
'X-Content-Type-Options': 'nosniff'
'x-ms-request-id': '2e03b8d4-66b1-48cf-8094-aa8bff57e8fb'
'x-ms-client-request-id': 'a644712a-4895-11ea-ba57-0a580af42708, a644712a-4895-11ea-ba57-0a580af42708'
'Strict-Transport-Security': 'max-age=31536000; includeSubDomains'
'X-Powered-By': 'ASP.NET'
'x-ms-ratelimit-remaining-subscription-writes': '1199'
'x-ms-correlation-request-id': '2e03b8d4-66b1-48cf-8094-aa8bff57e8fb'
'x-ms-routing-request-id': 'WESTEUROPE:20200206T040339Z:2e03b8d4-66b1-48cf-8094-aa8bff57e8fb'
'Date': 'Thu, 06 Feb 2020 04:03:38 GMT'
'Content-Length': '0'
```

Effectuez ensuite le suivi de l’opération qui en résulte en utilisant l’en-tête d’emplacement ou l’en-tête Azure-AsyncOperation avec une commande *GET*.

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/azurefiles/providers/Microsoft.RecoveryServices/vaults/azurefilesvault/backupOperations/dc62d524-427a-4093-968d-e951c0a0726e?api-version=2016-12-01
```

Une fois l’opération terminée, elle retourne 200 (OK) avec l’ID du travail de sauvegarde obtenu dans le corps de la réponse.

#### <a name="sample-response-body"></a>Exemple de corps de la réponse

```json
{
    "id": "dc62d524-427a-4093-968d-e951c0a0726e",
    "name": "dc62d524-427a-4093-968d-e951c0a0726e",
    "status": "Succeeded",
    "startTime": "2020-02-06T11:06:02.1327954Z",
    "endTime": "2020-02-06T11:06:02.1327954Z",
    "properties": {
        "objectType": "OperationStatusJobExtendedInfo",
        "jobId": "39282261-cb52-43f5-9dd0-ffaf66beeaef"
    }
}
```

Comme le travail de sauvegarde est une opération longue, il doit être suivi comme expliqué dans le [document surveiller les travaux avec une API REST](./backup-azure-arm-userestapi-managejobs.md#tracking-the-job).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [restaurer des partages de fichiers Azure à l’aide de l’API REST](restore-azure-file-share-rest-api.md).
