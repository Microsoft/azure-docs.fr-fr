---
title: Créer une image à partir d’une machine virtuelle (préversion)
description: Découvrez comment utiliser Azure PowerShell pour créer une image dans une galerie d’images partagées à partir d’une machine virtuelle existante dans Azure.
author: cynthn
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: imaging
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: 3a7ca8236307bbf8a419d2988e1a6dc1e4c40597
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94964862"
---
# <a name="preview-create-an-image-from-a-vm"></a>Aperçu : Créer une image à partir d’une machine virtuelle

Si vous souhaitez créer plusieurs machines virtuelles identiques à partir d’une machine virtuelle existante, vous pouvez utiliser cette dernière pour créer une image dans une galerie d’images partagées à l’aide d’Azure PowerShell. Vous pouvez également créer une image à partir d’une machine virtuelle à l’aide d’[Azure CLI](image-version-vm-cli.md).

Vous pouvez capturer une image à partir de machines virtuelles [spécialisées et généralisées](./windows/shared-image-galleries.md#generalized-and-specialized-images) en utilisant Azure PowerShell. 

Les images d’une galerie d’images comportent deux composants, que nous allons créer dans cet exemple :
- Une **définition d’image** transporte des informations sur l’image et ses conditions d’utilisation. Ces informations indiquent, par exemple, si l’image est Windows ou Linux, spécialisée ou généralisée, et comprennent les notes de publication et la mémoire maximale et minimale requise. Il s’agit d’une définition de type d’image. 
- Une **version d’image** est ce qui permet de créer une machine virtuelle lors de l’utilisation d’une galerie d’images partagées. Vous pouvez avoir plusieurs versions d’une image en fonction des besoins de votre environnement. Quand vous créez une machine virtuelle, la version d’image permet de créer des disques pour la machine virtuelle. Les versions d’image peuvent être utilisées plusieurs fois.


## <a name="before-you-begin"></a>Avant de commencer

Pour suivre cet article, vous devez disposer d’une galerie d’images partagées et d’une machine virtuelle dans Azure à utiliser comme source. 

Si un disque de données est attaché à la machine virtuelle, la taille du disque de données ne peut pas être supérieure à 1 To.

Dans cet article, remplacez les noms de ressources si nécessaire.


## <a name="get-the-gallery"></a>Obtenir la galerie

Vous pouvez répertorier toutes les galeries et définitions d’images par nom. Les résultats sont au format `gallery\image definition\image version`.

```azurepowershell-interactive
Get-AzResource -ResourceType Microsoft.Compute/galleries | Format-Table
```

Une fois que vous avez trouvé la galerie et les définitions d’images correctes, créez des variables pour celles-ci à utiliser ultérieurement. Cet exemple obtient la Galerie nommée *myGallery* dans le groupe de ressources *myResourceGroup*.

```azurepowershell-interactive
$gallery = Get-AzGallery `
   -Name myGallery `
   -ResourceGroupName myResourceGroup
```

## <a name="get-the-vm"></a>Obtenir la machine virtuelle

Pour voir la liste de machines virtuelles disponibles dans un groupe de ressources, utilisez la commande [Get-AzVM](/powershell/module/az.compute/get-azvm). Une fois le nom de la machine virtuelle et le groupe de ressources connus, vous pouvez réutiliser `Get-AzVM` pour obtenir l’objet de machine virtuelle et le stocker dans une variable afin de l’utiliser ultérieurement. Cet exemple obtient une machine virtuelle nommée *sourceVM* à partir du groupe de ressources « myResourceGroup » et l’assigne à la variable *$sourceVm*. 

```azurepowershell-interactive
$sourceVm = Get-AzVM `
   -Name sourceVM `
   -ResourceGroupName myResourceGroup
```

Il est recommandé d’arrêter\désallouer la machine virtuelle avant de créer une image à l’aide de la commande [Stop-AzVM](/powershell/module/az.compute/stop-azvm).

```azurepowershell-interactive
Stop-AzVM `
   -ResourceGroupName $sourceVm.ResourceGroupName `
   -Name $sourceVm.Name `
   -Force
```

## <a name="create-an-image-definition"></a>Créer une définition d’image 

Les définitions d’image créent un regroupement logique des images. Elles sont utilisées pour gérer les informations sur l’image. Les noms de définition d’image peuvent contenir des lettres majuscules ou minuscules, des chiffres, des tirets et des points. 

Lors de la définition de votre image, assurez-vous de disposer de toutes les informations correctes. Si vous avez généralisé la machine virtuelle (à l’aide de Sysprep pour Windows ou de waagent -deprovision pour Linux), vous devez créer une définition d’image à l’aide de la commande `-OsState generalized`. Si vous n’avez pas généralisé la machine virtuelle, créez une définition d’image à l’aide de la commande `-OsState specialized`.

Pour plus d’informations sur les valeurs que vous pouvez spécifier pour une définition d’image, consultez [Définitions d’image](./windows/shared-image-galleries.md#image-definitions).

Créez la définition d’image à l’aide de [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion). 

Dans cet exemple, la définition d’image se nomme *myImageDefinition* et est destinée à une machine virtuelle spécialisée exécutant Windows. Pour créer une définition pour des images à l’aide de Linux, utilisez `-OsType Linux`. 

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


## <a name="create-an-image-version"></a>Créer une version d’image

Créez une version d’image à l’aide de [New-AzGalleryImageVersion](/powershell/module/az.compute/new-azgalleryimageversion). 

Les caractères autorisés pour la version d’image sont les nombres et les points. Les nombres doivent être un entier 32 bits. Format: *MajorVersion*.*MinorVersion*.*Patch*.

Dans cet exemple, la version d'image, *1.0.0*, elle est répliquée dans les deux centres de données *USA Centre-Ouest* et *USA Centre Sud*. Lors du choix des régions cibles pour la réplication, n’oubliez pas que vous devez également inclure la région *source* en tant que cible pour la réplication.

Pour créer une version d’image à partir de la machine virtuelle, utilisez `$vm.Id.ToString()` comme `-SourceImageId`.

```azurepowershell-interactive
   $region1 = @{Name='South Central US';ReplicaCount=1}
   $region2 = @{Name='East US';ReplicaCount=2}
   $targetRegions = @($region1,$region2)

$job = $imageVersion = New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $imageDefinition.Name`
   -GalleryImageVersionName '1.0.0' `
   -GalleryName $gallery.Name `
   -ResourceGroupName $gallery.ResourceGroupName `
   -Location $gallery.Location `
   -TargetRegion $targetRegions  `
   -SourceImageId $sourceVm.Id.ToString() `
   -PublishingProfileEndOfLifeDate '2020-12-01' `  
   -asJob 
```

La réplication de l'image dans toutes les régions cibles peut nécessiter un certain temps et nous avons donc créé une tâche pour en suivre la progression. Pour afficher la progression de la tâche, entrez `$job.State`.

```azurepowershell-interactive
$job.State
```

> [!NOTE]
> Vous devez attendre que la version d’image soit totalement intégrée et répliquée avant de pouvoir utiliser la même image managée pour créer une autre version d’image.
>
> Vous pouvez également stocker votre image dans le Stockage Premium, en ajoutant `-StorageAccountType Premium_LRS`, ou dans le [stockage redondant interzone](../storage/common/storage-redundancy.md), en ajoutant `-StorageAccountType Standard_ZRS`, quand vous créez la version de l’image.
>

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez vérifié que la nouvelle version de l’image fonctionne correctement, vous pouvez créer une machine virtuelle. Créez une machine virtuelle à partir d’une [version d’image spécialisée](vm-specialized-image-version-powershell.md) ou d’une [version d’image généralisée](vm-generalized-image-version-powershell.md).

Pour savoir comment fournir des informations sur le plan d’achat, consultez [Donner des informations sur le plan d’achat de la Place de marché Azure lors de la création d’images](marketplace-images.md).
