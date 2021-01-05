---
title: Remplacer le disque du système d’exploitation d’une machine virtuelle Azure avec PowerShell
description: Utilisez PowerShell pour remplacer le disque du système d’exploitation utilisé par une machine virtuelle Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 04/24/2018
ms.author: cynthn
ms.openlocfilehash: 2d4abd68e60201a17e56ee105777614c42691bfc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87020336"
---
# <a name="change-the-os-disk-used-by-an-azure-vm-using-powershell"></a>Remplacer le disque du système d’exploitation utilisé par une machine virtuelle Azure à l’aide de PowerShell

Pour remplacer le disque d’une machine virtuelle existante par un disque de sauvegarde ou un autre disque de système d’exploitation, vous pouvez utiliser Azure PowerShell. Il est inutile de supprimer et de recréer la machine virtuelle. Vous pouvez même utiliser le disque managé d’un autre groupe de ressources s’il n’est pas déjà utilisé.

 

Vous devez arrêter/libérer la machine virtuelle, puis remplacer l’ID de ressource du disque managé par celui d’un autre disque managé.

Vérifiez que la taille et le type de stockage de la machine virtuelle sont compatibles avec le disque à attacher. Ainsi, si le disque que vous souhaitez utiliser bénéficie d’un stockage Premium, la machine virtuelle doit être de taille suffisante pour prendre en charge ce niveau de stockage (comme une série DS). Les deux disques doivent également avoir la même taille.
Et veillez à ne pas combiner une machine virtuelle non chiffrée avec un disque de système d’exploitation chiffré, car cela n’est pas pris en charge. Si la machine virtuelle n’utilise pas Azure Disk Encryption, le disque du système d’exploitation qui est échangé ne doit pas utiliser Azure Disk Encryption.

Obtenir la liste des disques d'un groupe de ressources à l'aide de [Get-AzDisk](/powershell/module/az.compute/get-azdisk)

```azurepowershell-interactive
Get-AzDisk -ResourceGroupName myResourceGroup | Format-Table -Property Name
```
 
Après avoir obtenu le nom du disque à utiliser, définissez-le comme disque du système d’exploitation pour la machine virtuelle. Cet exemple arrête/libère la machine virtuelle nommée *myVM* et affecte le disque nommé *newDisk* comme nouveau disque du système d’exploitation. 
 
```azurepowershell-interactive 
# Get the VM 
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM 

# Make sure the VM is stopped\deallocated
Stop-AzVM -ResourceGroupName myResourceGroup -Name $vm.Name -Force

# Get the new disk that you want to swap in
$disk = Get-AzDisk -ResourceGroupName myResourceGroup -Name newDisk

# Set the VM configuration to point to the new disk  
Set-AzVMOSDisk -VM $vm -ManagedDiskId $disk.Id -Name $disk.Name 

# Update the VM with the new OS disk
Update-AzVM -ResourceGroupName myResourceGroup -VM $vm 

# Start the VM
Start-AzVM -Name $vm.Name -ResourceGroupName myResourceGroup

```

**Étapes suivantes**

Pour créer une copie d’un disque, consultez [Effectuer la capture instantanée d’un disque](snapshot-copy-managed-disk.md).
