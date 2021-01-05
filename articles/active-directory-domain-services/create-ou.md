---
title: Créer une unité d’organisation (UO) dans Azure AD Domain Services | Microsoft Docs
description: Découvrez comment créer et gérer une unité d’organisation (UO) personnalisée dans un domaine managé Azure AD Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 52602ad8-2b93-4082-8487-427bdcfa8126
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: fbdfcc23553a27aaa4d7bbd4bff9c4f33cb8fd19
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620016"
---
# <a name="create-an-organizational-unit-ou-in-an-azure-active-directory-domain-services-managed-domain"></a>Créer une unité d’organisation (UO) sur un domaine dans un domaine managé Azure Active Directory Domain Services

Les unités d’organisation (UO) dans un domaine managé Active Directory Domain Services (AD DS) vous permettent de regrouper logiquement des objets tels que des comptes d’utilisateur, des comptes de service ou des comptes d’ordinateur. Vous pouvez ensuite affecter des administrateurs à des unités d’organisation spécifiques et appliquer une stratégie de groupe, donc des paramètres de configuration ciblés.

Les domaines managés Azure AD DS incluent les deux UO intégrées suivantes :

* *Ordinateurs AADDC* : contient des objets ordinateur associés à tous les ordinateurs qui sont joints au domaine managé.
* *Utilisateurs AADDC* : comprend les utilisateurs et les groupes qui y sont synchronisés à partir du locataire Azure AD.

Lors de la création et de l’exécution des charges de travail utilisant Azure AD DS, vous devrez peut-être créer des comptes de service pour que les applications s’authentifient elles-mêmes. Pour organiser ces comptes de service, vous créez souvent une unité d’organisation personnalisée dans le domaine managé, puis des comptes de service au sein de cette unité d’organisation.

Dans un environnement hybride, les unités d’organisation créées dans un environnement AD DS local ne sont pas synchronisées avec le domaine managé. Les domaines managés utilisent une structure d’unité d’organisation plate. Tous les comptes d’utilisateurs et les groupes sont stockés dans le conteneur *Utilisateurs AADDC*, en dépit de la synchronisation effectuée à partir de forêts ou de domaines locaux différents, même si vous y avez configuré une structure d’unités d’organisation hiérarchique.

Cet article vous explique comment créer une UO dans votre domaine géré.

## <a name="before-you-begin"></a>Avant de commencer

Pour faire ce qui est décrit dans cet article, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, suivez le tutoriel pour [créer et configurer une instance Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Une machine virtuelle de gestion Windows Server jointe au domaine managé Azure AD DS.
    * Si nécessaire, suivez le tutoriel [Créer une machine virtuelle de gestion][tutorial-create-management-vm].
* Un compte d’utilisateur membre du groupe *Administrateurs Azure AD DC* dans votre locataire Azure AD.

## <a name="custom-ou-considerations-and-limitations"></a>Considérations et limitations relatives aux unités d’organisation personnalisées

Lorsque vous créez des unités d’organisation personnalisées dans un domaine managé, vous bénéficiez d’une plus grande souplesse de gestion pour la gestion des utilisateurs et l’application de la stratégie de groupe. Par rapport à un environnement AD DS local, il existe certaines limitations et considérations à prendre en compte lors de la création et de la gestion d’une structure d’unité d’organisation personnalisée dans un domaine managé :

* Pour créer des unités d’organisation personnalisées, les utilisateurs doivent être membres du groupe *Administrateurs AAD DC*.
* Un utilisateur qui crée une unité d’organisation personnalisée se voit accorder des privilèges d’administration (contrôle total) sur cette unité d’organisation et est le propriétaire de la ressource.
    * Par défaut, le groupe *Administrateurs AAD DC* a également le contrôle total sur l’unité d’organisation personnalisée.
* Une unité d’organisation pour *Utilisateurs AADDC* est créée et contient tous les comptes d’utilisateur synchronisés à partir de votre locataire Azure AD.
    * Vous ne pouvez pas déplacer des utilisateurs ou des groupes de l’unité d’organisation *Utilisateurs AADDC* vers des unités d’organisation personnalisées que vous créez. Seuls les comptes d’utilisateurs ou les ressources créés dans le domaine managé peuvent être déplacés dans des unités d’organisation personnalisées.
* Les comptes d’utilisateur, groupes, comptes de service et objets ordinateur que vous créez dans des unités d’organisation personnalisées ne sont pas disponibles dans votre locataire Azure AD.
    * Ces objets n’apparaissent pas à l’aide de l’API Microsoft Graph ou dans l’interface utilisateur Azure AD ; ils sont uniquement disponibles dans votre domaine managé.

## <a name="create-a-custom-ou"></a>Créer une unité d’organisation personnalisée

Pour créer une unité d’organisation personnalisée, vous utilisez les outils d’administration Active Directory à partir d’une machine virtuelle jointe à un domaine. Le Centre d’administration Active Directory vous permet d’afficher, de modifier et de créer des ressources dans un domaine managé, notamment des unités d’organisation.

> [!NOTE]
> Pour créer une unité d’organisation personnalisée dans un domaine managé, vous devez être connecté à un compte d’utilisateur membre du groupe d *administrateurs du contrôleur de domaine AAD*.

1. Connectez-vous à votre machine virtuelle de gestion. Pour connaître les différentes étapes vous permettant de vous connecter au portail Azure, consultez la page [Se connecter à une machine virtuelle Windows Server][connect-windows-server-vm].
1. Dans l’écran d’accueil, sélectionnez **Outils d’administration**. La liste des outils de gestion disponibles qui ont été installés dans le tutoriel [Créer une machine virtuelle de gestion][tutorial-create-management-vm] s’affiche à l’écran.
1. Pour créer et gérer des unités d’organisation, sélectionnez **Centre d’administration Active Directory** dans la liste des outils d’administration.
1. Dans le volet de gauche, choisissez votre domaine managé, par exemple *aaddscontoso.com*. Une liste des unités d’organisation et des ressources s’affiche :

    ![Sélectionnez votre domaine managé dans le Centre d’administration Active Directory](./media/create-ou/create-ou-adac-overview.png)

1. Le volet **Tâches** s’affiche sur le côté droit du Centre d’administration Active Directory. Sous le domaine, par exemple *aaddscontoso.com*, sélectionnez **Nouveau > Unité d’organisation**.

    ![Sélectionnez l’option qui vous permettra de créer une nouvelle unité d’organisation dans le Centre d'administration Active Directory](./media/create-ou/create-ou-adac-new-ou.png)

1. Dans la boîte de dialogue **Créer une unité d’organisation**, indiquez un **Nom** pour la nouvelle unité d’organisation, par exemple *MyCustomOu*. Fournissez une brève description de l’unité d’organisation, telle qu’*unité d’organisation personnalisée pour les comptes de service*. Si vous le souhaitez, vous pouvez également renseigner le champ **Managée par** de l’unité d’organisation. Sélectionnez **OK** pour créer l’unité d’organisation personnalisée.

    ![Créez une unité d’organisation personnalisée à partir du Centre d’administration Active Directory](./media/create-ou/create-ou-dialog.png)

1. De retour dans le Centre d’administration Active Directory, l’unité d’organisation personnalisée est désormais listée et peut être utilisée :

    ![Unité d’organisation personnalisée disponible dans le Centre d’administration Active Directory](./media/create-ou/create-ou-done.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’utilisation des outils d’administration ou la création et l’utilisation de comptes de service, consultez les articles suivants :

* [Centre d'administration Active Directory : Prise en main](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd560651(v=ws.10))
* [Guide pas à pas des comptes de service (éventuellement en anglais)](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd548356(v=ws.10))

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[tutorial-create-management-vm]: tutorial-create-management-vm.md
[connect-windows-server-vm]: join-windows-vm.md#connect-to-the-windows-server-vm