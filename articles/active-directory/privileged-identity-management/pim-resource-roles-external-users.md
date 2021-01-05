---
title: Affecter des rôles de ressources Azure aux invités dans PIM - Azure AD | Microsoft Docs
description: Découvrez comment inviter des utilisateurs externes et attribuer des rôles de ressources Azure dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 11/08/2019
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: f4e2e18f3bb9d1c972d805a60493897d605921e4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010637"
---
# <a name="invite-guest-users-and-assign-azure-resource-roles-in-privileged-identity-management"></a>Inviter des utilisateurs invités et attribuer des rôles de ressources Azure dans Privileged Identity Management

Les utilisateurs invités Azure Active Directory (Azure AD) font partie des fonctionnalités de collaboration B2B (Business-to-Business) dans Azure AD, afin que vous puissiez gérer utilisateurs et fournisseurs invités externes en tant qu’invités dans Azure AD. Quand vous combinez B2B Collaboration et Azure AD Privileged Identity Management (PIM), vous pouvez étendre vos critères de conformité et de gouvernance à des invités. Par exemple, vous pouvez utiliser ces fonctionnalités de Privileged Identity Management pour des tâches liées aux identités Azure avec des invités :

- Attribuer l’accès à des ressources Azure spécifiques
- Activer l’accès juste-à-temps
- Spécifier la durée de l’attribution et la date de fin
- Demander l’authentification multifacteur lors de l’activation ou de l’attribution active
- Effectuer des révisions d’accès
- Utiliser des alertes et des journaux d’audit

Cet article explique comment convier un invité dans votre organisation et gérer son accès aux ressources Azure à l’aide d’Azure Active Directory Privileged Identity Management.

## <a name="when-would-you-invite-guests"></a>Quand voulez-vous convier des invités ?

Voici quelques exemples où vous pourriez convier des invités à votre organisation :

- Autoriser un fournisseur externe indépendant qui dispose uniquement d’un compte e-mail à accéder à vos ressources Azure dans le cadre d’un projet.
- Autoriser un partenaire externe d’une grande organisation qui utilise les services de fédération Active Directory (AD FS) locaux à accéder à votre application de notes de frais.
- Autoriser les ingénieurs du support technique ne faisant pas partie de votre organisation, par exemple les ingénieurs du support technique Microsoft, à accéder temporairement à vos ressources Azure pour résoudre des problèmes.

## <a name="how-does-collaboration-using-b2b-guests-work"></a>Comment fonctionne la collaboration avec des invités B2B ?

Lorsque vous utilisez une collaboration B2B, vous pouvez inviter un utilisateur externe à votre organisation en tant qu’invité. L’invité peut être géré en tant qu’utilisateur dans votre organisation, mais il doit être authentifié dans son organisation d’origine et non dans votre organisation Azure AD. Cela signifie que si l’invité n’a plus accès à son organisation d’origine, il n’a également plus accès à votre organisation. Par exemple, si l’invité quitte son organisation, il perd automatiquement l’accès à toutes les ressources que vous avez partagées avec lui dans Azure AD sans que vous ayez rien à faire. Pour plus d’informations sur B2B Collaboration, consultez [Présentation de l’accès utilisateur invité dans Azure Active Directory B2B](../external-identities/what-is-b2b.md).

![Diagramme montrant comment un utilisateur invité est authentifié dans son annuaire d’origine](./media/pim-resource-roles-external-users/b2b-external-user.png)

## <a name="check-guest-collaboration-settings"></a>Vérifier les paramètres de collaboration d’invité

Pour vous assurer que vous pouvez convier des invités dans votre organisation, vous devez vérifier les paramètres de collaboration d’invité.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Sélectionnez **Azure Active Directory** > **Paramètres utilisateur**.

1. Sélectionnez **Gérer les paramètres de collaboration externe**.

    ![Page des paramètres de collaboration externe montrant les paramètres de restriction d’autorisation, d’invitation et de collaboration](./media/pim-resource-roles-external-users/external-collaboration-settings.png)

1. Vérifiez que le commutateur **Les administrateurs et utilisateurs ayant le rôle d’Inviteur d’invités peuvent inviter** a la valeur **Oui**.

## <a name="invite-a-guest-and-assign-a-role"></a>Convier un invité et attribuer un rôle

Privileged Identity Management vous permet de convier un invité et de le rendre éligible à un rôle de ressource Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com/) avec un utilisateur membre du rôle [Administrateur de rôle privilégié](../roles/permissions-reference.md#privileged-role-administrator) ou [Administrateur d’utilisateurs](../roles/permissions-reference.md#user-administrator).

1. Ouvrez **Azure AD Privileged Identity Management**.

1. Sélectionnez **Ressources Azure**.

1. Utilisez le **filtre de ressources** pour filtrer la liste des ressources managées.

1. Sélectionnez la ressource que vous souhaitez gérer, comme une ressource, un groupe de ressources, un abonnement ou un groupe d’administration.

    Définissez l’étendue pour qu’elle corresponde uniquement à ce dont l’invité a besoin.

1. Sous Gérer, sélectionnez **Rôles** pour afficher la liste des rôles pour les ressources Azure.

    ![Liste des rôles de ressources Azure indiquant le nombre d’utilisateurs actifs et éligibles](./media/pim-resource-roles-external-users/resources-roles.png)

1. Sélectionnez le rôle minimum dont l’utilisateur a besoin.

    ![Page de rôle sélectionné répertoriant les membres actuels de ce rôle](./media/pim-resource-roles-external-users/selected-role.png)

1. Dans la page de rôle, sélectionnez **Ajouter un membre** pour ouvrir le volet Nouvelle affectation.

1. Cliquez sur **Sélectionner un membre ou un groupe**.

    ![Nouvelle affectation – Sélectionnez un volet de membre ou de groupes répertoriant des utilisateurs et des groupes avec une option Inviter](./media/pim-resource-roles-external-users/select-member-group.png)

1. Pour convier un invité, cliquez sur **Inviter**.

    ![Page Convier un invité avec des zones pour entrer une adresse e-mail et spécifier un message personnel](./media/pim-resource-roles-external-users/invite-guest.png)

1. Après avoir sélectionné un invité, cliquez sur **inviter**.

    L’invité doit être ajouté en tant que membre sélectionné.

1. Dans le volet **Sélectionner un membre ou un groupe**, cliquez sur **Sélectionner**.

1. Dans le volet **Paramètres d’appartenance**, sélectionnez le type et la durée de l’affectation.

    ![Nouvelle affectation – Page de paramètres d’appartenance contenant des options pour spécifier le type d’affectation, ainsi que les dates de début et de fin de celle-ci](./media/pim-resource-roles-external-users/membership-settings.png)

1. Pour terminer l’attribution, sélectionnez **Terminé**, puis **Ajouter**.

    L’attribution de rôle d’invité s’affiche dans votre liste de rôles.

    ![Page de rôles indiquant que l’invité est éligible](./media/pim-resource-roles-external-users/role-assignment.png)

## <a name="activate-role-as-a-guest"></a>Activer un rôle en tant qu’invité

Si vous êtes utilisateur externe, vous devez accepter l’invitation d’invité à l’organisation Azure AD et, le cas échéant, activer votre attribution de rôle.

1. Ouvrez l’e-mail contenant votre invitation. L’e-mail ressemble au suivant.

    ![E-mail d’invitation avec nom d’annuaire, un message personnel et un lien pour commencer](./media/pim-resource-roles-external-users/email-invite.png)

1. Sélectionnez le lien **Bien démarrer** dans l’e-mail.

1. Après avoir passé en revue les autorisations, cliquez sur **Accepter**.

    ![Examinez dans un navigateur la page des autorisations contenant une liste d’autorisations que l’organisation aimerait que vous passiez en revue](./media/pim-resource-roles-external-users/invite-accept.png)

1. Il se peut que vous soyez invité à accepter des conditions d’utilisation et à spécifier si vous souhaitez rester connecté. Dans le portail Azure, si vous êtes *éligible* à un rôle, vous n’avez pas encore accès aux ressources.

1. Pour activer votre attribution de rôle, ouvrez l’e-mail contenant le lien d’activation du rôle. L’e-mail ressemble au suivant.

    ![E-mail indiquant que vous êtes éligible à un rôle et contenant un lien Activer le rôle](./media/pim-resource-roles-external-users/email-role-assignment.png)

1. Sélectionnez **Activer le rôle** pour ouvrir vos rôles éligibles dans Privileged Identity Management.

    ![Page Mes rôles dans Privileged Identity Management listant vos rôles éligibles](./media/pim-resource-roles-external-users/my-roles-eligible.png)

1. Sous Action, sélectionnez le lien **Activer**.

    En fonction des paramètres du rôle, vous devez spécifier certaines informations pour activer le rôle.

1. Après avoir spécifié les paramètres du rôle, cliquez sur **Activer** pour activer le rôle.

    ![Page Activer affichant l’étendue et des options permettant de spécifier l’heure de début, la durée et le motif](./media/pim-resource-roles-external-users/activate-role.png)

    À moins que l’administrateur ne soit tenu d’approuver votre demande, vous devez à présent avoir accès aux ressources spécifiées.

## <a name="view-activity-for-a-guest"></a>Afficher l’activité d’un invité

Vous pouvez afficher des journaux d’audit pour effectuer le suivi de l’activité des invités.

1. En tant qu’administrateur, ouvrez Privileged Identity Management et sélectionnez la ressource qui a été partagée.

1. Sélectionnez **Audit des ressources** pour afficher l’activité de cette ressource. Voici un exemple d’activité pour un groupe de ressources.

    ![Ressources Azure – Page Audit des ressources indiquant le temps, le demandeur et l’action](./media/pim-resource-roles-external-users/audit-resource.png)

1. Pour afficher l’activité de l’invité, sélectionnez **Azure Active Directory** > **Utilisateurs** > *nom de l’invité*.

1. Sélectionnez **Journaux d’audit** pour afficher les journaux d’audit pour l’organisation. Si nécessaire, vous pouvez spécifier des filtres.

    ![Journaux d’audit d’annuaire indiquant la date, la cible, la personne ayant initié l’activité et l’activité](./media/pim-resource-roles-external-users/audit-directory.png)

## <a name="next-steps"></a>Étapes suivantes

- [Attribuer des rôles d’administrateur Azure AD dans Privileged Identity Management](pim-how-to-add-role-to-user.md)
- [Présentation de l’accès utilisateur invité dans Azure AD B2B Collaboration](../external-identities/what-is-b2b.md)