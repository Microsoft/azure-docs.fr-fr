---
title: Accorder l’accès pour gérer PIM - Azure Active Directory| Microsoft Docs
description: Découvrez comment accorder l’accès à d’autres administrateurs pour gérer Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 08/06/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: fba46ee5632f7411c433e4bba29201c59c552f21
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94835201"
---
# <a name="delegate-access-to-privileged-identity-management"></a>Déléguer l’accès à Privileged Identity Management

Pour déléguer l’accès à Privileged Identity Management (PIM), un administrateur général peut affecter d’autres utilisateurs au rôle Administrateur de rôle privilégié. Par défaut, les administrateurs de la sécurité et les lecteurs de sécurité disposent d’un accès en lecture seule à PIM. Pour accorder l’accès à PIM, le premier utilisateur peut affecter d’autres personnes au rôle **Administrateur de rôle privilégié**. Le rôle Administrateur de rôle privilégié est requis pour la gestion des rôles Azure AD uniquement. Les autorisations d’administrateur de rôle privilégié ne sont pas requises pour gérer les paramètres des ressources Azure.

> [!NOTE]
> La gestion de Privileged Identity Management nécessite Azure AD Multi-Factor Authentication. Les comptes Microsoft ne pouvant pas s’inscrire à Azure AD Multi-Factor Authentication, un utilisateur qui se connecte avec un compte Microsoft ne peut pas accéder à Privileged Identity Management.

Assurez-vous qu’il y a toujours au moins deux utilisateurs auxquels est affecté le rôle Administrateur de rôle privilégié, au cas où un utilisateur serait verrouillé ou son compte supprimé.

## <a name="delegate-access-to-manage-pim"></a>Déléguer l’accès pour gérer PIM

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Dans Azure AD, ouvrez **Privileged Identity Management**.

1. Sélectionnez des **rôles Azure AD**.

1. Sélectionnez **Rôles**.

    ![Rôles Azure AD Privileged Identity Management - Rôles](./media/pim-how-to-give-access-to-pim/pim-directory-roles-roles.png)

1. Sélectionnez le rôle **Administrateur de rôle privilégié** pour ouvrir la page des membres.

    ![Administrateur de rôle privilégié - Membres](./media/pim-how-to-give-access-to-pim/pim-pra-members.png)

1. Sélectionnez **Ajouter un membre** pour ouvrir le volet **Ajouter des membres managés**.

1. Cliquez sur **Sélectionner des membres** pour ouvrir le volet **Sélectionner des membres**.

    ![Administrateur de rôle privilégié - Sélectionner des membres](./media/pim-how-to-give-access-to-pim/pim-pra-select-members.png)

1. Sélectionnez un membre, puis cliquez sur **Sélectionner**.

1. Cliquez sur **OK** afin de rendre le membre éligible pour le rôle **Administrateur de rôle privilégié**.

    Lorsque vous affectez un nouveau rôle à un utilisateur dans PIM, celui-ci est automatiquement configuré comme **Éligible** pour activer le rôle.

1. Pour désigner un membre comme permanent, sélectionnez l’utilisateur dans la liste de membres Administrateur de rôle privilégié.

1. Cliquez sur **Plus**, puis sur **Désigner comme permanent** afin de rendre l’affectation permanente.

    ![Administrateur de rôle privilégié - Désigner comme permanent](./media/pim-how-to-give-access-to-pim/pim-pra-make-permanent.png)

1. Envoyez à l’utilisateur un lien pour [Commencer à utiliser Privileged Identity Management](pim-getting-started.md)

## <a name="remove-access-to-manage-pim"></a>Supprimer l’accès pour gérer PIM

Avant de supprimer quelqu’un du rôle Administrateur de rôle privilégié, assurez-vous toujours qu’au moins deux utilisateurs y sont affectés.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez des **rôles Azure AD**.

1. Sélectionnez **Rôles**.

1. Sélectionnez le rôle **Administrateur de rôle privilégié** pour ouvrir la page des membres.

1. Cochez la case en regard de l’utilisateur à supprimer, puis sélectionnez **Supprimer un membre**.

    ![Administrateur de rôle privilégié - Supprimer un membre](./media/pim-how-to-give-access-to-pim/pim-pra-remove-member.png)

1. Lorsque vous êtes invité à confirmer que vous voulez supprimer le membre du rôle, sélectionnez **Oui**.

## <a name="next-steps"></a>Étapes suivantes

- [Commencer à utiliser Privileged Identity Management](pim-getting-started.md)
