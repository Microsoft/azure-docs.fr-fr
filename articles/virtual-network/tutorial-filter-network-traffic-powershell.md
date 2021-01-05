---
title: Filtrer le trafic réseau - Azure PowerShell | Microsoft Docs
description: Dans cet article, vous allez apprendre à filtrer le trafic réseau vers un sous-réseau, avec un groupe de sécurité réseau, à l’aide de PowerShell.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
Customer intent: I want to filter network traffic to virtual machines that perform similar functions, such as web servers.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/30/2018
ms.author: kumud
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: c0b0d366518b786b33048d2d2f182abd787856ad
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89075101"
---
# <a name="filter-network-traffic-with-a-network-security-group-using-powershell"></a>Filtrer le trafic réseau avec un groupe de sécurité réseau à l’aide de PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vous pouvez filtrer le trafic réseau entrant dans un sous-réseau de réseau virtuel ou qui en sort, avec un groupe de sécurité réseau. Les groupes de sécurité réseau contiennent des règles de sécurité qui filtrent le trafic réseau par adresse IP, port et protocole. Les règles de sécurité sont appliquées aux ressources déployées dans un sous-réseau. Dans cet article, vous apprendrez comment :

* Créer un groupe de sécurité réseau et les règles associées
* Créer un réseau virtuel et associer un groupe de sécurité réseau à un sous-réseau
* Déployer des machines virtuelles dans un sous-réseau
* Tester les filtres de trafic

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser PowerShell en local, vous devez exécuter le module Azure PowerShell version 1.0.0 ou ultérieure pour les besoins de cet article. Exécutez `Get-Module -ListAvailable Az` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.

## <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Un groupe de sécurité réseau contient des règles de sécurité. Les règles de sécurité spécifient une source et une destination. Les sources et les destinations peuvent être des groupes de sécurité d’application.

### <a name="create-application-security-groups"></a>Créer des groupes de sécurité d’application

Tout d’abord, créez un groupe de ressources pour toutes les ressources créées dans cet article avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). L’exemple suivant crée un groupe de ressources dans l’emplacement *eastus* :

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

Créez un groupe de sécurité d’application avec [New-AzApplicationSecurityGroup](/powershell/module/az.network/new-azapplicationsecuritygroup). Un groupe de sécurité d’application vous permet de regrouper des serveurs avec des exigences de filtrage de port similaires. L’exemple suivant crée deux groupes de sécurité d’application.

```azurepowershell-interactive
$webAsg = New-AzApplicationSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Name myAsgWebServers `
  -Location eastus

$mgmtAsg = New-AzApplicationSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Name myAsgMgmtServers `
  -Location eastus
```

### <a name="create-security-rules"></a>Créer des règles de sécurité

Créez une règle de sécurité avec [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig). L’exemple suivant crée une règle qui autorise le trafic entrant à partir d’Internet vers le groupe de sécurité d’application *myWebServers* par les ports 80 et 443 :

```azurepowershell-interactive
$webRule = New-AzNetworkSecurityRuleConfig `
  -Name "Allow-Web-All" `
  -Access Allow `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 100 `
  -SourceAddressPrefix Internet `
  -SourcePortRange * `
  -DestinationApplicationSecurityGroupId $webAsg.id `
  -DestinationPortRange 80,443

The following example creates a rule that allows traffic inbound from the internet to the *myMgmtServers* application security group over port 3389:

$mgmtRule = New-AzNetworkSecurityRuleConfig `
  -Name "Allow-RDP-All" `
  -Access Allow `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 110 `
  -SourceAddressPrefix Internet `
  -SourcePortRange * `
  -DestinationApplicationSecurityGroupId $mgmtAsg.id `
  -DestinationPortRange 3389
```

Dans cet article, RDP (port 3389) est exposé sur Internet pour la machine virtuelle *myAsgMgmtServers*. Pour les environnements de production, au lieu d’exposer le port 3389 sur Internet, il est recommandé de vous connecter aux ressources Azure que vous souhaitez gérer à l’aide d’une connexion réseau [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou [privée](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

### <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Créez un groupe de sécurité réseau avec [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). L’exemple suivant permet de créer un groupe de sécurité réseau nommé *myNsg* :

```powershell-interactive
$nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -Name myNsg `
  -SecurityRules $webRule,$mgmtRule
```

## <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Créez un réseau virtuel avec [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). L’exemple suivant crée un réseau virtuel nommé *myVirtualNetwork* :

```azurepowershell-interactive
$virtualNetwork = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork `
  -AddressPrefix 10.0.0.0/16
```

Créez une configuration de sous-réseau avec [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig), puis écrivez la configuration du sous-réseau dans le réseau virtuel avec [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork). L’exemple suivant ajoute un sous-réseau nommé *mySubnet* au réseau virtuel et l’associe au groupe de sécurité réseau *myNsg* :

```azurepowershell-interactive
Add-AzVirtualNetworkSubnetConfig `
  -Name mySubnet `
  -VirtualNetwork $virtualNetwork `
  -AddressPrefix "10.0.2.0/24" `
  -NetworkSecurityGroup $nsg
$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="create-virtual-machines"></a>Créer des machines virtuelles

Avant de créer les machines virtuelles, récupérez l’objet de réseau virtuel avec le sous-réseau à l’aide de la commande [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) :

```powershell-interactive
$virtualNetwork = Get-AzVirtualNetwork `
 -Name myVirtualNetwork `
 -Resourcegroupname myResourceGroup
```

Créez une adresse IP publique pour chaque machine virtuelle avec [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) :

```powershell-interactive
$publicIpWeb = New-AzPublicIpAddress `
  -AllocationMethod Dynamic `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -Name myVmWeb

$publicIpMgmt = New-AzPublicIpAddress `
  -AllocationMethod Dynamic `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -Name myVmMgmt
```

Créez deux interfaces réseau avec [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface)et attribuez une adresse IP publique à l’interface réseau. L’exemple suivant crée une interface réseau, lui associe l’adresse IP publique *myVmWeb* et la rend membre du groupe de sécurité d’application *myAsgWebServers* :

```powershell-interactive
$webNic = New-AzNetworkInterface `
  -Location eastus `
  -Name myVmWeb `
  -ResourceGroupName myResourceGroup `
  -SubnetId $virtualNetwork.Subnets[0].Id `
  -ApplicationSecurityGroupId $webAsg.Id `
  -PublicIpAddressId $publicIpWeb.Id
```

L’exemple suivant crée une interface réseau, lui associe l’adresse IP publique *myVmMgmt* et la rend membre du groupe de sécurité d’application *myAsgMgmtServers* :

```powershell-interactive
$mgmtNic = New-AzNetworkInterface `
  -Location eastus `
  -Name myVmMgmt `
  -ResourceGroupName myResourceGroup `
  -SubnetId $virtualNetwork.Subnets[0].Id `
  -ApplicationSecurityGroupId $mgmtAsg.Id `
  -PublicIpAddressId $publicIpMgmt.Id
```

Créez deux machines virtuelles dans le réseau virtuel pour pouvoir valider le filtrage du trafic à une étape ultérieure.

Créez une configuration de machine virtuelle avec [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig), puis créez la machine virtuelle avec [New-AzVM](/powershell/module/az.compute/new-azvm). L’exemple suivant crée une machine virtuelle qui servira de serveur web. L’option `-AsJob` crée la machine virtuelle en arrière-plan. Vous pouvez donc passer à l’étape suivante :

```azurepowershell-interactive
# Create user object
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."

$webVmConfig = New-AzVMConfig `
  -VMName myVmWeb `
  -VMSize Standard_DS1_V2 | `
Set-AzVMOperatingSystem -Windows `
  -ComputerName myVmWeb `
  -Credential $cred | `
Set-AzVMSourceImage `
  -PublisherName MicrosoftWindowsServer `
  -Offer WindowsServer `
  -Skus 2016-Datacenter `
  -Version latest | `
Add-AzVMNetworkInterface `
  -Id $webNic.Id
New-AzVM `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -VM $webVmConfig `
  -AsJob
```

Créez une machine virtuelle comme un serveur d’administration :

```azurepowershell-interactive
# Create user object
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."

# Create the web server virtual machine configuration and virtual machine.
$mgmtVmConfig = New-AzVMConfig `
  -VMName myVmMgmt `
  -VMSize Standard_DS1_V2 | `
Set-AzVMOperatingSystem -Windows `
  -ComputerName myVmMgmt `
  -Credential $cred | `
Set-AzVMSourceImage `
  -PublisherName MicrosoftWindowsServer `
  -Offer WindowsServer `
  -Skus 2016-Datacenter `
  -Version latest | `
Add-AzVMNetworkInterface `
  -Id $mgmtNic.Id
New-AzVM `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -VM $mgmtVmConfig
```

La création de la machine virtuelle prend quelques minutes. Ne passez pas à l’étape suivante avant qu’Azure ait terminé la création de la machine virtuelle.

## <a name="test-traffic-filters"></a>Tester les filtres de trafic

Utilisez [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) pour retourner l’adresse IP publique d’une machine virtuelle. L’exemple suivant retourne l’adresse IP publique de la machine virtuelle *myVmMgmt* :

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVmMgmt `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

Utilisez la commande suivante pour créer une session Bureau à distance avec la machine virtuelle *myVmMgmt* à partir de votre ordinateur local. Remplacez `<publicIpAddress>` par l’adresse IP retournée par la commande précédente.

```
mstsc /v:<publicIpAddress>
```

Ouvrez le fichier .rdp téléchargé. Si vous y êtes invité, sélectionnez **Connexion**.

Entrez le nom d’utilisateur et le mot de passe spécifiés lors de la création de la machine virtuelle (il se peut que vous deviez choisir **Plus de choix**, puis **Utiliser un compte différent** pour spécifier les informations d’identification que vous avez entrées lors de la création de la machine virtuelle), puis sélectionnez **OK**. Un avertissement de certificat peut s’afficher pendant le processus de connexion. Sélectionnez**Oui** pour poursuivre le processus de connexion.

La connexion réussit, car le port 3389 autorise le trafic entrant depuis Internet vers le groupe de sécurité d’application *myAsgMgmtServers* dans lequel se situe l’interface réseau attachée à la machine virtuelle *myVmMgmt*.

Utilisez la commande suivante pour créer une connexion Bureau à distance vers la machine virtuelle *myVmWeb* à partir de la machine virtuelle *myVmMgmt*, avec la commande suivante, à partir de PowerShell :

``` 
mstsc /v:myvmWeb
```

La connexion réussit car une règle de sécurité par défaut au sein de chaque groupe de sécurité réseau autorise le trafic par tous les ports entre toutes les adresses IP au sein d’un réseau virtuel. Vous ne pouvez pas créer une connexion Bureau à distance à la machine virtuelle *myVmWeb* depuis Internet, car la règle de sécurité pour le *myAsgWebServers* n’autorise pas les données entrantes venant d’Internet sur le port 3389.

Utilisez la commande suivante pour installer Microsoft IIS sur la machine virtuelle *myVmWeb* à partir de PowerShell :

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

Une fois l’installation d’IIS terminée, déconnectez-vous de la machine virtuelle *myVmWeb*, ce qui vous laisse dans la connexion Bureau à distance de la machine virtuelle *myVmMgmt*. Pour afficher l’écran d’accueil d’IIS, ouvrez un navigateur web et accédez à http:\//myVmWeb.

Déconnectez-vous de la machine virtuelle *myVmMgmt*.

Sur votre ordinateur, entrez la commande suivante à partir de PowerShell pour récupérer l’adresse IP publique du serveur *myVmWeb* :

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVmWeb `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

Pour confirmer que vous pouvez accéder au serveur web *myVmWeb* sans être dans Azure, ouvrez un navigateur web sur votre ordinateur et accédez à `http://<public-ip-address-from-previous-step>`. La connexion réussit, car le port 80 autorise le trafic entrant depuis Internet vers le groupe de sécurité d’application *myAsgWebServers* dans lequel se situe l’interface réseau attachée à la machine virtuelle *myVmWeb*.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’un groupe de ressources, vous pouvez utiliser [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour le supprimer ainsi que toutes les ressources qu’il contient :

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez créé un groupe de sécurité réseau et vous l’avez associé à un sous-réseau d’un réseau virtuel. Pour en savoir plus sur les groupes de sécurité réseau, consultez [Vue d’ensemble d’un groupe de sécurité réseau](security-overview.md) et [Gérer un groupe de sécurité réseau](manage-network-security-group.md).

Azure achemine par défaut le trafic entre les sous-réseaux. À la place, vous pouvez choisir par exemple d’acheminer le trafic entre les sous-réseaux via une machine virtuelle, agissant comme un pare-feu. Pour savoir comment procéder, consultez [Créer une table de routage](tutorial-create-route-table-powershell.md).
