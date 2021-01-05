---
title: Prise en charge d’AD FS dans MSAL.NET | Azure
titleSuffix: Microsoft identity platform
description: Découvrez la prise en charge des services de fédération Active Directory (AD FS) dans la bibliothèque d’authentification Microsoft pour .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/16/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 4107b0785ef4635e0e5e745a35b605d2634a220d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88166226"
---
# <a name="active-directory-federation-services-support-in-msalnet"></a>Prendre en charge les services de fédération Active Directory (AD FS) dans MSAL.NET
Les services de fédération Active Directory (AD FS) de Windows Server vous permettent d'ajouter l'authentification et l'autorisation OpenID Connect et OAuth 2.0 aux applications que vous développez. Ces applications peuvent ensuite authentifier les utilisateurs directement auprès d'AD FS. Pour plus d’informations, consultez [Scénarios AD FS pour développeurs](/windows-server/identity/ad-fs/overview/ad-fs-openid-connect-oauth-flows-scenarios).

La bibliothèque d’authentification Microsoft pour .NET (MSAL.NET) prend en charge deux scénarios pour l’authentification auprès d’AD FS :

- MSAL.NET communique avec le service Azure Active Directory, qui lui-même est *fédéré* à AD FS.
- MSAL.NET communique **directement** avec une autorité ADFS. Cette fonctionnalité est uniquement prise en charge par AD FS 2019 et versions ultérieures. L'un des scénarios mis en évidence est la prise en charge d'[Azure Stack](https://azure.microsoft.com/overview/azure-stack/)


## <a name="msal-connects-to-azure-ad-which-is-federated-with-ad-fs"></a>MSAL se connecte à Azure AD, qui est fédéré à AD FS
MSAL.NET prend en charge la connexion à Azure AD, qui connecte les utilisateurs managés (utilisateurs managés dans Azure AD) ou fédérés (utilisateurs managés par un autre fournisseur d’identité, tel qu’AD FS). MSAL.NET ignore le fait que les utilisateurs sont fédérés. En ce qui le concerne, il communique avec Azure AD.

L’[autorité](msal-client-application-configuration.md#authority) que vous utilisez dans ce cas est l’autorité habituelle (nom d’hôte d’autorité + locataire, commun ou organisations).

### <a name="acquiring-a-token-interactively"></a>Acquisition d’un jeton de manière interactive
Lorsque vous appelez la méthode `AcquireTokenInteractive`, l’expérience utilisateur est généralement la suivante :

1. L’utilisateur entre son ID de compte.
2. Azure AD affiche brièvement le message « Vous êtes redirigé vers la page de votre organisation ».
3. L’utilisateur est redirigé vers la page de connexion du fournisseur d’identité. La page de connexion est en règle générale personnalisée avec le logo de l’organisation.

Les versions AD FS prises en charge dans ce scénario fédéré sont AD FS v2, AD FS v3 (Windows Server 2012 R2) et AD FS v4 (AD FS 2016).

### <a name="acquiring-a-token-using-acquiretokenbyintegratedauthentication-or-acquiretokenbyusernamepassword"></a>Acquisition d’un jeton à l’aide de AcquireTokenByIntegratedAuthentication ou de AcquireTokenByUsernamePassword
Lors de l’acquisition d’un jeton à l’aide des méthodes `AcquireTokenByIntegratedAuthentication` ou `AcquireTokenByUsernamePassword`, MSAL.NET obtient le fournisseur d’identité à contacter en fonction du nom d’utilisateur.  MSAL.NET reçoit un [jeton SAML 1.1](reference-saml-tokens.md) après avoir contacté le fournisseur d’identité.  MSAL.NET fournit ensuite le jeton SAML à Azure AD, sous la forme d’une assertion d’utilisateur (similaire au [flux on-behalf-of](msal-authentication-flows.md#on-behalf-of)) pour récupérer un jeton JWT.

## <a name="msal-connects-directly-to-ad-fs"></a>MSAL se connecte directement à AD FS
MSAL.NET prend en charge la connexion à AD FS 2019, qui est conforme à Open ID Connect et comprend PKCE et les étendues. Cette prise en charge n'est possible que si un Service Pack [KB 4490481](https://support.microsoft.com/en-us/help/4490481/windows-10-update-kb4490481) est appliqué à Windows Server. Lorsque vous vous connectez directement à AD FS, l’autorité que vous voulez utiliser pour générer votre application est similaire à `https://mysite.contoso.com/adfs/`.

Pour le moment, il n'est pas prévu de prendre en charge une connexion directe à :

- AD FS 16, car il ne prend pas en charge PKCE et utilise encore les ressources, et non l'étendue.
- AD FS v2, qui n'est pas conforme à OIDC.

 Si vous avez besoin de prendre en charge des scénarios qui exigent une connexion directe à AD FS 2016, utilisez la dernière version en date de la [Bibliothèque d'authentification Azure Active Directory](../azuread-dev/active-directory-authentication-libraries.md#microsoft-supported-client-libraries). Lorsque vous aurez mis à niveau votre système local vers AD FS 2019, vous serez en mesure d’utiliser MSAL.NET.

## <a name="next-steps"></a>Étapes suivantes

Pour l'authentification fédérée, consultez [Configurer le comportement de la connexion Azure Active Directory pour une application à l'aide d'une stratégie de découverte du domaine d'accueil](../manage-apps/configure-authentication-for-federated-users-portal.md).