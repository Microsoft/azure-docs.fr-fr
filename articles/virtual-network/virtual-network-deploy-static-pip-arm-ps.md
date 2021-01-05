---
title: Créer une machine virtuelle avec une adresse IP publique statique - PowerShell | Microsoft Docs
description: Créer une machine virtuelle avec une adresse IP publique statique à l’aide de PowerShell Les adresses IP publiques statiques sont des adresses qui ne changent jamais.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
ms.subservice: ip-services
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/08/2018
ms.author: allensu
ms.openlocfilehash: 3ca83836771af8448f6510ab27d0ac5f2973b35c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87287706"
---
# <a name="create-a-virtual-machine-with-a-static-public-ip-address-using-powershell"></a>Créer une machine virtuelle avec une adresse IP publique statique à l’aide de PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vous pouvez créer une machine virtuelle avec une adresse IP publique statique. Une adresse IP publique vous permet de communiquer avec une machine virtuelle à partir d’Internet. Attribuez une adresse IP publique statique, plutôt qu’une adresse dynamique, pour être certain que l’adresse ne changera jamais. Découvrez plus en détail les [adresses IP publiques statiques](virtual-network-ip-addresses-overview-arm.md#allocation-method). Pour changer le type de l’adresse IP publique attribuée à une machine virtuelle existante (de dynamique à statique), ou pour utiliser des adresses IP privées, consultez [Ajouter, modifier ou supprimer des adresses IP](virtual-network-network-interface-addresses.md). Les adresses IP publiques ont un [coût modique](https://azure.microsoft.com/pricing/details/ip-addresses) et le nombre d’adresses IP publiques que vous pouvez utiliser par abonnement est [limité](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Vous pouvez effectuer les étapes suivantes à partir de votre ordinateur local ou à l’aide d’Azure Cloud Shell. Pour pouvoir utiliser votre ordinateur local, vous devez avoir installé [Azure PowerShell](/powershell/azure/install-az-ps?toc=%2fazure%2fvirtual-network%2ftoc.json). Pour utiliser Azure Cloud Shell, sélectionnez **Essayer** dans l’angle supérieur droit de l’une des zones de commande suivantes. Cloud Shell vous connecte à Azure.

1. Si vous utilisez Cloud Shell, passez à l’étape 2. Ouvrez une session de commande et connectez-vous à Azure avec `Connect-AzAccount`.
2. Créez un groupe de ressources avec la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). L’exemple suivant crée un groupe de ressources dans la région Azure USA Est :

   ```azurepowershell-interactive
   New-AzResourceGroup -Name myResourceGroup -Location EastUS
   ```

3. Créez une machine virtuelle avec la commande [New-AzVM](/powershell/module/az.Compute/New-azVM). L’option `-AllocationMethod "Static"` attribue une adresse IP publique statique à la machine virtuelle. L’exemple suivant crée une machine virtuelle Windows Server avec une adresse IP statique de base (référence SKU) nommée *myPublicIpAddress*. Quand vous y êtes invité, indiquez le nom d’utilisateur et le mot de passe à utiliser comme informations d’identification de connexion pour la machine virtuelle :

   ```azurepowershell-interactive
   New-AzVm `
     -ResourceGroupName "myResourceGroup" `
     -Name "myVM" `
     -Location "East US" `
     -PublicIpAddressName "myPublicIpAddress" `
     -AllocationMethod "Static"
   ```

   Si l’adresse IP publique doit être une référence SKU standard, vous devez [créer une adresse IP publique](virtual-network-public-ip-address.md#create-a-public-ip-address), [créer une interface réseau](virtual-network-network-interface.md#create-a-network-interface), [attribuer l’adresse IP publique à l’interface réseau](virtual-network-network-interface-addresses.md#add-ip-addresses), puis [créer une machine virtuelle avec l’interface réseau](virtual-network-network-interface-vm.md#add-existing-network-interfaces-to-a-new-vm), une étape après l’autre. Découvrez plus en détail les [références SKU d’adresses IP publiques](virtual-network-ip-addresses-overview-arm.md#sku). Si la machine virtuelle doit être ajoutée au pool backend d’un équilibreur de charge Azure public, la référence SKU de l’adresse IP publique de la machine virtuelle doit correspondre à celle de l’équilibreur de charge. Pour plus d’informations, consultez [Azure Load Balancer](../load-balancer/skus.md).

4. Examinez l’adresse IP publique qui a été affectée et vérifiez qu’elle a été créée en tant qu’adresse statique, à l’aide de [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) :

   ```azurepowershell-interactive
   Get-AzPublicIpAddress `
     -ResourceGroupName "myResourceGroup" `
     -Name "myPublicIpAddress" `
     | Select "IpAddress", "PublicIpAllocationMethod" `
     | Format-Table
   ```

   Azure a affecté une adresse IP publique parmi les adresses utilisées dans la région dans laquelle vous avez créé la machine virtuelle. Vous pouvez télécharger la liste des plages (préfixes) pour les clouds Azure [Public](https://www.microsoft.com/download/details.aspx?id=56519), [Gouvernement américain](https://www.microsoft.com/download/details.aspx?id=57063), [Chine](https://www.microsoft.com/download/details.aspx?id=57062), et [Allemagne](https://www.microsoft.com/download/details.aspx?id=57064).

> [!WARNING]
> Ne modifiez pas les paramètres d’adresse IP dans le système d’exploitation de la machine virtuelle. Le système d’exploitation ne reconnaît pas les adresses IP publiques Azure. Même si vous avez la possibilité d’ajouter des paramètres d’adresse IP privée au système d’exploitation, nous vous déconseillons de le faire, à moins que cela soit nécessaire et pas avant d’avoir lu [Ajouter une adresse IP privée à un système d’exploitation](virtual-network-network-interface-addresses.md#private).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’un groupe de ressources, vous pouvez utiliser [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour le supprimer ainsi que toutes les ressources qu’il contient :

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [adresses IP publiques](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses) dans Azure
- En savoir plus sur les [paramètres d’adresse IP publique](virtual-network-public-ip-address.md#create-a-public-ip-address)
- En savoir plus sur les [adresses IP privées](virtual-network-ip-addresses-overview-arm.md#private-ip-addresses) et l’affectation d’une [adresse IP privée statique](virtual-network-network-interface-addresses.md#add-ip-addresses) à une machine virtuelle Azure
- En savoir plus sur la création de machines virtuelles [Linux](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json) et [Windows](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
