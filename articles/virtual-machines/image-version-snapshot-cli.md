---
title: CLI - Créer une image à partir d’une capture instantanée ou d’un disque managé dans Shared Image Gallery
description: Découvrez comment créer une image à partir d’une capture instantanée ou d’un disque managé dans Shared Image Gallery à l’aide d’Azure CLI.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 06/30/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: e694630d8bcd7879d9405152c4141fb6e5bad4e2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89297091"
---
# <a name="create-an-image-from-a-managed-disk-or-snapshot-in-a-shared-image-gallery-using-the-azure-cli"></a>Créer une image à partir d’un disque managé ou d’une capture instantanée dans Shared Image Gallery à l’aide d’Azure CLI

Si vous souhaitez migrer une capture instantanée ou un disque managé existants dans Shared Image Gallery, vous pouvez créer une image Shared Image Gallery directement à partir du disque managé ou de la capture instantanée. Une fois que vous avez testé votre nouvelle image, vous pouvez supprimer la capture instantanée ou le disque managé source. Vous pouvez également créer une image à partir d’un disque managé ou d’une capture instantanée dans Shared Image Gallery à l’aide [d’Azure PowerShell](image-version-snapshot-powershell.md).

Les images d’une galerie d’images comportent deux composants, que nous allons créer dans cet exemple :
- Une **définition d’image** transporte des informations sur l’image et ses conditions d’utilisation. Ces informations indiquent, par exemple, si l’image est Windows ou Linux, spécialisée ou généralisée, et comprennent les notes de publication et la mémoire maximale et minimale requise. Il s’agit d’une définition de type d’image. 
- Une **version d’image** est ce qui permet de créer une machine virtuelle lors de l’utilisation d’une galerie d’images partagées. Vous pouvez avoir plusieurs versions d’une image en fonction des besoins de votre environnement. Quand vous créez une machine virtuelle, la version d’image permet de créer des disques pour la machine virtuelle. Les versions d’image peuvent être utilisées plusieurs fois.


## <a name="before-you-begin"></a>Avant de commencer

Pour suivre l’exemple de cet article, vous devez avoir une capture instantanée ou un disque managé. 

Si vous voulez inclure un disque de données, la taille du disque de données ne peut pas être supérieure à 1 To.

Dans cet article, remplacez les noms de ressources si nécessaire.

## <a name="find-the-snapshot-or-managed-disk"></a>Rechercher la capture instantanée ou le disque managé 

Pour voir la liste des captures instantanées disponibles dans un groupe de ressources, utilisez la liste [az snapshot list](/cli/azure/snapshot#az-snapshot-list). 

```azurecli-interactive
az snapshot list --query "[].[name, id]" -o tsv
```

Vous pouvez également utiliser un disque managé au lieu d’une capture instantanée. Pour obtenir un disque managé, utilisez [az disk list](/cli/azure/disk#az-disk-list). 

```azurecli-interactive
az disk list --query "[].[name, id]" -o tsv
```

Une fois que vous avez l’ID de la capture instantanée ou du disque managé, vous pouvez l’affectez à une variable nommée `$source` à utiliser ultérieurement.

Vous pouvez utiliser le même processus pour obtenir tous les disques de données que vous souhaitez inclure dans votre image. Affectez-les à des variables, puis utilisez ces variables ultérieurement lorsque vous créez la version de l’image.


## <a name="find-the-gallery"></a>Trouvez la galerie

Vous aurez besoin d’informations sur la Galerie d’images afin de créer la définition de l’image.

Faites la liste des informations sur les galeries d’images disponibles à l’aide de [az sig list](/cli/azure/sig#az-sig-list). Notez le nom de la galerie du groupe de ressources dans lequel la galerie est utilisée ultérieurement.

```azurecli-interactive 
az sig list -o table
```


## <a name="create-an-image-definition"></a>Créer une définition d’image

Les définitions d’image créent un regroupement logique des images. Elles sont utilisées pour gérer les informations sur l’image. Les noms de définition d’image peuvent contenir des lettres majuscules ou minuscules, des chiffres, des tirets et des points. 

Lors de la définition de votre image, assurez-vous de disposer de toutes les informations correctes. Dans cet exemple, nous supposons que la capture instantanée ou le disque managé sont issus d’une machine virtuelle en cours d’utilisation et qui n’a pas été généralisée. Si le disque managé ou la capture instantanée ont été tirés d’un système d’exploitation généralisé (après l’exécution de Sysprep pour Windows ou [waagent](https://github.com/Azure/WALinuxAgent) `-deprovision` ou `-deprovision+user` pour Linux), remplacez `-OsState` par `generalized`. 

Pour plus d’informations sur les valeurs que vous pouvez spécifier pour une définition d’image, consultez [Définitions d’image](./linux/shared-image-galleries.md#image-definitions).

Créez une définition d’image dans la galerie avec la commande [az sig image-definition create](/cli/azure/sig/image-definition#az-sig-image-definition-create).

Dans cet exemple, la définition d’image se nomme *myImageDefinition* et est destinée à une image de système d’exploitation Linux [spécialisée](./linux/shared-image-galleries.md#generalized-and-specialized-images). Pour créer une définition pour des images utilisant un système d’exploitation Windows, utilisez `--os-type Windows`. 

Dans cet exemple, la galerie est nommée *myGallery*, elle se trouve dans le groupe de ressources *myGalleryRG*, et le nom de la définition d’image est *mImageDefinition*.

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
   --os-state specialized
```


## <a name="create-the-image-version"></a>Créer la version de l’image

Créez une version d’image à l’aide de la commande [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create). 

Les caractères autorisés pour la version d’image sont les nombres et les points. Les nombres doivent être un entier 32 bits. Format: *MajorVersion*.*MinorVersion*.*Patch*.

Dans cet exemple, la version de notre image est *1.0.0* et nous allons créer 1 réplica dans la région *USA Centre Sud* et 1 réplica dans la région *USA Est 2* à l’aide du stockage redondant interzone. Lors du choix des régions cibles pour la réplication, n’oubliez pas que vous devez également inclure la région *source* pour le disque managé ou la capture instantanée en tant que cible pour la réplication.

Transmettez l’ID de la capture instantanée ou du disque managé dans le paramètre `--os-snapshot`.


```azurecli-interactive 
az sig image-version create \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --gallery-image-definition $imageDef \
   --gallery-image-version 1.0.0 \
   --target-regions "southcentralus=1" "eastus2=1=standard_zrs" \
   --replica-count 2 \
   --os-snapshot $source
```

Si vous souhaitez inclure des disques de données dans l’image, vous devez inclure le paramètre de `--data-snapshot-luns` défini sur le numéro d’unité logique et le `--data-snapshots` défini sur l’identifiant du disque de données ou de la capture instantanée.

> [!NOTE]
> Vous devez attendre que la version d’image soit totalement intégrée et répliquée avant de pouvoir utiliser la même image managée pour créer une autre version d’image.
>
> Vous pouvez également stocker tous les réplicas de votre version d’image dans un [stockage redondant interzone](../storage/common/storage-redundancy.md) en ajoutant `--storage-account-type standard_zrs` lorsque vous créez la version d’image.
>

## <a name="next-steps"></a>Étapes suivantes

Créer une machine virtuelle à partir d’une [version image spécialisée](vm-specialized-image-version-cli.md).

Pour savoir comment fournir des informations sur le plan d'achat, consultez [Supply Azure Marketplace purchase plan information when creating images](marketplace-images.md) (Donner des informations sur le plan d'achat de la place de marché Azure lors de la création d’images).
