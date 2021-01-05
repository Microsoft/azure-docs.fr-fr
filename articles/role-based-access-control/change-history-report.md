---
title: Afficher les journaux d’activité pour voir les changements RBAC Azure
description: Affichez les journaux d’activité des changements de contrôle d’accès en fonction du rôle Azure (RBAC Azure) apportés aux ressources Azure au cours des 90 derniers jours.
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: 2bc68595-145e-4de3-8b71-3a21890d13d9
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/27/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.custom: H1Hack27Feb2017, devx-track-azurecli
ms.openlocfilehash: 3e4d75c59d0e3c97b0900dd72a27d4afbf4015c9
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492316"
---
# <a name="view-activity-logs-for-azure-rbac-changes"></a>Afficher les journaux d’activité pour voir les changements RBAC Azure

Vous avez parfois besoin d’informations sur les modifications du contrôle d’accès en fonction du rôle Azure (RBAC Azure), par exemple à des fins d’audit ou de dépannage. Quand un utilisateur apporte des changements à des attributions ou des définitions de rôle au sein de vos abonnements, ceux-ci sont journalisés dans la catégorie [Journal d’activité Azure](../azure-monitor/platform/platform-logs-overview.md). Vous pouvez afficher les journaux d’activité pour voir tous les changements RBAC Azure des 90 derniers jours.

## <a name="operations-that-are-logged"></a>Opérations journalisées

Voici les opérations RBAC Azure qui sont journalisées dans le journal d’activité :

- Créer une attribution de rôle
- Supprimer une attribution de rôle
- Créer ou mettre à jour une définition de rôle personnalisée
- Supprimer la définition de rôle personnalisée

## <a name="azure-portal"></a>Portail Azure

Pour commencer, le plus simple consiste à afficher les journaux d’activité avec le portail Azure. La capture d’écran suivante montre un exemple d’opérations d’attribution de rôle dans le journal d’activité. Elle contient également une option pour télécharger les journaux dans un fichier CSV.

![Journaux d’activité à l’aide du portail : capture d’écran](./media/change-history-report/activity-log-portal.png)

Le journal d’activité dans le portail comporte plusieurs filtres. Voici les filtres RBAC Azure :

| Filtrer | Valeur |
| --------- | --------- |
| Catégorie d'événements | <ul><li>Administratif</li></ul> |
| Opération | <ul><li>Créer une attribution de rôle</li><li>Supprimer une attribution de rôle</li><li>Créer ou mettre à jour une définition de rôle personnalisée</li><li>Supprimer la définition de rôle personnalisée</li></ul> |

Pour plus d’informations sur les journaux d’activité, consultez [Afficher les journaux d’activité pour surveiller les actions sur les ressources](../azure-resource-manager/management/view-activity-logs.md?toc=%2fazure%2fmonitoring-and-diagnostics%2ftoc.json).

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [az-powershell-update](../../includes/updated-for-az.md)]

Pour afficher les journaux d’activité avec Azure PowerShell, utilisez la commande [Get-AzLog](/powershell/module/Az.Monitor/Get-AzLog).

Cette commande liste tous les changements d’attribution de rôle dans un abonnement au cours des sept derniers jours :

```azurepowershell
Get-AzLog -StartTime (Get-Date).AddDays(-7) | Where-Object {$_.Authorization.Action -like 'Microsoft.Authorization/roleAssignments/*'}
```

Cette commande liste tous les changements de définition de rôle dans un groupe de ressources au cours des sept derniers jours :

```azurepowershell
Get-AzLog -ResourceGroupName pharma-sales -StartTime (Get-Date).AddDays(-7) | Where-Object {$_.Authorization.Action -like 'Microsoft.Authorization/roleDefinitions/*'}
```

Cette commande liste tous les changements d’attribution de rôle et de définition de rôle dans un abonnement au cours des sept derniers jours et affiche les résultats dans une liste :

```azurepowershell
Get-AzLog -StartTime (Get-Date).AddDays(-7) | Where-Object {$_.Authorization.Action -like 'Microsoft.Authorization/role*'} | Format-List Caller,EventTimestamp,{$_.Authorization.Action},Properties
```

```Example
Caller                  : alain@example.com
EventTimestamp          : 2/27/2020 9:18:07 PM
$_.Authorization.Action : Microsoft.Authorization/roleAssignments/write
Properties              :
                          statusCode     : Created
                          serviceRequestId: 11111111-1111-1111-1111-111111111111
                          eventCategory  : Administrative

Caller                  : alain@example.com
EventTimestamp          : 2/27/2020 9:18:05 PM
$_.Authorization.Action : Microsoft.Authorization/roleAssignments/write
Properties              :
                          requestbody    : {"Id":"22222222-2222-2222-2222-222222222222","Properties":{"PrincipalId":"33333333-3333-3333-3333-333333333333","RoleDefinitionId":"/subscriptions/00000000-0000-0000-0000-000000000000/providers
                          /Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c","Scope":"/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/pharma-sales"}}

```

Si vous utilisez un principal de service pour créer des attributions de rôles, la propriété Appelant est un ID d’objet. Vous pouvez utiliser [Get-AzADServicePrincipal](/powershell/module/az.resources/get-azadserviceprincipal) pour obtenir des informations sur le principal du service.

```Example
Caller                  : 44444444-4444-4444-4444-444444444444
EventTimestamp          : 6/4/2020 9:43:08 PM
$_.Authorization.Action : Microsoft.Authorization/roleAssignments/write
Properties              : 
                          statusCode     : Created
                          serviceRequestId: 55555555-5555-5555-5555-555555555555
                          category       : Administrative
```

## <a name="azure-cli"></a>Azure CLI

Pour afficher les journaux d’activité avec l’interface de ligne de commande Azure, utilisez la commande [az monitor activity-log list](/cli/azure/monitor/activity-log#az-monitor-activity-log-list).

Cette commande liste les journaux d’activité d’un groupe de ressources à partir du 27 février sur les sept jours suivants :

```azurecli
az monitor activity-log list --resource-group pharma-sales --start-time 2020-02-27 --offset 7d
```

Cette commande liste les journaux d’activité d’un fournisseur de ressources Authorization à partir du 27 février sur les sept jours suivants :

```azurecli
az monitor activity-log list --namespace "Microsoft.Authorization" --start-time 2020-02-27 --offset 7d
```

## <a name="azure-monitor-logs"></a>Journaux d’activité Azure Monitor

Les [journaux Azure Monitor](../azure-monitor/log-query/log-query-overview.md) sont un autre outil que vous pouvez utiliser pour collecter et analyser les changements RBAC Azure de toutes vos ressources Azure. Les journaux Azure Monitor possèdent les avantages suivants :

- Écriture de requêtes et d’une logique complexes
- Intégration aux alertes, à Power BI et à d’autres outils
- Enregistrement des données pour des périodes de rétention plus longues
- Références croisées à d’autres journaux d’activité, tels que ceux liés à la sécurité, aux machines virtuelles et aux journaux d’activité personnalisés

Voici les étapes de base pour bien démarrer :

1. [Créez un espace de travail Log Analytics](../azure-monitor/learn/quick-create-workspace.md).

1. [Configurez la solution Activity Log Analytics](../azure-monitor/platform/activity-log.md#activity-log-analytics-monitoring-solution) pour vos espaces de travail.

1. [Affichez les journaux d’activité](../azure-monitor/platform/activity-log.md#activity-log-analytics-monitoring-solution). Un moyen rapide d’accéder à la page Vue d’ensemble de la solution Activity Log Analytics consiste à cliquer sur l’option **Journaux**.

   ![Option journaux Azure Monitor dans le portail](./media/change-history-report/azure-log-analytics-option.png)

1. Si vous le souhaitez, vous pouvez utiliser [Azure Monitor Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md) pour interroger et afficher les journaux. Pour plus d’informations, consultez [Bien démarrer avec les requêtes de journal Azure Monitor](../azure-monitor/log-query/get-started-queries.md).

Voici une requête qui retourne les nouvelles attributions de rôle organisées par fournisseur de ressources cible :

```Kusto
AzureActivity
| where TimeGenerated > ago(60d) and Authorization contains "Microsoft.Authorization/roleAssignments/write" and ActivityStatus == "Succeeded"
| parse ResourceId with * "/providers/" TargetResourceAuthProvider "/" *
| summarize count(), makeset(Caller) by TargetResourceAuthProvider
```

Voici une requête qui retourne des changements d’attribution de rôle affichés dans un graphique :

```Kusto
AzureActivity
| where TimeGenerated > ago(60d) and Authorization contains "Microsoft.Authorization/roleAssignments"
| summarize count() by bin(TimeGenerated, 1d), OperationName
| render timechart
```

![Journaux d’activité à l’aide du portail Analytique avancée : capture d’écran](./media/change-history-report/azure-log-analytics.png)

## <a name="next-steps"></a>Étapes suivantes
* [Afficher des événements dans le journal d’activité](../azure-resource-manager/management/view-activity-logs.md?toc=%2fazure%2fmonitoring-and-diagnostics%2ftoc.json)
* [Surveiller l’activité d’abonnement avec le journal d’activité Azure](../azure-monitor/platform/platform-logs-overview.md)