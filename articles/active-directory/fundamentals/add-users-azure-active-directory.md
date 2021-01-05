---
title: Ajouter ou supprimer des utilisateurs - Azure Active Directory | Microsoft Docs
description: Instructions pour ajouter de nouveaux utilisateurs ou supprimer des utilisateurs existants à l’aide d’Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 11/12/2019
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: de529a8ffb0c72854904717c71dbc322c919f1a8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996693"
---
# <a name="add-or-delete-users-using-azure-active-directory"></a>Ajouter ou supprimer des utilisateurs à l’aide d’Azure Active Directory

Ajoutez de nouveaux utilisateurs ou supprimez des utilisateurs existants de votre organisation Azure Active Directory (Azure AD). Pour ajouter ou supprimer des utilisateurs, vous devez être Administrateur d’utilisateurs ou Administrateur général.

## <a name="add-a-new-user"></a>Ajouter un nouvel utilisateur

Vous pouvez créer un utilisateur à l’aide du portail Azure Active Directory.

Pour ajouter un nouvel utilisateur, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en tant qu’administrateur d’utilisateurs de l’organisation.

1. Recherchez et sélectionnez *Azure Active Directory* à partir de n’importe quelle page.

1. Sélectionnez **Utilisateurs**, puis **Nouvel utilisateur**.

    ![Ajouter un utilisateur via Utilisateurs - Tous les utilisateurs dans Azure AD](media/add-users-azure-active-directory/add-user-in-users-all-users.png)

1. Sur la page **Utilisateur**, entrez les informations pour cet utilisateur :

   - **Nom**. Obligatoire. Prénom et nom du nouvel utilisateur. Par exemple, *Mary Parker*.

   - **Nom d’utilisateur**. Obligatoire. Nom d’utilisateur du nouvel utilisateur. Par exemple : `mary@contoso.com`.

     Le domaine dans le nom d’utilisateur doit correspondre au nom de domaine par défaut initial, *\<yourdomainname>.onmicrosoft.com*, ou à un nom de domaine personnalisé, comme *contoso.com*. Pour plus d’informations sur la création d’un nom de domaine personnalisé, consultez [Ajouter votre nom de domaine personnalisé à l’aide du Portail Azure Active Directory](add-custom-domain.md).

   - **Groupes**. Si vous le souhaitez, vous pouvez ajouter l’utilisateur à un ou plusieurs groupes existants. Vous pouvez également ajouter l’utilisateur à des groupes ultérieurement. Pour plus d’informations sur l’ajout d’utilisateurs à des groupes, consultez [Créer un groupe de base et ajouter des membres avec Azure Active Directory](active-directory-groups-create-azure-portal.md).

   - **Rôle d’annuaire** : si vous devez attribuer des autorisations d’administration Azure AD à l’utilisateur, vous pouvez les ajouter à un rôle Azure AD. Vous pouvez attribuer à l'utilisateur le rôle d'Administrateur général, ou lui confier un ou plusieurs des rôles d'administrateur limités dans Azure AD. Pour plus d’informations sur l’affectation de rôles, consultez [Attribuer des rôles aux utilisateurs](active-directory-users-assign-role-azure-portal.md).

   - **Informations sur l’emploi** : Vous pouvez ajouter ici des informations supplémentaires sur l’utilisateur, ou le faire ultérieurement. Pour plus d’informations sur l’ajout d’informations sur l’utilisateur, consultez [Ajouter ou modifier des informations de profil utilisateur](active-directory-users-profile-azure-portal.md).

1. Copiez le mot de passe généré automatiquement fourni dans le champ **Mot de passe**. Vous devrez fournir ce mot de passe à l’utilisateur pour qu’il se connecte la première fois.

1. Sélectionnez **Create** (Créer).

L’utilisateur est créé et ajouté à votre organisation Azure AD.

## <a name="add-a-new-guest-user"></a>Ajouter un nouvel utilisateur invité

Vous pouvez également inviter un nouvel utilisateur invité à collaborer avec votre organisation en sélectionnant **Inviter un utilisateur** sur la page **Nouvel utilisateur**. Si les paramètres de collaboration externe de votre organisation sont configurés de telle sorte que vous êtes autorisé à inviter des invités, l'utilisateur recevra par e-mail une invitation qu'il devra accepter pour entamer la collaboration. Pour plus d'informations sur l'invitation d’utilisateurs en vue d’une collaboration B2B, voir [Inviter des utilisateurs B2B à Azure Active Directory](../external-identities/add-users-administrator.md)

## <a name="add-a-consumer-user"></a>Ajouter un utilisateur consommateur

Dans certains scénarios, vous souhaiterez créer manuellement des comptes consommateurs dans votre répertoire Azure Active Directory B2C (Azure AD B2C). Pour plus d'informations sur la création de comptes consommateurs, voir [Créer et supprimer des utilisateurs consommateurs dans Azure AD B2C](../../active-directory-b2c/manage-users-portal.md).

## <a name="add-a-new-user-within-a-hybrid-environment"></a>Ajouter un nouvel utilisateur au sein d’un environnement hybride

Si vous avez un environnement avec Azure Active Directory (cloud) et Windows Server Active Directory (local), vous pouvez ajouter de nouveaux utilisateurs en synchronisant les données du compte d’utilisateur existant. Pour obtenir plus d’informations sur les utilisateurs et environnements hybrides, consultez [Intégrer vos répertoires locaux avec Azure Active Directory](../hybrid/whatis-hybrid-identity.md).

## <a name="delete-a-user"></a>Supprimer un utilisateur

Vous pouvez supprimer un utilisateur existant à l’aide du portail Azure Active Directory.

Pour supprimer un utilisateur, effectuez les étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l'aide d'un compte d'administrateur d'utilisateurs de l'organisation.

1. Recherchez et sélectionnez *Azure Active Directory* à partir de n’importe quelle page.

1. Recherchez et sélectionnez l'utilisateur que vous souhaitez supprimer de votre locataire Azure AD. Par exemple, _Mary Parker_.

1. Sélectionnez **Supprimer l’utilisateur**.

    ![Page Utilisateurs - Tous avec l’option Supprimer l’utilisateur mis en surbrillance](media/add-users-azure-active-directory/delete-user-all-users-blade.png)

L’utilisateur est supprimé et n’apparaît plus sur la page **Utilisateurs : Tous les utilisateurs**. L’utilisateur est affiché sur la page **Utilisateurs supprimés** pendant 30 jours et peut être restauré durant cette période. Pour plus d’informations sur la restauration d’un utilisateur, consultez [Restaurer ou supprimer un utilisateur supprimé à l’aide d’Azure Active Directory](active-directory-users-restore.md).

Lorsqu'un utilisateur est supprimé, toutes les licences utilisées par celui-ci sont mises à la disposition d'autres utilisateurs.

>[!Note]
>Vous devez utiliser Windows Server Active Directory pour mettre à jour l’identité, les informations de contact ou sur l’emploi des utilisateurs dont la source d’autorité est Windows Server Active Directory. Une fois la mise à jour terminée, vous devez attendre la fin du prochain cycle de synchronisation pour constater les modifications.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez ajouté vos utilisateurs, vous pouvez exécuter les procédures de base suivantes :

- [Ajouter ou modifier les informations de profil](active-directory-users-profile-azure-portal.md)

- [Attribuer des rôles aux utilisateurs](active-directory-users-assign-role-azure-portal.md)

- [Créer un groupe de base et ajouter des membres](active-directory-groups-create-azure-portal.md)

- [Travailler avec des utilisateurs et groupes dynamiques](../enterprise-users/groups-create-rule.md)

Ou vous pouvez effectuer d’autres tâches de gestion des utilisateurs, comme [ajouter des utilisateurs invités depuis un autre répertoire](../external-identities/what-is-b2b.md) ou [restaurer un utilisateur supprimé](active-directory-users-restore.md). Pour en savoir plus sur les autres actions disponibles, consultez la [documentation Gestion des utilisateurs Azure Active Directory](../enterprise-users/index.yml).