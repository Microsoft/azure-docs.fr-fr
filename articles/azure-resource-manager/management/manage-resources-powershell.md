---
title: Gérer des ressources – Azure PowerShell
description: Utilisez Azure PowerShell et Azure Resource Manager pour gérer vos ressources. Montre comment déployer et supprimer des ressources.
author: mumian
ms.topic: conceptual
ms.date: 02/11/2019
ms.author: jgao
ms.openlocfilehash: ce3b1d9f8d01e722e4ec5078d92ef4148072da25
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185740"
---
# <a name="manage-azure-resources-by-using-azure-powershell"></a>Gérer les ressources Azure à l’aide d’Azure PowerShell

Découvrez comment utiliser Azure PowerShell avec [Azure Resource Manager](overview.md) pour gérer vos ressources Azure. Pour gérer des groupes de ressources, consultez [Gérer des groupes de ressources Azure à l’aide d’Azure PowerShell](manage-resource-groups-powershell.md).

Autres articles sur la gestion des ressources :

- [Gérer des ressources Azure à l’aide du portail Azure](manage-resources-portal.md)
- [Gérer des ressources Azure à l’aide d’Azure CLI](manage-resources-cli.md)

## <a name="deploy-resources-to-an-existing-resource-group"></a>Déployer des ressources sur un groupe de ressources existant

Vous pouvez déployer des ressources Azure directement à l’aide d’Azure PowerShell ou déployer un modèle Resource Manager pour créer des ressources Azure.

### <a name="deploy-a-resource"></a>Déployer une ressource

Le script suivant permet de créer un compte de stockage.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

# Create the storage account.
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroupName `
  -Name $storageAccountName `
  -Location $location `
  -SkuName "Standard_LRS"

# Retrieve the context.
$ctx = $storageAccount.Context
```

### <a name="deploy-a-template"></a>Déploiement d’un modèle

Le script suivant déploie un modèle de démarrage rapide pour créer un compte de stockage. Pour plus d’informations, consultez [Démarrage rapide : Créer des modèles Azure Resource Manager à l’aide de Visual Studio Code](../templates/quickstart-create-templates-use-visual-studio-code.md?tabs=PowerShell).

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json"
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -Location $location
```

Pour plus d’informations, consultez [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](../templates/deploy-powershell.md).

## <a name="deploy-a-resource-group-and-resources"></a>Déployer un groupe de ressources et des ressources

Vous pouvez créer un groupe de ressources et y déployer des ressources. Pour plus d'informations, consultez [Créer un groupe de ressources et déployer des ressources](../templates/deploy-to-subscription.md#resource-groups).

## <a name="deploy-resources-to-multiple-subscriptions-or-resource-groups"></a>Déployer des ressources sur plusieurs abonnements ou groupes de ressources

En général, vous déployez toutes les ressources dans votre modèle sur un seul groupe de ressources. Toutefois, il existe des scénarios dans lesquels vous pouvez souhaitez déployer un ensemble de ressources, tout en les plaçant dans différents groupes de ressources ou abonnements. Pour plus d’informations, consultez [Déployer des ressources Azure sur plusieurs abonnements ou groupes de ressources](../templates/deploy-to-resource-group.md).

## <a name="delete-resources"></a>Supprimer des ressources

Le script suivant montre comment supprimer un compte de stockage.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

Remove-AzStorageAccount -ResourceGroupName $resourceGroupName -AccountName $storageAccountName
```

Pour plus d’informations sur l’ordre dans lequel Azure Resource Manager supprime des ressources, voir [Suppression d’un groupe de ressources par Azure Resource Manager](delete-resource-group.md).

## <a name="move-resources"></a>Déplacer des ressources

Le script suivant montre comment déplacer un compte de stockage depuis un groupe de ressources vers un autre.

```azurepowershell-interactive
$srcResourceGroupName = Read-Host -Prompt "Enter the source Resource Group name"
$destResourceGroupName = Read-Host -Prompt "Enter the destination Resource Group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

$storageAccount = Get-AzResource -ResourceGroupName $srcResourceGroupName -ResourceName $storageAccountName
Move-AzResource -DestinationResourceGroupName $destResourceGroupName -ResourceId $storageAccount.ResourceId
```

Pour plus d’informations, consultez la page [Déplacement de ressources vers un nouveau groupe de ressources ou un abonnement](move-resource-group-and-subscription.md).

## <a name="lock-resources"></a>Verrouiller des ressources

Le verrouillage empêche d’autres utilisateurs de votre organisation de supprimer ou de modifier accidentellement des ressources stratégiques, telles qu’une ressource, un groupe de ressources ou un abonnement Azure. 

Le script suivant verrouille un compte de stockage afin d’empêcher sa suppression.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

New-AzResourceLock -LockName LockStorage -LockLevel CanNotDelete -ResourceGroupName $resourceGroupName -ResourceName $storageAccountName -ResourceType Microsoft.Storage/storageAccounts 
```

Le script suivant obtient tous les verrous pour un compte de stockage :

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

Get-AzResourceLock -ResourceGroupName $resourceGroupName -ResourceName $storageAccountName -ResourceType Microsoft.Storage/storageAccounts
```

Le script suivant supprime un verrou d’un compte de stockage :

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

$lockId = (Get-AzResourceLock -ResourceGroupName $resourceGroupName -ResourceName $storageAccountName -ResourceType Microsoft.Storage/storageAccounts).LockId
Remove-AzResourceLock -LockId $lockId
```

Pour plus d’informations, consultez [Verrouiller des ressources avec Azure Resource Manager](lock-resources.md).

## <a name="tag-resources"></a>Baliser des ressources

Le balisage facilite l’organisation logique des ressources et du groupe de ressources. Pour plus d’informations, voir [Organisation des ressources Azure à l’aide de balises](tag-resources.md#powershell).

## <a name="manage-access-to-resources"></a>Gérer l’accès aux ressources

Le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md) vous permet de gérer l’accès aux ressources dans Azure. Pour plus d’informations, consultez [Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md).

## <a name="next-steps"></a>Étapes suivantes

- Pour vous familiariser avec Azure Resource Manager, consultez [Vue d’ensemble d’Azure Resource Manager](overview.md).
- Pour vous familiariser avec la syntaxe des modèles Resource Manager, consultez [Comprendre la structure et la syntaxe des modèles Azure Resource Manager](../templates/template-syntax.md).
- Pour apprendre à développer des modèles, consultez les [tutoriels pas à pas](../index.yml).
- Pour accéder aux schémas liés aux modèles Azure Resource Manager, consultez [Informations de référence sur les modèles](/azure/templates/).