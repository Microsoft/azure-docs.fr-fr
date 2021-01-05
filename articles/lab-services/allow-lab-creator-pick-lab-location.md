---
title: Autoriser un créateur de labo à choisir une localisation dans Azure Lab Services
description: Cet article explique comment un administrateur de compte lab peut autoriser des créateurs de labo à choisir des localisations pour leurs labos.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 3b6c2d83414b1abc763755fbf15402b122c2186b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85444197"
---
# <a name="allow-lab-creator-to-pick-location-for-the-lab-in-azure-lab-services"></a>Autoriser un créateur de labo à choisir la localisation du labo dans Azure Lab Services
Dans Azure Lab Services, le propriétaire d’un compte lab peut permettre aux créateurs de labo (enseignants) de choisir une localisation pour le labo créé. Cette localisation peut être différente de celle du compte lab. Une localisation est un groupe de régions Azure. Par exemple, la localisation États-Unis est un groupe de régions comme USA Est, USA Ouest, etc. 

En tant que propriétaire de compte lab, vous pouvez sélectionner l’option **Autoriser le créateur du lab à choisir l’emplacement du lab** quand vous créez un compte lab ou après avoir créé un compte lab (ou sur un compte lab existant). 

## <a name="at-the-time-of-lab-account-creation"></a>Au moment de la création du compte lab
Quand vous créez un compte lab, vous voyez cette option sur le premier écran (onglet **Fonctions de base**). 

![Activer l’option au moment de la création du labo](./media/allow-lab-creator-pick-lab-location/create-lab-account.png)

Cette option est désactivée si vous sélectionnez un réseau virtuel pair pour votre compte lab dans l’onglet **Options avancées**.  

![Quand un réseau virtuel pair est activé](./media/allow-lab-creator-pick-lab-location/peer-virtual-network.png)


## <a name="after-the-lab-account-is-created"></a>Après la création du compte lab
Après avoir créé le compte lab, vous pouvez activer ou désactiver cette option en effectuant les étapes suivantes : 

1. Dans la page **Compte lab**, sélectionnez **Paramètres lab** dans le menu de gauche.
2. Sélectionnez l’option **Autoriser le créateur du lab à choisir l’emplacement du lab** si vous souhaitez autoriser le créateur du labo à sélectionner une localisation pour le labo. Si l’option est désactivée, les laboratoires sont automatiquement créés dans le même emplacement que celui dans lequel se trouve le compte de laboratoire. 
    
    Ce champ est désactivé lorsque vous sélectionnez un réseau virtuel pour le champ **Appairer un réseau virtuel**. Cela s’explique par le fait que les laboratoires dans le compte de laboratoire doivent être dans la même région que le compte de laboratoire pour pouvoir accéder aux ressources dans le réseau virtuel de pair. 
1. Sélectionnez **Enregistrer** dans la barre d’outils. 

    ![Paramètres du labo](./media/allow-lab-creator-pick-lab-location/lab-settings.png)

## <a name="no-virtual-network-and-location-selection-isnt-allowed"></a>Aucun réseau virtuel et choix de la localisation non autorisé
Dans ce scénario, vous n’avez pas activé l’option **Autoriser le créateur du lab à choisir l’emplacement du lab**. 

![Aucune localisation de labo](./media/allow-lab-creator-pick-lab-location/lab-no-location.png)

Dans ce cas, les créateurs de labo (enseignants) ne voient pas d’option permettant de choisir une localisation pour le labo. Ils voient le tarif horaire de chaque option de taille disponible. Quand ils créent un labo, celui-ci est créé dans une région Azure qui se trouve au même endroit que celle où se trouve le compte lab. Par exemple, si le compte lab se trouve dans la région **USA Ouest**, le labo peut être créé dans la région **USA Centre Sud**, mais pas dans la région **Canada Est**. Nous ne garantissons rien en ce qui concerne la région que nous choisissons si ce n’est qu’il s’agit de l’endroit du compte. Si la taille actuelle est restreinte, le créateur de labo verra une case à cocher avec les tailles que nous prenons généralement en charge, mais qui ne sont pas disponibles actuellement. 

## <a name="in-virtual-network-and-location-selection-isnt-allowed"></a>Dans un réseau virtuel et choix de localisation non autorisé
Dans ce scénario, l’option **Autoriser le créateur du lab à choisir l’emplacement du lab** est désactivée, car vous avez sélectionné un réseau virtuel pair pour le compte lab. Dans ce cas, les créateurs de labo voient le même écran qu’avec l’option précédente. Étant donné que toutes les machines virtuelles doivent se trouver dans la même région Azure que le réseau virtuel, le labo sera créé dans cette région Azure. Si la taille de cette région spécifique est restreinte, la taille est indiquée comme non disponible. 

## <a name="location-selection-is-enabled"></a>Choix de localisation autorisé
Quand vous sélectionnez **Autoriser le créateur du lab à choisir l’emplacement du lab**, les créateurs de labo (enseignants) voient une option permettant de sélectionner une localisation lors de la création d’un labo. 

![Choisir la localisation du labo](./media/allow-lab-creator-pick-lab-location/location-selection.png)

Les créateurs de labo voient la gamme de prix pour toutes les localisations prenant en charge la taille spécifique et peuvent en choisir une. Le labo sera créé dans une région Azure qui correspond à cette localisation.

Si une localisation est petite, elle n’apparaît pas dans la liste par défaut. Développez la liste déroulante, puis sélectionnez **Afficher les emplacements non disponibles pour cette taille**. 

![Afficher les localisations non disponibles](./media/allow-lab-creator-pick-lab-location/show-unavailable-locations.png)

## <a name="cost"></a>Coût
Jusqu’à présent, les tarifs étaient basés sur la taille de machine virtuelle choisie pour le labo. À présent, ils sont basés sur la combinaison de système d’exploitation, taille et localisation. 

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- [Connecter le réseau de votre labo avec un réseau virtuel pair](how-to-connect-peer-virtual-network.md)
- [Attacher une galerie d’images partagées à un lab](how-to-attach-detach-shared-image-gallery.md)
- [Ajouter un utilisateur comme propriétaire de labo](how-to-add-user-lab-owner.md)
- [Afficher les paramètres de pare-feu pour un labo](how-to-configure-firewall-settings.md)
- [Configurer les autres paramètres d’un lab](how-to-configure-lab-accounts.md)