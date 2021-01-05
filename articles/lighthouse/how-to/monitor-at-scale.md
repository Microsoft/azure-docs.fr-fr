---
title: Superviser les ressources déléguées à grande échelle
description: Découvrez comment utiliser efficacement les journaux Azure Monitor de manière scalable sur les locataires de clients que vous gérez.
ms.date: 12/14/2020
ms.topic: how-to
ms.openlocfilehash: 6c1cbde696ccf9131797a05db33553b8505216a4
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97509272"
---
# <a name="monitor-delegated-resources-at-scale"></a>Superviser les ressources déléguées à grande échelle

En tant que fournisseur de services, vous avez peut-être intégré les locataires de plusieurs clients à [Azure Lighthouse](../overview.md). Azure Lighthouse permet aux fournisseurs de services d’effectuer des opérations à grande échelle sur plusieurs locataires à la fois, améliorant ainsi l’efficacité des tâches de gestion.

Cette rubrique vous montre comment utiliser les [Journaux Azure Monitor](../../azure-monitor/platform/data-platform-logs.md) de manière scalable sur les locataires de clients que vous gérez. Bien que nous fassions référence aux fournisseurs de services et aux clients dans cette rubrique, ces instructions s’appliquent également aux [entreprises utilisant Azure Lighthouse pour gérer plusieurs locataires](../concepts/enterprise.md).

> [!NOTE]
> Assurez-vous que les utilisateurs de vos locataires gestionnaires disposent des [rôles nécessaires pour la gestion des espaces de travail Log Analytics](../../azure-monitor/platform/manage-access.md#manage-access-using-azure-permissions) sur vos abonnements client délégués.

## <a name="create-log-analytics-workspaces"></a>Créer des espaces de travail Log Analytics

Pour collecter des données, vous devez créer des espaces de travail Log Analytics. Ces espaces de travail Log Analytics sont des environnements uniques pour les données collectées par Azure Monitor. Chaque espace de travail dispose d’un référentiel de données et d’une configuration propres. Les sources de données et les solutions sont configurées de façon à stocker leurs données dans un espace de travail particulier.

Nous vous recommandons de créer ces espaces de travail directement dans les locataires des clients. Ainsi, leurs données restent dans leurs locataires au lieu d’être exportées vers les vôtres. Cela permet également d’effectuer une supervision centralisée de l’ensemble des ressources ou services pris en charge par Log Analytics. Ainsi, vous disposez d’une plus grande flexibilité par rapport aux types de données que vous supervisez.

Vous pouvez créer un espace de travail Log Analytics à l’aide du [portail Azure](../../azure-monitor/learn/quick-create-workspace.md), d’[Azure CLI](../../azure-monitor/learn/quick-create-workspace-cli.md) ou d’[Azure PowerShell](../../azure-monitor/platform/powershell-workspace-configuration.md).

> [!IMPORTANT]
> Même si tous les espaces de travail sont créés dans le locataire client, le fournisseur de ressources Microsoft.Insights doit également être inscrit sur un abonnement dans le locataire gestionnaire.

## <a name="deploy-policies-that-log-data"></a>Déployer des stratégies qui journalisent les données

Une fois que vous avez créé vos espaces de travail Log Analytics, vous pouvez déployer [Azure Policy](../../governance/policy/index.yml) sur les hiérarchies de vos clients afin que les données de diagnostic soient envoyées à l’espace de travail approprié dans chaque locataire. Les stratégies exactes que vous déployez peuvent varier en fonction des types de ressource à superviser.

Pour en savoir plus sur la création de stratégies, consultez [Tutoriel : Créer et gérer des stratégies pour assurer la conformité](../../governance/policy/tutorials/create-and-manage.md). Cet [outil communautaire](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/tools/azure-diagnostics-policy-generator) fournit un script qui vous aide à créer des stratégies permettant de superviser les types de ressource spécifiques de votre choix.

Une fois que vous avez déterminé les stratégies à déployer, vous pouvez [les déployer sur vos abonnements délégués à grande échelle](policy-at-scale.md).

## <a name="analyze-the-gathered-data"></a>Analyser les données collectées

Une fois que vous avez déployé vos stratégies, les données sont journalisées dans les espaces de travail Log Analytics que vous avez créés dans chaque locataire de client. Pour obtenir des insights sur tous les clients managés, vous pouvez utiliser des outils tels que les [classeurs Azure Monitor](../../azure-monitor/platform/workbooks-overview.md) afin de collecter et d’analyser des informations provenant de plusieurs sources de données.

## <a name="view-alerts-across-customers"></a>Afficher les alertes entre les clients

Vous pouvez afficher les [alertes](../../azure-monitor/platform/alerts-overview.md) pour les abonnements délégués dans les locataires client que vous gérez.

Pour actualiser automatiquement les alertes sur plusieurs clients, utilisez une requête [Azure Resource Graph](../../governance/resource-graph/overview.md) pour filtrer les alertes. Vous pouvez épingler la requête à votre tableau de bord et sélectionner tous les clients et abonnements appropriés.

L’exemple de requête suivant affiche les alertes de gravité 0 et 1, qui sont actualisées toutes les 60 minutes.

```kusto
alertsmanagementresources
| where type == "microsoft.alertsmanagement/alerts"
| where properties.essentials.severity =~ "Sev0" or properties.essentials.severity =~ "Sev1"
| where properties.essentials.monitorCondition == "Fired"
| where properties.essentials.startDateTime > ago(60m)
| project StartTime=properties.essentials.startDateTime,name,Description=properties.essentials.description, Severity=properties.essentials.severity, subscriptionId
| sort by tostring(StartTime)
```

## <a name="next-steps"></a>Étapes suivantes

- Explorez cet [exemple de classeur créé par MVP](https://github.com/scautomation/Azure-Automation-Update-Management-Workbooks), qui assure le suivi des rapports de conformité des correctifs en [interrogeant les journaux Update Management](../../automation/update-management/query-logs.md) dans plusieurs espaces de travail Log Analytics. 
- Découvrez [Azure Monitor](../../azure-monitor/index.yml).
- Découvrez les [journaux Azure Monitor](../../azure-monitor/platform/data-platform-logs.md).
- Découvrez les [Expériences de gestion inter-locataire](../concepts/cross-tenant-management-experience.md).