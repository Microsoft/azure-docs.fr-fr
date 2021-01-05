---
title: Limiter l’application Azure AD à un ensemble d’utilisateurs | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment limiter l’accès à vos applications inscrites dans Azure AD à un ensemble d’utilisateurs sélectionné.
services: active-directory
author: kalyankrishna1
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 09/24/2018
ms.author: kkrishna
ms.reviewer: jmprieur
ms.custom: aaddev
ms.openlocfilehash: a9b0090fa13a6d2f7a837a4f5ffee37d70893318
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88116901"
---
# <a name="how-to-restrict-your-azure-ad-app-to-a-set-of-users-in-an-azure-ad-tenant"></a>Procédure : Limiter votre application Azure AD à un ensemble d’utilisateurs dans un locataire Azure AD

Les applications inscrites dans un locataire Azure Active Directory (Azure AD) sont, par défaut, disponibles pour tous les utilisateurs du locataire qui parviennent à s’authentifier.

De même, avec une application [mutualisée](howto-convert-app-to-be-multi-tenant.md), tous les utilisateurs dans le locataire Azure AD où cette application est approvisionnée sont en mesure d’accéder à cette application après avoir réussi à s’authentifier auprès de leur locataire respectif.

Les développeurs et les administrateurs de locataires ont souvent des exigences impliquant de limiter une application à un certain ensemble d’utilisateurs. Pour ce faire, les développeurs peuvent utiliser des modèles d’autorisation courants comme le contrôle d’accès en fonction du rôle Azure (Azure RBAC), mais cette approche nécessite une quantité de travail importante.

Les développeurs et administrateurs de locataires peuvent également limiter une application à un ensemble spécifique d’utilisateurs ou de groupes de sécurité dans le locataire grâce à la fonctionnalité intégrée d’Azure AD.

## <a name="supported-app-configurations"></a>Configurations d’application prises en charge

La possibilité de limiter une application à un ensemble spécifique d’utilisateurs ou de groupes de sécurité dans un locataire est compatible avec les types d’applications suivants :

- applications configurées pour l’authentification unique fédérée avec l’authentification basée sur SAML ;
- applications de proxy d’application qui utilisent la pré-authentification Azure AD ;
- applications créées directement sur la plateforme d’application Azure AD qui utilisent l’authentification OAuth 2.0/OpenID Connect après qu’un utilisateur ou administrateur a donné son consentement à cette application.

     > [!NOTE]
     > Cette fonctionnalité est disponible pour les applications professionnelles et API web/applications web uniquement. Les applications qui sont inscrites en tant qu’applications [natives](./quickstart-register-app.md) ne peuvent pas être limitées à un ensemble d’utilisateurs ou de groupes de sécurité dans le locataire.

## <a name="update-the-app-to-enable-user-assignment"></a>Mettre à jour l’application pour permettre l’affectation d’utilisateurs

Il existe deux façons de créer une application avec l’affectation d’utilisateurs activée. L’une requiert le rôle d’**administrateur général**, mais pas le second.

### <a name="enterprise-applications-requires-the-global-administrator-role"></a>Applications d’entreprise (nécessite le rôle d’administrateur général)

1. Accédez au [**Portail Azure**](https://portal.azure.com/) et connectez-vous tant **qu’administrateur général**.
1. Dans la barre supérieure, sélectionnez le compte connecté. 
1. Sous **Répertoire**, sélectionnez le locataire Azure AD où l’application sera inscrite.
1. Dans le menu de navigation à gauche, sélectionnez **Azure Active Directory**. Si Azure Active Directory n’est pas disponible dans le volet de navigation, procédez comme suit :

    1. Sélectionnez **All services** (Tous les services) en haut du menu principal de navigation de gauche.
    1. Tapez **Azure Active Directory** dans la zone de recherche de filtre, puis sélectionnez l’élément **Azure Active Directory** dans les résultats.

1. Dans le volet **Azure Active Directory**, sélectionnez **Applications d’entreprise** dans le menu de navigation de gauche **d’Azure Active Directory**.
1. Sélectionnez **Toutes les applications** pour afficher la liste complète de vos applications.

     Si vous ne voyez pas l’application que vous souhaitez afficher ici, utilisez les filtres présents en haut de la liste **Toutes les applications** pour restreindre la liste ou bien faites-la défiler vers le bas pour localiser votre application.

1. Dans la liste qui s’affiche, sélectionnez l’application à laquelle vous souhaitez affecter un utilisateur ou un groupe de sécurité.
1. Sur la page **Vue d’ensemble** de l’application, sélectionnez **Propriétés** dans le menu de navigation de gauche de l’application.
1. Recherchez le paramètre **Affectation utilisateur requise?** et définissez-le sur **Oui**. Lorsque cette option a la valeur **Oui**, les utilisateurs du locataire doivent d’abord être affectés à cette application, sans quoi ils ne pourront pas se connecter à cette application.
1. Sélectionnez **Enregistrer** pour enregistrer cette modification de la configuration.

### <a name="app-registration"></a>Inscription d'application

1. Accéder au [**portail Azure**](https://portal.azure.com/).
1. Dans la barre supérieure, sélectionnez le compte connecté. 
1. Sous **Répertoire**, sélectionnez le locataire Azure AD où l’application sera inscrite.
1. Dans le menu de navigation à gauche, sélectionnez **Azure Active Directory**.
1. Dans le volet **Azure Active Directory**, sélectionnez **Inscriptions d’applications** dans le menu de navigation de gauche **d’Azure Active Directory**.
1. Créez ou sélectionnez l’application que vous souhaitez gérer. Vous devez être **propriétaire** de cette inscription d’application.
1. Dans la page **Vue d'ensemble** de l’application, suivez le lien **Application gérée dans l’annuaire local** sous les éléments essentiels en haut de la page. Cela vous permet de vous connecter à l’_application d’entreprise gérée_ de l’inscription de votre application.
1. Dans le panneau de navigation de gauche, sélectionnez **Propriétés**.
1. Recherchez le paramètre **Affectation utilisateur requise?** et définissez-le sur **Oui**. Lorsque cette option a la valeur **Oui**, les utilisateurs du locataire doivent d’abord être affectés à cette application, sans quoi ils ne pourront pas se connecter à cette application.
1. Sélectionnez **Enregistrer** pour enregistrer cette modification de la configuration.

## <a name="assign-users-and-groups-to-the-app"></a>Affecter des utilisateurs et des groupes à l’application

Une fois que vous avez configuré votre application pour activer l’affectation d’utilisateurs, vous pouvez poursuivre et affecter des utilisateurs et des groupes à l’application.

1. Sélectionnez le volet **Utilisateurs et groupes** dans le menu de navigation de gauche de l’application.
1. En haut de la liste **Utilisateurs et groupes**, sélectionnez le bouton **Ajouter un utilisateur** en haut de la liste pour ouvrir le volet **Ajouter une attribution**.
1. Sélectionnez le sélecteur **Utilisateurs** à partir du volet **Ajouter une attribution**. 

     Une liste des utilisateurs et des groupes de sécurité s’affiche, ainsi qu’une zone de texte pour rechercher et localiser un utilisateur ou un groupe spécifique. Cet écran vous permet de sélectionner plusieurs utilisateurs et groupes d’un coup.

1. Une fois que vous avez sélectionné les utilisateurs et groupes, appuyez sur le bouton **Sélectionner** en bas pour passer à l’étape suivante.
1. (Facultatif) Si vous avez défini des rôles d’application dans votre application, vous pouvez utiliser l’option **Sélectionner un rôle** pour affecter les utilisateurs et groupes sélectionnés à un des rôles de l’application. 
1. Appuyez sur le bouton **Attribuer** en bas pour terminer l’attribution des utilisateurs et des groupes à l’application. 
1. Vérifiez que les utilisateurs et les groupes ajoutés figurent dans la liste **Utilisateurs et groupes** mise à jour.

## <a name="more-information"></a>Informations complémentaires

- [Procédure : Ajouter des rôles d’application dans votre application](./howto-add-app-roles-in-azure-ad-apps.md)
- [Ajouter une autorisation à une application web ASP.NET Core à l'aide de rôles d'application et de revendications de rôles](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/5-WebApp-AuthZ/5-1-Roles)
- [Utilisation de groupes de sécurité et de rôles d’Application dans vos applications (vidéo)](https://www.youtube.com/watch?v=V8VUPixLSiM)
- [Azure Active Directory, désormais avec les revendications de groupe et les rôles d’application](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-Active-Directory-now-with-Group-Claims-and-Application/ba-p/243862)
- [Manifeste d’application Azure Active Directory](./reference-app-manifest.md)