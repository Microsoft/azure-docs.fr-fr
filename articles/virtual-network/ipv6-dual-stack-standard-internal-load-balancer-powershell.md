---
title: Déployer une application double pile IPv6 avec un Standard Load Balancer interne dans Azure - PowerShell
titlesuffix: Azure Virtual Network
description: Cet article montre comment déployer une application double pile IPv6 avec un équilibreur de charge Standard Load Balancer interne sur un réseau virtuel Azure à l’aide d’Azure PowerShell.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/14/2019
ms.author: kumud
ms.openlocfilehash: 759bd94aee98aa04dee56acf0e50ca90cd0541b8
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "96000604"
---
# <a name="deploy-an-ipv6-dual-stack-application-using-standard-internal-load-balancer-in-azure---powershell-preview"></a>Déployer une application double pile IPv6 avec un Standard Load Balancer interne dans Azure - PowerShell (préversion)

Cet article montre comment déployer dans Azure une application double pile (IPv4 + IPv6) incluant un sous-réseau et un réseau virtuel double pile, un équilibreur de charge Standard Load Balancer interne avec des configurations front-end doubles (IPv4 + IPv6), des machines virtuelles dont les cartes réseau présentent une configuration double IP, des groupes de sécurité réseau et des adresses IP publiques.

La procédure de création d’un équilibreur de charge interne compatible IPv6 est quasiment identique à la procédure de création d’un Load Balancer IPv6 exposé à Internet décrite [ici](virtual-network-ipv4-ipv6-dual-stack-standard-load-balancer-powershell.md). Les seules différences pour créer un équilibreur de charge interne concernent la configuration front-end, comme illustré dans l’exemple PowerShell ci-dessous :

```azurepowershell
 $frontendIPv6 = New-AzLoadBalancerFrontendIpConfig `
 -Name "dsLbFrontEnd_v6" `
 -PrivateIpAddress "fd00:db8:deca:deed::100" `
 -PrivateIpAddressVersion "IPv6" `
 -Subnet $DsSubnet
```

Les modifications qui font de la configuration ci-dessus une configuration front-end d’équilibreur de charge interne sont les suivantes :
- « IPv6 » est spécifié pour `PrivateIpAddressVersion`.
- L’argument `-PublicIpAddress` a été omis ou remplacé par `-PrivateIpAddress`. Notez que l’adresse privée doit être comprise dans la plage de l’espace d’adressage IP de sous-réseau dans lequel l’équilibreur de charge interne sera déployé. Si une `-PrivateIpAddress` statique est omise, l’adresse IPv6 libre suivante est sélectionnée à partir du sous-réseau sur lequel l’équilibreur de charge interne est déployé.
- Le sous-réseau double pile sur lequel l’équilibreur de charge interne sera déployé est spécifié avec un argument `-Subnet` ou `-SubnetId`.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser PowerShell en local, vous devez exécuter le module Azure PowerShell version 6.9.0 ou ultérieure pour les besoins de cet article. Exécutez `Get-Module -ListAvailable Az` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.

## <a name="prerequisites"></a>Prérequis
Avant de déployer une application double pile dans Azure, vous devez configurer votre abonnement pour cette fonctionnalité d’évaluation à l’aide de la commande Azure PowerShell suivante :

Effectuez l’inscription comme suit :
```azurepowershell
Register-AzProviderFeature -FeatureName AllowIPv6VirtualNetwork -ProviderNamespace Microsoft.Network
Register-AzProviderFeature -FeatureName AllowIPv6CAOnStandardLB -ProviderNamespace Microsoft.Network
```
Il faut compter 30 minutes pour l’inscription de la fonctionnalité. Vous pouvez vérifier le statut de l’inscription en exécutant la commande Azure PowerShell suivante : Vérifiez l’inscription en procédant comme suit :
```azurepowershell
Get-AzProviderFeature -FeatureName AllowIPv6VirtualNetwork -ProviderNamespace Microsoft.Network
Get-AzProviderFeature -FeatureName AllowIPv6CAOnStandardLB -ProviderNamespace Microsoft.Network
```
À l’issue de l’installation, exécutez la commande suivante :

```azurepowershell
Register-AzResourceProvider -ProviderNamespace Microsoft.Network
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Avant de pouvoir créer un réseau virtuel double pile, vous devez créer un groupe de ressources avec la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). L’exemple suivant crée un groupe de ressources nommé *dsStd_ILB_RG* à l’emplacement *east us* :

```azurepowershell
$rg = New-AzResourceGroup `
  -ResourceGroupName "dsStd_ILB_RG"  `
  -Location "east us"
```

## <a name="create-ipv4-and-ipv6-public-ip-addresses"></a>Créer des adresses IP IPv4 et IPv6 publiques
Pour accéder à vos machines virtuelles à partir d’Internet, vous avez besoin d’adresses IP publiques IPv4 et IPv6 pour les machines virtuelles. Créez des adresses IP publiques avec la commande [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress). L’exemple suivant crée des adresses IP IPv4 et IPv6 publiques nommées *RdpPublicIP_1* and *RdpPublicIP_2* dans le groupe de ressources *dsStd_ILB_RG* :

```azurepowershell
$RdpPublicIP_1 = New-AzPublicIpAddress `
  -Name "RdpPublicIP_1" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -AllocationMethod Static `
  -IpAddressVersion IPv4  `
  -sku Standard
  
$RdpPublicIP_2 = New-AzPublicIpAddress `
  -Name "RdpPublicIP_2" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -AllocationMethod Static `
  -IpAddressVersion IPv4  `
  -sku Standard
```
## <a name="create-the-virtual-network-and-the-subnet"></a>Créer le réseau virtuel et le sous-réseau

Créez un réseau virtuel à l’aide d’[New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) avec une configuration de sous-réseau double pile à l’aide de [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig). L’exemple suivant crée un réseau virtuel nommé *dsVnet* avec *dsSubnet*.

```azurepowershell
# Create dual stack subnet config
$DsSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name "dsSubnet" `
  -AddressPrefix "10.0.0.0/24","fd00:db8:deca:deed::/64"

# Create the virtual network
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -Name "dsVnet" `
  -AddressPrefix "10.0.0.0/16","fd00:db8:deca::/48"  `
  -Subnet $DsSubnet

#Refresh the fully populated subnet for use in load balancer frontend configuration
$DsSubnet = get-AzVirtualNetworkSubnetconfig -name dsSubnet -VirtualNetwork $vnet
```
## <a name="create-standard-load-balancer"></a>Créer un Standard Load Balancer

Dans cette section, vous allez établir la configuration IP front-end double (IPv4 et IPv6) et configurer le pool d’adresses back-end pour l’équilibreur de charge, puis créer un équilibreur de charge Standard Load Balancer.

### <a name="create-front-end-ip"></a>Créer une configuration IP front-end

Créez une configuration IP front-end avec [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig). L’exemple suivant crée les configurations IP front-end IPv4 et IPv6 nommées *dsLbFrontEnd_v4* et *dsLbFrontEnd_v6* :

```azurepowershell
$frontendIPv4 = New-AzLoadBalancerFrontendIpConfig `
  -Name "dsLbFrontEnd_v4" `
  -PrivateIpAddress "10.0.0.100"  `
  -PrivateIpAddressVersion "IPv4"   `
  -Subnet $DsSubnet

$frontendIPv6 = New-AzLoadBalancerFrontendIpConfig `
  -Name "dsLbFrontEnd_v6" `
  -PrivateIpAddress "fd00:db8:deca:deed::100"  `
  -PrivateIpAddressVersion "IPv6"   `
  -Subnet $DsSubnet

```

### <a name="configure-back-end-address-pool"></a>Configurer un pool d’adresses back-end

Créez un pool d’adresses back-end avec [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig). Les machines virtuelles se joignent à ce pool back-end dans les étapes suivantes. L’exemple suivant crée les pools d’adresses back-end nommés *dsLbBackEndPool_v4* et *dsLbBackEndPool_v6* pour inclure des machines virtuelles présentant à la fois des configurations de carte réseau IPv4 et IPv6 :

```azurepowershell
$backendPoolv4 = New-AzLoadBalancerBackendAddressPoolConfig -Name "dsLbBackEndPool_v4"

$backendPoolv6 = New-AzLoadBalancerBackendAddressPoolConfig -Name "dsLbBackEndPool_v6"
```

### <a name="create-a-load-balancer-rule"></a>Créer une règle d’équilibreur de charge

Une règle d’équilibrage de charge est utilisée pour définir la distribution du trafic vers les machines virtuelles. Vous définissez la configuration IP frontale pour le trafic entrant et le pool d’adresses IP principal pour recevoir le trafic, ainsi que le port source et le port de destination requis. Pour veiller à ce que seules les machines virtuelles saines reçoivent le trafic, vous devez également définir une sonde d’intégrité, le cas échéant. L’équilibreur de charge de base utilise une sonde IPv4 pour évaluer l’intégrité des points de terminaison IPv4 et IPv6 sur les machines virtuelles. L’équilibreur de charge standard prend en charge de manière explicite les sondes d’intégrité IPv6.

Créez une règle d’équilibreur de charge avec [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig). L’exemple suivant crée des règles d’équilibreur de charge nommées *dsLBrule_v4* et *dsLBrule_v6* et équilibre le trafic sur le port *TCP* *80* vers les configurations IP front-end IPv4 et IPv6 :

```azurepowershell
$lbrule_v4 = New-AzLoadBalancerRuleConfig `
  -Name "dsLBrule_v4" `
  -FrontendIpConfiguration $frontendIPv4 `
  -BackendAddressPool $backendPoolv4 `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80

$lbrule_v6 = New-AzLoadBalancerRuleConfig `
  -Name "dsLBrule_v6" `
  -FrontendIpConfiguration $frontendIPv6 `
  -BackendAddressPool $backendPoolv6 `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80
```

### <a name="create-load-balancer"></a>Créer un équilibreur de charge

Créez un équilibreur de charge Standard Load Balancer avec [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer). L’exemple suivant crée un équilibreur de charge Standard Load Balancer public nommé *myInternalLoadBalancer* à l’aide des configurations IP front-end IPv4 et IPv6, des pools de back-ends et des règles d’équilibrage de charge que vous avez créés lors des étapes précédentes :

```azurepowershell
$lb = New-AzLoadBalancer  `
  -ResourceGroupName $rg.ResourceGroupName  `
  -Location $rg.Location  `
  -Name  "MyInternalLoadBalancer"  `
  -Sku "Standard"  `
  -FrontendIpConfiguration  $frontendIPv4,$frontendIPv6  `
  -BackendAddressPool  $backendPoolv4,$backendPoolv6  `
  -LoadBalancingRule  $lbrule_v4,$lbrule_v6

```

## <a name="create-network-resources"></a>Créer des ressources réseau
Avant de déployer des machines virtuelles et de tester l’équilibreur de charge, vous devez créer des ressources réseau de prise en charge (groupe à haute disponibilité, groupe de sécurité réseau et cartes réseau virtuelles). 

### <a name="create-an-availability-set"></a>Créer un groupe à haute disponibilité
Pour améliorer la haute disponibilité de votre application, placez vos machines virtuelles dans un groupe à haute disponibilité.

Créez un groupe à haute disponibilité avec la commande [New-AzAvailabilitySet](/powershell/module/az.compute/new-azavailabilityset). L’exemple suivant permet de créer un groupe à haute disponibilité nommé *dsAVset* :

```azurepowershell
$avset = New-AzAvailabilitySet `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -Name "dsAVset" `
  -PlatformFaultDomainCount 2 `
  -PlatformUpdateDomainCount 2 `
  -Sku aligned
```

### <a name="create-network-security-group"></a>Création d’un groupe de sécurité réseau

Créez un groupe de sécurité réseau pour les règles qui régiront les communications entrantes et sortantes de votre VNet (réseau virtuel).

#### <a name="create-a-network-security-group-rule-for-port-3389"></a>Créer une règle de groupe de sécurité réseau pour le port 3389

Créez une règle de groupe de sécurité réseau pour autoriser les connexions RDP via le port 3389 avec [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig).

```azurepowershell
$rule1 = New-AzNetworkSecurityRuleConfig `
  -Name 'myNetworkSecurityGroupRuleRDP' `
  -Description 'Allow RDP' `
  -Access Allow `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 100 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 3389
```
#### <a name="create-a-network-security-group-rule-for-port-80"></a>Créer une règle de groupe de sécurité réseau pour le port 80

Créez une règle de groupe de sécurité réseau pour autoriser les connexions Internet entrantes via le port 80 avec [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig).

```azurepowershell
$rule2 = New-AzNetworkSecurityRuleConfig `
  -Name 'myNetworkSecurityGroupRuleHTTP' `
  -Description 'Allow HTTP' `
  -Access Allow `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 200 `
  -SourceAddressPrefix * `
  -SourcePortRange 80 `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80
```
#### <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Créez un groupe de sécurité réseau avec [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup).

```azurepowershell
$nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -Name "dsNSG1"  `
  -SecurityRules $rule1,$rule2
```
### <a name="create-nics"></a>Créer des cartes réseau

Créez des cartes réseau virtuelles avec [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface). L’exemple suivant crée deux cartes réseau virtuelles avec des configurations à la fois IPv4 et IPv6. (Une carte d’interface réseau virtuelle pour chaque machine virtuelle que vous créez pour votre application dans les étapes suivantes).

```azurepowershell

# Create the IPv4 configuration for NIC 1
$Ip4Config=New-AzNetworkInterfaceIpConfig `
  -Name dsIp4Config `
  -Subnet $vnet.subnets[0] `
  -PrivateIpAddressVersion IPv4 `
  -LoadBalancerBackendAddressPool $backendPoolv4 `
  -PublicIpAddress  $RdpPublicIP_1

# Create the IPv6 configuration
$Ip6Config=New-AzNetworkInterfaceIpConfig `
  -Name dsIp6Config `
  -Subnet $vnet.subnets[0] `
  -PrivateIpAddressVersion IPv6 `
  -LoadBalancerBackendAddressPool $backendPoolv6

# Create NIC 1
$NIC_1 = New-AzNetworkInterface `
  -Name "dsNIC1" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -NetworkSecurityGroupId $nsg.Id `
  -IpConfiguration $Ip4Config,$Ip6Config

# Create the IPv4 configuration for NIC 2
$Ip4Config=New-AzNetworkInterfaceIpConfig `
  -Name dsIp4Config `
  -Subnet $vnet.subnets[0] `
  -PrivateIpAddressVersion IPv4 `
  -LoadBalancerBackendAddressPool $backendPoolv4 `
  -PublicIpAddress  $RdpPublicIP_2

# Create NIC 2 reusing the IPv6 configuration from NIC 1
$NIC_2 = New-AzNetworkInterface `
  -Name "dsNIC2" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -NetworkSecurityGroupId $nsg.Id `
  -IpConfiguration $Ip4Config,$Ip6Config

```

### <a name="create-virtual-machines"></a>Créer des machines virtuelles

Définissez un nom d’utilisateur administrateur et un mot de passe pour les machines virtuelles avec [Get-Credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) :

```azurepowershell
$cred = get-credential -Message "DUAL STACK VNET SAMPLE:  Please enter the Administrator credential to log into the VM's"
```

Vous pouvez maintenant créer les machines virtuelles avec [New-AzVM](/powershell/module/az.compute/new-azvm). L’exemple suivant crée deux machines virtuelles ainsi que les composants de réseau virtuel nécessaires s’ils n’existent pas encore.

```azurepowershell
$vmsize = "Standard_A2"
$ImagePublisher = "MicrosoftWindowsServer"
$imageOffer = "WindowsServer"
$imageSKU = "2019-Datacenter"

$vmName= "dsVM1"
$VMconfig1 = New-AzVMConfig -VMName $vmName -VMSize $vmsize -AvailabilitySetId $avset.Id 3> $null | Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent 3> $null | Set-AzVMSourceImage -PublisherName $ImagePublisher -Offer $imageOffer -Skus $imageSKU -Version "latest" 3> $null | Set-AzVMOSDisk -Name "$vmName.vhd" -CreateOption fromImage  3> $null | Add-AzVMNetworkInterface -Id $NIC_1.Id  3> $null
$VM1 = New-AzVM -ResourceGroupName $rg.ResourceGroupName  -Location $rg.Location  -VM $VMconfig1


$vmName= "dsVM2"
$VMconfig2 = New-AzVMConfig -VMName $vmName -VMSize $vmsize -AvailabilitySetId $avset.Id 3> $null | Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent 3> $null | Set-AzVMSourceImage -PublisherName $ImagePublisher -Offer $imageOffer -Skus $imageSKU -Version "latest" 3> $null | Set-AzVMOSDisk -Name "$vmName.vhd" -CreateOption fromImage  3> $null | Add-AzVMNetworkInterface -Id $NIC_2.Id  3> $null
$VM2 = New-AzVM -ResourceGroupName $rg.ResourceGroupName  -Location $rg.Location  -VM $VMconfig2
```
## <a name="view-ipv6-dual-stack-virtual-network-in-azure-portal"></a>Afficher le réseau virtuel double pile IPv6 dans le Portail Microsoft Azure
Vous pouvez afficher le réseau virtuel double pile IPv6 dans le Portail Microsoft Azure en procédant comme suit :
1. Dans la barre de recherche du portail, saisissez *dsVnet*.
2. Quand le nom **dsVnet** apparaît dans les résultats de la recherche, sélectionnez-le. Cette opération affiche la page **Vue d’ensemble** du réseau virtuel double pile nommé *dsVnet*. Le réseau virtuel de pile double présente les deux cartes réseau, avec des configurations IPv4 et IPv6 situées dans le sous-réseau double pile nommé *dsSubnet*.

![Réseau virtuel double pile IPv6 avec Standard Load Balancer interne](./media/ipv6-dual-stack-standard-internal-load-balancer-powershell/ipv6-dual-stack-virtual-network.png)


> [!NOTE]
> L’adresse IPv6 du réseau virtuel Azure est disponible dans le Portail Microsoft Azure en lecture seule pour cette préversion.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, vous pouvez utiliser la commande [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées.

```azurepowershell
Remove-AzResourceGroup -Name dsStd_ILB_RG
```

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a permis de créer un équilibreur de charge Standard Load Balancer avec une configuration IP front-end double (IPv4 et IPv6). Vous avez également créé deux machines virtuelles incluant des cartes réseau avec des configurations IP doubles (IPV4 + IPv6), qui ont été ajoutées au pool back-end de l’équilibreur de charge. Pour en savoir plus sur la prise en charge du protocole IPv6 dans les réseaux virtuels Azure, voir [What is IPv6 for Azure Virtual Network?](ipv6-overview.md) (Rôle d’iPv6 pour un réseau virtuel Azure).
