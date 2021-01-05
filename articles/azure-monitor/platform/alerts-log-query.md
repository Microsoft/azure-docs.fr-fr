---
title: Optimisation des requêtes d’alerte de journal | Microsoft Docs
description: Recommandations pour l’écriture de requêtes d’alerte efficaces
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.date: 02/19/2019
ms.subservice: alerts
ms.openlocfilehash: e7c9c76816b5d1ee2eedfb7e54645e056906feef
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186624"
---
# <a name="optimizing-log-alert-queries"></a>Optimisation des requêtes d’alerte de journal
Cet article explique comment écrire et convertir des requêtes d’[alerte de journal](alerts-unified-log.md) pour obtenir des performances optimales. Les requêtes optimisées réduisent la latence et la charge des alertes qui s’exécutent fréquemment.

## <a name="how-to-start-writing-an-alert-log-query"></a>Comment commencer à écrire une requête de journal d’alerte

Les requêtes d’alerte commencent en [interrogeant les données du journal dans Log Analytics](alerts-log.md#create-a-log-alert-rule-with-the-azure-portal) qui indiquent le problème. Vous pouvez utiliser la [rubrique des exemples de requêtes d’alerte](../log-query/example-queries.md) pour comprendre ce que vous pouvez découvrir. Vous pouvez également [commencer à écrire votre propre requête](../log-query/log-analytics-tutorial.md). 

### <a name="queries-that-indicate-the-issue-and-not-the-alert"></a>Requêtes qui indiquent le problème et non l’alerte

Le flux d’alertes a été créé pour transformer les résultats qui indiquent le problème à une alerte. Par exemple, dans le cas d’une requête telle que :

``` Kusto
SecurityEvent
| where EventID == 4624
```

Si l’intention de l’utilisateur est d’alerter, quand ce type d’événement se produit, la logique d’alerte ajoute `count` à la requête. La requête suivante s’exécutera :

``` Kusto
SecurityEvent
| where EventID == 4624
| count
```

Il n’est pas nécessaire d’ajouter une logique d’alerte à la requête et cela peut même poser des problèmes. Dans l’exemple ci-dessus, si vous incluez `count` dans votre requête, la valeur générée sera toujours 1, car le service d’alerte exécutera `count` de `count`.

### <a name="avoid-limit-and-take-operators"></a>Éviter les opérateurs `limit` et `take`

L’utilisation de `limit` et `take` dans les requêtes peut augmenter la latence et la charge des alertes, car les résultats ne sont pas cohérents sur la durée. Il est préférable de les utiliser seulement si nécessaire.

## <a name="log-query-constraints"></a>Journaliser les contraintes de requête
Les [requêtes de journal dans Azure Monitor](../log-query/log-query-overview.md) commencent par un opérateur de table, [`search`](/azure/kusto/query/searchoperator) ou [`union`](/azure/kusto/query/unionoperator).

Les requêtes pour les règles d’alerte de journal doivent toujours commencer par une table afin de définir une étendue claire, ce qui améliore à la fois les performances des requêtes et la pertinence des résultats. Les requêtes dans les règles d’alerte s’exécutent fréquemment. Par conséquent, l’utilisation de `search` et `union` peut entraîner une surcharge excessive ajoutant de la latence à l’alerte, car elle nécessite l’analyse de plusieurs tables. Ces opérateurs réduisent également la capacité du service d’alerte à optimiser la requête.

Nous ne prenons pas en charge la création ou la modification des règles d’alerte de journal qui utilisent des opérateurs `search` ou `union`, si ce n’est pour des requêtes inter-ressources.

Par exemple la requête d’alerte suivante a pour étendue la table _SecurityEvent_ et recherche un ID d’événement spécifique. Il s’agit de la seule table que la requête doit traiter.

``` Kusto
SecurityEvent
| where EventID == 4624
```

Les règles d’alerte de journal utilisant des [requêtes inter-ressources](../log-query/cross-workspace-query.md) ne sont pas affectées par ce changement, car les requêtes inter-ressources utilisent un type d’opérateur `union`, qui limite l’étendue de la requête à des ressources spécifiques. L’exemple suivant serait une requête d’alerte de journal valide :

```Kusto
union
app('Contoso-app1').requests,
app('Contoso-app2').requests,
workspace('Contoso-workspace1').Perf 
```

>[!NOTE]
> Les [requêtes inter-ressources](../log-query/cross-workspace-query.md) sont prises en charge dans la nouvelle [API scheduledQueryRules](/rest/api/monitor/scheduledqueryrules). Si vous utilisez toujours l’[API d’alerte Log Analytics héritée](api-alerts.md) pour créer des alertes de journal, vous pouvez en apprendre plus [ici](alerts-log-api-switch.md) sur le basculement.

## <a name="examples"></a>Exemples
Les exemples suivants incluent des requêtes de journal qui utilisent `search` et `union`, et présentent la procédure permettant de modifier ces requêtes pour les utiliser dans les règles d’alerte.

### <a name="example-1"></a>Exemple 1
Vous souhaitez créer une règle d’alerte de journal avec la requête suivante qui récupère les informations de performances à l’aide de `search` : 

``` Kusto
search *
| where Type == 'Perf' and CounterName == '% Free Space'
| where CounterValue < 30
```

Pour modifier cette requête, commencez par exécuter la requête suivante afin d’identifier la table à laquelle appartiennent les propriétés :

``` Kusto
search *
| where CounterName == '% Free Space'
| summarize by $table
```

Le résultat de cette requête montre que la propriété _CounterName_ provient de la table _Perf_.

Vous pouvez utiliser ce résultat pour créer la requête suivante, que vous utiliseriez pour la règle d’alerte :

``` Kusto
Perf
| where CounterName == '% Free Space'
| where CounterValue < 30
```

### <a name="example-2"></a>Exemple 2
Vous souhaitez créer une règle d’alerte de journal avec la requête suivante qui récupère les informations de performances à l’aide de `search` : 

``` Kusto
search ObjectName =="Memory" and CounterName=="% Committed Bytes In Use"
| summarize Avg_Memory_Usage =avg(CounterValue) by Computer
| where Avg_Memory_Usage between(90 .. 95)  
```

Pour modifier cette requête, commencez par exécuter la requête suivante afin d’identifier la table à laquelle appartiennent les propriétés :

``` Kusto
search ObjectName=="Memory" and CounterName=="% Committed Bytes In Use"
| summarize by $table
```

Le résultat de cette requête montre que les propriétés _ObjectName_ et _CounterName_ proviennent de la table _Perf_.

Vous pouvez utiliser ce résultat pour créer la requête suivante, que vous utiliseriez pour la règle d’alerte :

``` Kusto
Perf
| where ObjectName =="Memory" and CounterName=="% Committed Bytes In Use"
| summarize Avg_Memory_Usage=avg(CounterValue) by Computer
| where Avg_Memory_Usage between(90 .. 95)
```

### <a name="example-3"></a>Exemple 3

Vous souhaitez créer une règle d’alerte de journal avec la requête suivante qui utilise `search` et `union` pour récupérer les informations de performances : 

``` Kusto
search (ObjectName == "Processor" and CounterName == "% Idle Time" and InstanceName == "_Total")
| where Computer !in (
    union *
    | where CounterName == "% Processor Utility"
    | summarize by Computer)
| summarize Avg_Idle_Time = avg(CounterValue) by Computer
```

Pour modifier cette requête, commencez par exécuter la requête suivante afin d’identifier la table à laquelle appartiennent les propriétés dans la première partie de la requête : 

``` Kusto
search (ObjectName == "Processor" and CounterName == "% Idle Time" and InstanceName == "_Total")
| summarize by $table
```

Le résultat de cette requête montre que toutes ces propriétés proviennent de la table _Perf_.

À présent, utilisez `union` avec la commande `withsource` pour identifier la table source ayant contribué à chaque ligne.

``` Kusto
union withsource=table *
| where CounterName == "% Processor Utility"
| summarize by table
```

Le résultat de cette requête montre que ces propriétés proviennent aussi de la table _Perf_.

Vous pouvez utiliser ces résultats pour créer la requête suivante, que vous utiliseriez pour la règle d’alerte : 

``` Kusto
Perf
| where ObjectName == "Processor" and CounterName == "% Idle Time" and InstanceName == "_Total"
| where Computer !in (
    (Perf
    | where CounterName == "% Processor Utility"
    | summarize by Computer))
| summarize Avg_Idle_Time = avg(CounterValue) by Computer
``` 

### <a name="example-4"></a>Exemple 4
Vous souhaitez créer une règle d’alerte de journal avec la requête suivante qui joint les résultats de deux requêtes `search` :

```Kusto
search Type == 'SecurityEvent' and EventID == '4625'
| summarize by Computer, Hour = bin(TimeGenerated, 1h)
| join kind = leftouter (
    search in (Heartbeat) OSType == 'Windows'
    | summarize arg_max(TimeGenerated, Computer) by Computer , Hour = bin(TimeGenerated, 1h)
    | project Hour , Computer
) on Hour
```

Pour modifier la requête, commencez par exécuter la requête suivante afin d’identifier la table qui contient les propriétés figurant du côté gauche de la jointure : 

``` Kusto
search Type == 'SecurityEvent' and EventID == '4625'
| summarize by $table
```

Le résultat indique que les propriétés figurant du côté gauche de la jointure appartiennent à la table _SecurityEvent_. 

Exécutez maintenant la requête suivante afin d’identifier la table qui contient les propriétés figurant du côté droit de la jointure : 
 
``` Kusto
search in (Heartbeat) OSType == 'Windows'
| summarize by $table
```
 
Le résultat indique que les propriétés figurant sur le côté droit de la jointure appartiennent à la table _Heartbeat_.

Vous pouvez utiliser ces résultats pour créer la requête suivante, que vous utiliseriez pour la règle d’alerte : 

``` Kusto
SecurityEvent
| where EventID == '4625'
| summarize by Computer, Hour = bin(TimeGenerated, 1h)
| join kind = leftouter (
    Heartbeat
    | where OSType == 'Windows'
    | summarize arg_max(TimeGenerated, Computer) by Computer , Hour = bin(TimeGenerated, 1h)
    | project Hour , Computer
) on Hour
```

## <a name="next-steps"></a>Étapes suivantes
- Apprenez-en davantage sur les [alertes de journal](alerts-log.md) dans Azure Monitor.
- Apprenez-en davantage sur les [requêtes de journal](../log-query/log-query-overview.md).