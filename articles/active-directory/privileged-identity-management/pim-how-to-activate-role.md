---
title: Activer mes rôles Azure AD dans PIM - Azure Active Directory | Microsoft Docs
description: Découvrez comment activer des rôles Azure AD dans Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 11/18/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 079dcdc9d1667809779489c5d1126bc85cafb881
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96511312"
---
# <a name="activate-my-azure-ad-roles-in-pim"></a>Activer mes rôles Azure AD dans PIM

Le service Azure Active Directory (Azure AD) Privileged Identity Management (PIM) simplifie la gestion par les entreprises de l’accès privilégié aux ressources dans Azure AD et d’autres services en ligne Microsoft, tels que Microsoft 365 ou Microsoft Intune.  

Si vous êtes *éligible* à un rôle d’administrateur, vous devez *activer* ce rôle quand vous devez effectuer des actions privilégiées. Par exemple, si vous gérez occasionnellement des fonctionnalités de Microsoft 365, les administrateurs de rôle privilégié de votre organisation peuvent ne pas vous attribuer un rôle d’administrateur général permanent, étant donné que ce rôle influe également sur les autres services. Au lieu de cela, ils peuvent vous attribuer des rôles Azure AD tels qu’administrateur Exchange Online. Vous pouvez faire une demande pour activer ce rôle lorsque vous avez besoin de ses privilèges. Vous aurez ainsi le contrôle d’administrateur pendant une période prédéterminée.

Cet article est destiné aux administrateurs qui doivent activer leur rôle Azure AD dans Privileged Identity Management.

## <a name="determine-your-version-of-pim"></a>Déterminer votre version de PIM

Depuis novembre 2019, la partie Rôles Azure AD de Privileged Identity Management est mise à jour vers une nouvelle version qui correspond à l’expérience des rôles de ressources Azure. Cela permet de créer des fonctionnalités supplémentaires et d’apporter des [modifications à l’API existante](azure-ad-roles-features.md#api-changes). Pendant que la nouvelle version est déployée, les procédures que vous suivez dans cet article dépendent de la version de Privileged Identity Management que vous possédez actuellement. Pour déterminer la version de Privileged Identity Management dont vous disposez, procédez de la manière décrite dans cette section. Une fois que vous connaissez votre version de Privileged Identity Management, vous pouvez sélectionner les procédures de cet article qui correspondent à cette version.

1. Connectez-vous au [Portail Azure](https://portal.azure.com/) à l’aide du rôle [Administrateur de rôle privilégié](../roles/permissions-reference.md#privileged-role-administrator).
1. Ouvrez **Azure AD Privileged Identity Management**. Si une bannière figure en haut de la page de présentation, suivez les instructions sous l’onglet **Nouvelle version** de cet article. Sinon, suivez les instructions sous l’onglet **Version précédente**.

    [![Select Azure AD > Privileged Identity Management.](media/pim-how-to-add-role-to-user/pim-new-version.png)](media/pim-how-to-add-role-to-user/pim-new-version.png#lightbox)

# <a name="new-version"></a>[Nouvelle version](#tab/new)

## <a name="activate-a-role-for-new-version"></a>Activer un rôle pour la nouvelle version

Lorsque vous devez assumer un rôle Azure AD, vous pouvez demander une activation en ouvrant **Mes rôles** dans Privileged Identity Management.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Ouvrez **Azure AD Privileged Identity Management**. Pour plus d’informations sur la façon d’ajouter la mosaïque Privileged Identity Management à votre tableau de bord, consultez [Commencer à utiliser Privileged Identity Management](pim-getting-started.md).

1. Sélectionnez **Mes rôles**, puis **Rôles Azure AD** pour afficher la liste de vos rôles Azure AD éligibles.

    ![Page Mes rôles montrant les rôles que vous pouvez activer](./media/pim-how-to-activate-role/my-roles.png)

1. Dans la liste **Rôles Azure AD**, recherchez le rôle que vous souhaitez activer.

    ![Rôles Azure AD : liste Mes rôles éligibles](./media/pim-how-to-activate-role/activate-link.png)

1. Sélectionnez **Activer** pour ouvrir le volet Activer.

    ![Rôles Azure AD : la page d’activation contient la durée et l’étendue](./media/pim-how-to-activate-role/activate-page.png)

1. Sélectionnez **Vérification supplémentaire requise**** et suivez les instructions pour effectuer une vérification de sécurité supplémentaire. Vous ne pouvez vous authentifier qu’une seule fois par session.

    ![Écran de vérification de sécurité permettant la saisie d’un code secret, par exemple](./media/pim-resource-roles-activate-your-roles/resources-mfa-enter-code.png)

1. Après l’authentification multifacteur, sélectionnez **Activer avant de continuer**.

    ![Vérifier mon identité à l’aide de l’authentification multifacteur avant activation du rôle](./media/pim-how-to-activate-role/activate-role-mfa-banner.png)

1. Si vous souhaitez spécifier une étendue réduite, sélectionnez **Étendue** pour ouvrir le volet des filtres. Dans le volet des filtres, vous pouvez spécifier les ressources Azure AD auxquelles vous devez accéder. Il est recommandé de demander l’accès aux seules ressources dont vous avez besoin.

1. Si nécessaire, spécifiez une heure de début personnalisée pour l’activation. Le rôle Azure AD sera activé après l’heure sélectionnée.

1. Dans la zone de texte **Raison**, entrez le motif de la demande d’activation.

1. Sélectionnez **Activer**.

    Si [l’activation du rôle nécessite une approbation](pim-resource-roles-approval-workflow.md), une notification s’affiche dans le coin supérieur droit de votre navigateur pour vous informer que la demande est en attente d’approbation.

    ![La requête d’activation est en attente de la notification d’approbation](./media/pim-resource-roles-activate-your-roles/resources-my-roles-activate-notification.png)

## <a name="view-the-status-of-your-requests-for-new-version"></a>Afficher l’état de vos requêtes pour la nouvelle version

Vous pouvez afficher l’état de vos demandes d’activation en attente.

1. Ouvrez Azure AD Privileged Identity Management.

1. Sélectionnez **Mes demandes** pour afficher la liste des demandes de vos rôles Azure AD et de rôles de ressources Azure.

    ![Mes demandes : page Azure AD montrant vos demandes en attente](./media/pim-how-to-activate-role/my-requests-page.png)

1. Faites défiler vers la droite pour afficher la colonne **État de la demande**.

## <a name="cancel-a-pending-request-for-new-version"></a>Annuler une demande en attente pour la nouvelle version

Si vous n’avez pas besoin de l’activation d’un rôle nécessitant une approbation, vous pouvez annuler une demande en attente à tout moment.

1. Ouvrez Azure AD Privileged Identity Management.

1. Sélectionnez **Mes demandes**.

1. Pour le rôle que vous souhaitez annuler, sélectionnez le lien **Annuler**.

    Lorsque vous sélectionnez Annuler, la requête est annulée. Pour réactiver le rôle, vous devez envoyer une nouvelle demande d’activation.

   ![Ma liste de demandes dans laquelle l’action d’annulation est mise en surbrillance](./media/pim-resource-roles-activate-your-roles/resources-my-requests-cancel.png)

## <a name="troubleshoot-for-new-version"></a>Résoudre les problèmes pour la nouvelle version

### <a name="permissions-are-not-granted-after-activating-a-role"></a>Les autorisations ne sont pas accordées après l’activation d’un rôle

Quand vous activez un rôle dans Privileged Identity Management, il est possible que l’activation ne se propage pas instantanément à tous les portails qui nécessitent le rôle privilégié. Parfois, même si la modification est propagée, son application immédiate peut être empêchée par la mise en cache web dans un portail. Si l’activation est retardée, voici les étapes à suivre.

1. Déconnectez-vous du portail Microsoft Azure, puis reconnectez-vous.

1. Dans Privileged Identity Management, vérifiez que vous êtes répertorié en tant que membre du rôle.

# <a name="previous-version"></a>[Version précédente](#tab/previous)

## <a name="activate-a-role-previous-version"></a>Activer un rôle (version précédente)

Lorsque vous avez besoin d’endosser un rôle Azure AD, vous pouvez demander une activation à l’aide de l’option de la navigation **Mes rôles** dans Privileged Identity Management.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Ouvrez **Azure AD Privileged Identity Management**. Pour plus d’informations sur la façon d’ajouter la mosaïque Privileged Identity Management à votre tableau de bord, consultez [Commencer à utiliser Privileged Identity Management](pim-getting-started.md).

1. Sélectionnez des **rôles Azure AD**.

1. Sélectionnez **Mes rôles** pour afficher la liste de vos rôles Azure AD éligibles.

    ![Rôles Azure AD - Liste Mes rôles montrant la liste des rôles éligibles ou actifs](./media/pim-how-to-activate-role/directory-roles-my-roles.png)

1. Recherchez un rôle à activer.

    ![Rôles Azure AD - Liste Mes rôles éligibles montrant le lien Activer](./media/pim-how-to-activate-role/directory-roles-my-roles-activate.png)

1. Sélectionnez **Activer** pour ouvrir le volet où se trouvent les informations d’activation du rôle.

1. Si votre rôle exige une authentification multifacteur (MFA), sélectionnez **Vérifier votre identité avant de continuer**. Vous ne devez vous authentifier qu’une seule fois par session.

    ![Volet de vérification de votre identité avec l’authentification multifacteur avant l’activation du rôle](./media/pim-how-to-activate-role/directory-roles-my-roles-mfa.png)

1. Sélectionnez **Vérifier mon identité** et suivez les instructions pour effectuer une vérification de sécurité supplémentaire.

    ![Page de vérification de sécurité supplémentaire vous demandant comment vous contacter](./media/pim-how-to-activate-role/additional-security-verification.png)

1. Sélectionnez **Activer** pour ouvrir le volet Activation.

    ![Volet d’activation spécifiant l’heure de début, la durée, le ticket et la raison](./media/pim-how-to-activate-role/directory-roles-activate.png)

1. Si nécessaire, spécifiez une heure de début personnalisée pour l’activation.

1. Spécifiez la durée de l’activation.

1. Dans la zone de texte **Raison de l’activation**, entrez le motif de la demande d’activation. Certains rôles vous obligent à fournir un numéro de ticket d’incident.

    ![Volet de fin de l’activation spécifiant l’heure de début, la durée, la raison et le ticket personnalisés](./media/pim-how-to-activate-role/directory-roles-activation-pane.png)

1. Sélectionnez **Activer**.

    Si le rôle ne nécessite pas d’approbation, un volet **État d’activation** s’affiche. Il présente l’état de l’activation.

    ![Page d’état de l’activation montrant les trois étapes d’activation](./media/pim-how-to-activate-role/activation-status.png)

    Une fois toutes les étapes terminées, sélectionnez le lien **Se déconnecter** pour vous déconnecter du portail Azure. Vous pouvez maintenant utiliser le rôle lors de votre prochaine connexion.

    Si [l’activation du rôle nécessite une approbation](./azure-ad-pim-approval-workflow.md), une notification Azure s’affiche dans le coin supérieur droit de votre navigateur pour vous informer que la demande est en attente d’approbation.

## <a name="view-the-status-of-your-requests-previous-version"></a>Afficher l’état de vos demandes (version précédente)

Vous pouvez afficher l’état de vos demandes d’activation en attente.

1. Ouvrez Azure AD Privileged Identity Management.

1. Sélectionnez des **rôles Azure AD**.

1. Sélectionnez **Mes demandes** pour afficher la liste de vos demandes.

    ![Rôles Azure AD - Liste Mes requêtes](./media/pim-how-to-activate-role/directory-roles-my-requests.png)

## <a name="deactivate-a-role-previous-version"></a>Désactiver un rôle (version précédente)

Une fois qu’un rôle a été activé, il se désactive automatiquement quand sa limite de temps (durée éligible) est atteinte.

Si vous terminez vos tâches d’administration plus tôt que prévu, vous pouvez également désactiver un rôle manuellement dans Azure AD Privileged Identity Management.

1. Ouvrez Azure AD Privileged Identity Management.

1. Sélectionnez des **rôles Azure AD**.

1. Sélectionnez **Mes rôles**.

1. Sélectionnez **Rôles actifs** pour afficher la liste des rôles actifs.

1. Accédez au rôle dont vous n’avez plus besoin, puis sélectionnez **Désactiver**.

## <a name="cancel-a-pending-request-previous-version"></a>Annuler une demande en attente (version précédente)

Si vous n’avez pas besoin de l’activation d’un rôle nécessitant une approbation, vous pouvez annuler une demande en attente à tout moment.

1. Ouvrez Azure AD Privileged Identity Management.

1. Sélectionnez des **rôles Azure AD**.

1. Sélectionnez **Mes demandes**.

1. Pour le rôle que vous souhaitez annuler, sélectionnez le bouton **Annuler**.

    Lorsque vous sélectionnez **Annuler**, la requête est annulée. Pour réactiver le rôle, vous devez envoyer une nouvelle demande d’activation.

   ![Liste Mes demandes, avec le bouton Annuler en surbrillance](./media/pim-how-to-activate-role/directory-role-cancel.png)

## <a name="troubleshoot-previous-version"></a>Résoudre les problèmes (version précédente)

### <a name="permissions-are-not-granted-after-activating-a-role"></a>Les autorisations ne sont pas accordées après l’activation d’un rôle

Quand vous activez un rôle dans Privileged Identity Management, il est possible que l’activation ne se propage pas instantanément à tous les portails qui nécessitent le rôle privilégié. Parfois, même si la modification est propagée, son application immédiate peut être empêchée par la mise en cache web dans un portail. Si l’activation est retardée, voici les étapes à suivre.

1. Déconnectez-vous du portail Microsoft Azure, puis reconnectez-vous.

    Lorsque vous activez un rôle Azure AD, vous voyez apparaître les phases de l’activation. Une fois toutes les phases terminées, un lien **Se déconnecter** s’affiche. Vous pouvez utiliser ce lien pour vous déconnecter. Cette procédure résout la plupart des cas de retard d’activation.

1. Dans Privileged Identity Management, vérifiez que vous êtes répertorié en tant que membre du rôle.

 ---

## <a name="next-steps"></a>Étapes suivantes

- [Activer mes rôles Azure AD dans Privileged Identity Management](pim-how-to-activate-role.md)
