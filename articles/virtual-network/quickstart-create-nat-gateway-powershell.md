---
title: Créer une passerelle NAT - Azure PowerShell
titlesuffix: Azure Virtual Network NAT
description: Ce guide de démarrage rapide montre comment créer une passerelle NAT à l’aide d’Azure PowerShell
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
Customer intent: I want to create a NAT gateway for outbound connectivity for my virtual network.
ms.service: virtual-network
ms.subservice: nat
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/18/2020
ms.author: allensu
ms.openlocfilehash: de4e32d79cf4dfb3a5f54544c65544297a2c0232
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993565"
---
# <a name="tutorial-create-a-nat-gateway-using-azure-powershell"></a>Tutoriel : Créer une passerelle NAT avec Azure PowerShell

Ce tutoriel vous montre comment utiliser le service NAT de réseau virtuel Azure. Vous allez créer une passerelle NAT pour fournir une connectivité sortante à une machine virtuelle dans Azure. 


[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Vous pouvez suivre ce tutoriel en utilisant Azure Cloud Shell, ou exécuter les commandes localement.  Si vous n’avez pas utilisé Azure Cloud Shell, [connectez-vous maintenant](https://shell.azure.com).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup?view=latest). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

L’exemple suivant crée un groupe de ressources nommé **myResourceGroupNAT** à l’emplacement **eastus2** :

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$loc = 'eastus2'
  
New-AzResourceGroup -Name $rsg -Location $loc
```

## <a name="create-the-nat-gateway"></a>Créer la passerelle NAT

Les options d’adresse IP publique pour la passerelle NAT sont les suivantes :

* **Adresses IP publiques**
* **Préfixes d’adresse IP publique**

Les deux peuvent être utilisées avec la passerelle NAT.

Nous allons ajouter une adresse IP publique et un préfixe d’adresse IP publique à ce scénario pour le démontrer.

### <a name="create-a-public-ip-address"></a>Créer une adresse IP publique

Pour accéder à l’Internet, vous avez besoin d’une ou de plusieurs adresses IP publiques pour la passerelle NAT. Utilisez la commande [New-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/new-azpublicipaddress?view=latest) pour créer une ressource d’adresse IP publique nommée **myPublicIP** dans **myResourceGroupNAT**. Le résultat de cette commande est stocké dans la variable **$publicIP** pour une utilisation ultérieure.

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$loc = 'eastus2'
$sku = 'Standard'
$pbnm = 'myPublicIP'
  
$publicIP = 
New-AzPublicIpAddress -Name $pbnm -ResourceGroupName $rsg -AllocationMethod Static -Location $loc -Sku $sku
```

### <a name="create-a-public-ip-prefix"></a>Créer un préfixe d’adresse IP publique

Utilisez la commande [New-AzPublicIpprefix](https://docs.microsoft.com/powershell/module/az.network/new-azpublicipprefix?view=latest) pour créer une ressource de préfixe d’adresse IP publique nommée **myPublicIPprefix** dans **myResourceGroupNAT**.  Le résultat de cette commande est stocké dans une variable nommée **$publicIPPrefix** pour une utilisation ultérieure.

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$loc = 'eastus2'
$pxnm = 'myPublicIPprefix'

$publicIPPrefix = 
New-AzPublicIpPrefix -Name $pxnm -ResourceGroupName $rsg -Location $loc -PrefixLength 31
```

### <a name="create-a-nat-gateway-resource"></a>Créer une ressource de passerelle NAT

Cette section explique en détail comment vous pouvez créer et configurer les composants suivants du service NAT à l’aide de la ressource de passerelle NAT :
  - Un pool d’adresses IP publiques et un préfixe d’adresse IP publique à utiliser pour les flux sortants qui sont traduits par la ressource de passerelle NAT.
  - Passer la valeur par défaut du délai d’inactivité de 4 minutes à 10 minutes.

Créez une passerelle NAT Azure globale avec [New-AzNatGateway](https://docs.microsoft.com/powershell/module/az.network/new-aznatgateway). Le résultat de cette commande crée une ressource de passerelle nommée **myNATgateway** qui utilise l’adresse IP publique **myPublicIP** et le préfixe d’adresse IP publique **myPublicIPprefix**. Le délai d’inactivité est défini sur 10 minutes.  Le résultat de cette commande est stocké dans une variable nommée **$natGateway** pour une utilisation ultérieure.

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$loc = 'eastus2'
$sku = 'Standard'
$gnm = 'myNATgateway'

$natGateway = 
New-AzNatGateway -Name $gnm -ResourceGroupName $rsg -PublicIpAddress $publicIP -PublicIpPrefix $publicIPPrefix -Location $loc -Sku $sku -IdleTimeoutInMinutes 10
  ```

À ce stade, la passerelle NAT est fonctionnelle et il ne reste plus qu’à définir quels sous-réseaux d’un réseau virtuel doivent l’utiliser.

## <a name="configure-virtual-network"></a>Configurer un réseau virtuel

Créez le réseau virtuel et associez le sous-réseau à la passerelle.

Créez un réseau virtuel nommé **myVnet** avec un sous-réseau nommé **mySubnet** à l’aide de [New-AzVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetworksubnetconfig?view=latest) dans **myResourceGroup** à l’aide de [New-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetwork?view=latest). L’espace d’adressage IP pour le réseau virtuel est **192.168.0.0/16**. Le sous-réseau au sein du réseau virtuel est **192.168.0.0/24**.  Le résultat des commandes est stocké dans les variables nommées **$subnet** et **$vnet** en vue d’une utilisation ultérieure.

```azurepowershell-interactive
$sbnm = 'mySubnet'
$vnnm = 'myVnet'
$rsg = 'myResourceGroupNAT'
$loc = 'eastus2'
$pfxsub = '192.168.0.0/24'
$pfxvn = '192.168.0.0/16'

$subnet = 
New-AzVirtualNetworkSubnetConfig -Name $sbnm -AddressPrefix $pfxsub -NatGateway $natGateway

$vnet = 
New-AzVirtualNetwork -Name $vnnm -ResourceGroupName $rsg -Location $loc -AddressPrefix $pfxvn -Subnet $subnet
```

Tout le trafic sortant vers les destinations Internet utilise désormais le service NAT.  Il n’est pas nécessaire de configurer un UDR.

## <a name="create-a-vm-to-use-the-nat-service"></a>Créer une machine virtuelle pour utiliser le service NAT

Nous allons à présent créer une machine virtuelle pour utiliser le service NAT.  Cette machine virtuelle dispose d’une adresse IP publique à utiliser comme adresse IP publique au niveau de l’instance pour vous permettre d’accéder à la machine virtuelle.  Le service NAT reconnaît la direction du flux et remplace la destination Internet par défaut dans votre sous-réseau. L’adresse IP publique de la machine virtuelle n’est pas utilisée pour les connexions sortantes.

### <a name="create-public-ip-for-source-vm"></a>Créer une adresse IP publique pour la machine virtuelle source

Nous créons une adresse IP publique à utiliser pour accéder à la machine virtuelle.  Utilisez la commande [New-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/new-azpublicipaddress?view=latest) pour créer une ressource d’adresse IP publique nommée **myPublicIPVM** dans **myResourceGroupNAT**.  Le résultat de cette commande est stocké dans une variable nommée **$publicIpVM** pour une utilisation ultérieure.

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$loc = 'eastus2'
$ipnm = 'myPublicIPVM'
$sku = 'Standard'

$publicIpVM = 
New-AzPublicIpAddress -Name $ipnm -ResourceGroupName $rsg -AllocationMethod Static -Location $loc -Sku $sku
```

### <a name="create-an-nsg-and-expose-ssh-endpoint-for-vm"></a>Créer un groupe de sécurité réseau et exposer le point de terminaison SSH pour la machine virtuelle

Les adresses IP publiques standard étant « sécurisées par défaut », nous devons créer un groupe de sécurité réseau pour autoriser SSH à l’accès entrant. Utilisez [New-AzNetworkSecurityGroup](https://docs.microsoft.com/powershell/module/az.network/new-aznetworksecuritygroup?view=latest) pour créer une ressource de groupe de sécurité réseau nommée **myNSG**. Utilisez [New-AzNetworkSecurityRuleConfig](https://docs.microsoft.com/powershell/module/az.network/new-aznetworksecurityruleconfig?view=latest) afin de créer pour l’accès SSH une règle de groupe de sécurité réseau nommée **ssh** dans **myResourceGroupNAT**.  Le résultat de cette commande est stocké dans une variable nommée **$nsg** pour une utilisation ultérieure.

```azurepowershell-interactive
$rnm = 'ssh'
$rdesc = 'SSH access'
$acc = 'Allow'
$pro = 'Tcp'
$dir = 'Inbound'
$pri = '100'
$prt = '22'
$rsg = 'myResourceGroupNAT'
$rnm = 'myNSG'
$loc = 'eastus2'

$sshrule = 
New-AzNetworkSecurityRuleConfig -Name $rnm -Description $rdesc -Access $acc -Protocol $pro -Direction $dir -Priority $pri -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange $prt

$nsg = 
New-AzNetworkSecurityGroup -ResourceGroupName $rsg -Name $rnm -Location $loc -SecurityRules $sshrule 
```

### <a name="create-nic-for-vm"></a>Créer une carte réseau pour une machine virtuelle

Créez une interface réseau nommée **myNic** à l’aide de la commande [New-AzNetworkInterface](https://docs.microsoft.com/powershell/module/az.network/new-aznetworkinterface?view=azps-2.8.0). Cette commande associe l’adresse IP publique au groupe de sécurité réseau. Le résultat de cette commande est stocké dans une variable nommée **$nic** pour une utilisation ultérieure.

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$nmn = 'myNic'
$loc = 'eastus2'

$nic = 
New-AzNetworkInterface -ResourceGroupName $rsg -Name $nmn -NetworkSecurityGroupID $nsg.Id -PublicIPAddressID $publicIPVM.Id -SubnetID $vnet.Subnets[0].Id -Location $loc
```

### <a name="create-vm"></a>Créer une machine virtuelle

#### <a name="create-ssh-key-pair"></a>Créer la paire de clés SSH

Vous aurez besoin d’une paire de clés SSH pour suivre ce guide de démarrage rapide. Si vous disposez déjà d’une paire de clés SSH, vous pouvez ignorer cette étape.

Utilisez ssh-keygen pour créer une paire de clés SSH.

```azurepowershell-interactive
ssh-keygen -t rsa -b 2048
```
Pour plus d’informations sur la création de paires de clés SSH, notamment l’utilisation de PuTTy, voir [Guide pratique pour utiliser des clés SSH avec Windows](https://docs.microsoft.com/azure/virtual-machines/linux/ssh-from-windows).

Si vous créez la paire de clés SSH à l’aide de Cloud Shell, celle-ci est stockée dans une image conteneur. Ce [compte de stockage est créé automatiquement](https://docs.microsoft.com/azure/cloud-shell/persisting-shell-storage). Ne supprimez pas le compte de stockage ni son partage de fichiers tant que vous n’avez pas récupéré vos clés.

#### <a name="create-vm-configuration"></a>Créer une configuration de machine virtuelle

Pour créer une machine virtuelle dans PowerShell, vous créez une configuration qui comprend les paramètres de l’image à utiliser, la taille et les options d’authentification. La configuration est utilisée pour générer la machine virtuelle.

Définissez les informations d’identification SSH, les informations du système d’exploitation et la taille de machine virtuelle. Dans cet exemple, la clé SSH est stockée dans ~/.ssh/id_rsa.pub.

```azurepowershell-interactive
#Define a credential object

$securePassword = 
ConvertTo-SecureString ' ' -AsPlainText -Force

$cred = 
New-Object System.Management.Automation.PSCredential ("azureuser", $securePassword)

# Create a virtual machine configuration

$vnm = 'myVM'
$vsz = 'Standard_D1'
$pub = 'Canonical'
$off = 'UbuntuServer'
$sku = '18.04-LTS'
$ver = 'latest'

$vmConfig = 
New-AzVMConfig -VMName $vnm -VMSize $vsz

Set-AzVMOperatingSystem -VM $vmConfig -Linux -ComputerName $vnm -Credential $cred -DisablePasswordAuthentication

Set-AzVMSourceImage -VM $vmConfig -PublisherName $pub -Offer $off -Skus $sku -Version $ver

Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id

# Configure the SSH key

$sshPublicKey = cat ~/.ssh/id_rsa.pub

Add-AzVMSshPublicKey -VM $vmconfig -KeyData $sshPublicKey -Path "/home/azureuser/.ssh/authorized_keys"

```
Combinez les définitions de configuration pour créer une machine virtuelle nommée **myVM** à l’aide de la commande [New-AzVM](/powershell/module/az.compute/new-azvm?view=azps-2.8.0) dans **myResourceGroupNAT**.

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$loc = 'eastus2'

New-AzVM -ResourceGroupName $rsg -Location $loc -VM $vmconfig
```

Attendez que la machine virtuelle soit prête pour le déploiement, puis passez aux étapes restantes.

## <a name="discover-the-ip-address-of-the-vm"></a>Découvrir l’adresse IP de la machine virtuelle

Tout d’abord, nous avons besoin de découvrir l’adresse IP de la machine virtuelle que vous avez créée. Pour obtenir l’adresse IP publique de la machine virtuelle, utilisez [Get-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/get-azpublicipaddress?view=latest). 

```azurepowershell-interactive
$rsg = 'myResourceGroupNAT'
$nmn = 'myPublicIPVM'

Get-AzPublicIpAddress -ResourceGroupName $rsg -Name $nmn | select IpAddress
``` 

>[!IMPORTANT]
>Copiez l’adresse IP publique, puis collez-la dans un bloc-notes afin de pouvoir l’utiliser pour accéder à la machine virtuelle.

### <a name="sign-in-to-vm"></a>Se connecter à la machine virtuelle

Les informations d’identification SSH doivent être stockées dans votre instance Cloud Shell depuis l’opération précédente.  Ouvrez une session [Azure Cloud Shell](https://shell.azure.com) dans votre navigateur. Utilisez l’adresse IP récupérée à l’étape précédente pour établir une connexion SSH à la machine virtuelle.

```bash
ssh azureuser@<ip-address-destination>
```

Vous êtes désormais prêt à utiliser le service NAT.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, vous pouvez utiliser la commande [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup?view=latest) pour supprimer le groupe de ressources et toutes les ressources qu’il contient.

```azurepowershell-interactive 
Remove-AzResourceGroup -Name myResourceGroupNAT
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé une passerelle NAT et une machine virtuelle pour l’utiliser. 

Consultez les métriques dans Azure Monitor pour découvrir le fonctionnement de votre service NAT. Diagnostiquez des problèmes, tels que l’épuisement des ressources des ports SNAT disponibles.  L’épuisement des ressources de ports SNAT est résolu en ajoutant des ressources supplémentaires, d’adresse IP publique ou de préfixe d’adresse IP publique, ou des deux à la fois.


- Apprenez-en davantage sur le service [NAT de Réseau virtuel Azure](./nat-overview.md).
- Apprenez-en davantage sur la [ressource de passerelle NAT](./nat-gateway-resource.md).
- Guide de démarrage rapide du déploiement d’une [ressource de passerelle NAT avec Azure CLI](./quickstart-create-nat-gateway-cli.md).
- Guide de démarrage rapide du déploiement d’une [ressource de passerelle NAT avec Azure PowerShell](./quickstart-create-nat-gateway-powershell.md).
- Guide de démarrage rapide du déploiement d’une [ressource de passerelle NAT avec le portail Azure](./quickstart-create-nat-gateway-portal.md).
> [!div class="nextstepaction"]


