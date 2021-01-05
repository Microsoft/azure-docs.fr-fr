---
title: Créer un tableau de bord dans le portail Azure avec PowerShell
description: Découvrez comment créer un tableau de bord dans le portail Azure à l’aide d’Azure PowerShell.
ms.topic: quickstart
ms.custom: devx-track-azurepowershell
ms.date: 07/24/2020
ms.openlocfilehash: 02e243a7296555d73427f8e31c4abdf9c3e56735
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745738"
---
# <a name="quickstart-create-an-azure-portal-dashboard-with-powershell"></a>Démarrage rapide : Créer un tableau de bord dans le portail Azure avec PowerShell

Dans le portail Azure, un tableau de bord est une vue ciblée et organisée de vos ressources cloud. Cet article est consacré au processus d’utilisation du module PowerShell Az.Portal pour créer un tableau de bord.
Le tableau de bord affiche les performances d’une machine virtuelle, ainsi que des liens et des informations statiques.

## <a name="requirements"></a>Spécifications

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

Si vous choisissez d’utiliser PowerShell localement, cet article vous demande d’installer le module Az PowerShell et de vous connecter à votre compte Azure avec l’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Pour en savoir plus sur l’installation du module Az PowerShell, consultez [Installer Azure PowerShell](/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Tant que le module PowerShell **Az.Portal** est en préversion, vous devez l’installer séparément du module PowerShell Az à l’aide de la commande suivante : `Install-Module`. Quand ce module PowerShell sera en disponibilité générale, il fera partie intégrante des versions futures du module PowerShell Az et sera disponible en mode natif dans Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.Portal
```

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

## <a name="choose-a-specific-azure-subscription"></a>Choisir un abonnement Azure spécifique

Si vous avez plusieurs abonnements Azure, sélectionnez l’abonnement approprié dans lequel les ressources doivent être facturées. Sélectionnez un abonnement spécifique avec l’applet de commande [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="define-variables"></a>Définir des variables

Vous utiliserez plusieurs types d’informations de façon répétée. Créez des variables pour stocker les informations.

```azurepowershell-interactive
# Name of resource group used throughout this article
$resourceGroupName = 'myResourceGroup'

# Azure region
$location = 'centralus'

# Dashboard Title
$dashboardTitle = 'Simple VM Dashboard'

# Dashboard Name
$dashboardName = $dashboardTitle -replace '\s'

# Your Azure Subscription ID
$subscriptionID = (Get-AzContext).Subscription.Id

# Name of test VM
$vmName = 'SimpleWinVM'
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un [groupe de ressources Azure](../azure-resource-manager/management/overview.md) avec l’applet de commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées en tant que groupe.

L’exemple suivant crée un groupe de ressources en fonction du nom de la variable `$resourceGroupName` dans la région spécifiée de la variable `$location`.

```azurepowershell-interactive
New-AzResourceGroup -Name $resourceGroupName -Location $location
```

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Le tableau de bord que vous créez dans la prochaine partie de ce guide de démarrage rapide nécessite une machine virtuelle existante. Créez une machine virtuelle en effectuant les étapes suivantes.

Stockez les informations d’identification de connexion pour la machine virtuelle dans une variable. Le mot de passe doit être complexe. Il s’agit d’un nouveau nom d’utilisateur et d’un nouveau mot de passe. Ce n’est pas, par exemple, le compte que vous utilisez pour vous connecter à Azure. Pour plus d’informations, consultez les [exigences relatives aux noms d’utilisateur](../virtual-machines/windows/faq.md#what-are-the-username-requirements-when-creating-a-vm) et les [exigences relatives aux mots de passe](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).

```azurepowershell-interactive
$Cred = Get-Credential
```

Créez la machine virtuelle.

```azurepowershell-interactive
$AzVmParams = @{
  ResourceGroupName = $resourceGroupName
  Name = $vmName
  Location = $location
  Credential = $Cred
}
New-AzVm @AzVmParams
```

Le déploiement de la machine virtuelle commence alors et prend quelques minutes. Une fois le déploiement terminé, passez à la section suivante.

## <a name="download-the-dashboard-template"></a>Télécharger le modèle de tableau de bord

Étant donné que les tableaux de bord Azure sont des ressources, ils peuvent être représentés au format JSON. Le code suivant télécharge la représentation JSON d’un exemple de tableau de bord. Pour plus d’informations, consultez [Structure des tableaux de bord Azure](./azure-portal-dashboards-structure.md).

```azurepowershell-interactive
$myPortalDashboardTemplateUrl = 'https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/azure-portal/portal-dashboard-template-testvm.json'

$myPortalDashboardTemplatePath = "$env:TEMP\portal-dashboard-template-testvm.json"

Invoke-WebRequest -Uri $myPortalDashboardTemplateUrl -OutFile $myPortalDashboardTemplatePath -UseBasicParsing
```

## <a name="customize-the-template"></a>Personnaliser le modèle

Personnalisez le modèle téléchargé en exécutant le code suivant.

```azurepowershell
$Content = Get-Content -Path $myPortalDashboardTemplatePath -Raw
$Content = $Content -replace '<subscriptionID>', $subscriptionID
$Content = $Content -replace '<rgName>', $resourceGroupName
$Content = $Content -replace '<vmName>', $vmName
$Content = $Content -replace '<dashboardTitle>', $dashboardTitle
$Content = $Content -replace '<location>', $location
$Content | Out-File -FilePath $myPortalDashboardTemplatePath -Force
```

Pour plus d’informations, consultez [Référence sur le modèle de tableaux de bord Microsoft.Portal](/azure/templates/microsoft.portal/dashboards).

## <a name="deploy-the-dashboard-template"></a>Déployer le modèle de tableau de bord

Vous pouvez utiliser la cmdlet `New-AzPortalDashboard` qui fait partie du module Az.Portal pour déployer le modèle directement à partir de PowerShell.

```azurepowershell
$DashboardParams = @{
  DashboardPath = $myPortalDashboardTemplatePath
  ResourceGroupName = $resourceGroupName
  DashboardName = $dashboardName
}
New-AzPortalDashboard @DashboardParams
```

## <a name="review-the-deployed-resources"></a>Vérifier les ressources déployées

Vérifiez que le tableau de bord a été correctement créé.

```azurepowershell
Get-AzPortalDashboard -Name $dashboardName -ResourceGroupName $resourceGroupName
```

Vérifiez que vous pouvez voir les données relatives à la machine virtuelle à partir du portail Azure.

1. Dans le portail Azure, sélectionnez **Tableau de bord**.

   ![Navigation du portail Azure jusqu’au tableau de bord](media/quickstart-portal-dashboard-powershell/navigate-to-dashboards.png)

1. Dans la page Tableau de bord, sélectionnez **Tableau de bord Machine virtuelle simple**.

   ![Accéder au tableau de bord Machine virtuelle simple](media/quickstart-portal-dashboard-powershell/select-simple-vm-dashboard.png)

1. Vérifiez le tableau de bord. Vous pouvez voir qu’une partie du contenu est statique, mais il y a également des graphiques qui indiquent les performances de la machine virtuelle.

   ![Examiner le tableau de bord Machine virtuelle simple](media/quickstart-portal-dashboard-powershell/review-simple-vm-dashboard.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer la machine virtuelle et le tableau de bord associé, supprimez le groupe de ressources qui les contient.

> [!CAUTION]
> L’exemple suivant supprime le groupe de ressources spécifié et toutes les ressources qu’il contient.
> Si des ressources en dehors du cadre de cet article existent dans le groupe de ressources spécifié, elles seront également supprimées.

```azurepowershell-interactive
Remove-AzResourceGroup -Name $resourceGroupName
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les applets de commande du module PowerShell Az.Portal, consultez :

> [!div class="nextstepaction"]
> [Microsoft Azure PowerShell : applets de commande de tableau de bord du portail](/powershell/module/Az.Portal/)