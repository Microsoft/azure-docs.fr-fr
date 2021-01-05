---
title: Exemples de code pour la plateforme d’identité Microsoft
description: Fournit un index des exemples de code de la plateforme d’identité Microsoft (point de terminaison v2.0) disponibles, organisés par scénario.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: sample
ms.workload: identity
ms.date: 11/04/2020
ms.author: marsma
ms.reviewer: jmprieur
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 169a8dba979571a5f4b8c8056737fce72db1d058
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97629865"
---
# <a name="microsoft-identity-platform-code-samples-v20-endpoint"></a>Exemples de code de la plateforme d’identité Microsoft (point de terminaison v2.0)

Vous pouvez utiliser la plateforme d’identités Microsoft pour effectuer les opérations suivantes :

- Ajouter l’authentification et l’autorisation à vos applications web et API web.
- Exiger un jeton d’accès pour accéder à une API web protégée.

Cet article fournit une brève description et des liens vers des exemples pour le point de terminaison V2 de la plateforme d’identité Microsoft. Ces exemples vous montrent comment procéder, et fournissent des extraits de code que vous pouvez utiliser dans vos applications. Dans la page d’exemples de code, vous trouverez des rubriques Lisez-moi détaillées, qui offrent une aide relative à la configuration requise, à l’installation et à la configuration. Les commentaires figurant dans le code sont là pour vous aider à comprendre les sections critiques.

Pour comprendre le scénario de base pour chaque type d’exemple, voir [Types d’applications pour la plateforme d’identité Microsoft](v2-app-types.md).

Vous pouvez également contribuer aux exemples sur GitHub. Pour en savoir plus, consultez [Documentation et exemples Microsoft Azure Active Directory](https://github.com/Azure-Samples?page=3&query=active-directory).

## <a name="single-page-applications"></a>Applications monopages

Ces exemples montrent comment écrire une application monopage sécurisée avec la plateforme d’identité Microsoft. Ils utilisent l’une des saveurs disponibles de MSAL.js.

| Plateforme | Description | Lien |
| -------- | --------------------- | -------- |
| ![Cette image montre le logo JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core) | L’application à page unique (SPA) appelle Microsoft Graph |[javascript-graphapi-v2](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2) |
| ![Cette image montre le logo JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser) | L’application à page unique (SPA) appelle Microsoft Graph en utilisant un flux de code d’authentification avec PKCE |[JavaScript-v2](https://github.com/Azure-Samples/ms-identity-javascript-v2) |
| ![Cette image montre le logo JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core) | L’application à page unique (SPA) appelle B2C |[b2c-javascript-msal-singlepageapp](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp) |
| ![Cette image montre le logo JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser) | L’application monopage appelle B2C en utilisant un flux de code d’authentification avec PKCE |[b2c-javascript-spa](https://github.com/Azure-Samples/ms-identity-b2c-javascript-spa) |
| ![Cette image montre le logo Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL-Angular)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular)| L’application à page unique (SPA) appelle Microsoft Graph  | [active-directory-javascript-singlepageapp-angular](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-angular) |
| ![Cette image montre le logo Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL-Angular)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular)| L’application à page unique (SPA) appelle une API web personnalisée | [ms-identity-javascript-angular-spa-aspnetcore-webapi](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnetcore-webapi) |
| ![Cette image montre le logo Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL-Angular)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular) | L’application à page unique (SPA) appelle B2C |[active-directory-b2c-javascript-angular-spa](https://github.com/Azure-Samples/active-directory-b2c-javascript-angular-spa) |
| ![Cette image montre le logo React](media/sample-v2-code/logo_react.png) [React (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core)| L’application à page unique (SPA) appelle une API web qui, à son tour, appelle Microsoft Graph  | [ms-identity-javascript-react-spa-dotnetcore-webapi-obo](https://github.com/Azure-Samples/ms-identity-javascript-react-spa-dotnetcore-webapi-obo) |
| ![Cette image montre le logo JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser) | L’application monopage appelle une API web qui, à son tour, appelle Microsoft Graph  | [ms-identity-javascript-tutorial-chapter4-obo](https://github.com/Azure-Samples/ms-identity-javascript-tutorial/tree/main/4-AdvancedGrants/1-call-api-graph) |
| ![Cette image montre le logo Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL-Angular)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular) | L’application à page unique (SPA) appelle une API web personnalisée avec des rôles d’application et des groupes de sécurité |[ms-identity-javascript-angular-spa-dotnetcore-webapi-roles-groups](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-dotnetcore-webapi-roles-groups) |
| ![Cette image montre le logo Blazor](media/sample-v2-code/logo-blazor.png) [Blazor WebAssembly (MSAL-JS)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser) | Tutoriel Blazor WebAssembly pour connecter les utilisateurs et appeler les API avec Azure Active Directory |[ms-identity-blazor-wasm](https://github.com/Azure-Samples/ms-identity-blazor-wasm) |

## <a name="web-applications"></a>Applications web

Les exemples suivants illustrent des applications web qui connectent les utilisateurs. Certains exemples montrent également l’application qui appelle Microsoft Graph ou votre propre API web avec l’identité de l’utilisateur.

| Plateforme | Connecte uniquement les utilisateurs | Connecte les utilisateurs et appelle Microsoft Graph |
| -------- | ------------------- | --------------------------------- |
| ![Cette illustration montre le logo ASP.NET Core](media/sample-v2-code/logo_NETcore.png)</p>ASP.NET Core | [Didacticiel de connexion des utilisateurs par la WebApp ASP.NET Core](https://aka.ms/aspnetcore-webapp-sign-in) | Même exemple dans la phase où [l’application web ASP.NET Core appelle Microsoft Graph](https://aka.ms/aspnetcore-webapp-call-msgraph)</p>Exemple avancé [Accès au cache de jeton de l’utilisateur connecté à partir d’applications en arrière-plan, d’API et de services](https://github.com/Azure-Samples/ms-identity-dotnet-advanced-token-cache) |
| ![Cette image affiche le logo ASP.NET Framework](media/sample-v2-code/logo_NETframework.png)</p>ASP.NET Core | [Playbook de migration d’application AD FS vers Azure AD pour les développeurs](https://github.com/Azure-Samples/ms-identity-dotnet-adfs-to-aad) pour découvrir comment migrer de manière sécurisée vos applications intégrées à Active Directory Federation Services (AD FS) vers Azure Active Directory (Azure AD) | |
| ![Cette image affiche le logo ASP.NET Framework](media/sample-v2-code/logo_NETframework.png)</p> ASP.NET | [Démarrage rapide avec ASP.NET](https://github.com/AzureAdQuickstarts/AppModelv2-WebApp-OpenIDConnect-DotNet) </p> [dotnet-webapp-openidconnect-v2](https://github.com/azure-samples/active-directory-dotnet-webapp-openidconnect-v2)  |  [dotnet-admin-restricted-scopes-v2](https://github.com/azure-samples/active-directory-dotnet-admin-restricted-scopes-v2) </p> |[msgraph-training-aspnetmvcapp](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp)
| ![Cette image affiche le logo Java](media/sample-v2-code/logo_java.png)  |                   | [ms-identity-java-webapp](https://github.com/Azure-Samples/ms-identity-java-webapp) |
| ![Cette image affiche le logo Java](media/sample-v2-code/logo_java.png)  | [ms-identity-b2c-java-servlet-webapp-authentication](https://github.com/Azure-Samples/ms-identity-b2c-java-servlet-webapp-authentication)|  |
| ![Cette image affiche le logo Python](media/sample-v2-code/logo_python.png)  | [ms-identity-python-flask-webapp-authentication](https://github.com/Azure-Samples/ms-identity-python-flask-webapp-authentication) | [ms-identity-python-webapp](https://github.com/Azure-Samples/ms-identity-python-webapp) |
| ![Cette image affiche le logo Python](media/sample-v2-code/logo_python.png)  | Tutoriel sur la connexion des utilisateurs avec l’[application web Python Flask](https://github.com/Azure-Samples/ms-identity-python-flask-tutorial/blob/main/README.md)                   | Tutoriel sur la connexion des utilisateurs et l’appel de Graph avec l’[application web Python Flask](https://github.com/Azure-Samples/ms-identity-python-flask-tutorial/blob/main/README.md) |
| ![Cette image affiche le logo Python](media/sample-v2-code/logo_python.png)  | Tutoriel sur la connexion des utilisateurs par B2C avec l’[application web Python Flask](https://github.com/Azure-Samples/ms-identity-b2c-python-flask-webapp-authentication/blob/main/README.md)                    |  |
| ![Cette image affiche le logo Ruby](media/sample-v2-code/logo_ruby.png) |                   | [msgraph-training-rubyrailsapp](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp) |
| ![Cette image montre le logo Blazor](media/sample-v2-code/logo-blazor.png)</p>Serveur Blazor | [Tutoriel sur la connexion des utilisateurs avec l’application Blazor Server](https://github.com/Azure-Samples/ms-identity-blazor-server/tree/main/WebApp-OIDC) | [L’application Blazor Server appelle Microsoft Graph](https://github.com/Azure-Samples/ms-identity-blazor-server/tree/main/WebApp-graph-user/Call-MSGraph)</p>Tutoriel par chapitre : [Application Blazor Server pour connecter les utilisateurs et appeler les API avec Azure Active Directory](https://github.com/Azure-Samples/ms-identity-blazor-server) |

## <a name="desktop-and-mobile-public-client-apps"></a>Applications clientes publiques mobiles et de bureau

Les exemples suivants illustrent des applications clientes publiques (applications de bureau ou mobiles) qui accèdent à l’API Graph ou à votre API web pour le compte d’un utilisateur. Outre l’exemple *Desktop (console) avec WAM*, toutes ces applications clientes utilisent la bibliothèque d’authentification Microsoft (MSAL).

| Application cliente | Plateforme | Flux/octroi | Appelle Microsoft Graph | Appelle une API web ASP.NET Core |
| ------------------ | -------- |  ----------| ---------- | ------------------------- |
| Tutoriel de bureau (.NET Core) - Utiliser éventuellement :</p>- le cache de jeton Implémente multiplateforme</p>- une interface web personnalisée | ![Cette image affiche le logo .NET/C#](media/sample-v2-code/logo_NETcore.png) | [Code d’autorisation](msal-authentication-flows.md#authorization-code)| [ms-identity-dotnet-desktop-tutorial](https://github.com/azure-samples/ms-identity-dotnet-desktop-tutorial) | |
| Bureau (WPF)      | ![Cette image affiche le logo de bureau .NET/C#](media/sample-v2-code/logo_NET.png) | [Code d’autorisation](msal-authentication-flows.md#authorization-code)| [dotnet-desktop-msgraph-v2](https://github.com/azure-samples/active-directory-dotnet-desktop-msgraph-v2) | [dotnet-native-aspnetcore-v2](https://aka.ms/msidentity-aspnetcore-webapi) |
| Bureau (Console)   | ![Image montrant le logo .NET/C# (bureau)](media/sample-v2-code/logo_NET.png) | [Authentification Windows intégrée](msal-authentication-flows.md#integrated-windows-authentication) | [dotnet-iwa-v2](https://github.com/azure-samples/active-directory-dotnet-iwa-v2) |  |
| Bureau (Console)   | ![Cette image affiche le logo Java](media/sample-v2-code/logo_java.png) | [Authentification Windows intégrée](msal-authentication-flows.md#integrated-windows-authentication) |[ms-identity-java-desktop](https://github.com/Azure-Samples/ms-identity-java-desktop/) |  |
| Bureau (Console)   | ![Voici le logo .NET/C# (bureau)](media/sample-v2-code/logo_NETcore.png) | [Nom d’utilisateur/mot de passe](msal-authentication-flows.md#usernamepassword) |[dotnetcore-up-v2](https://github.com/azure-samples/active-directory-dotnetcore-console-up-v2) |  |
| Bureau (console) avec WAM  | ![Voici le logo pour .NET/C# (bureau)](media/sample-v2-code/logo_NETcore.png) | Interactif avec [Gestionnaire de comptes web](/windows/uwp/security/web-account-manager) (WAM) |[dotnet-native-uwp-wam](https://github.com/azure-samples/active-directory-dotnet-native-uwp-wam) |  |
| Bureau (Console)   | ![Cette image affiche le logo Java](media/sample-v2-code/logo_java.png) | [Nom d’utilisateur/mot de passe](msal-authentication-flows.md#usernamepassword) |[ms-identity-java-desktop](https://github.com/Azure-Samples/ms-identity-java-desktop/) |  |
| Bureau (Console)   | ![Cette image affiche le logo Python](media/sample-v2-code/logo_python.png) | [Nom d’utilisateur/mot de passe](msal-authentication-flows.md#usernamepassword) |[ms-identity-python-desktop](https://github.com/Azure-Samples/ms-identity-python-desktop) |  |
| Mobile (Android, iOS, UWP)   | ![Cette image affiche le logo .NET/C# (Xamarin)](media/sample-v2-code/logo_xamarin.png) | [Code d’autorisation](msal-authentication-flows.md#authorization-code) |[xamarin-native-v2](https://github.com/azure-samples/active-directory-xamarin-native-v2) |  |
| Mobile (iOS)       | ![Cette image affiche iOS/Objective-C ou Swift](media/sample-v2-code/logo_iOS.png) | [Code d’autorisation](msal-authentication-flows.md#authorization-code) |[ios-swift-objc-native-v2](https://github.com/azure-samples/active-directory-ios-swift-native-v2) </p> [ios-native-nxoauth2-v2](https://github.com/azure-samples/active-directory-ios-native-nxoauth2-v2) |  |
| Bureau (macOS)       | macOS | [Code d’autorisation](msal-authentication-flows.md#authorization-code) |[macOS-swift-objc-native-v2](https://github.com/Azure-Samples/ms-identity-macOS-swift-objc) |  |
| Mobile (Android-Java)   | ![Cette image affiche le logo Android](media/sample-v2-code/logo_Android.png) | [Code d’autorisation](msal-authentication-flows.md#authorization-code) |  [Android-Java](https://github.com/Azure-Samples/ms-identity-android-java) |  |
| Mobile (Android-Kotlin)   | ![Cette image affiche le logo Android](media/sample-v2-code/logo_Android.png) | [Code d’autorisation](msal-authentication-flows.md#authorization-code) |  [Android-Kotlin](https://github.com/Azure-Samples/ms-identity-android-kotlin) |  |

## <a name="daemon-applications"></a>Applications de démon

L’exemple suivant présente une application qui accède à l’API Microsoft Graph avec sa propre identité (sans utilisateur).

| Application cliente | Plateforme | Flux/Grant | Appelle Microsoft Graph |
| ------------------ | -------- | ---------- | -------------------- |
| Console | ![Cette image affiche le logo .NET Core](media/sample-v2-code/logo_NETcore.png)</p> ASP.NET  | [Informations d’identification du client](msal-authentication-flows.md#client-credentials) | [dotnetcore-daemon-v2](https://github.com/azure-samples/active-directory-dotnetcore-daemon-v2) |
| Application web | ![Capture d’écran montrant le logo ASP.NET.](media/sample-v2-code/logo_NETframework.png)</p> ASP.NET  | [Informations d’identification du client](msal-authentication-flows.md#client-credentials) | [dotnet-daemon-v2](https://github.com/azure-samples/active-directory-dotnet-daemon-v2) |
| Console | ![Cette image affiche le logo Java](media/sample-v2-code/logo_java.png) | [Informations d’identification du client](msal-authentication-flows.md#client-credentials) | [ms-identity-java-daemon](https://github.com/Azure-Samples/ms-identity-java-daemon) |
| Console | ![Cette image affiche le logo Python](media/sample-v2-code/logo_python.png) | [Informations d’identification du client](msal-authentication-flows.md#client-credentials) | [ms-identity-python-daemon](https://github.com/Azure-Samples/ms-identity-python-daemon) |

## <a name="headless-applications"></a>Applications sans affichage

L’exemple suivant présente une application cliente publique qui s’exécute sur un appareil sans navigateur web. Cette application peut être un outil en ligne de commande ou une application qui s’exécute sur un système Linux ou Mac, ou encore une application IoT. Cet exemple présente une application accédant à l’API Graph au nom d’un utilisateur qui se connecte de manière interactive sur un autre appareil (par exemple, un téléphone mobile). Cette application cliente utilise une bibliothèque d’authentification Microsoft (MSAL).

| Application cliente | Plateforme | Flux/Grant | Appelle Microsoft Graph |
| ------------------ | -------- |  ----------| ---------- |
| Bureau (Console)   | ![Cette image affiche le logo .NET/C# (bureau)](media/sample-v2-code/logo_NETcore.png) | [Flux de code d’appareil](msal-authentication-flows.md#device-code) |[dotnetcore-devicecodeflow-v2](https://github.com/azure-samples/active-directory-dotnetcore-devicecodeflow-v2) |
| Bureau (Console)   | ![Cette image affiche le logo Java](media/sample-v2-code/logo_java.png) | [Flux de code d’appareil](msal-authentication-flows.md#device-code) |[ms-identity-java-devicecodeflow](https://github.com/Azure-Samples/ms-identity-java-devicecodeflow) |
| Bureau (Console)   | ![Cette image affiche le logo Python](media/sample-v2-code/logo_python.png) | [Flux de code d’appareil](msal-authentication-flows.md#device-code) |[ms-identity-python-devicecodeflow](https://github.com/Azure-Samples/ms-identity-python-devicecodeflow) |

## <a name="multi-tenant-saas-applications"></a>Applications SaaS multilocataires

Les exemples suivants montrent comment configurer votre application de façon à lui faire accepter les connexions de n’importe quel locataire Azure Active Directory (Azure AD). En configurant votre application comme étant *multilocataire*, vous pouvez proposer une application **SaaS** (Software as a Service) à de nombreuses organisations, permettant ainsi à leurs utilisateurs de se connecter à votre application après avoir consenti à utiliser leur compte.

| Plateforme | Description | Lien |
| -------- | --------------------- | -------- |
| ![Cette image montre le logo JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser) | Application monopage (SPA) multilocataire appelant l’API Graph |[ms-identity-javascript-angular-spa-aspnet-webapi-multitenant](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnet-webapi-multitenant/tree/master/Chapter1) |
| ![Cette image montre le logo Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL-Angular)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular) | L’application à page unique (SPA) multi-locataire appelle une API web personnalisée multi-locataire |[ms-identity-javascript-angular-spa-aspnet-webapi-multitenant](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnet-webapi-multitenant/tree/master/Chapter2) |
| ![Cette image montre le logo ASP.NET Core](media/sample-v2-code/logo_NETcore.png) [.NET Core (MSAL.NET)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) | Application web ASP.NET Core MVC appelant l’API Graph |[active-directory-aspnetcore-webapp-openidconnect-v2](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-3-Multi-Tenant) |
| ![Cette image montre le logo ASP.NET Core](media/sample-v2-code/logo_NETcore.png) [.NET Core (MSAL.NET)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) | Application web ASP.NET Core MVC appelant l’API Web ASP.NET Core |[active-directory-aspnetcore-webapp-openidconnect-v2](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/4-WebApp-your-API/4-3-AnyOrg) |

## <a name="web-apis"></a>API Web

Les exemples suivants montrent comment protéger une API web avec le point de terminaison de la plateforme d’identité Microsoft, et comment appeler une API en aval à partir de l’API web.

| Plateforme | Exemple |
| -------- | ------------------- |
| ![Cette illustration montre le logo ASP.NET Core](media/sample-v2-code/logo_NETcore.png)</p>ASP.NET Core | API web ASP.NET Core (service) associée à [dotnet-native-aspnetcore-v2](https://aka.ms/msidentity-aspnetcore-webapi-calls-msgraph)  |
| ![Cette image affiche le logo ASP.NET](media/sample-v2-code/logo_NET.png)</p>ASP.NET MVC | API web (service) associée à [ms-identity-aspnet-webapi-onbehalfof](https://github.com/Azure-Samples/ms-identity-aspnet-webapi-onbehalfof) |
| ![Cette image affiche le logo Java](media/sample-v2-code/logo_java.png) | (service) API web de [ms-identity-java-webapi](https://github.com/Azure-Samples/ms-identity-java-webapi) |
| ![Cette image affiche le logo Node.js](media/sample-v2-code/logo_nodejs.png) | API web (service) de [active-directory-javascript-nodejs-webapi-v2](https://github.com/Azure-Samples/active-directory-javascript-nodejs-webapi-v2) |
| ![Cette image affiche le logo Node.js](media/sample-v2-code/logo_nodejs.png) | API web B2C (service) de [active-directory-b2c-javascript-nodejs-webapi](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi) |

## <a name="azure-functions-as-web-apis"></a>Fonctions Azure en tant qu’API web

Les exemples suivants montrent comment protéger une fonction Azure en utilisant HttpTrigger et en exposant une API web avec le point de terminaison de la plateforme d’identité Microsoft, et comment appeler une API en aval à partir de l’API web.

| Plateforme | Exemple |
| -------- | ------------------- |
| ![Cette illustration montre le logo ASP.NET Core](media/sample-v2-code/logo_NETcore.png)</p>ASP.NET Core | Fonction Azure avec API web ASP.NET Core (service) de [dotnet-native-aspnetcore-v2](https://github.com/Azure-Samples/ms-identity-dotnet-webapi-azurefunctions)  |
| ![Cette image affiche le logo Python](media/sample-v2-code/logo_python.png)</p>Python | API web (service) de [Python](https://github.com/Azure-Samples/ms-identity-python-webapi-azurefunctions) |
| ![Cette image affiche le logo Node.js](media/sample-v2-code/logo_nodejs.png)</p>Node.js | API web (service) de [Node.js and passport-azure-ad](https://github.com/Azure-Samples/ms-identity-nodejs-webapi-azurefunctions) |
| ![Cette image affiche le logo Node.js](media/sample-v2-code/logo_nodejs.png)</p>NodeJS | API web (service) de [NodeJS and passport-azure-ad using on behalf of](https://github.com/Azure-Samples/ms-identity-nodejs-webapi-onbehalfof-azurefunctions) |

## <a name="other-microsoft-graph-samples"></a>Autres exemples Microsoft Graph

Pour découvrir des [exemples](https://github.com/microsoftgraph/msgraph-community-samples/tree/master/samples#aspnet) et des tutoriels qui illustrent différents modèles d’utilisation de l’API Microsoft Graph, notamment l’authentification auprès d’Azure AD, consultez [Exemples et tutoriels de la communauté Microsoft Graph](https://github.com/microsoftgraph/msgraph-community-samples).

## <a name="see-also"></a>Voir aussi

[Concept et référence de l’API Microsoft Graph](/graph/use-the-api?context=graph%2fapi%2fbeta&view=graph-rest-beta&preserve-view=true)
