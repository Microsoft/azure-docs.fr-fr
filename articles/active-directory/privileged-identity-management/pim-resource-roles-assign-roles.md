---
title: Attribuer des rôles de ressource Azure dans Privileged Identity Management - Azure Active Directory | Microsoft Docs
description: Découvrez comment attribuer des rôles de ressources Azure dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 10/23/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 011b50449f0a02b128bc000535f93e1809fd9777
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92673883"
---
# <a name="assign-azure-resource-roles-in-privileged-identity-management"></a>Attribuer des rôles de ressources Azure dans Privileged Identity Management

Azure Active Directory (Azure AD) Privileged Identity Management (PIM) peut gérer les rôles de ressources intégrés d’Azure, ainsi que les rôles personnalisés, notamment (liste non exhaustive) :

- Propriétaire
- Administrateur de l'accès utilisateur
- Contributeur
- Administrateur de la sécurité
- Gestionnaire de sécurité

> [!NOTE]
> Les utilisateurs ou membres d’un groupe affecté aux rôles d’abonnement Propriétaire ou Administrateur d’accès utilisateur et les administrateurs d’Azure AD Global qui activent la gestion des abonnements dans Azure AD disposent des droits d’administrateur de ressources par défaut. Ces administrateurs peuvent assigner des rôles, configurer des paramètres de rôle et revoir les accès à l’aide de Privileged Identity Management pour Azure Resources. Un utilisateur ne peut pas gérer Privileged Identity Management pour les ressources sans autorisations d’administrateur de ressources. Affichez la liste des [rôles Azure intégrés](../../role-based-access-control/built-in-roles.md).

## <a name="assign-a-role"></a>Attribuer un rôle

Suivez ces étapes pour rendre un utilisateur éligible pour un rôle de ressource Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com/) avec les autorisations de rôle Propriétaire ou Administrateur de l’accès utilisateur.

    Pour plus d’informations sur la façon d’accorder l’accès à un autre administrateur pour la gestion de Privileged Identity Management, consultez [Accorder l’accès à d’autres administrateurs pour la gestion de Privileged Identity Management](pim-how-to-give-access-to-pim.md).

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Ressources Azure**.

1. Utilisez le filtre de ressources pour rechercher les ressources managées que vous recherchez.

    ![Liste des ressources Azure à gérer](./media/pim-resource-roles-assign-roles/resources-list.png)

1. Sélectionnez la ressource que vous souhaitez gérer pour ouvrir la page de présentation des ressources.

1. Sous **Gérer** , sélectionnez **Rôles** pour afficher la liste des rôles pour les ressources Azure.

    ![Rôles de ressources Azure](./media/pim-resource-roles-assign-roles/resources-roles.png)

1. Sélectionnez **Ajouter des affectations** pour ouvrir le volet **Ajouter des affectations**.

1. Choisissez **Sélectionner un rôle** pour ouvrir la page **Sélectionner un rôle**.

    ![Volet Nouvelle affectation](./media/pim-resource-roles-assign-roles/resources-select-role.png)

1. Sélectionnez un rôle que vous souhaitez attribuer et cliquez sur **Sélectionner**.

    Le volet **Sélectionner un membre ou un groupe** s’ouvre.

1. Sélectionnez un membre ou un groupe que vous souhaitez affecter au rôle, puis cliquez sur **Sélectionner**.

    ![Volet Sélectionner un membre ou un groupe](./media/pim-resource-roles-assign-roles/resources-select-member-or-group.png)

1. Dans l’onglet **Paramètres** , liste **Type d’affectation** , sélectionnez **Éligible** ou **Actif**.

    ![Volet des paramètres des appartenances](./media/pim-resource-roles-assign-roles/resources-membership-settings-type.png)

    Azure AD Privileged Identity Management pour les ressources Azure fournit deux types distincts d’attribution :

    - Les attributions **éligibles** exigent des membres qu’ils effectuent une action pour utiliser ce rôle. Il peut s’agir de procéder à une vérification de l’authentification multifacteur (MFA), de fournir une justification professionnelle ou de demander une approbation aux approbateurs désignés.

    - Les attributions **actives** n’exigent pas des membres qu’ils effectuent une action pour utiliser ce rôle. Les membres attribués comme étant actifs détiennent à tout moment les privilèges affectés au rôle.

1. Pour spécifier une durée d’attribution spécifique, modifiez les dates et heures de début et de fin.

1. Lorsque vous avez terminé, sélectionnez **Attribuer**.

1. Une fois la nouvelle attribution de rôle créée, une notification d’état s’affiche.

    ![Nouvelle affectation - Notification](./media/pim-resource-roles-assign-roles/resources-new-assignment-notification.png)

## <a name="update-or-remove-an-existing-role-assignment"></a>Mettre à jour ou supprimer une attribution de rôle existante

Suivez ces étapes pour mettre à jour ou supprimer une attribution de rôle existante.

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Ressources Azure**.

1. Sélectionnez la ressource que vous souhaitez gérer pour ouvrir sa page de présentation.

1. Sous **Gérer** , sélectionnez **Rôles** pour afficher la liste des rôles pour les ressources Azure.

    ![Rôles de ressources Azure - Sélectionner un rôle](./media/pim-resource-roles-assign-roles/resources-update-select-role.png)

1. Sélectionnez le rôle que vous souhaitez mettre à jour ou supprimer.

1. Recherchez l’attribution de rôle sous les onglets **Rôles éligibles** et **Rôles actifs**.

    ![Mettre à jour ou supprimer une attribution de rôle](./media/pim-resource-roles-assign-roles/resources-update-remove.png)

1. Sélectionnez **Mettre à jour** ou **Supprimer** pour mettre à jour ou supprimer l’attribution de rôle.

    Pour obtenir des informations sur l’extension d’une attribution de rôle, consultez [Étendre ou renouveler des rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-renew-extend.md).

## <a name="next-steps"></a>Étapes suivantes

- [Étendre ou renouveler des rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-renew-extend.md)
- [Configurer les paramètres des rôles de ressources Azure dans Privileged Identity Management](pim-resource-roles-configure-role-settings.md)
- [Attribuer des rôles dans Azure AD dans Privileged Identity Management](pim-how-to-add-role-to-user.md)
