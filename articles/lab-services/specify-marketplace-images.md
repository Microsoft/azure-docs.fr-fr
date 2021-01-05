---
title: Spécifier les images de la Place de Marché pour un labo dans Azure Lab Services
description: Cet article explique comment spécifier les images de la Place de Marché, que le créateur de labo peut utiliser pour créer des labos dans un compte lab dans Azure Lab Services.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 5c81c8d7a15d67055729a29e98f5b7e30f3d0764
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96434734"
---
# <a name="specify-marketplace-images-available-to-lab-creators"></a>Spécifier les images de la Place de marché accessibles aux créateurs lab
En tant que propriétaire d’un compte de laboratoire, vous pouvez spécifier les images de la place de marché que les créateurs de laboratoire peuvent utiliser pour créer des laboratoires dans le compte de laboratoire. 

## <a name="select-images-available-for-labs"></a>Sélectionner les images disponibles pour les labos
Sélectionnez **Images de la Place de marché** à gauche du menu. Par défaut, la liste complète des images (activées et désactivées) s’affiche. Vous pouvez filtrer la liste pour afficher uniquement les images activées/désactivées en sélectionnant l’option **Enabled only (Activées uniquement)** /**Disabled only (Désactivées uniquement)** dans la liste déroulante en haut. 
    
![Page des images de la Place de marché](./media/tutorial-setup-lab-account/marketplace-images-page.png)

Les images de la place de marché affichées dans la liste sont uniquement celles qui correspondent aux conditions suivantes :
    
- Crée une seule machine virtuelle.
- Utilise Azure Resource Manager pour approvisionner des machines virtuelles
- Ne nécessite pas l’achat d’un plan de licences supplémentaire

## <a name="disable-images-for-a-lab"></a>Désactiver les images pour un labo 
Pour désactiver une image pour un labo, sélectionnez **... (points de suspension)** dans la dernière colonne, puis sélectionnez **Désactiver l’image**. 

![Désactiver une image](./media/tutorial-setup-lab-account/disable-one-image.png) 

Vous pouvez également activer la case à cocher à côté du nom de l’image, puis sélectionner **Désactiver les images sélectionnées** dans la barre d’outils. 

Pour désactiver plusieurs images simultanément, activez les cases à cocher à côté des noms des images, puis sélectionnez **Désactiver les images sélectionnées** dans la barre d’outils. 

![Désactiver plusieurs images](./media/tutorial-setup-lab-account/disable-multiple-images.png) 


## <a name="enable-images-for-a-lab"></a>Activer des images pour un labo
Pour activer une image désactivée, sélectionnez **... (points de suspension)** dans la dernière colonne, puis sélectionnez **Activer l’image**. Vous pouvez également activer la case à cocher à côté du nom de l’image, puis sélectionner **Activer les images sélectionnées** dans la barre d’outils. 

Pour activer plusieurs images simultanément, activez les cases à cocher à côté des noms des images, puis sélectionnez **Activer les images sélectionnées** dans la barre d’outils. 

## <a name="enable-images-at-the-time-of-lab-creation"></a>Activer des images au moment de la création du lab
Vous pouvez activer davantage d’images au moment de la création du lab : 

1. Connectez-vous au [site web Azure Lab Services](https://labs.azure.com) avec les **informations d’identification du propriétaire du compte Lab**
2. Sélectionnez l’image de machine virtuelle par défaut ou la flèche Bas. 
3. Sélectionnez **Activer d'autres options d'image**. 

    ![Activer d'autres options d'image](./media/specify-marketplace-images/enable-more-images-menu.png)
4. Suivez les instructions de la section précédente pour activer les images sélectionnées. 
5. Vous devrez peut-être fermer la fenêtre **Nouveau lab** et la rouvrir pour voir les images sélectionnées à l’étape précédente. 



## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- [En tant que propriétaire de labo, créer et gérer des labos](how-to-manage-classroom-labs.md)
- [En tant que propriétaire de labo, configurer et publier des modèles](how-to-create-manage-template.md)
- [En tant que propriétaire de labo, configurer et contrôler l’utilisation d’un labo](how-to-configure-student-usage.md)
- [En tant qu’utilisateur de labo, accéder aux labos](how-to-use-classroom-lab.md)
