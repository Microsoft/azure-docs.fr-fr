---
title: Copier une version d'image à partir d'une autre galerie avec Azure CLI
description: Copiez une version d’image à partir d’une autre galerie avec Azure CLI.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: e8aabcd1c68272a78b3c1fe88913c5a62496f681
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88225833"
---
# <a name="copy-an-image-from-another-gallery-using-the-azure-cli"></a>Copiez une image à partir d'une autre galerie en utilisant Azure CLI.

Si vous avez plusieurs galeries dans votre organisation, vous pouvez également créer des versions d’image à partir de versions d’images existantes stockées dans d’autres galeries. Par exemple, vous pouvez avoir une galerie de développement et de test pour créer et tester de nouvelles images. Lorsqu’elles sont prêtes à être utilisées en production, vous pouvez les copier dans une galerie de production à l’aide de cet exemple. Vous pouvez également créer une image à partir d’une image dans une autre galerie à l’aide d’[Azure PowerShell](image-version-another-gallery-powershell.md).



## <a name="before-you-begin"></a>Avant de commencer

Pour suivre cet article, vous devez disposer d’une galerie source, d’une définition d’image et d’une version d’image existantes. Vous devez également disposer d’une galerie de destination. 

La version de l’image source doit être répliquée vers la région où se trouve votre galerie de destination. 

Lorsque vous suivez les étapes de cet article, remplacez les noms de ressources si nécessaire.



## <a name="get-information-from-the-source-gallery"></a>Obtenir des informations de la galerie source

Vous avez besoin d’informations de la définition de l’image source afin de pouvoir en créer une copie dans votre nouvelle galerie.

Répertoriez les informations sur les galeries d’images disponibles à l’aide de la commande [az sig list](/cli/azure/sig#az-sig-list) pour trouver des informations sur la galerie source.

```azurecli-interactive 
az sig list -o table
```

Répertoriez les définitions d’images d’une galerie avec la commande [az sig image-definition list](/cli/azure/sig/image-definition#az-sig-image-definition-list). Dans cet exemple, nous recherchons des définitions d’images dans la galerie nommée *myGallery* dans le groupe de ressources *myGalleryRG*.

```azurecli-interactive 
az sig image-definition list \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   -o table
```

Répertoriez les versions d’une image dans une galerie, à l’aide de [az sig image-version list](/cli/azure/sig/image-version#az-sig-image-version-list) pour trouver la version de l’image que vous souhaitez copier dans votre nouvelle galerie. Dans cet exemple, nous recherchons toutes les versions d’image qui font partie de la définition d’image *myImageDefinition*.

```azurecli-interactive
az sig image-version list \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   -o table
```

Une fois que vous disposez de toutes les informations dont vous avez besoin, vous pouvez obtenir l’ID de la version de l’image source à l’aide de [az sig image-version show](/cli/azure/sig/image-version#az-sig-image-version-show).

```azurecli-interactive
az sig image-version show \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 \
   --query "id" -o tsv
```


## <a name="create-the-image-definition"></a>Créer la définition d’image 

Vous devez créer une définition d’image correspondant à la définition d’image de votre version d’image source. Vous pouvez voir toutes les informations dont vous avez besoin pour recréer la définition d’image dans votre nouvelle galerie à l’aide de [Get-AzGalleryImageDefinition](/cli/azure/sig/image-definition#az-sig-image-definition-show).

```azurecli-interactive
az sig image-definition show \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition
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
  "osType": "Linux",
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

Créez une définition d’image dans votre nouvelle galerie à l’aide des informations de la sortie ci-dessus.


```azurecli-interactive 
az sig image-definition create \
   --resource-group myNewGalleryRG \
   --gallery-name myNewGallery \
   --gallery-image-definition myImageDefinition \
   --publisher myPublisher \
   --offer myOffer \
   --sku mySKU \
   --os-type Linux \
   --hyper-v-generation V1 \
   --os-state specialized 
```


## <a name="create-the-image-version"></a>Créer la version de l’image

Créez des versions avec la commande [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create). L’ID de l’image managée sert de base de référence pour la création de la version d’image. Vous pouvez utiliser [az image list](/cli/azure/image?view#az-image-list) pour obtenir des informations sur les images qui se trouvent dans un groupe de ressources. 

Les caractères autorisés pour la version d’image sont les nombres et les points. Les nombres doivent être un entier 32 bits. Format: *MajorVersion*.*MinorVersion*.*Patch*.

Dans cet exemple, la version de notre image est *1.0.0* et nous allons créer 1 réplica dans la région *USA Centre Sud* et 1 réplica dans la région *USA Est* à l’aide du stockage redondant interzone.


```azurecli-interactive 
az sig image-version create \
   --resource-group myNewGalleryRG \
   --gallery-name myNewGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 \
   --target-regions "southcentralus=1" "eastus=1=standard_zrs" \
   --replica-count 2 \
   --managed-image "/subscriptions/<Subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition/versions/1.0.0"
```

> [!NOTE]
> Vous devez attendre que la version d’image soit totalement intégrée et répliquée avant de pouvoir utiliser la même image managée pour créer une autre version d’image.
>
> Vous pouvez également stocker votre image dans le Stockage Premium, en ajoutant `--storage-account-type  premium_lrs`, ou dans le [stockage redondant interzone](../storage/common/storage-redundancy.md), en ajoutant `--storage-account-type  standard_zrs`, quand vous créez la version de l’image.
>

## <a name="next-steps"></a>Étapes suivantes

Créez une machine virtuelle à partir d’une version d’image [généralisée](vm-generalized-image-version-cli.md) ou [spécialisée](vm-specialized-image-version-cli.md).

Par ailleurs, essayez [Azure Image Builder (en préversion)](./linux/image-builder-overview.md) pour automatiser la création d’une version d’image. Vous pouvez même l’utiliser pour mettre à jour et [créer une nouvelle version de l’image à partir d’une image existante](./linux/image-builder-gallery-update-image-version.md). 

Pour savoir comment fournir des informations sur le plan d'achat, consultez [Donner des informations sur le plan d'achat de la Place de marché Azure lors de la création d'images](marketplace-images.md).
