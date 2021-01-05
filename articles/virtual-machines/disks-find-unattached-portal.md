---
title: Identifier les disques Azure non attachés - Portail Azure
description: Comment rechercher à l’aide du portail Azure les disques managés et non managés (disques durs virtuels/objets blob de pages) Azure non attachés.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 06/01/2020
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: 6632d65fa07788e35b24c2f957e713f824f6b091
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87542736"
---
# <a name="find-and-delete-unattached-azure-managed-and-unmanaged-disks---azure-portal"></a>Trouver et supprimer les disques managés et non managés Azure non attachés - Portail Azure

Par défaut, lorsque vous supprimez une machine virtuelle dans Azure, tous les disques qui sont attachés à cette machine ne sont pas supprimés. Ainsi, vous pouvez éviter une perte de données causée par la suppression involontaire de machines virtuelles. Après la suppression d’une machine virtuelle, vous continuez de payer pour les disques non attachés. Cet article explique comment rechercher et supprimer à l’aide du portail Azure tous les disques non attachés pour réduire les coûts inutiles.

## <a name="managed-disks-find-and-delete-unattached-disks"></a>Disques managés : Rechercher et supprimer les disques non attachés

Si vous avez des disques managés non attachés et que vous n’avez plus besoin des données qu’ils contiennent, le processus suivant explique comment les rechercher à partir du portail Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Recherchez et sélectionnez **Disques**.

    Le panneau **Disques** liste tous vos disques. Tout disque contenant « **-**  » dans la colonne **Propriétaire** est un disque non attaché.

    [![Screenshot of the managed disks blade, if a disk has - in the Owner column, it is an unattached disk.](media/disks-find-unattached-portal/managed-disk-unattached-owner.png)](media/disks-find-unattached-portal/managed-disk-owner-unattached.png#lightbox)

1. Sélectionnez le disque non attaché que vous souhaitez supprimer. Le panneau du disque s’ouvre.
1. Dans le panneau du disque, vous pouvez vérifier que l’état du disque est non attaché, puis sélectionner **Supprimer**.

    :::image type="content" source="media/disks-find-unattached-portal/delete-managed-disk-unattached.png" alt-text="Capture d’écran d’un panneau de disque managé individuel. Ce panneau indique « non attaché » dans l’état du disque s’il n’est pas attaché. Vous pouvez supprimer ce disque si vous n’avez plus besoin de conserver ses données":::

## <a name="unmanaged-disks-find-and-delete-unattached-disks"></a>Disques non managés : Rechercher et supprimer les disques non attachés

Les disques non managés sont des fichiers VHD qui sont stockés en tant [qu’objets blob de pages](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-page-blobs) dans des [comptes de stockage Azure](../storage/common/storage-account-overview.md).

Si vous avez des disques non managés qui ne sont pas attachés à une machine virtuelle, que vous n’avez plus besoin des données qu’ils contiennent et que vous souhaitez les supprimer, le processus suivant explique comment le faire à partir du portail Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Recherchez et sélectionnez **Disques (classiques)** .

    La liste de tous vos disques non managés apparaît. Tout disque contenant « **-**  » dans la colonne **Attaché à** est un disque non attaché.

    :::image type="content" source="media/disks-find-unattached-portal/unmanaged-disk-unattached-attached-to.png" alt-text="Capture d’écran d’un panneau de disque managé individuel. Ce panneau indique « non attaché » dans l’état du disque s’il n’est pas attaché. Vous pouvez supprimer ce disque si vous n’avez plus besoin de conserver ses données":::

1. Sélectionnez le disque non attaché que vous souhaitez supprimer. Le panneau du disque s’ouvre.

1. Dans le panneau du disque, vous pouvez vérifier que ce dernier n’est pas attaché, car **Attaché à** contient toujours **-** .

    :::image type="content" source="media/disks-find-unattached-portal/unmanaged-disk-unattached-select-blade.png" alt-text="Capture d’écran d’un panneau de disque managé individuel. Ce panneau indique « non attaché » dans l’état du disque s’il n’est pas attaché. Vous pouvez supprimer ce disque si vous n’avez plus besoin de conserver ses données":::

1. Sélectionnez **Supprimer**.

    :::image type="content" source="media/disks-find-unattached-portal/delete-unmanaged-disk-unattached.png" alt-text="Capture d’écran d’un panneau de disque managé individuel. Ce panneau indique « non attaché » dans l’état du disque s’il n’est pas attaché. Vous pouvez supprimer ce disque si vous n’avez plus besoin de conserver ses données":::

## <a name="next-steps"></a>Étapes suivantes

Si vous souhaitez une méthode automatisée de recherche et de suppression de comptes de stockage non attachés, consultez nos articles sur l’[interface CLI](linux/find-unattached-disks.md) ou [PowerShell](windows/find-unattached-disks.md).

Pour plus d’informations, consultez [Supprimer un compte de stockage](../storage/common/storage-account-create.md#delete-a-storage-account) et [Identify Orphaned Disks Using PowerShell](/archive/blogs/ukplatforms/azure-cost-optimisation-series-identify-orphaned-disks-using-powershell) (Identifier les disques orphelins à l’aide de PowerShell)
