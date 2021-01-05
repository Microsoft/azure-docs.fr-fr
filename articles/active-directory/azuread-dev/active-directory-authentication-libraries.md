---
title: Bibliothèques d’authentification Azure Active Directory | Microsoft Docs
description: La bibliothèque d'authentification Azure AD (ADAL) permet aux développeurs d’applications clientes d’authentifier facilement les utilisateurs sur Active Directory (AD) en local ou sur le cloud, puis d'obtenir des jetons d'accès pour sécuriser les appels d'API.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.topic: reference
ms.workload: identity
ms.date: 12/01/2018
ms.author: ryanwi
ms.reviewer: saeeda, jmprieur
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: 7b6ffd885e1be2dd59cea87cacd6e5fd5e0f8f49
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861168"
---
# <a name="azure-active-directory-authentication-libraries"></a>Bibliothèques d’authentification d’Azure Active Directory

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

La bibliothèque d’authentification Azure Active Directory v1.0 (ADAL, Azure Active Directory Authentication Library) permet aux développeurs d’applications d’authentifier les utilisateurs dans une instance locale d’Active Directory (AD) ou dans le cloud, puis d’obtenir des jetons pour sécuriser les appels d’API. La bibliothèque ADAL facilite l’authentification pour les développeurs grâce à des fonctionnalités comme :

- Un cache de jeton configurable qui stocke les jetons d’accès et les jetons d’actualisation
- L’actualisation automatique des jetons lorsqu’un jeton d’accès expire et qu’un jeton d’actualisation est disponible
- La prise en charge des appels de méthode asynchrones

> [!NOTE]
> Vous recherchez les bibliothèques Azure AD v2.0 (MSAL) ? Consultez le [guide des bibliothèques MSAL](../develop/reference-v2-libraries.md).
>
>

## <a name="microsoft-supported-client-libraries"></a>Bibliothèques clientes prises en charge par Microsoft

| Plateforme | Bibliothèque | Téléchargement | Code source | Exemple | Informations de référence
| --- | --- | --- | --- | --- | --- |
| .NET Client, Windows Store, UWP, Xamarin iOS et Android |ADAL .NET v3 |[NuGet](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet) | [Application de bureau](../develop/quickstart-v2-windows-desktop.md) |[Référence](/dotnet/api/microsoft.identitymodel.clients.activedirectory) |
| JavaScript |ADAL.js |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-js) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-js) |[Application à page unique](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi) | |
| iOS, macOS |ADAL |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-objc/releases) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-objc) |[Application iOS](../develop/quickstart-v2-ios.md) | [Référence](http://cocoadocs.org/docsets/ADAL/2.5.1/)|
| Android |ADAL |[Maven](https://search.maven.org/search?q=g:com.microsoft.aad+AND+a:adal&core=gav) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-android) |[Application Android](../develop/quickstart-v2-android.md) | [JavaDocs](https://javadoc.io/doc/com.microsoft.aad/adal/)|
| Node.js |ADAL |[npm](https://www.npmjs.com/package/adal-node) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-nodejs) | [Application web Node.js](https://github.com/Azure-Samples/active-directory-node-webapp-openidconnect)|[Référence](/javascript/api/overview/azure/activedirectory) |
| Java |ADAL4J |[Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3Aadal4j%20g%3Acom.microsoft.azure) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-java) |[Applications web Java](https://github.com/Azure-Samples/active-directory-java-webapp-openidconnect) |[Référence](https://javadoc.io/doc/com.microsoft.azure/adal4j) |
| Python |ADAL |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-python) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-python) |[Application Web Python](https://github.com/Azure-Samples/active-directory-python-webapp-graphapi) |[Référence](https://adal-python.readthedocs.io/) |

## <a name="microsoft-supported-server-libraries"></a>Bibliothèques serveur prises en charge par Microsoft

| Plateforme | Bibliothèque | Téléchargement | Code source | Exemple | Informations de référence
| --- | --- | --- | --- | --- | --- |
| .NET |OWIN pour Azure AD|[NuGet](https://www.nuget.org/packages/Microsoft.Owin.Security.ActiveDirectory/) |[GitHub](https://github.com/aspnet/AspNetKatana/tree/dev/src/Microsoft.Owin.Security.ActiveDirectory) |[Application MVC](../develop/quickstart-v2-aspnet-webapp.md) | |
| .NET |OWIN pour OpenIDConnect |[NuGet](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect) |[GitHub](https://github.com/aspnet/AspNetKatana/tree/dev/src/Microsoft.Owin.Security.OpenIdConnect) |[Application web](https://github.com/AzureADSamples/WebApp-OpenIDConnect-DotNet) | |
| .NET |OWIN pour WS-Federation |[NuGet](https://www.nuget.org/packages/Microsoft.Owin.Security.WsFederation) |[GitHub](https://github.com/aspnet/AspNetKatana/tree/dev/src/Microsoft.Owin.Security.WsFederation) |[Application Web MVC](https://github.com/AzureADSamples/WebApp-WSFederation-DotNet) | |
| .NET |Extensions de protocole d’identité pour .NET 4.5 |[NuGet](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocol.Extensions) |[GitHub](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet) | | |
| .NET |Gestionnaire JWT pour .NET 4.5 |[NuGet](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) |[GitHub](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet) | | |
| Node.js |Azure AD Passport |[npm](https://www.npmjs.com/package/passport-azure-ad) |[GitHub](https://github.com/AzureAD/passport-azure-ad) | [API Web](../develop/authentication-flows-app-scenarios.md)| |

## <a name="scenarios"></a>Scénarios

Voici trois scénarios courants d’utilisation de la bibliothèque ADAL sur un client qui accède à une ressource distante :

### <a name="authenticating-users-of-a-native-client-application-running-on-a-device"></a>Authentification des utilisateurs d’une application cliente native exécutée sur un appareil

Dans ce scénario, un développeur dispose d’une application mobile cliente ou de bureau qui doit accéder à une ressource distante, par exemple une API web. L’API web n’autorise pas les appels anonymes et doit être appelée dans le contexte d’un utilisateur authentifié. L’API web est préconfigurée de manière à approuver les jetons d’accès émis par un locataire Azure AD spécifique. Azure AD est préconfiguré pour émettre des jetons d’accès pour cette ressource. Pour appeler l’API web à partir du client, le développeur utilise la bibliothèque ADAL afin de faciliter l’authentification auprès d’Azure AD. La méthode d’utilisation de la bibliothèque ADAL la plus sûre consiste à faire en sorte qu’elle affiche l’interface utilisateur pour recueillir les informations d’identification de l’utilisateur (sous forme de fenêtre de navigateur).

La bibliothèque ADAL facilite l’authentification de l’utilisateur, l’obtention d’un jeton d’accès et d’un jeton d’actualisation à partir d’Azure AD, puis l’appel de l’API web à l’aide du jeton d’accès.

Pour voir un exemple de code qui illustre ce scénario en utilisant l’authentification auprès d’Azure AD, consultez la section [Native Client WPF Application to Web API](https://github.com/azureadsamples/nativeclient-dotnet).

### <a name="authenticating-a-confidential-client-application-running-on-a-web-server"></a>Authentification d’une application cliente confidentielle exécutée sur un serveur web

Dans ce scénario, un développeur a une application fonctionnant sur un serveur qui doit accéder à une ressource distante, par exemple une API web. L’API web n’autorisant pas les appels anonymes, elle doit être appelée à partir d’un service autorisé. L’API web est préconfigurée de manière à approuver les jetons d’accès émis par un locataire Azure AD spécifique. Azure AD est préconfiguré pour délivrer des jetons d’accès pour cette ressource à un service avec des informations d’identification client (ID client et secret). La bibliothèque ADAL facilite l’authentification du service auprès d’Azure AD en retournant un jeton d’accès qui peut être utilisé pour appeler l’API web. La bibliothèque ADAL gère également la durée de vie du jeton d'accès en le mettant en cache et en le renouvelant si nécessaire. Pour voir un exemple de code qui illustre ce scénario, consultez [Daemon console Application to Web API](https://github.com/AzureADSamples/Daemon-DotNet).

### <a name="authenticating-a-confidential-client-application-running-on-a-server-on-behalf-of-a-user"></a>Authentification d’une application cliente confidentielle exécutée sur un serveur, pour le compte d’un utilisateur

Dans ce scénario, un développeur a une application web fonctionnant sur un serveur qui doit accéder à une ressource distante, par exemple une API web. L’API web n’autorisant pas les appels anonymes, elle doit être appelée à partir d’un service autorisé pour le compte d’un utilisateur authentifié. L’API web est préconfigurée de manière à approuver les jetons d’accès émis par un locataire Azure spécifique, et Azure AD est préconfiguré pour délivrer des jetons d’accès pour cette ressource à un service avec des informations d’identification client. Une fois l’utilisateur authentifié dans l’application web, l’application peut obtenir d’Azure AD un code d’autorisation pour l’utilisateur. L'application web peut ensuite utiliser ADAL pour obtenir d’Azure AD un jeton d'accès et un jeton d’actualisation pour le compte de l'utilisateur en utilisant le code d'autorisation et les informations d’identification client associés à l'application. Une fois l'application web en possession du jeton d'accès, elle peut appeler l'API web jusqu’à expiration du jeton. Lorsque le jeton expire, l'application web peut utiliser la bibliothèque ADAL pour obtenir un nouveau jeton d'accès en utilisant le jeton d’actualisation reçu précédemment. Pour voir un exemple de code qui illustre ce scénario, consultez [Native client to Web API to Web API](https://github.com/Azure-Samples/active-directory-dotnet-webapi-onbehalfof).

## <a name="see-also"></a>Voir aussi

- [Le guide du développeur Azure Active Directory](v1-overview.md)
- [Scénarios d’authentification pour Azure Active Directory](v1-authentication-scenarios.md)
- [Exemples de code Azure Active Directory](sample-v1-code.md)
