---
title: Gérer des listes d’utilisateurs Azure Lab Services à partir de Teams
description: Découvrez comment gérer des listes d’utilisateurs Azure Lab Services à partir de Teams.
ms.topic: article
ms.date: 10/07/2020
ms.openlocfilehash: fa8f96a84be5c4c91e0153216e15963ec0e3d6f8
ms.sourcegitcommit: ba7fafe5b3f84b053ecbeeddfb0d3ff07e509e40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91946493"
---
# <a name="manage-lab-services-user-lists-from-teams"></a>Gérer les listes d’utilisateurs de Lab Services à partir de Teams

Quand un laboratoire est créé dans Teams (voir [Commencer et créer un laboratoire Lab Services à partir de Teams](how-to-get-started-create-lab-within-teams.md)), la liste d’utilisateurs du laboratoire est automatiquement renseignée et synchronisée avec l’appartenance à l’équipe. Chaque personne de l’équipe, y compris les propriétaires, les membres et les invités, est automatiquement ajoutée à la liste d’utilisateurs du laboratoire. Azure Lab Services gère une synchronisation pour l’appartenance à l’équipe et une synchronisation automatique est déclenchée toutes les 24 heures. 

## <a name="sync-users"></a>Synchroniser les utilisateurs

Les enseignants peuvent utiliser le bouton **Synchroniser** pour déclencher une synchronisation manuelle une fois l’appartenance à l’équipe mise à jour. 

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/how-to-manage-users-with-teams/sync-users.png" alt-text="Synchroniser les utilisateurs":::

Une fois la synchronisation automatique ou manuelle terminée, les conditions suivantes s’appliquent selon que le laboratoire a été publié ou non.

* Si le laboratoire n’a pas été publié au moins une fois :
    * Les utilisateurs seront ajoutés ou supprimés de la liste des utilisateurs du laboratoire en fonction des modifications apportées à l’appartenance de l’équipe. 
* Si le laboratoire a été publié au moins une fois, en plus d’ajouter ou de supprimer des utilisateurs, la capacité du laboratoire est automatiquement mise à jour.
    * S’il y a des ajouts à l’équipe, de nouvelles machines virtuelles sont créées.
    * Si un utilisateur a été supprimé de l’équipe, la machine virtuelle associée est également supprimée.

## <a name="next-steps"></a>Étapes suivantes

Une fois que le modèle de machine virtuelle est configuré et que les enseignants choisissent de publier le modèle, le nombre de machines virtuelles équivalant au nombre d’utilisateurs dans la liste d’utilisateurs du laboratoire est créé. Une fois le laboratoire publié et les machines virtuelles créées, les utilisateurs sont automatiquement inscrits auprès du laboratoire et les machines virtuelles sont affectées à leur première connexion à Azure Lab Services, c’est-à-dire quand ils accèdent pour la première fois à l’onglet ayant l’application **Azure Lab Services**. 

Pour publier le modèle de machine virtuelle, accédez à la fenêtre Lab Services de Teams, sélectionnez l’onglet **Modèle** -> **...**  -> **Publier**.

Pour gérer des pools de machines virtuelles, consultez [Gérer un pool de machines virtuelles dans Lab Services à partir de Teams](how-to-manage-vm-pool-within-teams.md).

### <a name="also-review"></a>Consultez également

Voir les articles suivants :

- [Utilisation d’Azure Lab Services dans Teams](lab-services-within-teams-overview.md)
- [Prise en main et création d’un laboratoire Lab Services à partir de Teams](how-to-get-started-create-lab-within-teams.md)
- [Créer des planifications Lab Services à partir de Teams](how-to-create-schedules-within-teams.md)
- [Accéder à une machine virtuelle (vue étudiant) dans Lab Services à partir de Teams](how-to-access-vm-for-students-within-teams.md)

