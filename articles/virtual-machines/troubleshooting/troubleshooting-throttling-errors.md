---
title: Résolution des erreurs de limitation dans Azure | Microsoft Docs
description: Erreurs de limitation, nouvelles tentatives et interruption dans Azure Compute.
services: virtual-machines
documentationcenter: ''
author: changov
manager: gwallace
editor: ''
tags: azure-resource-manager,azure-service-management
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.date: 09/18/2018
ms.author: changov
ms.reviewer: vashan, rajraj
ms.openlocfilehash: b1cc8a43423ecd33218948aaa001fc34877eac60
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87074276"
---
# <a name="troubleshooting-api-throttling-errors"></a>Résolution des erreurs de limitation d’API 

Les requêtes Azure Compute peuvent être limitées à un abonnement et spécifiées par région pour optimiser les performances globales du service. Nous garantissons que tous les appels adressés au fournisseur de ressources de calcul Azure qui gère les ressources sous l’espace de noms Microsoft.Compute ne dépassent pas le taux de requêtes d’API autorisé maximal. Ce document décrit la limitation d’API, les détails sur la résolution des problèmes de limitation et les bonnes pratiques pour éviter toute limitation.  

## <a name="throttling-by-azure-resource-manager-vs-resource-providers"></a>Limitation par Azure Resource Manager ou les fournisseurs de ressources  

En tant que porte d’entrée d’Azure, Azure Resource Manager procède à l’authentification et à la validation de premier niveau ainsi qu’à la limitation de toutes les requêtes d’API entrantes. Les limites de débit d’appels et en-têtes HTTP de réponse de diagnostic associés Azure Resource Manager sont décrits [ici](../../azure-resource-manager/management/request-limits-and-throttling.md).
 
Quand un client API Azure reçoit une erreur de limitation, l’état HTTP est 429, « Trop de requêtes ». Pour comprendre si la limitation des requêtes est effectuée par Azure Resource Manager ou un fournisseur de ressources sous-jacent comme CRP, examinez l’en-tête `x-ms-ratelimit-remaining-subscription-reads` à la recherche de requêtes GET et les en-têtes de réponse `x-ms-ratelimit-remaining-subscription-writes` à la recherche de requêtes autres que GET. Si le nombre d’appels restants est proche de 0, la limite des appels générale de l’abonnement définie par Azure Resource Manager a été atteinte. Les activités de tous les clients de l’abonnement sont comptées ensemble. Sinon, la limitation provient du fournisseur de ressources cible (celui désigné par le segment `/providers/<RP>` de l’URL de requête). 

## <a name="call-rate-informational-response-headers"></a>En-têtes de réponse d’information de débit d’appels 

| En-tête                            | Format de valeur                           | Exemple                               | Description                                                                                                                                                                                               |
|-----------------------------------|----------------------------------------|---------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| x-ms-ratelimit-remaining-resource |```<source RP>/<policy or bucket>;<count>```| Microsoft.Compute/HighCostGet3Min;159 | Nombre d’appels d’API restants pour la stratégie de limitation portant sur le groupe d’opérations ou de compartiments de ressources avec la cible de cette requête                                                                   |
| x-ms-request-charge               | ```<count>```                             | 1                                     | Nombre d’appels « facturés » pour cette requête HTTP dans le calcul de la limite de la stratégie applicable. Il s’agit généralement de 1. Les requêtes de lots, par exemple pour la mise à l’échelle d’un groupe de machines virtuelles identiques, peuvent facturer plusieurs comptes. |


Notez qu’une requête d’API peut être soumise à plusieurs stratégies de limitation. Il y aura un en-tête `x-ms-ratelimit-remaining-resource` distinct pour chaque stratégie. 

Voici un exemple de réponse pour supprimer une requête de groupe de machines virtuelles identiques.

```
x-ms-ratelimit-remaining-resource: Microsoft.Compute/DeleteVMScaleSet3Min;107 
x-ms-ratelimit-remaining-resource: Microsoft.Compute/DeleteVMScaleSet30Min;587 
x-ms-ratelimit-remaining-resource: Microsoft.Compute/VMScaleSetBatchedVMRequests5Min;3704 
x-ms-ratelimit-remaining-resource: Microsoft.Compute/VmssQueuedVMOperations;4720 
```

## <a name="throttling-error-details"></a>Détails de l’erreur de limitation

L’état HTTP 429 est couramment utilisé pour refuser une requête, car une limite de débit d’appels est atteinte. Une réponse d’erreur de limitation typique du fournisseur de ressources de calcul ressemblera à l’exemple ci-dessous (seuls les en-têtes appropriés sont affichés) :

```
HTTP/1.1 429 Too Many Requests
x-ms-ratelimit-remaining-resource: Microsoft.Compute/HighCostGet3Min;46
x-ms-ratelimit-remaining-resource: Microsoft.Compute/HighCostGet30Min;0
Retry-After: 1200
Content-Type: application/json; charset=utf-8
{
  "code": "OperationNotAllowed",
  "message": "The server rejected the request because too many requests have been received for this subscription.",
  "details": [
    {
      "code": "TooManyRequests",
      "target": "HighCostGet30Min",
      "message": "{\"operationGroup\":\"HighCostGet30Min\",\"startTime\":\"2018-06-29T19:54:21.0914017+00:00\",\"endTime\":\"2018-06-29T20:14:21.0914017+00:00\",\"allowedRequestCount\":800,\"measuredRequestCount\":1238}"
    }
  ]
}

```

La stratégie avec le nombre d’appels restants de 0 est celle en raison de laquelle l’erreur de limitation est retournée. Dans ce cas, il s’agit de `HighCostGet30Min`. Le format global du corps de la réponse est le format d’erreur d’API Azure Resource Manager général (conforme à OData). Le code d’erreur principal, `OperationNotAllowed`, est celui que le fournisseur de ressources de calcul utilise pour signaler les erreurs de limitation (parmi d’autres types d’erreurs du client). La propriété `message` de la ou des erreurs internes contient une structure JSON sérialisée avec les détails de la violation de limitation.

Comme illustré ci-dessus, chaque erreur de limitation inclut l’en-tête `Retry-After` qui fournit le nombre minimal de secondes pendant lesquelles le client doit attendre avant de réessayer la requête. 

## <a name="api-call-rate-and-throttling-error-analyzer"></a>Taux d’appel d’API et analyseur d’erreur de limitation
Une préversion d’une fonctionnalité de résolution des problèmes est disponible pour l’API du fournisseur de ressources de calcul. Ces applets de commande PowerShell fournissent des statistiques sur le taux de requête d’API par intervalle de temps par opération et sur les violations de limitation par groupe d’opération (stratégie) :
-   [Export-AzLogAnalyticRequestRateByInterval](/powershell/module/az.compute/export-azloganalyticrequestratebyinterval)
-   [Export-AzLogAnalyticThrottledRequest](/powershell/module/az.compute/export-azloganalyticthrottledrequest)

Les statistiques d’appels d’API peuvent fournir de précieuses informations sur le comportement des clients d’un abonnement, et faciliter l’identification des modèles d’appels qui provoquent la limitation.

À l’heure actuelle, l’une des limitations de l’analyseur est qu’il ne compte pas les requêtes relatives aux types de ressources disque et instantané (prenant en charge les disques managés). Dans la mesure où il recueille des données à partir de la télémétrie de CRP, il ne peut pas non plus aider à identifier les erreurs de limitation ARM. Toutefois, celles-ci peuvent être identifiées facilement d’après les en-têtes de réponse ARM distinctifs, comme indiqué précédemment.

Les applets de commande PowerShell utilisent une API de service REST, qui peut être facilement appelée directement par les clients (mais sans aucune prise en charge formelle pour le moment). Pour afficher le format de requête HTTP, exécutez les applets de commande avec le commutateur -Debug ou surveillez leur exécution avec Fiddler.


## <a name="best-practices"></a>Meilleures pratiques 

- Ne retentez pas les erreurs d’API du service Azure de manière inconditionnelle et/ou immédiate. Une occurrence courante pour le code client est d’entrer dans une boucle de nouvelle tentative rapide quand il rencontre une erreur qui n’est pas renouvelable. Les nouvelles tentatives vont finalement dépasser la limite des appels autorisée pour le groupe d’opérations cible et avoir un impact sur d’autres clients de l’abonnement. 
- Dans les cas d’automation d’API de volumes importants, envisagez d’implémenter une limitation automatique côté client proactif quand le nombre d’appels disponible pour un groupe d’opérations cible descend en dessous du seuil minimum. 
- Lors du suivi des opérations asynchrones, respectez les mentions d’en-tête Retry-After. 
- Si le code client a besoin d’informations sur une machine virtuelle spécifique, interrogez cette machine virtuelle directement au lieu de répertorier toutes les machines virtuelles dans le groupe de ressources qui les contient ou l’ensemble de l’abonnement, puis choisissez la machine virtuelle nécessaire côté client. 
- Si le code client a besoin de machines virtuelles, de disques et de captures instantanées d’un emplacement Azure spécifique, utilisez le formulaire basé sur l’emplacement de la requête au lieu d’interroger toutes les machines virtuelles de l’abonnement et de les filtrer par emplacement côté client : requête `GET /subscriptions/<subId>/providers/Microsoft.Compute/locations/<location>/virtualMachines?api-version=2017-03-30` aux points de terminaison régionaux du fournisseur de ressources de calcul. 
-   Lors de la création ou de la mise à jour des ressources d’API en particulier, dans les machines virtuelles et dans les groupes de machines virtuelles identiques, il est beaucoup plus efficace d’effectuer le suivi de l’opération asynchrone retournée jusqu’à la fin que d’interroger l’URL de la ressource (selon `provisioningState`).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le guide du mécanisme de nouvelle tentative pour d’autres services dans Azure, consultez [Guide du mécanisme de nouvelle tentative relatif aux différents services](/azure/architecture/best-practices/retry-service-specific)
