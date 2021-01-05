---
title: Exemples de code pour Azure Active Directory v1.0 | Microsoft Docs
description: Fournit un index des exemples de code Azure Active Directory (point de terminaison v1.0), organisés par scénario.
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.topic: sample
ms.workload: identity
ms.date: 07/15/2019
ms.author: ryanwi
ms.reviewer: jmprieur
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: 662a0f53a5d5e8e02ef3b29640009fd07bf4f1d0
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97652142"
---
# <a name="azure-active-directory-code-samples-v10-endpoint"></a>Exemples de code Azure Active Directory (point de terminaison v1.0)

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

Vous pouvez utiliser Microsoft Azure Active Directory (Azure AD) pour ajouter l'authentification et l'autorisation à vos applications web et API web.

Cette section fournit des liens vers des exemples que vous pouvez utiliser pour en savoir plus sur le point de terminaison Azure AD v1.0. Ces exemples vous montrent comment procéder et contiennent des extraits de code que vous pouvez utiliser dans vos applications. Dans la page d’exemples de code, vous trouverez des rubriques Lisez-moi qui offrent une aide relative à la configuration requise, à l’installation et à la configuration. Le code est également accompagné de commentaires pour vous aider à comprendre les sections critiques.

> [!NOTE]
> Si vous êtes intéressé par des exemples de code Azure AD V2, consultez [Exemples de code v2.0 par scénario](../develop/sample-v2-code.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

Pour comprendre le scénario de base de chaque type d’exemple, consultez [Scénarios d’authentification pour Azure AD](v1-authentication-scenarios.md).

Vous pouvez également contribuer à nos exemples sur GitHub. Pour en savoir plus, consultez [Documentation et exemples Microsoft Azure Active Directory](https://github.com/Azure-Samples?page=3&query=active-directory).

## <a name="single-page-applications"></a>Applications monopages

Cet exemple montre comment écrire une application monopage sécurisée avec Azure AD.

| Plateforme | Appelle sa propre API | Appelle une autre API web |
|--|--|--|
| ![Cette image affiche le logo JavaScript](media/sample-v2-code/logo-js.png) | [javascript-singlepageapp](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi) |
| ![Cette image affiche le logo Angular JS](media/sample-v2-code/logo-angular.png) | [angularjs-singlepageapp](https://github.com/Azure-Samples/active-directory-angularjs-singlepageapp) | [angularjs-singlepageapp-cors](https://github.com/Azure-Samples/active-directory-angularjs-singlepageapp-dotnet-webapi) |

## <a name="web-applications"></a>Applications web

### <a name="web-applications-signing-in-users-calling-microsoft-graph-or-a-web-api-with-the-users-identity"></a>Applications web connectant les utilisateurs, appelant Microsoft Graph ou une API web avec l’identité de l’utilisateur

Les exemples suivants illustrent des applications web qui connectent les utilisateurs. Certaines de ces applications appellent également Microsoft Graph ou votre propre API web dans le nom de l’utilisateur connecté.

| Plateforme | Connecte uniquement les utilisateurs | Appelle Microsoft Graph | Appelle une autre API web ASP.NET ou ASP.NET Core 2.0 |
|--|--|--|--|
| ![Cette illustration montre le logo ASP.NET Core](media/sample-v2-code/logo-netcore.png)</p>ASP.NET Core 2.0 | [dotnet-webapp-openidconnect-aspnetcore](https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-aspnetcore) | [webapp-webapi-multitenant-openidconnect-aspnetcore](https://github.com/Azure-Samples/active-directory-webapp-webapi-multitenant-openidconnect-aspnetcore/) </p>(AAD Graph) | [dotnet-webapp-webapi-openidconnect-aspnetcore](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-openidconnect-aspnetcore) |
| ![Cette image affiche le logo ASP.NET Framework](media/sample-v2-code/logo-netframework.png)</p> ASP.NET 4.5 | </p> [webapp-WSFederation-dotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation) </p> [dotnet-webapp-webapi-oauth2-useridentity](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-oauth2-useridentity) | [dotnet-webapp-multitenant-openidconnect](https://github.com/Azure-Samples/active-directory-dotnet-webapp-multitenant-openidconnect)</p> (AAD Graph) |
| ![Cette image affiche le logo Python](media/sample-v2-code/logo-python.png) |  | [python-webapp-graphapi](https://github.com/Azure-Samples/active-directory-python-webapp-graphapi) |
| ![Cette image affiche le logo Java](media/sample-v2-code/logo-java.png) |  | [java-webapp-openidconnect](https://github.com/azure-samples/active-directory-java-webapp-openidconnect) |
| ![Cette image affiche le logo PHP](media/sample-v2-code/logo-php.png) |  | [php-graphapi-web](https://github.com/Azure-Samples/active-directory-php-graphapi-web) |

### <a name="web-applications-demonstrating-role-based-access-control-authorization"></a>Applications web illustrant le contrôle d’accès en fonction du rôle (autorisation)

Les exemples suivants montrent comment implémenter le contrôle d’accès en fonction du rôle (RBAC). Le contrôle d’accès en fonction du rôle permet de restreindre les autorisations de certaines fonctionnalités dans une application web à certains utilisateurs. Les utilisateurs sont autorisés selon qu’ils appartiennent à un **groupe Azure AD** ou qu’ils ont un **rôle** d’application donné.

| Plateforme | Exemple |
|--|--|
| ![Cette image affiche le logo ASP.NET Framework](media/sample-v2-code/logo-netframework.png)</p> ASP.NET 4.5 | [dotnet-webapp-groupclaims](https://github.com/Azure-Samples/active-directory-dotnet-webapp-groupclaims) </p>  [dotnet-webapp-roleclaims](https://github.com/Azure-Samples/active-directory-dotnet-webapp-roleclaims) | Application web MVC .NET 4.5 qui utilise des **rôles** Azure AD pour l’autorisation |

## <a name="desktop-and-mobile-public-client-applications-calling-microsoft-graph-or-a-web-api"></a>Applications mobiles clientes et de bureau appelant Microsoft Graph ou une API web

Les exemples suivants illustrent des applications clientes publiques (applications de bureau/mobiles) qui accèdent à l’API web ou à Microsoft Graph pour le compte d’un utilisateur. Selon les plateformes et les appareils, les applications peuvent connecter les utilisateurs de différentes façons (flux/privilèges) :

- de manière interactive,
- de manière silencieuse (avec l’Authentification Windows intégrée sous Windows ou une combinaison nom d’utilisateur/mot de passe),
- en déléguant la connexion interactive à un autre appareil (flux de code d’appareil utilisé sur les appareils qui ne fournissent pas de contrôles web).

Application cliente | Plateforme | Flux/Grant | Appelle Microsoft Graph | Appelle une API web ASP.NET ou ASP.NET Core 2.x
------------------ | -------- | ---------- | -------------------- | -------------------------
Bureau (WPF)           | ![Cette image affiche le logo .NET/C#](media/sample-v2-code/logo-net.png)  | Interactive | Partie de [dotnet-native-multitarget](https://github.com/azure-samples/active-directory-dotnet-native-multitarget) | [Dotnet-native-desktop](https://github.com/Azure-Samples/active-directory-dotnet-native-desktop) </p> [dotnet-native-aspnetcore](https://github.com/Azure-Samples/active-directory-dotnet-native-aspnetcore/)</p> [dotnet-webapi-manual-jwt-validation](https://github.com/azure-samples/active-directory-dotnet-webapi-manual-jwt-validation)
Mobile (UWP)            | .![Cette image affiche le logo .NET/C#/UWP](media/sample-v2-code/logo-windows.png)   | Interactive | [dotnet-native-uwp-wam](https://github.com/azure-samples/active-directory-dotnet-native-uwp-wam) </p> Cet exemple utilise [WAM](/windows/uwp/security/web-account-manager), et non [ADAL.NET](https://aka.ms/adalnet)|  [dotnet-windows-store](https://github.com/Azure-Samples/active-directory-dotnet-windows-store) (application UWP utilisant ADAL.NET pour appeler une API web de locataire unique) </p> [dotnet-webapi-multitenant-windows-store](https://github.com/Azure-Samples/active-directory-dotnet-webapi-multitenant-windows-store) (application UWP utilisant ADAL.NET pour appeler une API web multilocataire)|
Mobile (Android, iOS, UWP)   | ![Cette image affiche le logo .NET/C# (Xamarin)](media/sample-v2-code/logo-xamarin.png) | Interactive | [dotnet-native-multitarget](https://github.com/azure-samples/active-directory-dotnet-native-multitarget) |
Mobile (Android)           | ![Cette image affiche le logo Android](media/sample-v2-code/logo-android.png) | Interactive |   [android](https://github.com/Azure-Samples/active-directory-android) |
Mobile (iOS)           | ![Cette image affiche iOS/Objective-C ou Swift](media/sample-v2-code/logo-ios.png) | Interactive |   [nativeClient-iOS](https://github.com/azureadquickstarts/nativeclient-ios) |
Bureau (Console)          | ![Cette image affiche le logo .NET/C#](media/sample-v2-code/logo-net.png) | Nom d’utilisateur / mot de passe </p>  Authentification Windows intégrée | | [dotnet-native-headless](https://github.com/azure-samples/active-directory-dotnet-native-headless)
Bureau (Console)          | ![Cette image affiche le logo Java](media/sample-v2-code/logo-java.png) | Nom d’utilisateur / mot de passe | | [java-native-headless](https://github.com/Azure-Samples/active-directory-java-native-headless)
Bureau (Console)           | ![Cette image affiche le logo .NET Core/C#](media/sample-v2-code/logo-netcore.png) | Flux de code d’appareil | | [dotnet-deviceprofile](https://github.com/Azure-Samples/active-directory-dotnet-deviceprofile)

## <a name="daemon-applications-accessing-web-apis-with-the-applications-identity"></a>Applications de démon (accédant à des API web avec l’identité de l’application)

Les exemples suivants illustrent des applications web ou de bureau qui accèdent à l’API web ou Microsoft Graph sans utilisateur (avec l’identité de l’application).

Application cliente | Plateforme | Flux/Grant | Appelle une API web ASP.NET ou ASP.NET Core 2.0
------------------ | -------- | ---------- | -------------------- 
Application de démon (Console)          | ![Cette image affiche le logo .NET Framework](media/sample-v2-code/logo-netframework.png) | Informations d’identification du client avec certificat ou secret d’application | [dotnet-daemon](https://github.com/azure-samples/active-directory-dotnet-daemon)</p> [dotnet-daemon-certificate-credential](https://github.com/azure-samples/active-directory-dotnet-daemon-certificate-credential)
Application de démon (Console)         | ![Cette image affiche le logo .NET Core](media/sample-v2-code/logo-netcore.png) | Informations d’identification du client avec certificat| [dotnetcore-daemon-certificate-credential](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-certificate-credential)
Application web ASP.NET  | ![Cette image affiche le logo .NET Framework](media/sample-v2-code/logo-netframework.png) | Informations d'identification du client | [dotnet-webapp-webapi-oauth2-appidentity](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-oauth2-appidentity)

## <a name="web-apis"></a>API Web

### <a name="web-api-protected-by-azure-active-directory"></a>API web protégée par Azure Active Directory

L’exemple suivant montre comment protéger une API web node.js avec Azure AD.

Dans les sections précédentes de cet article, vous pouvez également trouver d’autres exemples qui illustrent une application cliente **appelant** une **API web** ASP.NET ou ASP.NET Core. Ces exemples ne sont pas mentionnés dans cette section, mais vous les trouverez dans la dernière colonne des tableaux ci-dessus ou ci-dessous

| Plateforme | Exemple |
|--------|-------------------|
| ![Cette image affiche le logo Node.js](media/sample-v2-code/logo-nodejs.png)  | [node-webapi](https://github.com/Azure-Samples/active-directory-node-webapi) |

### <a name="web-api-calling-microsoft-graph-or-another-web-api"></a>API web appelant Microsoft Graph ou une autre API web

Les exemples suivants présentent une API web qui appelle une autre API web. Le deuxième exemple montre comment gérer l’accès conditionnel.

| Plateforme |  Appelle Microsoft Graph | Appelle une autre API web ASP.NET ou ASP.NET Core 2.0 |
| -------- |  --------------------- | ------------------------- |
| ![Cette image affiche le logo ASP.NET Framework](media/sample-v2-code/logo-netframework.png)</p> ASP.NET 4.5 | [dotnet-webapi-onbehalfof](https://github.com/azure-samples/active-directory-dotnet-webapi-onbehalfof) </p> [dotnet-webapi-onbehalfof-ca](https://github.com/azure-samples/active-directory-dotnet-webapi-onbehalfof-ca) | [dotnet-webapi-onbehalfof](https://github.com/azure-samples/active-directory-dotnet-webapi-onbehalfof) </p> [dotnet-webapi-onbehalfof-ca](https://github.com/azure-samples/active-directory-dotnet-webapi-onbehalfof-ca) |

## <a name="other-microsoft-graph-samples"></a>Autres exemples Microsoft Graph

Pour obtenir des exemples et des tutoriels qui illustrent différents modèles d’utilisation de l’API Microsoft Graph, notamment l’authentification auprès d’Azure AD, consultez [Exemples et tutoriels de la communauté Microsoft Graph](https://github.com/microsoftgraph/msgraph-community-samples).

## <a name="see-also"></a>Voir aussi

- [Guide du développeur Azure Active Directory](v1-overview.md)
- [Bibliothèques d’authentification Azure Active Directory](active-directory-authentication-libraries.md)
- [Concept et référence de l’API Microsoft Graph](/graph/use-the-api)