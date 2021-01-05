---
title: Récupérer les mesures avec l’API REST
titleSuffix: Azure Load Balancer
description: Dans cet article, commencez à utiliser les API REST Azure pour collecter des mesures d’intégrité et d’utilisation pour Azure Load Balancer.
services: sql-database
author: asudbring
manager: KumudD
ms.service: load-balancer
ms.custom: REST, seodec18
ms.topic: how-to
ms.date: 11/19/2019
ms.author: allensu
ms.openlocfilehash: fbf21ba359ae7914acd2d812ec9e12f9f3ee557a
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97511941"
---
# <a name="get-load-balancer-usage-metrics-using-the-rest-api"></a>Récupérer les métriques d’usage de Load Balancer avec l’API REST

Collectez le nombre d’octets traités par [Standard Load Balancer](./load-balancer-overview.md) pour un intervalle de temps à l’aide de l’[API REST Azure](/rest/api/azure/).

Pour consulter la documentation de référence complète et d’autres exemples avec l’API REST, reportez-vous à [Informations de référence sur l’API REST Azure Monitor](/rest/api/monitor). 

## <a name="build-the-request"></a>Générer la demande

Utilisez la requête GET suivante pour collecter la métrique [ByteCount](./load-balancer-standard-diagnostics.md#multi-dimensional-metrics) depuis Standard Load Balancer. 

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/loadBalancers/{loadBalancerName}/providers/microsoft.insights/metrics?api-version=2018-01-01&metricnames=ByteCount&timespan=2018-06-05T03:00:00Z/2018-06-07T03:00:00Z
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
| resourceGroupName | Nom du groupe de ressources qui contient la ressource. Vous pouvez obtenir cette valeur à partir de l’API Azure Resource Manager, de CLI ou du portail. |
| loadBalancerName | Nom de l’équilibreur de charge Azure Load Balancer. |
| Noms de métriques | Liste des [métriques Load Balancer](./load-balancer-standard-diagnostics.md) valides, séparées par des virgules. |
| api-version | Version de l’API à utiliser pour la requête.<br /><br /> Ce document aborde la version d’API `2018-01-01`, incluse dans l’URL ci-dessus.  |
| intervalle de temps | Intervalle de temps de la requête. Il s’agit d’une chaîne au format `startDateTime_ISO/endDateTime_ISO`. Ce paramètre facultatif est défini pour retourner une journée de données dans l’exemple. |
| &nbsp; | &nbsp; |

### <a name="request-body"></a>Corps de la demande

Aucun corps de requête n’est nécessaire pour cette opération.

## <a name="handle-the-response"></a>Gérer la réponse

Le code d’état 200 est retourné lorsque la liste des valeurs de métriques est retournée. La liste complète des codes d’erreur est disponible dans la [documentation de référence](/rest/api/monitor/metrics/list#errorresponse).

## <a name="example-response"></a>Exemple de réponse 

```json
{
    "cost": 0,
    "timespan": "2018-06-05T03:00:00Z/2018-06-07T03:00:00Z",
    "interval": "PT1M",
    "value": [
        {
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/loadBalancers/{loadBalancerName}/providers/Microsoft.Insights/metrics/ByteCount",
            "type": "Microsoft.Insights/metrics",
            "name": {
                "value": "ByteCount",
                "localizedValue": "Byte Count"
            },
            "unit": "Count",
            "timeseries": [
                {
                    "metadatavalues": [],
                    "data": [
                        {
                            "timeStamp": "2018-06-06T17:24:00Z",
                            "total": 1067921034.0
                        },
                        {
                            "timeStamp": "2018-06-06T17:25:00Z",
                            "total": 0.0
                        },
                        {
                            "timeStamp": "2018-06-06T17:26:00Z",
                            "total": 3781344.0
                        },
                    ]
                }
            ]
        }
    ],
    "namespace": "Microsoft.Network/loadBalancers",
    "resourceregion": "eastus"
}
```