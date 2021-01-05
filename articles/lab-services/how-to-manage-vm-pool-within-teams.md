---
title: Gérer un pool de machines virtuelles dans Azure Lab Services à partir de Teams
description: Découvrez comment gérer un pool de machines virtuelles dans Azure Lab Services à partir de Teams.
ms.topic: article
ms.date: 10/07/2020
ms.openlocfilehash: b838e0561bb48b20450e49aaef37baf3c9ecb4d0
ms.sourcegitcommit: ba7fafe5b3f84b053ecbeeddfb0d3ff07e509e40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91946502"
---
# <a name="manage-a-vm-pool-in-lab-services-from-teams"></a>Gérer un pool de machines virtuelles dans Lab Services à partir de Teams

La création d’une machine virtuelle démarre dès que le modèle de machine virtuelle est publié pour la première fois. Des machines virtuelles correspondant au nombre d’utilisateurs dans la liste des utilisateurs du laboratoire sont créées. Les machines virtuelles sont automatiquement attribuées aux élèves lors de leur première connexion à Azure Lab Services. 

## <a name="publish-a-template-and-manage-a-vm-pool"></a>Publication d’un modèle et gestion d’un pool de machines virtuelles

Pour publier le modèle, accédez à la fenêtre Lab Services de Teams, sélectionnez l’onglet **Modèle** -> **...**  -> **Publier**.

Une fois que le modèle de machine virtuelle est configuré et que les enseignants choisissent de publier le modèle, le nombre de machines virtuelles équivalant au nombre d’utilisateurs dans la liste d’utilisateurs du laboratoire est créé. Une fois le laboratoire publié et les machines virtuelles créées, les utilisateurs sont automatiquement inscrits auprès du laboratoire et les machines virtuelles sont affectées à leur première connexion à Azure Lab Services, c’est-à-dire quand ils accèdent pour la première fois à l’onglet ayant l’application **Azure Lab Services**. 

Quand une synchronisation de liste d’utilisateurs est déclenchée, la capacité du laboratoire (nombre de machines virtuelles dans le laboratoire) est automatiquement mise à jour en fonction des modifications apportées à l’appartenance de l’équipe. De nouvelles machines virtuelles sont créées à mesure que de nouveaux utilisateurs sont ajoutés, et les machines virtuelles affectées aux utilisateurs supprimés de l’équipe sont également supprimées. Pour plus d’informations, consultez [Comment gérer les utilisateurs dans Teams](how-to-manage-user-lists-within-teams.md). 

Les enseignants peuvent continuer à accéder aux machines virtuelles des étudiants directement à partir de l’onglet Pool de machines virtuelles. Les enseignants peuvent accéder aux machines virtuelles qui leur sont affectées à partir de l’onglet **Pool de machines virtuelles** ou en cliquant sur le bouton **Mes machines virtuelles** (coin supérieur droit de l’écran). 

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/how-to-manage-vm-pool-with-teams/vm-pool.png" alt-text="Pool de machines virtuelles":::

## <a name="next-steps"></a>Étapes suivantes

Voir les articles suivants :

- [Utilisation d’Azure Lab Services dans Teams](lab-services-within-teams-overview.md)
- [Prise en main et création d’un laboratoire Lab Services à partir de Teams](how-to-get-started-create-lab-within-teams.md)
- [Gérer les listes d’utilisateurs de Lab Services à partir de Teams](how-to-manage-user-lists-within-teams.md)
- [Créer des planifications Lab Services à partir de Teams](how-to-create-schedules-within-teams.md)
- [Accéder à une machine virtuelle (vue étudiant) dans Lab Services à partir de Teams](how-to-access-vm-for-students-within-teams.md)


