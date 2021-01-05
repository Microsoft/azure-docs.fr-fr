---
title: 'Démarrage rapide : Configurer l’authentification unique (SSO) basée sur OIDC d’une application dans votre locataire Azure Active Directory (Azure AD)'
description: Ce guide de démarrage rapide vous présente en détail le processus de configuration de l’authentification unique (SSO) basée sur OIDC d’une application dans votre locataire Azure Active Directory (Azure AD).
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: quickstart
ms.workload: identity
ms.date: 07/01/2020
ms.author: kenwith
ms.openlocfilehash: 2ff35a8d86534a99b8957a9272f9a1eb59fc27ab
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97399976"
---
# <a name="quickstart-set-up-oidc-based-single-sign-on-sso-for-an-application-in-your-azure-active-directory-azure-ad-tenant"></a>Démarrage rapide : Configurer l’authentification unique (SSO) basée sur OIDC d’une application dans votre locataire Azure Active Directory (Azure AD)

Familiarisez-vous avec les connexions utilisateur simplifiées en configurant l’authentification unique d’une application que vous avez ajoutée à votre locataire Azure Active Directory (Azure AD). Dès l’authentification unique configurée, vos utilisateurs peuvent se connecter à une application avec leurs informations d’identification Azure AD. L’authentification unique est incluse dans l’édition gratuite d’Azure AD.

Pour en savoir plus sur l’authentification unique basée sur OIDC, consultez [Comprendre l’authentification unique basée sur OIDC](configure-oidc-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour configurer l’authentification unique d’une application que vous avez ajoutée à votre locataire Azure AD, vous avez besoin des éléments suivants :

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un des rôles suivants : Administrateur général, Administrateur d’application cloud, Administrateur d’application ou propriétaire du principal de service.
- Une application prenant en charge SSO et qui a déjà été préconfigurée puis ajoutée à la galerie Azure AD. La plupart des applications peuvent utiliser Azure AD pour SSO. Les applications de la galerie Azure AD sont préconfigurées. Si votre application n’est pas listée ou qu’il s’agit d’une application personnalisée, vous pouvez toujours l’utiliser avec Azure AD. Consultez les tutoriels et autres documents dans la table des matières. Ce guide de démarrage rapide porte sur les applications qui ont été préconfigurées pour l’authentification unique puis ajoutées à la galerie Azure AD par les développeurs d’applications.
- Facultatif : effectuer [Voir vos applications](view-applications-portal.md).
- Facultatif : effectuer [Ajouter une application](add-application-portal.md).
- Facultatif : effectuer [Configurer une application](add-application-portal-configure.md).
- Facultatif : suivre [Affecter des utilisateurs à une application](add-application-portal-assign-users.md).

>[!IMPORTANT]
>Pour tester les étapes de ce guide de démarrage rapide, utilisez un environnement de non-production.

## <a name="enable-single-sign-on-for-an-app"></a>Activer l’authentification unique pour une application

Quand vous ajoutez une application qui utilise la norme OIDC pour l’authentification unique, vous disposez d’un bouton de configuration. Quand vous sélectionnez le bouton, vous accédez au site des applications et vous terminez le processus d’inscription de l’application. Le processus d’ajout d’une application est décrit dans le guide de démarrage rapide Ajouter une application, précédemment dans cette série. Si vous configurez une application qui a déjà été ajoutée, consultez le premier guide de démarrage rapide. Il écrit les différentes étapes à effectuer pour afficher les applications se trouvant déjà dans votre locataire. 

Pour configurer l’authentification unique pour une application :

1. Dans le guide de démarrage rapide présenté précédemment dans cette série, vous avez appris à ajouter une application qui utilisera votre locataire Azure AD pour la gestion des identités. Si le développeur de l’application a utilisé la norme OIDC pour implémenter l’authentification unique, un bouton d’inscription s’affiche lors de l’ajout de l’application. 

    :::image type="content" source="media/add-application-portal-setup-oidc-sso/sign-up-oidc-sso.png" alt-text="La capture d’écran montre l’option d’authentification unique et le bouton d’inscription." lightbox="media/add-application-portal-setup-oidc-sso/sign-up-oidc-sso.png":::


2. Sélectionnez **S’inscrire**. Vous êtes alors dirigé vers la page de connexion des développeurs d’applications. Connectez-vous à l’aide des informations d’identification de connexion Azure Active Directory. 

   > [!IMPORTANT]
    > Si vous disposez déjà d’un abonnement à l’application, la validation des détails de l’utilisateur et des informations sur le locataire/l’annuaire se produit. Si l’application n’est pas en mesure de vérifier l’utilisateur, elle vous redirige pour que vous vous inscriviez au service d’application ou à la page d’erreur.

3. Une fois l’authentification réussie, une boîte de dialogue s’affiche pour vous demander le consentement administrateur. Sélectionnez **Consentement pour le compte de votre organisation**, puis **Accepter**. Pour en savoir plus sur le consentement de l’utilisateur et de l’administrateur, consultez [Comprendre le consentement de l’utilisateur et de l’administrateur](../develop/howto-convert-app-to-be-multi-tenant.md#understand-user-and-admin-consent).

    :::image type="content" source="media/add-application-portal-setup-oidc-sso/consent.png" alt-text="La capture d’écran montre l’écran de consentement pour une application." lightbox="media/add-application-portal-setup-oidc-sso/consent.png":::

4. L’application est ajoutée à votre locataire et la page d’accueil de l’application s’affiche.


> [!TIP]
> Vous pouvez automatiser la gestion des applications à l’aide de l’API Graph. Consultez [Automatiser la gestion des applications avec l’API Microsoft Graph](/graph/application-saml-sso-configure-api).

Voici une vidéo qui fournit des détails supplémentaires sur l’ajout d’une application basée sur OIDC à Azure AD.

Ajout d’une application basée sur OIDC dans Azure Active Directory

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4HoNI]

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous aurez terminé cette série de guides de démarrage rapide, supprimez l’application pour nettoyer votre locataire de test. La suppression de l’application est traitée dans le dernier guide de démarrage rapide de cette série : consultez [Supprimer une application](delete-application-portal.md).

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant pour découvrir comment supprimer une application.
> [!div class="nextstepaction"]
> [Supprimer une application](delete-application-portal.md)
