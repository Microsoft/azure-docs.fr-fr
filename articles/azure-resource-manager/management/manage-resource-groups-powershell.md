---
title: Gérer des groupes de ressources - Azure PowerShell
description: Utilisez Azure PowerShell pour gérer vos groupes de ressources dans Azure Resource Manager. Montre comment créer, lister et supprimer des groupes de ressources.
author: mumian
ms.topic: conceptual
ms.date: 09/01/2020
ms.author: jgao
ms.openlocfilehash: e5449dc26197a0aa2e2b0da49ad9e5ca4fb19e82
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185757"
---
# <a name="manage-azure-resource-manager-resource-groups-by-using-azure-powershell"></a>Gérer des groupes de ressources Azure Resource Manager à l'aide d'Azure PowerShell

Apprenez à utiliser Azure PowerShell avec [Azure Resource Manager](overview.md) pour gérer vos groupes de ressources Azure. Pour gérer des ressources Azure, consultez [Gérer les ressources Azure à l'aide d'Azure PowerShell](manage-resources-powershell.md).

Autres articles sur la gestion des groupes de ressources :

- [Gérer les groupes de ressources Azure à l’aide du portail Azure](manage-resources-portal.md)
- [Gérer les groupes de ressources Azure à l'aide d'Azure CLI](manage-resources-cli.md)

## <a name="what-is-a-resource-group"></a>Qu’est-ce qu’un groupe de ressources

Un groupe de ressources est un conteneur réunissant les ressources associées d’une solution Azure. Le groupe de ressources peut inclure toutes les ressources de la solution, ou uniquement celles que vous souhaitez gérer en tant que groupe. Pour déterminer comment allouer des ressources aux groupes de ressources, choisissez l’approche la plus pertinente pour votre organisation. En règle générale, il convient d’ajouter des ressources qui partagent le même cycle de vie dans un même groupe de ressources afin de pouvoir facilement les déployer, les mettre à jour et les supprimer en tant que groupe.

Le groupe de ressources stocke des métadonnées sur les ressources. Quand vous spécifiez un emplacement pour le groupe de ressources, vous indiquez donc où stocker ces métadonnées. Pour des raisons de conformité, vous devrez peut-être vous assurer que vos données sont stockées dans une région particulière.

Le groupe de ressources stocke des métadonnées sur les ressources. Lorsque vous spécifiez un emplacement pour le groupe de ressources, vous indiquez où stocker ces métadonnées.

## <a name="create-resource-groups"></a>Créer des groupes de ressources

Le script PowerShell suivant crée un groupe de ressources.

```azurepowershell-interactive
New-AzResourceGroup -Name demoResourceGroup -Location westus
```

## <a name="list-resource-groups"></a>Répertorier les groupes de ressources

Le script PowerShell suivant répertorie les groupes de ressources de votre abonnement.

```azurepowershell-interactive
Get-AzResourceGroup
```

Pour obtenir un groupe de ressources :

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"

Get-AzResourceGroup -Name $resourceGroupName
```

## <a name="delete-resource-groups"></a>Supprimer des groupes de ressources

Le script PowerShell suivant supprime un groupe de ressources :

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"

Remove-AzResourceGroup -Name $resourceGroupName
```

Pour plus d’informations sur l’ordre dans lequel Azure Resource Manager supprime des ressources, voir [Suppression d’un groupe de ressources par Azure Resource Manager](delete-resource-group.md).

## <a name="deploy-resources-to-an-existing-resource-group"></a>Déployer des ressources sur un groupe de ressources existant

Voir [Déployer des ressources sur un groupe de ressources existant](manage-resources-powershell.md#deploy-resources-to-an-existing-resource-group).

Pour valider le déploiement d'un groupe de ressources, consultez [Test-AzResourceGroupDeployment](/powershell/module/Az.Resources/Test-AzResourceGroupDeployment?view=azps-1.3.0).

## <a name="deploy-a-resource-group-and-resources"></a>Déployer un groupe de ressources et des ressources

Vous pouvez créer un groupe de ressources et déployer des ressources sur ce groupe à l'aide d'un modèle Resource Manager. Pour plus d'informations, consultez [Créer un groupe de ressources et déployer des ressources](../templates/deploy-to-subscription.md#resource-groups).

## <a name="redeploy-when-deployment-fails"></a>Redéploiement en cas d’échec du déploiement

Cette fonctionnalité est également appelée *Annulation en cas d’erreur*. Pour plus d'informations, consultez [Redéploiement en cas d'échec du déploiement](../templates/rollback-on-error.md).

## <a name="move-to-another-resource-group-or-subscription"></a>Déplacer vers un autre groupe de ressources ou abonnement

Vous pouvez déplacer les ressources du groupe vers un autre groupe de ressources. Pour plus d’informations, consultez la page [Déplacement de ressources vers un nouveau groupe de ressources ou un abonnement](move-resource-group-and-subscription.md).

## <a name="lock-resource-groups"></a>Verrouiller des groupes de ressources

Le verrouillage empêche d’autres utilisateurs de votre organisation de supprimer ou de modifier accidentellement des ressources stratégiques, telles qu’une ressource, un groupe de ressources ou un abonnement Azure. 

Le script suivant verrouille un groupe de ressources pour empêcher sa suppression.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"

New-AzResourceLock -LockName LockGroup -LockLevel CanNotDelete -ResourceGroupName $resourceGroupName 
```

Le script suivant obtient tous les verrous relatifs à un groupe de ressources :

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"

Get-AzResourceLock -ResourceGroupName $resourceGroupName 
```

Pour plus d’informations, consultez [Verrouiller des ressources avec Azure Resource Manager](lock-resources.md).

## <a name="tag-resource-groups"></a>Baliser des groupes de ressources

Vous pouvez appliquer des balises à des groupes de ressources pour organiser logiquement vos ressources. Pour plus d’informations, voir [Organisation des ressources Azure à l’aide de balises](tag-resources.md#powershell).

## <a name="export-resource-groups-to-templates"></a>Exporter des groupes de ressources dans des modèles

Une fois que vous avez configuré votre groupe de ressources, vous pouvez afficher un modèle Resource Manager pour celui-ci. L’exportation du modèle offre deux avantages :

- Automatisez les futurs déploiements de la solution, car le modèle contient la totalité de l’infrastructure.
- Familiarisez-vous avec la syntaxe des modèles en consultant la notation JSON (JavaScript Object Notation) qui représente votre solution.

Pour exporter toutes les ressources d’un groupe de ressources, utilisez l’applet de commande [AzResourceGroup d’exportation](/powershell/module/az.resources/Export-AzResourceGroup) et indiquez le nom du groupe de ressources.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"

Export-AzResourceGroup -ResourceGroupName $resourceGroupName
```

Il enregistre le modèle dans un fichier local.

Plutôt que d’exporter toutes les ressources dans le groupe de ressources, vous pouvez sélectionner les ressources à exporter.

Pour exporter une ressource, transmettez cet ID de ressource.

```azurepowershell-interactive
$resource = Get-AzResource `
  -ResourceGroupName <resource-group-name> `
  -ResourceName <resource-name> `
  -ResourceType <resource-type>
Export-AzResourceGroup `
  -ResourceGroupName <resource-group-name> `
  -Resource $resource.ResourceId
```

Pour exporter plusieurs ressources, transmettez les ID de ressource dans un tableau.

```azurepowershell-interactive
Export-AzResourceGroup `
  -ResourceGroupName <resource-group-name> `
  -Resource @($resource1.ResourceId, $resource2.ResourceId)
```

Lorsque vous exportez le modèle, vous pouvez spécifier si des paramètres sont utilisés dans celui-ci. Par défaut, les paramètres des noms de ressource sont inclus, mais ils n’ont pas de valeur par défaut. Vous devez transmettre cette valeur de paramètre pendant le déploiement.

```json
"parameters": {
  "serverfarms_demoHostPlan_name": {
    "defaultValue": null,
    "type": "String"
  },
  "sites_webSite3bwt23ktvdo36_name": {
    "defaultValue": null,
    "type": "String"
  }
}
```

Dans la ressource, le paramètre est utilisé pour le nom.

```json
"resources": [
  {
    "type": "Microsoft.Web/serverfarms",
    "apiVersion": "2016-09-01",
    "name": "[parameters('serverfarms_demoHostPlan_name')]",
    ...
  }
]
```

Si vous utilisez le paramètre `-IncludeParameterDefaultValue` lors de l’exportation du modèle, le paramètre de modèle inclut une valeur par défaut qui est définie sur la valeur actuelle. Vous pouvez utiliser cette valeur par défaut ou remplacer la valeur par défaut en transmettant une valeur différente.

```json
"parameters": {
  "serverfarms_demoHostPlan_name": {
    "defaultValue": "demoHostPlan",
    "type": "String"
  },
  "sites_webSite3bwt23ktvdo36_name": {
    "defaultValue": "webSite3bwt23ktvdo36",
    "type": "String"
  }
}
```

Si vous utilisez le paramètre `-SkipResourceNameParameterization` lors de l’exportation du modèle, les paramètres des noms de ressources ne sont pas inclus dans le modèle. Au lieu de cela, le nom de ressource est défini directement sur la ressource sur sa valeur actuelle. Vous ne pouvez pas personnaliser le nom au cours du déploiement.

```json
"resources": [
  {
    "type": "Microsoft.Web/serverfarms",
    "apiVersion": "2016-09-01",
    "name": "demoHostPlan",
    ...
  }
]
```

La fonctionnalité d’exportation de modèle ne prend pas en charge l’exportation des ressources Azure Data Factory. Pour en savoir plus sur la façon dont vous pouvez exporter des ressources Data Factory, consultez [Copier ou cloner une fabrique de données dans Azure Data Factory](../../data-factory/copy-clone-data-factory.md).

Pour exporter des ressources créées par le biais du modèle de déploiement classique, vous devez [les migrer vers le modèle de déploiement de Resource Manager](../../virtual-machines/migration-classic-resource-manager-overview.md).

Pour plus d’informations, consultez [Export d’une ressource unique ou de plusieurs ressources vers un modèle dans le portail Azure](../templates/export-template-portal.md).

## <a name="manage-access-to-resource-groups"></a>Gérer l'accès aux groupes de ressources

Le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md) vous permet de gérer l’accès aux ressources dans Azure. Pour plus d’informations, consultez [Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md).

## <a name="next-steps"></a>Étapes suivantes

- Pour vous familiariser avec Azure Resource Manager, consultez [Vue d’ensemble d’Azure Resource Manager](overview.md).
- Pour vous familiariser avec la syntaxe des modèles Resource Manager, consultez [Comprendre la structure et la syntaxe des modèles Azure Resource Manager](../templates/template-syntax.md).
- Pour apprendre à développer des modèles, consultez les [tutoriels pas à pas](../index.yml).
- Pour accéder aux schémas liés aux modèles Azure Resource Manager, consultez [Informations de référence sur les modèles](/azure/templates/).