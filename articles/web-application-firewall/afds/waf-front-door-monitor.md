---
title: Surveillance et journalisation du pare-feu d’applications web Azure
description: Découvrez le pare-feu d’applications web (WAF) avec la surveillance et la journalisation FrontDoor
author: vhorne
ms.service: web-application-firewall
ms.topic: article
services: web-application-firewall
ms.date: 06/09/2020
ms.author: victorh
ms.openlocfilehash: 596374d4f3f188e08a10bd25b36b178cc79a6e57
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84808948"
---
# <a name="azure-web-application-firewall-monitoring-and-logging"></a>Surveillance et journalisation du pare-feu d’applications web Azure

La surveillance et la journalisation du pare-feu d’applications web (WAF) Azure sont fournies via la journalisation et l’intégration avec Azure Monitor et les journaux Azure Monitor.

## <a name="azure-monitor"></a>Azure Monitor

La journalisation WAF avec Front Door est intégrée à [Azure Monitor](../../azure-monitor/overview.md). Azure Monitor vous permet d’effectuer le suivi des informations de diagnostic, y compris des alertes et des journaux WAF. Vous pouvez configurer la supervision WAF dans la ressource Front Door sur le portail sous l’onglet **Diagnostics** ou directement via le service Azure Monitor.

À partir du portail Azure, accédez au type de ressource Front Door. Sous l’onglet **Supervision**/**Métriques** sur la gauche, ajoutez **WebApplicationFirewallRequestCount** pour suivre le nombre de requêtes qui correspondent à des règles WAF. Vous pouvez créer des filtres personnalisés sur la base des types d’action et des noms de règle.

:::image type="content" source="../media/waf-frontdoor-monitor/waf-frontdoor-metrics.png" alt-text="WAFMetrics ":::

## <a name="logs-and-diagnostics"></a>Journaux et diagnostics

WAF avec Front Door fournit des rapports détaillés sur chaque menace détectée. La journalisation est intégrée aux journaux Diagnostics Azure et les alertes sont enregistrées au format json. Ces journaux d’activité peuvent être intégrés aux [journaux d’activité Azure Monitor](../../azure-monitor/insights/azure-networking-analytics.md).

![WAFDiag](../media/waf-frontdoor-monitor/waf-frontdoor-diagnostics.png)

[FrontdoorAccessLog](../../frontdoor/front-door-diagnostics.md) journalise toutes les requêtes. FrontdoorWebApplicationFirewallLog journalise toutes les requêtes qui correspondent à une règle WAF ayant le schéma ci-dessous :

| Propriété  | Description |
| ------------- | ------------- |
|Action|Action effectuée sur la requête|
| ClientIp | Adresse IP du client à l’origine de la demande. S’il existait un en-tête X-Forwarded-For dans la requête, l’adresse IP du client est sélectionnée dans le champ d’en-tête. |
| ClientPort | Adresse IP du port du client qui a effectué la requête. |
| Détails|Détails supplémentaires sur la requête correspondante |
|| matchVariableName : nom du paramètre http de la requête correspondant, par exemple, noms d’en-tête|
|| matchVariableValue : valeurs qui ont déclenché la correspondance|
| Host | En-tête de l’hôte de la requête correspondante |
| Policy | Nom de la stratégie WAF correspondant à la requête. |
| PolicyMode | Mode d’opération de la stratégie WAF. Les valeurs possibles sont « Prevention » et « Detection » |
| RequestUri | URI complet de la requête mise en correspondance. |
| RuleName | Nom de la règle WAF correspondant à la requête. |
| SocketIp | Adresse IP source vue par WAF. Cette adresse IP est basée sur la session TCP, indépendamment des en-têtes de requête.|
| TrackingReference | Chaîne de référence unique qui identifie une requête traitée par Front Door, également envoyée en tant qu’en-tête X-Azure-Ref au client. Nécessaire pour pouvoir effectuer une recherche détaillée dans les journaux d’accès pour une requête spécifique. |

L’exemple de requête suivant retourne les journaux WAF sur les requêtes bloquées :

``` WAFlogQuery
AzureDiagnostics
| where ResourceType == "FRONTDOORS" and Category == "FrontdoorWebApplicationFirewallLog"
| where action_s == "Block"

```

Voici un exemple de requête enregistrée dans le journal WAF :

``` WAFlogQuerySample
{
    "time":  "2020-06-09T22:32:17.8376810Z",
    "category": "FrontdoorWebApplicationFirewallLog",
    "operationName": "Microsoft.Network/FrontDoorWebApplicationFirewallLog/Write",
    "properties":
    {
        "clientIP":"xxx.xxx.xxx.xxx",
        "clientPort":"52097",
        "socketIP":"xxx.xxx.xxx.xxx",
        "requestUri":"https://wafdemofrontdoorwebapp.azurefd.net:443/?q=%27%20or%201=1",
        "ruleName":"Microsoft_DefaultRuleSet-1.1-SQLI-942100",
        "policy":"WafDemoCustomPolicy",
        "action":"Block",
        "host":"wafdemofrontdoorwebapp.azurefd.net",
        "trackingReference":"08Q3gXgAAAAAe0s71BET/QYwmqtpHO7uAU0pDRURHRTA1MDgANjMxNTAwZDAtOTRiNS00YzIwLTljY2YtNjFhNzMyOWQyYTgy",
        "policyMode":"prevention",
        "details":
            {
            "matches":
                [{
                "matchVariableName":"QueryParamValue:q",
                "matchVariableValue":"' or 1=1"
                }]
            }
     }
}
```

L’exemple de requête suivant retourne les entrées AccessLogs :

``` AccessLogQuery
AzureDiagnostics
| where ResourceType == "FRONTDOORS" and Category == "FrontdoorAccessLog"

```

Voici un exemple de requête enregistrée sans le journal d'accès :

``` AccessLogSample
{
"time": "2020-06-09T22:32:17.8383427Z",
"category": "FrontdoorAccessLog",
"operationName": "Microsoft.Network/FrontDoor/AccessLog/Write",
 "properties":
    {
    "trackingReference":"08Q3gXgAAAAAe0s71BET/QYwmqtpHO7uAU0pDRURHRTA1MDgANjMxNTAwZDAtOTRiNS00YzIwLTljY2YtNjFhNzMyOWQyYTgy",
    "httpMethod":"GET",
    "httpVersion":"2.0",
    "requestUri":"https://wafdemofrontdoorwebapp.azurefd.net:443/?q=%27%20or%201=1",
    "requestBytes":"715",
    "responseBytes":"380",
    "userAgent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4157.0 Safari/537.36 Edg/85.0.531.1",
    "clientIp":"xxx.xxx.xxx.xxx",
    "socketIp":"xxx.xxx.xxx.xxx",
    "clientPort":"52097",
    "timeTaken":"0.003",
    "securityProtocol":"TLS 1.2",
    "routingRuleName":"WAFdemoWebAppRouting",
    "rulesEngineMatchNames":[],
    "backendHostname":"wafdemowebappuscentral.azurewebsites.net:443",
    "sentToOriginShield":false,
    "httpStatusCode":"403",
    "httpStatusDetails":"403",
    "pop":"SJC",
    "cacheStatus":"CONFIG_NOCACHE"
    }
}

```

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [Front Door](../../frontdoor/front-door-overview.md).
