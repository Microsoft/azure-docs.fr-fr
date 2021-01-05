---
title: Scénarios de flux et d’applications pour l’authentification de la plateforme d’identités Microsoft | Azure
description: Découvrez les scénarios d’application pour la plateforme d’identités Microsoft, notamment l’authentification des identités, l’acquisition de jetons et l’appel d’API protégées.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 03/03/2020
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started
ms.openlocfilehash: 383bdaebaa8a1795a9bfddb7da23e172aea111a0
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96762932"
---
# <a name="authentication-flows-and-application-scenarios"></a>Flux d’authentification et scénarios d’applications

Le point de terminaison de la plateforme d’identités Microsoft (v2.0) prend en charge l’authentification pour différents types d’architectures d’applications modernes. Toutes les architectures sont basées sur les protocoles standard [OAuth 2.0 et OpenID Connect](active-directory-v2-protocols.md). Les [bibliothèques d’authentification de la plateforme d’identités Microsoft](reference-v2-libraries.md) permettent aux applications d’authentifier les identités et d’acquérir des jetons afin d’accéder aux API protégées.

Cet article décrit les flux d’authentification et les scénarios d’application dans lesquels ils sont utilisés :

## <a name="application-categories"></a>Catégories d’applications

Les jetons peuvent être obtenus à partir de plusieurs types d’applications, notamment :

- les applications web
- Applications mobiles
- Applications de bureau
- API Web

Les jetons peuvent également être obtenus par les applications qui sont exécutées sur des appareils qui n’ont pas de navigateur ou qui s’exécutent sur IoT.

Les sections suivantes décrivent les catégories d’applications.

### <a name="protected-resources-vs-client-applications"></a>Ressources protégées et applications clientes

Les scénarios d’authentification impliquent deux activités :

- **Acquisition de jetons de sécurité pour une API web protégée** : Nous vous recommandons d’utiliser les [bibliothèques de client prises en charge par Microsoft](reference-v2-libraries.md#microsoft-supported-client-libraries) pour acquérir des jetons. Nous vous recommandons plus particulièrement d’utiliser la famille de bibliothèques MSAL (Microsoft Authentication Library).
- **Protection d’une API web ou d’une application web** : l’une des tâches les plus complexes dans la protection de ces ressources est la validation du jeton de sécurité. Sur certaines plateformes, Microsoft propose des [bibliothèques d’intergiciels (middleware)](reference-v2-libraries.md#microsoft-supported-server-middleware-libraries).

### <a name="with-users-or-without-users"></a>Avec utilisateurs ou sans utilisateurs

La plupart des scénarios d’authentification acquièrent des jetons pour le compte d’utilisateurs connectés.

![Scénarios avec utilisateurs](media/scenarios/scenarios-with-users.svg)

Toutefois, il existe également des applications démon. Dans ces scénarios, les applications acquièrent des jetons pour elles-mêmes, sans intervention de l’utilisateur.

![Scénarios avec applications démon](media/scenarios/daemon-app.svg)

### <a name="single-page-public-client-and-confidential-client-applications"></a>Applications clientes monopages, publiques et confidentielles

Les jetons de sécurité peuvent être acquis par plusieurs types d’applications. On peut diviser ces applications en trois catégories. Chacune est utilisée avec différents objets et bibliothèques.

- **Applications monopages** : Également appelées « applications SPA », il s’agit d’applications web dans lesquelles les jetons sont acquis par une application JavaScript ou TypeScript qui s’exécute dans le navigateur. De nombreuses applications modernes ont un front-end d’application monopage écrit principalement en JavaScript. L’application utilise souvent un framework comme Angular, React ou Vue. MSAL.js est la seule bibliothèque d’authentification Microsoft prenant en charge les applications monopages.

- **Applications clientes publiques** : Les applications qui appartiennent à cette catégorie, comme les types d’applications qui suivent, connectent toujours les utilisateurs :
  - Applications de bureau qui appellent des API web pour le compte d’utilisateurs connectés
  - Applications mobiles
  - Applications s’exécutant sur des appareils qui n’ont pas de navigateur, comme ceux qui s’exécutent sur IoT
  
- **Applications clientes confidentielles** : Les applications appartenant à cette catégorie sont les suivantes :
  - Applications web qui appellent une API web
  - API web qui appellent une API web
  - Applications démon, même implémentées en tant que service de console tel qu’un démon Linux ou un service Windows

### <a name="sign-in-audience"></a>Audience de connexion

les flux d’authentification disponibles diffèrent en fonction de l’audience de connexion. Certains flux sont disponibles uniquement pour les comptes professionnels ou scolaires. D’autres sont disponibles pour les comptes professionnels ou scolaires, et pour les comptes Microsoft personnels.

Pour plus d’informations, consultez l’article [Types de comptes pris en charge](v2-supported-account-types.md#account-type-support-in-authentication-flows).

## <a name="application-scenarios"></a>Scénarios d’application

Le point de terminaison de la plateforme d’identités Microsoft prend en charge l’authentification pour les architectures d’application suivantes :

- Applications monopages
- les applications web
- API Web
- Applications mobiles
- Applications natives
- Applications démon
- Applications côté serveur

Les applications utilisent les différents flux d’authentification pour connecter les utilisateurs et obtenir des jetons pour appeler des API protégées.

### <a name="single-page-application"></a>Application monopage

De nombreuses applications web modernes sont créées en tant qu’applications monopages côté client. Ces applications utilisent JavaScript ou un framework comme Angular, Vue ou React. Ces applications s’exécutent dans un navigateur web.

Les applications monopages se différencient des applications web traditionnelles côté serveur au niveau des caractéristiques d’authentification. Avec la plateforme d’identités Microsoft, les applications monopages peuvent connecter des utilisateurs et obtenir des jetons pour accéder à des services back-end ou à des API web. La plateforme d’identité Microsoft propose deux types d'autorisation pour les applications JavaScript : 

| MSAL.js (2.x) | MSAL.js (1.x) |
|---|---|
| ![Une autorisation pour application monopage](media/scenarios/spa-app-auth.svg) | ![Une application monopage implicite](media/scenarios/spa-app.svg) |

### <a name="web-app-that-signs-in-a-user"></a>Application web qui connecte un utilisateur

![Une application web qui connecte un utilisateur](media/scenarios/scenario-webapp-signs-in-users.svg)

Pour protéger une application web qui connecte un utilisateur :

- Si vous développez en .NET, vous utilisez ASP.NET ou ASP.NET Core avec le middleware OpenID Connect ASP.NET. La protection d’une ressource implique la validation du jeton de sécurité, qui est effectuée par les [extensions IdentityModel pour .NET](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/wiki), et non par les bibliothèques MSAL.

- Si vous développez en Node.js, vous utilisez [Passport.js](https://github.com/AzureAD/passport-azure-ad).

Pour plus d’informations, consultez [Application web qui connecte les utilisateurs](scenario-web-app-sign-user-overview.md).

### <a name="web-app-that-signs-in-a-user-and-calls-a-web-api-on-behalf-of-the-user"></a>Application web qui connecte un utilisateur et appelle une API web pour le compte de l’utilisateur

![Une application web appelant des API web](media/scenarios/web-app.svg)

Pour appeler une API web à partir d’une application web pour le compte d’un utilisateur, utilisez le flux de code d’autorisation et stockez les jetons acquis dans le cache de jetons. Le cas échéant, MSAL actualise les jetons et le contrôleur acquiert en mode silencieux des jetons à partir du cache.

Pour plus d’informations, consultez [Application web qui appelle des API web](scenario-web-app-call-api-overview.md).

### <a name="desktop-app-that-calls-a-web-api-on-behalf-of-a-signed-in-user"></a>Application de bureau qui appelle une API web pour le compte d’un utilisateur connecté

Pour qu’une application de bureau puisse appeler une API web qui connecte des utilisateurs, utilisez les méthodes d’acquisition de jetons interactives des bibliothèques MSAL. Avec ces méthodes interactives, vous pouvez contrôler l’expérience de l’interface utilisateur de connexion. MSAL utilise un navigateur web pour cette interaction.

![Une application de bureau appelant une API web](media/scenarios/desktop-app.svg)

Il existe une autre possibilité pour les applications hébergées sur Windows qui s’exécutent sur des ordinateurs joints à un domaine Windows ou à Azure Active Directory (Azure AD). Ces applications peuvent acquérir un jeton en mode silencieux à l’aide de l’[authentification Windows intégrée](https://aka.ms/msal-net-iwa).

Les applications qui s’exécutent sur un appareil sans navigateur peuvent toujours appeler une API pour le compte d’un utilisateur. Pour s’authentifier, l’utilisateur doit se connecter sur un autre appareil doté d’un navigateur web. Ce scénario nécessite que vous utilisiez le [flux de code d’appareil](https://aka.ms/msal-net-device-code-flow).

![Flux de code d’appareil](media/scenarios/device-code-flow-app.svg)

Notez que le [flux Nom d’utilisateur/Mot de passe](scenario-desktop-acquire-token.md#username-and-password) est disponible dans les applications clientes publiques, même si nous ne recommandons pas son utilisation. Ce flux est toujours nécessaire dans certains scénarios comme DevOps.

L’utilisation du flux Nom d’utilisateur/Mot de passe contraint vos applications. Par exemple, les applications ne peuvent pas connecter un utilisateur qui doit utiliser une authentification multifacteur ou l’outil Accès conditionnel dans Azure AD. Vos applications ne bénéficient pas non plus de l’authentification unique. L’authentification à l’aide du flux Nom d’utilisateur/Mot de passe va à l’encontre des principes de l’authentification moderne et n’est fournie que pour des raisons d’héritage.

Dans les applications de bureau, si vous souhaitez que le cache de jetons soit conservé, vous pouvez personnaliser la [sérialisation du cache de jetons](scenario-desktop-acquire-token.md#file-based-token-cache). En implémentant une [double sérialisation du cache de jetons](scenario-desktop-acquire-token.md#dual-token-cache-serialization-msal-unified-cache--adal-v3), vous pouvez utiliser des caches de jetons à compatibilité descendante et ascendante. Ces jetons prennent en charge les générations précédentes de bibliothèques d’authentification. Les bibliothèques spécifiques incluent Azure AD Authentication Library pour .NET (ADAL.NET) version 3 et version 4.

Pour plus d’informations, consultez [Application de bureau qui appelle des API web](scenario-desktop-overview.md).

### <a name="mobile-app-that-calls-a-web-api-on-behalf-of-an-interactive-user"></a>Application mobile qui appelle une API web pour le compte d’un utilisateur interactif

Comme l’application de bureau, l’application mobile appelle les méthodes d’acquisition de jeton interactives MSAL afin d’acquérir un jeton pour appeler une API web.

![Une application mobile appelant une API web](media/scenarios/mobile-app.svg)

MSAL iOS et MSAL Android utilisent le navigateur web du système par défaut. Toutefois, vous pouvez leur donner pour instruction d’utiliser l’affichage web incorporé. Il existe des spécificités qui dépendent de la plateforme mobile : Plateforme Windows universelle (UWP), iOS ou Android.

Certains scénarios, comme ceux qui impliquent un accès conditionnel lié à l’identité d’appareil ou à l’inscription de l’appareil, nécessitent l’installation d’un répartiteur sur l’appareil. Le portail d’entreprise Microsoft sur Android et Microsoft Authenticator sur Android et iOS sont des exemples de répartiteurs. MSAL peut désormais interagir avec les répartiteurs. Pour plus d’informations sur les répartiteurs, consultez [Leveraging brokers on Android and iOS](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/leveraging-brokers-on-Android-and-iOS).

Pour plus d’informations, consultez [Application mobile qui appelle des API web](scenario-mobile-overview.md).

> [!NOTE]
> Vous pouvez appliquer des stratégies de protection des applications à votre application mobile qui utilise MSAL.iOS, MSAL.Android ou MSAL.NET sur Xamarin. Par exemple, les stratégies peuvent empêcher un utilisateur de copier du texte protégé. L’application mobile est gérée par Intune et reconnue par Intune en tant qu’application gérée. Pour plus d’informations, consultez l’article [Présentation du Microsoft Intune App SDK](/intune/app-sdk).
>
> Le [SDK d’application Intune](/intune/app-sdk-get-started) est distinct des bibliothèques MSAL et interagit avec Azure AD de façon autonome.

### <a name="protected-web-api"></a>API web protégée

Vous pouvez utiliser le point de terminaison de la plateforme d’identités Microsoft pour sécuriser des services web, comme l’API web RESTful de votre application. Une API web protégée est appelée à l’aide d’un jeton d’accès. Le jeton aide à sécuriser les données de l’API et authentifie les requêtes entrantes. L’appelant d’une API web ajoute un jeton d’accès dans l’en-tête d’autorisation d’une requête HTTP.

Si vous souhaitez protéger votre API web ASP.NET ou ASP.NET Core, vous devez valider le jeton d’accès. Pour cette validation, vous utilisez le middleware JWT ASP.NET. La validation est effectuée par la bibliothèque d’[extensions IdentityModel pour .NET](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/wiki), non par MSAL.NET.

Pour plus d’informations, consultez [API web protégée](scenario-protected-web-api-overview.md).

### <a name="web-api-that-calls-another-web-api-on-behalf-of-a-user"></a>API web qui appelle une autre API web pour le compte d’un utilisateur

Si vous souhaitez que votre API web protégée puisse appeler une autre API web pour le compte d’un utilisateur, votre application doit acquérir un jeton pour l’API web en aval. De tels appels sont parfois qualifiés d’appels *de service à service*. Les API web qui appellent d’autres API web doivent fournir une sérialisation de cache personnalisée.

![Une API web appelant une autre API web](media/scenarios/web-api.svg)

Pour plus d’informations, consultez [API web qui appelle des API web](scenario-web-api-call-api-overview.md).

### <a name="daemon-app-that-calls-a-web-api-in-the-daemons-name"></a>Application démon qui appelle une API web dans le nom du démon

Les applications qui contiennent des processus de longue durée ou qui fonctionnent sans interaction utilisateur doivent également disposer d’un moyen d’accès aux API web sécurisées. Ces applications peuvent s’authentifier et obtenir des jetons à l’aide de leur identité d’application. L’application prouve son identité à l’aide d’un certificat ou d’une clé secrète client.

Vous pouvez écrire des applications démon qui acquièrent un jeton pour l’application appelante à l’aide des méthodes d’acquisition des [informations d’identification du client](scenario-daemon-acquire-token.md#acquiretokenforclient-api) MSAL. Ces méthodes exigent un secret client, que vous ajoutez à l’inscription de l’application dans Azure AD. L’application partage ensuite le secret avec le démon appelé. Parmi ces secrets, citons par exemple les mots de passe d’application, l’assertion de certificat ou l’assertion du client.

![Une application démon appelée par d’autres applications et API](media/scenarios/daemon-app.svg)

Pour plus d’informations, consultez [Application démon qui appelle des API web](scenario-daemon-overview.md).

## <a name="scenarios-and-supported-authentication-flows"></a>Scénarios et flux d’authentification pris en charge

Vous pouvez utiliser les flux d’authentification pour implémenter les scénarios d’applications qui exigent des jetons. Il n’existe pas de mappage un-à-un entre des scénarios d’applications et des flux d’authentification.

Les scénarios qui impliquent l’acquisition de jetons sont également mappés à des flux d’authentification OAuth 2.0. Pour plus d’informations, consultez [Protocoles OAuth 2.0 et OpenID Connect sur la plateforme d’identités Microsoft](active-directory-v2-protocols.md).

<table>
 <thead>
  <tr><th>Scénario</th> <th>Procédure pas à pas de scénario</th> <th>Flux et octroi OAuth 2.0</th> <th>Public visé</th></tr>
 </thead>
 <tbody>
  <tr>
   <td><a href="scenario-spa-overview.md"><img alt="Single-Page App with Auth code" src="media/scenarios/spa-app-auth.svg"></a></td>
   <td><a href="scenario-spa-overview.md">Application à page unique</a></td>
   <td><a href="v2-oauth2-auth-code-flow.md">Code d’autorisation</a> avec PKCE</td>
   <td>Comptes professionnels ou scolaires, comptes personnels et Azure Active Directory B2C (Azure AD B2C)</td>
 </tr>

  <tr>
   <td><a href="scenario-spa-overview.md"><img alt="Single-Page App with Implicit" src="media/scenarios/spa-app.svg"></a></td>
   <td><a href="scenario-spa-overview.md">Application à page unique</a></td>
   <td><a href="v2-oauth2-implicit-grant-flow.md">Implicite</a></td>
   <td>Comptes professionnels ou scolaires, comptes personnels et Azure Active Directory B2C (Azure AD B2C)</td>
 </tr>

  <tr>
   <td><a href="scenario-web-app-sign-user-overview.md"><img alt="Web app that signs in users" src="media/scenarios/scenario-webapp-signs-in-users.svg"></a></td>
   <td><a href="scenario-web-app-sign-user-overview.md">Application web qui connecte les utilisateurs</a></td>
   <td><a href="v2-oauth2-auth-code-flow.md">Code d’autorisation</a></td>
   <td>Comptes professionnels ou scolaires, comptes personnels et Azure AD B2C</td>
 </tr>

  <tr>
   <td><a href="scenario-web-app-call-api-overview.md"><img alt="Web app that calls web APIs" src="media/scenarios/web-app.svg"></a></td>
   <td><a href="scenario-web-app-call-api-overview.md">Application web qui appelle des API web</a></td>
   <td><a href="v2-oauth2-auth-code-flow.md">Code d’autorisation</a></td>
   <td>Comptes professionnels ou scolaires, comptes personnels et Azure AD B2C</td>
 </tr>

  <tr>
   <td rowspan="3"><a href="scenario-desktop-overview.md"><img alt=Desktop app that calls web APIs" src="media/scenarios/desktop-app.svg"></a></td>
   <td rowspan="4"><a href="scenario-desktop-overview.md">Application de bureau qui appelle des API web</a></td>
   <td>Interactif à l’aide de <a href="v2-oauth2-auth-code-flow.md">code d’autorisation</a> avec PKCE</td>
   <td>Comptes professionnels ou scolaires, comptes personnels et Azure AD B2C</td>
 </tr>

  <tr>
   <td>Authentification Windows intégrée</td>
   <td>Comptes professionnels ou scolaires</td>
 </tr>

  <tr>
   <td><a href="v2-oauth-ropc.md">Mot de passe de propriétaire de la ressource</a></td>
   <td>Comptes professionnels ou scolaires et Azure AD B2C</td>
 </tr>

  <tr>
   <td><a href="scenario-desktop-acquire-token.md#command-line-tool-without-a-web-browser"><img alt="Browserless application" src="media/scenarios/device-code-flow-app.svg"></a></td>
   <td><a href="v2-oauth2-device-code.md">Code d’appareil</a></td>
   <td>Comptes professionnels ou scolaires, comptes personnels et Azure AD B2C</td>
 </tr>

 <tr>
   <td rowspan="2"><a href="scenario-mobile-overview.md"><img alt="Mobile app that calls web APIs" src="media/scenarios/mobile-app.svg"></a></td>
   <td rowspan="2"><a href="scenario-mobile-overview.md">Application mobile qui appelle des API web</a></td>
   <td>Interactif à l’aide de <a href="v2-oauth2-auth-code-flow.md">code d’autorisation</a> avec PKCE</td>
   <td>Comptes professionnels ou scolaires, comptes personnels et Azure AD B2C</td>
 </tr>

  <tr>
   <td><a href="v2-oauth-ropc.md">Mot de passe de propriétaire de la ressource</a></td>
   <td>Comptes professionnels ou scolaires et Azure AD B2C</td>
 </tr>

  <tr>
   <td><a href="scenario-daemon-overview.md"><img alt="Daemon app that calls web APIs" src="media/scenarios/daemon-app.svg"></a></td>
   <td><a href="scenario-daemon-overview.md">Application démon qui appelle des API web</a></td>
   <td><a href="v2-oauth2-client-creds-grant-flow.md">Informations d’identification du client</a></td>
   <td>Autorisations d’application uniquement, sans utilisateur, et utilisées uniquement dans les organisations Azure AD</td>
 </tr>

  <tr>
   <td><a href="scenario-web-api-call-api-overview.md"><img alt="Web API that calls web APIs" src="media/scenarios/web-api.svg"></a></td>
   <td><a href="scenario-web-api-call-api-overview.md">API web qui appelle des API web</a></td>
   <td><a href="v2-oauth2-on-behalf-of-flow.md">On-behalf-of</a></td>
   <td>Comptes professionnels ou scolaires et comptes personnels</td>
 </tr>

 </tbody>
</table>

## <a name="scenarios-and-supported-platforms-and-languages"></a>Scénarios avec les plateformes et langages pris en charge

Les bibliothèques d’authentification Microsoft prennent en charge plusieurs plateformes :

- JavaScript
- .NET Framework
- .NET Core
- Windows 10/UWP
- Xamarin.iOS
- Xamarin.Android
- iOS natif
- macOS
- Android natif
- Java
- Python

Vous pouvez également utiliser différents langages pour générer vos applications.

> [!NOTE]
> Certains types d’applications ne sont pas disponibles sur toutes les plateformes.

Dans la colonne Windows du tableau suivant, chaque fois que .NET Core est mentionné, .NET Framework est également possible. Ce dernier est omis pour éviter d’encombrer la table.

|Scénario  | Windows | Linux | Mac | iOS | Android
|--|--|--|--|--|--|--|
| [Application à page unique](scenario-spa-overview.md) <br/>[![Autorisation d’application monopage](media/scenarios/spa-app-auth.svg)](scenario-spa-overview.md) | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png) MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js
| [Application à page unique](scenario-spa-overview.md) <br/>[![Application monopage implicite](media/scenarios/spa-app.svg)](scenario-spa-overview.md) | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png) MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js
| [Application web qui connecte les utilisateurs](scenario-web-app-sign-user-overview.md) <br/>[![Application web qui connecte les utilisateurs](media/scenarios/scenario-webapp-signs-in-users.svg)](scenario-web-app-sign-user-overview.md) | ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core | ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core | ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core
| [Application web qui appelle des API web](scenario-web-app-call-api-overview.md) <br/> <br/>[![Application web qui appelle des API web](media/scenarios/web-app.svg)](scenario-web-app-call-api-overview.md) | ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core + MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png) <br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>Flask + MSAL Python| ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core + MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>Flask + MSAL Python| ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core + MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/> ![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>Flask + MSAL Python
| [Application de bureau qui appelle des API web](scenario-desktop-overview.md) <br/> <br/>[![Application de bureau qui appelle des API web](media/scenarios/desktop-app.svg)](scenario-desktop-overview.md) ![Flux de code de l’appareil](media/scenarios/device-code-flow-app.svg) | ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/> ![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python <br/> ![iOS / Objective C ou swift](media/sample-v2-code/small_logo_iOS.png) MSAL.objc |
| [Application mobile qui appelle des API web](scenario-mobile-overview.md) <br/> [![Application mobile qui appelle des API web](media/scenarios/mobile-app.svg)](scenario-mobile-overview.md) | ![UWP](media/sample-v2-code/small_logo_windows.png) MSAL.NET ![Xamarin](media/sample-v2-code/small_logo_xamarin.png) MSAL.NET | | | ![iOS / Objective C ou swift](media/sample-v2-code/small_logo_iOS.png) MSAL.objc | ![Android](media/sample-v2-code/small_logo_Android.png) MSAL.Android
| [Application démon](scenario-daemon-overview.md) <br/> [![Application démon](media/scenarios/daemon-app.svg)](scenario-daemon-overview.md) | ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png) MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python
| [API web qui appelle des API web](scenario-web-api-call-api-overview.md) <br/><br/> [![API web qui appelle des API web](media/scenarios/web-api.svg)](scenario-web-api-call-api-overview.md) | ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core + MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core + MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core + MSAL.NET ![MSAL Java](media/sample-v2-code/small_logo_java.png)<br/>MSAL Java<br/>![MSAL Python](media/sample-v2-code/small_logo_python.png)<br/>MSAL Python

Pour plus d’informations, consultez [Prise en charge Microsoft des bibliothèques selon le système d’exploitation et le langage](reference-v2-libraries.md#microsoft-supported-libraries-by-os--language).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [notions de base de l’authentification](./authentication-vs-authorization.md) et les [jetons d’accès dans la plateforme d’identités Microsoft](access-tokens.md)
* En savoir plus sur la [sécurisation de l’accès aux applications IoT](/azure/architecture/example-scenario/iot-aad/iot-aad)
