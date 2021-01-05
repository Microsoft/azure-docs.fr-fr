---
title: 'PowerShell : Utiliser des groupes de placements de proximité'
description: Découvrez comment créer et utiliser des groupes de placements de proximité à l’aide d’Azure PowerShell.
services: virtual-machines
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 01/27/2020
ms.author: cynthn
ms.reviewer: zivr
ms.openlocfilehash: 9ea986b338d977102d78e9c12bcbe5b2f2c510e7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87083444"
---
# <a name="deploy-vms-to-proximity-placement-groups-using-powershell"></a>Déployer des machines virtuelles dans des groupes de placements avec PowerShell


Pour que les machines virtuelles soient aussi proches que possible, avec la latence la plus faible possible, déployez-les dans un [groupe de placements de proximité](co-location.md#proximity-placement-groups).

Le groupe de placements de proximité est un regroupement logique utilisé pour s’assurer que les ressources de calcul Azure se trouvent proches les unes des autres. Les groupes de placements de proximité sont utiles pour les charges de travail où une latence faible est requise.


## <a name="create-a-proximity-placement-group"></a>Créer un groupe de placements de proximité
Créez un groupe de placements de proximité à l’aide de la cmdlet [New-AzProximityPlacementGroup](/powershell/module/az.compute/new-azproximityplacementgroup). 

```azurepowershell-interactive
$resourceGroup = "myPPGResourceGroup"
$location = "East US"
$ppgName = "myPPG"
New-AzResourceGroup -Name $resourceGroup -Location $location
$ppg = New-AzProximityPlacementGroup `
   -Location $location `
   -Name $ppgName `
   -ResourceGroupName $resourceGroup `
   -ProximityPlacementGroupType Standard
```

## <a name="list-proximity-placement-groups"></a>Liste de groupes de placements de proximité

Vous pouvez créer une liste de tous les groupes de placements de proximité à l’aide de la cmdlet [Get-AzProximityPlacementGroup](/powershell/module/az.compute/get-azproximityplacementgroup).

```azurepowershell-interactive
Get-AzProximityPlacementGroup
```


## <a name="create-a-vm"></a>Créer une machine virtuelle

Créez une machine virtuelle dans le groupe de placements de proximité avec `-ProximityPlacementGroup $ppg.Id` pour faire référence à l’ID de groupe de placements de proximité quand vous utilisez [New-AzVM](/powershell/module/az.compute/new-azvm) pour créer la machine virtuelle.

```azurepowershell-interactive
$vmName = "myVM"

New-AzVm `
  -ResourceGroupName $resourceGroup `
  -Name $vmName `
  -Location $location `
  -OpenPorts 3389 `
  -ProximityPlacementGroup $ppg.Id
```

Vous pouvez voir la machine virtuelle dans le groupe de placements avec [Get-AzProximityPlacementGroup](/powershell/module/az.compute/get-azproximityplacementgroup).

```azurepowershell-interactive
Get-AzProximityPlacementGroup -ResourceId $ppg.Id |
    Format-Table -Property VirtualMachines -Wrap
```

### <a name="move-an-existing-vm-into-a-proximity-placement-group"></a>Déplacer une machine virtuelle existante vers un groupe de placements de proximité

Vous pouvez également ajouter une machine virtuelle existante à un groupe de placements de proximité. Vous devez d’abord arrêter/libérer la machine virtuelle, puis la mettre à jour et la redémarrer.

```azurepowershell-interactive
$ppg = Get-AzProximityPlacementGroup -ResourceGroupName myPPGResourceGroup -Name myPPG
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
Stop-AzVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName
Update-AzVM -VM $vm -ResourceGroupName $vm.ResourceGroupName -ProximityPlacementGroupId $ppg.Id
Start-AzVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName
```

### <a name="move-an-existing-vm-out-of-a-proximity-placement-group"></a>Retirer une machine virtuelle existante d’un groupe de placements de proximité

Pour retirer une machine virtuelle d’un groupe de placements de proximité, vous devez d’abord l’arrêter/la libérer, puis la mettre à jour et la redémarrer.

```azurepowershell-interactive
$ppg = Get-AzProximityPlacementGroup -ResourceGroupName myPPGResourceGroup -Name myPPG
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
Stop-AzVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName
$vm.ProximityPlacementGroup = ""
Update-AzVM -VM $vm -ResourceGroupName $vm.ResourceGroupName 
Start-AzVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName
```


## <a name="availability-sets"></a>Groupes à haute disponibilité
Vous pouvez également créer un groupe à haute disponibilité dans votre groupe de placements de proximité. Utilisez le même paramètre `-ProximityPlacementGroup`avec la cmdlet [New-AzAvailabilitySet](/powershell/module/az.compute/new-azavailabilityset) pour créer un groupe à haute disponibilité et toutes les machines virtuelles créées dans le groupe à haute disponibilité seront également créées dans le même groupe de placements de proximité.

Pour ajouter un groupe à haute disponibilité existant à un groupe de placements de proximité ou pour l’en retirer, vous devez d’abord arrêter toutes les machines virtuelles dans le groupe à haute disponibilité. 

### <a name="move-an-existing-availability-set-into-a-proximity-placement-group"></a>Déplacer un groupe à haute disponibilité existant vers un groupe de placements de proximité

```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$avSetName = "myAvailabilitySet"
$avSet = Get-AzAvailabilitySet -ResourceGroupName $resourceGroup -Name $avSetName
$vmIds = $avSet.VirtualMachinesReferences
foreach ($vmId in $vmIDs){
    $string = $vmID.Id.Split("/")
    $vmName = $string[8]
    Stop-AzVM -ResourceGroupName $resourceGroup -Name $vmName -Force
    } 

$ppg = Get-AzProximityPlacementGroup -ResourceGroupName myPPG -Name myPPG
Update-AzAvailabilitySet -AvailabilitySet $avSet -ProximityPlacementGroupId $ppg.Id
foreach ($vmId in $vmIDs){
    $string = $vmID.Id.Split("/")
    $vmName = $string[8]
    Start-AzVM -ResourceGroupName $resourceGroup -Name $vmName 
    } 
```

### <a name="move-an-existing-availability-set-out-of-a-proximity-placement-group"></a>Retirer un groupe à haute disponibilité existant d’un groupe de placements de proximité

```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$avSetName = "myAvailabilitySet"
$avSet = Get-AzAvailabilitySet -ResourceGroupName $resourceGroup -Name $avSetName
$vmIds = $avSet.VirtualMachinesReferences
foreach ($vmId in $vmIDs){
    $string = $vmID.Id.Split("/")
    $vmName = $string[8]
    Stop-AzVM -ResourceGroupName $resourceGroup -Name $vmName -Force
    } 

$avSet.ProximityPlacementGroup = ""
Update-AzAvailabilitySet -AvailabilitySet $avSet 
foreach ($vmId in $vmIDs){
    $string = $vmID.Id.Split("/")
    $vmName = $string[8]
    Start-AzVM -ResourceGroupName $resourceGroup -Name $vmName 
    } 
```

## <a name="scale-sets"></a>Groupes identiques

Vous pouvez également créer un groupe identique dans votre groupe de placements de proximité. Utilisez le même paramètre `-ProximityPlacementGroup` avec [New-AzVmss](/powershell/module/az.compute/new-azvmss) pour créer un jeu de mise à l’échelle et toutes les instances seront créées dans le même groupe de placements de proximité.


Pour ajouter un groupe identique existant à un groupe de placements de proximité ou pour l’en retirer, vous devez d’abord arrêter le groupe identique. 

### <a name="move-an-existing-scale-set-into-a-proximity-placement-group"></a>Déplacer un groupe identique existant vers un groupe de placements de proximité

```azurepowershell-interactive
$ppg = Get-AzProximityPlacementGroup -ResourceGroupName myPPG -Name myPPG
$vmss = Get-AzVmss -ResourceGroupName myVMSSResourceGroup -VMScaleSetName myScaleSet
Stop-AzVmss -VMScaleSetName $vmss.Name -ResourceGroupName $vmss.ResourceGroupName
Update-AzVmss -VMScaleSetName $vmss.Name -ResourceGroupName $vmss.ResourceGroupName -ProximityPlacementGroupId $ppg.Id
Start-AzVmss -VMScaleSetName $vmss.Name -ResourceGroupName $vmss.ResourceGroupName
```

### <a name="move-an-existing-scale-set-out-of-a-proximity-placement-group"></a>Retirer un groupe identique existant d’un groupe de placements de proximité

```azurepowershell-interactive
$vmss = Get-AzVmss -ResourceGroupName myVMSSResourceGroup -VMScaleSetName myScaleSet
Stop-AzVmss -VMScaleSetName $vmss.Name -ResourceGroupName $vmss.ResourceGroupName
$vmss.ProximityPlacementGroup = ""
Update-AzVmss -VirtualMachineScaleSet $vmss -VMScaleSetName $vmss.Name -ResourceGroupName $vmss.ResourceGroupName  
Start-AzVmss -VMScaleSetName $vmss.Name -ResourceGroupName $vmss.ResourceGroupName
```

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également utiliser [Azure CLI](../linux/proximity-placement-groups.md) pour créer des groupes de placements de proximité.
