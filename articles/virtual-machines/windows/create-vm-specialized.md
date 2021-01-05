---
title: Créer une machine virtuelle Windows à partir d’un disque dur virtuel spécialisé dans Azure
description: Créez une machine virtuelle Windows en attachant un disque managé spécialisé comme disque du système d’exploitation à l’aide du modèle de déploiement Resource Manager.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 10/10/2019
ms.author: cynthn
ms.openlocfilehash: cddc7f4f453f22b0cb36b1d3a1e9c2fba2dcabaf
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96455098"
---
# <a name="create-a-windows-vm-from-a-specialized-disk-by-using-powershell"></a>Créer une machine virtuelle Windows à partir d’un disque spécialisé à l’aide de PowerShell

Créez une machine virtuelle en attachant un disque managé spécialisé en tant que disque du système d’exploitation. Un disque spécialisé est une copie d’un disque dur virtuel d’une machine virtuelle existante qui contient les comptes d’utilisateurs, applications et autres données d’état de votre machine virtuelle d’origine. 

Vous disposez de plusieurs options :
* [Utilisez un disque managé existant](#option-1-use-an-existing-disk). Cette option est utile si vous avez une machine virtuelle qui ne fonctionne pas correctement. Vous pouvez supprimer la machine virtuelle, puis réutiliser le disque managé pour créer une machine virtuelle. 
* [Charger un disque dur virtuel](#option-2-upload-a-specialized-vhd) 
* [Copier une machine virtuelle Azure existante à l’aide de captures instantanées](#option-3-copy-an-existing-azure-vm)

Vous pouvez également utiliser le portail Azure pour [créer une machine virtuelle à partir d’un disque dur virtuel spécialisé](create-vm-specialized-portal.md).

Cet article montre comment utiliser des disques managés. Si vous avez un déploiement hérité qui nécessite l’utilisation d’un compte de stockage, consultez [Créer une machine virtuelle à partir d’un disque dur virtuel spécialisé dans un compte de stockage](/previous-versions/azure/virtual-machines/windows/sa-create-vm-specialized).

> [!IMPORTANT]
> 
> Lorsque vous utilisez un disque spécialisé pour créer une machine virtuelle, la nouvelle machine conserve le nom d’ordinateur de la machine virtuelle d’origine. D’autres informations spécifiques à l’ordinateur (par exemple le CMID) sont également conservées. Dans certains cas, ces informations en doublon peuvent provoquer des problèmes. Quand vous copiez une machine virtuelle, vous devez connaître les types d’informations spécifiques à l’ordinateur dont vos applications dépendent.  
> N’utilisez donc pas de disque spécialisé si vous souhaitez créer plusieurs machines virtuelles. Pour les déploiements plus importants, [créez une image](capture-image-resource.md), puis [utilisez-la pour créer plusieurs machines virtuelles](create-vm-generalized-managed.md).

Nous vous recommandons de limiter le nombre de déploiements simultanés à 20 machines virtuelles provenant d’un seul disque dur virtuel ou d’une seule capture instantanée. 

## <a name="option-1-use-an-existing-disk"></a>Option 1 : Utiliser un disque existant

Si vous avez supprimé une machine virtuelle et souhaitez réutiliser le disque du système d'exploitation pour créer une nouvelle machine virtuelle, utilisez [Get-AzDisk](/powershell/module/az.compute/get-azdisk).

```powershell
$resourceGroupName = 'myResourceGroup'
$osDiskName = 'myOsDisk'
$osDisk = Get-AzDisk `
-ResourceGroupName $resourceGroupName `
-DiskName $osDiskName
```
Vous pouvez maintenant joindre ce disque en tant que disque de système d’exploitation à une [nouvelle machine virtuelle](#create-the-new-vm).

## <a name="option-2-upload-a-specialized-vhd"></a>Option 2 : Charger un disque dur virtuel spécialisé

Vous pouvez charger le disque dur virtuel à partir d’une machine virtuelle spécialisée créée avec un outil de virtualisation local tel que Hyper-V, ou d’une machine virtuelle exportée à partir d’un autre cloud.

### <a name="prepare-the-vm"></a>Préparation de la machine virtuelle
Utilisez le disque dur virtuel en l’état pour créer une machine virtuelle. 
  
  * [Préparez un disque dur virtuel Windows à charger sur Azure](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). **Ne généralisez pas** la machine virtuelle à l’aide de Sysprep.
  * Supprimez tous les outils et agents de virtualisation invités qui sont installés sur la machine virtuelle (notamment les outils VMware).
  * Vérifiez que la machine virtuelle est configurée pour obtenir l’adresse IP et les paramètres DNS à partir de DHCP. De cette façon, le serveur obtient une adresse IP dans le réseau virtuel au démarrage. 


### <a name="upload-the-vhd"></a>Charger le disque dur virtuel

Vous pouvez désormais charger un disque dur virtuel directement dans un disque managé. Pour plus d’instructions, consultez [Charger un disque dur virtuel dans Azure à l'aide d'Azure PowerShell](disks-upload-vhd-to-managed-disk-powershell.md).

## <a name="option-3-copy-an-existing-azure-vm"></a>Option 3 : Copier une machine virtuelle Azure existante

Vous pouvez créer une copie d’une machine virtuelle qui utilise des disques managés en prenant une capture instantanée de la machine virtuelle, puis en utilisant cette capture instantanée pour créer un disque managé et une machine virtuelle.

Si vous souhaitez copier une machine virtuelle existante dans une autre région, vous souhaiterez peut-être utiliser azcopy pour [créer une copie d’un disque dans une autre région](disks-upload-vhd-to-managed-disk-powershell.md#copy-a-managed-disk). 

### <a name="take-a-snapshot-of-the-os-disk"></a>Prendre une capture instantanée du disque de système d’exploitation

Vous pouvez prendre une capture instantanée d’une machine virtuelle entière (y compris tous les disques) ou d’un seul disque. Les étapes suivantes vous montrent comment créer une capture instantanée du disque du système d'exploitation de votre machine virtuelle à l'aide de la cmdlet [New-AzSnapshot](/powershell/module/az.compute/new-azsnapshot). 

Commencez par définir des paramètres. 

 ```powershell
$resourceGroupName = 'myResourceGroup' 
$vmName = 'myVM'
$location = 'westus' 
$snapshotName = 'mySnapshot'  
```

Obtenez l’objet machine virtuelle.

```powershell
$vm = Get-AzVM -Name $vmName `
   -ResourceGroupName $resourceGroupName
```
Obtenez le nom du disque de système d’exploitation.

 ```powershell
$disk = Get-AzDisk -ResourceGroupName $resourceGroupName `
   -DiskName $vm.StorageProfile.OsDisk.Name
```

Créez la configuration de capture instantanée. 

 ```powershell
$snapshotConfig =  New-AzSnapshotConfig `
   -SourceUri $disk.Id `
   -OsType Windows `
   -CreateOption Copy `
   -Location $location 
```

Prenez la capture instantanée.

```powershell
$snapShot = New-AzSnapshot `
   -Snapshot $snapshotConfig `
   -SnapshotName $snapshotName `
   -ResourceGroupName $resourceGroupName
```


Pour utiliser cette capture instantanée afin de créer une machine virtuelle hautement performante, ajoutez le paramètre `-AccountType Premium_LRS` à la commande New-AzSnapshot. Ce paramètre crée la capture instantanée et la stocke en tant que disque managé Premium. Les disques managés Premium sont plus chers que les disques Standard. Vérifiez donc que vous avez besoin de disques Premium avant d’utiliser ce paramètre.

### <a name="create-a-new-disk-from-the-snapshot"></a>Créer un disque à partir de la capture instantanée

Créez un disque managé à partir de la capture instantanée en utilisant la commande [New-AzDisk](/powershell/module/az.compute/new-azdisk). Cet exemple utilise *myOSDisk* comme nom du disque.

Créez un groupe de ressources pour la nouvelle machine virtuelle.

```powershell
$destinationResourceGroup = 'myDestinationResourceGroup'
New-AzResourceGroup -Location $location `
   -Name $destinationResourceGroup
```

Définissez le nom du disque de système d’exploitation. 

```powershell
$osDiskName = 'myOsDisk'
```

Créez le disque géré.

```powershell
$osDisk = New-AzDisk -DiskName $osDiskName -Disk `
    (New-AzDiskConfig  -Location $location -CreateOption Copy `
    -SourceResourceId $snapshot.Id) `
    -ResourceGroupName $destinationResourceGroup
```


## <a name="create-the-new-vm"></a>Créer la machine virtuelle 

Créez la mise en réseau et d’autres ressources de machine virtuelle que la nouvelle machine virtuelle doit utiliser.

### <a name="create-the-subnet-and-virtual-network"></a>Créer le sous-réseau et le réseau virtuel

Créez le [réseau virtuel](../../virtual-network/virtual-networks-overview.md) et le sous-réseau de la machine virtuelle.

1. Créez le sous-réseau. L’exemple suivant crée un sous-réseau nommé *mySubnet* dans le groupe de ressources *myDestinationResourceGroup*, et définit le préfixe d’adresse de sous-réseau sur *10.0.0.0/24*.
   
    ```powershell
    $subnetName = 'mySubNet'
    $singleSubnet = New-AzVirtualNetworkSubnetConfig `
       -Name $subnetName `
       -AddressPrefix 10.0.0.0/24
    ```
    
2. Création du réseau virtuel. Cet exemple définit *myVnetName* comme nom de réseau virtuel, *USA Ouest* comme emplacement et *10.0.0.0/16* comme préfixe d’adresse du réseau virtuel. 
   
    ```powershell
    $vnetName = "myVnetName"
    $vnet = New-AzVirtualNetwork `
       -Name $vnetName -ResourceGroupName $destinationResourceGroup `
       -Location $location `
       -AddressPrefix 10.0.0.0/16 `
       -Subnet $singleSubnet
    ```    
    

### <a name="create-the-network-security-group-and-an-rdp-rule"></a>Créer le groupe de sécurité réseau et une règle RDP
Pour pouvoir vous connecter à votre machine virtuelle avec le protocole RDP (Remote Desktop Protocol), vous devez disposer d’une règle de sécurité qui autorise l’accès RDP sur le port 3389. Dans notre exemple, le disque dur virtuel de la nouvelle machine virtuelle a été créé à partir d’une machine virtuelle spécialisée. Vous pouvez donc utiliser un compte de la machine virtuelle source pour le protocole RDP.

Cet exemple donne au groupe de sécurité réseau le nom *myNsg* et à la règle RDP le nom *myRdpRule*.

```powershell
$nsgName = "myNsg"

$rdpRule = New-AzNetworkSecurityRuleConfig -Name myRdpRule -Description "Allow RDP" `
    -Access Allow -Protocol Tcp -Direction Inbound -Priority 110 `
    -SourceAddressPrefix Internet -SourcePortRange * `
    -DestinationAddressPrefix * -DestinationPortRange 3389
$nsg = New-AzNetworkSecurityGroup `
   -ResourceGroupName $destinationResourceGroup `
   -Location $location `
   -Name $nsgName -SecurityRules $rdpRule
    
```

Pour plus d’informations sur les points de terminaison et les règles NSG, consultez [Ouverture de ports sur une machine virtuelle dans Azure avec PowerShell](nsg-quickstart-powershell.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

### <a name="create-a-public-ip-address-and-nic"></a>Créer une adresse IP publique et une carte réseau
Pour établir la communication avec la machine virtuelle dans le réseau virtuel, vous avez besoin d’une [adresse IP publique](../../virtual-network/public-ip-addresses.md) et d’une interface réseau.

1. Créez l’adresse IP publique. Dans cet exemple, le nom d’adresse IP publique est défini sur *myIP*.
   
    ```powershell
    $ipName = "myIP"
    $pip = New-AzPublicIpAddress `
       -Name $ipName -ResourceGroupName $destinationResourceGroup `
       -Location $location `
       -AllocationMethod Dynamic
    ```       
    
2. Créez la carte réseau. Dans cet exemple, le nom de carte réseau est défini sur *myNicName*.
   
    ```powershell
    $nicName = "myNicName"
    $nic = New-AzNetworkInterface -Name $nicName `
       -ResourceGroupName $destinationResourceGroup `
       -Location $location -SubnetId $vnet.Subnets[0].Id `
       -PublicIpAddressId $pip.Id `
       -NetworkSecurityGroupId $nsg.Id
    ```
    


### <a name="set-the-vm-name-and-size"></a>Définir le nom et la taille de machine virtuelle

Cet exemple définit *myVM* en tant que nom de machine virtuelle, et *Standard_A2* pour la taille de celle-ci.

```powershell
$vmName = "myVM"
$vmConfig = New-AzVMConfig -VMName $vmName -VMSize "Standard_A2"
```

### <a name="add-the-nic"></a>Ajouter la carte réseau
    
```powershell
$vm = Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id
```
    

### <a name="add-the-os-disk"></a>Ajouter le disque de système d’exploitation 

Ajoutez le disque du système d'exploitation à la configuration en utilisant la commande [Set-AzVMOSDisk](/powershell/module/az.compute/set-azvmosdisk). Cet exemple définit la taille du disque sur *128 Go* et attache le disque géré en tant que disque de système d’exploitation *Windows*.
 
```powershell
$vm = Set-AzVMOSDisk -VM $vm -ManagedDiskId $osDisk.Id -StorageAccountType Standard_LRS `
    -DiskSizeInGB 128 -CreateOption Attach -Windows
```

### <a name="complete-the-vm"></a>Terminer la machine virtuelle 

Créez la machine virtuelle en utilisant la commande [New-AzVM](/powershell/module/az.compute/new-azvm) avec les configurations que nous venons de créer.

```powershell
New-AzVM -ResourceGroupName $destinationResourceGroup -Location $location -VM $vm
```

Si cette commande aboutit, vous obtenez une sortie similaire à celle-ci :

```powershell
RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK   

```

### <a name="verify-that-the-vm-was-created"></a>Vérifier que la machine virtuelle a été créée
Vous devez voir la machine virtuelle récemment créée soit dans le [portail Azure](https://portal.azure.com) sous **Parcourir** > **Machines virtuelles**, soit en utilisant les commandes PowerShell suivantes.

```powershell
$vmList = Get-AzVM -ResourceGroupName $destinationResourceGroup
$vmList.Name
```

## <a name="next-steps"></a>Étapes suivantes
Connectez-vous à votre nouvelle machine virtuelle. Pour plus d’informations, consultez [Connexion à une machine virtuelle Azure exécutant Windows](connect-logon.md).