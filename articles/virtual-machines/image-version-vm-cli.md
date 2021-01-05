---
title: Créer une image depuis une machine virtuelle
description: Découvrez comment créer une image dans une galerie d’images partagées à partir d’une machine virtuelle dans Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/01/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.custom: devx-track-azurecli
ms.openlocfilehash: fa4a649115f8e89c27f435888b682b7de36e9894
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87533918"
---
# <a name="create-an-image-version-from-a-vm-in-azure-using-the-azure-cli"></a>Créer une version d’image à partir d’une machine virtuelle dans Azure à l’aide d’Azure CLI

Si vous souhaitez créer plusieurs machines virtuelles identiques à partir d’une machine virtuelle existante, vous pouvez utiliser cette dernière pour créer une image dans une galerie d’images partagées à l’aide d’Azure CLI. Vous pouvez également créer une image à partir d’une machine virtuelle à l’aide d’[Azure PowerShell](image-version-vm-powershell.md).

Une **version d’image** est ce qui vous permet de créer une machine virtuelle quand vous utilisez une galerie d’images partagées. Vous pouvez avoir plusieurs versions d’une image en fonction des besoins de votre environnement. Quand vous utilisez une version d’image pour créer une machine virtuelle, la version d’image permet de créer des disques pour la nouvelle machine virtuelle. Les versions d’image peuvent être utilisées plusieurs fois.


## <a name="before-you-begin"></a>Avant de commencer

Pour suivre cet article, vous devez disposer d’une galerie d’images partagées. 

Vous devez également disposer d’une machine virtuelle dans Azure, dans la même région que votre galerie. 

Si un disque de données est attaché à la machine virtuelle, la taille du disque de données ne peut pas être supérieure à 1 To.

Au cours de cet article, remplacez les noms de ressources si nécessaire.

## <a name="get-information-about-the-vm"></a>Obtenir des informations sur la machine virtuelle

Vous pouvez obtenir la liste des machines virtuelles disponibles à l’aide de la commande [az vm list](/cli/azure/vm#az-vm-list). 

```azurecli-interactive
az vm list --output table
```

Une fois que vous connaissez le nom de la machine virtuelle et le groupe de ressources dans lequel elle se trouve, récupérez l’ID de la machine virtuelle à l’aide de la commande [az vm get-instance-view](/cli/azure/vm#az-vm-get-instance-view). 

```azurecli-interactive
az vm get-instance-view -g MyResourceGroup -n MyVm --query id
```


## <a name="create-an-image-definition"></a>Créer une définition d’image

Les définitions d’image créent un regroupement logique des images. Elles sont utilisées pour gérer les informations sur les versions d’image créées au sein de celles-ci. 

Les noms de définition d’image peuvent contenir des lettres majuscules ou minuscules, des chiffres, des tirets et des points. 

Assurez-vous que le type de votre définition d’image est approprié. Si vous avez généralisé la machine virtuelle (à l’aide de Sysprep pour Windows ou de waagent -deprovision pour Linux), vous devez créer une définition d’image généralisée à l’aide de la commande `--os-state generalized`. Si vous souhaitez utiliser la machine virtuelle sans supprimer de comptes d’utilisateur, créez une définition d’image spécialisée à l’aide de la commande `--os-state specialized`.

Pour plus d’informations sur les valeurs que vous pouvez spécifier pour une définition d’image, consultez [Définitions d’image](./linux/shared-image-galleries.md#image-definitions).

Créez une définition d’image dans la galerie avec la commande [az sig image-definition create](/cli/azure/sig/image-definition#az-sig-image-definition-create).

Dans cet exemple, la définition d’image est nommée *myImageDefinition* et est destinée à une image de système d’exploitation Linux [spécialisée](./linux/shared-image-galleries.md#generalized-and-specialized-images). Pour créer une définition pour des images utilisant un système d’exploitation Windows, utilisez `--os-type Windows`. 

```azurecli-interactive 
az sig image-definition create \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --publisher myPublisher \
   --offer myOffer \
   --sku mySKU \
   --os-type Linux \
   --os-state specialized
```


## <a name="create-the-image-version"></a>Créer la version de l’image

Créez une version d’image à partir de la machine virtuelle à l’aide de la commande [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create).  

Les caractères autorisés pour la version d’image sont les nombres et les points. Les nombres doivent être un entier 32 bits. Format: *MajorVersion*.*MinorVersion*.*Patch*.

Dans cet exemple, la version de notre image est *1.0.0* et nous allons créer 2 réplicas dans la région *USA Centre-Ouest*, 1 réplica dans la région *USA Centre Sud* et 1 réplica dans la région *USA Est 2* à l’aide du stockage redondant interzone. Les régions de réplication doivent inclure la région dans laquelle se trouve la machine virtuelle source.

Remplacez la valeur de `--managed-image` dans cet exemple par l’ID de votre machine virtuelle récupéré à l’étape précédente.

```azurecli-interactive 
az sig image-version create \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 \
   --target-regions "westcentralus" "southcentralus=1" "eastus=1=standard_zrs" \
   --replica-count 2 \
   --managed-image "/subscriptions/<Subscription ID>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM"
```

> [!NOTE]
> Vous devez attendre que la version d’image soit totalement intégrée et répliquée avant de pouvoir utiliser la même image managée pour créer une autre version d’image.
>
> Vous pouvez également stocker votre image dans le Stockage Premium, en ajoutant `--storage-account-type  premium_lrs`, ou dans le [stockage redondant interzone](../storage/common/storage-redundancy.md), en ajoutant `--storage-account-type  standard_zrs`, quand vous créez la version de l’image.
>

## <a name="next-steps"></a>Étapes suivantes

Créez une machine virtuelle à partir de l’[image généralisée](vm-generalized-image-version-cli.md) à l’aide d’Azure CLI.

Pour savoir comment fournir des informations sur le plan d’achat, consultez [Donner des informations sur le plan d’achat de la Place de marché Azure lors de la création d’images](marketplace-images.md).
