---
title: 'Tutoriel : Créer un groupe de machines virtuelles identiques Windows'
description: Découvrez comment utiliser Azure PowerShell pour créer et déployer une application hautement disponible sur les machines virtuelles Windows à l’aide d’un groupe de machines virtuelles identiques
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: windows
ms.date: 11/30/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: b3853ddc71d1a9be26b2492764a9b341446e0eeb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89078739"
---
# <a name="tutorial-create-a-virtual-machine-scale-set-and-deploy-a-highly-available-app-on-windows-with-azure-powershell"></a>Tutoriel : Créer un groupe de machines virtuelles identiques et déployer une application hautement disponible sur Windows avec Azure PowerShell
Un groupe de machines virtuelles identiques vous permet de déployer et de gérer un ensemble de machines virtuelles identiques prenant en charge la mise à l’échelle automatique. Vous pouvez mettre à l’échelle manuellement le nombre de machines virtuelles du groupe identique. Vous pouvez également définir des règles pour mettre à l’échelle automatiquement en fonction de l’utilisation des ressources (processeur, demande de mémoire, trafic réseau, etc.). Ce tutoriel explique comment déployer un groupe de machines virtuelles identiques dans Azure et vous apprend à :

> [!div class="checklist"]
> * Utiliser l’extension de script personnalisé pour définir un site IIS à mettre à l’échelle
> * Créer un équilibrage de charge pour votre groupe identique
> * Créer un groupe de machines virtuelles identiques
> * Augmenter ou réduire le nombre d’instances dans un groupe identique
> * Créer des règles de mise à l’échelle automatique

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. 

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.

## <a name="scale-set-overview"></a>Vue d’ensemble des groupes identiques
Un groupe de machines virtuelles identiques vous permet de déployer et de gérer un ensemble de machines virtuelles identiques prenant en charge la mise à l’échelle automatique. Les machines virtuelles d’un groupe identique sont réparties entre les domaines d’erreur logique et de mise à jour, dans un ou plusieurs *groupes de placement*. Les groupes de placement contiennent des machines virtuelles configurées de manière similaire, semblables à des [groupes à haute disponibilité](tutorial-availability-sets.md).

Les machines virtuelles sont créées en fonction des besoins dans un groupe identique. En définissant des règles de mise à l’échelle automatique, vous pouvez contrôler quand et comment les machines virtuelles sont ajoutées ou supprimées au niveau du groupe identique. Ces règles peuvent se déclencher en fonction de mesures telles que la charge du processeur, l’utilisation de la mémoire ou le trafic réseau.

Les groupes identiques prennent en charge jusqu’à 1 000 machines virtuelles lorsque vous utilisez une image de plateforme Azure. Pour les charges de travail qui s’accompagnent de contraintes importantes en matière d’installation ou de personnalisation de machines virtuelles, vous pouvez [créer une image de machine virtuelle personnalisée](tutorial-custom-images.md). Vous pouvez créer un maximum de 600 machines virtuelles dans un groupe identique lorsque vous utilisez une image personnalisée.


## <a name="create-a-scale-set"></a>Créer un groupe identique
Créez un groupe de machines virtuelles identiques avec [New-AzVmss](/powershell/module/az.compute/new-azvmss). L’exemple suivant crée un groupe identique nommé *myScaleSet* qui utilise l’image de plateforme *Windows Server 2016 Datacenter*. Les ressources réseau Azure pour le réseau virtuel, l’adresse IP publique et l’équilibreur de charge sont automatiquement créées. Quand vous y êtes invité, vous pouvez définir vos propres informations d’identification d’administration pour les instances de machine virtuelle du groupe identique :

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -Location "EastUS" `
  -VMScaleSetName "myScaleSet" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicyMode "Automatic"
```

La création et la configuration des l’ensemble des ressources et des machines virtuelles du groupe identique prennent quelques minutes.


## <a name="deploy-sample-application"></a>Déployer un exemple d’application
Pour tester votre groupe identique, installez une application web de base. L’extension de script personnalisé Azure permet de télécharger et d’exécuter un script qui installe IIS sur les instances de machine virtuelle. Cette extension est utile pour la configuration post-déploiement, l’installation de logiciels ou toute autre tâche de configuration ou de gestion. Pour plus d’informations, consultez [Vue d’ensemble de l’extension de script personnalisé](../extensions/custom-script-windows.md).

Utilisez l’extension de script personnalisé pour installer un serveur web IIS de base. Appliquez l’extension de script personnalisé qui installe IIS comme suit :

```azurepowershell-interactive
# Define the script for your Custom Script Extension to run
$publicSettings = @{
    "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate-iis.ps1");
    "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis.ps1"
}

# Get information about the scale set
$vmss = Get-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet"

# Use Custom Script Extension to install IIS and configure basic website
Add-AzVmssExtension -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.8 `
  -Setting $publicSettings

# Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

## <a name="allow-traffic-to-application"></a>Autoriser le trafic vers l’application

Pour autoriser l’accès à l’application web de base, créez un groupe de sécurité réseau avec [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) et [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). Pour plus d’informations, consultez [Fonctionnalités réseau pour les groupes de machines virtuelles identiques Azure](../../virtual-machine-scale-sets/virtual-machine-scale-sets-networking.md).

```azurepowershell-interactive
# Get information about the scale set
$vmss = Get-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet"

#Create a rule to allow traffic over port 80
$nsgFrontendRule = New-AzNetworkSecurityRuleConfig `
  -Name myFrontendNSGRule `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 200 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80 `
  -Access Allow

#Create a network security group and associate it with the rule
$nsgFrontend = New-AzNetworkSecurityGroup `
  -ResourceGroupName  "myResourceGroupScaleSet" `
  -Location EastUS `
  -Name myFrontendNSG `
  -SecurityRules $nsgFrontendRule

$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName  "myResourceGroupScaleSet" `
  -Name myVnet

$frontendSubnet = $vnet.Subnets[0]

$frontendSubnetConfig = Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name mySubnet `
  -AddressPrefix $frontendSubnet.AddressPrefix `
  -NetworkSecurityGroup $nsgFrontend

Set-AzVirtualNetwork -VirtualNetwork $vnet

# Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

## <a name="test-your-scale-set"></a>Tester votre groupe identique
Pour voir votre groupe identique en action, obtenez l’adresse IP publique de votre équilibreur de charge avec [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress). L’exemple suivant affiche l’adresse IP pour *myPublicIP* qui a été créée dans le cadre du groupe identique :

```azurepowershell-interactive
Get-AzPublicIPAddress `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -Name "myPublicIPAddress" | select IpAddress
```

Entrez l’adresse IP publique dans un navigateur web. L’application Web s’affiche, avec notamment le nom d’hôte de la machine virtuelle sur laquelle l’équilibrage de charge a distribué le trafic :

![Site IIS en cours d’exécution](./media/tutorial-create-vmss/running-iis-site.png)

Pour voir le groupe identique en action, vous pouvez forcer l’actualisation de votre navigateur web pour visualiser la distribution de trafic par l’équilibrage de charge sur l’ensemble des machines virtuelles exécutant votre application.


## <a name="management-tasks"></a>Tâches de gestion
Tout au long du cycle de vie du groupe identique, vous devrez peut-être exécuter une ou plusieurs tâches de gestion. En outre, vous souhaiterez peut-être créer des scripts pour automatiser les diverses tâches liées au cycle de vie. Azure PowerShell offre un moyen rapide d’effectuer ces tâches. Voici quelques tâches courantes.

### <a name="view-vms-in-a-scale-set"></a>Afficher les machines virtuelles d’un groupe identique
Pour afficher la liste des instances de machine virtuelle dans un groupe identique, utilisez [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm) comme suit :

```azurepowershell-interactive
Get-AzVmssVM `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet"
```

L’exemple suivant montre deux instances de machine virtuelle dans le groupe identique :

```powershell
ResourceGroupName                 Name Location             Sku InstanceID ProvisioningState
-----------------                 ---- --------             --- ---------- -----------------
MYRESOURCEGROUPSCALESET   myScaleSet_0   eastus Standard_DS1_v2          0         Succeeded
MYRESOURCEGROUPSCALESET   myScaleSet_1   eastus Standard_DS1_v2          1         Succeeded
```

Pour afficher des informations supplémentaires sur une instance spécifique de la machine virtuelle, ajoutez le paramètre `-InstanceId` à [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm). L’exemple suivant présente des informations sur l’instance de machine virtuelle *1* :

```azurepowershell-interactive
Get-AzVmssVM `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet" `
  -InstanceId "1"
```


### <a name="increase-or-decrease-vm-instances"></a>Augmenter ou diminuer les instances de machines virtuelles
Pour afficher le nombre d’instances actuellement présentes dans un groupe identique, utilisez [Get-AzVmss](/powershell/module/az.compute/get-azvmss) et interrogez *sku.capacity* :

```azurepowershell-interactive
Get-AzVmss -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet" | `
  Select -ExpandProperty Sku
```

Vous pouvez ensuite augmenter ou diminuer manuellement le nombre de machines virtuelles présentes dans le groupe identique avec [Update-AzVmss](/powershell/module/az.compute/update-azvmss). L’exemple suivant fixe le nombre de machines virtuelles présentes dans votre groupe identique à *3* :

```azurepowershell-interactive
# Get current scale set
$scaleset = Get-AzVmss `
  -ResourceGroupName "myResourceGroupScaleSet" `
  -VMScaleSetName "myScaleSet"

# Set and update the capacity of your scale set
$scaleset.sku.capacity = 3
Update-AzVmss -ResourceGroupName "myResourceGroupScaleSet" `
    -Name "myScaleSet" `
    -VirtualMachineScaleSet $scaleset
```

Plusieurs minutes sont nécessaires avant que le nombre d’instances spécifié soit mis à jour pour votre groupe identique.


### <a name="configure-autoscale-rules"></a>Configurer des règles de mise à l’échelle automatique
Au lieu d’adapter manuellement le nombre d’instances présentes dans votre groupe identique, vous pouvez définir des règles de mise à l’échelle automatique. Ces règles surveillent les instances présentes dans votre groupe identique et répondent en conséquence en fonction des métriques et des seuils que vous définissez. L’exemple suivant augmente le nombre d’instances d’une unité dès que la charge moyenne du processeur dépasse 60 % sur une période de 5 minutes. Si la charge moyenne du processeur descend ensuite en dessous de 30 % sur une période de 5 minutes, le nombre d’instances diminue d’une unité :

```azurepowershell-interactive
# Define your scale set information
$mySubscriptionId = (Get-AzSubscription)[0].Id
$myResourceGroup = "myResourceGroupScaleSet"
$myScaleSet = "myScaleSet"
$myLocation = "East US"
$myScaleSetId = (Get-AzVmss -ResourceGroupName $myResourceGroup -VMScaleSetName $myScaleSet).Id 

# Create a scale up rule to increase the number instances after 60% average CPU usage exceeded for a 5-minute period
$myRuleScaleUp = New-AzAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId $myScaleSetId `
  -Operator GreaterThan `
  -MetricStatistic Average `
  -Threshold 60 `
  -TimeGrain 00:01:00 `
  -TimeWindow 00:05:00 `
  -ScaleActionCooldown 00:05:00 `
  -ScaleActionDirection Increase `
  -ScaleActionValue 1

# Create a scale down rule to decrease the number of instances after 30% average CPU usage over a 5-minute period
$myRuleScaleDown = New-AzAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId $myScaleSetId `
  -Operator LessThan `
  -MetricStatistic Average `
  -Threshold 30 `
  -TimeGrain 00:01:00 `
  -TimeWindow 00:05:00 `
  -ScaleActionCooldown 00:05:00 `
  -ScaleActionDirection Decrease `
  -ScaleActionValue 1

# Create a scale profile with your scale up and scale down rules
$myScaleProfile = New-AzAutoscaleProfile `
  -DefaultCapacity 2  `
  -MaximumCapacity 10 `
  -MinimumCapacity 2 `
  -Rule $myRuleScaleUp,$myRuleScaleDown `
  -Name "autoprofile"

# Apply the autoscale rules
Add-AzAutoscaleSetting `
  -Location $myLocation `
  -Name "autosetting" `
  -ResourceGroup $myResourceGroup `
  -TargetResourceId $myScaleSetId `
  -AutoscaleProfile $myScaleProfile
```

Pour plus d’informations de conception sur l’utilisation de la mise à l’échelle automatique, consultez [Meilleures pratiques relatives à la mise à l’échelle automatique](/azure/architecture/best-practices/auto-scaling).


## <a name="next-steps"></a>Étapes suivantes
Ce didacticiel vous a montré comment créer un groupe de machines virtuelles identiques. Vous avez appris à :

> [!div class="checklist"]
> * Utiliser l’extension de script personnalisé pour définir un site IIS à mettre à l’échelle
> * Créer un équilibrage de charge pour votre groupe identique
> * Créer un groupe de machines virtuelles identiques
> * Augmenter ou réduire le nombre d’instances dans un groupe identique
> * Créer des règles de mise à l’échelle automatique

Passez au didacticiel suivant pour en savoir plus sur les concepts de l’équilibrage de charge des machines virtuelles.

> [!div class="nextstepaction"]
> [Équilibrage de charge des machines virtuelles](tutorial-load-balancer.md)
