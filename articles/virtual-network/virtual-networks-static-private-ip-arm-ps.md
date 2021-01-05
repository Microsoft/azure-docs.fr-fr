---
title: Créer une machine virtuelle avec une adresse IP privée statique - Azure PowerShell
description: Apprenez à créer une machine virtuelle avec une adresse IP privée à l’aide de PowerShell.
services: virtual-network
documentationcenter: na
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
ms.assetid: d5f18929-15e3-40a2-9ee3-8188bc248ed8
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/07/2019
ms.author: kumud
ms.custom: ''
ms.openlocfilehash: ced76b73a8a08e6886cf0cef04c74a82d05c75dd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84708125"
---
# <a name="create-a-virtual-machine-with-a-static-private-ip-address-using-powershell"></a>Créer une machine virtuelle avec une adresse IP privée statique à l’aide de PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vous pouvez créer une machine virtuelle avec une adresse IP privée statique. Attribuez une adresse IP privée statique, plutôt qu’une adresse dynamique, si vous souhaitez sélectionner quelle adresse d’un sous-réseau attribuer à une machine virtuelle. Découvrez plus en détail les [adresses IP privées statiques](virtual-network-ip-addresses-overview-arm.md#allocation-method). Pour modifier le type de l’adresse IP privée attribuée à une machine virtuelle existante (de dynamique à statique), ou pour utiliser des adresses IP publiques, consultez [Add, change, or remove IP addresses for an Azure network interface](virtual-network-network-interface-addresses.md) (Ajouter, modifier ou supprimer des adresses IP pour une interface réseau Azure).

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Vous pouvez effectuer les étapes suivantes à partir de votre ordinateur local ou à l’aide d’Azure Cloud Shell. Pour pouvoir utiliser votre ordinateur local, vous devez avoir installé [Azure PowerShell](/powershell/azure/install-az-ps?toc=%2fazure%2fvirtual-network%2ftoc.json). Pour utiliser Azure Cloud Shell, sélectionnez **Essayer** dans l’angle supérieur droit de l’une des zones de commande suivantes. Cloud Shell vous connecte à Azure.

1. Si vous utilisez Cloud Shell, passez à l’étape 2. Ouvrez une session de commande et connectez-vous à Azure avec `Connect-AzAccount`.
2. Créez un groupe de ressources avec la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). L’exemple suivant crée un groupe de ressources dans la région Azure USA Est :

   ```azurepowershell-interactive
   $RgName = "myResourceGroup"
   $Location = "eastus"
   New-AzResourceGroup -Name $RgName -Location $Location
   ```

3. Créez une configuration de sous-réseau et un réseau virtuel avec les commandes [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) et [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) :

   ```azurepowershell-interactive
   # Create a subnet configuration
   $SubnetConfig = New-AzVirtualNetworkSubnetConfig `
   -Name MySubnet `
   -AddressPrefix 10.0.0.0/24

   # Create a virtual network
   $VNet = New-AzVirtualNetwork `
   -ResourceGroupName $RgName `
   -Location $Location `
   -Name MyVNet `
   -AddressPrefix 10.0.0.0/16 `
   -Subnet $subnetConfig

   # Get the subnet object for use in a later step.
   $Subnet = Get-AzVirtualNetworkSubnetConfig -Name $SubnetConfig.Name -VirtualNetwork $VNet
   ```

4. Créez une interface réseau dans le réseau virtuel et attribuez une adresse IP privée du sous-réseau à l’interface réseau avec les commandes [New-AzNetworkInterfaceIpConfig](/powershell/module/Az.Network/New-AzNetworkInterfaceIpConfig) et [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) :

   ```azurepowershell-interactive
   $IpConfigName1 = "IPConfig-1"
   $IpConfig1     = New-AzNetworkInterfaceIpConfig `
     -Name $IpConfigName1 `
     -Subnet $Subnet `
     -PrivateIpAddress 10.0.0.4 `
     -Primary

   $NIC = New-AzNetworkInterface `
     -Name MyNIC `
     -ResourceGroupName $RgName `
     -Location $Location `
     -IpConfiguration $IpConfig1
   ```

5. Créez une configuration de machine virtuelle avec [New-AzVMConfig](/powershell/module/Az.Compute/New-AzVMConfig), puis créez la machine virtuelle avec [New-AzVM](/powershell/module/az.Compute/New-azVM). Lorsque vous y êtes invité, indiquez un nom d’utilisateur et un mot de passe à utiliser comme informations d’identification pour vous connecter à la machine virtuelle :

   ```azurepowershell-interactive
   $VirtualMachine = New-AzVMConfig -VMName MyVM -VMSize "Standard_DS3"
   $VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName MyServerVM -ProvisionVMAgent -EnableAutoUpdate
   $VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
   $VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2012-R2-Datacenter' -Version latest
   New-AzVM -ResourceGroupName $RgName -Location $Location -VM $VirtualMachine -Verbose
   ```

> [!WARNING]
> Même si vous avez la possibilité d’ajouter des paramètres d’adresse IP privée au système d’exploitation, nous vous déconseillons de le faire avant d’avoir lu [Add a private IP address to an operating system](virtual-network-network-interface-addresses.md#private) (Ajouter une adresse IP privée à un système d’exploitation).
> 
> 
> <a name = "change-the-allocation-method-for-a-private-ip-address-assigned-to-a-network-interface"></a>
> 
> [!IMPORTANT]
> Pour accéder à la machine virtuelle à partir d’Internet, vous devez attribuer une adresse IP publique à la machine virtuelle. Vous pouvez également remplacer une attribution d’adresse IP privée dynamique en attribution statique. Pour plus d’informations, consultez [Add, change, or remove IP addresses for an Azure network interface](virtual-network-network-interface-addresses.md) (Ajouter, modifier ou supprimer des adresses IP pour une interface réseau Azure). En outre, il est recommandé de limiter le trafic réseau vers votre machine virtuelle en associant un groupe de sécurité réseau à l’interface réseau, au sous-réseau que vous avez créé dedans ou aux deux. Pour plus d’informations, consultez [Create, change, or delete a network security group](manage-network-security-group.md) (Créer, modifier ou supprimer un groupe de sécurité réseau).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’un groupe de ressources, vous pouvez utiliser [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour le supprimer ainsi que toutes les ressources qu’il contient :

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [adresses IP privées](virtual-network-ip-addresses-overview-arm.md#private-ip-addresses) et l’affectation d’une [adresse IP privée statique](virtual-network-network-interface-addresses.md#add-ip-addresses) à une machine virtuelle Azure.
- En savoir plus sur la création de machines virtuelles [Linux](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json) et [Windows](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
