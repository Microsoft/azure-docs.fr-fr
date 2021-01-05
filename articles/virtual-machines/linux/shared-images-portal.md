---
title: Créer des images de machine virtuelle Linux Azure partagées à l’aide du portail
description: Découvrez comment utiliser le portail Azure pour créer et partager des images de machine virtuelle Linux.
author: cynthn
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: 2661715164cc6aa5f5ff587f2ddf28c0918445d4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015988"
---
# <a name="create-a-shared-image-gallery-using-the-portal"></a>Créer une galerie d’images partagées à l’aide du portail

Une [galerie d’images partagées](shared-image-galleries.md) simplifie considérablement le partage d’images personnalisées dans votre organisation. Les images personnalisées sont comme des images de la Place de marché, sauf que vous les créez vous-même. Les images personnalisées peuvent être utilisées pour amorcer des tâches de déploiement comme le préchargement des applications, les configurations d’application et d’autres configurations de système d’exploitation. 

Le service Shared Image Gallery vous permet de partager vos images de machine virtuelle personnalisées avec d’autres personnes de votre organisation, dans la même région ou des régions différentes, au sein d’un locataire Azure AD. Choisissez les images à partager, les régions dans lesquelles vous souhaitez les rendre disponibles et les personnes avec lesquelles vous voulez les partager. Vous pouvez créer plusieurs galeries afin de regrouper logiquement les images partagées. 

La galerie est une ressource de niveau supérieur qui fournit le contrôle d'accès en fonction du rôle Azure (Azure RBAC) complet. Les versions des images peuvent être gérées, et vous pouvez choisir de répliquer chaque version d’image vers un autre ensemble de régions Azure. La galerie fonctionne uniquement avec les images managées.

La fonctionnalité Galerie d’images partagées présente plusieurs types de ressources. Dans cet article, nous allons utiliser ou générer ce qui suit :


[!INCLUDE [virtual-machines-shared-image-gallery-resources](../../../includes/virtual-machines-shared-image-gallery-resources.md)]

<br>





## <a name="before-you-begin"></a>Avant de commencer

Pour suivre l’exemple de cet article, vous devez avoir l’image managée d’une machine virtuelle généralisée ou l’instantané d’une machine virtuelle spécialisée. Vous pouvez suivre le [Tutoriel : Créer l’image personnalisée d’une machine virtuelle Azure avec Azure PowerShell](tutorial-custom-images.md) pour créer une image managée, ou [Créer un instantané](../windows/snapshot-copy-managed-disk.md) pour une machine virtuelle spécialisée. Pour les images managées et les instantanés, la taille du disque de données ne peut pas être supérieure à 1 To.

Au cours de cet article, remplacez les noms du groupe de ressources et de la machine virtuelle si nécessaire.

 
[!INCLUDE [virtual-machines-common-shared-images-portal](../../../includes/virtual-machines-common-shared-images-portal.md)]

## <a name="create-vms"></a>Créer des machines virtuelles 

Vous pouvez maintenant créer une ou plusieurs machines virtuelles. Cet exemple crée une machine virtuelle nommée *myVMfromImage*, dans le groupe *myResourceGroup* dans le centre de données *USA Est*.

1. Accédez à la définition de votre image. Vous pouvez utiliser le filtre de ressources pour afficher toutes les définitions d’images disponibles.
1. Dans la page de définition de votre image, sélectionnez **Créer une machine virtuelle** dans le menu situé en haut.
1. Pour **Groupe de ressources**, sélectionnez **Créer** et entrez *myResourceGroup* comme nom.
1. Dans **Nom de la machine virtuelle**, tapez *myVM*.
1. Pour **Région**, sélectionnez *USA Est*.
1. Pour **Options de disponibilité**, conservez la valeur par défaut *Aucune redondance d’infrastructure nécessaire*.
1. La valeur sous **Image** est automatiquement renseignée avec la version d’image `latest` si vous avez démarré à partir de la page de la définition d’image.
1. Pour **Taille**, choisissez une taille de machine virtuelle dans la liste des tailles disponibles, puis **Sélectionner**.
1. Sous **Compte administrateur**, si la machine virtuelle source était généralisée, entrez votre **nom d’utilisateur** et votre **clé publique SSH**. Si la machine virtuelle source était spécialisée, ces options sont grisées, car les informations de la machine virtuelle source sont utilisées.
1. Si vous souhaitez autoriser l’accès à distance à la machine virtuelle, sous **Ports d’entrée publics**, choisissez **Autoriser les ports sélectionnés**, puis sélectionnez **SSH (22)** dans la liste déroulante. Si vous ne souhaitez pas autoriser l’accès à distance à la machine virtuelle, conservez la sélection **Aucun** pour **Ports d’entrée publics**.
1. Lorsque vous avez terminé, sélectionnez le bouton **Vérifier + créer** en bas de la page.
1. Une fois la machine virtuelle validée, sélectionnez **Créer** en bas de la page pour démarrer le déploiement.


## <a name="clean-up-resources"></a>Nettoyer les ressources

Dès que vous n’en avez plus besoin, vous pouvez supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées. Pour cela, sélectionnez le groupe de ressources de la machine virtuelle, sélectionnez **Supprimer**, puis confirmez le nom du groupe de ressources à supprimer.

Si vous souhaitez supprimer des ressources individuelles, vous devez les supprimer dans l’ordre inverse. Par exemple, pour supprimer une définition d’image, vous devez supprimer toutes les versions de l’image créées à partir de cette image.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également créer la ressource de galerie d’images partagées à l’aide de modèles. Plusieurs modèles de démarrage rapide Azure sont disponibles : 

- [Créer une galerie d’images partagées](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [Créer une définition d’image dans une galerie d’images partagées](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [Créer une version d’image dans une galerie d’images partagées](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)
- [Créer une machine virtuelle à partir d’une version d’image](https://azure.microsoft.com/resources/templates/101-vm-from-sig/)

Pour plus d’informations sur les galeries d’images partagées, consultez la [vue d’ensemble](shared-image-galleries.md). Si vous rencontrez des problèmes, consultez [Résoudre les problèmes des galeries d’images partagées](../troubleshooting-shared-images.md).

