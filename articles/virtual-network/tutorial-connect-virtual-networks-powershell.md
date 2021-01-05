---
title: Connecter des réseaux virtuels avec VNet Peering – Azure PowerShell
description: Dans cet article, vous apprendrez à connecter des réseaux virtuels à l’aide du peering de réseaux virtuels en utilisant Azure PowerShell.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
tags: azure-resource-manager
Customer intent: I want to connect two virtual networks so that virtual machines in one virtual network can communicate with virtual machines in the other virtual network.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: kumud
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6ec552ea525abe6a84bb5e34e00ad317cae038bf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89077838"
---
# <a name="connect-virtual-networks-with-virtual-network-peering-using-powershell"></a>Connecter des réseaux virtuels à l’aide du peering de réseaux virtuels en utilisant PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vous pouvez connecter des réseaux virtuels entre eux à l’aide du peering de réseaux virtuels. Une fois que les deux réseaux virtuels sont appairés, leurs ressources peuvent communiquer entre elles avec les mêmes bande passante et latence, comme si elles se trouvaient sur le même réseau virtuel. Dans cet article, vous apprendrez comment :

* Créer deux réseaux virtuels
* Connecter deux réseaux virtuels à l’aide du peering de réseaux virtuels
* Déployer une machine virtuelle sur chaque réseau virtuel
* Établir une communication entre les machines virtuelles

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser PowerShell en local, vous devez exécuter le module Azure PowerShell version 1.0.0 ou ultérieure pour les besoins de cet article. Exécutez `Get-Module -ListAvailable Az` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.

## <a name="create-virtual-networks"></a>Créer des réseaux virtuels

Avant de créer un réseau virtuel, vous devez créer un groupe de ressources pour le réseau virtuel et toutes les autres ressources créées dans cet article. Créez un groupe de ressources avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus*.

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

Créez un réseau virtuel avec [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). L’exemple suivant permet de créer un réseau virtuel nommé *myVirtualNetwork1* avec le préfixe d’adresse *10.0.0.0/16*.

```azurepowershell-interactive
$virtualNetwork1 = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork1 `
  -AddressPrefix 10.0.0.0/16
```

Créez une configuration de sous-réseau à l’aide de la commande [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig). L’exemple suivant crée une configuration de sous-réseau avec un préfixe d’adresse 10.0.0.0/24 :

```azurepowershell-interactive
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
  -Name Subnet1 `
  -AddressPrefix 10.0.0.0/24 `
  -VirtualNetwork $virtualNetwork1
```

Écrivez la configuration du sous-réseau dans le réseau virtuel à l’aide de la commande [Set-AzVirtualNetwork](/powershell/module/az.network/Set-azVirtualNetwork), ce qui crée le sous-réseau :

```azurepowershell-interactive
$virtualNetwork1 | Set-AzVirtualNetwork
```

Créez un réseau virtuel avec un préfixe d’adresse 10.1.0.0/16 et un sous-réseau :

```azurepowershell-interactive
# Create the virtual network.
$virtualNetwork2 = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork2 `
  -AddressPrefix 10.1.0.0/16

# Create the subnet configuration.
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
  -Name Subnet1 `
  -AddressPrefix 10.1.0.0/24 `
  -VirtualNetwork $virtualNetwork2

# Write the subnet configuration to the virtual network.
$virtualNetwork2 | Set-AzVirtualNetwork
```

## <a name="peer-virtual-networks"></a>Appairer des réseaux virtuels

Créez un Peering [Add-AzVirtualNetworkPeering](/powershell/module/az.network/add-azvirtualnetworkpeering). L’exemple suivant appaire *myVirtualNetwork1* avec *myVirtualNetwork2*.

```azurepowershell-interactive
Add-AzVirtualNetworkPeering `
  -Name myVirtualNetwork1-myVirtualNetwork2 `
  -VirtualNetwork $virtualNetwork1 `
  -RemoteVirtualNetworkId $virtualNetwork2.Id
```

Dans la sortie obtenue après l’exécution de la commande ci-dessus, vous constatez que la valeur de **PeeringState** (état de l’appairage) est *Initiated*. Le peering reste dans l’état *Initiated* jusqu’à ce que vous créiez le peering entre *myVirtualNetwork2* et *myVirtualNetwork1*. Créez un peering entre *myVirtualNetwork2* et *myVirtualNetwork1*.

```azurepowershell-interactive
Add-AzVirtualNetworkPeering `
  -Name myVirtualNetwork2-myVirtualNetwork1 `
  -VirtualNetwork $virtualNetwork2 `
  -RemoteVirtualNetworkId $virtualNetwork1.Id
```

Dans la sortie obtenue après l’exécution de la commande ci-dessus, vous constatez que **PeeringState** (état de l’appairage) est *Connected*. Azure a également changé l’état du peering *myVirtualNetwork1-myVirtualNetwork2* en *Connected*. Vérifiez que l’état du Peering *myVirtualNetwork1-myVirtualNetwork2* a été remplacé par *Connected* à l’aide de [Get-AzVirtualNetworkPeering](/powershell/module/az.network/get-azvirtualnetworkpeering).

```azurepowershell-interactive
Get-AzVirtualNetworkPeering `
  -ResourceGroupName myResourceGroup `
  -VirtualNetworkName myVirtualNetwork1 `
  | Select PeeringState
```

Les ressources d’un réseau virtuel ne peuvent pas communiquer avec les ressources d’un autre réseau virtuel tant que la valeur de **PeeringState** pour les appairages dans les deux réseaux virtuels soit *Connected*.

## <a name="create-virtual-machines"></a>Créer des machines virtuelles

Créez une machine virtuelle sur chaque réseau virtuel afin de pouvoir établir une communication entre elles dans une étape ultérieure.

### <a name="create-the-first-vm"></a>Créer la première machine virtuelle

Créez une machine virtuelle avec [New-AzVM](/powershell/module/az.compute/new-azvm). L’exemple suivant crée une machine virtuelle nommée *myVm1* dans le réseau virtuel *myVirtualNetwork1*. L’option `-AsJob` crée la machine virtuelle en arrière-plan. Vous pouvez donc passer à l’étape suivante. Lorsque vous y êtes invité, entrez le nom d’utilisateur et le mot de passe pour vous connecter à la machine virtuelle.

```azurepowershell-interactive
New-AzVm `
  -ResourceGroupName "myResourceGroup" `
  -Location "East US" `
  -VirtualNetworkName "myVirtualNetwork1" `
  -SubnetName "Subnet1" `
  -ImageName "Win2016Datacenter" `
  -Name "myVm1" `
  -AsJob
```

### <a name="create-the-second-vm"></a>Créer la seconde machine virtuelle

```azurepowershell-interactive
New-AzVm `
  -ResourceGroupName "myResourceGroup" `
  -Location "East US" `
  -VirtualNetworkName "myVirtualNetwork2" `
  -SubnetName "Subnet1" `
  -ImageName "Win2016Datacenter" `
  -Name "myVm2"
```

La création de la machine virtuelle ne nécessite que quelques minutes. Attendez qu’Azure ait créé la machine virtuelle et renvoyé une sortie à PowerShell pour passer aux étapes suivantes.

## <a name="communicate-between-vms"></a>Établir une communication entre les machines virtuelles

Vous pouvez vous connecter à l’adresse IP publique d’une machine virtuelle depuis Internet. Utilisez [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) pour retourner l’adresse IP publique d’une machine virtuelle. L’exemple suivant retourne l’adresse IP publique de la machine virtuelle *myVm1* :

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVm1 `
  -ResourceGroupName myResourceGroup | Select IpAddress
```

Utilisez la commande suivante pour créer une session Bureau à distance avec la machine virtuelle *myVm1* à partir de votre ordinateur local. Remplacez `<publicIpAddress>` par l’adresse IP retournée par la commande précédente.

```
mstsc /v:<publicIpAddress>
```

Un fichier .rdp (Remote Desktop Protocol) est créé, téléchargé sur votre ordinateur, puis ouvert. Entrez le nom d’utilisateur et le mot de passe (il se peut que vous deviez choisir **Plus de choix**, puis **Utiliser un compte différent** pour spécifier les informations d’identification que vous avez entrées lors de la création de la machine virtuelle), puis cliquez sur **OK**. Un avertissement de certificat peut s’afficher pendant le processus de connexion. Cliquez sur **Oui** ou **Continuer** pour continuer le processus de connexion.

Sur la machine virtuelle *myVm1*, autorisez le protocole ICMP (Internet Control Message Protocol) à travers le pare-feu Windows afin d’effectuer un test ping pour cette machine virtuelle à partir de *myVm2* lors d’une étape ultérieure, en utilisant PowerShell :

```powershell
New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4
```

Bien que ce test ping soit utilisé pour établir une communication entre les machines virtuelles, il n’est pas recommandé d’autoriser le protocole IMCP via le pare-feu Windows lors de déploiements de production.

Pour établir une connexion avec la machine virtuelle *myVm2*, entrez la commande suivante à partir d’une invite de commandes sur la machine virtuelle *myVm1* :

```
mstsc /v:10.1.0.4
```

Étant donné que vous avez activé la commande ping sur *myVm1*, vous pouvez maintenant effectuer un test ping par adresse IP à partir d’une invite de commandes sur la machine virtuelle *myVm2* :

```
ping 10.0.0.4
```

Vous recevez quatre réponses. Déconnectez vos sessions RDP sur *myVm1* et *myVm2*.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’avez plus besoin d’un groupe de ressources, utilisez [Remove-AzResourcegroup](/powershell/module/az.resources/remove-azresourcegroup) pour le supprimer, ainsi que toutes les ressources qu’il contient.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à connecter deux réseaux situés dans la même région Azure à l’aide du peering de réseaux virtuels. Vous pouvez également appairer des réseaux virtuels situés dans des [régions différentes](virtual-network-manage-peering.md#cross-region) et dans des [abonnements Azure différents](create-peering-different-subscriptions.md#powershell). Vous pouvez aussi créer des [conceptions réseau hub-and-spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke#virtual-network-peering) avec le peering. Pour en savoir plus sur le peering de réseaux virtuels, consultez [Aperçu de présentation du peering de réseaux virtuels](virtual-network-peering-overview.md) et [Gérer les peerings de réseau virtuels](virtual-network-manage-peering.md).

Vous pouvez [connecter votre propre ordinateur à un réseau virtuel](../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json) via un VPN et interagir avec les ressources dans un réseau virtuel, ou dans des réseaux virtuels appairés. Consultez les [exemples de script](powershell-samples.md) pour obtenir des scripts réutilisables permettant d’accomplir un grand nombre des tâches présentées dans les articles sur les réseaux virtuels.
