---
title: Activer le service Profil utilisateur SharePoint avec Azure AD DS | Microsoft Docs
description: Découvrez comment configurer un domaine managé Azure Active Directory Domain Services pour prendre en charge la synchronisation de profils pour SharePoint Server
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 938a5fbc-2dd1-4759-bcce-628a6e19ab9d
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: 8fbde10ccf5a7f083f5fbddaadd6668d880a1fac
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619825"
---
# <a name="configure-azure-active-directory-domain-services-to-support-user-profile-synchronization-for-sharepoint-server"></a>Configurer Azure Active Directory Domain Services pour prendre en charge la synchronisation de profils utilisateur pour SharePoint Server

SharePoint Server propose un service de synchronisation de profils utilisateur. Cette fonctionnalité permet le stockage de profils utilisateur à un emplacement centralisé accessible à divers sites et batteries de serveurs SharePoint. Pour configurer le service de profil utilisateur SharePoint Server, les autorisations appropriées doivent être accordées dans un domaine managé Azure Active Directory Domain Services (Azure AD DS). Pour plus d’informations, consultez [Synchronisation de profils utilisateur dans SharePoint Server](/SharePoint/administration/user-profile-service-administration).

Cet article vous montre comment configurer Azure AD DS pour autoriser le service de synchronisation de profils utilisateur SharePoint Server.

## <a name="before-you-begin"></a>Avant de commencer

Pour faire ce qui est décrit dans cet article, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, suivez le tutoriel pour [créer et configurer un domaine managé Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Une machine virtuelle de gestion Windows Server jointe au domaine managé Azure AD DS.
    * Si nécessaire, suivez le tutoriel [Créer une machine virtuelle de gestion][tutorial-create-management-vm].
* Un compte d’utilisateur membre du groupe *Administrateurs Azure AD DC* dans votre locataire Azure AD.
* Un compte de service SharePoint pour le service de synchronisation de profils utilisateur.
    * Si nécessaire, consultez [Planifier des comptes d’administration et de service dans SharePoint Server][sharepoint-service-account].

## <a name="service-accounts-overview"></a>Vue d’ensemble des comptes de service

Dans un domaine managé, un groupe de sécurité nommé *AAD DC Service Accounts* existe dans l’unité d’organisation *Users*. Les privilèges suivants sont délégués aux membres de ce groupe de sécurité :

- Le privilège **Répliquer les changements de répertoire** au DSE racine.
- Le privilège **Répliquer les changements de répertoire** dans le contexte de nommage *Configuration* (conteneur `cn=configuration`).

Le groupe de sécurité *AAD DC Service Accounts* est aussi membre du groupe intégré *Accès compatible pré-Windows 2000*.

Quand il est ajouté à ce groupe de sécurité, le compte de service du service de synchronisation de profils utilisateur SharePoint Server se voit accorder les privilèges nécessaires à son bon fonctionnement.

## <a name="enable-support-for-sharepoint-server-user-profile-sync"></a>Activer la prise en charge de la synchronisation de profils utilisateur SharePoint Server

Le compte de service pour SharePoint Server a besoin des privilèges adéquats pour répliquer les modifications apportées au répertoire et permettre à la synchronisation de profils utilisateur SharePoint Server de fonctionner correctement. Pour fournir ces privilèges, ajoutez le compte de service utilisé pour la synchronisation de profils utilisateur SharePoint au groupe *AAD DC Service Accounts*.

À partir de votre machine virtuelle de gestion Azure AD DS, effectuez les étapes suivantes :

> [!NOTE]
> Pour modifier l’appartenance au groupe dans un domaine managé, vous devez être connecté à un compte d’utilisateur qui est membre du groupe *AAD DC Administrators*.

1. Dans l’écran d’accueil, sélectionnez **Outils d’administration**. La liste des outils de gestion disponibles qui ont été installés dans le tutoriel [Créer une machine virtuelle de gestion][tutorial-create-management-vm] s’affiche à l’écran.
1. Pour gérer l’appartenance au groupe, sélectionnez **Centre d’administration Active Directory** dans la liste des outils d’administration.
1. Dans le volet de gauche, choisissez votre domaine managé, par exemple *aaddscontoso.com*. Une liste d’unités d’organisation et de ressources existantes s’affiche.
1. Sélectionnez l’unité d’organisation **Users**, puis choisissez le groupe de sécurité *AAD DC Service Accounts*.
1. Sélectionnez **Membres**, puis choisissez **Ajouter...** .
1. Entrez le nom du compte de service SharePoint, puis sélectionnez **OK**. Dans l’exemple suivant, le compte de service SharePoint se nomme *spadmin* :

    ![Ajouter le compte de service SharePoint au groupe de sécurité AAD DC Service Accounts](./media/deploy-sp-profile-sync/add-member-to-aad-dc-service-accounts-group.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Gérer la synchronisation de profils utilisateur dans SharePoint Server](/SharePoint/administration/manage-profile-synchronization).

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[tutorial-create-management-vm]: tutorial-create-management-vm.md

<!-- EXTERNAL LINKS -->
[sharepoint-service-account]: /sharepoint/security-for-sharepoint-server/plan-for-administrative-and-service-accounts