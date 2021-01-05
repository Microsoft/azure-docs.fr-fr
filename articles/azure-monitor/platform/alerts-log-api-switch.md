---
title: Mise à niveau vers l'API Alertes de journal actuelle d'Azure Monitor
description: Apprenez à basculer vers l'API ScheduledQueryRules des alertes de journal
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.date: 05/30/2019
ms.subservice: alerts
ms.openlocfilehash: 868a8eb6cf38d471eb9dc1f47c903404d05ffc0c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91294510"
---
# <a name="upgrade-to-the-current-log-alerts-api-from-legacy-log-analytics-alert-api"></a>Mise à niveau vers l'API Alertes de journal actuelle à partir de l'API Alerte héritée de Log Analytics

> [!NOTE]
> Cet article concerne uniquement Azure public (**et non** Azure Government ou le cloud Azure Chine).

> [!NOTE]
> Une fois qu'un utilisateur choisit de changer de préférence pour adopter l'[API scheduledQueryRules](/rest/api/monitor/scheduledqueryrules) actuelle, il est impossible de revenir à l'ancienne [API Alerte héritée de Log Analytics](api-alerts.md).

Autrefois, les utilisateurs utilisaient l'[API Alerte héritée de Log Analytics](api-alerts.md) pour gérer les règles d'alerte de journal. Les espaces de travail actuels utilisent l'[API ScheduledQueryRules](/rest/api/monitor/scheduledqueryrules). Cet article décrit le processus de basculement de l'API héritée vers l'API actuelle et les avantages de celui-ci.

## <a name="benefits"></a>Avantages

- Modèle unique pour la création de règles d'alerte (auparavant, trois modèles distincts étaient nécessaires).
- API unique pour les espaces de travail Log Analytics ou les ressources Application Insights.
- [Prise en charge des cmdlets PowerShell](alerts-log.md#managing-log-alerts-using-powershell).
- Alignement des niveaux de gravité avec tous les autres types d'alertes.
- Possibilité de créer des [alertes de journal inter-espaces de travail](../log-query/cross-workspace-query.md) couvrant différentes ressources externes telles que les espaces de travail Log Analytics ou les ressources Application Insights.
- Les utilisateurs peuvent spécifier des dimensions pour fractionner les alertes à l'aide du paramètre « Agréger sur ».
- Les alertes de journal disposent d'une période prolongée allant jusqu'à deux jours de données (celle-ci était auparavant limitée à un jour).

## <a name="impact"></a>Impact

- Les nouvelles règles doivent toutes être créées/modifiées avec l'API actuelle. Consultez [Exemple d'utilisation via Azure Resource Template](alerts-log-create-templates.md) et [Exemple d'utilisation via PowerShell](alerts-log.md#managing-log-alerts-using-powershell).
- Comme les règles deviennent des ressources suivies par Azure Resource Manager dans l'API actuelle et qu'elles doivent être uniques, l'ID de ressource des règles adopte la structure suivante : `<WorkspaceName>|<savedSearchId>|<scheduleId>|<ActionId>`. Les noms d'affichage de la règle d'alerte restent inchangés.

## <a name="process"></a>Process

Dans la plupart des cas, le processus de basculement n'est pas interactif et ne nécessite pas d'étapes manuelles. Vos règles d'alerte ne sont pas arrêtées ou bloquées, ni pendant ni après le basculement.
Passez cet appel pour basculer toutes les règles d'alerte associées à l'espace de travail Log Analytics spécifique :

```
PUT /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/alertsversion?api-version=2017-04-26-preview
```

Avec un corps de la demande qui contient le code JSON ci-dessous :

```json
{
    "scheduledQueryRulesEnabled" : true
}
```

Voici un exemple d'utilisation d'[ARMClient](https://github.com/projectkudu/ARMClient), un outil de ligne de commande open source qui simplifie l'appel d'API ci-dessus :

```powershell
$switchJSON = '{"scheduledQueryRulesEnabled": "true"}'
armclient PUT /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/alertsversion?api-version=2017-04-26-preview $switchJSON
```

Si le basculement est un succès, la réponse est :

```json
{
    "version": 2,
    "scheduledQueryRulesEnabled" : true
}
```

## <a name="check-switching-status-of-workspace"></a>Vérifier l'état du basculement de l'espace de travail

Vous pouvez également utiliser cet appel d'API pour vérifier l'état du basculement :

```
GET /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/alertsversion?api-version=2017-04-26-preview
```

Vous pouvez aussi utiliser l'outil [​​ARMClient](https://github.com/projectkudu/ARMClient) :

```powershell
armclient GET /subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>/alertsversion?api-version=2017-04-26-preview
```

Si l'espace de travail Log Analytics a été basculé vers l'[API SchedulerQueryRules](/rest/api/monitor/scheduledqueryrules), la réponse est :

```json
{
    "version": 2,
    "scheduledQueryRulesEnabled" : true
}
```
Si l'espace de travail Log Analytics n'a pas été basculé, la réponse est :

```json
{
    "version": 2,
    "scheduledQueryRulesEnabled" : false
}
```

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [Azure Monitor - Alertes de journal](alerts-unified-log.md).
- En savoir plus sur [la gestion de vos alertes de journal à l'aide de l'API](alerts-log-create-templates.md).
- En savoir plus sur [la gestion des alertes de journal à l'aide de PowerShell](alerts-log.md#managing-log-alerts-using-powershell).
- En savoir plus sur l’[expérience Alertes Azure](./alerts-overview.md).
