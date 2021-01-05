---
title: Exclure des disques de machines virtuelles Azure de la réplication avec Azure Site Recovery et Azure PowerShell
description: Découvrez comment exclure des disques pour les machines virtuelles Azure avec Azure Site Recovery à l’aide d’Azure PowerShell.
author: sideeksh
manager: rochakm
ms.topic: how-to
ms.date: 02/18/2019
ms.openlocfilehash: a21460279420c46b11c43615ae5ecc7bfa81de4d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86135805"
---
# <a name="exclude-disks-from-powershell-replication-of-azure-vms"></a>Exclure des disques de la réplication Powershell de machines virtuelles Azure

Cet article explique comment exclure des disques lors de la réplication de machines virtuelles Azure. Vous pouvez exclure des disques pour optimiser la bande passante utilisée pour la réplication ou les ressources côté cible que ces disques utilisent. Actuellement, cette fonctionnalité est uniquement disponible par le biais d’Azure PowerShell.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer :

- Assurez-vous que vous comprenez [l’architecture et les composants de récupération d’urgence](azure-to-azure-architecture.md).
- Vérifiez les [exigences de prise en charge](azure-to-azure-support-matrix.md) pour tous les composants.
- Assurez-vous que vous disposez du module AzureRm PowerShell « Az ». Pour installer ou mettre à jour powerShell, consultez la section relative à [l’installation du module Azure PowerShell](/powershell/azure/install-az-ps).
- Assurez-vous d’avoir créé un coffre Recovery Services et des machines virtuelles protégées au moins une fois. Si tel n’est pas le cas, suivez le processus décrit dans [Configurer la récupération d’urgence pour les machines virtuelles Azure à l’aide d’Azure PowerShell](azure-to-azure-powershell.md).
- Si vous recherchez des informations sur l’ajout de disques à une machine virtuelle Azure compatible avec la réplication, [consultez cet article](azure-to-azure-enable-replication-added-disk.md).

## <a name="why-exclude-disks-from-replication"></a>Pourquoi exclure des disques de la réplication ?
Vous devrez peut-être exclure les disques de la réplication, car :

- Votre machine virtuelle a atteint les [Limites d’Azure Site Recovery pour répliquer les données](./azure-to-azure-support-matrix.md).

- Les données hautement évolutives sur le disque exclu ne sont pas importantes ou n’ont pas besoin d’être répliquées.

- Vous voulez économiser des ressources de stockage et réseau en ne répliquant pas ces données.

## <a name="how-to-exclude-disks-from-replication"></a>Comment exclure des disques de la réplication ?

Dans notre exemple, nous répliquons une machine virtuelle disposant d’un système d’exploitation et de trois disques de données se trouvant dans la région USA Est et USA Ouest 2. Le nom de la machine virtuelle est *AzureDemoVM*. Nous excluons le disque 1 et conservons les disques 2 et 3.

## <a name="get-details-of-the-virtual-machines-to-replicate"></a>Obtenir des informations détaillées des machines virtuelles à répliquer

```azurepowershell
# Get details of the virtual machine
$VM = Get-AzVM -ResourceGroupName "A2AdemoRG" -Name "AzureDemoVM"

Write-Output $VM     
```

```
ResourceGroupName  : A2AdemoRG
Id                 : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/A2AdemoRG/providers/Microsoft.Compute/virtualMachines/AzureDemoVM
VmId               : 1b864902-c7ea-499a-ad0f-65da2930b81b
Name               : AzureDemoVM
Type               : Microsoft.Compute/virtualMachines
Location           : eastus
Tags               : {}
DiagnosticsProfile : {BootDiagnostics}
HardwareProfile    : {VmSize}
NetworkProfile     : {NetworkInterfaces}
OSProfile          : {ComputerName, AdminUsername, WindowsConfiguration, Secrets}
ProvisioningState  : Succeeded
StorageProfile     : {ImageReference, OsDisk, DataDisks}
```

Obtenir des informations sur les disques de la machine virtuelle. Ces informations seront utilisées plus tard au démarrage de la réplication de la machine virtuelle.

```azurepowershell
$OSDiskVhdURI = $VM.StorageProfile.OsDisk.Vhd
$DataDisk1VhdURI = $VM.StorageProfile.DataDisks[0].Vhd
```

## <a name="replicate-an-azure-virtual-machine"></a>Répliquer une machine virtuelle Azure

Dans l’exemple suivant, nous supposons que vous disposez déjà d’un compte de stockage de cache, d’une stratégie de réplication et de mappages. Si tel n’est pas le cas, suivez le processus décrit dans [Configurer la récupération d’urgence pour les machines virtuelles Azure à l’aide d’Azure PowerShell](azure-to-azure-powershell.md).

Répliquez une machine virtuelle Azure avec des *disques managés*.

```azurepowershell

#Get the resource group that the virtual machine must be created in when failed over.
$RecoveryRG = Get-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"

#Specify replication properties for each disk of the VM that is to be replicated (create disk replication configuration).

#OsDisk
$OSdiskId =  $vm.StorageProfile.OsDisk.ManagedDisk.Id
$RecoveryOSDiskAccountType = $vm.StorageProfile.OsDisk.ManagedDisk.StorageAccountType
$RecoveryReplicaDiskAccountType =  $vm.StorageProfile.OsDisk.ManagedDisk.StorageAccountType

$OSDiskReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id `
         -DiskId $OSdiskId -RecoveryResourceGroupId  $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType  $RecoveryReplicaDiskAccountType `
         -RecoveryTargetDiskAccountType $RecoveryOSDiskAccountType

# Data Disk 1 i.e StorageProfile.DataDisks[0] is excluded, so we will provide it during the time of replication.

# Data disk 2
$datadiskId2  = $vm.StorageProfile.DataDisks[1].ManagedDisk.id
$RecoveryReplicaDiskAccountType =  $vm.StorageProfile.DataDisks[1]. StorageAccountType
$RecoveryTargetDiskAccountType = $vm.StorageProfile.DataDisks[1]. StorageAccountType

$DataDisk2ReplicationConfig  = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $CacheStorageAccount.Id `
         -DiskId $datadiskId2 -RecoveryResourceGroupId  $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType  $RecoveryReplicaDiskAccountType `
         -RecoveryTargetDiskAccountType $RecoveryTargetDiskAccountType

# Data Disk 3

$datadiskId3  = $vm.StorageProfile.DataDisks[2].ManagedDisk.id
$RecoveryReplicaDiskAccountType =  $vm.StorageProfile.DataDisks[2]. StorageAccountType
$RecoveryTargetDiskAccountType = $vm.StorageProfile.DataDisks[2]. StorageAccountType

$DataDisk3ReplicationConfig  = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $CacheStorageAccount.Id `
         -DiskId $datadiskId3 -RecoveryResourceGroupId  $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType  $RecoveryReplicaDiskAccountType `
         -RecoveryTargetDiskAccountType $RecoveryTargetDiskAccountType

#Create a list of disk replication configuration objects for the disks of the virtual machine that are to be replicated.
$diskconfigs = @()
$diskconfigs += $OSDiskReplicationConfig, $DataDisk2ReplicationConfig, $DataDisk3ReplicationConfig


#Start replication by creating a replication protected item. Using a GUID for the name of the replication protected item to ensure uniqueness of name.
$TempASRJob = New-ASRReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId
```

Une fois l’opération de lancement de réplication terminée, les données de la machine virtuelle sont répliquées vers la région de récupération.

Vous pouvez accéder au portail Azure et voir les machines virtuelles répliquées sous « éléments répliqués ».

Le processus de réplication démarre en envoyant une copie des disques de réplication de la machine virtuelle dans la région de récupération. Cette phase est appelée phase de réplication initiale.

Une fois cette phase terminée, la réplication passe à la phase de synchronisation différentielle. À ce stade, la machine virtuelle est protégée. Sélectionnez la machine virtuelle protégée pour voir si des disques sont exclus.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur l’[exécution d’un test de basculement](site-recovery-test-failover-to-azure.md).
