---
title: Azure Lab Services dans Microsoft Teams
description: Fournit une vue d’ensemble de l’utilisation d’Azure Lab Services dans Microsoft Teams.
ms.topic: conceptual
ms.date: 10/07/2020
ms.openlocfilehash: f39613427ed174dfca80d4a48be9473ab7025e79
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96433921"
---
# <a name="azure-lab-services-within-microsoft-teams"></a>Azure Lab Services dans Microsoft Teams

Azure Lab Services peut être exploité dans Microsoft Teams à l’aide de l’application Teams **Azure Lab Services**. Tout propriétaire d’équipe disposant d’un accès propriétaire/contributeur/créateur aux comptes de labo peut créer des labos et provisionner des machines virtuelles pour tous les membres de l’équipe.

Cet article présente les avantages de l’utilisation d’Azure Lab Services dans Teams et fournit des liens vers d’autres articles fournissant des instructions sur la façon de créer et de gérer des labos au sein de Teams. 

> [!NOTE]
>L’application Teams **Azure Lab Services** ne peut être ajoutée qu’à une équipe ; elle ne peut pas être ajoutée à des conversations individuelles ou à des conversations de groupe.

## <a name="benefits"></a>Avantages

L’intégration d’Azure Lab Services à Microsoft Teams aide les enseignants à configurer un environnement de type salle de classe et à fournir des environnements de labo virtuels au sein de l’équipe (salle de classe) : 

* Les enseignants peuvent configurer des labos pour que les étudiants puissent accéder à leurs machines virtuelles depuis Teams, sans quitter Teams et avoir à accéder au [site web Azure Lab Services](https://labs.azure.com).
* Authentification unique (SSO) depuis Teams vers Azure Lab Services.
* Les propriétaires d’équipes et de labos n’ont pas besoin de conserver les listes de classe dans deux systèmes différents : la liste des utilisateurs de labo est automatiquement renseignée à partir de l’appartenance à l’équipe et une synchronisation est effectuée automatiquement toutes les 24 heures. 
* Après la publication initiale de la machine virtuelle modèle, la capacité du labo (c’est-à-dire le nombre de machines virtuelles dans le labo) est automatiquement ajustée en fonction de l’ajout et de la suppression d’utilisateurs de l’appartenance à l’équipe. 
* Les propriétaires d’équipe et de labo ne voient que les labos associés à l’équipe et les étudiants ne voient que les machines virtuelles, qui sont provisionnées pour l’équipe concernée. 
* Les utilisateurs sont automatiquement inscrits au labo et les machines virtuelles sont automatiquement attribuées lors de la première connexion après la publication du labo. Les enseignants n’ont pas besoin d’envoyer des invitations et les étudiants n’ont pas besoin de s’inscrire au labo séparément.  

## <a name="next-steps"></a>Étapes suivantes

Voir les articles suivants :

- [Bien démarrer et créer un labo dans Teams](how-to-get-started-create-lab-within-teams.md)
- [Gérer les listes d’utilisateurs de labo dans Teams](how-to-manage-user-lists-within-teams.md)
- [Gérer le pool de machines virtuelles du labo dans Teams](how-to-manage-vm-pool-within-teams.md)
- [Créer et gérer des planifications de labo dans Teams](how-to-create-schedules-within-teams.md)
- [Accéder à une machine virtuelle dans Teams – Vue étudiant](how-to-access-vm-for-students-within-teams.md)
