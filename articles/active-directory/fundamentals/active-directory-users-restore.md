---
title: Restaurer ou supprimer définitivement un utilisateur récemment supprimé - Azure AD
description: Comment afficher les utilisateurs pouvant être restaurés, restaurer un utilisateur supprimé et supprimer de façon définitive un utilisateur avec Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 10/23/2020
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: bd4d42e1cafa58aaf6a47188c9e86c24ac175047
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92514387"
---
# <a name="restore-or-remove-a-recently-deleted-user-using-azure-active-directory"></a>Restaurer ou supprimer un utilisateur supprimé à l’aide d’Azure Active Directory
Lorsque vous supprimez un utilisateur, son compte reste à l’état suspendu pendant 30 jours. Pendant ces 30 jours, le compte de l’utilisateur peut être restauré, avec l’ensemble de ses propriétés. Au terme de cette période de 30 jours, le processus de suppression permanent est démarré automatiquement.

Vous pouvez afficher les utilisateurs pouvant être restaurés, restaurer un utilisateur supprimé ou supprimer de manière définitive un utilisateur à l’aide d’Azure Active Directory (Azure AD) dans le portail Azure.

>[!Important]
>Un utilisateur supprimé de manière définitive ne peut pas être restauré, ni par vous, ni par le support technique de Microsoft.

## <a name="required-permissions"></a>Autorisations requises
Vous devez disposer d’un des rôles suivants pour restaurer et supprimer définitivement des utilisateurs.

- Administrateur général

- Prise en charge de niveau 1 de partenaire

- Prise en charge de niveau 2 de partenaire

- Administrateur d’utilisateurs

## <a name="view-your-restorable-users"></a>Afficher les utilisateurs pouvant être restaurés
Vous pouvez afficher tous les utilisateurs qui ont été supprimés il y a moins de 30 jours. Ces utilisateurs peuvent être restaurés.

### <a name="to-view-your-restorable-users"></a>Pour afficher les utilisateurs pouvant être restaurés
1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l'aide d'un compte d'administrateur général de l'organisation.

2. Sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Utilisateurs supprimés** .

    Passez en revue la liste des utilisateurs pouvant être restaurés.

    ![Page Utilisateurs - Utilisateurs supprimés, avec les utilisateurs pouvant encore être restaurés](media/active-directory-users-restore/users-deleted-users-view-restorable.png)

## <a name="restore-a-recently-deleted-user"></a>Restaurer un utilisateur supprimé récemment

Lorsqu'un compte d'utilisateur est supprimé de l'organisation, l'état de celui-ci est défini sur suspendu et toutes les informations en lien avec l'organisation sont conservées. Lorsque vous restaurez un utilisateur, ces informations en lien avec l'organisation sont également restaurées.

> [!Note]
> Une fois qu'un utilisateur est restauré, les licences qui lui avaient été attribuées avant la suppression sont également restaurées même si aucun siège n'est disponible pour ces licences. Si vous utilisez alors plus de licences que ce que vous n'en avez achetées, votre organisation peut temporairement se retrouver en situation de non-conformité en termes d'utilisation des licences.

### <a name="to-restore-a-user"></a>Pour restaurer un utilisateur
1. Sur la page **Utilisateurs - Utilisateurs supprimés** , recherchez et sélectionnez l’un des utilisateurs disponibles. Par exemple, _Mary Parker_ .

2. Sélectionnez **Restaurer utilisateur** .

    ![Page Utilisateurs - Utilisateurs supprimés, avec l’option Restaurer utilisateur mise en surbrillance](media/active-directory-users-restore/users-deleted-users-restore-user.png)

## <a name="permanently-delete-a-user"></a>Supprimer définitivement un utilisateur.
Vous pouvez supprimer définitivement un utilisateur de votre organisation sans attendre la fin du délai de 30 jours avant la suppression automatique. Un utilisateur supprimé définitivement ne peut pas être restauré, ni par vous, ni par un autre administrateur, ni par le support technique de Microsoft.

>[!Note]
>Si vous supprimez définitivement un utilisateur par erreur, vous devrez créer un nouvel utilisateur et saisir manuellement toutes les informations associées. Pour plus d’informations sur la création d’un utilisateur, consultez [Ajouter ou supprimer des utilisateurs](add-users-azure-active-directory.md).

### <a name="to-permanently-delete-a-user"></a>Pour supprimer définitivement un utilisateur

1. Sur la page **Utilisateurs - Utilisateurs supprimés** , recherchez et sélectionnez l’un des utilisateurs disponibles. Par exemple, _Rae Huff_ .

2. Sélectionnez **Supprimer définitivement** .

    ![Page Utilisateurs - Utilisateurs supprimés, avec l’option Supprimer l’utilisateur mise en surbrillance](media/active-directory-users-restore/users-deleted-users-permanent-delete-user.png)

## <a name="next-steps"></a>Étapes suivantes
Une fois que vous avez restauré ou supprimé vos utilisateurs, vous pouvez :

- [Ajouter ou supprimer des utilisateurs](add-users-azure-active-directory.md)

- [Attribuer des rôles aux utilisateurs](active-directory-users-assign-role-azure-portal.md)

- [Ajouter ou modifier les informations de profil](active-directory-users-profile-azure-portal.md)

- [Ajouter des utilisateurs invités à partir d'une autre organisation](../external-identities/what-is-b2b.md)

Pour en savoir plus sur les autres tâches de gestion des utilisateurs disponibles, consultez la [documentation Gestion des utilisateurs Azure AD](../enterprise-users/index.yml).
