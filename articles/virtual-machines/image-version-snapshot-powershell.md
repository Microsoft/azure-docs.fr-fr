---
title: PowerShell – créer une image à partir d’une capture instantanée ou d’un disque managé dans Shared Image Gallery
description: Découvrez comment créer une image à partir d’une capture instantanée ou d’un disque managé dans Shared Image Gallery à l’aide de PowerShell.
author: cynthn
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: imaging
ms.workload: infrastructure
ms.date: 06/30/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: 2ebff0d86c27bcdbc11d23e18116b33b4ea838a6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89300253"
---
# <a name="create-an-image-from-a-managed-disk-or-snapshot-in-a-shared-image-gallery-using-powershell"></a>Créez une image à partir d’un disque managé ou d’une capture instantanée dans Shared Image Gallery à l’aide de PowerShell

Si vous souhaitez migrer une capture instantanée ou un disque managé existants dans Shared Image Gallery, vous pouvez créer une image Shared Image Gallery directement à partir du disque managé ou de la capture instantanée. Une fois que vous avez testé votre nouvelle image, vous pouvez supprimer la capture instantanée ou le disque managé source. Vous pouvez également créer une image à partir d’un disque managé ou d’une capture instantanée dans Shared Image Gallery à l’aide d’[Azure CLI](image-version-snapshot-cli.md).

Les images d’une galerie d’images comportent deux composants, que nous allons créer dans cet exemple :
- Une **définition d’image** transporte des informations sur l’image et ses conditions d’utilisation. Ces informations indiquent, par exemple, si l’image est Windows ou Linux, spécialisée ou généralisée, et comprennent les notes de publication et la mémoire maximale et minimale requise. Il s’agit d’une définition de type d’image. 
- Une **version d’image** est ce qui permet de créer une machine virtuelle lors de l’utilisation d’une galerie d’images partagées. Vous pouvez avoir plusieurs versions d’une image en fonction des besoins de votre environnement. Quand vous créez une machine virtuelle, la version d’image permet de créer des disques pour la machine virtuelle. Les versions d’image peuvent être utilisées plusieurs fois.


## <a name="before-you-begin"></a>Avant de commencer

Pour suivre l’exemple de cet article, vous devez avoir une capture instantanée ou un disque managé. 

Si vous voulez inclure un disque de données, la taille du disque de données ne peut pas être supérieure à 1 To.

Dans cet article, remplacez les noms de ressources si nécessaire.


## <a name="get-the-snapshot-or-managed-disk"></a>Obtenir la capture instantanée ou le disque managé

Pour voir la liste des captures instantanées disponibles dans un groupe de ressources, utilisez la liste [Get-AzSnapshot](/powershell/module/az.compute/get-azsnapshot). 

```azurepowershell-interactive
get-azsnapshot | Format-Table -Property Name,ResourceGroupName
```

Une fois le nom de la capture instantanée et le groupe de ressources connus, vous pouvez réutiliser `Get-AzSnapshot` pour obtenir l’objet de capture instantanée et le stocker dans une variable afin de l’utiliser ultérieurement. Cet exemple obtient une capture instantanée nommée *mySnapshot* à partir du groupe de ressources « myResourceGroup » et l’assigne à la variable *$source*. 

```azurepowershell-interactive
$source = Get-AzSnapshot `
   -SnapshotName mySnapshot `
   -ResourceGroupName myResourceGroup
```

Vous pouvez également utiliser un disque managé au lieu d’une capture instantanée. Pour obtenir un disque managé, utilisez [Get-AzDisk](/powershell/module/az.compute/get-azdisk). 

```azurepowershell-interactive
Get-AzDisk | Format-Table -Property Name,ResourceGroupName
```

Puis, récupérez le disque managé et attribuez-le à la variable `$source`.

```azurepowershell-interactive
$source = Get-AzDisk `
   -Name myDisk
   -ResourceGroupName myResourceGroup
```

Vous pouvez utiliser les mêmes cmdlets pour obtenir tous les disques de données que vous souhaitez inclure dans votre image. Affectez-les à des variables, puis utilisez ces variables ultérieurement lorsque vous créez la version de l’image.


## <a name="get-the-gallery"></a>Obtenir la galerie

Vous pouvez répertorier toutes les galeries et définitions d’images par nom. Les résultats sont au format `gallery\image definition\image version`.

```azurepowershell-interactive
Get-AzResource -ResourceType Microsoft.Compute/galleries | Format-Table
```

Une fois que vous avez trouvé la galerie de droite, créez une variable à utiliser ultérieurement. Cet exemple obtient la galerie nommée *myGallery* dans le groupe de ressources *myResourceGroup*.

```azurepowershell-interactive
$gallery = Get-AzGallery `
   -Name myGallery `
   -ResourceGroupName myResourceGroup
```


## <a name="create-an-image-definition"></a>Créer une définition d’image 

Les définitions d’image créent un regroupement logique des images. Elles sont utilisées pour gérer les informations sur l’image. Les noms de définition d’image peuvent contenir des lettres majuscules ou minuscules, des chiffres, des tirets et des points. 

Lors de la définition de votre image, assurez-vous de disposer de toutes les informations correctes. Dans cet exemple, nous supposons que la capture instantanée ou le disque managé sont issus d’une machine virtuelle en cours d’utilisation et qui n’a pas été généralisée. Si le disque managé ou la capture instantanée ont été tirés d’un système d’exploitation généralisé (après l’exécution de Sysprep pour Windows ou [waagent](https://github.com/Azure/WALinuxAgent) `-deprovision` ou `-deprovision+user` pour Linux), remplacez `-OsState` par `generalized`. 

Pour plus d’informations sur les valeurs que vous pouvez spécifier pour une définition d’image, consultez [Définitions d’image](./windows/shared-image-galleries.md#image-definitions).

Créez la définition d’image à l’aide de [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion). Dans cet exemple, la définition d’image est nommée *myImageDefinition* et est destinée à un système d’exploitation Windows spécialisé. Pour créer une définition pour des images utilisant un système d’exploitation Linux, utilisez `-OsType Linux`. 

```azurepowershell-interactive
$imageDefinition = New-AzGalleryImageDefinition `
   -GalleryName $gallery.Name `
   -ResourceGroupName $gallery.ResourceGroupName `
   -Location $gallery.Location `
   -Name 'myImageDefinition' `
   -OsState specialized `
   -OsType Windows `
   -Publisher 'myPublisher' `
   -Offer 'myOffer' `
   -Sku 'mySKU'
```

### <a name="purchase-plan-information"></a>Informations sur le plan d’achat

Dans certains cas, vous devez transmettre des informations sur le plan d’achat lors de la création d’une machine virtuelle à partir d’une image basée sur une image de la place de marché Azure. Dans ces cas-là, nous vous recommandons d’inclure les informations du plan d’achat dans la définition de l’image. Dans ce cas, consultez [Fournir des informations sur le plan d’achat de la place de marché Azure lors de la création d’images](marketplace-images.md).


## <a name="create-an-image-version"></a>Créer une version d’image

Créez une version d’image à partir de la capture instantanée à l’aide de [New-AzGalleryImageVersion](/powershell/module/az.compute/new-azgalleryimageversion). 

Les caractères autorisés pour la version d’image sont les nombres et les points. Les nombres doivent être un entier 32 bits. Format: *MajorVersion*.*MinorVersion*.*Patch*.

Si vous souhaitez que votre image contienne un disque de données, en plus du disque du système d’exploitation, ajoutez le paramètre `-DataDiskImage` et définissez-le sur l’ID du disque managé ou de la capture instantanée du disque de données.

Dans cet exemple, la version d'image, *1.0.0*, elle est répliquée dans les deux centres de données *USA Centre-Ouest* et *USA Centre Sud*. Lors du choix des régions cibles pour la réplication, n’oubliez pas que vous devez également inclure la région *source* en tant que cible pour la réplication.


```azurepowershell-interactive
$region1 = @{Name='South Central US';ReplicaCount=1}
$region2 = @{Name='West Central US';ReplicaCount=2}
$targetRegions = @($region1,$region2)
$job = $imageVersion = New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $imageDefinition.Name `
   -GalleryImageVersionName '1.0.0' `
   -GalleryName $gallery.Name `
   -ResourceGroupName $gallery.ResourceGroupName `
   -Location $gallery.Location `
   -TargetRegion $targetRegions  `
   -OSDiskImage @{Source = @{Id=$source.Id}; HostCaching = "ReadOnly" } `
   -PublishingProfileEndOfLifeDate '2025-01-01' `
   -asJob 
```

La réplication de l'image dans toutes les régions cibles peut nécessiter un certain temps et nous avons donc créé une tâche pour en suivre la progression. Pour afficher la progression de la tâche, entrez `$job.State`.

```azurepowershell-interactive
$job.State
```

> [!NOTE]
> Vous devez attendre que la version d’image soit totalement intégrée et répliquée avant de pouvoir utiliser la même capture instantanée pour créer une autre version d’image. 
>
> Vous pouvez également stocker votre version d’image dans un [stockage redondant interzone](../storage/common/storage-redundancy.md) en ajoutant `-StorageAccountType Standard_ZRS` lorsque vous créez la version d’image.
>

## <a name="delete-the-source"></a>Supprimer la source

Une fois que vous avez vérifié que la nouvelle version de l’image fonctionne correctement, vous pouvez supprimer la source de l’image à l’aide de [Remove-AzSnapshot](/powershell/module/Az.Compute/Remove-AzSnapshot) ou [Remove-AzDisk](/powershell/module/az.compute/remove-azdisk).


## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez vérifié que la réplication est terminée, vous pouvez créer une machine virtuelle à partir de l’[image spécialisée](vm-specialized-image-version-powershell.md).

Pour savoir comment fournir des informations sur le plan d'achat, consultez [Supply Azure Marketplace purchase plan information when creating images](marketplace-images.md) (Donner des informations sur le plan d'achat de la place de marché Azure lors de la création d’images).
