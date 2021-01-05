---
title: Prise en charge des stratégies de protection des applications et de l’authentification unique dans les applications mobiles que vous développez | Azure
titleSuffix: Microsoft identity platform
description: Explication et présentation de la création d’applications mobiles qui prennent en charge l’authentification unique et les stratégies de protection des applications à l’aide de la plateforme Microsoft Identity et de l’intégration d’Azure Active Directory.
services: active-directory
author: knicholasa
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/14/2020
ms.author: nichola
ms.openlocfilehash: 59924c1f876d08aa504f19c5d6c86dca32fbd1e2
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96173462"
---
# <a name="support-single-sign-on-and-app-protection-policies-in-mobile-apps-you-develop"></a>Prise en charge des stratégies de protection des applications et de l’authentification unique dans les applications mobiles que vous développez

L’authentification unique (SSO, Single Sign-On) est une offre clé de la plateforme d’identités Microsoft et d’Azure Active Directory, fournissant des connexions faciles et sécurisées pour les utilisateurs de votre application. En outre, les stratégies de protection des applications (APP) activent la prise en charge des stratégies de sécurité clés qui assurent la sécurité des données de votre utilisateur. Ensemble, ces fonctionnalités permettent de sécuriser les connexions utilisateur et la gestion des données de votre application.

Cet article explique pourquoi l’authentification unique et l’application sont importantes et fournit des conseils de haut niveau pour la création d’applications mobiles qui prennent en charge ces fonctionnalités. Cela s’applique à la fois aux applications pour téléphone et tablette. Si vous êtes un administrateur informatique qui souhaite déployer l’authentification unique dans l’abonnement Azure Active Directory de son organisation, consultez notre [guide relatif à la planification d’un déploiement d’authentification unique](../manage-apps/plan-sso-deployment.md)

## <a name="about-single-sign-on-and-app-protection-policies"></a>À propos de l’authentification unique et des stratégies de protection des applications

[L’authentification unique (SSO)](../manage-apps/plan-sso-deployment.md) permet à un utilisateur de se connecter une seule fois et d’accéder à d’autres applications sans entrer à nouveau les informations d’identification. Cela facilite l’accès aux applications et élimine la nécessité pour les utilisateurs de mémoriser de longues listes de noms d’utilisateur et de mots de passe. Son implémentation dans votre application facilite l’accès et l’utilisation de votre application.

En outre, l’activation de l’authentification unique dans votre application déverrouille les nouveaux mécanismes d’authentification fournis avec l’authentification moderne, comme [les connexions sans mot de passe](../authentication/concept-authentication-passwordless.md). Les noms d’utilisateur et mots de passe sont l’un des vecteurs d’attaque les plus courants pour les applications, et l’activation de l’authentification unique vous permet de limiter ce risque en appliquant un accès conditionnel ou des connexions avec mot de passe qui ajoutent de la sécurité ou s’appuient sur des mécanismes d’authentification plus sécurisés. Autoriser l’authentification unique permet également [la déconnexion unique](v2-protocols-oidc.md#single-sign-out). Cela est utile dans les situations telles que les applications de travail qui seront utilisées sur des appareils partagés.

[Les politiques de protection des applications (APP)](/mem/intune/apps/app-protection-policy) garantissent que les données d’une organisation restent sûres et contenues. Elles permettent aux entreprises de gérer et de protéger leurs données dans une application, et de contrôler les personnes qui peuvent accéder à l’application et à ses données. L’implémentation des politiques de protection des applications permet à votre application de connecter des utilisateurs à des ressources protégées par des politiques d’accès conditionnel et de transférer en toute sécurité des données vers et à partir d’autres applications protégées. Les scénarios déverrouillés par les stratégies de protection des applications incluent la nécessité d’un code confidentiel pour ouvrir une application, le contrôle du partage des données entre les applications et la prévention de l’enregistrement des données de l’application d’entreprise dans un emplacement de stockage personnel.

## <a name="implementing-single-sign-on"></a>Implémentation de l’authentification unique

Nous vous recommandons d’utiliser ce qui suit pour permettre à votre application de tirer parti de l’authentification unique.

### <a name="use-microsoft-authentication-library-msal"></a>Utiliser la bibliothèque d’authentification Microsoft (MSAL)

Le meilleur choix pour l’implémentation de l’authentification unique dans votre application consiste à utiliser [la bibliothèque d’authentification Microsoft (MSAL)](msal-overview.md). En utilisant MSAL, vous pouvez ajouter l’authentification à votre application avec un minimum de code et d’appels d’API, obtenir les fonctionnalités complètes de la [Microsoft Identity Platform](./index.yml)et laisser Microsoft gérer la maintenance d’une solution d’authentification sécurisée. Par défaut, MSAL ajoute la prise en charge de l’authentification unique pour votre application. En outre, l’utilisation de MSAL est une exigence si vous envisagez également d’implémenter des politiques de protection des applications.

> [!NOTE]
> Il est possible de configurer MSAL pour utiliser une vue Web incorporée. Cela empêchera l’authentification unique. Utilisez le comportement par défaut (autrement dit, le navigateur Web système) pour garantir le fonctionnement de l’authentification unique.

Si vous utilisez actuellement la [bibliothèque ADAL](../azuread-dev/active-directory-authentication-libraries.md) dans votre application, nous vous recommandons vivement de [la migrer vers MSAL](msal-migration.md), car [ADAL est dépréciée](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/update-your-applications-to-use-microsoft-authentication-library/ba-p/1257363).

Pour les applications iOS, nous disposons d’un [démarrage rapide](quickstart-v2-ios.md) qui vous montre comment configurer des connexions à l’aide de MSAL, ainsi que des [conseils pour la configuration de MSAL pour différents scénarios d’authentification unique](single-sign-on-macos-ios.md).

Pour les applications Android, nous disposons d’un [démarrage rapide](quickstart-v2-android.md) qui vous montre comment configurer des connexions à l’aide de MSAL, ainsi que des conseils pour [activer l’authentification unique entre applications sur Android à l’aide de MSAL](msal-android-single-sign-on.md).

### <a name="use-the-system-web-browser"></a>Utiliser le navigateur Web système

Un navigateur web est nécessaire pour l’authentification interactive. Pour les applications mobiles qui utilisent des bibliothèques d’authentification modernes autres que MSAL (autrement dit, d’autres bibliothèques OpenID Connect ou SAML), ou si vous implémentez votre propre code d’authentification, vous devez utiliser le navigateur système comme surface d’authentification pour activer l’authentification unique.

Google vous aide à faire cela dans les applications Android : [Onglets personnalisés chrome-Google Chrome](https://developer.chrome.com/multidevice/android/customtabs).

Apple vous aide à faire cela dans les applications iOS : [Authentification d’un utilisateur par le biais d’un service Web | Documentation pour développeurs Apple](https://developer.apple.com/documentation/authenticationservices/authenticating_a_user_through_a_web_service).

> [!TIP]
> Le [plug-in d’authentification unique pour les appareils Apple](apple-sso-plugin.md) autorise l’authentification unique pour les applications iOS qui utilisent des vues Web incorporées sur des appareils gérés à l’aide d’Intune. Nous recommandons MSAL et le navigateur système comme meilleure option pour le développement d’applications qui activent l’authentification unique pour tous les utilisateurs, mais cela permet l’authentification unique dans certains scénarios où cela n’est pas possible.

## <a name="enable-app-protection-policies"></a>Activer les politiques de protection des applications

Pour activer les stratégies de protection des applications, utilisez la [MSAL (Microsoft Authentication Library)](msal-overview.md). MSAL est la bibliothèque d’authentification et d’autorisation de la plateforme d’identité de Microsoft, et le SDK Intune est développé pour fonctionner en tandem avec celui-ci.

En outre, vous devez utiliser une application Broker pour l’authentification. Le répartiteur requiert que l’application fournisse des informations sur l’application et l’appareil pour garantir la conformité de l’application. les utilisateurs d’iOS utilisent l’application [Microsoft Authenticator](../user-help/user-help-auth-app-sign-in.md) et les utilisateurs Android utilisent l’application Microsoft Authenticator ou l’application [portail d’entreprise](https://play.google.com/store/apps/details?id=com.microsoft.windowsintune.companyportal) pour [l’authentification répartie](./msal-android-single-sign-on.md). Par défaut, MSAL utilise un service Broker comme premier choix pour la réalisation d’une demande d’authentification. par conséquent, l’utilisation du répartiteur pour l’authentification est activée automatiquement pour votre application lors de l’utilisation de MSAL.

Enfin, [ajouter le kit de développement logiciel (SDK) Intune](/mem/intune/developer/app-sdk-get-started) à votre application pour activer les stratégies de protection des applications. Le kit de développement logiciel (SDK) est le plus souvent suivi d’un modèle d’interception et applique automatiquement des politiques de protection d’application pour déterminer si les actions prises par l’application sont autorisées ou non. Il existe également des API que vous pouvez appeler manuellement pour indiquer à l’application s’il existe des restrictions sur certaines actions.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Planifier le déploiement de l’authentification unique Azure Active Directory](../manage-apps/plan-sso-deployment.md)
- [Procédure : Configurer l’authentification unique sur macOS et iOS](single-sign-on-macos-ios.md)
- [Plug-in Microsoft Enterprise SSO pour les appareils Apple (préversion)](apple-sso-plugin.md)
- [Authentification répartie dans Android](./msal-android-single-sign-on.md)
- [Agents d’autorisation et comment les activer](./msal-android-single-sign-on.md)
- [Prise en main du SDK des applications Microsoft Intune](/mem/intune/developer/app-sdk-get-started)
- [Configurer les paramètres du SDK des applications Intune](/mem/intune/developer/app-sdk-ios#configure-settings-for-the-intune-app-sdk)
- [Applications protégées par Microsoft Intune](/mem/intune/apps/apps-supported-intune-apps)