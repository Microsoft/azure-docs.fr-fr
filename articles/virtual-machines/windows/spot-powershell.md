---
title: Utiliser PowerShell pour déployer des machines virtuelles Azure Spot
description: Découvrez comment utiliser Azure PowerShell afin de déployer des machines virtuelles Spot pour réaliser des économies sur les coûts.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 06/26/2020
ms.author: cynthn
ms.reviewer: jagaveer
ms.openlocfilehash: 44d23710db169fa27aaba8928d421918bef93fec
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91825128"
---
# <a name="deploy-spot-vms-using-azure-powershell"></a>Déployer des machines virtuelles Spot à l’aide d’Azure PowerShell


L’utilisation de [machines virtuelles Spot](../spot-vms.md) vous permet de tirer parti de notre capacité inutilisée en réalisant des économies significatives. Dès qu’Azure a besoin de récupérer toute la capacité, l’infrastructure Azure supprime les machines virtuelles Spot. Les machines virtuelles Spot sont donc appropriées pour les charges de travail capables de gérer les interruptions, comme les travaux de traitement par lots, les environnements de développement et de test, les charges de travail de calcul importantes, entre autres.

Les tarifs des machines virtuelles Spot sont variables, en fonction de la région et de la référence SKU. Pour plus d’informations, consultez les prix des machines virtuelles pour [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) et [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). Pour plus d’informations sur la définition du prix maximal, consultez [Machines virtuelles Spot - Tarifs](../spot-vms.md#pricing).

Vous avez la possibilité de définir un prix maximal que vous êtes prêt à payer, par heure, pour la machine virtuelle. Le prix maximal d’une machine virtuelle Spot peut être défini en dollars américains (USD), en utilisant jusqu’à 5 décimales. Par exemple, la valeur `0.98765` correspond à un prix maximal de 0,98765 $ USD par heure. Si vous définissez `-1` comme prix maximal, la machine virtuelle n’est pas supprimée en fonction du prix. Le prix de la machine virtuelle sera le prix actuel de Spot ou le prix d’une machine virtuelle standard, la valeur la plus faible étant retenue, à condition que la capacité et le quota soient disponibles.


## <a name="create-the-vm"></a>Création de la machine virtuelle

Créez une machine virtuelle Spot avec [New-AzVmConfig](/powershell/module/az.compute/new-azvmconfig) pour créer la configuration. Incluez `-Priority Spot` et affectez à `-MaxPrice` :
- `-1` afin que la machine virtuelle ne soit pas supprimée en fonction du prix.
- un montant en dollars, jusqu’à 5 chiffres. Par exemple, `-MaxPrice .98765` signifie que la machine virtuelle sera désallouée une fois que le prix d’une machine virtuelle Spot atteindra 0,98765 $ par heure.


Cet exemple crée une machine virtuelle Spot qui n’est pas désallouée en fonction du prix (uniquement quand Azure a besoin de récupérer toute la capacité). La stratégie d’éviction est définie pour désallouer la machine virtuelle, afin qu’elle puisse être redémarrée ultérieurement. Si vous souhaitez supprimer la machine virtuelle et le disque sous-jacent lorsque la machine virtuelle est supprimée, définissez `-EvictionPolicy` sur `Delete` dans `New-AzVMConfig`.


```azurepowershell-interactive
$resourceGroup = "mySpotRG"
$location = "eastus"
$vmName = "mySpotVM"
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."
New-AzResourceGroup -Name $resourceGroup -Location $location
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
   -Name mySubnet -AddressPrefix 192.168.1.0/24
$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroup `
   -Location $location -Name MYvNET -AddressPrefix 192.168.0.0/16 `
   -Subnet $subnetConfig
$pip = New-AzPublicIpAddress -ResourceGroupName $resourceGroup -Location $location `
  -Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleRDP  -Protocol Tcp `
  -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 3389 -Access Allow
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $resourceGroup -Location $location `
  -Name myNetworkSecurityGroup -SecurityRules $nsgRuleRDP
$nic = New-AzNetworkInterface -Name myNic -ResourceGroupName $resourceGroup -Location $location `
  -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

# Create a virtual machine configuration and set this to be a Spot VM

$vmConfig = New-AzVMConfig -VMName $vmName -VMSize Standard_D1 -Priority "Spot" -MaxPrice -1 -EvictionPolicy Deallocate | `
Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred | `
Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2016-Datacenter -Version latest | `
Add-AzVMNetworkInterface -Id $nic.Id

New-AzVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig
```

Une fois la machine virtuelle créée, vous pouvez effectuer une requête pour afficher le prix maximal pour toutes les machines virtuelles du groupe de ressources.

```azurepowershell-interactive
Get-AzVM -ResourceGroupName $resourceGroup | `
   Select-Object Name,@{Name="maxPrice"; Expression={$_.BillingProfile.MaxPrice}}
```

## <a name="simulate-an-eviction"></a>Simuler une éviction

Vous pouvez [simuler l’éviction](/rest/api/compute/virtualmachines/simulateeviction) d’une machine virtuelle spot, afin de tester l’efficacité de la réponse de votre application en vue d’une éviction soudaine. 

Remplacez les éléments suivants avec vos informations : 

- `subscriptionId`
- `resourceGroupName`
- `vmName`


```http
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/simulateEviction?api-version=2020-06-01
```

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également créer une machine virtuelle Spot à l’aide d’[Azure CLI](../linux/spot-cli.md), du [portail](../spot-portal.md) ou d’un [modèle](../linux/spot-template.md).

Interroger les informations de tarification actuelles à l’aide de l’[API des prix de vente au détail d’Azure](/rest/api/cost-management/retail-prices/azure-retail-prices) pour plus d’informations sur la tarification Spot. Les `meterName` et `skuName` contiennent tous les deux `Spot`.

Si vous rencontrez une erreur, consultez [Codes d’erreur](../error-codes-spot.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).