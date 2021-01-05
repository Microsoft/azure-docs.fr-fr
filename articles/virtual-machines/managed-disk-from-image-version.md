---
title: Créer un disque géré à partir d’une version d’image
description: Créez un disque géré à partir d’une version d’image dans une galerie d’images partagées.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 10/27/2020
ms.author: cynthn
ms.reviewer: olayemio
ms.openlocfilehash: 40d3ff736194c4634b949af52ee7b09db20dd06d
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026129"
---
# <a name="create-a-managed-disk-from-an-image-version"></a>Créer un disque géré à partir d’une version d’image

Si nécessaire, vous pouvez exporter le système d’exploitation ou un disque de données unique à partir d’une version d’image en tant que disque géré à partir d’une version d’image stockée dans une galerie d’images partagées.


## <a name="cli"></a>Interface de ligne de commande

Listez les versions d’image dans une galerie avec [az sig image-version list](/cli/azure/sig/image-version#az_sig_image_version_list). Dans cet exemple, nous recherchons toutes les versions d’image qui font partie de la définition d’image *myImageDefinition* dans la galerie d’images *myGallery*.

```azurecli-interactive
az sig image-version list \
   --resource-group myResourceGroup\
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   -o table
```

Affectez à la variable `source` l’ID de la version de l’image, puis utilisez [az disk create](/cli/azure/disk?view=azure-cli-latest#az_disk_create) pour créer le disque géré. 

Dans cet exemple, nous exportons le disque du système d’exploitation de la version d’image pour créer un disque géré nommé *myManagedOSDisk*, dans la région *EastUS*, dans un groupe de ressources nommé *myResourceGroup*. 

```azurecli-interactive
source="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<galleryImageDefinition>/versions/<imageVersion>"

az disk create --resource-group myResourceGroup --location EastUS --name myManagedOSDisk --gallery-image-reference $source 
```



Si vous souhaitez exporter un disque de données à partir de la version d’image, ajoutez `--gallery-image-reference-lun` pour spécifier l’emplacement LUN du disque de données à exporter. 

Dans cet exemple, nous exportons le disque de données situé sur le LUN 0 de la version d’image pour créer un disque géré nommé *myManagedDataDisk*, dans la région *EastUS*, dans un groupe de ressources nommé *myResourceGroup*. 

```azurecli-interactive
source="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<galleryImageDefinition>/versions/<imageVersion>"

az disk create --resource-group myResourceGroup --location EastUS --name myManagedDataDisk --gallery-image-reference $source --gallery-image-reference-lun 0
``` 

## <a name="powershell"></a>PowerShell

Listez les versions d’image dans une galerie avec [Get-AzResource](/powershell/module/az.resources/get-azresource). 

```azurepowershell-interactive
Get-AzResource `
   -ResourceType Microsoft.Compute/galleries/images/versions | `
   Format-Table -Property Name,ResourceId,ResourceGroupName
```

Une fois que vous disposez de toutes les informations dont vous avez besoin, vous pouvez utiliser [Get-AzGalleryImageVersion](/powershell/module/az.compute/get-azgalleryimageversion) pour obtenir la version de l’image source que vous souhaitez utiliser et l’affecter à une variable. Dans cet exemple, nous obtenons la version d’image `1.0.0`, de la définition `myImageDefinition`, dans la galerie source `myGallery`, dans le groupe de ressources `myResourceGroup`.

```azurepowershell-interactive
$sourceImgVer = Get-AzGalleryImageVersion `
   -GalleryImageDefinitionName myImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myResourceGroup `
   -Name 1.0.0
```

Après avoir défini la variable `source` sur l’ID de la version d’image, utilisez [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig) pour créer une configuration de disque, et [New-AzDisk](/powershell/module/az.compute/new-azdisk) pour créer le disque. 

Dans cet exemple, nous exportons le disque du système d’exploitation de la version d’image pour créer un disque géré nommé *myManagedOSDisk*, dans la région *EastUS*, dans un groupe de ressources nommé *myResourceGroup*. 

Créez une configuration de disque.
```azurepowershell-interactive
$diskConfig = New-AzDiskConfig `
   -Location EastUS `
   -CreateOption FromImage `
   -GalleryImageReference @{Id = $sourceImgVer.Id}
```

Créez le disque.

```azurepowershell-interactive
New-AzDisk -Disk $diskConfig `
   -ResourceGroupName myResourceGroup `
   -DiskName myManagedOSDisk
```

Si vous souhaitez exporter un disque de données sur la version d’image, ajoutez un LUN à la configuration de disque pour spécifier l’emplacement LUN du disque de données à exporter. 

Dans cet exemple, nous exportons le disque de données situé sur le LUN 0 de la version d’image pour créer un disque géré nommé *myManagedDataDisk*, dans la région *EastUS*, dans un groupe de ressources nommé *myResourceGroup*. 

Créez une configuration de disque.
```azurepowershell-interactive
$diskConfig = New-AzDiskConfig `
   -Location EastUS `
   -CreateOption FromImage `
   -GalleryImageReference @{Id = $sourceImgVer.Id; Lun=0}
```

Créez le disque.

```azurepowershell-interactive
New-AzDisk -Disk $diskConfig `
   -ResourceGroupName myResourceGroup `
   -DiskName myManagedDataDisk
```

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également créer une version d’image à partir d’un disque géré à l’aide d’[Azure CLI](image-version-managed-image-cli.md) ou de [PowerShell](image-version-managed-image-powershell.md).


