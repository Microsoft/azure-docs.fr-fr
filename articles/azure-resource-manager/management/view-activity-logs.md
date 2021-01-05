---
title: Afficher les journaux d’activité Azure pour surveiller les ressources
description: Utilisez les journaux d’activité pour passer en revue les actions et les erreurs des utilisateurs. Affiche le Portail Microsoft Azure, PowerShell, l’interface de ligne de commande Azure et REST.
ms.topic: conceptual
ms.date: 05/13/2019
ms.openlocfilehash: 14015e9b2792515e6818af551b8bd9f54c686bee
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91371590"
---
# <a name="view-activity-logs-to-monitor-actions-on-resources"></a>Afficher les journaux d’activité pour surveiller les actions sur les ressources

Les journaux d’activité vous permettent de déterminer :

* Les opérations qui ont été effectuées sur les ressources de votre abonnement
* Qui a démarré l'opération
* Le moment où a eu lieu l’opération
* L’état de l’opération
* Les valeurs d’autres propriétés qui peuvent vous aider à effectuer des recherches sur l’opération

Le journal d’activité contient toutes les opérations d’écriture (PUT, POST, DELETE) de vos ressources. Il n'inclut pas les opérations de lecture (GET). Pour obtenir la liste des actions de ressource, consultez [Opérations du fournisseur de ressources Azure](../../role-based-access-control/resource-provider-operations.md). Vous pouvez utiliser les journaux d’activité pour rechercher une erreur lors de la résolution de problèmes ou pour surveiller la manière dont un utilisateur de votre organisation modifie une ressource.

Les journaux d’activité sont conservés pendant 90 jours. Vous pouvez interroger n'importe quelle plage de dates, à condition que la date de début ne remonte pas à plus de 90 jours.

Vous pouvez récupérer des informations dans les journaux d’activité par le biais du portail, de PowerShell, de l’interface de ligne de commande Azure, de l’API REST Insights ou de [Insights .NET Library](https://www.nuget.org/packages/Microsoft.Azure.Insights/).

## <a name="azure-portal"></a>Portail Azure

Pour afficher les journaux d’activité via le portail, procédez comme suit :

1. Dans le menu du Portail Azure, sélectionnez **Surveiller** ou recherchez et sélectionnez **Surveiller** dans n’importe quelle page.

    ![Sélectionner Surveiller](./media/view-activity-logs/select-monitor-from-menu.png)

1. Sélectionnez **Journal d'activité**.

    ![Sélectionner un journal d’activité](./media/view-activity-logs/select-activity-log.png)

1. Un résumé des opérations récentes s'affiche. Un ensemble de filtres par défaut est appliqué aux opérations. Notez que les informations sur le résumé incluent la personne qui a démarré l’action ainsi que le moment d’exécution de celle-ci.

    ![Afficher le résumé des opérations récentes](./media/view-activity-logs/audit-summary.png)

1. Pour exécuter rapidement un ensemble de filtres prédéfini, sélectionnez **Quick Insights**.

    ![Sélectionner Quick Insights](./media/view-activity-logs/select-quick-insights.png)

1. Sélectionnez l'une des options. Par exemple, sélectionnez **Déploiements ayant échoué** pour afficher les erreurs des déploiements.

    ![Sélectionner les déploiements ayant échoué](./media/view-activity-logs/select-failed-deployments.png)

1. Notez que les filtres ont été modifiés pour se concentrer sur les erreurs de déploiement des dernières 24 heures. Seules les opérations qui correspondent aux filtres sont affichées.

    ![Filtres de vue](./media/view-activity-logs/view-filters.png)

1. Pour vous concentrer sur des opérations spécifiques, changez les filtres ou appliquez-en de nouveaux. Par exemple, l'image suivante présente une nouvelle valeur pour l'**Intervalle de temps**, et le **Type de ressource** est défini sur des comptes de stockage.

    ![Définir des options de filtre](./media/view-activity-logs/set-filter.png)

1. Si vous avez ultérieurement besoin de réexécuter la requête, sélectionnez **Épingler les filtres actuels**.

    ![Épingler des filtres](./media/view-activity-logs/pin-filters.png)

1. Donnez un nom au filtre.

    ![Filtres de nom](./media/view-activity-logs/name-filters.png)

1. Le filtre est disponible dans le tableau de bord. Dans le menu du Portail Azure, sélectionnez **Tableau de bord**.

    ![Afficher le filtre sur le tableau de bord](./media/view-activity-logs/activity-log-on-dashboard.png)

1. Dans le portail, vous pouvez afficher les modifications apportées à une ressource. Revenez à l’affichage par défaut dans Monitor, puis sélectionnez une opération qui impliquait la modification d’une ressource.

    ![Sélectionner une opération](./media/view-activity-logs/select-operation.png)

1. Sélectionnez **Historique des modifications (préversion)** et choisissez une des opérations disponibles.

    ![Sélectionner l’historique des modifications](./media/view-activity-logs/select-change-history.png)

1. Les modifications apportées à la ressource sont affichées.

    ![Afficher les modifications](./media/view-activity-logs/show-changes.png)

Pour en savoir plus sur l’historique des modifications, consultez [Obtenir les modifications des ressources](../../governance/resource-graph/how-to/get-resource-changes.md).

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Pour récupérer les entrées de journal, exécutez la commande **Get-AzLog** . Vous spécifiez des paramètres supplémentaires pour filtrer la liste des entrées. Si vous ne spécifiez pas les heures de début et de fin, les entrées des sept derniers jours sont renvoyées.

```azurepowershell-interactive
Get-AzLog -ResourceGroup ExampleGroup
```

L’exemple suivant montre comment utiliser le journal d’activité pour rechercher les opérations effectuées pendant une période spécifique. Les dates de début et de fin sont indiquées dans un format de date.

```azurepowershell-interactive
Get-AzLog -ResourceGroup ExampleGroup -StartTime 2019-05-05T06:00 -EndTime 2019-05-09T06:00
```

Vous pouvez également utiliser les fonctions de date pour spécifier la plage de dates, par exemple, les 14 derniers jours.

```azurepowershell-interactive
Get-AzLog -ResourceGroup ExampleGroup -StartTime (Get-Date).AddDays(-14)
```

Vous pouvez rechercher les actions exécutées par un utilisateur spécifique.

```azurepowershell-interactive
Get-AzLog -ResourceGroup ExampleGroup -StartTime (Get-Date).AddDays(-14) -Caller someone@contoso.com
```

Vous pouvez filtrer les résultats sur les opérations ayant échoué.

```azurepowershell-interactive
Get-AzLog -ResourceGroup ExampleGroup -Status Failed
```

Vous pouvez vous focaliser sur une erreur en examinant le message d’état pour cette entrée.

```azurepowershell-interactive
(Get-AzLog -ResourceGroup ExampleGroup -Status Failed).Properties.Content.statusMessage | ConvertFrom-Json
```

Vous pouvez sélectionner des valeurs spécifiques pour limiter les données renvoyées.

```azurepowershell-interactive
Get-AzLog -ResourceGroupName ExampleGroup | Format-table EventTimeStamp, Caller, @{n='Operation'; e={$_.OperationName.value}}, @{n='Status'; e={$_.Status.value}}, @{n='SubStatus'; e={$_.SubStatus.LocalizedValue}}
```

En fonction de l’heure de début que vous spécifiez, les commandes précédentes peuvent retourner une longue liste d’opérations pour le groupe de ressources. Vous pouvez filtrer les résultats de votre recherche en fournissant des critères de recherche. Par exemple, vous pouvez filtrer par type d'opération.

```azurepowershell-interactive
Get-AzLog -ResourceGroup ExampleGroup | Where-Object {$_.OperationName.value -eq "Microsoft.Resources/deployments/write"}
```

Vous pouvez utiliser Resource Graph pour afficher l’historique des modifications d’une ressource. Pour plus d’informations, consultez [Obtenir les modifications des ressources](../../governance/resource-graph/how-to/get-resource-changes.md).

## <a name="azure-cli"></a>Azure CLI

Pour récupérer des entrées de journal, exécutez la commande [az monitor activity-log list](/cli/azure/monitor/activity-log#az-monitor-activity-log-list) avec un décalage pour indiquer la période.

```azurecli-interactive
az monitor activity-log list --resource-group ExampleGroup --offset 7d
```

L’exemple suivant montre comment utiliser le journal d’activité pour rechercher les opérations effectuées pendant une période spécifique. Les dates de début et de fin sont indiquées dans un format de date.

```azurecli-interactive
az monitor activity-log list -g ExampleGroup --start-time 2019-05-01 --end-time 2019-05-15
```

Vous pouvez rechercher les actions effectuées par un utilisateur particulier, même pour un groupe de ressources qui n’existe plus.

```azurecli-interactive
az monitor activity-log list -g ExampleGroup --caller someone@contoso.com --offset 5d
```

Vous pouvez filtrer les résultats sur les opérations ayant échoué.

```azurecli-interactive
az monitor activity-log list -g ExampleGroup --status Failed --offset 1d
```

Vous pouvez vous focaliser sur une erreur en examinant le message d’état pour cette entrée.

```azurecli-interactive
az monitor activity-log list -g ExampleGroup --status Failed --offset 1d --query [].properties.statusMessage
```

Vous pouvez sélectionner des valeurs spécifiques pour limiter les données renvoyées.

```azurecli-interactive
az monitor activity-log list -g ExampleGroup --offset 1d --query '[].{Operation: operationName.value, Status: status.value, SubStatus: subStatus.localizedValue}'
```

En fonction de l’heure de début que vous spécifiez, les commandes précédentes peuvent retourner une longue liste d’opérations pour le groupe de ressources. Vous pouvez filtrer les résultats de votre recherche en fournissant des critères de recherche. Par exemple, vous pouvez filtrer par type d'opération.

```azurecli-interactive
az monitor activity-log list -g ExampleGroup --offset 1d --query "[?operationName.value=='Microsoft.Storage/storageAccounts/write']"
```

Vous pouvez utiliser Resource Graph pour afficher l’historique des modifications d’une ressource. Pour plus d’informations, consultez [Obtenir les modifications des ressources](../../governance/resource-graph/how-to/get-resource-changes.md).

## <a name="rest-api"></a>API REST

Les opérations REST à utiliser avec le journal d’activité font partie de l’ [API REST Insights](/rest/api/monitor/). Pour récupérer les événements du journal d’activité, consultez [Liste des événements de gestion dans un abonnement](/rest/api/monitor/activitylogs).

## <a name="next-steps"></a>Étapes suivantes

* Les journaux d’activité Azure sont utilisables avec Power BI pour obtenir des informations plus détaillées sur les actions de votre abonnement. Consultez le billet de blog sur [l’affichage et l’analyse des journaux d’activité Azure dans Power BI](https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/).
* Pour en savoir plus sur la définition de stratégies de sécurité, consultez [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/role-assignments-portal.md).
* Pour voir plus de détails sur les changements apportés à vos applications de la couche d’infrastructure jusqu’au déploiement des applications, consultez [Utiliser l’analyse des changements d’application dans Azure Monitor](../../azure-monitor/app/change-analysis.md).
* Pour en savoir plus sur les commandes permettant d’afficher les opérations de déploiement, consultez [Voir les opérations de déploiement](../templates/deployment-history.md).
* Pour savoir comment empêcher des suppressions sur une ressource pour tous les utilisateurs, consultez [Verrouiller des ressources avec Azure Resource Manager](lock-resources.md).
* Pour obtenir la liste des opérations disponibles pour chaque fournisseur Microsoft Azure Resource Manager, consultez [Opérations du fournisseur de ressources Azure](../../role-based-access-control/resource-provider-operations.md).
