---
title: Configurer la préférence de routage pour une machine virtuelle – Azure PowerShell
description: Apprenez à créer une machine virtuelle avec une adresse IP publique et choix d’une préférence de routage à l’aide d’Azure PowerShell.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/01/2020
ms.author: mnayak
ms.openlocfilehash: 65f5b27ecc7b526c4b3e4f5ed62ac90149df1ce1
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96510605"
---
# <a name="configure-routing-preference-for-a-vm-using-azure-powershell"></a>Configurer la préférence de routage pour une machine virtuelle à l’aide d’Azure PowerShell

Cet article explique comment configurer la préférence de routage pour une machine virtuelle. Le trafic lié à Internet à partir de la machine virtuelle est routé via le réseau du fournisseur de services Internet lorsque vous choisissez l’option de préférence de routage **Internet**. Le routage par défaut passe par le réseau Microsoft mondial.

Cet article explique comment créer une machine virtuelle avec une adresse IP publique définie pour acheminer le trafic via le réseau du fournisseur de services Internet à l’aide d’Azure PowerShell.

> [!IMPORTANT]
> La préférence de routage est actuellement en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="create-a-resource-group"></a>Créer un groupe de ressources
1. Si vous utilisez Cloud Shell, passez à l’étape 2. Ouvrez une session de commande et connectez-vous à Azure avec `Connect-AzAccount`.
2. Créez un groupe de ressources avec la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). L’exemple suivant crée un groupe de ressources dans la région Azure USA Est :

    ```azurepowershell
    $rg = New-AzResourceGroup -Name MyResourceGroup -Location EastUS
    ```

## <a name="create-a-public-ip-address"></a>Créer une adresse IP publique

Pour accéder à vos machines virtuelles à partir d’Internet, vous avez besoin d’adresses IP publiques. Créez des adresses IP publiques avec la commande [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress). L’exemple suivant crée une adresse IP publique IPv4 nommée *MyPublicIP* avec un type de préférence de routage *Internet* dans le groupe de ressources *MyResourceGroup* situé dans la région *USA Est* :

```azurepowershell-interactive
$iptagtype="RoutingPreference"
$tagName = "Internet"
$ipTag = New-AzPublicIpTag -IpTagType $iptagtype -Tag $tagName 
# attach the tag
$publicIp = New-AzPublicIpAddress  `
-Name "MyPublicIP" `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location `
-IpTag $ipTag `
-AllocationMethod Static `
-Sku Standard `
-IpAddressVersion IPv4
```

## <a name="create-network-resources"></a>Créer des ressources réseau

Avant de déployer un machine virtuelle, vous devez créer des ressources réseau associées (groupe de sécurité réseau, réseau virtuel et carte réseau virtuelle).

### <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Créez un groupe de sécurité réseau avec [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). L’exemple suivant permet de créer un groupe de sécurité réseau nommé *myNSG*

```azurepowershell
$nsg = New-AzNetworkSecurityGroup `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-Name "myNSG"
```

### <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Créez un réseau virtuel avec [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). L’exemple suivant crée un réseau virtuel nommé *myVNET* avec un sous-réseau nommé *mySubNet* :

### <a name="create-a-subnet"></a>Créer un sous-réseau

```azurepowershell
$subnet = New-AzVirtualNetworkSubnetConfig `
-Name "mySubnet" `
-AddressPrefix "10.0.0.0/24"
```

```azurepowershell
# Create a virtual network
$vnet = New-AzVirtualNetwork `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-Name "myVNET" `
-AddressPrefix "10.0.0.0/16" `
-Subnet $subnet
```

### <a name="create-a-nic"></a>Créer une carte réseau

Créer des cartes réseau virtuelles avec [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface. L’exemple suivant crée une carte réseau virtuelle.

```azurepowershell
# Create an IP Config
$ipconfig=New-AzNetworkInterfaceIpConfig `
-Name myIpConfig `
-Subnet $vnet.subnets[0] `
-PrivateIpAddressVersion IPv4 `
-PublicIpAddress  $publicIp

# Create a NIC
$nic = New-AzNetworkInterface `
-Name "mynic" `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-NetworkSecurityGroupId $nsg.Id `
-IpConfiguration $ipconfig 
```

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Définissez un nom d’utilisateur administrateur et un mot de passe pour les machines virtuelles avec [Get-Credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) :

```azurepowershell
 $cred = get-credential -Message "Routing Preference SAMPLE:  Please enter the Administrator credential to log into the VM."
```

Vous pouvez maintenant créer la machine virtuelle avec [New-AzVM](/powershell/module/az.compute/new-azvm). L’exemple suivant crée deux machines virtuelles ainsi que les composants de réseau virtuel nécessaires s’ils n’existent pas encore.

```azurepowershell
 $vmsize = "Standard_A2"
 $ImagePublisher = "MicrosoftWindowsServer"
 $imageOffer = "WindowsServer"
 $imageSKU = "2019-Datacenter"

 $vmName= "myVM"
 $vmconfig = New-AzVMConfig -VMName $vmName -VMSize $vmsize | Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate | Set-AzVMSourceImage -PublisherName $ImagePublisher -Offer $imageOffer -Skus $imageSKU -Version "latest" | Set-AzVMOSDisk -Name "$vmName.vhd" -CreateOption "FromImage" | Add-AzVMNetworkInterface -Id $nic.Id 
 $VM1 = New-AzVM -ResourceGroupName $rg.ResourceGroupName  -Location $rg.Location  -VM $vmconfig
```

## <a name="allow-network-traffic-to-the-vm"></a>Autoriser le trafic réseau vers la machine virtuelle

Avant de connecter à l’adresse IP publique à partir d’Internet, vérifiez que les ports nécessaires sont ouverts dans tout groupe de sécurité réseau que vous avez associé à l’interface réseau, au sous-réseau dans lequel se trouve l’interface réseau ou aux deux. Vous pouvez afficher les règles de sécurité effectives pour une interface réseau et de son sous-réseau à l’aide du [Portail Microsoft Azure](diagnose-network-traffic-filter-problem.md#diagnose-using-azure-portal), de [l’interface de ligne de commande](diagnose-network-traffic-filter-problem.md#diagnose-using-azure-cli) ou de [PowerShell](diagnose-network-traffic-filter-problem.md#diagnose-using-powershell).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, vous pouvez utiliser la commande [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées.

 ```azurepowershell
 Remove-AzResourceGroup -Name MyResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur la [préférence de routage dans les adresses IP publiques](routing-preference-overview.md).
* En savoir plus sur les [adresses IP publiques](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses) dans Azure.
* En savoir plus sur les [paramètres d’adresse IP publique](virtual-network-public-ip-address.md#create-a-public-ip-address).
