---
title: Créer une machine virtuelle Windows dans une zone à l’aide d’Azure PowerShell
description: Créer une machine virtuelle Windows dans une zone de disponibilité avec Azure PowerShell
author: cynthn
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.workload: infrastructure
ms.date: 03/27/2018
ms.author: cynthn
ms.custom: ''
ms.openlocfilehash: 95d97605f00a3a80c097eb16695c9d296e17c33d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87825240"
---
# <a name="create-a-windows-virtual-machine-in-an-availability-zone-with-powershell"></a>Créer une machine virtuelle Windows dans une zone de disponibilité avec PowerShell

Cet article explique en détail comment utiliser Azure PowerShell pour créer une machine virtuelle Azure exécutant Windows Server 2016 dans une zone de disponibilité Azure. Une [zone de disponibilité](../../availability-zones/az-overview.md) est une zone physiquement séparée dans une région Azure. Utilisez les zones de disponibilité pour protéger vos applications et vos données dans l’éventualité peu probable d’une défaillance ou d’une perte d’un centre de données entier.

Pour utiliser une zone de disponibilité, créez votre machine virtuelle dans une [région Azure prise en charge](../../availability-zones/az-region.md).

 

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous à votre abonnement Azure avec la commande `Connect-AzAccount` et suivez les instructions à l’écran.

```powershell
Connect-AzAccount
```

## <a name="check-vm-sku-availability"></a>Vérifier la disponibilité de la référence SKU de machine virtuelle
La disponibilité des tailles de machine virtuelle, ou des références SKU, peut varier selon la région et le fuseau horaire. Pour vous aider à planifier l’utilisation des Zones de disponibilité, vous pouvez répertorier les références SKU de machine virtuelle disponibles par zone et par région Azure. Cela permet de s’assure que vous choisissez une taille de machine virtuelle appropriée et obtenez la résilience de votre choix entre les zones. Pour plus d’informations sur les différents types de machine virtuelle et les tailles, consultez [Vue d’ensemble des tailles de machine virtuelle](../sizes.md).

Vous pouvez voir les références SKU de machines virtuelles disponibles avec la commande [Get-AzComputeResourceSku](/powershell/module/az.compute/get-azcomputeresourcesku). L’exemple suivant répertorie les références SKU de machine virtuelle disponibles dans la région *eastus2* :

```powershell
Get-AzComputeResourceSku | where {$_.Locations.Contains("eastus2")};
```

La sortie est similaire à l’exemple condensé suivant, qui présente les Zones de disponibilité dans lesquelles chaque taille de machine virtuelle est disponible :

```powershell
ResourceType                Name  Location      Zones   [...]
------------                ----  --------      -----
virtualMachines  Standard_DS1_v2   eastus2  {1, 2, 3}
virtualMachines  Standard_DS2_v2   eastus2  {1, 2, 3}
[...]
virtualMachines     Standard_F1s   eastus2  {1, 2, 3}
virtualMachines     Standard_F2s   eastus2  {1, 2, 3}
[...]
virtualMachines  Standard_D2s_v3   eastus2  {1, 2, 3}
virtualMachines  Standard_D4s_v3   eastus2  {1, 2, 3}
[...]
virtualMachines   Standard_E2_v3   eastus2  {1, 2, 3}
virtualMachines   Standard_E4_v3   eastus2  {1, 2, 3}
```


## <a name="create-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources Azure avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Dans cet exemple, un groupe de ressources appelé *myResourceGroup* est créé dans la région *eastus2*. 

```powershell
New-AzResourceGroup -Name myResourceGroup -Location EastUS2
```

## <a name="create-networking-resources"></a>Création de ressources de mise en réseau

### <a name="create-a-virtual-network-subnet-and-a-public-ip-address"></a>Créer un réseau virtuel, un sous-réseau et une adresse IP publique 
Ces ressources sont utilisées pour fournir une connectivité réseau à la machine virtuelle et la connecter à Internet. Créez l’adresse IP dans une zone de disponibilité, *2* dans cet exemple. Dans une étape ultérieure, vous créez la machine virtuelle dans la même zone que celle utilisée pour créer l’adresse IP.

```powershell
# Create a subnet configuration
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24

# Create a virtual network
$vnet = New-AzVirtualNetwork -ResourceGroupName myResourceGroup -Location eastus2 `
    -Name myVNet -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig

# Create a public IP address in an availability zone and specify a DNS name
$pip = New-AzPublicIpAddress -ResourceGroupName myResourceGroup -Location eastus2 -Zone 2 `
    -AllocationMethod Static -IdleTimeoutInMinutes 4 -Name "mypublicdns$(Get-Random)" -Sku Standard
```

### <a name="create-a-network-security-group-and-a-network-security-group-rule"></a>Créez un groupe de sécurité réseau et une règle de groupe de sécurité réseau 
Le groupe de sécurité réseau permet sécurise la machine virtuelle avec des règles entrantes et sortantes. Dans ce cas, une règle entrante est créée pour le port 3389, qui autorise les connexions Bureau à distance entrantes. Nous souhaitons également créer une règle entrante pour le port 80, qui autorise le trafic web entrant.

```powershell
# Create an inbound network security group rule for port 3389
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleRDP  -Protocol Tcp `
    -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
    -DestinationPortRange 3389 -Access Allow

# Create an inbound network security group rule for port 80
$nsgRuleWeb = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleWWW  -Protocol Tcp `
    -Direction Inbound -Priority 1001 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
    -DestinationPortRange 80 -Access Allow

# Create a network security group
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName myResourceGroup -Location eastus2 `
    -Name myNetworkSecurityGroup -SecurityRules $nsgRuleRDP,$nsgRuleWeb
```

### <a name="create-a-network-card-for-the-virtual-machine"></a>Créer une carte réseau pour la machine virtuelle 
Créez une carte réseau avec [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) pour la machine virtuelle. La carte réseau connecte la machine virtuelle à un sous-réseau, un groupe de sécurité réseau et une adresse IP publique.

```powershell
# Create a virtual network card and associate with public IP address and NSG
$nic = New-AzNetworkInterface -Name myNic -ResourceGroupName myResourceGroup -Location eastus2 `
    -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id
```

## <a name="create-virtual-machine"></a>Créer une machine virtuelle

Créez une configuration de machine virtuelle. Cette configuration inclut les paramètres qui sont utilisés lors du déploiement de la machine virtuelle, comme une image de machine virtuelle, la taille et la configuration de l’authentification. La taille *Standard_DS1_v2* de cet exemple est prise en charge dans les zones de disponibilité. Cette configuration spécifie également la zone de disponibilité que vous définissez lors de la création de l’adresse IP. Lors de l’exécution de cette étape, vous êtes invité à saisir vos informations d’identification. Les valeurs que vous saisissez sont configurées comme le nom d’utilisateur et le mot de passe pour la machine virtuelle.

```powershell
# Define a credential object
$cred = Get-Credential

# Create a virtual machine configuration
$vmConfig = New-AzVMConfig -VMName myVM -VMSize Standard_DS1_v2 -Zone 2 | `
    Set-AzVMOperatingSystem -Windows -ComputerName myVM -Credential $cred | `
    Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer `
    -Skus 2016-Datacenter -Version latest | Add-AzVMNetworkInterface -Id $nic.Id
```

Créez la machine virtuelle avec [New-AzVM](/powershell/module/az.compute/new-azvm).

```powershell
New-AzVM -ResourceGroupName myResourceGroup -Location eastus2 -VM $vmConfig
```

## <a name="confirm-zone-for-managed-disk"></a>Confirmer la zone de disque managé

Vous avez créé la ressource d’adresse IP de la machine virtuelle dans la même zone de disponibilité que celle de la machine virtuelle. La ressource de disque managé de la machine virtuelle est créée dans la même zone de disponibilité. Vous pouvez le vérifier avec [Get-AzDisk](/powershell/module/az.compute/get-azdisk) :

```powershell
Get-AzDisk -ResourceGroupName myResourceGroup
```

Le résultat montre que le disque géré se trouve dans la même zone de disponibilité que la machine virtuelle :

```powershell
ResourceGroupName  : myResourceGroup
AccountType        : PremiumLRS
OwnerId            : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.
                     Compute/virtualMachines/myVM
ManagedBy          : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx//resourceGroups/myResourceGroup/providers/Microsoft.
                     Compute/virtualMachines/myVM
Sku                : Microsoft.Azure.Management.Compute.Models.DiskSku
Zones              : {2}
TimeCreated        : 9/7/2017 6:57:26 PM
OsType             : Windows
CreationData       : Microsoft.Azure.Management.Compute.Models.CreationData
DiskSizeGB         : 127
EncryptionSettings :
ProvisioningState  : Succeeded
Id                 : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.
                     Compute/disks/myVM_OsDisk_1_bd921920bb0a4650becfc2d830000000
Name               : myVM_OsDisk_1_bd921920bb0a4650becfc2d830000000
Type               : Microsoft.Compute/disks
Location           : eastus2
Tags               : {}
```


## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à créer une machine virtuelle dans une zone de disponibilité. Apprenez-en davantage sur la [disponibilité](../availability.md) des machines virtuelles Azure.