---
title: Changer les rôles de ressources pour un package d’accès dans la gestion des droits d’utilisation d’Azure Active Directory - Azure Active Directory
description: Découvrez comment changer les rôles de ressources pour un package d’accès existant dans la gestion des droits d’utilisation d’Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1b6e2ac9d80c1c3bf76b4a3d4c44f0654100670f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89567416"
---
# <a name="change-resource-roles-for-an-access-package-in-azure-ad-entitlement-management"></a>Changer les rôles de ressources pour un package d’accès dans la gestion des droits d’utilisation d’Azure Active Directory

En tant que gestionnaire des packages d'accès, vous pouvez à tout moment modifier les ressources d'un package d'accès sans vous soucier de donner à l'utilisateur l’accès aux nouvelles ressources ou de supprimer son accès aux ressources précédentes. Cet article explique comment changer les rôles de ressources pour un package d’accès existant.

Cette vidéo présente une vue d’ensemble du changement d’un package d’accès.

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE3LD4Z]

## <a name="check-catalog-for-resources"></a>Rechercher des ressources dans le catalogue

Si vous devez ajouter des ressources à un package d’accès, vérifiez si les ressources dont vous avez besoin sont disponibles dans le catalogue. Si vous êtes gestionnaire de package d’accès, vous ne pouvez pas ajouter de ressources à un catalogue, même si vous êtes propriétaire de celles-ci. Vous ne pouvez utiliser que les ressources disponibles dans le catalogue.

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs ou gestionnaire de package d’accès

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Catalogue**, puis ouvrez le catalogue.

1. Dans le menu de gauche, cliquez sur **Resources** pour afficher la liste des ressources de ce catalogue.

    ![Liste des ressources d’un catalogue](./media/entitlement-management-access-package-resources/catalog-resources.png)

1. Si vous êtes gestionnaire de package d’accès et que vous devez ajouter des ressources au catalogue, vous pouvez demander au propriétaire du catalogue de les ajouter.

## <a name="add-resource-roles"></a>Ajouter des rôles de ressources

Un rôle de ressources est un ensemble d’autorisations associées à une ressource. Pour mettre des ressources à la disposition des utilisateurs, ajoutez des rôles de ressources à votre package d'accès. Vous pouvez ajouter des rôles de ressources pour des groupes, des équipes, des applications et des sites SharePoint.

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs ou gestionnaire de package d’accès

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Packages d’accès**, puis ouvrez le package d'accès.

1. Dans le menu de gauche, cliquez sur **Rôles de ressources**.

1. Cliquez sur **Ajouter des rôles de ressources** pour ouvrir la page Ajouter des rôles de ressources à un package d'accès.

    ![Package d'accès - Ajouter des rôles de ressources](./media/entitlement-management-access-package-resources/resource-roles-add.png)

1. Selon ce que vous souhaitez ajouter, un groupe, une équipe, une application ou un site SharePoint, procédez de la manière décrite ci-après, dans l’une des sections dédiées aux rôles de ressources.

## <a name="add-a-group-or-team-resource-role"></a>Ajouter un rôle de ressources de groupe ou d’équipe

Vous pouvez configurer la gestion des droits d’utilisation afin d’ajouter automatiquement des utilisateurs à un groupe ou à une équipe dans Microsoft Teams, quand un package d’accès leur est attribué. 

- Si un groupe ou une équipe fait partie d’un package d’accès et qu’un utilisateur est affecté à ce package d’accès, l’utilisateur est ajouté à ce groupe ou à cette équipe, s’il n’y figure pas.
- À l’expiration de l’affectation du package d’accès d’un utilisateur, ce dernier est retiré du groupe ou de l’équipe, sauf si un autre package d’accès incluant ce même groupe ou cette même équipe lui est actuellement affecté.

Vous pouvez sélectionner n’importe quel [groupe de sécurité Azure AD ou groupe Microsoft 365](../fundamentals/active-directory-groups-create-azure-portal.md). Les administrateurs peuvent ajouter n'importe quel groupe à un catalogue ; les propriétaires de catalogue peuvent ajouter n'importe quel groupe au catalogue si ce groupe leur appartient. Gardez à l'esprit les contraintes Azure AD suivantes lors de la sélection d'un groupe :

- Si un utilisateur, y compris un invité, est ajouté en tant que membre à un groupe ou à une équipe, il peut voir tous les autres membres de ce groupe ou de cette équipe.
- Azure AD ne peut pas modifier l'appartenance d'un groupe qui a été synchronisé à partir de Windows Server Active Directory à l’aide d’Azure AD Connect ou a été créé dans Exchange Online en tant que groupe de distribution.  
- L'adhésion à des groupes dynamiques ne peut pas être mise à jour par l'ajout ou la suppression d'un membre, de sorte que l'adhésion à des groupes dynamiques ne convient pas pour la gestion des droits d'utilisation.

Pour plus d’informations, consultez [Comparer les groupes](/office365/admin/create-groups/compare-groups) et [Groupes Microsoft 365 et Microsoft Teams](/microsoftteams/office-365-groups).

1. Sur la page **Ajouter des rôles de ressources à un package d’accès**, cliquez sur **Groupes et équipes** pour ouvrir le volet Sélectionner les groupes.

1. Sélectionnez les groupes et équipes que vous voulez inclure dans le package d’accès.

    ![Package d'accès - Ajouter des rôles de ressources - Sélectionner les groupes](./media/entitlement-management-access-package-resources/group-select.png)

1. Cliquez sur **Sélectionner**.

    Une fois l’équipe ou le groupe sélectionné, la colonne **Sous-type** affiche l’un des sous-types suivants :

    | Sous-type | Description |
    | --- | --- |
    | Sécurité | Utilisé pour octroyer l’accès à des ressources. |
    | Distribution | Utilisé pour envoyer des notifications à un groupe de personnes. |
    | Microsoft 365 | Groupe Microsoft 365 non activé pour Teams. Utilisé pour la collaboration entre les utilisateurs, à l’intérieur et à l’extérieur de votre organisation. |
    | Équipe | Groupe Microsoft 365 activé pour Teams. Utilisé pour la collaboration entre les utilisateurs, à l’intérieur et à l’extérieur de votre organisation. |

1. Dans la liste **Rôle**, Sélectionnez **Propriétaire** ou **Membre**.

    En règle générale, sélectionnez le rôle Membre. Si vous sélectionnez le rôle Propriétaire, les utilisateurs pourront ajouter ou supprimer d'autres membres ou propriétaires.

    ![Package d’accès – Ajouter un rôle de ressources pour un groupe ou une équipe](./media/entitlement-management-access-package-resources/group-role.png)

1. Cliquez sur **Add**.

    Tous les utilisateurs ayant déjà des affectations au package d’accès deviennent automatiquement membres de ce groupe ou de cette équipe lors de l’ajout.

## <a name="add-an-application-resource-role"></a>Ajouter un rôle de ressources d'application

Vous pouvez demander à Azure AD d'accorder automatiquement aux utilisateurs l'accès à une application d'entreprise Azure AD, y compris les applications SaaS ainsi que les applications de votre organisation fédérées sur Azure AD, lorsqu'un package d'accès est affecté à un utilisateur. Pour les applications qui s'intègrent à Azure AD par le biais de l'authentification unique fédérée, Azure AD émettra des jetons de fédération pour les utilisateurs affectés à l'application.

Les applications peuvent avoir plusieurs rôles. Lorsque vous ajoutez une application à un package d'accès, si cette application comporte plusieurs rôles, vous devez spécifier le rôle approprié pour ces utilisateurs. Si vous développez des applications, vous pouvez en savoir plus sur l’ajout de ces rôles à vos applications dans le [Guide pratique pour configurer les revendications de rôle émises dans le jeton SAML pour les applications d’entreprise](../develop/active-directory-enterprise-app-role-management.md).

Une fois qu'un rôle d'application fait partie d'un package d'accès :

- Si un utilisateur se voit attribuer ce package d'accès, il est ajouté à ce rôle d'application, s'il n’y figure pas déjà.
- À l'expiration de l'affectation du package d'accès d'un utilisateur, son accès sera retiré du groupe, sauf si un autre package d'accès incluant ce rôle d’application lui est affecté.

Voici quelques considérations à prendre en compte lors de la sélection d'une application :

- Des groupes peuvent également être affectés aux rôles d’applications.  Vous pouvez choisir d'ajouter un groupe au lieu d'un rôle d'application dans un package d'accès, mais l'application ne sera pas visible pour l'utilisateur dans le package d'accès du portail Mon Accès.

1. Sur la page **Ajouter des rôles de ressources à un package d'accès**, cliquez sur **Applications** pour ouvrir le volet Sélectionner les applications.

1. Sélectionnez les applications que vous voulez inclure dans le package d'accès.

    ![Package d'accès - Ajouter des rôles de ressources - Sélectionner les applications](./media/entitlement-management-access-package-resources/application-select.png)

1. Cliquez sur **Sélectionner**.

1. Dans la liste **Rôle**, sélectionnez un rôle d'application.

    ![Package d'accès - Ajouter des rôles de ressources pour une application](./media/entitlement-management-access-package-resources/application-role.png)

1. Cliquez sur **Add**.

    Tous les utilisateurs ayant déjà des affectations au package d'accès auront automatiquement accès à cette application lorsqu'elle est ajoutée.

## <a name="add-a-sharepoint-site-resource-role"></a>Ajouter un rôle de ressources de site SharePoint

Azure AD peut automatiquement donner aux utilisateurs l'accès à un site SharePoint Online ou à une collection de sites SharePoint Online lorsqu'un package d'accès leur est affecté.

1. Sur la page **Ajouter des rôles de ressources à un package d'accès**, cliquez sur **Sites SharePoint** pour ouvrir le volet Sélectionner les sites SharePoint Online.

1. Sélectionnez les sites SharePoint Online que vous voulez inclure dans le package d'accès.

    ![Package d'accès - Ajouter des rôles de ressources - Sélectionner les sites SharePoint Online](./media/entitlement-management-access-package-resources/sharepoint-site-select.png)

1. Cliquez sur **Sélectionner**.

1. Dans la liste **Rôle**, sélectionnez un rôle de site SharePoint Online.

    ![Package d'accès - Ajouter un rôle de ressource pour un site SharePoint Online](./media/entitlement-management-access-package-resources/sharepoint-site-role.png)

1. Cliquez sur **Add**.

    Tous les utilisateurs ayant déjà des affectations au package d'accès auront automatiquement accès à ce site SharePoint Online lorsqu'il est ajouté.

## <a name="remove-resource-roles"></a>Supprimer des rôles de ressources

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs ou gestionnaire de package d’accès

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Packages d’accès**, puis ouvrez le package d'accès.

1. Dans le menu de gauche, cliquez sur **Rôles de ressources**.

1. Dans la liste des rôles de ressources, recherchez le rôle de ressources que vous souhaitez supprimer.

1. Cliquez sur le bouton de sélection ( **...** ), puis sur **Supprimer le rôle de ressources**.

    Tous les utilisateurs ayant déjà des affectations au package d'accès verront leur accès automatiquement révoqué à ce rôle de ressources lorsque ce dernier est supprimé.

## <a name="when-changes-are-applied"></a>Quand des modifications sont appliquées

Dans le cadre de la gestion des droits d'utilisation, Azure AD traitera plusieurs fois par jour les modifications en masse apportées aux affectations et ressources de vos packages d'accès. Ainsi, si vous effectuez une affectation ou modifiez les rôles de ressources de votre package d'accès, la mise en application de ce changement dans Azure AD peut prendre 24 heures, plus le délai nécessaire pour propager ces modifications à d'autres services en ligne Microsoft ou applications SaaS connectées. Si votre changement n'affecte que quelques objets, quelques minutes suffiront pour l'appliquer à Azure AD, après quoi d'autres composants Azure AD détecteront ce changement et mettront à jour les applications SaaS. Si votre modification affecte des milliers d'objets, elle prendra plus de temps. Par exemple, si vous avez un package d'accès avec 2 applications et 100 affectations d'utilisateurs, et que vous décidez d'ajouter un rôle de site SharePoint à ce package d'accès, un certain délai peut s’écouler avant que tous les utilisateurs fassent partie de ce rôle de site SharePoint. Vous pouvez suivre l'état d'avancement grâce au journal d'audit Azure AD, au journal d’approvisionnement Azure AD et aux journaux d'audit du site SharePoint.

Lorsque vous supprimez un membre d’une équipe, il est également supprimé du groupe Microsoft 365. La suppression de la fonctionnalité de conversation de l’équipe peut être retardée. Pour plus d’informations, consultez l’article [Appartenance au groupe](/microsoftteams/office-365-groups#group-membership).

## <a name="next-steps"></a>Étapes suivantes

- [Créer un groupe de base et ajouter des membres avec Azure Active Directory](../fundamentals/active-directory-groups-create-azure-portal.md)
- [Procédure : configurer les revendications de rôle émises dans le jeton SAML pour les applications d’entreprise](../develop/active-directory-enterprise-app-role-management.md)
- [Présentation de SharePoint Online](/sharepoint/introduction)