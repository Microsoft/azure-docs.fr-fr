---
title: Joindre ou détacher une galerie d’images partagées dans Azure Lab Services | Microsoft Docs
description: Cet article explique comment joindre une galerie d'images partagées à un labo de classe dans Azure Lab Services.
ms.topic: article
ms.date: 09/11/2020
ms.openlocfilehash: ae0870139d2320fa079f6705956e124f61479882
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660098"
---
# <a name="attach-or-detach-a-shared-image-gallery-in-azure-lab-services"></a>Joindre ou détacher une galerie d’images partagées dans Azure Lab Services
Cet article vous montre comment joindre ou détacher une galerie d’images partagées pour un compte de laboratoire. 

> [!NOTE]
> Quand vous [enregistrez une image de modèle d’un lab](how-to-use-shared-image-gallery.md#save-an-image-to-the-shared-image-gallery) dans Azure Lab Services vers Shared Image Gallery, l’image est chargée dans la galerie en tant qu’image spécialisée. Les [images spécialisées](../virtual-machines/windows/shared-image-galleries.md#generalized-and-specialized-images) conservent les informations spécifiques à la machine et les profils utilisateur. Vous pouvez toujours charger directement une image généralisée dans la galerie en dehors d’Azure Lab Services. 
>
> Un créateur de lab peut créer un modèle de machine virtuelle basé sur des images généralisées et spécialisées dans Azure Lab Services. 

## <a name="scenarios"></a>Scénarios
Voici les deux scénarios que cette fonctionnalité prend en charge : 

- Un administrateur de compte lab joint une galerie d’images partagées au compte lab, puis charge une image dans la galerie d’images partagées en dehors du contexte d’un laboratoire. Ensuite, les créateurs de laboratoires peuvent utiliser cette image de la galerie d’images partagées pour créer des laboratoires. 
- Un administrateur de compte lab attache une galerie d’images partagées au compte lab. Un créateur de laboratoire (formateur) enregistre l’image personnalisée de son laboratoire dans la galerie d’images partagées. Ensuite, d’autres créateurs de laboratoires peuvent sélectionner cette image à partir de la galerie d’images partagées afin de créer un modèle pour leurs laboratoires. 

    Quand une image est enregistrée dans une galerie d’images partagées, Azure Lab Services réplique l’image enregistrée dans d’autres régions disponibles au sein de la même [zone géographique](https://azure.microsoft.com/global-infrastructure/geographies/). Ainsi, l’image est disponible pour les labos créés dans d’autres régions de la même zone géographique. L’enregistrement d’images dans une galerie d’images partagées entraîne un coût supplémentaire, qui comprend le coût de toutes les images répliquées. Ce coût est distinct du coût d’utilisation d’Azure Lab Services. Pour plus d’informations sur les tarifs liés à Shared Image Gallery, consultez [Vue d’ensemble de Shared Image Gallery - Facturation](../virtual-machines/windows/shared-image-galleries.md#billing).

> [!IMPORTANT]
> Lors de l’utilisation de Shared Image Gallery, Azure Lab Services prend en charge uniquement les images avec moins de 128 Go d’espace disque pour le système d’exploitation. Les images avec plus de 128 Go d’espace disque ou plusieurs disques n’apparaissent pas dans la liste des images de machine virtuelle lors de la création du labo.

## <a name="configure-at-the-time-of-lab-account-creation"></a>Effectuer la configuration au moment de la création du compte lab
Lorsque vous créez un compte de laboratoire, vous pouvez joindre une galerie d’images partagées au compte de laboratoire. Vous pouvez sélectionner une galerie d’images partagées existante dans la liste déroulante ou en créer une. Pour créer et joindre une galerie d’images partagées pour le compte de laboratoire, sélectionnez **Créer**, entrez un nom pour la galerie, puis saisissez **OK**. 

![Configurer la galerie d’images partagées lors de la création du compte de laboratoire](./media/how-to-use-shared-image-gallery/new-lab-account.png)

## <a name="configure-after-the-lab-account-is-created"></a>Effectuer la configuration une fois le compte de laboratoire créé
Une fois que le compte de laboratoire est créé, vous pouvez effectuer les tâches suivantes :

- Créer et joindre une galerie d’images partagées
- Joindre une galerie d’images partagées au compte de laboratoire
- Détacher une galerie d’images partagées du compte de laboratoire

## <a name="create-and-attach-a-shared-image-gallery"></a>Créer et joindre une galerie d’images partagées
1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le menu de gauche, sélectionnez **Tous les services**. Sélectionnez **Lab Services** dans la section **DEVOPS**. Si vous sélectionnez l’étoile (`*`) en regard de **Lab Services**, celui-ci est ajouté à la section **FAVORIS** dans le menu de gauche. Les fois suivantes, sélectionnez **Lab Services** sous **FAVORIS**.

    ![Tous les services -> Lab Services](./media/tutorial-setup-lab-account/select-lab-accounts-service.png)
3. Sélectionnez votre compte de laboratoire pour afficher la page **Compte Lab**. 
4. Sélectionnez l’option **Galerie d’images partagé** dans le menu de gauche, puis choisissez **+Créer** sur la barre d’outils.  

    ![Bouton Créer une galerie d’images partagées](./media/how-to-use-shared-image-gallery/new-shared-image-gallery-button.png)
5. Dans la fenêtre **Créer une galerie image partagées**, saisissez un **nom** pour la galerie, puis saisissez **OK**. 

    ![Créer une galerie d’images partagées](./media/how-to-use-shared-image-gallery/create-shared-image-gallery-window.png)

    Azure Lab Services crée la galerie d’images partagées et la joint au compte de laboratoire. Tous les laboratoires créés avec ce compte de laboratoire ont accès à la galerie d’images partagées jointe. 

    ![Galerie d’images jointe](./media/how-to-use-shared-image-gallery/image-gallery-in-list.png)

    Dans le volet inférieur, vous pouvez voir les images de la galerie d’images partagées. Dans cette nouvelle galerie, il n’y a aucune image. Lorsque vous chargez des images dans la galerie, elles apparaissent sur cette page.     

    Toutes les images de la galerie d’images partagées jointe sont activées par défaut. Vous pouvez activer ou désactiver les images sélectionnées en les choisissant dans la liste et en utilisant le bouton **Activer les images sélectionnées** ou **Désactiver les images sélectionnées**.

## <a name="attach-an-existing-shared-image-gallery"></a>Joindre une galerie d’images partagées existante
La procédure suivante vous montre comment joindre une galerie d’images partagées existante à un compte de laboratoire. 

1. Sur la page **Compte de laboratoire**, sélectionnez **Galerie d’images partagées** dans le menu de gauche, et choisissez **Joindre** dans la barre d’outils. 

    ![Galerie d’images partagées - Bouton Ajouter](./media/how-to-use-shared-image-gallery/sig-attach-button.png)
5. Sur la page **Attacher une galerie d’images partagées existante**, sélectionnez votre galerie d’images partagées et cliquez sur **OK**.

    ![Sélectionner une galerie existante](./media/how-to-use-shared-image-gallery/select-image-gallery.png)
6. Vous voyez l’écran suivant : 

    ![Ma galerie dans la liste](./media/how-to-use-shared-image-gallery/my-gallery-in-list.png)
    
    Dans cet exemple, il n’y pas encore d’images dans la galerie d’images partagées.

    Une identité Azure Lab Services est ajoutée en tant que contributeur à la galerie d’images partagées qui est jointe au laboratoire. Cela permet aux formateurs ou administrateurs informatiques d’enregistrer les images de machines virtuelles dans la galerie d’images partagées. Tous les laboratoires créés avec ce compte de laboratoire ont accès à la galerie d’images partagées jointe. 

    Toutes les images de la galerie d’images partagées jointe sont activées par défaut. Vous pouvez activer ou désactiver les images sélectionnées en les choisissant dans la liste et en utilisant le bouton **Activer les images sélectionnées** ou **Désactiver les images sélectionnées**. 

## <a name="detach-a-shared-image-gallery"></a>Détacher une galerie d’images partagées
Une seule galerie d’images partagées peut être jointe à un laboratoire. Si vous souhaitez en joindre une autre, commencez par détacher la galerie existante. Pour détacher une galerie d’images partagées de votre laboratoire, sélectionnez l’option **Détacher** de la barre d’outils, puis confirmez l’opération de détachement. 

![Détacher une galerie d’images partagées du compte de laboratoire](./media/how-to-use-shared-image-gallery/detach.png)

## <a name="next-steps"></a>Étapes suivantes
Pour savoir comment enregistrer une image de laboratoire dans la galerie d’images partagées ou utiliser une image de cette galerie afin de créer une machine virtuelle, voir [How to use shared image gallery](how-to-use-shared-image-gallery.md) (Comment utiliser une galerie d’images partagées).

Pour plus d’informations sur les galeries d’images partagées en général, voir [Shared Image Gallery overview](../virtual-machines/windows/shared-image-galleries.md) (Vue d’ensemble de la galerie d’images partagées).