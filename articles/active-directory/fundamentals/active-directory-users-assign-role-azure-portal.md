---
title: Attribuer des rôles Azure AD aux utilisateurs - Azure Active Directory | Microsoft Docs
description: Instructions sur l’attribution de rôles administrateur et non-administrateur aux utilisateurs avec Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 08/31/2020
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: fe0c7eec5c023fcba3152ad6329d318210a0c2ab
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92370931"
---
# <a name="assign-administrator-and-non-administrator-roles-to-users-with-azure-active-directory"></a>Attribuer des rôles administrateur et non-administrateur aux utilisateurs avec Azure Active Directory

Dans Azure Active Directory (Azure AD), si l’un de vos utilisateurs a besoin de l’autorisation de gérer les ressources Azure AD, vous devez l’assigner à un rôle qui fournit les autorisations nécessaires. Pour plus d’informations sur les rôles qui gèrent les ressources Azure et sur les rôles qui gèrent les ressources Azure AD, consultez [Rôles d’administrateur d’abonnement classique, rôles Azure et rôles Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).

Pour plus d’informations sur les rôles Azure AD disponibles, consultez [Attribution de rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md). Pour ajouter des utilisateurs, consultez [Ajouter de nouveaux utilisateurs à Azure Active Directory](add-users-azure-active-directory.md).

## <a name="assign-roles"></a>Attribuer des rôles

Une méthode courante pour assigner des rôles Azure AD à un utilisateur se trouve dans la page **Rôles affectés** d’un utilisateur. Vous pouvez également configurer l’éligibilité de l’utilisateur à disposer du droit d’être promu juste-à-temps dans un rôle avec Privileged Identity Management (PIM). Pour plus d’informations sur l’utilisation de PIM, consultez [Privileged Identity Management](../privileged-identity-management/index.yml).

> [!Note]
> Si vous disposez d’un plan de licences Azure AD Premium P2 et que vous utilisez déjà PIM, toutes les tâches de gestion des rôles sont effectuées dans l’[expérience Privileged Identity Management](../roles/manage-roles-portal.md). Cette fonctionnalité est actuellement limitée à l’attribution d’un seul rôle à la fois. Actuellement, vous ne pouvez pas sélectionner plusieurs rôles et les affecter à un utilisateur tous en même temps.
>
> ![Rôles Azure AD gérés dans PIM pour les utilisateurs qui se servent déjà de PIM et qui disposent d’une licence Premium P2](./media/active-directory-users-assign-role-azure-portal/pim-manages-roles-for-p2.png)

## <a name="assign-a-role-to-a-user"></a>Affecter un rôle à un utilisateur

1. Accédez au [portail Azure](https://portal.azure.com/) et connectez-vous à l’aide d’un compte d’administrateur général pour l’annuaire.

2. Recherchez et sélectionnez **Azure Active Directory** .

      ![Recherche dans le portail Azure d'Azure Active Directory](media/active-directory-users-assign-role-azure-portal/search-azure-active-directory.png)

3. Sélectionnez **Utilisateurs** .

4. Recherchez et sélectionnez l’utilisateur qui obtient l’attribution de rôle. Par exemple, _Alain Charon_ .

      ![Page Tous les utilisateurs - Sélectionner l’utilisateur](media/active-directory-users-assign-role-azure-portal/directory-role-select-user.png)

5. Sur la page **Alain Charon - Profil** , sélectionnez **Rôles affectés** .

    La page **Alain Charon - Rôles d’administration** s’affiche.

6. Sélectionnez **Ajouter des attributions** , le rôle à assigner à Alain (par exemple, _Administrateur d’application_ ), puis choisissez **Sélectionner** .

    ![Page Rôles affectés, affichant le rôle sélectionné](media/active-directory-users-assign-role-azure-portal/directory-role-select-role.png)

    Le rôle Administrateur d’application est assigné à Alain Charon et il apparaît dans la page **Alain Charon - Rôles d’administration** .

## <a name="remove-a-role-assignment"></a>Supprimer une attribution de rôle

Si vous devez supprimer l’attribution de rôle d’un utilisateur, vous pouvez également le faire à partir de la page **Alain Charon - Rôles d’administration** .

### <a name="to-remove-a-role-assignment-from-a-user"></a>Pour supprimer une attribution de rôle d’un utilisateur

1. Sélectionnez **Azure Active Directory** , **Utilisateurs** , puis recherchez et sélectionnez l’utilisateur auquel vous souhaitez supprimer un rôle assigné. Par exemple, _Alain Charon_ .

2. Sélectionnez **Rôles affectés** , **Administrateur d’application** , puis sélectionnez **Supprimer l’affectation** .

    ![Page Rôles affectés, affichant le rôle sélectionné et l’option Supprimer](media/active-directory-users-assign-role-azure-portal/directory-role-remove-role.png)

    Le rôle Administrateur d’application d’Alain Charon est supprimé et il n’apparaît plus dans la page **Alain Charon - Rôles d’administration** .

## <a name="next-steps"></a>Étapes suivantes

- [Ajouter ou supprimer des utilisateurs](add-users-azure-active-directory.md)

- [Ajouter ou modifier les informations de profil](active-directory-users-profile-azure-portal.md)

- [Ajouter des utilisateurs invités à partir d’un autre répertoire](../external-identities/what-is-b2b.md)

Vous pouvez consulter d’autres tâches de gestion des utilisateurs disponibles dans [Documentation Gestion des utilisateurs Azure Active Directory](../enterprise-users/index.yml).