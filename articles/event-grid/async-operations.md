---
title: État des opérations asynchrones Event Grid
description: Décrit comment effectuer le suivi des opérations asynchrones Event Grid dans Azure. Elle affiche les valeurs que vous utilisez pour obtenir l’état d’une opération de longue durée.
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: baae7b097a0b696d405c0e7ea3d3bdeb326f23b1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89011682"
---
# <a name="track-event-grid-asynchronous-azure-operations"></a>Suivre les opérations asynchrones Event Grid Azure
Certaines opérations REST Azure s’exécutent de façon asynchrone, car elles ne peuvent pas être effectuées rapidement. Cet article explique comment effectuer le suivi de l’état des opérations asynchrones via les valeurs retournées dans la réponse.  

## <a name="status-codes-for-asynchronous-operations"></a>Codes d’état pour les opérations asynchrones
Initialement, une opération asynchrone retourne un code d’état HTTP de type :

* 201 (créé)
* 202 (accepté) 

Lorsque l’opération se termine correctement, elle renvoie :

* 200 (OK)
* 204 (Pas de contenu) 

Reportez-vous à la [documentation de l’API REST](/rest/api/) pour voir les réponses liées à l’opération que vous exécutez.

## <a name="monitor-status-of-operation"></a>Surveiller l’état de l'opération
Les opérations REST asynchrones retournent des valeurs d’en-tête, qui vous permettent de déterminer l’état de l’opération. Il existe potentiellement trois valeurs d’en-tête à examiner :

* `Azure-AsyncOperation`- URL pour vérifier l’état de l’opération en cours. Si votre opération renvoie cette valeur, utilisez-la toujours (au lieu de Location) pour suivre l’état de l’opération.
* `Location`- URL pour déterminer si une opération est terminée. Utilisez cette valeur uniquement quand la valeur Azure-AsyncOperation n’est pas retournée.
* `Retry-After`- Le nombre de secondes à attendre avant de vérifier l’état de l’opération asynchrone.

Toutefois, certaines opérations asynchrones ne retournent pas toutes ces valeurs. Par exemple, vous devrez évaluer la valeur d’en-tête Azure-AsyncOperation pour une opération et la valeur d’en-tête Location pour une autre opération. 

Vous récupérez les valeurs d’en-tête de la même façon que vous récupérez une valeur d’en-tête pour une demande. Par exemple, en C#, vous récupérez la valeur d’en-tête d’un objet `HttpWebResponse` nommé `response` avec le code suivant :

```cs
response.Headers.GetValues("Azure-AsyncOperation").GetValue(0)
```

## <a name="azure-asyncoperation-request-and-response"></a>Demande et réponse Azure-AsyncOperation

Pour obtenir l’état de l’opération asynchrone, envoyez une requête GET à l’URL dans la valeur d’en-tête Azure-AsyncOperation.

Le corps de la réponse de cette opération contient des informations sur l’opération. L’exemple suivant montre les valeurs possibles retournées par l’opération :

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

Seule la valeur `status` est renvoyée pour toutes les réponses. L’objet d’erreur est retourné lorsque l’état indique que l'opération a échoué (Failed) ou a été annulée (Canceled). Toutes les autres valeurs sont facultatives ; par conséquent, la réponse que vous recevez peut être différente de l’exemple.

## <a name="provisioningstate-values"></a>Valeurs provisioningState

Les opérations qui créent, mettent à jour ou suppriment (PUT, PATCH, DELETE) une ressource retournent généralement une valeur `provisioningState`. Lorsqu’une opération est terminée, une des trois valeurs suivantes est retournée : 

* Opération réussie
* Échec
* Opération annulée

Toutes les autres valeurs indiquent que l’opération est en cours d’exécution. Le fournisseur de ressources peut retourner une valeur personnalisée qui indique son état. Par exemple, vous pouvez recevoir l'état d'acceptation **Accepted** lorsque la demande est reçue et en cours d’exécution.


## <a name="example-requests-and-responses"></a>Exemples de demandes et de réponses

### <a name="start-virtual-machine-202-with-azure-asyncoperation"></a>Démarrez la machine virtuelle (202 avec Azure-AsyncOperation)
Cet exemple montre comment déterminer l’état de l'opération **start** pour les machines virtuelles. La demande initiale présente le format suivant :

```HTTP
POST 
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Compute/virtualMachines/{vm-name}/start?api-version=2016-03-30
```

Renvoie le code d’état 202. Les valeurs d’en-tête incluent :

```HTTP
Azure-AsyncOperation : https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Compute/locations/{region}/operations/{operation-id}?api-version=2016-03-30
```

Pour vérifier l’état de l’opération asynchrone, l’envoi d’une autre demande à cette URL.

```HTTP
GET 
https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Compute/locations/{region}/operations/{operation-id}?api-version=2016-03-30
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

Cet exemple montre comment déterminer l’état de l'opération **deployments** pour le déploiement de ressources sur Azure. La demande initiale présente le format suivant :

```HTTP
PUT
https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/microsoft.resources/deployments/{deployment-name}?api-version=2016-09-01
```

Il renvoie le code d’état 201. Le corps de la réponse inclut :

```json
"provisioningState":"Accepted",
```

Les valeurs d’en-tête incluent :

```HTTP
Azure-AsyncOperation: https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Resources/deployments/{deployment-name}/operationStatuses/{operation-id}?api-version=2016-09-01
```

Pour vérifier l’état de l’opération asynchrone, l’envoi d’une autre demande à cette URL.

```HTTP
GET 
https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Resources/deployments/{deployment-name}/operationStatuses/{operation-id}?api-version=2016-09-01
```

Le corps de la réponse contient l’état de l’opération :

```json
{"status":"Running"}
```

Lorsque le déploiement est terminé, la réponse contient :

```json
{"status":"Succeeded"}
```

### <a name="create-storage-account-202-with-location-and-retry-after"></a>Créer le compte de stockage (202 avec Location et Retry-After)

Cet exemple montre comment déterminer l’état de l'opération **create** pour les comptes de stockage. La demande initiale présente le format suivant :

```HTTP
PUT
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}?api-version=2016-01-01
```

Et le corps de la demande contient les propriétés du compte de stockage :

```json
{ "location": "South Central US", "properties": {}, "sku": { "name": "Standard_LRS" }, "kind": "Storage" }
```

Renvoie le code d’état 202. Les valeurs d’en-tête incluent les deux valeurs suivantes :

```HTTP
Location: https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Storage/operations/{operation-id}?monitor=true&api-version=2016-01-01
Retry-After: 17
```

Une fois le nombre de secondes spécifié dans Retry-After écoulé, vérifiez l’état de l’opération asynchrone en envoyant une autre demande à cette URL.

```HTTP
GET 
https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Storage/operations/{operation-id}?monitor=true&api-version=2016-01-01
```

Si la demande est toujours en cours d’exécution, vous recevez un code d’état 202. Si la demande est terminée, vous recevez un code d’état 200, et le corps de la réponse contient les propriétés du compte de stockage qui a été créé.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d'informations sur chaque opération REST, consultez la [documentation de l'API REST](/rest/api/).
* Pour plus d'informations sur le déploiement de modèles via l'API REST de Resource Manager, consultez [Déployer des ressources à l'aide de modèles Resource Manager et de l'API REST Resource Manager](../azure-resource-manager/templates/deploy-rest.md).