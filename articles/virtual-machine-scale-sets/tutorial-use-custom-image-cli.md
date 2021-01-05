---
title: Tutoriel - Utiliser une image de machine virtuelle personnalisée dans un groupe identique avec Azure CLI
description: Découvrez comment utiliser Azure CLI pour créer une image de machine virtuelle personnalisée que vous pouvez utiliser pour déployer un groupe de machines virtuelles identiques
author: cynthn
ms.service: virtual-machine-scale-sets
ms.subservice: imaging
ms.topic: tutorial
ms.date: 05/01/2020
ms.author: cynthn
ms.custom: mvc, devx-track-azurecli
ms.reviewer: akjosh
ms.openlocfilehash: 62cf7c979be83454ae2433befcdbf4f5d8e5524f
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94516541"
---
# <a name="tutorial-create-and-use-a-custom-image-for-virtual-machine-scale-sets-with-the-azure-cli"></a>Tutoriel : Créer et utiliser une image personnalisée pour des groupes de machines virtuelles identiques avec Azure CLI
Lorsque vous créez un groupe identique, vous spécifiez une image à utiliser lors du déploiement des instances de machine virtuelle. Pour réduire le nombre de tâches une fois que les instances de machine virtuelle sont déployées, vous pouvez utiliser une image de machine virtuelle personnalisée. Cette image de machine virtuelle personnalisée inclut les configurations ou installations des applications requises. Toutes les instances de machine virtuelle créées dans le groupe identique utilisent l’image de machine virtuelle personnalisée et sont prêtes à répondre au trafic des applications. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer une galerie Shared Image Gallery
> * Créer une définition d’image spécialisée
> * Créer une version d’image
> * Créer un groupe identique à partir d’une image spécialisée
> * Partager une galerie d’images


[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.4.0 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="overview"></a>Vue d’ensemble

Une [galerie d’images partagées](shared-image-galleries.md) simplifie considérablement le partage d’images personnalisées dans votre organisation. Les images personnalisées sont comme des images de la Place de marché, sauf que vous les créez vous-même. Les images personnalisées peuvent être utilisées pour amorcer des configurations comme le préchargement des applications, les configurations d’application et d’autres configurations de système d’exploitation. 

Shared Image Gallery vous permet de partager vos images de machines virtuelles personnalisées avec d’autres utilisateurs. Choisissez les images à partager, les régions dans lesquelles vous souhaitez les rendre disponibles et les personnes avec lesquelles vous voulez les partager. 

## <a name="create-and-configure-a-source-vm"></a>Créer et configurer une machine virtuelle source

Commencez par créer un groupe de ressource avec [az group create](/cli/azure/group), puis créez une machine virtuelle avec [az vm create](/cli/azure/vm). Cette machine virtuelle est ensuite utilisée comme source de l’image. L’exemple suivant crée une machine virtuelle nommée *myVM* dans le groupe de ressources nommé *myResourceGroup* :

```azurecli-interactive
az group create --name myResourceGroup --location eastus

az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image ubuntults \
  --admin-username azureuser \
  --generate-ssh-keys
```

> [!IMPORTANT]
> L’**ID** de votre machine virtuelle est affichée dans la sortie de la commande [az vm create](/cli/azure/vm). Copiez-le à un emplacement sûr pour pouvoir l’utiliser plus loin dans ce tutoriel.

L’adresse IP publique de votre machine virtuelle est également affichée dans la sortie de la commande [az vm create](/cli/azure/vm). La connexion SSH à l’adresse IP publique de votre machine virtuelle se présente comme suit :

```console
ssh azureuser@<publicIpAddress>
```

Pour personnaliser votre machine virtuelle, nous allons installer un serveur web de base. Lorsque l’instance de machine virtuelle dans le groupe identique est déployée, elle doit avoir tous les packages nécessaires pour exécuter une application web. Utilisez `apt-get` pour installer *NGINX* comme suit :

```bash
sudo apt-get install -y nginx
```

Quand vous avez terminé, tapez `exit` pour déconnecter la connexion SSH.

## <a name="create-an-image-gallery"></a>Créer une galerie d’images 

Une galerie d’images est la ressource principale utilisée pour activer le partage d’image. 

Les caractères autorisés pour le nom de galerie sont les lettres majuscules ou minuscules, les chiffres et les points. Le nom de galerie ne peut pas contenir de tirets.   Les noms de galerie doivent être uniques dans votre abonnement. 

Créez une galerie d’images à l’aide de [az sig create](/cli/azure/sig#az-sig-create). L’exemple suivant crée un groupe de ressources nommé *myGalleryRG* dans *USA Est* et une galerie nommée *myGallery*.

```azurecli-interactive
az group create --name myGalleryRG --location eastus
az sig create --resource-group myGalleryRG --gallery-name myGallery
```

## <a name="create-an-image-definition"></a>Créer une définition d’image

Les définitions d’image créent un regroupement logique des images. Elles sont utilisées pour gérer les informations sur les versions d’image créées au sein de celles-ci. 

Les noms de définition d’image peuvent contenir des lettres majuscules ou minuscules, des chiffres, des tirets et des points. 

Vérifiez que le type de votre définition d’image est approprié. Si vous avez généralisé la machine virtuelle (à l’aide de Sysprep pour Windows ou de waagent -deprovision pour Linux), vous devez créer une définition d’image généralisée à l’aide de la commande `--os-state generalized`. Si vous souhaitez utiliser la machine virtuelle sans supprimer de comptes d’utilisateur, créez une définition d’image spécialisée à l’aide de la commande `--os-state specialized`.

Pour plus d’informations sur les valeurs que vous pouvez spécifier pour une définition d’image, consultez [Définitions d’image](../virtual-machines/linux/shared-image-galleries.md#image-definitions).

Créez une définition d’image dans la galerie avec la commande [az sig image-definition create](/cli/azure/sig/image-definition#az-sig-image-definition-create).

Dans cet exemple, la définition d’image se nomme *myImageDefinition* et est destinée à une image de système d’exploitation Linux [spécialisée](../virtual-machines/linux/shared-image-galleries.md#generalized-and-specialized-images). Pour créer une définition pour des images utilisant un système d’exploitation Windows, utilisez `--os-type Windows`. 

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

> [!IMPORTANT]
> L’**ID** de votre définition d’image est indiqué dans la sortie de la commande. Copiez-le à un emplacement sûr pour pouvoir l’utiliser plus loin dans ce tutoriel.


## <a name="create-the-image-version"></a>Créer la version de l’image

Créez une version d’image à partir de la machine virtuelle à l’aide de la commande [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create).  

Les caractères autorisés pour la version d’image sont les nombres et les points. Les nombres doivent être un entier 32 bits. Format: *MajorVersion*.*MinorVersion*.*Patch*.

Dans cet exemple, la version de notre image est *1.0.0* et nous allons créer un réplica dans la région *USA Centre Sud* et un réplica dans la région *USA Est 2*. Les régions de réplication doivent inclure la région dans laquelle se trouve la machine virtuelle source.

Remplacez la valeur de `--managed-image` dans cet exemple par l’ID de votre machine virtuelle récupéré à l’étape précédente.

```azurecli-interactive 
az sig image-version create \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 \
   --target-regions "southcentralus=1" "eastus=1" \
   --managed-image "/subscriptions/<Subscription ID>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM"
```

> [!NOTE]
> Vous devez attendre que la version d’image soit totalement intégrée et répliquée avant de pouvoir utiliser la même image managée pour créer une autre version d’image.
>
> Vous pouvez également stocker votre image dans le Stockage Premium, en ajoutant `--storage-account-type  premium_lrs`, ou dans le [stockage redondant interzone](../storage/common/storage-redundancy.md), en ajoutant `--storage-account-type  standard_zrs`, quand vous créez la version de l’image.
>




## <a name="create-a-scale-set-from-the-image"></a>Créer un groupe identique à partir de l’image
Créez un groupe identique à partir de l’image spécialisée à l’aide de la commande [`az vmss create`](/cli/azure/vmss#az-vmss-create). 

Créez le groupe identique avec la commande [`az vmss create`](/cli/azure/vmss#az-vmss-create), en spécifiant le paramètre --specialized pour indiquer que l’image est une image spécialisée. 

Utilisez l’ID de définition d’image pour `--image` afin de créer les instances de groupe identique à partir de la version la plus récente de l’image disponible. Vous pouvez également créer les instances de groupe identique à partir d’une version spécifique en fournissant l’ID de version de l’image pour `--image`. 

Créez un groupe identique nommé *myScaleSet* avec la dernière version de l’image *myImageDefinition* que nous avons créée plus tôt.

```azurecli
az group create --name myResourceGroup --location eastus
az vmss create \
   --resource-group myResourceGroup \
   --name myScaleSet \
   --image "/subscriptions/<Subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition" \
   --specialized
```

La création et la configuration des l’ensemble des ressources et des machines virtuelles du groupe identique prennent quelques minutes.


## <a name="test-your-scale-set"></a>Tester votre groupe identique
Pour autoriser le trafic à atteindre votre groupe identique, vérifiez que le serveur web fonctionne correctement et créez une règle d’équilibreur de charge avec [az network lb rule create](/cli/azure/network/lb/rule). L’exemple suivant crée une règle nommée *myLoadBalancerRuleWeb* qui autorise le trafic sur le port *TCP* *80* :

```azurecli-interactive
az network lb rule create \
  --resource-group myResourceGroup \
  --name myLoadBalancerRuleWeb \
  --lb-name myScaleSetLB \
  --backend-pool-name myScaleSetLBBEPool \
  --backend-port 80 \
  --frontend-ip-name loadBalancerFrontEnd \
  --frontend-port 80 \
  --protocol tcp
```

Pour voir votre groupe identique en action, obtenez l’adresse IP publique de votre équilibreur de charge avec [az network public-ip show](/cli/azure/network/public-ip). L’exemple suivant obtient l’adresse IP pour *myScaleSetLBPublicIP* qui a été créée dans le cadre du groupe identique :

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroup \
  --name myScaleSetLBPublicIP \
  --query [ipAddress] \
  --output tsv
```

Entrez l’adresse IP publique dans votre navigateur web. La page web NGINX par défaut s’affiche comme dans l’exemple suivant :

![Nginx s’exécutant à partir de l’image de machine virtuelle personnalisée](media/tutorial-use-custom-image-cli/default-nginx-website.png)



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

Pour plus d’informations sur la façon de partager des ressources à l’aide d’Azure RBAC, consultez [Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure CLI](../role-based-access-control/role-assignments-cli.md).


## <a name="clean-up-resources"></a>Nettoyer les ressources
Pour supprimer votre groupe identique et les ressources supplémentaires, supprimez le groupe de ressources et toutes ses ressources avec [az group delete](/cli/azure/group). Le paramètre `--no-wait` retourne le contrôle à l’invite de commandes sans attendre que l’opération se termine. Le paramètre `--yes` confirme que vous souhaitez supprimer les ressources sans passer par une invite supplémentaire à cette fin.

```azurecli-interactive
az group delete --name myResourceGroup --no-wait --yes
```


## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez appris à créer et utiliser une image de machine virtuelle personnalisée pour vos groupes identiques avec Azure CLI :

> [!div class="checklist"]
> * Créer une galerie Shared Image Gallery
> * Créer une définition d’image spécialisée
> * Créer une version d’image
> * Créer un groupe identique à partir d’une image spécialisée
> * Partager une galerie d’images

Passez au didacticiel suivant pour apprendre à déployer des applications dans votre groupe identique.

> [!div class="nextstepaction"]
> [Déployer des applications dans vos groupes identiques](tutorial-install-apps-cli.md)
