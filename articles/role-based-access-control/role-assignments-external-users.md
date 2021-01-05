---
title: Ajouter ou supprimer des attributions de rôle Azure pour des utilisateurs externes à l’aide du portail Azure – Azure RBAC
description: Découvrez comment accorder l’accès aux ressources Azure pour les utilisateurs externes à une organisation en utilisant le portail Azure et le contrôle d’accès en fonction du rôle Azure (Azure RBAC).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.assetid: ''
ms.service: role-based-access-control
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 11/25/2019
ms.author: rolyon
ms.reviewer: skwan
ms.custom: it-pro
ms.openlocfilehash: 846e1a83f3cba5f87210ae4f825b5fac4f1569c6
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94648376"
---
# <a name="add-or-remove-azure-role-assignments-for-external-guest-users-using-the-azure-portal"></a>Ajouter ou supprimer des attributions de rôle Azure pour des utilisateurs invités externes à l’aide du portail Azure

La fonctionnalité de [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](overview.md) permet une meilleure gestion de la sécurité pour les grandes organisations et pour les PME travaillant avec des collaborateurs, fournisseurs ou travailleurs indépendants externes qui doivent pouvoir accéder à des ressources spécifiques de votre environnement, mais pas nécessairement à l’ensemble de l’infrastructure ou à des étendues liées à la facturation. Vous pouvez utiliser les fonctionnalités disponibles dans [Azure Active Directory B2B](../active-directory/external-identities/what-is-b2b.md) pour collaborer avec des utilisateurs invités externes. Vous pouvez aussi utiliser le contrôle d’accès en fonction du rôle (RBAC) pour n’accorder que les autorisations dont les utilisateurs invités ont besoin dans votre environnement.

## <a name="prerequisites"></a>Prérequis

Pour ajouter ou supprimer des attributions de rôles, vous devez disposer :

- d’autorisations `Microsoft.Authorization/roleAssignments/write` et `Microsoft.Authorization/roleAssignments/delete`, telles que [Administrateur de l’accès utilisateur](built-in-roles.md#user-access-administrator) ou [Propriétaire de l’accès utilisateur](built-in-roles.md#owner)

## <a name="when-would-you-invite-guest-users"></a>Quand voulez-vous convier des utilisateurs invités ?

Voici quelques exemples de scénarios où vous pourriez convier des utilisateurs invités à votre organisation pour leur octroyer des autorisations :

- Autoriser un fournisseur externe indépendant qui dispose uniquement d’un compte e-mail à accéder à vos ressources Azure dans le cadre d’un projet.
- Autoriser un partenaire externe à gérer certaines ressources ou un abonnement entier.
- Autoriser les ingénieurs du support technique ne faisant pas partie de votre organisation, par exemple les ingénieurs du support technique Microsoft, à accéder temporairement à vos ressources Azure pour résoudre des problèmes.

## <a name="permission-differences-between-member-users-and-guest-users"></a>Différences d’autorisation entre les utilisateurs membres et les utilisateurs invités

Les membres natifs d’un annuaire (utilisateurs membres) ont des autorisations différentes de celles des utilisateurs invités appartenant à un autre annuaire, comme un invité à la collaboration B2B (utilisateurs invités). Par exemple, les utilisateurs membres peuvent lire presque toutes les informations d’annuaire tandis que les utilisateurs invités disposent d’autorisations limitées sur l’annuaire. Pour plus d’informations sur les utilisateurs membres et les utilisateurs invités, consultez [Quelles sont les autorisations utilisateur par défaut dans Azure Active Directory ?](../active-directory/fundamentals/users-default-permissions.md)

## <a name="add-a-guest-user-to-your-directory"></a>Ajouter un utilisateur invité à votre annuaire

Effectuez les étapes suivantes pour ajouter un utilisateur invité à votre annuaire à l’aide de la page Azure Active Directory.

1. Assurez-vous que les paramètres de collaboration externe de votre organisation sont configurés de telle sorte que vous êtes autorisé à inviter des invités. Pour plus d’informations, consultez [Permettre une collaboration B2B externe et gérer les utilisateurs autorisés à en inviter d’autres](../active-directory/external-identities/delegate-invitations.md).

1. Dans le portail Azure, cliquez sur **Azure Active Directory** > **Utilisateurs** > **Nouvel utilisateur invité**.

    ![Fonctionnalité Nouvel utilisateur invité dans le portail Azure](./media/role-assignments-external-users/invite-guest-user.png)

1. Suivez les étapes pour ajouter un nouvel utilisateur invité. Pour plus d’informations, consultez [Ajouter des utilisateurs Azure Active Directory B2B Collaboration dans le portail Azure](../active-directory/external-identities/add-users-administrator.md#add-guest-users-to-the-directory).

Après avoir ajouté un utilisateur invité dans l’annuaire, vous pouvez envoyer à l’utilisateur invité un lien direct vers une application partagée, ou l’utilisateur invité peut cliquer sur l’URL d’échange dans l’e-mail d’invitation.

![E-mail d’invitation envoyé à un utilisateur invité](./media/role-assignments-external-users/invite-email.png)

Pour que l’utilisateur invité soit en mesure d’accéder à votre annuaire, il doit compléter le processus d’invitation.

![Révision des autorisations de l’utilisateur invité](./media/role-assignments-external-users/invite-review-permissions.png)

Pour plus d’informations sur le processus d’invitation, consultez [Utilisation d’invitations Azure Active Directory B2B Collaboration](../active-directory/external-identities/redemption-experience.md).

## <a name="add-a-role-assignment-for-a-guest-user"></a>Ajouter une attribution de rôle pour un utilisateur invité

Dans Azure RBAC, vous attribuez un rôle pour accorder un accès. Pour ajouter une attribution de rôle pour un utilisateur invité, vous suivez les [mêmes étapes](role-assignments-portal.md#add-a-role-assignment) que pour un utilisateur membre, un groupe, un principal du service ou une identité managée. Suivez ces étapes pour ajouter une attribution de rôle pour un utilisateur invité dans différentes étendues.

1. Dans le portail Azure, cliquez sur **Tous les services**.

1.  Sélectionnez l’ensemble de ressources auquel s’applique l’accès, également appelé étendue. Par exemple, vous pouvez sélectionner **Groupes d’administration**, **Abonnements**, **Groupes de ressources**, ou une ressource.

1. Cliquez sur la ressource spécifique.

1. Cliquez sur **Contrôle d’accès (IAM)** .

    La capture d’écran suivante montre un exemple de panneau Contrôle d’accès (IAM) pour un groupe de ressources. Si vous modifiez le contrôle d’accès ici, cette modification s’applique simplement au groupe de ressources.

    ![Panneau Contrôle d’accès (IAM) pour un groupe de ressources](./media/role-assignments-external-users/access-control-resource-group.png)

1. Cliquez sur l’onglet **Attributions de rôles** afin d’afficher toutes les attributions de rôles pour cette étendue.

1. Cliquez sur **Ajouter** > **Ajouter une attribution de rôle** pour ouvrir le volet Ajouter une attribution de rôle.

    Si vous n’avez pas les autorisations pour attribuer des rôles, l’option Ajouter une attribution de rôle sera désactivée.

    ![Menu Ajouter une attribution de rôle](./media/shared/add-role-assignment-menu.png)

    Le volet Ajouter une attribution de rôle s’ouvre.

1. Dans la liste déroulante **Rôle**, sélectionnez un rôle, tel que **Contributeur de machines virtuelles**.

1. Dans la liste **Sélectionner**, sélectionnez l’utilisateur invité. Si vous ne voyez pas le l’utilisateur dans la liste, vous pouvez saisir du texte dans la zone **Sélectionner** pour rechercher des noms d’affichage, des adresses e-mail et des identificateurs d’objet dans l’annuaire.

   ![Volet Ajouter une attribution de rôle](./media/role-assignments-external-users/add-role-assignment.png)

1. Cliquez sur **Enregistrer** pour attribuer le rôle à l’étendue sélectionnée.

    ![Attribution de rôle pour le contributeur de machine virtuelle](./media/role-assignments-external-users/access-control-role-assignments.png)

## <a name="add-a-role-assignment-for-a-guest-user-not-yet-in-your-directory"></a>Ajouter une attribution de rôle pour un utilisateur invité ne figurant pas encore dans votre annuaire

Pour ajouter une attribution de rôle pour un utilisateur invité, vous suivez les [mêmes étapes](role-assignments-portal.md#add-a-role-assignment) que pour un utilisateur membre, un groupe, un principal du service ou une identité managée.

Si l’utilisateur invité ne figure pas encore dans votre annuaire, vous pouvez l’inviter directement à partir du volet Ajouter une attribution de rôle.

1. Dans le portail Azure, cliquez sur **Tous les services**.

1.  Sélectionnez l’ensemble de ressources auquel s’applique l’accès, également appelé étendue. Par exemple, vous pouvez sélectionner **Groupes d’administration**, **Abonnements**, **Groupes de ressources**, ou une ressource.

1. Cliquez sur la ressource spécifique.

1. Cliquez sur **Contrôle d’accès (IAM)** .

1. Cliquez sur l’onglet **Attributions de rôles** afin d’afficher toutes les attributions de rôles pour cette étendue.

1. Cliquez sur **Ajouter** > **Ajouter une attribution de rôle** pour ouvrir le volet Ajouter une attribution de rôle.

    ![Menu Ajouter une attribution de rôle](./media/shared/add-role-assignment-menu.png)

    Le volet Ajouter une attribution de rôle s’ouvre.

1. Dans la liste déroulante **Rôle**, sélectionnez un rôle, tel que **Contributeur de machines virtuelles**.

1. Dans la liste **Sélectionner**, tapez l’adresse e-mail de la personne que vous voulez inviter, puis sélectionnez cette personne.

   ![Inviter un utilisateur invité dans le volet Ajouter une attribution de rôle](./media/role-assignments-external-users/add-role-assignment-new-guest.png)

1. Cliquez sur **Enregistrer** pour ajouter l’utilisateur invité à votre annuaire, attribuer le rôle, puis envoyer une invitation.

    Après quelques minutes, vous voyez une notification de l’attribution de rôle et des informations sur l’invitation.

    ![Notification d’attribution de rôle et d’utilisateur invité](./media/role-assignments-external-users/invited-user-notification.png)

1. Pour inviter manuellement l’utilisateur invité, cliquez avec le bouton droit et copiez le lien d’invitation inclus dans la notification. Ne cliquez pas sur le lien d’invitation, car cela démarre le processus d’invitation.

    Le lien d’invitation est au format suivant :

    `https://invitations.microsoft.com/redeem/...`

1. Envoyez le lien d’invitation à l’utilisateur invité pour qu’il termine le processus d’invitation.

    Pour plus d’informations sur le processus d’invitation, consultez [Utilisation d’invitations Azure Active Directory B2B Collaboration](../active-directory/external-identities/redemption-experience.md).

## <a name="remove-a-guest-user-from-your-directory"></a>Supprimer un utilisateur invité de votre annuaire

Avant de supprimer un utilisateur invité d’un annuaire, vous devez d’abord supprimer toutes ses attributions de rôle. Suivez ces étapes pour supprimer un utilisateur invité d’un annuaire.

1. Ouvrez **Contrôle d’accès (IAM)** dans une étendue, comme un groupe d’administration, un abonnement, un groupe de ressources ou une ressource, dans laquelle l’utilisateur invité a une attribution de rôle.

1. Cliquez sur l’onglet **Attributions de rôles** afin d’afficher toutes les attributions de rôle.

1. Dans la liste des attributions de rôle, ajoutez une coche en regard de l’utilisateur invité comportant l’attribution de rôle à supprimer.

   ![Supprimer une attribution de rôle](./media/role-assignments-external-users/remove-role-assignment-select.png)

1. Cliquez sur **Supprimer**.

   ![Supprimer le message d’attribution de rôle](./media/role-assignments-external-users/remove-role-assignment.png)

1. Dans le message d’attribution de rôle qui s’affiche, cliquez sur **Oui**.

1. Dans la barre de navigation gauche, cliquez sur **Azure Active Directory** > **Utilisateurs**.

1. Cliquez sur l’utilisateur invité à supprimer.

1. Cliquez sur **Supprimer**.

   ![Supprimer un utilisateur invité](./media/role-assignments-external-users/delete-guest-user.png)

1. Dans le message de suppression qui apparaît, cliquez sur **Yes** (Oui).

## <a name="troubleshoot"></a>Dépanner

### <a name="guest-user-cannot-browse-the-directory"></a>Impossible pour l’utilisateur invité de parcourir l’annuaire

Les utilisateurs invités disposent d'autorisations d'annuaire limitées. Par exemple, les utilisateurs invités ne peuvent pas parcourir l’annuaire ni rechercher des groupes ou des applications. Pour plus d’informations, consultez [Quelles sont les autorisations utilisateur par défaut dans Azure Active Directory ?](../active-directory/fundamentals/users-default-permissions.md)

![Impossible pour l’utilisateur invité de parcourir les utilisateurs figurant dans un annuaire](./media/role-assignments-external-users/directory-no-users.png)

Si un utilisateur invité a besoin de privilèges supplémentaires sur l’annuaire, vous pouvez lui attribuer un rôle d’annuaire. Si vous voulez vraiment qu’un utilisateur invité dispose d’un accès en lecture complet à votre annuaire, vous pouvez l’ajouter au rôle [Lecteurs de répertoire](../active-directory/roles/permissions-reference.md) dans Azure AD. Pour plus d’informations, consultez [Accorder des autorisations aux utilisateurs d’organisations partenaires dans votre locataire Azure Active Directory](../active-directory/external-identities/add-guest-to-role.md).

![Attribuer le rôle Lecteur de répertoire](./media/role-assignments-external-users/directory-roles.png)

### <a name="guest-user-cannot-browse-users-groups-or-service-principals-to-assign-roles"></a>Impossible pour l’utilisateur invité de parcourir les utilisateurs, les groupes ou les principaux du service pour attribuer des rôles

Les utilisateurs invités disposent d'autorisations d'annuaire limitées. Même si un utilisateur invité est [propriétaire](built-in-roles.md#owner) dans une étendue, s’il essaie d’ajouter une attribution de rôle pour accorder l’accès à une autre personne, il ne peut pas parcourir la liste des utilisateurs, des groupes ou des principaux du service.

![Impossible pour l’utilisateur invité de parcourir les principaux de sécurité pour attribuer des rôles](./media/role-assignments-external-users/directory-no-browse.png)

Si l’utilisateur invité connaît le nom de connexion exact d’une personne dans l’annuaire, il peut lui accorder l’accès. Si vous voulez vraiment qu’un utilisateur invité dispose d’un accès en lecture complet à votre annuaire, vous pouvez l’ajouter au rôle [Lecteurs de répertoire](../active-directory/roles/permissions-reference.md) dans Azure AD. Pour plus d’informations, consultez [Accorder des autorisations aux utilisateurs d’organisations partenaires dans votre locataire Azure Active Directory](../active-directory/external-identities/add-guest-to-role.md).

### <a name="guest-user-cannot-register-applications-or-create-service-principals"></a>Impossible pour l’utilisateur invité d’inscrire des applications ou de créer des principaux du service

Les utilisateurs invités disposent d'autorisations d'annuaire limitées. Si un utilisateur invité a besoin de pouvoir inscrire des applications ou de créer des principaux du service, vous pouvez l’ajouter au rôle [Développeur d’applications](../active-directory/roles/permissions-reference.md) dans Azure AD. Pour plus d’informations, consultez [Accorder des autorisations aux utilisateurs d’organisations partenaires dans votre locataire Azure Active Directory](../active-directory/external-identities/add-guest-to-role.md).

![Impossible pour l’utilisateur invité d’inscrire des applications](./media/role-assignments-external-users/directory-access-denied.png)

### <a name="guest-user-does-not-see-the-new-directory"></a>Nouvel annuaire invisible pour l’utilisateur invité

Si un utilisateur invité a reçu l’autorisation d’accéder à un nouvel annuaire, mais qu’il ne le voit pas listé dans le portail Azure quand il essaie de basculer vers son volet **Annuaire + abonnement**, vérifiez qu’il a bien achevé le processus d’invitation. Pour plus d’informations sur le processus d’invitation, consultez [Utilisation d’invitations Azure Active Directory B2B Collaboration](../active-directory/external-identities/redemption-experience.md).

### <a name="guest-user-does-not-see-resources"></a>Ressources invisibles pour l’utilisateur

Si un utilisateur invité a reçu l’autorisation d’accéder à un annuaire, mais qu’il ne voit pas les ressources auxquelles il est autorisé à accéder dans le portail Azure, vérifiez qu’il a sélectionné le bon annuaire. Un utilisateur invité peut avoir accès à plusieurs annuaires. Pour changer d’annuaire, dans le coin supérieur gauche, cliquez sur **Annuaire + abonnement**, puis cliquez sur l’annuaire approprié.

![Volet Annuaires + abonnements dans le portail Azure](./media/role-assignments-external-users/directory-subscription.png)

## <a name="next-steps"></a>Étapes suivantes

- [Ajouter des utilisateurs Azure Active Directory B2B Collaboration dans le Portail Azure](../active-directory/external-identities/add-users-administrator.md)
- [Propriétés d’un utilisateur Azure Active Directory B2B Collaboration](../active-directory/external-identities/user-properties.md)
- [Éléments de l’e-mail d’invitation de collaboration B2B - Azure Active Directory](../active-directory/external-identities/invitation-email-elements.md)
- [Ajouter un utilisateur invité en tant que coadministrateur](classic-administrators.md#add-a-guest-user-as-a-co-administrator)