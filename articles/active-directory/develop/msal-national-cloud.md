---
title: Utiliser MSAL dans une application cloud nationale | Azure
titleSuffix: Microsoft identity platform
description: Microsoft Authentication Library (MSAL) permet aux développeurs d’applications d’acquérir des jetons afin d’appeler des API web sécurisées. Ces API web peuvent être Microsoft Graph, d’autres API Microsoft, des API web de partenaires ou vos propres API web. MSAL prend en charge plusieurs architectures et plateformes d’application.
services: active-directory
author: negoe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/22/2019
ms.author: negoe
ms.reviewer: marsma, nacanuma
ms.custom: aaddev
ms.openlocfilehash: 2f631ad650b20110084ff902043447f26c2d4ab3
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96453602"
---
# <a name="use-msal-in-a-national-cloud-environment"></a>Utiliser MSAL dans un environnement de cloud national

Les [clouds nationaux](authentication-national-cloud.md), également appelés clouds souverains, sont des instances d’Azure physiquement isolées. Ces régions d’Azure aident à garantir que les conditions de résidence, de souveraineté et de conformité des données sont respectées au sein de limites géographiques.

En plus du cloud Microsoft global, Microsoft Authentication Library (MSAL) permet aux développeurs d’applications dans des clouds nationaux d’acquérir des jetons pour authentifier et appeler des API web sécurisées. Ces API web peuvent être Microsoft Graph ou d’autres API Microsoft.

Outre le cloud global, Azure Active Directory (Azure AD) est déployé dans les clouds nationaux suivants :  

- Azure Government
- Azure China 21Vianet
- Azure Germany

Ce guide montre comment se connecter à des comptes professionnels et scolaires, obtenir un jeton accès et appeler l’API Microsoft Graph dans l’environnement [cloud Azure Government](https://azure.microsoft.com/global-infrastructure/government/).

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous répondez aux exigences suivantes.

### <a name="choose-the-appropriate-identities"></a>Choisir les identités appropriées

Les applications [Azure Government](../../azure-government/index.yml) peuvent utiliser des identités Azure AD Government et des identités Azure AD Public pour authentifier des utilisateurs. Étant donné que vous pouvez utiliser l’une de ces identités, vous devez déterminer quel point de terminaison d’autorité à choisir pour votre scénario :

- Azure AD Public : généralement utilisé si votre organisation possède déjà un locataire Azure AD Public pour prendre en charge Microsoft 365 (Public ou GCC) ou une autre application.
- Azure AD Government : généralement utilisé si votre organisation dispose d’un locataire Azure AD Government pour prendre en charge Office 365 (GCC High ou DoD) ou si elle crée un nouveau locataire dans Azure Government AD.

Après avoir choisi, une attention particulière est portée là où vous effectuez votre inscription d’application. Si vous choisissez des identités Azure AD Public pour votre application Azure Government, vous devez inscrire l’application dans votre locataire Azure AD Public.

### <a name="get-an-azure-government-subscription"></a>Obtenir un abonnement Azure Government

Pour obtenir un abonnement Azure Government, consultez [Gestion et connexion à votre abonnement dans Azure Government](../../azure-government/compare-azure-government-global-azure.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/global-infrastructure/government/request/) avant de commencer.

Pour plus d’informations sur l’utilisation d’un cloud national avec un langage de programmation particulier, choisissez l’onglet correspondant à votre langage :

## <a name="net"></a>[.NET](#tab/dotnet)

Vous pouvez utiliser MSAL.NET pour connecter des utilisateurs, acquérir des jetons et appeler l’API Microsoft Graph dans des clouds nationaux.

Les didacticiels suivants montrent comment créer une application Web MVC .NET Core 2.2. L’application utilise OpenID Connect pour connecter des utilisateurs avec un compte professionnel et scolaire dans une organisation appartenant à un cloud national.

- Pour connecter des utilisateurs et acquérir des jetons, suivez ce didacticiel : [Créez une application web de base ASP.NET en connectant les utilisateurs dans des clouds souverains avec la plateforme d’identité Microsoft](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-4-Sovereign#build-an-aspnet-core-web-app-signing-in-users-in-sovereign-clouds-with-the-microsoft-identity-platform).
- Pour appeler l’API Microsoft Graph, suivez ce didacticiel : [Utilisation de la plateforme Microsoft Identity pour appeler l’API Microsoft Graph à partir d’une application web ASP.NET Core 2.x, pour le compte d’un utilisateur qui se connecte à l’aide de son compte professionnel et scolaire dans le cloud national Microsoft](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-4-Sovereign-Call-MSGraph#using-the-microsoft-identity-platform-to-call-the-microsoft-graph-api-from-an-an-aspnet-core-2x-web-app-on-behalf-of-a-user-signing-in-using-their-work-and-school-account-in-microsoft-national-cloud).

## <a name="javascript"></a>[JavaScript](#tab/javascript)

Pour activer votre application MSAL.js pour les clouds souverains :

### <a name="step-1-register-your-application"></a>Étape 1 : Inscrivez votre application

1. Connectez-vous au [portail Azure](https://portal.azure.us/).

   Pour rechercher des points de terminaison de portail Azure pour d’autres clouds nationaux, consultez [Points de terminaison d’inscription d’application](authentication-national-cloud.md#app-registration-endpoints).

1. Si votre compte vous propose un accès à plusieurs locataires, sélectionnez votre compte en haut à droite et définissez votre session de portail sur le locataire Azure AD souhaité.
1. Accédez à la page [Inscriptions d’applications](https://aka.ms/ra/ff) de la plateforme d’identité Microsoft pour les développeurs.
1. Lorsque la page **Inscrire une application** s’affiche, entrez le nom de votre application.
1. Sous **Types de comptes pris en charge**, sélectionnez **Comptes dans un annuaire organisationnel**.
1. Sous la section **URI de redirection**, sélectionnez la plateforme **Web** et définissez la valeur sur l’URL de l’application basée sur votre serveur web. Consultez les prochaines sections pour obtenir des instructions permettant de définir et d’obtenir l’URL de redirection dans Visual Studio et Node.
1. Sélectionnez **Inscription**.
1. Sur la page **Vue d’ensemble**, notez la valeur **ID d’application (client)** .
1. Ce didacticiel vous demande d’activer le [flux d’octroi implicite](v2-oauth2-implicit-grant-flow.md). Dans le volet gauche de l’application inscrite, sélectionnez **Authentification**.
1. Dans **Paramètres avancés**, sous **Octroi implicite**, cochez les cases **Jetons d’ID** et **Jetons d’accès**. Les jetons d’ID et les jetons d’accès sont nécessaires, car cette application doit connecter des utilisateurs et appeler une API.
1. Sélectionnez **Enregistrer**.

### <a name="step-2--set-up-your-web-server-or-project"></a>Étape 2 :  Configurer le serveur web ou projet

- [Télécharger les fichiers de projet](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/quickstart.zip) pour un serveur web local, tel que Node.

  or

- [Télécharger le projet Visual Studio](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/vsquickstart.zip).

Passez ensuite à [Configurer votre application SPA JavaScript](#step-4-configure-your-javascript-spa) pour configurer l’exemple de code avant de l’exécuter.

### <a name="step-3-use-the-microsoft-authentication-library-to-sign-in-the-user"></a>Étape 3 : Utiliser Microsoft Authentication Library pour connecter l’utilisateur

Suivez les étapes du [didacticiel JavaScript](tutorial-v2-javascript-spa.md#create-your-project) pour créer votre projet et l’intégrer avec MSAL pour connecter l’utilisateur.

### <a name="step-4-configure-your-javascript-spa"></a>Étape 4 : Configurer une application SPA JavaScript

Dans le fichier `index.html` créé au cours de la configuration du projet, ajoutez les informations d’inscription d’application. Ajoutez le code ci-après dans la partie supérieure entre les balises `<script></script>` dans le corps de votre fichier `index.html` :

```javascript
const msalConfig = {
    auth:{
        clientId: "Enter_the_Application_Id_here",
        authority: "https://login.microsoftonline.us/Enter_the_Tenant_Info_Here",
        }
}

const graphConfig = {
        graphEndpoint: "https://graph.microsoft.us",
        graphScopes: ["user.read"],
}

// create UserAgentApplication instance
const myMSALObj = new UserAgentApplication(msalConfig);
```

Dans ce code :

- `Enter_the_Application_Id_here` est la valeur **ID d’application (client)** pour l’application que vous avez inscrite.
- `Enter_the_Tenant_Info_Here` est défini sur l’une des options suivantes :
    - Si votre application prend en charge les **Comptes dans cet annuaire organisationnel**, remplacez cette valeur par l’ID de locataire ou le nom du locataire (par exemple, contoso.microsoft.com).
    - Si votre application prend en charge **Comptes dans un annuaire organisationnel**, remplacez cette valeur par `organizations`.

    Pour rechercher des points de terminaison d’authentification pour tous les clouds nationaux, consultez [Points de terminaison d’authentification Azure AD](./authentication-national-cloud.md#azure-ad-authentication-endpoints).

    > [!NOTE]
    > Les comptes Microsoft personnels ne sont pas pris en charge dans les clouds nationaux.

- `graphEndpoint` est le point de terminaison Microsoft Graph pour Microsoft Cloud for US Government.

   Pour rechercher des points de terminaison Microsoft Graph pour tous les clouds nationaux, consultez [Points de terminaison Microsoft Graph dans des clouds nationaux](/graph/deployments#microsoft-graph-and-graph-explorer-service-root-endpoints).

## <a name="python"></a>[Python](#tab/python)

Pour activer votre application MSAL Python pour les clouds souverains :

- Inscrivez votre application dans un portail spécifique en fonction du cloud. Pour plus d’informations sur la façon de choisir le portail, reportez-vous à [Points de terminaison d’inscription d’application](authentication-national-cloud.md#app-registration-endpoints)
- Utilisez n’importe lequel des [exemple](https://github.com/AzureAD/microsoft-authentication-library-for-python/tree/dev/sample) du référentiel en apportant quelques modifications à la configuration, en fonction du cloud, comme mentionné ci-dessous.
- Utilisez une autorité spécifique, en fonction du cloud dans lequel vous avez inscrit l’application. Pour plus d’informations sur les autorités pour les différents clouds, consultez [Points de terminaison Azure AD Authentication](authentication-national-cloud.md#azure-ad-authentication-endpoints).

    Voici un exemple d’autorité :

    ```json
    "authority": "https://login.microsoftonline.us/Enter_the_Tenant_Info_Here"
    ```

- L’appel de l’API Microsoft Graph nécessite une URL de point de terminaison spécifique au cloud que vous utilisez. Pour rechercher des points de terminaison Microsoft Graph pour tous les clouds nationaux, consultez [Points de terminaison racine de service de jetons et d’inscription de l’application](/graph/deployments#microsoft-graph-and-graph-explorer-service-root-endpoints).

    Voici un exemple de point de terminaison Microsoft Graph, avec l’étendue :

    ```json
    "endpoint" : "https://graph.microsoft.us/v1.0/me"
    "scope": "User.Read"
    ```

## <a name="java"></a>[Java](#tab/java)

Pour activer votre application MSAL pour Java pour les clouds souverains :

- Inscrivez votre application dans un portail spécifique en fonction du cloud. Pour plus d’informations sur la façon de choisir le portail, reportez-vous à [Points de terminaison d’inscription d’application](authentication-national-cloud.md#app-registration-endpoints)
- Utilisez n’importe lequel des [exemples](https://github.com/AzureAD/microsoft-authentication-library-for-java/tree/dev/src/samples) du référentiel en apportant quelques modifications à la configuration, en fonction du cloud, comme mentionné ci-dessous.
- Utilisez une autorité spécifique, en fonction du cloud dans lequel vous avez inscrit l’application. Pour plus d’informations sur les autorités pour les différents clouds, consultez [Points de terminaison Azure AD Authentication](authentication-national-cloud.md#azure-ad-authentication-endpoints).

Voici un exemple d’autorité :

```json
"authority": "https://login.microsoftonline.us/Enter_the_Tenant_Info_Here"
```

- L’appel de l’API Microsoft Graph nécessite une URL de point de terminaison spécifique au cloud que vous utilisez. Pour rechercher des points de terminaison Microsoft Graph pour tous les clouds nationaux, consultez [Points de terminaison racine de service de jetons et d’inscription de l’application](/graph/deployments#microsoft-graph-and-graph-explorer-service-root-endpoints).

Voici un exemple de point de terminaison Graph, avec l’étendue :

```json
"endpoint" : "https://graph.microsoft.us/v1.0/me"
"scope": "User.Read"
```

## <a name="objective-c"></a>[Objective-C](#tab/objc)

MSAL pour iOS et macOS peut être utilisé pour acquérir des jetons dans les clouds ​​nationaux, mais une configuration supplémentaire est nécessaire lors de la création de `MSALPublicClientApplication`.

Par exemple, si vous souhaitez que votre application soit mutualisée dans un cloud national (US Government, en l’occurrence), vous pouvez écrire :

```objc
MSALAADAuthority *aadAuthority =
                [[MSALAADAuthority alloc] initWithCloudInstance:MSALAzureUsGovernmentCloudInstance
                                                   audienceType:MSALAzureADMultipleOrgsAudience
                                                      rawTenant:nil
                                                          error:nil];

MSALPublicClientApplicationConfig *config =
                [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<your-client-id-here>"
                                                                redirectUri:@"<your-redirect-uri-here>"
                                                                  authority:aadAuthority];

NSError *applicationError = nil;
MSALPublicClientApplication *application =
                [[MSALPublicClientApplication alloc] initWithConfiguration:config error:&applicationError];
```

## <a name="swift"></a>[Swift](#tab/swift)

MSAL pour iOS et macOS peut être utilisé pour acquérir des jetons dans les clouds ​​nationaux, mais une configuration supplémentaire est nécessaire lors de la création de `MSALPublicClientApplication`.

Par exemple, si vous souhaitez que votre application soit mutualisée dans un cloud national (US Government, en l’occurrence), vous pouvez écrire :

```swift
let authority = try? MSALAADAuthority(cloudInstance: .usGovernmentCloudInstance, audienceType: .azureADMultipleOrgsAudience, rawTenant: nil)

let config = MSALPublicClientApplicationConfig(clientId: "<your-client-id-here>", redirectUri: "<your-redirect-uri-here>", authority: authority)
if let application = try? MSALPublicClientApplication(configuration: config) { /* Use application */}
```

---

## <a name="next-steps"></a>Étapes suivantes

Consultez [Points de terminaison d’authentificaiton cloud](authentication-national-cloud.md) pour obtenir une liste des URL du portail Azure et des points de terminaison de jeton de chaque cloud.

Documentation sur le cloud national :

- [Azure Government](../../azure-government/index.yml)
- [Azure China 21Vianet](/azure/china/)
- [Azure Allemagne](../../germany/index.yml)