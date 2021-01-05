---
title: Exemples de code Azure Active Directory B2C | Microsoft Docs
description: Exemples de code pour des applications mobiles, de bureau, web et monopages Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 10/02/2020
ms.custom: mvc
ms.topic: sample
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: 8642f4284770df54704a4e7066054d3bb4640486
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96545866"
---
# <a name="azure-active-directory-b2c-code-samples"></a>Exemples de code Azure Active Directory B2C

Les tableaux suivants fournissent des liens vers des exemples pour des applications iOS, Android, .Net et Node.js, entre autres.

## <a name="mobile-and-desktop-apps"></a>Applications de bureau et mobiles

| Exemple | Description |
|--------| ----------- |
| [ios-swift-native-msal](https://github.com/Azure-Samples/active-directory-b2c-ios-swift-native-msal) | Un exemple iOS dans Swift qui authentifie les utilisateurs Azure AD B2C et appelle une API à l’aide d’OAuth 2.0 |
| [android-native-msal](https://github.com/Azure-Samples/ms-identity-android-java#b2cmodefragment-class) | Une application Android simple expliquant comment utiliser MSAL pour authentifier les utilisateurs via Azure Active Directory B2C et accéder à une API web avec les jetons générés. |
| [ios-native-appauth](https://github.com/Azure-Samples/active-directory-b2c-ios-native-appauth) | Un exemple qui montre comment vous pouvez utiliser une bibliothèque tierce pour générer une application iOS en Objective-C qui authentifie les utilisateurs d’identités Microsoft auprès de notre service d’identité Azure AD B2C. |
| [android-native-appauth](https://github.com/Azure-Samples/active-directory-b2c-android-native-appauth) | Un exemple qui montre comment vous pouvez utiliser une bibliothèque tierce pour générer une application Android qui authentifie les utilisateurs d’identités Microsoft auprès de notre service d’identité B2C et qui appelle une API web à l’aide de jetons d’accès OAuth 2.0. |
| [dotnet-desktop](https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop) | Un exemple qui montre comment une application .NET de bureau Windows (WPF) peut connecter un utilisateur à l’aide d’Azure AD B2C, récupérer un jeton d’accès à l’aide de MSAL.NET et appeler une API. |
| [xamarin-native](https://github.com/Azure-Samples/active-directory-b2c-xamarin-native) | Une application Xamarin Forms simple expliquant comment utiliser MSAL pour authentifier les utilisateurs via Azure Active Directory B2C et accéder à une API web avec les jetons générés. |

## <a name="web-apps-and-apis"></a>API et applications web

| Exemple | Description |
|--------| ----------- |
| [dotnet-webapp-and-webapi](https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi) | Un exemple combiné d’application web .NET qui appelle une API web .NET, toutes deux protégées via Azure AD B2C. |
| [dotnetcore-webapp-openidconnect](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-5-B2C) | Une application web ASP.NET Core qui utilise OpenID Connect pour connecter des utilisateurs dans Azure AD B2C. |
| [dotnetcore-webapp-msal-api](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/4-WebApp-your-API/4-2-B2C) | Une application web ASP.NET Core qui peut connecter un utilisateur à l’aide d’Azure AD B2C, récupérer un jeton d’accès à l’aide de MSAL.NET et appeler une API. |
| [openidconnect-nodejs](https://github.com/AzureADQuickStarts/B2C-WebApp-OpenIDConnect-NodeJS) | Une application Node.js qui offre un moyen simple et rapide pour configurer une application web avec Express à l’aide d’OpenID Connect. |
| [javascript-nodejs-webapi](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi) | Une petite API web node.js pour Azure AD B2C qui montre comment protéger votre API web et accepter les jetons d’accès B2C à l’aide de passport.js. |
| [ms-identity-python-webapp](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/master/README_B2C.md) | Illustre comment intégrer B2C de la Plateforme d’identités Microsoft à une application web Python.  |

## <a name="single-page-apps"></a>Applications à page unique

| Exemple | Description |
|--------| ----------- |
| [ms-identity-b2c-javascript-spa](https://github.com/Azure-Samples/ms-identity-b2c-javascript-spa) | Une application monopage (SPA) appelant une API web. L’authentification est effectuée avec Azure AD B2C en utilisant MSAL.js. Cet exemple utilise le flux de code d’autorisation avec PKCE. |
| [javascript-msal-singlepageapp](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp) | Une application monopage (SPA) appelant une API web. L’authentification est effectuée avec Azure AD B2C en utilisant MSAL.js. Cet exemple utilise le flux implicite.|

## <a name="saml-test-application"></a>Application de test SAML

| Exemple | Description |
|--------| ----------- |
| [saml-sp-tester](https://github.com/azure-ad-b2c/saml-sp-tester/tree/master/source-code) | Application de test SAML pour tester Azure AD B2C, configurée pour servir de fournisseur d’identité SAML. |

## <a name="api-connectors"></a>Connecteurs d’API

Les tableaux suivants fournissent des liens vers des exemples de code pour tirer parti des API web dans vos flux utilisateur avec des [connecteurs d’API](api-connectors-overview.md).

### <a name="azure-function-quickstarts"></a>Démarrages rapides Azure Function

| Exemple                                                                                                                          | Description                                                                                                                                               |
| ------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [.NET Core](https://github.com/Azure-Samples/active-directory-dotnet-external-identities-api-connector-azure-function-validate) | Cet exemple de fonction Azure .NET Core montre comment limiter les connexions à des domaines de messagerie spécifiques et valider les informations fournies par l’utilisateur. |
| [Node.JS](https://github.com/Azure-Samples/active-directory-nodejs-external-identities-api-connector-azure-function-validate)   | Cet exemple de fonction Azure Node.js montre comment limiter les connexions à des domaines de messagerie spécifiques et valider les informations fournies par l’utilisateur.  |
| [Python](https://github.com/Azure-Samples/active-directory-python-external-identities-api-connector-azure-function-validate)    | Cet exemple de fonction Azure Python montre comment limiter les connexions à des domaines de messagerie spécifiques et valider les informations fournies par l’utilisateur.    |


### <a name="automated-fraud-protection-services--captcha"></a>Services automatisés de protection contre les fraudes et CAPTCHA
| Exemple                                                                                                            | Description                                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| [Protection contre les fraudes et mauvaises utilisations Arkose Labs](https://github.com/Azure-Samples/active-directory-b2c-node-sign-up-user-flow-arkose) | Cet exemple montre comment protéger les inscriptions utilisateur à l’aide du service de protection contre les fraudes et mauvaises utilisations Arkose Labs. |
| [reCAPTCHA](https://github.com/Azure-Samples/active-directory-b2c-node-sign-up-user-flow-captcha) | Cet exemple montre comment protéger les inscriptions utilisateur à l’aide d’un test reCAPTCHA pour empêcher les mauvaises utilisations automatisées. |


### <a name="identity-verification"></a>Vérification d’identité

| Exemple                                                                                                            | Description                                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| [IDology](https://github.com/Azure-Samples/active-directory-dotnet-external-identities-idology-identity-verification) | Cet exemple montre comment vérifier une identité d’utilisateur dans le cadre de vos flux d’inscription à l’aide d’un connecteur d’API pour l’intégration à IDology. |
| [Experian](https://github.com/Azure-Samples/active-directory-dotnet-external-identities-experian-identity-verification) | Cet exemple montre comment vérifier une identité d’utilisateur dans le cadre de vos flux d’inscription à l’aide d’un connecteur d’API pour l’intégration à Experian. |


### <a name="other"></a>Autres

| Exemple                                                                                                            | Description                                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| [Code d’invitation](https://github.com/Azure-Samples/active-directory-b2c-node-sign-up-user-flow-invitation-code) | Cet exemple montre comment limiter l’inscription à des publics spécifiques à l’aide de codes d’invitation.|
| [Connecteurs d’API – Exemples de la communauté](https://github.com/azure-ad-b2c/api-connector-samples) | Ce dépôt contient des exemples gérés par la communauté qui portent sur des scénarios activés par des connecteurs d’API.|
