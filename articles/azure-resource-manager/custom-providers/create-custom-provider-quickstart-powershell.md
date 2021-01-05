---
title: Créer un fournisseur de ressources personnalisé Azure avec Azure PowerShell
description: Décrit comment créer un fournisseur de ressources personnalisé Azure avec Azure PowerShell
author: MSEvanhi
ms.author: evanhi
ms.topic: quickstart
ms.devlang: azurepowershell
ms.date: 09/22/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6912f7f6fdc88c5d611bfbfd78f15e5f7a949f70
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91951840"
---
# <a name="quickstart-create-an-azure-custom-resource-provider-with-azure-powershell"></a>Démarrage rapide : Créer un fournisseur de ressources personnalisé Azure avec Azure PowerShell

Dans ce guide de démarrage rapide, vous apprendrez à créer votre propre fournisseur de ressources personnalisé Azure à l’aide du module PowerShell [AZ.CustomProviders](/powershell/module/az.customproviders).

> [!CAUTION]
> Les fournisseurs personnalisés Azure sont actuellement en préversion publique. Cette préversion est fournie sans contrat de niveau de service. Il n’est pas recommandé pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="requirements"></a>Spécifications

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

Si vous choisissez d’utiliser PowerShell localement, cet article vous demande d’installer le module Az PowerShell et de vous connecter à votre compte Azure avec l’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Pour en savoir plus sur l’installation du module Az PowerShell, consultez [Installer Azure PowerShell](/powershell/azure/install-az-ps). Si vous choisissez d’utiliser Cloud Shell, consultez [Vue d’ensemble d’Azure Cloud Shell](../../cloud-shell/overview.md) pour plus d’informations.

> [!IMPORTANT]
> Tant que le module PowerShell **Az.CustomProviders** est en préversion, vous devez l’installer séparément à l’aide de l’applet de commande `Install-Module`. Quand ce module PowerShell sera en disponibilité générale, il fera partie intégrante des versions futures du module PowerShell Az et sera disponible en mode natif dans Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.CustomProviders
```

Si vous avez plusieurs abonnements Azure, sélectionnez l’abonnement approprié dans lequel les ressources doivent être facturées. Sélectionnez un abonnement spécifique avec l’applet de commande [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un [groupe de ressources Azure](../../azure-resource-manager/management/overview.md) avec l’applet de commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées en tant que groupe.

L’exemple suivant crée un groupe de ressources avec un nom spécifié et à l’emplacement indiqué.

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location westus2
```

## <a name="create-a-custom-resource-provider"></a>Créer un fournisseur de ressources personnalisé

Pour créer ou mettre à jour un fournisseur de ressources personnalisé, vous devez utiliser l’applet de commande [New-AzCustomProvider](/powershell/module/az.customproviders/new-azcustomprovider) comme indiqué dans l’exemple suivant.

```azurepowershell-interactive
New-AzCustomProvider -ResourceGroupName myResourceGroup -Name Namespace.Type -Location westus2 -ResourceType @{Name='CustomRoute1'; Endpoint='https://www.contoso.com/'}
```

## <a name="get-the-custom-resource-provider-manifest"></a>Obtenir le manifeste du fournisseur de ressources personnalisé

Pour récupérer des informations sur le manifeste du fournisseur de ressources personnalisé, utilisez l’applet de commande [Get-AzCustomProvider](/powershell/module/az.customproviders/get-azcustomprovider) comme indiqué dans l’exemple suivant.

```azurepowershell-interactive
Get-AzCustomProvider -ResourceGroupName myResourceGroup -Name Namespace.Type | Format-List
```

## <a name="create-an-association"></a>Créer une association

Pour créer ou mettre à jour une association, vous devez utiliser l’applet de commande [New-AzCustomProviderAssociation](/powershell/module/az.customproviders/new-azcustomproviderassociation) comme indiqué dans l’exemple suivant.

```azurepowershell-interactive
$provider = Get-AzCustomProvider -ResourceGroupName myResourceGroup -Name Namespace.Type
New-AzCustomProviderAssociation -Scope $resourceId -Name MyAssoc -TargetResourceId $provider.Id
```

## <a name="get-an-association"></a>Obtenir une association

Pour récupérer des informations sur une association, vous devez utiliser l’applet de commande [Get-AzCustomProviderAssociation](/powershell/module/az.customproviders/get-azcustomproviderassociation) comme indiqué dans l’exemple suivant.

```azurepowershell-interactive
Get-AzCustomProviderAssociation -Scope $resourceId -Name MyAssoc
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez pas besoin des ressources que vous avez créées dans cet article, vous pouvez les supprimer en exécutant l’exemple suivant.

### <a name="delete-an-association"></a>Supprimer une association

Pour supprimer une association, utilisez l’applet de commande [Remove-AzCustomProviderAssociation](/powershell/module/az.customproviders/remove-azcustomproviderassociation) . L’exemple suivant supprime une association.

```azurepowershell
Remove-AzCustomProviderAssociation -Scope $id -Name Namespace.Type
```

### <a name="delete-a-custom-resource-provider"></a>Supprimer un fournisseur de ressources personnalisé

Pour supprimer un fournisseur de ressources personnalisé, utilisez l’applet de commande [Remove-AzCustomProvider](/powershell/module/az.customproviders/remove-azcustomprovider). L’exemple suivant supprime un fournisseur de ressources personnalisé.

```azurepowershell-interactive
Remove-AzCustomProvider -ResourceGroupName myResourceGroup -Name Namespace.Type
```

### <a name="delete-the-resource-group"></a>Supprimer le groupe de ressources

> [!CAUTION]
> L’exemple suivant supprime le groupe de ressources spécifié et toutes les ressources qu’il contient.
> Si des ressources en dehors du cadre de cet article existent dans le groupe de ressources spécifié, elles seront également supprimées.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur les [fournisseurs de ressources personnalisés Azure](overview.md).