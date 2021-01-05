---
title: Cloner une image managée vers une galerie d’images partagées
description: Découvrez comment utiliser Azure PowerShell pour cloner une image managée vers une version d’image dans une galerie d’images partagées.
author: cynthn
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: imaging
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: 92cae59978b172993c779e9a486ff67d82309800
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349920"
---
# <a name="clone-a-managed-image-to-a-shared-image-gallery-image"></a>Cloner une image managée vers une image de galerie d’images partagées

Si vous souhaitez cloner et déplacer une image managée existante dans Shared Image Gallery, vous pouvez créer directement une image Shared Image Gallery à partir de l’image managée. Une fois que vous avez testé votre nouvelle image, vous pouvez supprimer l’image managée source. Vous pouvez également effectuer une migration depuis une image managée vers une galerie d’images partagées à l’aide d’[Azure CLI](image-version-managed-image-cli.md).

Les images d’une galerie d’images comportent deux composants, que nous allons créer dans cet exemple :
- Une **définition d’image** transporte des informations sur l’image et ses conditions d’utilisation. Ces informations indiquent, par exemple, si l’image est Windows ou Linux, spécialisée ou généralisée, et comprennent les notes de publication et la mémoire maximale et minimale requise. Il s’agit d’une définition de type d’image. 
- Une **version d’image** est ce qui permet de créer une machine virtuelle lors de l’utilisation d’une galerie d’images partagées. Vous pouvez avoir plusieurs versions d’une image en fonction des besoins de votre environnement. Quand vous créez une machine virtuelle, la version d’image permet de créer des disques pour la machine virtuelle. Les versions d’image peuvent être utilisées plusieurs fois.


## <a name="before-you-begin"></a>Avant de commencer

Pour suivre l’exemple de cet article, vous devez avoir une image managée. Si l’image managée contient un disque de données, la taille du disque de données ne peut pas être supérieure à 1 To.

Au cours de cet article, remplacez les noms du groupe de ressources et de la machine virtuelle si nécessaire.

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

Lors de la définition de votre image, assurez-vous de disposer de toutes les informations correctes. Étant donné que les images managées sont toujours généralisées, vous devez définir `-OsState generalized`. 

Pour plus d’informations sur les valeurs que vous pouvez spécifier pour une définition d’image, consultez [Définitions d’image](./windows/shared-image-galleries.md#image-definitions).

Créez la définition d’image à l’aide de [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion). Dans cet exemple, la définition d’image est nommée *myImageDefinition* et est destinée à un système d’exploitation Windows généralisé. Pour créer une définition pour des images utilisant un système d’exploitation Linux, utilisez `-OsType Linux`. 

```azurepowershell-interactive
$imageDefinition = New-AzGalleryImageDefinition `
   -GalleryName $gallery.Name `
   -ResourceGroupName $gallery.ResourceGroupName `
   -Location $gallery.Location `
   -Name 'myImageDefinition' `
   -OsState generalized `
   -OsType Windows `
   -Publisher 'myPublisher' `
   -Offer 'myOffer' `
   -Sku 'mySKU'
```

## <a name="get-the-managed-image"></a>Obtenir l'image managée

Pour afficher une liste d’images disponibles dans un groupe de ressources, utilisez [Get-AzImage](/powershell/module/az.compute/get-azimage). Une fois le nom de l'image et son groupe de ressources connus, vous pouvez réutiliser `Get-AzImage` pour obtenir l’objet image et le stocker dans une variable à des fins d'utilisation ultérieure. Cet exemple permet d'obtenir une image nommée *myImage* à partir du groupe de ressources « myResourceGroup » et l’assigne à la variable *$managedImage*. 

```azurepowershell-interactive
$managedImage = Get-AzImage `
   -ImageName myImage `
   -ResourceGroupName myResourceGroup
```


## <a name="create-an-image-version"></a>Créer une version d’image

Créez une version d’image à partir d’une image managée à l’aide de [New-AzGalleryImageVersion](/powershell/module/az.compute/new-azgalleryimageversion). 

Les caractères autorisés pour la version d’image sont les nombres et les points. Les nombres doivent être un entier 32 bits. Format: *MajorVersion*.*MinorVersion*.*Patch*.

Dans cet exemple, la version d'image, *1.0.0*, elle est répliquée dans les deux centres de données *USA Centre-Ouest* et *USA Centre Sud*. Lors du choix des régions cibles pour la réplication, n’oubliez pas que vous devez également inclure la région *source* en tant que cible pour la réplication. 


```azurepowershell-interactive
$region1 = @{Name='South Central US';ReplicaCount=1}
$region2 = @{Name='West Central US';ReplicaCount=2}
$targetRegions = @($region1,$region2)
$job = $imageVersion = New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $imageDefinition.Name `
   -GalleryImageVersionName '1.0.0' `
   -GalleryName $gallery.Name `
   -ResourceGroupName $imageDefinition.ResourceGroupName `
   -Location $imageDefinition.Location `
   -TargetRegion $targetRegions  `
   -SourceImageId $managedImage.Id.ToString() `
   -PublishingProfileEndOfLifeDate '2020-12-31' `
   -asJob 
```

La réplication de l'image dans toutes les régions cibles peut nécessiter un certain temps et nous avons donc créé une tâche pour en suivre la progression. Pour afficher la progression, entrez `$job.State`.

```azurepowershell-interactive
$job.State
```


> [!NOTE]
> Vous devez attendre que la version d’image soit totalement intégrée et répliquée avant de pouvoir utiliser la même image managée pour créer une autre version d’image. 
>
> Vous pouvez également stocker votre image dans le Stockage Premium, en ajoutant `-StorageAccountType Premium_LRS`, ou dans le [stockage redondant interzone](../storage/common/storage-redundancy.md), en ajoutant `-StorageAccountType Standard_ZRS`, quand vous créez la version de l’image.
>

## <a name="delete-the-managed-image"></a>Supprimer l’image managée

Une fois que vous avez vérifié que la nouvelle version de l’image fonctionne correctement, vous pouvez supprimer l’image managée.

```azurepowershell-interactive
Remove-AzImage `
   -ImageName $managedImage.Name `
   -ResourceGroupName $managedImage.ResourceGroupName
```

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez vérifié que la réplication est terminée, vous pouvez créer une machine virtuelle à partir de l’[image généralisée](vm-generalized-image-version-powershell.md).

Pour savoir comment fournir des informations sur le plan d'achat, consultez [Supply Azure Marketplace purchase plan information when creating images](marketplace-images.md) (Donner des informations sur le plan d'achat de la place de marché Azure lors de la création d’images).
