---
title: Copiez une image à partir d’une autre galerie en utilisant PowerShell
description: Copiez une image à partir d’une autre galerie en utilisant Azure PowerShell.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: 2f26716a4d04b574bf393f502758a725948419da
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89299947"
---
# <a name="copy-an-image-from-another-gallery-using-powershell"></a>Copiez une image à partir d’une autre galerie en utilisant PowerShell

Si vous avez plusieurs galeries dans votre organisation, vous pouvez créer des images à partir d’images stockées dans d’autres galeries. Par exemple, vous pouvez avoir une galerie de développement et de test pour créer et tester de nouvelles images. Lorsqu’elles sont prêtes à être utilisées en production, vous pouvez les copier dans une galerie de production à l’aide de cet exemple. Vous pouvez également créer une image à partir d’une image dans une autre galerie à l’aide d’[Azure CLI](image-version-another-gallery-cli.md).


## <a name="before-you-begin"></a>Avant de commencer

Pour suivre cet article, vous devez disposer d’une galerie source, d’une définition d’image et d’une version d’image existantes. Vous devez également disposer d’une bibliothèque de destination. 

La version de l’image source doit être répliquée vers la région où se trouve votre bibliothèque de destination. 

Nous allons créer de nouvelles définition d’image et version d’image dans votre bibliothèque de destination.


Dans cet article, remplacez les noms de ressources si nécessaire.


## <a name="get-the-source-image"></a>Obtenir l’image source 

Vous aurez besoin d’informations de la définition de l’image source afin de pouvoir en créer une copie dans votre bibliothèque de destination.

Répertoriez les informations sur les galeries, définitions d’images et versions d’images existantes à l’aide de la cmdlet [Get-AzResource](/powershell/module/az.resources/get-azresource).

Les résultats sont au format `gallery\image definition\image version`.

```azurepowershell-interactive
Get-AzResource `
   -ResourceType Microsoft.Compute/galleries/images/versions | `
   Format-Table -Property Name,ResourceGroupName
```

Une fois que vous disposez de toutes les informations dont vous avez besoin, vous pouvez obtenir l’ID de la version de l’image source à l’aide de la cmdlet [Get-AzGalleryImageVersion](/powershell/module/az.compute/get-azgalleryimageversion). Dans cet exemple, nous obtenons la version d’image `1.0.0`, de la définition `myImageDefinition`, dans la galerie source `myGallery`, dans le groupe de ressources `myResourceGroup`.

```azurepowershell-interactive
$sourceImgVer = Get-AzGalleryImageVersion `
   -GalleryImageDefinitionName myImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myResourceGroup `
   -Name 1.0.0
```


## <a name="create-the-image-definition"></a>Créer la définition d’image 

Vous devez créer une définition d’image correspondant à la définition d’image de votre source. Vous pouvez voir toutes les informations dont vous avez besoin pour recréer la définition d’image à l’aide de la cmdlet [Get-AzGalleryImageDefinition](/powershell/module/az.compute/get-azgalleryimagedefinition).

```azurepowershell-interactive
Get-AzGalleryImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myResourceGroup `
   -Name myImageDefinition
```


Vous obtenez un résultat semblable à ce qui suit :

```output
{
  "description": null,
  "disallowed": null,
  "endOfLifeDate": null,
  "eula": null,
  "hyperVgeneration": "V1",
  "id": "/subscriptions/1111abcd-1a23-4b45-67g7-1234567de123/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition",
  "identifier": {
    "offer": "myOffer",
    "publisher": "myPublisher",
    "sku": "mySKU"
  },
  "location": "eastus",
  "name": "myImageDefinition",
  "osState": "Specialized",
  "osType": "Windows",
  "privacyStatementUri": null,
  "provisioningState": "Succeeded",
  "purchasePlan": null,
  "recommended": null,
  "releaseNoteUri": null,
  "resourceGroup": "myGalleryRG",
  "tags": null,
  "type": "Microsoft.Compute/galleries/images"
}
```

Créez une nouvelle définition d’image dans votre bibliothèque de destination à l’aide de la cmdlet [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion) et des informations de la sortie ci-dessus.


Dans cet exemple, la définition d’image est nommée *myDestinationImgDef* dans la galerie nommée *myDestinationGallery*.


```azurepowershell-interactive
$destinationImgDef  = New-AzGalleryImageDefinition `
   -GalleryName myDestinationGallery `
   -ResourceGroupName myDestinationRG `
   -Location WestUS `
   -Name 'myDestinationImgDef' `
   -OsState specialized `
   -OsType Windows `
   -HyperVGeneration v1 `
   -Publisher 'myPublisher' `
   -Offer 'myOffer' `
   -Sku 'mySKU'
```


## <a name="create-the-image-version"></a>Créer la version de l’image

Créez une version d’image à l’aide de la cmdlet [New-AzGalleryImageVersion](/powershell/module/az.compute/new-azgalleryimageversion). Vous devez transmettre l’ID de l’image source dans le paramètre `--managed-image` pour créer la version d’image dans votre bibliothèque de destination. 

Les caractères autorisés pour la version d’image sont les nombres et les points. Les nombres doivent être un entier 32 bits. Format: *MajorVersion*.*MinorVersion*.*Patch*.

Dans cet exemple, la bibliothèque de destination est nommée *myDestinationGallery*, dans le groupe de ressources *myDestinationRG*, dans l’emplacement *USA Ouest*. La version de notre image est *1.0.0*, et nous allons créer 1 réplica dans la région *USA Centre Sud* et 2 réplicas dans la région *USA Ouest*. 


```azurepowershell-interactive
$region1 = @{Name='South Central US';ReplicaCount=1}
$region2 = @{Name='West US';ReplicaCount=2}
$targetRegions = @($region1,$region2)

$job = $imageVersion = New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $destinationImgDef.Name`
   -GalleryImageVersionName '1.0.0' `
   -GalleryName myDestinationGallery `
   -ResourceGroupName myDestinationRG `
   -Location WestUS `
   -TargetRegion $targetRegions  `
   -Source $sourceImgVer.Id.ToString() `
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

Créez une machine virtuelle à partir d’une version d’image [généralisée](vm-generalized-image-version-powershell.md) ou [spécialisée](vm-specialized-image-version-powershell.md).

[Générateur d’image Azure (préversion)](./linux/image-builder-overview.md) permet d’automatiser la création d’une version de l’image. Vous pouvez même l’utiliser pour mettre à jour et [créer une nouvelle version de l’image à partir d’une image existante](./linux/image-builder-gallery-update-image-version.md). 

Pour savoir comment fournir des informations sur le plan d'achat, consultez [Supply Azure Marketplace purchase plan information when creating images](marketplace-images.md) (Donner des informations sur le plan d'achat de la place de marché Azure lors de la création d’images).
