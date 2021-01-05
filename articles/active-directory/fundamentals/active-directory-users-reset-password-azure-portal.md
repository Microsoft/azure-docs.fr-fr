---
title: Réinitialiser le mot de passe d’un utilisateur - Azure Active Directory | Microsoft Docs
description: Instructions de réinitialisation du mot de passe d’un utilisateur à l’aide d’Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.assetid: fad5624b-2f13-4abc-b3d4-b347903a8f16
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
ms.date: 09/05/2018
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 397c74203aae2f52ce81844695266cc36fdf3042
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92370897"
---
# <a name="reset-a-users-password-using-azure-active-directory"></a>Réinitialiser le mot de passe d’un utilisateur à l’aide d’Azure Active Directory

En tant qu’administrateur, vous pouvez réinitialiser le mot de passe d’un utilisateur en cas d’oubli, si l’utilisateur ne peut plus accéder à un appareil verrouillé, ou bien s’il n’a jamais reçu son mot de passe.

>[!Note]
>Vous ne serez pas en mesure de réinitialiser le mot de passe d’un utilisateur si votre locataire Azure AD n’est pas son répertoire de base. Cela signifie que si votre utilisateur se connecte à votre organisation à l’aide d’un compte venant d’une autre organisation, un compte Microsoft ou un compte Google, vous ne pourrez réinitialiser son mot de passe.<br><br>Si votre utilisateur a une source d’autorité comme Windows Server Active Directory, vous ne serez en mesure de réinitialiser le mot de passe que si vous avez activé la réécriture du mot de passe.<br><br>Si votre utilisateur a une source d’autorité comme Azure AD externe, il se peut que vous ne puissiez pas réinitialiser le mot de passe. Seul l’utilisateur, ou un administrateur dans Azure AD externe, peut réinitialiser le mot de passe.

>[!Note]
>Si vous n’êtes pas administrateur et si vous recherchez des instructions de réinitialisation pour votre propre mot de passe professionnel ou scolaire, consultez [Réinitialiser votre mot de passe professionnel ou scolaire](../user-help/active-directory-passwords-update-your-own-password.md).

## <a name="to-reset-a-password"></a>Pour réinitialiser un mot de passe

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en tant qu’administrateur d’utilisateurs ou administrateur de mots de passe. Pour obtenir plus d’informations sur les rôles disponibles, consultez l’article [Attribution de rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md#available-roles).

2. Sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , recherchez et sélectionnez l’utilisateur qui a besoin de la réinitialisation, puis sélectionnez **Réinitialiser le mot de passe**.

    La page **Alain Charon - profil** s’affiche avec l’option **Réinitialiser le mot de passe**.

    ![Page du profil de l’utilisateur, avec l’option Réinitialiser le mot de passe mis en surbrillance](media/active-directory-users-reset-password-azure-portal/user-profile-reset-password-link.png)

3. Sur la page **Réinitialiser le mot de passe** , sélectionnez **Réinitialiser le mot de passe**.

    > [!Note]
    > Lorsque Azure Active Directory est utilisé, un mot de passe temporaire est automatiquement généré pour l’utilisateur. Lorsque vous utilisez Active Directory en local, vous créez le mot de passe pour l’utilisateur.

4. Copiez le mot de passe et donnez-le à l’utilisateur. L’utilisateur devra le modifier lors de sa prochaine connexion.

    >[!Note]
    >Le mot de passe temporaire n’expire jamais. La prochaine fois que l’utilisateur se connectera, le mot de passe fonctionnera toujours, quel que soit le temps écoulé dans depuis sa création.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez réinitialisé le mot de passe de votre utilisateur, vous pouvez exécuter les procédures de base suivantes :

- [Ajouter ou supprimer des utilisateurs](add-users-azure-active-directory.md)

- [Attribuer des rôles aux utilisateurs](active-directory-users-assign-role-azure-portal.md)

- [Ajouter ou modifier les informations de profil](active-directory-users-profile-azure-portal.md)

- [Créer un groupe de base et ajouter des membres](active-directory-groups-create-azure-portal.md)

Vous pouvez également suivre des scénarios utilisateur plus complexes, comme l’affectation de délégués, l’utilisation de stratégies et le partage de comptes d’utilisateur. Pour en savoir plus sur les autres actions disponibles, consultez la [documentation Gestion des utilisateurs Azure Active Directory](../enterprise-users/index.yml).
