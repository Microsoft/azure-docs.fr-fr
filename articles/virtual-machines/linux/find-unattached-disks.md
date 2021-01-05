---
title: Azure CLI – Recherche et suppression de disques non attachés, managés ou non
description: Comment rechercher et supprimer à l’aide d’Azure CLI les disques managés et non managés (disques durs virtuels/objets blob de pages) Azure non attachés.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 03/30/2018
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: 0c3e8bb2ff6f3313e851a4253a95a5ad923a8f70
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016214"
---
# <a name="find-and-delete-unattached-azure-managed-and-unmanaged-disks-using-the-azure-cli"></a>Rechercher et supprimer les disques managés et non managés Azure qui ne sont pas attachés à l’aide d’Azure CLI
Par défaut, lorsque vous supprimez une machine virtuelle dans Azure, tous les disques qui sont attachés à cette machine ne sont pas supprimés. Cette fonctionnalité permet d’éviter une perte de données causée par la suppression involontaire de machines virtuelles. Après la suppression d’une machine virtuelle, vous continuez de payer pour les disques non attachés. Cet article explique comment rechercher et supprimer tous les disques non attachés pour réduire les coûts inutiles. 


## <a name="managed-disks-find-and-delete-unattached-disks"></a>Disques managés : Rechercher et supprimer les disques non attachés 

Le script suivant recherche des [disques managés](../managed-disks-overview.md) non attachés en examinant la valeur de la propriété **ManagedBy**. Lorsqu’un disque managé est attaché à une machine virtuelle, la propriété **ManagedBy** contient l’ID de ressource de la machine virtuelle. Lorsqu’un disque managé est non attaché, la propriété **ManagedBy** a la valeur null. Le script examine tous les disques managés dans un abonnement Azure. Lorsque le script localise un disque managé dont la propriété **ManagedBy** a la valeur null, il détermine que le disque n’est pas attaché.

>[!IMPORTANT]
>En premier lieu, exécutez le script en définissant la variable **deleteUnattachedDisks** sur 0. Cette action vous permet de rechercher et d’afficher tous les disques managés non attachés.
>
>Après avoir examiné tous les disques non attachés, exécutez de nouveau le script et définissez la valeur de la variable **deleteUnattachedDisks** sur 1. Cette action vous permet de supprimer tous les disques managés non attachés.
>

```azurecli
# Set deleteUnattachedDisks=1 if you want to delete unattached Managed Disks
# Set deleteUnattachedDisks=0 if you want to see the Id of the unattached Managed Disks
deleteUnattachedDisks=0
unattachedDiskIds=$(az disk list --query '[?managedBy==`null`].[id]' -o tsv)
for id in ${unattachedDiskIds[@]}
do
    if (( $deleteUnattachedDisks == 1 ))
    then

        echo "Deleting unattached Managed Disk with Id: "$id
        az disk delete --ids $id --yes
        echo "Deleted unattached Managed Disk with Id: "$id

    else
        echo $id
    fi
done
```

## <a name="unmanaged-disks-find-and-delete-unattached-disks"></a>Disques non managés : Rechercher et supprimer les disques non attachés 

Les disques non managés sont des fichiers VHD qui sont stockés en tant [qu’objets blob de pages](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-page-blobs) dans des [comptes de stockage Azure](../../storage/common/storage-account-overview.md). Le script suivant recherche les disques non managés non attachés (objets blob de pages) en examinant la valeur de la propriété **LeaseStatus**. Lorsqu’un disque non managé est attaché à une machine virtuelle, la valeur de la propriété **LeaseStatus** est définie sur **Locked**. Lorsqu’un disque non managé est non attaché, la valeur de la propriété **LeaseStatus** est définie sur **Unlocked**. Le script examine tous les disques non managés dans l’ensemble des comptes de stockage Azure d’un abonnement Azure. Lorsque le script localise un disque non managé dont une propriété **LeaseStatus** a la valeur **Unlocked**, il détermine que le disque n’est pas attaché.

>[!IMPORTANT]
>En premier lieu, exécutez le script en définissant la variable **deleteUnattachedVHDs** sur 0. Cette action vous permet de rechercher et d’afficher tous les disques durs virtuels non managés non attachés.
>
>Après avoir examiné tous les disques non attachés, exécutez de nouveau le script et définissez la valeur de la variable **deleteUnattachedVHDs** sur 1. Cette action vous permet de supprimer tous les disques durs virtuels non managés non attachés.
>

```azurecli
# Set deleteUnattachedVHDs=1 if you want to delete unattached VHDs
# Set deleteUnattachedVHDs=0 if you want to see the details of the unattached VHDs
deleteUnattachedVHDs=0
storageAccountIds=$(az storage account list --query [].[id] -o tsv)
for id in ${storageAccountIds[@]}
do
    connectionString=$(az storage account show-connection-string --ids $id --query connectionString -o tsv)
    containers=$(az storage container list --connection-string $connectionString --query [].[name] -o tsv)

    for container in ${containers[@]}
    do 
        
        blobs=$(az storage blob list -c $container --connection-string $connectionString --query "[?properties.blobType=='PageBlob' && ends_with(name,'.vhd')].[name]" -o tsv)
        
        for blob in ${blobs[@]}
        do
            leaseStatus=$(az storage blob show -n $blob -c $container --connection-string $connectionString --query "properties.lease.status" -o tsv)
            
            if [ "$leaseStatus" == "unlocked" ]
            then 

                if (( $deleteUnattachedVHDs == 1 ))
                then 

                    echo "Deleting VHD: "$blob" in container: "$container" in storage account: "$id

                    az storage blob delete --delete-snapshots include  -n $blob -c $container --connection-string $connectionString

                    echo "Deleted VHD: "$blob" in container: "$container" in storage account: "$id
                else
                    echo "StorageAccountId: "$id" container: "$container" VHD: "$blob
                fi

            fi
        done
    done
done 
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Supprimer un compte de stockage](../../storage/common/storage-account-create.md#delete-a-storage-account) .