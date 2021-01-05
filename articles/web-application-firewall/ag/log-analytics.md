---
title: Utiliser Log Analytics pour examiner les journaux du pare-feu d’applications web sur Application Gateway
description: Cet article vous montre comment utiliser Log Analytics pour examiner les journaux du pare-feu d’applications web sur Application Gateway
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: article
ms.date: 09/06/2019
ms.author: victorh
ms.openlocfilehash: 733a9af589718e9abe8736491023003744428cd6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91361062"
---
# <a name="use-log-analytics-to-examine-application-gateway-web-application-firewall-waf-logs"></a>Utiliser Azure Log Analytics pour examiner les journaux du pare-feu d’applications web (WAF) sur Application Gateway

Une fois que votre WAF Application Gateway est opérationnel, vous pouvez activer les journaux pour examiner ce qui se produit avec chaque requête. Les journaux de pare-feu donnent des informations sur ce que le WAF évalue, met en correspondance et bloque. Avec Azure Monitor Log Analytics, vous pouvez examiner les données contenues dans les journaux de pare-feu pour obtenir encore plus d’informations. Pour plus d’informations sur la création d’un espace de travail Log Analytics, consultez [Créer un espace de travail Log Analytics dans le portail Azure](../../azure-monitor/learn/quick-create-workspace.md). Pour plus d’informations sur les requêtes de journal, consultez [Vue d’ensemble des requêtes de journal dans Azure Monitor](../../azure-monitor/log-query/log-query-overview.md).

## <a name="import-waf-logs"></a>Importer les journaux du WAF

Pour importer les journaux du pare-feu dans Log Analytics, consultez [Intégrité du serveur principal, journaux de ressources et métriques pour la passerelle Application Gateway](../../application-gateway/application-gateway-diagnostics.md#diagnostic-logging). Une fois les journaux du pare-feu dans votre espace de travail Log Analytics, vous pouvez afficher les données, écrire des requêtes, créer des visualisations, et ajouter ces dernières au tableau de bord de votre portail.

## <a name="explore-data-with-examples"></a>Explorer les données avec des exemples

Pour afficher les données brutes dans le journal du pare-feu, vous pouvez exécuter la requête suivante :

```
AzureDiagnostics 
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
```

Cela ressemblera à la requête suivante :

![Requête Log Analytics](../media/log-analytics/log-query.png)

Vous pouvez explorer les données et tracer des graphiques ou créer des visualisations à partir d’ici. Consultez les requêtes suivantes comme point de départ :

### <a name="matchedblocked-requests-by-ip"></a>Requêtes mises en correspondance/bloquées par adresse IP

```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
| summarize count() by clientIp_s, bin(TimeGenerated, 1m)
| render timechart
```

### <a name="matchedblocked-requests-by-uri"></a>Requêtes mises en correspondance/bloquées par URI

```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
| summarize count() by requestUri_s, bin(TimeGenerated, 1m)
| render timechart
```

### <a name="top-matched-rules"></a>Principales règles mises en correspondance

```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
| summarize count() by ruleId_s, bin(TimeGenerated, 1m)
| where count_ > 10
| render timechart
```

### <a name="top-five-matched-rule-groups"></a>Cinq principaux groupes de règles mis en correspondance

```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
| summarize Count=count() by details_file_s, action_s
| top 5 by Count desc
| render piechart
```

## <a name="add-to-your-dashboard"></a>Ajouter à votre tableau de bord

Une fois que vous créez une requête, vous pouvez l’ajouter à votre tableau de bord.  Sélectionnez **Épingler au tableau de bord** dans le coin supérieur droit de l’espace de travail Log Analytics. Avec les quatre requêtes précédentes épinglées à un exemple de tableau de bord, voici les données que vous pouvez voir en un coup d’œil :

![Capture d’écran montrant un tableau de bord Azure dans lequel vous pouvez ajouter votre requête.](../media/log-analytics/dashboard.png)

## <a name="next-steps"></a>Étapes suivantes

[Intégrité du serveur principal, journaux de ressources et métriques pour la passerelle Application Gateway](../../application-gateway/application-gateway-diagnostics.md)