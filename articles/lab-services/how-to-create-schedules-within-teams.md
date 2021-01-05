---
title: Créer des planifications Azure Lab Services dans Teams
description: Découvrez comment créer des planifications Azure Lab Services dans Teams.
ms.topic: article
ms.date: 10/07/2020
ms.openlocfilehash: 202b202f99868875a51d13e95fbcac677246cc05
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92042335"
---
# <a name="create-and-manage-lab-services-schedules-within-teams"></a>Créer et gérer des planifications Azure Lab Services dans Teams

Les planifications vous permettent de configurer un labo de classe de sorte que les machines virtuelles utilisées dans le labo démarrent et s’arrêtent automatiquement à une heure précise. Vous pouvez définir une planification ponctuelle ou périodique. Les procédures suivantes décrivent comment créer et gérer des planifications pour un labo de classe : 

Voici comment les planifications affectent les machines virtuelles de labo : 

- Le modèle de machine virtuelle n’est pas inclus dans les planifications. 
- Seules les machines virtuelles affectées sont démarrées. Cela signifie que si une machine n’est pas demandée par un utilisateur final (étudiant), elle ne démarrera pas aux heures planifiées. 
- Toutes les machines virtuelles (qu’elles soient demandées par un utilisateur ou non) sont arrêtées en fonction de la planification du labo. 

> [!IMPORTANT]
> Le temps d’exécution planifié des machines virtuelles ne compte pas dans le quota alloué à un utilisateur. Le quota s’applique au temps pendant lequel un étudiant utilise les machines virtuelles en dehors des heures planifiées. 

Les utilisateurs peuvent créer, modifier et supprimer des planifications de labo dans Teams de la même façon que sur le [site web des labos](https://labs.azure.com). Reportez-vous à l’article sur la [création et la gestion des planifications](how-to-create-schedules-within-teams.md).

## <a name="automatic-shutdown-and-disconnect-settings"></a>Paramètres d’arrêt et de déconnexion automatiques

Vous pouvez activer plusieurs fonctionnalités de contrôle des coûts d’arrêt automatique afin d’éviter de manière proactive les coûts supplémentaires lorsque les machines virtuelles ne sont pas utilisées activement. La combinaison des trois fonctionnalités suivantes d’arrêt et de déconnexion automatiques permet d’intercepter la plupart des cas où les utilisateurs laissent accidentellement leurs machines virtuelles en marche :
 
- Déconnectez automatiquement les utilisateurs des machines virtuelles que le système d’exploitation juge inactives.
- Arrêt automatique des machines virtuelles quand les utilisateurs se déconnectent.
- Arrêter automatiquement les machines virtuelles démarrées, mais auxquelles les utilisateurs ne se connectent pas.

Pour plus de détails, reportez-vous à l’article sur la [configuration des paramètres d’arrêt automatique pour un labo](how-to-enable-shutdown-disconnect.md).

## <a name="next-steps"></a>Étapes suivantes

Voir les articles suivants :

- [Utilisation d’Azure Lab Services dans Teams](lab-services-within-teams-overview.md)
- [Prise en main et création d’un labo dans Teams](how-to-get-started-create-lab-within-teams.md)
- [Gestion des listes d’utilisateurs de labo dans Teams](how-to-manage-user-lists-within-teams.md)
- [Gestion du pool de machines virtuelles du labo dans Teams](how-to-manage-vm-pool-within-teams.md)
- [Accès à une machine virtuelle dans Teams – Affichage étudiant](how-to-access-vm-for-students-within-teams.md)
