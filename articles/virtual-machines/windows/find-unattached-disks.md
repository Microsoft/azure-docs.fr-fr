---
title: Trouver et supprimer les disques managés et non managés Azure non attachés
description: Comment rechercher et supprimer les disques (disques durs virtuels/objets blob de pages) managés et non managés Azure non attachés à l’aide d’Azure PowerShell.
author: roygara
ms.service: virtual-machines-windows
ms.topic: how-to
ms.date: 02/22/2019
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: 4f217a53c23df4f161207aaceb528680ddcddbe7
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91972796"
---
# <a name="find-and-delete-unattached-azure-managed-and-unmanaged-disks"></a>Trouver et supprimer les disques managés et non managés Azure non attachés

Par défaut, lorsque vous supprimez une machine virtuelle dans Azure, tous les disques qui sont attachés à cette machine ne sont pas supprimés. Cette fonctionnalité permet d’éviter une perte de données causée par la suppression involontaire de machines virtuelles. Après la suppression d’une machine virtuelle, vous continuez de payer pour les disques non attachés. Cet article explique comment rechercher et supprimer tous les disques non attachés pour réduire les coûts inutiles.

## <a name="managed-disks-find-and-delete-unattached-disks"></a>Disques managés : Rechercher et supprimer les disques non attachés

Le script suivant recherche des [disques managés](../managed-disks-overview.md) non attachés en examinant la valeur de la propriété **ManagedBy**. Lorsqu’un disque managé est attaché à une machine virtuelle, la propriété **ManagedBy** contient l’ID de ressource de la machine virtuelle. Lorsqu’un disque managé est non attaché, la propriété **ManagedBy** a la valeur null. Le script examine tous les disques managés dans un abonnement Azure. Lorsque le script localise un disque managé dont la propriété **ManagedBy** a la valeur null, il détermine que le disque n’est pas attaché.

>[!IMPORTANT]
>En premier lieu, exécutez le script en définissant la variable **deleteUnattachedDisks** sur 0. Cette action vous permet de rechercher et d’afficher tous les disques managés non attachés.
>
>Après avoir examiné tous les disques non attachés, exécutez de nouveau le script et définissez la valeur de la variable **deleteUnattachedDisks** sur 1. Cette action vous permet de supprimer tous les disques managés non attachés.

```azurepowershell-interactive
# Set deleteUnattachedDisks=1 if you want to delete unattached Managed Disks
# Set deleteUnattachedDisks=0 if you want to see the Id of the unattached Managed Disks
$deleteUnattachedDisks=0
$managedDisks = Get-AzDisk
foreach ($md in $managedDisks) {
    # ManagedBy property stores the Id of the VM to which Managed Disk is attached to
    # If ManagedBy property is $null then it means that the Managed Disk is not attached to a VM
    if($md.ManagedBy -eq $null){
        if($deleteUnattachedDisks -eq 1){
            Write-Host "Deleting unattached Managed Disk with Id: $($md.Id)"
            $md | Remove-AzDisk -Force
            Write-Host "Deleted unattached Managed Disk with Id: $($md.Id) "
        }else{
            $md.Id
        }
    }
 }
```

## <a name="unmanaged-disks-find-and-delete-unattached-disks"></a>Disques non managés : Rechercher et supprimer les disques non attachés

Les disques non managés sont des fichiers VHD qui sont stockés en tant [qu’objets blob de pages](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-page-blobs) dans des [comptes de stockage Azure](../../storage/common/storage-account-overview.md). Le script suivant recherche les disques non managés non attachés (objets blob de pages) en examinant la valeur de la propriété **LeaseStatus**. Lorsqu’un disque non managé est attaché à une machine virtuelle, la valeur de la propriété **LeaseStatus** est définie sur **Locked**. Lorsqu’un disque non managé est non attaché, la valeur de la propriété **LeaseStatus** est définie sur **Unlocked**. Le script examine tous les disques non managés dans l’ensemble des comptes de stockage Azure d’un abonnement Azure. Lorsque le script localise un disque non managé dont une propriété **LeaseStatus** a la valeur**Unlocked**, il détermine que le disque n’est pas attaché.

>[!IMPORTANT]
>En premier lieu, exécutez le script en définissant la variable **deleteUnattachedVHDs** sur `$false`. Cette action vous permet de rechercher et d’afficher tous les disques durs virtuels non managés non attachés.
>
>Après avoir examiné tous les disques non attachés, exécutez de nouveau le script et définissez la valeur de la variable **deleteUnattachedVHDs** sur `$true`. Cette action vous permet de supprimer tous les disques durs virtuels non managés non attachés.

```azurepowershell-interactive
# Set deleteUnattachedVHDs=$true if you want to delete unattached VHDs
# Set deleteUnattachedVHDs=$false if you want to see the Uri of the unattached VHDs
$deleteUnattachedVHDs=$false
$storageAccounts = Get-AzStorageAccount
foreach($storageAccount in $storageAccounts){
    $storageKey = (Get-AzStorageAccountKey -ResourceGroupName $storageAccount.ResourceGroupName -Name $storageAccount.StorageAccountName)[0].Value
    $context = New-AzStorageContext -StorageAccountName $storageAccount.StorageAccountName -StorageAccountKey $storageKey
    $containers = Get-AzStorageContainer -Context $context
    foreach($container in $containers){
        $blobs = Get-AzStorageBlob -Container $container.Name -Context $context
        #Fetch all the Page blobs with extension .vhd as only Page blobs can be attached as disk to Azure VMs
        $blobs | Where-Object {$_.BlobType -eq 'PageBlob' -and $_.Name.EndsWith('.vhd')} | ForEach-Object { 
            #If a Page blob is not attached as disk then LeaseStatus will be unlocked
            if($_.ICloudBlob.Properties.LeaseStatus -eq 'Unlocked'){
                    if($deleteUnattachedVHDs){
                        Write-Host "Deleting unattached VHD with Uri: $($_.ICloudBlob.Uri.AbsoluteUri)"
                        $_ | Remove-AzStorageBlob -Force
                        Write-Host "Deleted unattached VHD with Uri: $($_.ICloudBlob.Uri.AbsoluteUri)"
                    }
                    else{
                        $_.ICloudBlob.Uri.AbsoluteUri
                    }
            }
        }
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Supprimer un compte de stockage](../../storage/common/storage-account-create.md#delete-a-storage-account) et [Identify Orphaned Disks Using PowerShell](/archive/blogs/ukplatforms/azure-cost-optimisation-series-identify-orphaned-disks-using-powershell) (Identifier les disques orphelins à l’aide de PowerShell)