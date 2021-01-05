---
title: Découvrez comment l’approvisionnement s’intègre aux journaux Azure Monitor dans Azure Active Directory.
description: Découvrez comment l’approvisionnement s’intègre aux journaux Azure Monitor dans Azure Active Directory.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: conceptual
ms.workload: identity
ms.date: 10/12/2020
ms.author: kenwith
ms.reviewer: arvinh,luleon
ms.openlocfilehash: 167ed7e5c00452db4ee77e10236fec3ff86f0439
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96175098"
---
# <a name="understand-how-provisioning-integrates-with-azure-monitor-logs"></a>Découvrez comment l’approvisionnement s’intègre aux journaux Azure Monitor

L’approvisionnement s’intègre aux journaux Azure Monitor et à Log Analytics. Grâce à l’analyse Azure, vous pouvez effectuer diverses opérations, comme créer des classeurs, également appelés tableaux de bord, stocker des journaux d’approvisionnement pendant plus de 30 jours et créer des requêtes et des alertes personnalisées. Cet article explique comment l’approvisionnement des journaux est intégré aux journaux Azure Monitor. Pour en savoir plus sur le fonctionnement général des journaux d’approvisionnement, consultez [Journaux d’approvisionnement](../reports-monitoring/concept-provisioning-logs.md).

## <a name="enabling-provisioning-logs"></a>Activation des journaux d’approvisionnement

Vous devez déjà avoir une connaissance de Log Analytics et de l’analyse Azure. Si ce n’est pas le cas, apprenez-en plus sur le sujet, puis revenez sur cette page pour découvrir les journaux d’approvisionnement d’application. Pour en savoir plus sur l’analyse Azure, consultez [Vue d’ensemble d’Azure Monitor](../../azure-monitor/overview.md). Pour en savoir plus sur les journaux Azure Monitor et Log Analytics, consultez [Vue d’ensemble des requêtes de journal dans Azure Monitor](../../azure-monitor/log-query/log-query-overview.md).

Une fois que vous avez configuré l’analyse Azure, vous pouvez activer les journaux pour l’approvisionnement des applications. L’option se trouve sur la page **Paramètres de diagnostic**.

:::image type="content" source="media/application-provisioning-log-analytics/diagnostic-settings.png" alt-text="Accéder aux paramètres de diagnostic" lightbox="media/application-provisioning-log-analytics/diagnostic-settings.png":::

:::image type="content" source="media/application-provisioning-log-analytics/enable-log-analytics.png" alt-text="Activer les journaux d’approvisionnement d’application" lightbox="media/application-provisioning-log-analytics/enable-log-analytics.png":::

> [!NOTE]
> Si vous venez d’approvisionner un espace de travail, il peut falloir un certain temps avant de pouvoir y envoyer des journaux. Si vous recevez une erreur indiquant que l’abonnement n’est pas inscrit pour utiliser *microsoft.insights* revenez après quelques minutes.
 
## <a name="understanding-the-data"></a>Vue d’ensemble des données
Le flux de données sous-jacent que l’approvisionnement envoie aux visionneuses de journaux est presque identique. Les journaux Azure Monitor suivent quasiment le même flux que l’interface utilisateur du Portail Azure et l’API Azure. Il n’existe que quelques **différences** dans les champs du journal, comme indiqué dans le tableau suivant. Pour en savoir plus sur ces champs, consultez [Répertorier provisioningObjectSummary](/graph/api/provisioningobjectsummary-list?preserve-view=true&tabs=http&view=graph-rest-beta).

|Journaux d’activité Azure Monitor   |Interface utilisateur du portail Azure   |API Azure |
|----------|-----------|------------|
|errorDescription |reason |resultDescription |
|status |resultType |resultType |
|activityDateTime |TimeGenerated |TimeGenerated |


## <a name="azure-monitor-workbooks"></a>Classeurs Azure Monitor

Les classeurs Azure Monitor fournissent un canevas flexible pour l’analyse des données. Ils permettent également de créer des rapports visuels enrichis au sein du Portail Azure. Pour plus d’informations, consultez [Vue d’ensemble des classeurs Azure Monitor](../../azure-monitor/platform/workbooks-overview.md).

L’approvisionnement d’applications est fourni avec un ensemble de classeurs prédéfinis. Vous pouvez les trouver dans la page Classeurs. Pour afficher les données, vous devez vous assurer que tous les filtres (timeRange, jobID, appName) sont remplis. Vous devez également vous assurer que vous avez configuré une application, sinon aucune donnée n’est disponible dans les journaux.

:::image type="content" source="media/application-provisioning-log-analytics/workbooks.png" alt-text="Classeurs d’approvisionnement d’application" lightbox="media/application-provisioning-log-analytics/workbooks.png":::

:::image type="content" source="media/application-provisioning-log-analytics/report.png" alt-text="Tableau de bord d’approvisionnement d’applications" lightbox="media/application-provisioning-log-analytics/report.png":::

## <a name="custom-queries"></a>Requêtes personnalisées

Vous pouvez créer des requêtes personnalisées et afficher les données dans des tableaux de bord Azure. Pour savoir comment procéder, consultez [Créer et partager des tableaux de bord de données Log Analytics](../../azure-monitor/log-query/get-started-queries.md). Veillez également à consulter [Vue d’ensemble des requêtes de journal dans Azure Monitor](../../azure-monitor/log-query/log-query-overview.md).

Voici quelques exemples pour prendre en main l’approvisionnement d’applications.

Interroger les journaux d’un utilisateur en fonction de son ID dans le système source :
```kusto
AADProvisioningLogs
| extend SourceIdentity = parse_json(SourceIdentity)
| where tostring(SourceIdentity.Id) == "49a4974bb-5011-415d-b9b8-78caa7024f9a"
```

Total par ErrorCode :
```kusto
AADProvisioningLogs
| summarize count() by ErrorCode = ResultSignature
```

Total d’événements par jour et par action :
```kusto
AADProvisioningLogs
| where TimeGenerated > ago(7d)
| summarize count() by Action, bin(TimeGenerated, 1d)
```

Prenez 100 événements et les propriétés de clé du projet :
```kusto
AADProvisioningLogs
| extend SourceIdentity = parse_json(SourceIdentity)
| extend TargetIdentity = parse_json(TargetIdentity)
| extend ServicePrincipal = parse_json(ServicePrincipal)
| where tostring(SourceIdentity.identityType) == "Group"
| project tostring(ServicePrincipal.Id), tostring(ServicePrincipal.Name), ModifiedProperties, JobId, Id, CycleId, ChangeId, Action, SourceIdentity.identityType, SourceIdentity.details, TargetIdentity.identityType, TargetIdentity.details, ProvisioningSteps
|take 100
```

## <a name="custom-alerts"></a>Alertes personnalisées

Azure Monitor vous permet de configurer des alertes personnalisées afin de recevoir des notifications sur les principaux événements liés à l’approvisionnement. Par exemple, vous souhaiterez peut-être recevoir une alerte en cas de défaillance. Ou peut-être en cas de pics de désactivation ou de suppression. Un autre exemple d’alerte que vous pourriez vouloir recevoir est l’absence d’approvisionnement, qui indique un problème.

Pour en savoir plus sur les alertes, consultez [Répondre aux événements avec les alertes Azure Monitor](../../azure-monitor/learn/tutorial-response.md).

Alerte en cas de pic de défaillances. Remplacez jobID par le jobID de votre application.

:::image type="content" source="media/application-provisioning-log-analytics/alert1.png" alt-text="Alerte en cas de pic de défaillances." lightbox="media/application-provisioning-log-analytics/alert1.png":::

Il peut y avoir un problème qui a provoqué l’arrêt de l’exécution du service d’approvisionnement. Utilisez l’alerte suivante pour détecter quand il n’y a pas d’événements d’approvisionnement pendant un intervalle de temps donné.

:::image type="content" source="media/application-provisioning-log-analytics/alert2.png" alt-text="Il peut y avoir un problème qui a provoqué l’arrêt de l’exécution du service d’approvisionnement." lightbox="media/application-provisioning-log-analytics/alert2.png":::

Alerte en cas de pic de désactivations ou de suppressions.

:::image type="content" source="media/application-provisioning-log-analytics/alert3.png" alt-text="Alerte en cas de pic de désactivations ou de suppressions." lightbox="media/application-provisioning-log-analytics/alert3.png":::


## <a name="community-contributions"></a>Contributions de la communauté

Nous adoptons une approche open source et basée sur la communauté pour les requêtes et les tableaux de bord d’approvisionnement des applications. Si vous avez créé une requête, une alerte ou un classeur et que vous estimez que d’autres pourraient les trouver utiles, publiez-les sur le [référentiel GitHub d’AzureMonitorCommunity](https://github.com/microsoft/AzureMonitorCommunity). Envoyez-nous un e-mail avec un lien. Nous le passerons en revue et le publierons dans le service afin que d’autres puissent en bénéficier. Vous pouvez nous contacter à l’adresse provisioningfeedback@microsoft.com.

## <a name="next-steps"></a>Étapes suivantes

- [Analytique des journaux d'activité](../reports-monitoring/howto-analyze-activity-logs-log-analytics.md)
- [Bien démarrer avec les requêtes de journal d’activité Azure Monitor](../../azure-monitor/log-query/get-started-queries.md)
- [Créer et gérer des groupes d’alertes dans le portail Azure](../../azure-monitor/platform/action-groups.md)
- [Installer et utiliser les vues Log Analytics pour Azure Active Directory](../reports-monitoring/howto-install-use-log-analytics-views.md)
- [API de journaux de provisionnement](/graph/api/resources/provisioningobjectsummary?preserve-view=true&view=graph-rest-beta.md)