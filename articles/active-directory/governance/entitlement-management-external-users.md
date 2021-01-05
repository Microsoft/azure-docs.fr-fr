---
title: Régir l’accès des utilisateurs externes dans la gestion des droits d’utilisation Azure AD – Azure Active Directory
description: Découvrez plus en détail les paramètres à spécifier pour régir l’accès des utilisateurs externes à la gestion des droits d'utilisation Azure Active Directory.
services: active-directory
documentationCenter: ''
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 12/23/2020
ms.author: barclayn
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: b356d5dff453b598eeb773af1a56fc50193e9e16
ms.sourcegitcommit: 6e2d37afd50ec5ee148f98f2325943bafb2f4993
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97746658"
---
# <a name="govern-access-for-external-users-in-azure-ad-entitlement-management"></a>Régir l’accès des utilisateurs externes dans la gestion des droits d’utilisation Azure AD

La gestion des droits d'utilisation Azure AD utilise [Azure AD B2B (Business-to-Business)](../external-identities/what-is-b2b.md) pour collaborer avec des personnes extérieures à votre organisation dans un autre annuaire. Avec Azure AD B2B, les utilisateurs externes s’authentifient auprès de leur annuaire de base, mais disposent d’une représentation dans votre annuaire. La représentation dans votre annuaire permet à l’utilisateur d’avoir accès à vos ressources.

Cet article décrit les paramètres que vous pouvez spécifier pour régir l’accès des utilisateurs externes.

## <a name="how-entitlement-management-can-help"></a>De quelle façon la gestion des droits d’utilisation peut aider

Lorsque vous utilisez l'expérience d'invitation [Azure AD B2B](../external-identities/what-is-b2b.md), vous devez déjà connaître les adresses e-mail des utilisateurs invités externes que vous souhaitez importer dans votre annuaire de ressources, et avec lesquels vous voulez collaborer. Cela fonctionne très bien lorsque vous travaillez sur un projet plus petit ou à court terme et que vous connaissez déjà tous les participants, mais s'avère plus complexe si vous souhaitez collaborer avec un grand nombre d'utilisateurs ou si les participants changent au fil du temps.  Par exemple, vous pouvez travailler avec une autre organisation et avoir un point de contact avec cette organisation, mais au fil du temps, d'autres utilisateurs de cette organisation auront également besoin d'un accès.

Avec la gestion des droits d'utilisation, vous pouvez définir une stratégie autorisant les utilisateurs des organisations que vous spécifiez à pouvoir demander eux-mêmes un package d'accès. Vous pouvez spécifier si une approbation est requise, ainsi qu'une date d’expiration de l’accès. Si une approbation est exigée, vous pouvez également inviter un ou plusieurs utilisateurs de l'organisation externe dans votre annuaire, et le ou les désigner comme approbateurs, car ils sont susceptibles de savoir quels utilisateurs externes de leur organisation ont besoin d'un accès. Une fois le package d'accès configuré, vous pouvez envoyer le lien de ce package à la personne à contacter (sponsor) au sein de l'organisation externe. Cette personne peut le partager avec d'autres utilisateurs de l'organisation externe et ceux-ci peuvent à leur tour utiliser ce lien pour demander le package d'accès. Les utilisateurs de cette organisation déjà invités dans votre répertoire peuvent également utiliser ce lien.

Lorsqu'une demande est approuvée, la gestion des droits d'utilisation accorde à l'utilisateur l'accès nécessaire, notamment une invitation si cet utilisateur n'est pas encore présent dans votre répertoire. Azure AD crée automatiquement un compte invité B2B pour cet utilisateur. Notez qu’un administrateur peut avoir précédemment restreint les organisations autorisées à collaborer, en définissant une [liste d'autorisations ou de refus B2B](../external-identities/allow-deny-list.md) pour autoriser ou refuser les invitations liées à d’autres organisations.  Si l’utilisateur ne figure pas sur la liste d'autorisations ou de refus, il ne sera pas invité.

Pour limiter dans le temps l'accès de l'utilisateur externe, vous spécifiez une date d'expiration dans la stratégie, comme 180 jours. Une fois ces 180 jours écoulés et en l'absence d’extension, la gestion des droits d'utilisateur supprime tous les accès associés à ce package d'accès. Par défaut, si l'utilisateur invité par le biais de la gestion des droits d'utilisation ne dispose d'aucune autre attribution de package d'accès, lorsqu'il perd sa dernière attribution, son compte invité est bloqué pendant 30 jours, puis supprimé. Et ce, afin d'éviter la prolifération de comptes inutiles. Comme décrit dans les sections suivantes, ces paramètres sont configurables.

## <a name="how-access-works-for-external-users"></a>Comment l’accès fonctionne pour les utilisateurs externes

Le diagramme et les étapes ci-dessous offrent une vue d’ensemble de la façon dont les utilisateurs externes sont autorisés à accéder à un package d’accès.

![Diagramme montrant le cycle de vie des utilisateurs externes](./media/entitlement-management-external-users/external-users-lifecycle.png)

1. Vous [ajoutez une organisation connectée](entitlement-management-organization.md) pour le répertoire ou domaine Azure AD avec lequel vous souhaitez collaborer.

1. Dans votre annuaire, vous créez un package d’accès qui comprend une stratégie [Pour les utilisateurs qui ne sont pas dans votre annuaire](entitlement-management-access-package-create.md#for-users-not-in-your-directory).

1. Vous envoyez un [lien du portail Mon accès](entitlement-management-access-package-settings.md) à votre contact, à l’organisation externe, afin qu’il puisse le partager avec ses utilisateurs qui veulent demander le package d’accès.

1. Un utilisateur externe (**Demandeur A** dans cet exemple) utilise le lien vers le portail Mon accès pour [demander l’accès](entitlement-management-request-access.md) au package d’accès. La façon dont l’utilisateur se connecte dépend du type d’authentification du répertoire ou domaine défini dans l’organisation connectée.

1. Un approbateur [approuve la demande](entitlement-management-request-approve.md) (ou la demande est approuvée automatiquement).

1. La demande passe à l’[état de remise en cours](entitlement-management-process.md).

1. À l’aide du processus d’invitation B2B, un compte d’utilisateur invité est créé dans votre annuaire : **Demandeur A (Invité)**  dans cet exemple. Si une [liste verte ou une liste d’exclusion](../external-identities/allow-deny-list.md) est définie, le paramètre de liste est appliqué.

1. L’utilisateur invité a reçu l’accès à toutes les ressources du package d’accès. Les modifications apportées à Azure AD et à d’autres services Microsoft Online Services, ou à des applications SaaS connectées, peuvent prendre un certain temps. Pour plus d’informations, consultez [Lorsque les modifications sont appliquées](entitlement-management-access-package-resources.md#when-changes-are-applied).

1. L’utilisateur externe reçoit un e-mail l’informant que son accès a été [autorisé](entitlement-management-process.md).

1. Pour accéder aux ressources, l’utilisateur externe peut cliquer sur le lien dans l’e-mail, ou tenter d’accéder directement à une des ressources de l’annuaire pour terminer le processus d’invitation.

1. Selon les paramètres de stratégie, au fil du temps, l’attribution du package d’accès à l’utilisateur externe expire, et l’accès de l’utilisateur externe est supprimé.

1. En fonction du cycle de vie des paramètres des utilisateurs externes, lorsque l’utilisateur externe n’a plus d’attribution de package d’accès, il ne peut plus se connecter et le compte d’utilisateur invité est supprimé de votre annuaire.

## <a name="settings-for-external-users"></a>Paramètres pour les utilisateurs externes

Pour vous assurer que les personnes extérieures à votre organisation peuvent demander des packages d’accès et obtenir l’accès aux ressources de ces packages d’accès, vous devez vérifier si certains paramètres sont correctement configurés.

### <a name="enable-catalog-for-external-users"></a>Activer le catalogue pour les utilisateurs externes

- Par défaut, lorsque vous créez un [nouveau catalogue](entitlement-management-catalog-create.md), il est activé pour autoriser les utilisateurs externes à demander des packages d’accès dans le catalogue. Assurez-vous que le paramètre **Activé pour les utilisateurs externes** est défini sur **Oui**.

    ![Modifier les paramètres du catalogue](./media/entitlement-management-shared/catalog-edit.png)

### <a name="configure-your-azure-ad-b2b-external-collaboration-settings"></a>Configurer vos paramètres de collaboration B2B externe Azure AD

- Autoriser les invités à inviter d’autres invités dans votre répertoire signifie que des invitations d’invités peuvent se produire en dehors de la gestion des droits d’utilisation. Nous vous recommandons de définir **Les invités peuvent inviter** sur **Non** pour autoriser uniquement les invitations correctement régies.
- Si vous utilisez la liste verte B2B, vous devez vous assurer que tous les domaines que vous souhaitez associer à la gestion des droits d’utilisation sont ajoutés à la liste. De même, si vous utilisez la liste d’exclusion B2B, vous devez vous assurer que tous les domaines auxquels vous souhaitez vous associer ne sont pas ajoutés à la liste.
- Si vous créez une stratégie de gestion des droits d'utilisation pour **Tous les utilisateurs** (toutes les organisations connectées + tous les nouveaux utilisateurs externes) et qu’un utilisateur n’appartient pas à une organisation connectée dans votre annuaire, une organisation connectée est automatiquement créée pour celui-ci quand il demande le package. Tout paramètre de liste d’autorisation ou de refus B2B que vous avez est prioritaire. Par conséquent, veillez à inclure les domaines que vous prévoyez d’inclure dans cette stratégie dans votre liste verte si vous en utilisez une, et excluez-les de votre liste d’exclusion si vous utilisez une liste d’exclusion.
- Si vous souhaitez créer une stratégie de gestion des droits d’utilisation incluant **Tous les utilisateurs** (toutes les organisations connectées + tous les nouveaux utilisateurs externes), vous devez d’abord activer l’authentification par code secret à usage unique d’e-mail pour votre répertoire. Pour plus d’informations, consultez [Authentification par code secret à usage unique d’e-mail (préversion)](../external-identities/one-time-passcode.md).
- Pour plus d’informations sur les paramètres de collaboration B2B externe Azure AD, consultez [Permettre une collaboration B2B externe et gérer les utilisateurs autorisés à en inviter d’autres](../external-identities/delegate-invitations.md).

    ![Paramètres de collaboration externe Azure AD](./media/entitlement-management-external-users/collaboration-settings.png)

### <a name="review-your-conditional-access-policies"></a>Passer en revue vos stratégies d’accès conditionnel

- Veillez à exclure les invités des stratégies d’accès conditionnel que les nouveaux utilisateurs invités ne seront pas en mesure de respecter, car cela les empêchera de se connecter à votre répertoire. Par exemple, les invités n’ont probablement pas d’appareil inscrit, ne se trouvent pas à un emplacement connu et ne veulent pas se réinscrire pour l’authentification multifacteur (MFA). Par conséquent, l’ajout de ces exigences dans une stratégie d’accès conditionnel empêchera les invités d’utiliser la gestion des droits d’utilisation. Pour plus d’informations, consultez [Que sont les conditions dans l’accès conditionnel Azure Active Directory ?](../conditional-access/concept-conditional-access-conditions.md).

    ![Paramètres d’exclusion des stratégies d’accès conditionnel Azure AD](./media/entitlement-management-external-users/conditional-access-exclude.png)

### <a name="review-your-sharepoint-online-external-sharing-settings"></a>Passer en revue vos paramètres de partage externe SharePoint Online

- Si vous souhaitez inclure des sites SharePoint Online dans vos packages d’accès pour des utilisateurs externes, assurez-vous que votre paramètre de partage externe au niveau de l’organisation est défini sur **Tout le monde** (les utilisateurs n’ont pas besoin de se connecter) ou **Invités nouveaux et existants** (les invités doivent se connecter ou fournir un code de vérification). Pour plus d’informations, consultez [Activer ou désactiver le partage externe](/sharepoint/turn-external-sharing-on-or-off#change-the-organization-level-external-sharing-setting).

- Si vous souhaitez restreindre le partage externe en dehors de la gestion des droits d’utilisation, vous pouvez définir le paramètre de partage externe sur **Invités existants**. Ensuite, seuls les nouveaux utilisateurs qui sont invités par l’intermédiaire de la gestion des droits d’utilisation pourront accéder à ces sites. Pour plus d’informations, consultez [Activer ou désactiver le partage externe](/sharepoint/turn-external-sharing-on-or-off#change-the-organization-level-external-sharing-setting).

- Assurez-vous que les paramètres au niveau du site autorisent l’accès invité (mêmes options que celles précédemment listées). Pour plus d’informations, consultez [Activer ou désactiver le partage externe pour un site](/sharepoint/change-external-sharing-site).

### <a name="review-your-microsoft-365-group-sharing-settings"></a>Examinez vos paramètres de partage de groupe Microsoft 365

- Si vous souhaitez inclure des groupes Microsoft 365 dans vos packages d’accès pour des utilisateurs externes, assurez-vous que le paramètre **Permettre aux utilisateurs d’ajouter de nouveaux invités à l’organisation** est défini sur **Activé** pour autoriser l’accès invité. Pour plus d’informations, consultez [Gérer l’accès invité à des groupes Microsoft 365](/Microsoft 365/admin/create-groups/manage-guest-access-in-groups?view=Microsoft 365-worldwide#manage-groups-guest-access).

- Si vous souhaitez que les utilisateurs externes puissent accéder au site SharePoint Online et aux ressources associées à un groupe Microsoft 365, veillez à activer le partage externe SharePoint Online. Pour plus d’informations, consultez [Activer ou désactiver le partage externe](/sharepoint/turn-external-sharing-on-or-off#change-the-organization-level-external-sharing-setting).

- Pour plus d’informations sur la façon de définir la stratégie d’invité pour les groupes Microsoft 365 au niveau du répertoire dans PowerShell, consultez [Exemple : Configurer une stratégie d’invité pour les groupes au niveau du répertoire](../enterprise-users/groups-settings-cmdlets.md#example-configure-guest-policy-for-groups-at-the-directory-level).

### <a name="review-your-teams-sharing-settings"></a>Passer en revue vos paramètres de partage des équipes

- Si vous souhaitez inclure des équipes dans vos packages d’accès pour des utilisateurs externes, assurez-vous que le paramètre **Autoriser l’accès invité dans Microsoft Teams** est défini sur **Activé** pour autoriser l’accès invité. Pour plus d’informations, consultez [Configurer l’accès invité dans le centre d’administration Microsoft Teams](/microsoftteams/set-up-guests#configure-guest-access-in-the-teams-admin-center).

## <a name="manage-the-lifecycle-of-external-users"></a>Gérer le cycle de vie des utilisateurs externes

Vous pouvez sélectionner ce qui se passe lorsqu’un utilisateur externe, qui a été invité à accéder à votre annuaire par le biais d’une demande de package d’accès en cours d’approbation, ne dispose plus d’attributions de package d’accès. Cela peut se produire si l’utilisateur abandonne toutes les attributions de package d’accès, ou si l’attribution de son dernier package d’accès arrive à expiration. Par défaut, lorsqu’un utilisateur externe n’a plus d’attributions de package d’accès, il ne peut pas se connecter à votre annuaire. Au bout de 30 jours, son compte d’utilisateur invité est supprimé de votre annuaire.

**Rôle prérequis :** Administrateur général ou Administrateur d’utilisateurs

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, dans la section **Gestion des droits d’utilisation**, cliquez sur **Paramètres**.

1. Cliquez sur **Modifier**.

    ![Paramètres de la gestion du cycle de vie des utilisateurs externes](./media/entitlement-management-external-users/settings-external-users.png)

1. Dans la section **Gérer le cycle de vie des utilisateurs externes**, sélectionnez les différents paramètres pour les utilisateurs externes.

1. Si, lorsqu’un utilisateur externe perd sa dernière attribution aux packages d’accès, vous souhaitez l’empêcher de se connecter à cet annuaire, définissez **Empêcher l'utilisateur externe de se connecter à cet annuaire** sur **Oui**.

    > [!NOTE]
    > Si un utilisateur n’est pas autorisé à se connecter à ce répertoire, il ne peut pas redemander le package d’accès ou demander un accès supplémentaire dans ce répertoire. Ne configurez pas le blocage de leur connexion s’ils doivent par la suite demander l’accès à d’autres packages d’accès.

1. Si, lorsqu’un utilisateur externe perd sa dernière attribution aux packages d’accès, vous souhaitez supprimer son compte d’utilisateur invité dans ce répertoire, définissez **Supprimer l’utilisateur externe**  sur **Oui**.

    > [!NOTE]
    > La gestion des droits d'utilisation supprime uniquement les comptes qui ont été invités par l’intermédiaire de la gestion des droits d'utilisation. Notez également qu’un utilisateur ne pourra pas se connecter et sera supprimé de ce répertoire, même s’il a été ajouté aux ressources du répertoire qui n’étaient pas des attributions de packages d’accès. Si l’invité était présent dans ce répertoire avant de recevoir des attributions de package d’accès, il sera conservé. Toutefois, s’il a été invité par le biais d’une attribution de package d’accès, et qu’après avoir été invité il a également été affecté à un site OneDrive Entreprise ou SharePoint Online, il sera toujours supprimé.

1. Si vous souhaitez supprimer le compte d’utilisateur invité du répertoire, vous pouvez définir le nombre de jours avant sa suppression. Si vous souhaitez supprimer le compte d’utilisateur invité dès qu’il perd la dernière attribution à un package d’accès, définissez **Nombre de jours avant la suppression de l’utilisateur externe de cet annuaire**  sur **0**.

1. Cliquez sur **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes

- [Ajouter une organisation connectée](entitlement-management-organization.md)
- [Pour les utilisateurs qui ne sont pas dans votre annuaire](entitlement-management-access-package-request-policy.md#for-users-not-in-your-directory)
- [Résolution des problèmes](entitlement-management-troubleshoot.md)