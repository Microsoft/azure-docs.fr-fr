---
title: Obtenir les données d’utilisation d’une machine virtuelle Azure à l’aide de l’API REST
description: Utilisez les API REST Azure pour collecter des métriques d’utilisation sur une machine virtuelle.
author: rloutlaw
ms.service: virtual-machines
ms.subservice: monitoring
ms.custom: REST
ms.topic: how-to
ms.date: 06/13/2018
ms.author: routlaw
ms.openlocfilehash: 0a448d4b4538d62c1631cf16d88ec0a7e7f2a399
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97508031"
---
# <a name="get-virtual-machine-usage-metrics-using-the-rest-api"></a>Obtenir les métriques d’utilisation d’une machine virtuelle à l’aide de l’API REST

Cet exemple montre comment récupérer l’utilisation du processeur pour une machine virtuelle Linux à l’aide de [l’API REST Azure](/rest/api/azure/).

Pour consulter la documentation de référence complète et d’autres exemples avec l’API REST, reportez-vous à [Informations de référence sur l’API REST Azure Monitor](/rest/api/monitor). 

## <a name="build-the-request"></a>Générer la demande

Utilisez la requête GET suivante pour collecter les métriques [Pourcentage d’UC](../../azure-monitor/platform/metrics-supported.md#microsoftcomputevirtualmachines) à partir d’une machine virtuelle.

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmname}/providers/microsoft.insights/metrics?api-version=2018-01-01&metricnames=Percentage%20CPU&timespan=2018-06-05T03:00:00Z/2018-06-07T03:00:00Z
```

### <a name="request-headers"></a>En-têtes de requête

Les en-têtes suivants sont requis : 

|En-tête de requête|Description|  
|--------------------|-----------------|  
|*Content-Type :*|Obligatoire. Défini sur `application/json`.|  
|*Authorization :*|Obligatoire. Défini sur un [jeton d’accès](/rest/api/azure/#authorization-code-grant-interactive-clients) `Bearer` valide. |  

### <a name="uri-parameters"></a>Paramètres URI

| Nom | Description |
| :--- | :---------- |
| subscriptionId | L’ID d’abonnement d’un abonnement Azure. Si vous avez plusieurs abonnements, consultez [Utilisation de plusieurs abonnements](/cli/azure/manage-azure-subscriptions-azure-cli?view=azure-cli-latest). |
| resourceGroupName | Nom du groupe de ressources Azure associé à la ressource. Vous pouvez obtenir cette valeur à partir de l’API Azure Resource Manager, de l’interface CLI ou du portail. |
| vmname | Nom de la machine virtuelle Azure. |
| metricnames | Liste des [métriques Load Balancer](../../load-balancer/load-balancer-standard-diagnostics.md) valides, séparées par des virgules. |
| api-version | Version de l’API à utiliser pour la requête.<br /><br /> Ce document aborde la version d’API `2018-01-01`, incluse dans l’URL ci-dessus.  |
| intervalle de temps | Chaîne au format `startDateTime_ISO/endDateTime_ISO` qui définit l’intervalle de temps des métriques retournées. Ce paramètre facultatif est défini pour retourner une journée de données dans l’exemple. |
| &nbsp; | &nbsp; |

### <a name="request-body"></a>Corps de la demande

Aucun corps de requête n’est nécessaire pour cette opération.

## <a name="handle-the-response"></a>Gérer la réponse

Le code d’état 200 est retourné lorsque la liste des valeurs de métriques est retournée. La liste complète des codes d’erreur est disponible dans la [documentation de référence](/rest/api/monitor/metrics/list#errorresponse).

## <a name="example-response"></a>Exemple de réponse 

```json
{
    "cost": 0,
    "timespan": "2018-06-08T23:48:10Z/2018-06-09T00:48:10Z",
    "interval": "PT1M",
    "value": [
        {
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmname}/providers/microsoft.insights/metrics?api-version=2018-01-01&metricnames=Percentage%20CPU",
            "type": "Microsoft.Insights/metrics",
            "name": {
                "value": "Percentage CPU",
                "localizedValue": "Percentage CPU"
            },
            "unit": "Percent",
            "timeseries": [
                {
                    "metadatavalues": [],
                    "data": [
                        {
                            "timeStamp": "2018-06-08T23:48:00Z",
                            "average": 0.44
                        },
                        {
                            "timeStamp": "2018-06-08T23:49:00Z",
                            "average": 0.31
                        },
                        {
                            "timeStamp": "2018-06-08T23:50:00Z",
                            "average": 0.29
                        },
                        {
                            "timeStamp": "2018-06-08T23:51:00Z",
                            "average": 0.29
                        },
                        {
                            "timeStamp": "2018-06-08T23:52:00Z",
                            "average": 0.285
                        } ]
                } ]
        } ]
}
```
