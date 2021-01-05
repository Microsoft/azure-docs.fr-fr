---
title: Fichier Include
description: Fichier Include
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 04/25/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 32978257c3e209dc78a29c6e8ae0d1c4ae016a5b
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95554911"
---
## <a name="update-resources"></a>Mettre à jour les ressources

Il existe certaines limitations concernant les éléments qui peuvent être mis à jour. Les éléments suivants peuvent être mis à jour : 

Galerie d’images partagées :
- Description

Définition d’image :
- Processeurs virtuels recommandés
- Mémoire recommandée
- Description
- Date de fin de vie

Version d’image :
- Nombre de réplicas régionaux
- Régions cibles
- Exclusion du dernier
- Date de fin de vie

Si vous prévoyez d’ajouter des régions de réplica, ne supprimez pas l’image managée source. En effet, elle est nécessaire pour la réplication de la version de l’image vers d’autres régions. 

Mettez à jour la description d’une galerie à l’aide du paramètre [Update-AzGallery](/powershell/module/az.compute/update-azgallery).

```azurepowershell-interactive
Update-AzGallery `
   -Name $gallery.Name ` 
   -ResourceGroupName $resourceGroup.Name
```

Cet exemple montre comment utiliser [Update-AzGalleryImageDefinition](/powershell/module/az.compute/update-azgalleryimagedefinition) pour mettre à jour la date de fin de vie de notre définition d’image.

```azurepowershell-interactive
Update-AzGalleryImageDefinition `
   -GalleryName $gallery.Name `
   -Name $galleryImage.Name `
   -ResourceGroupName $resourceGroup.Name `
   -EndOfLifeDate 01/01/2030
```

Cet exemple montre comment utiliser le paramètre [Update-AzGalleryImageVersion](/powershell/module/az.compute/update-azgalleryimageversion) pour empêcher que cette version de l’image soit utilisée en tant qu’image *la plus récente*.

```azurepowershell-interactive
Update-AzGalleryImageVersion `
   -GalleryImageDefinitionName $galleryImage.Name `
   -GalleryName $gallery.Name `
   -Name $galleryVersion.Name `
   -ResourceGroupName $resourceGroup.Name `
   -PublishingProfileExcludeFromLatest
```

Cet exemple montre comment utiliser [Update-AzGalleryImageVersion](/powershell/module/az.compute/update-azgalleryimageversion) pour que cette version de l’image soit considérée comme l’image *la plus récente*.

```azurepowershell-interactive
Update-AzGalleryImageVersion `
   -GalleryImageDefinitionName $galleryImage.Name `
   -GalleryName $gallery.Name `
   -Name $galleryVersion.Name `
   -ResourceGroupName $resourceGroup.Name `
   -PublishingProfileExcludeFromLatest:$false
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous supprimez des ressources, vous devez commencer par le dernier élément figurant dans les ressources imbriquées, à savoir la version de l’image. Une fois que les versions sont supprimées, vous pouvez supprimer la définition d’image. Vous ne pouvez pas supprimer la galerie tant que toutes les ressources qu’elle contient n’ont pas été supprimées.

```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$gallery = "myGallery"
$imageDefinition = "myImageDefinition"
$imageVersion = "myImageVersion"

Remove-AzGalleryImageVersion `
   -GalleryImageDefinitionName $imageDefinition `
   -GalleryName $gallery `
   -Name $imageVersion `
   -ResourceGroupName $resourceGroup

Remove-AzGalleryImageDefinition `
   -ResourceGroupName $resourceGroup `
   -GalleryName $gallery `
   -GalleryImageDefinitionName $imageDefinition

Remove-AzGallery `
   -Name $gallery `
   -ResourceGroupName $resourceGroup

Remove-AzResourceGroup -Name $resourceGroup
```