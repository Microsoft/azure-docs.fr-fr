---
title: État des opérations asynchrones
description: Décrit comment effectuer le suivi des opérations asynchrones dans Azure. Elle affiche les valeurs que vous utilisez pour obtenir l’état d’une opération de longue durée.
ms.topic: conceptual
ms.date: 08/21/2020
ms.custom: seodec18
ms.openlocfilehash: e2c5ba137d5277466cf1b382d2b0b1bc02259f00
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88723450"
---
# <a name="track-asynchronous-azure-operations"></a>Suivre les opérations asynchrones Azure

Certaines opérations REST Azure s’exécutent de façon asynchrone, car elles ne peuvent pas être effectuées rapidement. Cet article explique comment effectuer le suivi de l’état des opérations asynchrones via les valeurs retournées dans la réponse.  

## <a name="status-codes-for-asynchronous-operations"></a>Codes d’état pour les opérations asynchrones

Initialement, une opération asynchrone retourne un code d’état HTTP de type :

* 201 (créé)
* 202 (accepté)

Lorsque l’opération se termine correctement, elle renvoie :

* 200 (OK)
* 204 (Pas de contenu)

Reportez-vous à la [documentation de l’API REST](/rest/api/azure/) pour voir les réponses liées à l’opération que vous exécutez.

Une fois le code de réponse 201 ou 202 obtenu, vous êtes prêt à superviser l’état de l’opération.

## <a name="url-to-monitor-status"></a>URL pour superviser l’état

Il existe deux manières différentes de superviser l’état de l’opération asynchrone. Pour déterminer l’approche correcte, vous devez examiner les valeurs d’en-tête retournées par votre requête d’origine. Tout d’abord, recherchez :

* `Azure-AsyncOperation`- URL pour vérifier l’état de l’opération en cours. Si votre opération retourne cette valeur, utilisez-la pour suivre l’état de l’opération.
* `Retry-After`- Le nombre de secondes à attendre avant de vérifier l’état de l’opération asynchrone.

Si `Azure-AsyncOperation` n’est pas l’une des valeurs d’en-tête, recherchez :

* `Location`- URL pour déterminer si une opération est terminée. Utilisez cette valeur uniquement quand la valeur Azure-AsyncOperation n’est pas retournée.
* `Retry-After`- Le nombre de secondes à attendre avant de vérifier l’état de l’opération asynchrone.

## <a name="azure-asyncoperation-request-and-response"></a>Demande et réponse Azure-AsyncOperation

Si vous avez une URL de la valeur d’en-tête `Azure-AsyncOperation`, envoyez une requête GET à cette URL. Utilisez la valeur de `Retry-After` pour planifier la fréquence de vérification de l’état. Vous obtiendrez un objet de réponse qui indique l’état de l’opération. Une réponse différente est retournée lors de la vérification de l’état de l’opération avec l’URL de `Location`. Pour plus d’informations sur la réponse à partir d’une URL d’emplacement, consultez [Créer le compte de stockage (202 avec Location et Retry-After)](#create-storage-account-202-with-location-and-retry-after).

Les propriétés de la réponse peuvent varier, mais incluent toujours l’état de l’opération asynchrone.

```json
{
    "status": "{status-value}"
}
```

L’exemple suivant montre d’autres valeurs qui peuvent être retournées par l’opération :

```json
{
    "id": "{resource path from GET operation}",
    "name": "{operation-id}",
    "status" : "Succeeded | Failed | Canceled | {resource provider values}",
    "startTime": "2017-01-06T20:56:36.002812+00:00",
    "endTime": "2017-01-06T20:56:56.002812+00:00",
    "percentComplete": {double between 0 and 100 },
    "properties": {
        /* Specific resource provider values for successful operations */
    },
    "error" : {
        "code": "{error code}",  
        "message": "{error description}"
    }
}
```

L’objet d’erreur est retourné lorsque l’état indique que l'opération a échoué (Failed) ou a été annulée (Canceled). Toutes les autres valeurs sont facultatives. La réponse que vous recevez peut être différente de l’exemple.

## <a name="provisioningstate-values"></a>Valeurs provisioningState

Les opérations qui créent, mettent à jour ou suppriment (PUT, PATCH, DELETE) une ressource retournent généralement une valeur `provisioningState`. Lorsqu’une opération est terminée, une des trois valeurs suivantes est retournée :

* Opération réussie
* Échec
* Opération annulée

Toutes les autres valeurs indiquent que l’opération est en cours d’exécution. Le fournisseur de ressources peut retourner une valeur personnalisée qui indique son état. Par exemple, vous pouvez recevoir l'état d'acceptation **Accepted** lorsque la demande est reçue et en cours d’exécution.

## <a name="example-requests-and-responses"></a>Exemples de demandes et de réponses

### <a name="start-virtual-machine-202-with-azure-asyncoperation"></a>Démarrez la machine virtuelle (202 avec Azure-AsyncOperation)

Cet exemple montre comment déterminer l’état de l’[opération de démarrage pour les machines virtuelles](/rest/api/compute/virtualmachines/start). La demande initiale présente le format suivant :

```HTTP
POST 
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Compute/virtualMachines/{vm-name}/start?api-version=2019-12-01
```

Renvoie le code d’état 202. Les valeurs d’en-tête incluent :

```HTTP
Azure-AsyncOperation : https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Compute/locations/{region}/operations/{operation-id}?api-version=2019-12-01
```

Pour vérifier l’état de l’opération asynchrone, l’envoi d’une autre demande à cette URL.

```HTTP
GET 
https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Compute/locations/{region}/operations/{operation-id}?api-version=2019-12-01
```

Le corps de la réponse contient l’état de l’opération :

```json
{
  "startTime": "2017-01-06T18:58:24.7596323+00:00",
  "status": "InProgress",
  "name": "9a062a88-e463-4697-bef2-fe039df73a02"
}
```

### <a name="deploy-resources-201-with-azure-asyncoperation"></a>Déployer des ressources (201 avec Azure-AsyncOperation)

Cet exemple montre comment déterminer l’état de l’[opération de déploiement pour le déploiement de ressources](/rest/api/resources/deployments/createorupdate) sur Azure. La demande initiale présente le format suivant :

```HTTP
PUT
https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/microsoft.resources/deployments/{deployment-name}?api-version=2020-06-01
```

Il renvoie le code d’état 201. Le corps de la réponse inclut :

```json
"provisioningState":"Accepted",
```

Les valeurs d’en-tête incluent :

```HTTP
Azure-AsyncOperation: https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Resources/deployments/{deployment-name}/operationStatuses/{operation-id}?api-version=2020-06-01
```

Pour vérifier l’état de l’opération asynchrone, l’envoi d’une autre demande à cette URL.

```HTTP
GET 
https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Resources/deployments/{deployment-name}/operationStatuses/{operation-id}?api-version=2020-06-01
```

Le corps de la réponse contient l’état de l’opération :

```json
{
    "status": "Running"
}
```

Lorsque le déploiement est terminé, la réponse contient :

```json
{
    "status": "Succeeded"
}
```

### <a name="create-storage-account-202-with-location-and-retry-after"></a>Créer le compte de stockage (202 avec Location et Retry-After)

Cet exemple montre comment déterminer l’état de l’[opération de création pour des comptes de stockage](/rest/api/storagerp/storageaccounts/create). La demande initiale présente le format suivant :

```HTTP
PUT
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}?api-version=2019-06-01
```

Et le corps de la demande contient les propriétés du compte de stockage :

```json
{
    "location": "South Central US",
    "properties": {},
    "sku": {
        "name": "Standard_LRS"
    },
    "kind": "Storage"
}
```

Renvoie le code d’état 202. Les valeurs d’en-tête incluent les deux valeurs suivantes :

```HTTP
Location: https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Storage/operations/{operation-id}?monitor=true&api-version=2019-06-01
Retry-After: 17
```

Une fois le nombre de secondes spécifié dans Retry-After écoulé, vérifiez l’état de l’opération asynchrone en envoyant une autre demande à cette URL.

```HTTP
GET 
https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Storage/operations/{operation-id}?monitor=true&api-version=2019-06-01
```

Si la demande est toujours en cours d’exécution, vous recevez un code d’état 202. Si la demande est terminée, vous recevez un code d’état 200, et le corps de la réponse contient les propriétés du compte de stockage qui a été créé.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d'informations sur chaque opération REST, consultez la [documentation de l'API REST](/rest/api/azure/).
* Pour plus d'informations sur le déploiement de modèles via l'API REST de Resource Manager, consultez [Déployer des ressources à l'aide de modèles Resource Manager et de l'API REST Resource Manager](../templates/deploy-rest.md).