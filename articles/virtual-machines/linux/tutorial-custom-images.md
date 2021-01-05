---
title: 'Tutoriel : Créer des images de machine virtuelle personnalisées avec l’interface de ligne de commande Azure'
description: Dans ce tutoriel, vous allez apprendre à utiliser Azure CLI pour créer une image de machine virtuelle personnalisée dans Azure.
author: cynthn
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.custom: mvc, devx-track-azurecli
ms.reviewer: akjosh
ms.openlocfilehash: 22609465abfa0cbf30165bc9327d786b3244357e
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94844776"
---
# <a name="tutorial-create-a-custom-image-of-an-azure-vm-with-the-azure-cli"></a>Tutoriel : Créer une image personnalisée d’une machine virtuelle Azure avec Azure CLI

Les images personnalisées sont comme des images de la Place de marché, sauf que vous les créez vous-même. Les images personnalisées peuvent être utilisées pour amorcer des configurations comme le préchargement des applications, les configurations d’application et d’autres configurations de système d’exploitation. Ce didacticiel explique comment créer votre propre image personnalisée d’une machine virtuelle Azure. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer une galerie Shared Image Gallery
> * Créer une définition d’image
> * Créer une version d’image
> * Créer une machine virtuelle à partir d’une image 
> * Partager une galerie d’images


Ce tutoriel utilise l’interface CLI disponible dans [Azure Cloud Shell](../../cloud-shell/overview.md), qui est constamment mise à jour vers la dernière version. Pour ouvrir Cloud Shell, sélectionnez **Essayer** en haut d’un bloc de code.

Si vous choisissez d’installer et d’utiliser l’interface CLI localement, ce tutoriel exige Azure CLI version 2.4.0 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI]( /cli/azure/install-azure-cli).

## <a name="overview"></a>Vue d’ensemble

Une [galerie d’images partagées](shared-image-galleries.md) simplifie considérablement le partage d’images personnalisées dans votre organisation. Les images personnalisées sont comme des images de la Place de marché, sauf que vous les créez vous-même. Les images personnalisées peuvent être utilisées pour amorcer des configurations comme le préchargement des applications, les configurations d’application et d’autres configurations de système d’exploitation. 

Shared Image Gallery vous permet de partager vos images de machines virtuelles personnalisées avec d’autres utilisateurs. Choisissez les images à partager, les régions dans lesquelles vous souhaitez les rendre disponibles et les personnes avec lesquelles vous voulez les partager. 

La fonctionnalité Galerie d’images partagées a plusieurs types de ressources :

[!INCLUDE [virtual-machines-shared-image-gallery-resources](../../../includes/virtual-machines-shared-image-gallery-resources.md)]

## <a name="before-you-begin"></a>Avant de commencer

Les étapes ci-dessous expliquent comment prendre une machine virtuelle existante et la transformer en une image personnalisée réutilisable que vous pouvez utiliser pour créer de nouvelles instances de machines virtuelles.

Pour exécuter l’exemple dans ce didacticiel, vous devez disposer d’une machine virtuelle. Si nécessaire, vous pouvez consulter le [guide de démarrage rapide de l’interface CLI](quick-create-cli.md) pour créer une machine virtuelle à utiliser pour ce tutoriel. Au cours du tutoriel, remplacez les noms des ressources si nécessaire.

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. 

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.

## <a name="create-an-image-gallery"></a>Créer une galerie d’images 

Une galerie d’images est la ressource principale utilisée pour activer le partage d’image. 

Les caractères autorisés pour le nom de galerie sont les lettres majuscules ou minuscules, les chiffres et les points. Le nom de galerie ne peut pas contenir de tirets.   Les noms de galerie doivent être uniques dans votre abonnement. 

Créez une galerie d’images à l’aide de [az sig create](/cli/azure/sig#az-sig-create). L’exemple suivant crée un groupe de ressources nommé *myGalleryRG* dans *USA Est* et une galerie nommée *myGallery*.

```azurecli-interactive
az group create --name myGalleryRG --location eastus
az sig create --resource-group myGalleryRG --gallery-name myGallery
```

## <a name="get-information-about-the-vm"></a>Obtenir des informations sur la machine virtuelle

Vous pouvez obtenir la liste des machines virtuelles disponibles à l’aide de la commande [az vm list](/cli/azure/vm#az-vm-list). 

```azurecli-interactive
az vm list --output table
```

Une fois que vous connaissez le nom de la machine virtuelle et le groupe de ressources dans lequel elle se trouve, récupérez l’ID de la machine virtuelle à l’aide de la commande [az vm get-instance-view](/cli/azure/vm#az-vm-get-instance-view). 

```azurecli-interactive
az vm get-instance-view -g MyResourceGroup -n MyVm --query id
```

Copiez l’ID de votre machine virtuelle pour une utilisation ultérieure.

## <a name="create-an-image-definition"></a>Créer une définition d’image

Les définitions d’image créent un regroupement logique des images. Elles sont utilisées pour gérer les informations sur les versions d’image créées au sein de celles-ci. 

Les noms de définition d’image peuvent contenir des lettres majuscules ou minuscules, des chiffres, des tirets et des points. 

Pour plus d’informations sur les valeurs que vous pouvez spécifier pour une définition d’image, consultez [Définitions d’image](./shared-image-galleries.md#image-definitions).

Créez une définition d’image dans la galerie avec la commande [az sig image-definition create](/cli/azure/sig/image-definition#az-sig-image-definition-create). 

Dans cet exemple, la définition d’image se nomme *myImageDefinition* et est destinée à une image de système d’exploitation Linux [spécialisée](./shared-image-galleries.md#generalized-and-specialized-images). 

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

Copiez l’ID de la définition d’image à partir de la sortie pour l’utiliser ultérieurement.

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
> Vous pouvez également stocker votre image dans le Stockage Premium, en ajoutant `--storage-account-type  premium_lrs`, ou dans le [stockage redondant interzone](../../storage/common/storage-redundancy.md), en ajoutant `--storage-account-type  standard_zrs`, quand vous créez la version de l’image.
>

 
## <a name="create-the-vm"></a>Création de la machine virtuelle

Créez la machine virtuelle avec la commande [az vm create](/cli/azure/vm#az-vm-create), en spécifiant le paramètre --specialized pour indiquer que l’image est une image spécialisée. 

Utilisez l’ID de définition d’image pour `--image` afin de créer la machine virtuelle à partir de la version la plus récente de l’image disponible. Vous pouvez également créer la machine virtuelle à partir d’une version spécifique en fournissant l’ID de version de l’image pour `--image`. 

Dans cet exemple, nous créons une machine virtuelle à partir de la dernière version de l’image *myImageDefinition*.

```azurecli
az group create --name myResourceGroup --location eastus
az vm create --resource-group myResourceGroup \
    --name myVM \
    --image "/subscriptions/<Subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition" \
    --specialized
```

## <a name="share-the-gallery"></a>Partager la galerie

Vous pouvez partager des images entre abonnements à l’aide du contrôle d’accès en fonction du rôle Azure (Azure RBAC). Vous pouvez partager des images au niveau de la galerie, de la définition d’image ou de la version d’image. Tout utilisateur disposant d’autorisations de lecture sur une version d’image, même dans plusieurs abonnements, peut déployer une machine virtuelle à partir de la version d’image.

Nous vous recommandons de partager l’accès avec d’autres utilisateurs au niveau de la galerie. Pour obtenir l’ID d’objet de votre galerie, utilisez [az sig show](/cli/azure/sig#az-sig-show).

```azurecli-interactive
az sig show \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --query id
```

Utilisez l’ID d’objet en tant qu’étendue, ainsi qu’une adresse e-mail et [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) pour donner aux utilisateurs un accès à la galerie d’images partagées. Remplacez `<email-address>` et `<gallery iD>` par vos propres informations.

```azurecli-interactive
az role assignment create \
   --role "Reader" \
   --assignee <email address> \
   --scope <gallery ID>
```

Pour plus d’informations sur la façon de partager des ressources à l’aide d’Azure RBAC, consultez [Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure CLI](../../role-based-access-control/role-assignments-cli.md).

## <a name="azure-image-builder"></a>Azure Image Builder

Azure propose également un service, basé sur Packer, nommé [Azure VM Image Builder](./image-builder-overview.md). Il vous suffit de décrire vos personnalisations dans un modèle, et celui-ci gère la création d’images. 

## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel vous montré comment créer une image de machine virtuelle. Vous avez appris à :

> [!div class="checklist"]
> * Créer une galerie Shared Image Gallery
> * Créer une définition d’image
> * Créer une version d’image
> * Créer une machine virtuelle à partir d’une image 
> * Partager une galerie d’images

Pour découvrir les machines virtuelles hautement disponibles, passez au didacticiel suivant.

> [!div class="nextstepaction"]
> [Créer des machines virtuelles hautement disponibles](tutorial-availability-sets.md)
