---
title: Inscrire votre application pour utiliser Azure Active Directory | Microsoft Docs
description: Destiné aux professionnels de l’informatique, cet article fournit des instructions pour l’intégration d’applications Azure à Active Directory.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 10/30/2018
ms.author: kenwith
ms.collection: M365-identity-device-management
ms.openlocfilehash: de16c947c59f5a0111b9325dbefe7daf1268fb40
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94649158"
---
# <a name="develop-line-of-business-apps-for-azure-active-directory"></a>Développer des applications métier pour Azure Active Directory
Ce guide fournit une vue d’ensemble du développement d’applications métier pour Azure Active Directory. Il s’adresse aux administrateurs généraux de systèmes Active Directory/Microsoft 365.

## <a name="overview"></a>Vue d’ensemble
La création d’applications intégrées à Azure AD permet aux utilisateurs de votre organisation de bénéficier de l’authentification unique avec Microsoft 365. En disposant de l’application dans Azure AD, vous pouvez contrôler la stratégie d’authentification pour celle-ci. Pour en savoir plus sur l’accès conditionnel et la méthode à suivre pour protéger les applications avec l’authentification Multi-Factor Authentication, voir [Configuration des règles d’accès](../authentication/tutorial-enable-azure-mfa.md).

Inscrivez votre application pour utiliser Azure Active Directory. Inscrire l’application signifie que vos développeurs peuvent utiliser Azure AD pour authentifier les utilisateurs et demander l’accès aux ressources de l’utilisateur, telles que le courrier électronique, le calendrier et des documents.

Tout membre de votre annuaire (pas les invités) peut inscrire une application, procédé également appelé *création d’un objet d’application*. Si vous ne parvenez pas à inscrire une application, cela signifie que l’administrateur général de votre annuaire a restreint cette fonctionnalité et que vous devrez peut-être le contacter afin [d’obtenir les droits appropriés](../roles/delegate-app-roles.md#assign-built-in-application-admin-roles) pour inscrire l’application. Pour en savoir plus sur la façon de limiter les autorisations de l’utilisateur, consultez [Déléguer des autorisations d’inscription d’application dans Azure Active Directory](../roles/delegate-app-roles.md#restrict-who-can-create-applications).

En inscrivant une application, tout utilisateur peut effectuer les opérations suivantes :

* Obtenir pour son application une identité reconnue par Azure AD.
* Obtenir un ou plusieurs secrets/clés que l’application peut utiliser pour s’authentifier auprès d’Active Directory.
* Personnaliser l’application avec un nom personnalisé, un logo, etc., dans le portail Azure.
* Appliquer les fonctionnalités d’autorisation Azure AD pour l’application, notamment :

  * Contrôle d’accès en fonction du rôle
  * Azure Active Directory en tant que serveur d’autorisation oAuth (sécuriser une API exposée par l’application)
* Déclarer les autorisations requises nécessaires au bon fonctionnement de l’application, notamment :

     - Autorisations de l’application (administrateurs généraux uniquement). Par exemple : Appartenance à un rôle dans une autre application Azure AD ou appartenance à un rôle par rapport à une ressource, un groupe de ressources ou un abonnement Azure
     - Autorisations déléguées (tout utilisateur). Par exemple : Azure AD, connexion et lecture de profil

> [!NOTE]
> Par défaut, tout membre peut inscrire une application. Pour savoir comment limiter les autorisations d’inscription d’applications à des membres spécifiques, reportez-vous au document [Comment les applications sont ajoutées à Azure AD](../develop/active-directory-how-applications-are-added.md#who-has-permission-to-add-applications-to-my-azure-ad-instance).
>
>

Voici les opérations que vous devez effectuer en tant qu’administrateur général pour aider les développeurs à préparer leurs applications pour la production :

* Configurer des règles d’accès (stratégie d’accès/MFA)
* Configurer l’application pour qu’elle demande l’affectation de l’utilisateur et affecter des utilisateurs
* Supprimer l’expérience de consentement d’utilisateur par défaut

## <a name="configure-access-rules"></a>Configurer des règles d’accès
Configurer des règles d’accès par application de vos applications SaaS. Par exemple, vous pouvez requérir un MFA, ou autoriser l’accès aux utilisateurs uniquement sur les réseaux approuvés. Pour plus d’informations à ce sujet, voir [Configuration des règles d’accès](../authentication/tutorial-enable-azure-mfa.md).

## <a name="configure-the-app-to-require-user-assignment-and-assign-users"></a>Configurer l’application pour qu’elle demande l’affectation de l’utilisateur et affecter des utilisateurs
Par défaut, les utilisateurs peuvent accéder aux applications sans affectation. Toutefois, si l’application expose des rôles ou que vous souhaitez qu’elle s’affiche sur Mes applications d’un utilisateur, vous devez demander l’affectation de l’utilisateur.

Si vous êtes abonné à Azure AD Premium ou Enterprise Mobility Suite (EMS), nous vous recommandons fortement d’utiliser les groupes. L’affectation de groupes à l’application vous permet de déléguer la gestion d’accès en continu au propriétaire du groupe. Vous pouvez créer un groupe ou demander à la personne responsable au sein de votre organisation de créer un groupe à l’aide de votre dispositif de gestion de groupe.

[Attribution d’utilisateurs et de groupes à une application](./assign-user-or-group-access-portal.md)  


## <a name="suppress-user-consent"></a>Supprimer le consentement de l’utilisateur
Par défaut, chaque utilisateur doit se soumettre à une expérience de consentement pour se connecter. L’expérience de la demande aux utilisateurs d’un consentement à l’octroi d’autorisations à une application peut être déconcertante pour les utilisateurs qui ne sont pas familiarisés avec la prise de décision de ce type.

Pour les applications de confiance, vous pouvez simplifier l’expérience utilisateur en accordant le consentement à l’application au nom de votre organisation.

Pour en savoir plus sur le consentement de l’utilisateur et sur l’expérience du consentement dans Azure, consultez [Intégration d’applications dans Azure Active Directory](../develop/quickstart-register-app.md).

## <a name="related-articles"></a>Articles connexes
* [Offrir un accès à distance sécurisé aux applications locales](application-proxy.md)
* [Gestion de l’accès aux applications](what-is-access-management.md)