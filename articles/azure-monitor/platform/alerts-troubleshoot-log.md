---
title: Alertes de journal de résolution de problèmes dans Azure Monitor | Microsoft Docs
description: Problèmes, erreurs et résolutions courants pour les règles d’alertes de journal dans Azure.
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.subservice: alerts
ms.date: 10/29/2018
ms.openlocfilehash: e9a1bef582053eccdbfef63c2159cf540ffd9bfb
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186590"
---
# <a name="troubleshoot-log-alerts-in-azure-monitor"></a>Alertes de journal de résolution de problèmes dans Azure Monitor  

Cet article vous explique comment résoudre les problèmes courants avec les alertes de journal dans Azure Monitor. Il propose également des réponses aux problèmes courants liés à la fonctionnalité et à la configuration des alertes de journal.

Les alertes de journal permettent aux utilisateurs d’utiliser une requête [Log Analytics](../log-query/log-analytics-tutorial.md) pour évaluer les journaux de ressources à chaque fréquence définie, et de déclencher une alerte en fonction des résultats. Les règles peuvent déclencher une ou plusieurs actions à l’aide des [groupes d’actions](./action-groups.md). [En savoir plus sur les fonctionnalités et la terminologie des alertes de journal](alerts-unified-log.md).

> [!NOTE]
> Cet article ne couvre pas les cas où le portail Microsoft Azure affiche une règle d’alerte déclenchée et où une notification n’est pas effectuée via un groupe d'actions associé. Dans de tels cas, reportez-vous aux détails relatifs à la résolution des problèmes [ici](./alerts-troubleshoot.md#action-or-notification-on-my-alert-did-not-work-as-expected).

## <a name="log-alert-didnt-fire"></a>L'alerte de journal ne s'est pas déclenchée

### <a name="data-ingestion-time-for-logs"></a>Durée d’ingestion de données pour les journaux d’activité

Azure Monitor traite des téraoctets de journaux de clients du monde entier, ce qui peut entraîner une [latence dans l’ingestion des journaux](./data-ingestion-time.md).

Les journaux sont des données semi-structurées et, par nature, ont une latence plus importante que les métriques. Si vous rencontrez plus de quatre minutes de retard dans les alertes déclenchées, vous devez envisager d’utiliser des [alertes de métrique](alerts-metric-overview.md). Vous pouvez envoyer des données au magasin de métriques à partir de journaux à l’aide d’[alertes de métrique pour les journaux](alerts-metric-logs.md).

Le système réitère l’évaluation de l’alerte plusieurs fois afin de réduire la latence. Une fois les données reçues, l’alerte se déclenche, ce qui, dans la plupart des cas, n’équivaut pas à l’heure d’enregistrement du journal.

### <a name="incorrect-query-time-range-configured"></a>Configuration d’un intervalle de temps de requête incorrect

L’intervalle de temps de requête est défini dans la définition de la condition de règle. Ce champ est appelé **Période** pour les espaces de travail et Application Insights, et **Remplacer l’intervalle de temps de la requête** pour tous les autres types de ressources. Comme dans Log Analytics, l’intervalle de temps limite les données de requête à la période spécifiée. Même si la commande **ago** est utilisée dans la requête, l’intervalle de temps s’applique. 

Par exemple, une requête analyse 60 minutes lorsque l’intervalle de temps est de 60 minutes, et ce, même si le texte contient **ago(1d)** . L’intervalle de temps et le filtrage du temps de requête doivent correspondre. Dans cet exemple, la modification de la **période** / **Remplacer l’intervalle de temps de la requête** à une journée fonctionnera comme prévu.

![Période](media/alert-log-troubleshoot/LogAlertTimePeriod.png)

### <a name="actions-are-muted-in-the-alert-rule"></a>Les actions sont en sourdine dans la règle d’alerte

Les alertes de journal offrent la possibilité de mettre en sourdine les actions d’alerte déclenchées pendant un laps de temps défini. Ce champ est appelé **Supprimer les alertes** dans les espaces de travail et Application Insights. Dans tous les autres types de ressources, il s’agit de **Mettre les actions en sourdine**. 

Un problème courant est que vous pensez que l’alerte n’a pas déclenché les actions en raison d’un problème de service, même si elle a été mise en sourdine par la configuration de la règle.

![Supprimer les alertes](media/alert-log-troubleshoot/LogAlertSuppress.png)

### <a name="metric-measurement-alert-rule-with-splitting-using-the-legacy-log-analytics-api"></a>Règle d’alerte de mesure de métrique avec fractionnement à l’aide de l’API Log Analytics héritée

[Mesure de métrique](alerts-unified-log.md#calculation-of-measure-based-on-a-numeric-column-such-as-cpu-counter-value) est un type d’alerte de journal basé sur un résumé des résultats de séries chronologiques. Ces règles permettent de regrouper par colonnes afin de [fractionner les alertes](alerts-unified-log.md#split-by-alert-dimensions). Si vous utilisez l’API Log Analytics héritée, le fractionnement ne fonctionnera pas comme prévu. Le choix du regroupement dans l’API héritée n’est pas pris en charge.

L’API ScheduledQueryRules actuelle vous permet de définir **Agrégation activée** dans les règles [Mesure de métrique](alerts-unified-log.md#calculation-of-measure-based-on-a-numeric-column-such-as-cpu-counter-value), qui fonctionnent comme prévu. [En savoir plus sur le basculement sur l’API ScheduledQueryRules actuelle](alerts-log-api-switch.md).

## <a name="log-alert-fired-unnecessarily"></a>Alerte de journal déclenchée inutilement

Une [règle d’alerte de journal dans Azure Monitor](./alerts-log.md) configurée peut être déclenchée de façon inattendue. Les sections suivantes décrivent certaines raisons courantes.

### <a name="alert-triggered-by-partial-data"></a>Alerte déclenchée par des données partielles

Azure Monitor traite des téraoctets de journaux de clients du monde entier, ce qui peut entraîner une [latence dans l’ingestion des journaux](./data-ingestion-time.md).

Les journaux sont des données semi-structurées et, par nature, ont une latence plus importante que les métriques. Si vous rencontrez de nombreux déclenchements d’alerte par erreur dans les alertes déclenchées, vous devez envisager d’utiliser des [alertes de métrique](alerts-metric-overview.md). Vous pouvez envoyer des données au magasin de métriques à partir de journaux à l’aide d’[alertes de métrique pour les journaux](alerts-metric-logs.md).

Les alertes de journal fonctionnent mieux lorsque vous tentez de détecter des données dans les journaux. Cela fonctionne moins bien lorsque vous tentez de détecter un manque de données dans les journaux. Par exemple, l’alerte sur la pulsation de la machine virtuelle. 

Bien qu’il existe des capacités intégrées pour prévenir les fausses alertes, elles peuvent toujours se produire sur des données très latentes (plus de 30 minutes) et des données avec des pics de latence.

### <a name="query-optimization-issues"></a>Problèmes d’optimisation des requêtes

Le service d’alerte modifie votre requête pour optimiser la charge et la latence des alertes. Le flux d’alertes a été créé pour transformer les résultats qui indiquent le problème à une alerte. Par exemple, dans le cas d’une requête telle que :

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

Il n’est pas nécessaire d’ajouter une logique d’alerte à la requête et cela peut même poser des problèmes. Dans l’exemple ci-dessus, si vous incluez `count` dans votre requête, la valeur générée sera toujours 1, car le service d’alerte exécutera `count` sur `count`.

La requête optimisée est ce que le service d’alerte de journal exécute. Vous pouvez exécuter la requête modifiée dans le [portail](../log-query/log-query-overview.md) ou l’[API](/rest/api/loganalytics/) Log Analytics.

Pour les espaces de travail et Application Insights, elle est appelée **Requête à exécuter** dans le volet de conditions. Dans tous les autres types de ressources, sélectionnez **Voir la requête d’alerte finale** sous l’onglet Condition.

![Requête à exécuter](media/alert-log-troubleshoot/LogAlertPreview.png)

## <a name="log-alert-was-disabled"></a>L’alerte de journal a été désactivée.

Les sections suivantes mentionnent certaines raisons pour lesquelles Azure Monitor peut désactiver une règle d’alerte de journal. Nous avons également inclus un [exemple du journal d’activité qui est envoyé lorsqu’une règle est désactivée](#activity-log-example-when-rule-is-disabled).

### <a name="alert-scope-no-longer-exists-or-was-moved"></a>L’étendue de l’alerte n’existe plus ou a été déplacée

Lorsque les ressources d’étendue d’une règle d’alerte ne sont plus valides, l’exécution de la règle échoue. Dans ce cas, la facturation s’arrête également.

Azure Monitor désactive l’alerte de journal après une semaine en cas d’échecs continus.

### <a name="query-used-in-a-log-alert-isnt-valid"></a>La requête utilisée dans une alerte de journal n’est pas valide

Lorsqu’une règle d’alerte de journal est créée, la bonne syntaxe de la requête est vérifiée. Cependant, la requête fournie dans la règle d’alerte de journal peut parfois commencer à échouer. Voici quelques raisons courantes :

- Les règles ont été créées via l’API et la validation a été ignorée par l’utilisateur.
- La requête [s’exécute sur plusieurs ressources](../log-query/cross-workspace-query.md) et une ou plusieurs des ressources ont été supprimées ou déplacées.
- La [requête échoue](https://dev.loganalytics.io/documentation/Using-the-API/Errors), car :
    - La solution de journalisation n’a pas été [déployée sur l’espace de travail](../insights/solutions.md#install-a-monitoring-solution), de sorte que les tables ne sont pas créées.
    - Les données ont cessé d’alimenter une table de la requête depuis plus de 30 jours.
    - Les [tables de journaux personnalisés](data-sources-custom-logs.md) n’ont pas encore créées, puisque le flux de données n’a pas démarré.
- Des modifications apportées au [langage de requête](/azure/kusto/query/) incluent un format révisé pour les commandes et les fonctions. Par conséquent, la requête fournie précédemment n’est plus valide.

[Azure Advisor](../../advisor/advisor-overview.md) vous avertit de ce comportement. Le service ajoute une recommandation pour la règle d’alerte de journal concernée. La catégorie utilisée est « Haute disponibilité » avec un impact moyen et la description « Réparer votre règle d’alerte de journal pour garantir la supervision ».

## <a name="alert-rule-quota-was-reached"></a>Le quota de la règle d’alerte a été atteint

Le nombre de règles d’alerte de recherche dans les journaux par abonnement et par ressource est soumis aux limites de quota décrites [ici](../service-limits.md).

### <a name="recommended-steps"></a>Étapes recommandées
    
Si vous avez atteint la limite de quota, les étapes suivantes peuvent vous aider à résoudre le problème.

1. Essayez de supprimer ou de désactiver les règles d’alerte de recherche dans les journaux qui ne sont plus utilisées.
1. Essayez d’utiliser [le fractionnement des alertes par dimensions](alerts-unified-log.md#split-by-alert-dimensions) pour réduire le nombre de règles. Ces règles peuvent analyser de nombreuses ressources et cas de détection.
1. Si vous avez besoin d’augmenter la limite de quota, continuez pour ouvrir une demande de support et fournissez les informations suivantes :

    - ID d’abonnement et ID de ressource pour lesquels la limite de quota doit être relevée.
    - Raison de l’augmentation du quota.
    - Type de ressource pour l’augmentation de quota : **Log Analytics**, **Application Insights**, etc.
    - Limite de quota demandée.


### <a name="to-check-the-current-usage-of-new-log-alert-rules"></a>Pour vérifier l’utilisation actuelle des nouvelles règles d’alerte de journal
    
#### <a name="from-the-azure-portal"></a>À partir du portail Azure

1. Ouvrez l’écran *Alertes*, puis sélectionnez *Gérer les règles d’alerte*.
2. Filtrez sur l’abonnement approprié à l’aide du contrôle de liste déroulante *Abonnement*.
3. Veillez à ne PAS filtrer sur un groupe de ressources, un type de ressource ou une ressource spécifique.
4. Dans le contrôle de liste déroulante *Type de signal*, sélectionnez « Recherche dans les journaux »
5. Vérifiez que le contrôle de liste déroulante *État* a la valeur « Activé ».
6. Le nombre total de règles d’alerte de recherche dans les journaux sera affiché au-dessus de la liste des règles.

#### <a name="from-api"></a>À partir de l’API

- PowerShell - [Get-AzScheduledQueryRule](/powershell/module/az.monitor/get-azscheduledqueryrule)
- API REST - [Répertorier par abonnement](/rest/api/monitor/scheduledqueryrules/listbysubscription)

## <a name="activity-log-example-when-rule-is-disabled"></a>Exemple de journal d’activité lorsque la règle est désactivée

Si la requête échoue pendant sept jours consécutifs, Azure Monitor désactive l’alerte de journal et arrête la facturation de la règle. Vous pouvez déterminer l’heure exacte à laquelle Azure Monitor a désactivé l’alerte de journal dans le [journal d’activité Azure](../../azure-resource-manager/management/view-activity-logs.md). Consultez l’exemple suivant :

```json
{
    "caller": "Microsoft.Insights/ScheduledQueryRules",
    "channels": "Operation",
    "claims": {
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/spn": "Microsoft.Insights/ScheduledQueryRules"
    },
    "correlationId": "abcdefg-4d12-1234-4256-21233554aff",
    "description": "Alert: test-bad-alerts is disabled by the System due to : Alert has been failing consistently with the same exception for the past week",
    "eventDataId": "f123e07-bf45-1234-4565-123a123455b",
    "eventName": {
        "value": "",
        "localizedValue": ""
    },
    "category": {
        "value": "Administrative",
        "localizedValue": "Administrative"
    },
    "eventTimestamp": "2019-03-22T04:18:22.8569543Z",
    "id": "/SUBSCRIPTIONS/<subscriptionId>/RESOURCEGROUPS/<ResourceGroup>/PROVIDERS/MICROSOFT.INSIGHTS/SCHEDULEDQUERYRULES/TEST-BAD-ALERTS",
    "level": "Informational",
    "operationId": "",
    "operationName": {
        "value": "Microsoft.Insights/ScheduledQueryRules/disable/action",
        "localizedValue": "Microsoft.Insights/ScheduledQueryRules/disable/action"
    },
    "resourceGroupName": "<Resource Group>",
    "resourceProviderName": {
        "value": "MICROSOFT.INSIGHTS",
        "localizedValue": "Microsoft Insights"
    },
    "resourceType": {
        "value": "MICROSOFT.INSIGHTS/scheduledqueryrules",
        "localizedValue": "MICROSOFT.INSIGHTS/scheduledqueryrules"
    },
    "resourceId": "/SUBSCRIPTIONS/<subscriptionId>/RESOURCEGROUPS/<ResourceGroup>/PROVIDERS/MICROSOFT.INSIGHTS/SCHEDULEDQUERYRULES/TEST-BAD-ALERTS",
    "status": {
        "value": "Succeeded",
        "localizedValue": "Succeeded"
    },
    "subStatus": {
        "value": "",
        "localizedValue": ""
    },
    "submissionTimestamp": "2019-03-22T04:18:22.8569543Z",
    "subscriptionId": "<SubscriptionId>",
    "properties": {
        "resourceId": "/SUBSCRIPTIONS/<subscriptionId>/RESOURCEGROUPS/<ResourceGroup>/PROVIDERS/MICROSOFT.INSIGHTS/SCHEDULEDQUERYRULES/TEST-BAD-ALERTS",
        "subscriptionId": "<SubscriptionId>",
        "resourceGroup": "<ResourceGroup>",
        "eventDataId": "12e12345-12dd-1234-8e3e-12345b7a1234",
        "eventTimeStamp": "03/22/2019 04:18:22",
        "issueStartTime": "03/22/2019 04:18:22",
        "operationName": "Microsoft.Insights/ScheduledQueryRules/disable/action",
        "status": "Succeeded",
        "reason": "Alert has been failing consistently with the same exception for the past week"
    },
    "relatedEvents": []
}
```

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [alertes de journal dans Azure](./alerts-unified-log.md).
- En savoir plus sur la [configuration des alertes de journal](../log-query/log-query-overview.md).
- Découvrez plus en détail les [requêtes dans les journaux](../log-query/log-query-overview.md).