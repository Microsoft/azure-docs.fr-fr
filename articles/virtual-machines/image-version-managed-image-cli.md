---
title: Clonage d’une image managée dans une version d’image avec Azure CLI
description: Découvrez comment cloner une image managée vers une version d’image dans Shared Image Gallery avec Azure CLI.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.custom: devx-track-azurecli
ms.openlocfilehash: 0e53bebf9cdb8c0fc084d04550c7444c1c01be50
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352862"
---
# <a name="clone-a-managed-image-to-an-image-version-using-the-azure-cli"></a>Clonage d’une image managée dans une version d’image avec Azure CLI
Si vous souhaitez cloner une image managée existante dans Shared Image Gallery, vous pouvez créer directement une image Shared Image Gallery à partir de l’image managée. Une fois que vous avez testé votre nouvelle image, vous pouvez supprimer l’image managée source. Vous pouvez également effectuer une migration depuis une image managée vers une galerie d’images partagées à l’aide de [PowerShell](image-version-managed-image-powershell.md).

Les images d’une galerie d’images comportent deux composants, que nous allons créer dans cet exemple :
- Une **définition d’image** transporte des informations sur l’image et ses conditions d’utilisation. Ces informations indiquent, par exemple, si l’image est Windows ou Linux, spécialisée ou généralisée, et comprennent les notes de publication et la mémoire maximale et minimale requise. Il s’agit d’une définition de type d’image. 
- Une **version d’image** est ce qui permet de créer une machine virtuelle lors de l’utilisation d’une galerie d’images partagées. Vous pouvez avoir plusieurs versions d’une image en fonction des besoins de votre environnement. Quand vous créez une machine virtuelle, la version d’image permet de créer des disques pour la machine virtuelle. Les versions d’image peuvent être utilisées plusieurs fois.


## <a name="before-you-begin"></a>Avant de commencer

Pour suivre cet article, vous devez disposer d’une [galerie d’images partagées](shared-images-cli.md). 

Pour suivre l’exemple de cet article, vous devez avoir l’image managée d’une machine virtuelle généralisée. Pour plus d’informations, consultez [Capturer une image managée](./linux/capture-image.md). Si l’image managée contient un disque de données, la taille du disque de données ne peut pas être supérieure à 1 To.

Au cours de cet article, remplacez les noms du groupe de ressources et de la machine virtuelle si nécessaire.



## <a name="create-an-image-definition"></a>Créer une définition d’image

Les images managées étant toujours des images généralisées, vous allez créer une définition d’image en utilisant `--os-state generalized` pour une image généralisée.

Les noms de définition d’image peuvent contenir des lettres majuscules ou minuscules, des chiffres, des tirets et des points. 

Pour plus d’informations sur les valeurs que vous pouvez spécifier pour une définition d’image, consultez [Définitions d’image](./linux/shared-image-galleries.md#image-definitions).

Créez une définition d’image dans la galerie avec la commande [az sig image-definition create](/cli/azure/sig/image-definition#az-sig-image-definition-create).

Dans cet exemple, la définition d’image est nommée *myImageDefinition* et est destinée à une image de système d’exploitation Linux [généralisée](./linux/shared-image-galleries.md#generalized-and-specialized-images). Pour créer une définition pour des images utilisant un système d’exploitation Windows, utilisez `--os-type Windows`. 

```azurecli-interactive 
resourceGroup=myGalleryRG
gallery=myGallery
imageDef=myImageDefinition
az sig image-definition create \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --gallery-image-definition $imageDef \
   --publisher myPublisher \
   --offer myOffer \
   --sku mySKU \
   --os-type Linux \
   --os-state generalized
```


## <a name="create-the-image-version"></a>Créer la version de l’image

Créez des versions avec la commande [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create). L’ID de l’image managée sert de base de référence pour la création de la version d’image. Vous pouvez utiliser [az image list](/cli/azure/image?view#az-image-list) pour obtenir les ID de vos images. 

```azurecli-interactive
az image list --query "[].[name, id]" -o tsv
```

Les caractères autorisés pour la version d’image sont les nombres et les points. Les nombres doivent être un entier 32 bits. Format: *MajorVersion*.*MinorVersion*.*Patch*.

Dans cet exemple, la version de notre image est *1.0.0* et nous allons créer 1 réplica dans la région *USA Centre Sud* et 1 réplica dans la région *USA Est 2* à l’aide du stockage redondant interzone. Lors du choix des régions cibles pour la réplication, n’oubliez pas que vous devez également inclure la région *source* en tant que cible pour la réplication.

Transmettez l’ID de l’image managée dans le paramètre `--managed-image`.


```azurecli-interactive 
imageID="/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myImage"
az sig image-version create \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --gallery-image-definition $imageDef \
   --gallery-image-version 1.0.0 \
   --target-regions "southcentralus=1" "eastus2=1=standard_zrs" \
   --replica-count 2 \
   --managed-image $imageID
```

> [!NOTE]
> Vous devez attendre que la version d’image soit totalement intégrée et répliquée avant de pouvoir utiliser la même image managée pour créer une autre version d’image.
>
> Vous pouvez également stocker tous les réplicas de votre version d’image dans un [stockage redondant interzone](../storage/common/storage-redundancy.md) en ajoutant `--storage-account-type standard_zrs` lorsque vous créez la version d’image.
>

## <a name="next-steps"></a>Étapes suivantes

Créez une machine virtuelle à partir d’une [version d’image généralisée](vm-generalized-image-version-cli.md).

Pour savoir comment fournir des informations sur le plan d’achat, consultez [Donner des informations sur le plan d’achat de la Place de marché Azure lors de la création d’images](marketplace-images.md).
