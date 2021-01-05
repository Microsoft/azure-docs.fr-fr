---
title: Déployer des hôtes dédiés Azure à l’aide d’Azure PowerShell
description: Déployez des machines virtuelles sur des hôtes dédiés à l’aide d’Azure PowerShell.
author: cynthn
ms.service: virtual-machines-windows
ms.topic: how-to
ms.workload: infrastructure
ms.date: 11/12/2020
ms.author: cynthn
ms.reviewer: zivr
ms.openlocfilehash: 2f8f2d9eb14e1272af126c9a6d6663f41aaee33f
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97005084"
---
# <a name="deploy-vms-to-dedicated-hosts-using-the-azure-powershell"></a>Déployez des machines virtuelles sur des hôtes dédiés à l’aide d’Azure PowerShell

Cet article vous guide dans la création d’un [hôte dédié](../dedicated-hosts.md) Azure pour héberger vos machines virtuelles. 

Assurez-vous d’avoir installé Azure PowerShell version 2.8.0 ou ultérieure et d’être connecté à un compte Azure avec `Connect-AzAccount`. 

## <a name="limitations"></a>Limites

- Actuellement, les hôtes dédiés ne prennent pas en charge les groupes de machines virtuelles identiques.
- Les tailles et les types de matériel disponibles pour les hôtes dédiés varient selon la région. Pour en savoir plus, consultez la [page de tarification de l’hôte](https://aka.ms/ADHPricing).

## <a name="create-a-host-group"></a>Créer un groupe hôte

Un **groupe hôte** est une ressource qui représente une collection d’hôtes dédiés. Vous créez un groupe hôte dans une région et une zone de disponibilité, et lui ajoutez des hôtes. Lors de la planification de la haute disponibilité, vous pouvez accéder à des options supplémentaires. Vous pouvez utiliser l’une des options suivantes, ou les deux, avec vos hôtes dédiés : 
- Application sur plusieurs zones de disponibilité. Dans ce cas, vous devez disposer d’un groupe hôte dans chacune des zones que vous souhaitez utiliser.
- Application sur plusieurs domaines d’erreur mappés à des racks physiques. 
 
Dans les deux cas, vous devez fournir le nombre de domaines d’erreur pour votre groupe hôte. Si vous ne souhaitez pas appliquer plusieurs domaines d’erreur dans votre groupe, utilisez un seul domaine d’erreur. 

Vous pouvez également choisir d’utiliser des zones de disponibilité et des domaines d’erreur. Cet exemple crée un groupe hôte dans la zone 1, avec 2 domaines d’erreur. 


```azurepowershell-interactive
$rgName = "myDHResourceGroup"
$location = "EastUS"

New-AzResourceGroup -Location $location -Name $rgName
$hostGroup = New-AzHostGroup `
   -Location $location `
   -Name myHostGroup `
   -PlatformFaultDomain 2 `
   -ResourceGroupName $rgName `
   -Zone 1
```


Ajoutez le paramètre `-SupportAutomaticPlacement true` pour que vos machines virtuelles et vos instances de groupes identiques soient placées automatiquement sur les hôtes au sein d’un groupe hôte. Pour plus d’informations, consultez [Sélection élective manuelle ou automatique](../dedicated-hosts.md#manual-vs-automatic-placement).


## <a name="create-a-host"></a>Créer un hôte

À présent, créons un hôte dédié dans le groupe hôte. En plus d’un nom pour l’hôte, vous devez fournir la référence SKU pour l’hôte. La référence SKU de l’hôte capture la série de machines virtuelles prises en charge, ainsi que la génération du matériel pour l’hôte dédié.

Pour en savoir plus sur les références SKU et la tarification des hôtes, consultez la section relative à la [tarification des hôtes dédiés Azure](https://aka.ms/ADHPricing).

Si vous définissez un nombre de domaines d’erreur pour votre groupe hôte, vous êtes invité à spécifier le domaine d’erreur de votre hôte. Dans cet exemple, nous définissons le domaine d’erreur de l’hôte sur 1.


```azurepowershell-interactive
$dHost = New-AzHost `
   -HostGroupName $hostGroup.Name `
   -Location $location -Name myHost `
   -ResourceGroupName $rgName `
   -Sku DSv3-Type1 `
   -AutoReplaceOnFailure 1 `
   -PlatformFaultDomain 1
```

## <a name="create-a-vm"></a>Créer une machine virtuelle

Créez une machine virtuelle sur l’hôte dédié. 

Si vous avez spécifié une zone de disponibilité lors de la création de votre groupe hôte, vous devez utiliser la même zone quand vous créez la machine virtuelle. Pour cet exemple, étant donné que notre groupe hôte se trouve dans la zone 1, nous devons créer la machine virtuelle dans la zone 1.  


```azurepowershell-interactive
$cred = Get-Credential
New-AzVM `
   -Credential $cred `
   -ResourceGroupName $rgName `
   -Location $location `
   -Name myVM `
   -HostId $dhost.Id `
   -Image Win2016Datacenter `
   -Zone 1 `
   -Size Standard_D4s_v3
```

> [!WARNING]
> Toute machine virtuelle créée sur un hôte qui ne dispose pas d’un nombre de ressources suffisant présente un état ÉCHEC. 

## <a name="check-the-status-of-the-host"></a>Vérifier l’état de l’hôte

Vous pouvez vérifier l’état d’intégrité de l’hôte et le nombre de machines virtuelles que vous pouvez encore déployer sur l’hôte à l’aide de la commande [GetAzHost](/powershell/module/az.compute/get-azhost), avec le paramètre `-InstanceView`.

```azurepowershell-interactive
Get-AzHost `
   -ResourceGroupName $rgName `
   -Name myHost `
   -HostGroupName $hostGroup.Name `
   -InstanceView
```

Le résultat ressemble à ce qui suit :

```
ResourceGroupName      : myDHResourceGroup
PlatformFaultDomain    : 1
AutoReplaceOnFailure   : True
HostId                 : 12345678-1234-1234-abcd-abc123456789
ProvisioningTime       : 7/28/2019 5:31:01 PM
ProvisioningState      : Succeeded
InstanceView           : 
  AssetId              : abc45678-abcd-1234-abcd-123456789abc
  AvailableCapacity    : 
    AllocatableVMs[0]  : 
      VmSize           : Standard_D2s_v3
      Count            : 32
    AllocatableVMs[1]  : 
      VmSize           : Standard_D4s_v3
      Count            : 16
    AllocatableVMs[2]  : 
      VmSize           : Standard_D8s_v3
      Count            : 8
    AllocatableVMs[3]  : 
      VmSize           : Standard_D16s_v3
      Count            : 4
    AllocatableVMs[4]  : 
      VmSize           : Standard_D32-8s_v3
      Count            : 2
    AllocatableVMs[5]  : 
      VmSize           : Standard_D32-16s_v3
      Count            : 2
    AllocatableVMs[6]  : 
      VmSize           : Standard_D32s_v3
      Count            : 2
    AllocatableVMs[7]  : 
      VmSize           : Standard_D64-16s_v3
      Count            : 1
    AllocatableVMs[8]  : 
      VmSize           : Standard_D64-32s_v3
      Count            : 1
    AllocatableVMs[9]  : 
      VmSize           : Standard_D64s_v3
      Count            : 1
  Statuses[0]          : 
    Code               : ProvisioningState/succeeded
    Level              : Info
    DisplayStatus      : Provisioning succeeded
    Time               : 7/28/2019 5:31:01 PM
  Statuses[1]          : 
    Code               : HealthState/available
    Level              : Info
    DisplayStatus      : Host available
Sku                    : 
  Name                 : DSv3-Type1
Id                     : /subscriptions/10101010-1010-1010-1010-101010101010/re
sourceGroups/myDHResourceGroup/providers/Microsoft.Compute/hostGroups/myHostGroup/hosts
/myHost
Name                   : myHost
Location               : eastus
Tags                   : {}
```

## <a name="create-a-scale-set"></a>Créer un groupe identique 

Lorsque vous déployez un groupe identique, vous spécifiez le groupe hôte.

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -VMScaleSetName "myDHScaleSet" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicyMode "Automatic"`
  -HostGroupId $hostGroup.Id
```

Si vous souhaitez choisir manuellement l’hôte sur lequel déployer le groupe identique, ajoutez `--host` et le nom de l’hôte.



## <a name="add-an-existing-vm"></a>Ajouter une machine virtuelle existante 

Vous pouvez ajouter une machine virtuelle existante à un hôte dédié, mais la machine virtuelle doit d’abord être arrêtée\libérée. Avant de déplacer une machine virtuelle vers un hôte dédié, vérifiez que la configuration de la machine virtuelle est prise en charge :

- La taille de la machine virtuelle doit appartenir à la même famille de tailles que l’hôte dédié. Par exemple, si votre hôte dédié est DSv3, la taille de la machine virtuelle peut être Standard_D4s_v3, mais pas Standard_A4_v2. 
- La machine virtuelle doit être située dans la même région que l’hôte dédié.
- La machine virtuelle ne peut pas faire partie d’un groupe de placements de proximité. Supprimez la machine virtuelle du groupe de placements de proximité avant de la déplacer vers un hôte dédié. Pour plus d’informations, consultez [Déplacer une machine virtuelle hors d’un groupe de placements de proximité](./proximity-placement-groups.md#move-an-existing-vm-out-of-a-proximity-placement-group).
- La machine virtuelle ne peut pas se trouver dans un groupe à haute disponibilité.
- Si la machine virtuelle se trouve dans une zone de disponibilité, il doit s’agir de la même zone de disponibilité que celle du groupe hôte. Les paramètres de zone de disponibilité de la machine virtuelle et du groupe hôte doivent correspondre.

Remplacez les valeurs des variables par vos propres informations.

```azurepowershell-interactive
$vmRGName = "movetohost"
$vmName = "myVMtoHost"
$dhRGName = "myDHResourceGroup"
$dhGroupName = "myHostGroup"
$dhName = "myHost"

$myDH = Get-AzHost `
   -HostGroupName $dhGroupName `
   -ResourceGroupName $dhRGName `
   -Name $dhName
   
$myVM = Get-AzVM `
   -ResourceGroupName $vmRGName `
   -Name $vmName
   
$myVM.Host = New-Object Microsoft.Azure.Management.Compute.Models.SubResource

$myVM.Host.Id = "$myDH.Id"

Stop-AzVM `
   -ResourceGroupName $vmRGName `
   -Name $vmName -Force
   
Update-AzVM `
   -ResourceGroupName $vmRGName `
   -VM $myVM -Debug
   
Start-AzVM `
   -ResourceGroupName $vmRGName `
   -Name $vmName
```


## <a name="clean-up"></a>Nettoyer

Vous êtes facturé pour vos hôtes dédiés, même si aucune machine virtuelle n’est déployée. Supprimez tous les hôtes non utilisés pour réduire les coûts.  

Vous pouvez supprimer un hôte uniquement lorsqu’il n’est plus utilisé par aucune machine virtuelle. Supprimez les machines virtuelles avec la commande [Remove-AzVM](/powershell/module/az.compute/remove-azvm).

```azurepowershell-interactive
Remove-AzVM -ResourceGroupName $rgName -Name myVM
```

Cela fait, vous pouvez supprimer l’hôte avec la commande [Remove-AzHost](/powershell/module/az.compute/remove-azhost).

```azurepowershell-interactive
Remove-AzHost -ResourceGroupName $rgName -Name myHost
```

Une fois que vous avez supprimé tous vos hôtes, utilisez la commande [Remove-AzHostGroup](/powershell/module/az.compute/remove-azhostgroup) pour supprimer le groupe hôte. 

```azurepowershell-interactive
Remove-AzHost -ResourceGroupName $rgName -Name myHost
```

Vous pouvez également supprimer l’intégralité du groupe de ressources via une seule commande [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup). Cette opération supprime toutes les ressources créées dans le groupe, y compris l’ensemble des machines virtuelles, des hôtes et des groupes hôtes.
 
```azurepowershell-interactive
Remove-AzResourceGroup -Name $rgName
```


## <a name="next-steps"></a>Étapes suivantes

- Un exemple de modèle, disponible [ici](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-dedicated-hosts/README.md), utilise les zones et les domaines d’erreur pour offrir une résilience maximale dans une région.

- Le [Portail Microsoft Azure](../dedicated-hosts-portal.md) permet également de déployer des hôtes dédiés.