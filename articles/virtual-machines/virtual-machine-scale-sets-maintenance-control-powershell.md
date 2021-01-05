---
title: Contrôle de maintenance pour les mises à niveau d’images de système d’exploitation sur les groupes de machines virtuelles identiques Azure à l’aide de PowerShell
description: Découvrez comment contrôler le moment où les mises à niveau automatiques des images de système d’exploitation sont déployées sur vos groupes de machines virtuelles identiques Azure à l’aide du contrôle de maintenance et de PowerShell.
author: ju-shim
ms.service: virtual-machine-scale-sets
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 09/11/2020
ms.author: jushiman
ms.openlocfilehash: d8acab17e9d8dfc078b46f6a279cc671a70b0a50
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91974836"
---
# <a name="preview-maintenance-control-for-os-image-upgrades-on-azure-virtual-machine-scale-sets-using-powershell"></a>Aperçu : Contrôle de maintenance pour les mises à niveau d’images de système d’exploitation sur les groupes de machines virtuelles identiques Azure à l’aide de PowerShell

Le contrôle de maintenance vous permet de choisir quand appliquer automatiquement les mises à niveau d’images de système d’exploitation invité à vos groupes de machines virtuelles identiques. Cette rubrique traite des options de contrôle de maintenance d’Azure PowerShell. Pour plus d’informations sur l’utilisation du contrôle de maintenance, consultez [Contrôle maintenance pour les groupes de machines virtuelles identiques Azure](virtual-machine-scale-sets-maintenance-control.md).

> [!IMPORTANT]
> Le contrôle de maintenance des mises à niveau d’images de système d’exploitation sur les groupes de machines virtuelles identiques Azure est actuellement disponible en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


## <a name="enable-the-powershell-module"></a>Activer le module PowerShell

Vérifiez que `PowerShellGet` est à jour.    

```azurepowershell-interactive  
Install-Module -Name PowerShellGet -Repository PSGallery -Force 
``` 

Installez le module PowerShell `Az.Maintenance`.     

```azurepowershell-interactive  
Install-Module -Name Az.Maintenance
``` 

Si vous installez localement, veillez à ouvrir votre invite PowerShell en tant qu’administrateur.

Vous pouvez également être invité à confirmer que vous souhaitez installer à partir d’un *référentiel non approuvé*. Tapez `Y` ou sélectionnez **Oui pour tout** afin d’installer le module.

## <a name="connect-to-an-azure-account"></a>Se connecter à un compte Azure

Connectez-vous au compte Azure souhaité à l’aide de [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) et [Set-AzAccount](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Connect-AzAccount
Set-AzContext 00a000aa-0a00-0a0a-00aa-a00a000aaa00

$RGName="myMaintenanceRG"
$MaintenanceConfig="myMaintenanceConfig"
$location="eastus2"
$vmss="myMaintenanceVMSS"
```

## <a name="create-a-maintenance-configuration"></a>Créer une configuration de maintenance

Créez un groupe de ressources en tant que conteneur pour votre configuration. Dans cet exemple, un groupe de ressources nommé *myMaintenanceRG* est créé dans *eastus2*. Vous pouvez ignorer cette partie si vous disposez déjà d’un groupe de ressources. Remplacez simplement le nom du groupe de ressources par votre nom dans le reste des exemples.

```azurepowershell-interactive
New-AzResourceGroup `
   -Location $location `
   -Name $RGName
```

Utilisez [New-AzMaintenanceConfiguration](/powershell/module/az.maintenance/new-azmaintenanceconfiguration) pour créer une configuration de maintenance. Cet exemple crée une configuration de maintenance nommée *myConfig*, étendue à l’image de système d'exploitation. 

```azurepowershell-interactive
$config = New-AzMaintenanceConfiguration `
   -ResourceGroup $RGName `
   -Name $MaintenanceConfig `
   -MaintenanceScope OSImage `
   -Location $location `
   -StartDateTime "2020-10-01 00:00" `
   -TimeZone "Pacific Standard Time" `
   -Duration "05:00" `
   -RecurEvery "Day"
```

> [!IMPORTANT]
> La **durée** de maintenance doit être de *5 heures* ou plus. La **périodicité** de la maintenance doit être définie sur *Jour*.

L’utilisation de `-MaintenanceScope OSImage` garantit l’utilisation de la configuration de maintenance pour contrôler les mises à jour du système d’exploitation invité.

Si vous essayez de créer une configuration portant le même nom mais dans un autre emplacement, vous obtenez une erreur. Les noms de configuration doivent être uniques dans votre groupe de ressources.

Vous pouvez rechercher les configurations de maintenance disponibles à l’aide de [Get-AzMaintenanceConfiguration](/powershell/module/az.maintenance/get-azmaintenanceconfiguration).

```azurepowershell-interactive
Get-AzMaintenanceConfiguration | Format-Table -Property Name,Id
```

## <a name="associate-your-virtual-machine-scale-set-to-the-maintenance-configuration"></a>Associer votre groupe de machines virtuelles identiques à une configuration de maintenance

Un groupe de machines virtuelles identiques peut être associé à n’importe quelle configuration de maintenance et ce, indépendamment de la région et de l’abonnement de la configuration de maintenance. En s’inscrivant à la configuration de maintenance, les nouvelles mises à jour d’image de système d’exploitation correspondant au groupe identique sont automatiquement planifiées lors de la fenêtre de maintenance disponible suivante.

Utilisez [New-AzConfigurationAssignment](/powershell/module/az.maintenance/new-azconfigurationassignment) pour associer votre groupe de machines virtuelles identiques à une configuration de maintenance.

```azurepowershell-interactive
New-AzConfigurationAssignment `
   -ResourceGroupName $RGName `
   -Location $location `
   -ResourceName $vmss `
   -ResourceType VirtualMachineScaleSets `
   -ProviderName Microsoft.Compute `
   -ConfigurationAssignmentName $config.Name`
   -MaintenanceConfigurationId $config.Id
``` 

## <a name="enable-automatic-os-upgrade"></a>Activez les mises à niveau automatiques du système d’exploitation

Vous pouvez activer les mises à niveau automatiques du système d’exploitation pour chaque groupe de machines virtuelles identiques qui utilisera le contrôle de maintenance. Pour obtenir des mises à niveau automatiques du système d’exploitation sur votre groupe de machines virtuelles identiques, consultez le document [Mises à niveau automatiques d’images de système d’exploitation de groupes de machines virtuelles identiques Azure](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md). 


## <a name="next-steps"></a>Étapes suivantes

Découvrir la maintenance et les mises à jour pour les machines virtuelles s’exécutant dans Azure.

> [!div class="nextstepaction"]
> [Maintenance et mises à jour](maintenance-and-updates.md)