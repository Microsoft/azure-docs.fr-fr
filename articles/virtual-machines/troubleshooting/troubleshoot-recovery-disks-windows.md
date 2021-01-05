---
title: Utilisation d’une machine virtuelle de dépannage Windows avec Azure PowerShell | Microsoft Docs
description: Découvrez comment résoudre les problèmes de machines virtuelles Windows dans Azure en connectant le disque du système d’exploitation à une machine virtuelle de récupération à l’aide d’Azure PowerShell
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 08/09/2018
ms.author: genli
ms.openlocfilehash: 63d5e4bcc57a734fc5ea455da7c2db940a4b8ec3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88654682"
---
# <a name="troubleshoot-a-windows-vm-by-attaching-the-os-disk-to-a-recovery-vm-using-azure-powershell"></a>Résoudre les problèmes d’une machine virtuelle Windows en connectant le disque du système d’exploitation à une machine virtuelle de récupération à l’aide d’Azure PowerShell
Si votre machine virtuelle Windows dans Azure rencontre une erreur de démarrage ou de disque, vous devrez peut-être appliquer la procédure de résolution de problèmes directement sur le disque. Comme exemple courant, citons l’échec de mise à jour d’une application qui empêche le bon démarrage de la machine virtuelle. Cet article vous explique comment utiliser Azure PowerShell pour connecter le disque à une autre machine virtuelle Windows et corriger les éventuelles erreurs, puis réparer votre machine virtuelle d’origine. 

> [!Important]
> Les scripts dans cet article s’appliquent uniquement aux machines virtuelles qui utilisent [Disque managé](../managed-disks-overview.md). 

 

## <a name="recovery-process-overview"></a>Vue d’ensemble du processus de récupération
Nous pouvons maintenant utiliser Azure PowerShell et changer le disque de système d’exploitation pour une machine virtuelle. Il n’est désormais plus nécessaire de supprimer et de recréer la machine virtuelle.

Le processus de résolution de problème se présente comme suit :

1. Arrêter la machine virtuelle affectée.
2. Créer un instantané à partir du disque de système d’exploitation de la machine virtuelle.
3. Créer un disque à partir de la capture instantanée du disque de système d’exploitation.
4. Attacher le disque en tant que disque de données à une machine virtuelle de récupération.
5. Se connecter à la machine virtuelle de récupération. Modifier les fichiers ou exécuter des outils afin de corriger les problèmes sur le disque du système d’exploitation copié.
6. Démonter et détacher le disque à partir de la machine virtuelle de récupération.
7. Changer le disque de système d’exploitation pour la machine virtuelle affectée.

Vous pouvez utiliser les commandes de réparation de machine virtuelle pour automatiser les étapes 1, 2, 3, 4, 6 et 7. Pour obtenir plus de documentation et d’instructions, consultez [Réparer une machine virtuelle Windows à l’aide des commandes de réparation de machine virtuelle Azure](repair-windows-vm-using-azure-virtual-machine-repair-commands.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

Vérifiez que la [dernière version d’Azure PowerShell](/powershell/azure/) est installée et connectez-vous à votre abonnement :

```powershell
Connect-AzAccount
```

Dans les exemples suivants, remplacez les noms des paramètres par vos propres valeurs. 

## <a name="determine-boot-issues"></a>Identifier les problèmes de démarrage
Vous pouvez afficher une capture d’écran de votre machine virtuelle dans Azure pour vous aider à résoudre les problèmes de démarrage. Cette capture d’écran peut vous aider à identifier la cause de l’échec de démarrage d’une machine virtuelle. L’exemple suivant récupère la capture d’écran de la machine virtuelle Windows nommée `myVM` dans le groupe de ressources nommé `myResourceGroup` :

```powershell
Get-AzVMBootDiagnosticsData -ResourceGroupName myResourceGroup `
    -Name myVM -Windows -LocalPath C:\Users\ops\
```

Examinez la capture d’écran afin d’identifier la raison pour laquelle le démarrage de la machine virtuelle est mis en échec. Notez les messages d’erreur ou les codes d’erreur spécifiques fournis.

## <a name="stop-the-vm"></a>Arrêtez la machine virtuelle.

L’exemple suivant arrête la machine virtuelle nommée `myVM` dans le groupe de ressource nommé `myResourceGroup` :

```powershell
Stop-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"
```

Attendez que la machine virtuelle ait terminé la suppression avant d’effectuer l’étape suivante.


## <a name="create-a-snapshot-from-the-os-disk-of-the-vm"></a>Créer un instantané à partir du disque de système d’exploitation de la machine virtuelle

L’exemple suivant crée une capture instantanée avec le nom `mySnapshot` à partir du disque de système d’exploitation de la machine virtuelle nommée « myVM ». 

```powershell
$resourceGroupName = 'myResourceGroup' 
$location = 'eastus' 
$vmName = 'myVM'
$snapshotName = 'mySnapshot'  

#Get the VM
$vm = get-azvm `
-ResourceGroupName $resourceGroupName `
-Name $vmName

#Create the snapshot configuration for the OS disk
$snapshot =  New-AzSnapshotConfig `
-SourceUri $vm.StorageProfile.OsDisk.ManagedDisk.Id `
-Location $location `
-CreateOption copy

#Take the snapshot
New-AzSnapshot `
   -Snapshot $snapshot `
   -SnapshotName $snapshotName `
   -ResourceGroupName $resourceGroupName 
```

Une capture instantanée est une copie complète en lecture seule d’un disque dur virtuel. Elle ne peut pas être attachée à une machine virtuelle. Dans l’étape suivante, vous allez créer un disque à partir de cet instantané.

## <a name="create-a-disk-from-the-snapshot"></a>Créer un disque à partir de la capture instantanée

Ce script crée un disque managé portant le nom `newOSDisk` à partir de l’instantané nommé `mysnapshot`.  

```powershell
#Set the context to the subscription Id where Managed Disk will be created
#You can skip this step if the subscription is already selected

$subscriptionId = 'yourSubscriptionId'

Select-AzSubscription -SubscriptionId $SubscriptionId

#Provide the name of your resource group
$resourceGroupName ='myResourceGroup'

#Provide the name of the snapshot that will be used to create Managed Disks
$snapshotName = 'mySnapshot' 

#Provide the name of the Managed Disk
$diskName = 'newOSDisk'

#Provide the size of the disks in GB. It should be greater than the VHD file size.
$diskSize = '128'

#Provide the storage type for Managed Disk. Premium_LRS or Standard_LRS.
$storageType = 'Standard_LRS'

#Provide the Azure region (e.g. westus) where Managed Disks will be located.
#This location should be same as the snapshot location
#Get all the Azure location using command below:
#Get-AzLocation
$location = 'eastus'

$snapshot = Get-AzSnapshot -ResourceGroupName $resourceGroupName -SnapshotName $snapshotName 
 
$diskConfig = New-AzDiskConfig -AccountType $storageType -Location $location -CreateOption Copy -SourceResourceId $snapshot.Id
 
New-AzDisk -Disk $diskConfig -ResourceGroupName $resourceGroupName -DiskName $diskName
```
Vous disposez maintenant d’une copie du disque de système d’exploitation d’origine. Vous pouvez monter ce disque sur une autre machine virtuelle Windows, à des fins de dépannage.

## <a name="attach-the-disk-to-another-windows-vm-for-troubleshooting"></a>Attacher le disque à une autre machine virtuelle Windows pour la résolution de problèmes

À présent, attachez la copie du disque de système d’exploitation d’origine à une machine virtuelle en tant que disque de données. Ce processus vous permet de corriger les éventuelles erreurs de configuration, ou d’examiner des fichiers journaux système ou d’application supplémentaires sur le disque. L’exemple suivant attache le disque nommé `newOSDisk` à la machine virtuelle nommée `RecoveryVM`.

> [!NOTE]
> Pour attacher le disque, la copie du disque de système d’exploitation d’origine et la machine virtuelle de récupération doivent se trouver sur le même emplacement.

```powershell
$rgName = "myResourceGroup"
$vmName = "RecoveryVM"
$location = "eastus" 
$dataDiskName = "newOSDisk"
$disk = Get-AzDisk -ResourceGroupName $rgName -DiskName $dataDiskName 

$vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName 

$vm = Add-AzVMDataDisk -CreateOption Attach -Lun 0 -VM $vm -ManagedDiskId $disk.Id

Update-AzVM -VM $vm -ResourceGroupName $rgName
```

## <a name="connect-to-the-recovery-vm-and-fix-issues-on-the-attached-disk"></a>Se connecter à la machine virtuelle de récupération et résoudre les problèmes sur le disque attaché

1. Effectuez une connexion Bureau à distance sur votre machine virtuelle de récupération à l’aide des informations d’identification appropriées. L’exemple suivant télécharge le fichier de connexion Bureau à distance pour la machine virtuelle nommée `RecoveryVM` dans le groupe de ressources nommé `myResourceGroup` et le télécharge vers `C:\Users\ops\Documents` »

    ```powershell
    Get-AzRemoteDesktopFile -ResourceGroupName "myResourceGroup" -Name "RecoveryVM" `
        -LocalPath "C:\Users\ops\Documents\myVMRecovery.rdp"
    ```

2. Le disque de données doit être automatiquement détecté et attaché. Affichez la liste des volumes attachés pour déterminer la lettre de lecteur comme suit :

    ```powershell
    Get-Disk
    ```

    L’exemple de sortie suivant montre le disque connecté à un disque **2**. (Vous pouvez également utiliser `Get-Volume` pour afficher la lettre de lecteur) :

    ```powershell
    Number   Friendly Name   Serial Number   HealthStatus   OperationalStatus   Total Size   Partition
                                                                                             Style
    ------   -------------   -------------   ------------   -----------------   ----------   ----------
    0        Virtual HD                                     Healthy             Online       127 GB MBR
    1        Virtual HD                                     Healthy             Online       50 GB MBR
    2        newOSDisk                                  Healthy             Online       127 GB MBR
    ```

Dès que la copie du disque de système d’exploitation d’origine est montée, vous pouvez exécuter les procédures de maintenance et de dépannage voulues. Une fois que vous avez résolu les problèmes, passez aux étapes suivantes.

## <a name="unmount-and-detach-original-os-disk"></a>Démonter et détacher le disque de système d’exploitation d’origine
Après avoir corrigé les erreurs, vous démontez et détachez le disque existant de votre machine virtuelle de récupération. Vous ne pouvez pas utiliser votre disque avec une autre machine virtuelle avant la libération du bail associant le disque à la machine virtuelle de récupération.

1. À partir de votre session Bureau à distance, démontez le disque de données sur votre machine virtuelle de récupération. Vous devez obtenir le numéro du disque à partir de l’applet de commande `Get-Disk` précédente. Ensuite, utilisez `Set-Disk` pour définir le disque comme étant hors connexion :

    ```powershell
    Set-Disk -Number 2 -IsOffline $True
    ```

    Confirmez que le disque est désormais défini comme étant hors connexion en utilisant `Get-Disk` à nouveau. L’exemple de sortie suivant montre que le disque est désormais défini comme étant hors connexion :

    ```powershell
    Number   Friendly Name   Serial Number   HealthStatus   OperationalStatus   Total Size   Partition
                                                                                             Style
    ------   -------------   -------------   ------------   -----------------   ----------   ----------
    0        Virtual HD                                     Healthy             Online       127 GB MBR
    1        Virtual HD                                     Healthy             Online       50 GB MBR
    2        Msft Virtu...                                  Healthy             Offline      127 GB MBR
    ```

2. Quittez la session Bureau à distance. À partir de votre session Azure PowerShell, supprimez le disque nommé `newOSDisk` de la machine virtuelle nommée « RecoveryVM ».

    ```powershell
    $myVM = Get-AzVM -ResourceGroupName "myResourceGroup" -Name "RecoveryVM"
    Remove-AzVMDataDisk -VM $myVM -Name "newOSDisk"
    Update-AzVM -ResourceGroup "myResourceGroup" -VM $myVM
    ```

## <a name="change-the-os-disk-for-the-affected-vm"></a>Changer le disque de système d’exploitation pour la machine virtuelle affectée

Vous pouvez utiliser Azure PowerShell pour échanger les disques de système d’exploitation. Il est inutile de supprimer et de recréer la machine virtuelle.

Cet exemple arrête la machine virtuelle nommée `myVM` et affecte le disque nommé `newOSDisk` en tant que nouveau disque de système d’exploitation. 

```powershell
# Get the VM 
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM 

# Make sure the VM is stopped\deallocated
Stop-AzVM -ResourceGroupName myResourceGroup -Name $vm.Name -Force

# Get the new disk that you want to swap in
$disk = Get-AzDisk -ResourceGroupName myResourceGroup -Name newDisk

# Set the VM configuration to point to the new disk  
Set-AzVMOSDisk -VM $vm -ManagedDiskId $disk.Id -Name $disk.Name  -sto

# Update the VM with the new OS disk. Possible values of StorageAccountType include: 'Standard_LRS' and 'Premium_LRS'
Update-AzVM -ResourceGroupName myResourceGroup -VM $vm -StorageAccountType <Type of the storage account >

# Start the VM
Start-AzVM -Name $vm.Name -ResourceGroupName myResourceGroup
```

## <a name="verify-and-enable-boot-diagnostics"></a>Vérifier et activer les diagnostics de démarrage

L’exemple suivant active l’extension de diagnostic sur la machine virtuelle nommée `myVMDeployed`, dans le groupe de ressources nommé `myResourceGroup` :

```powershell
$myVM = Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVMDeployed"
Set-AzVMBootDiagnostics -ResourceGroupName myResourceGroup -VM $myVM -enable
Update-AzVM -ResourceGroup "myResourceGroup" -VM $myVM
```

## <a name="next-steps"></a>Étapes suivantes
Si vous rencontrez des problèmes pour vous connecter à votre machine virtuelle, consultez [Dépannage d’une connexion Bureau à distance à une machine virtuelle Azure](troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Pour résoudre les problèmes liés à l’accès aux applications exécutées sur votre machine virtuelle, consultez [Résoudre les problèmes de connectivité des applications sur une machine virtuelle Windows](troubleshoot-app-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

Pour plus d’informations sur l’utilisation de Resource Manager, consultez la page [Présentation d’Azure Resource Manager](../../azure-resource-manager/management/overview.md).
