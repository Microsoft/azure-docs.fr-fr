---
title: Ajouter des utilisateurs de collaboration B2B en tant que professionnels de l’information - Azure AD
description: La collaboration B2B permet aux professionnels de l’information et aux propriétaires d’application d’ajouter des utilisateurs invités pour accéder à Azure AD | Microsoft Docs
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 12/19/2018
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 352d5218f91fa55134a7515b6aeee49857847468
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96173207"
---
# <a name="how-users-in-your-organization-can-invite-guest-users-to-an-app"></a>Comment procéder pour inviter des utilisateurs invités à accéder à une application ?

Une fois qu’un utilisateur invité a été ajouté au répertoire dans Azure AD, un propriétaire d’application peut lui envoyer un lien direct vers l’application qu’il souhaite partager. Les administrateurs Azure AD peuvent également configurer la gestion en libre-service pour les applications SAML ou de la galerie dans leur locataire Azure AD. Les propriétaires d’application peuvent ainsi gérer leurs propres utilisateurs invités, même si ces derniers n’ont pas encore été ajoutés au répertoire. Lorsqu’une application est configurée pour être utilisée en libre-service, le propriétaire de l’application utilise le Panneau d’accès pour inviter un utilisateur invité ou ajouter un utilisateur invité à un groupe ayant accès à l’application. La gestion des applications en libre-service pour les applications SAML et de la galerie nécessite que l’administrateur procède à une configuration initiale. Voici un résumé des étapes de configuration (pour obtenir des instructions plus détaillées, consultez la section [Conditions préalables](#prerequisites) ci après) :

 - Activer la gestion de groupes en libre-service pour votre abonné
 - Créer un groupe à affecter à l’application et convertir l’utilisateur en propriétaire
 - Configurer l’application pour le libre-service et affecter le groupe à l’application

> [!NOTE]
> Cet article décrit comment configurer la gestion en libre-service pour les applications SAML et de la galerie que vous avez ajoutées à votre locataire Azure AD. Vous pouvez également [configurer des groupes de Microsoft 365 libre-service](../enterprise-users/groups-self-service-management.md) afin que vos utilisateurs puissent gérer l’accès à leurs propres groupes de Microsoft 365. Pour plus d’informations sur la façon dont les utilisateurs peuvent partager des fichiers et des applications Office avec les utilisateurs invités, consultez [Accès invité dans des groupes de Microsoft 365](https://support.office.com/article/guest-access-in-office-365-groups-bfc7a840-868f-4fd6-a390-f347bf51aff6) et [Partager des fichiers ou dossiers SharePoint](https://support.office.com/article/share-sharepoint-files-or-folders-1fe37332-0f9a-4719-970e-d2578da4941c).

## <a name="invite-a-guest-user-to-an-app-from-the-access-panel"></a>Inviter un utilisateur invité à une application depuis le Panneau d’accès

Après avoir configuré une application pour le libre-service, les propriétaires d’application peuvent utiliser leur propre Panneau d’accès pour inviter un utilisateur invité à l’application qu’ils souhaitent partager. L’utilisateur invité ne doit pas nécessairement être ajouté à Azure AD à l’avance. 

1. Ouvrez votre Panneau d’accès en accédant à `https://myapps.microsoft.com`.
2. Sélectionnez l’application, sélectionnez les points de suspension ( **...** ), puis **Gérer l’application**.
 
   ![Capture d’écran montrant le sous-menu Gérer l’application de l’application Salesforce](media/add-users-iw/access-panel-manage-app.png)
 
3. En haut de la liste des utilisateurs, sélectionnez **+** .
   
   ![Capture d’écran montrant le symbole plus pour ajouter des membres à l’application](media/add-users-iw/access-panel-manage-app-add-user.png)
   
4. Dans la zone de recherche **Ajouter des membres**, saisissez l’adresse e-mail de l’utilisateur invité. Vous pouvez éventuellement inclure un message d’accueil.
   
   ![Capture d’écran montrant la fenêtre Ajouter des membres pour l’ajout d’un invité](media/add-users-iw/access-panel-invitation.png)
   
5. Sélectionnez **Ajouter** pour envoyer une invitation à l’utilisateur invité. Après avoir envoyé l’invitation, le compte d’utilisateur est automatiquement ajouté au répertoire en tant qu’invité.

## <a name="invite-someone-to-join-a-group-that-has-access-to-the-app"></a>Inviter une personne à rejoindre un groupe ayant accès à l’application
Après avoir configuré une application pour le libre-service, les propriétaires d’application peuvent inviter des utilisateurs invités à rejoindre les groupes qu’ils gèrent ayant accès aux applications qu’ils souhaitent partager. Il n’est pas nécessaire que les utilisateurs invités figurent déjà dans le répertoire. Le propriétaire de l’application suit ces étapes pour inviter un utilisateur invité à rejoindre le groupe afin qu’il puisse accéder à l’application.

1. Assurez-vous d’être propriétaire du groupe en libre-service ayant accès à l’application que vous souhaitez partager.
2. Ouvrez votre Panneau d’accès en accédant à `https://myapps.microsoft.com`.
3. Sélectionnez l’application **Groupes**.
   
   ![Capture d’écran montrant l’application Groupes dans le volet d’accès](media/add-users-iw/access-panel-groups.png)
   
4. Sous **Groupes dont je suis propriétaire**, sélectionnez le groupe qui a accès à l’application que vous souhaitez partager.
   
   ![Capture d’écran montrant où sélectionner un groupe sous Groupes dont je suis propriétaire](media/add-users-iw/access-panel-groups-i-own.png)
   
5. En haut de la liste des membres du groupe, sélectionnez **+** .
   
   ![Capture d’écran montrant le symbole plus pour ajouter des membres au groupe](media/add-users-iw/access-panel-groups-add-member.png)
   
6. Dans la zone de recherche **Ajouter des membres**, saisissez l’adresse e-mail de l’utilisateur invité. Vous pouvez éventuellement inclure un message d’accueil.
   
   ![Capture d’écran montrant la fenêtre Ajouter des membres pour l’ajout d’un invité](media/add-users-iw/access-panel-invitation.png)
   
7. Sélectionnez **Ajouter** pour envoyer automatiquement l’invitation à l’utilisateur invité. Après avoir envoyé l’invitation, le compte d’utilisateur est automatiquement ajouté au répertoire en tant qu’invité.


## <a name="prerequisites"></a>Prérequis

La gestion des applications en libre-service nécessite qu’un administrateur général et un administrateur Azure AD procèdent à une configuration initiale. Dans le cadre de cette configuration, vous allez configurer l’application pour le libre-service et affecter un groupe à l’application que le propriétaire de l’application peut gérer. Vous pouvez également configurer le groupe de manière à autoriser n’importe qui à demander à le rejoindre, mais en exigeant l’approbation du propriétaire. (En savoir plus sur la [gestion des groupes en libre-service](../enterprise-users/groups-self-service-management.md).) 

> [!NOTE]
> Vous ne pouvez pas ajouter d'utilisateurs invités à un groupe dynamique ou à un groupe synchronisé sur un Active Directory local.

### <a name="enable-self-service-group-management-for-your-tenant"></a>Activer la gestion de groupes en libre-service pour votre abonné
1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général.
2. Dans le volet de navigation, sélectionnez **Azure Active Directory**.
3. Sélectionnez **Groupes**.
4. Sous **Paramètres**, sélectionnez **Général**.
5. Sous **Gestion de groupes en libre-service**, en regard de **Les propriétaires peuvent gérer les demandes d'appartenance au groupe dans le panneau d'accès**, sélectionnez **Oui**.
6. Sélectionnez **Enregistrer**.

### <a name="create-a-group-to-assign-to-the-app-and-make-the-user-an-owner"></a>Créer un groupe à affecter à l’application et convertir l’utilisateur en propriétaire
1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général ou administrateur Azure AD.
2. Dans le volet de navigation, sélectionnez **Azure Active Directory**.
3. Sélectionnez **Groupes**.
4. Sélectionnez **Nouveau groupe**.
5. Sous **Type de groupe**, sélectionnez **Sécurité**.
6. Saisissez un **nom de groupe** et une **description**.
7. Sous **Type d’appartenance**, sélectionnez **Affecté**.
8. Sélectionnez **Créer** et fermez la page **Groupe**.
9. Sur la page **Groupes - Tous les groupes**, ouvrez le groupe. 
10. Sous **Gérer**, sélectionnez **Propriétaires** > **Ajouter des propriétaires**. Recherchez l’utilisateur qui doit gérer l’accès à l’application. Sélectionnez-le, puis cliquez sur **Sélectionner**.

### <a name="configure-the-app-for-self-service-and-assign-the-group-to-the-app"></a>Configurer l’application pour le libre-service et affecter le groupe à l’application
1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général ou administrateur Azure AD.
2. Dans le volet de navigation, sélectionnez **Azure Active Directory**.
3. Sous **Gérer**, sélectionnez **Applications d’entreprise** > **Toutes les applications**.
4. Dans la liste des applications, recherchez l’application et ouvrez-la.
5. Sous **Gérer**, sélectionnez **Authentification unique** et configurez l’application pour l’authentification unique. (Pour en savoir plus, consultez [Gérer l’authentification unique pour les applications d’entreprise](../manage-apps/add-application-portal-setup-sso.md).)
6. Sous **Gérer**, sélectionnez **Libre-service** et configurez l’accès à l’application en libre-service. (Pour en savoir plus, consultez [Utiliser l’accès aux applications en libre-service](../manage-apps/access-panel-manage-self-service-access.md).) 

    > [!NOTE]
    > Pour le paramètre **À quel groupe les utilisateurs attribués doivent-ils être ajoutés ?** , sélectionnez le groupe que vous avez créé dans la section précédente.
7. Sous **Gérer**, sélectionnez **Utilisateurs et groupes** et vérifiez que le groupe en libre-service que vous avez créé apparaît dans la liste.
8. Pour ajouter l’application au panneau d’accès du propriétaire du groupe, sélectionnez **Ajouter un utilisateur** > **Utilisateurs et groupes**. Recherchez le propriétaire du groupe, sélectionnez l’utilisateur, cliquez sur **Sélectionner**, puis cliquez sur **Attribuer** pour l’ajouter à l’application.

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants sur Azure AD B2B Collaboration :

- [Qu'est-ce que la collaboration B2B d'Azure AD ?](what-is-b2b.md)
- [Comment les administrateurs Azure Active Directory ajoutent-ils des utilisateurs B2B Collaboration ?](add-users-administrator.md)
- [Utilisation d’une invitation B2B Collaboration](redemption-experience.md)
- [Prix des identités externes](external-identities-pricing.md)