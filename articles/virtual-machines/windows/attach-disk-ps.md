---
title: Attacher un disque de données à une machine virtuelle Windows dans Azure à l’aide de PowerShell
description: Comment attacher un disque de données nouveau ou existant à une machine virtuelle Windows à l’aide de PowerShell avec le modèle de déploiement Resource Manager.
author: roygara
ms.service: virtual-machines-windows
ms.topic: how-to
ms.date: 10/16/2018
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: baeeeae54829fad7378df597965eec5ba2039d20
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91972813"
---
# <a name="attach-a-data-disk-to-a-windows-vm-with-powershell"></a>Attacher un disque de données à une machine virtuelle Windows dans Azure avec PowerShell

Cet article vous explique comment attacher des disques nouveaux et existants à une machine virtuelle Windows à l’aide de PowerShell. 

Commencez par passer en revue ces conseils :

* La taille de la machine virtuelle détermine le nombre de disques de données que vous pouvez attacher . Pour plus d’informations, consultez [Tailles des machines virtuelles](../sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
* Pour utiliser des disques SSD Premium, il vous faut un [type de machine virtuelle compatible avec le stockage Premium](../sizes-memory.md), par exemple une machine virtuelle de série DS ou GS.

Cet article utilise PowerShell dans [Azure Cloud Shell](../../cloud-shell/overview.md), qui est constamment mise à jour vers la dernière version. Pour ouvrir Cloud Shell, sélectionnez **Essayer** en haut d’un bloc de code.

## <a name="add-an-empty-data-disk-to-a-virtual-machine"></a>Ajouter un disque de données vide à une machine virtuelle

Cet exemple montre comment ajouter un disque de données vide à une machine virtuelle existante.

### <a name="using-managed-disks"></a>Utilisation de disques managés

```azurepowershell-interactive
$rgName = 'myResourceGroup'
$vmName = 'myVM'
$location = 'East US' 
$storageType = 'Premium_LRS'
$dataDiskName = $vmName + '_datadisk1'

$diskConfig = New-AzDiskConfig -SkuName $storageType -Location $location -CreateOption Empty -DiskSizeGB 128
$dataDisk1 = New-AzDisk -DiskName $dataDiskName -Disk $diskConfig -ResourceGroupName $rgName

$vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName 
$vm = Add-AzVMDataDisk -VM $vm -Name $dataDiskName -CreateOption Attach -ManagedDiskId $dataDisk1.Id -Lun 1

Update-AzVM -VM $vm -ResourceGroupName $rgName
```

### <a name="using-managed-disks-in-an-availability-zone"></a>Utilisation de disques managés dans un groupe à haute disponibilité

Pour créer un disque dans une zone de disponibilité, utilisez [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig) avec le paramètre `-Zone`. L’exemple suivant crée un disque dans la zone *1*.

```powershell
$rgName = 'myResourceGroup'
$vmName = 'myVM'
$location = 'East US 2'
$storageType = 'Premium_LRS'
$dataDiskName = $vmName + '_datadisk1'

$diskConfig = New-AzDiskConfig -SkuName $storageType -Location $location -CreateOption Empty -DiskSizeGB 128 -Zone 1
$dataDisk1 = New-AzDisk -DiskName $dataDiskName -Disk $diskConfig -ResourceGroupName $rgName

$vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName 
$vm = Add-AzVMDataDisk -VM $vm -Name $dataDiskName -CreateOption Attach -ManagedDiskId $dataDisk1.Id -Lun 1

Update-AzVM -VM $vm -ResourceGroupName $rgName
```

### <a name="initialize-the-disk"></a>Initialiser le disque

Après avoir ajouté un disque vide, vous devez l’initialiser. Pour initialiser le disque, vous pouvez vous connecter à une machine virtuelle et utiliser la gestion des disques. Si vous avez activé [WinRM](/windows/desktop/winrm/portal) et un certificat sur la machine virtuelle lors de sa création, vous pouvez utiliser PowerShell à distance pour initialiser le disque. Vous pouvez également utiliser une extension de script personnalisée :

```azurepowershell-interactive
    $location = "location-name"
    $scriptName = "script-name"
    $fileName = "script-file-name"
    Set-AzVMCustomScriptExtension -ResourceGroupName $rgName -Location $locName -VMName $vmName -Name $scriptName -TypeHandlerVersion "1.4" -StorageAccountName "mystore1" -StorageAccountKey "primary-key" -FileName $fileName -ContainerName "scripts"
```

Le fichier de script peut contenir du code pour initialiser les disques, par exemple :

```azurepowershell-interactive
    $disks = Get-Disk | Where partitionstyle -eq 'raw' | sort number

    $letters = 70..89 | ForEach-Object { [char]$_ }
    $count = 0
    $labels = "data1","data2"

    foreach ($disk in $disks) {
        $driveLetter = $letters[$count].ToString()
        $disk | 
        Initialize-Disk -PartitionStyle MBR -PassThru |
        New-Partition -UseMaximumSize -DriveLetter $driveLetter |
        Format-Volume -FileSystem NTFS -NewFileSystemLabel $labels[$count] -Confirm:$false -Force
    $count++
    }
```

## <a name="attach-an-existing-data-disk-to-a-vm"></a>Ajouter un disque de données existant à une machine virtuelle

Vous pouvez attacher un disque géré existant à une machine virtuelle en tant que disque de données.

```azurepowershell-interactive
$rgName = "myResourceGroup"
$vmName = "myVM"
$location = "East US" 
$dataDiskName = "myDisk"
$disk = Get-AzDisk -ResourceGroupName $rgName -DiskName $dataDiskName 

$vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName 

$vm = Add-AzVMDataDisk -CreateOption Attach -Lun 0 -VM $vm -ManagedDiskId $disk.Id

Update-AzVM -VM $vm -ResourceGroupName $rgName
```

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également déployer des disques managés à l’aide de modèles. Pour plus d’informations, consultez [Utilisation de disques managés dans les modèles Azure Resource Manager](../using-managed-disks-template-deployments.md) ou le [modèle de démarrage rapide](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-multiple-data-disk) consacré au déploiement de plusieurs disques de données.