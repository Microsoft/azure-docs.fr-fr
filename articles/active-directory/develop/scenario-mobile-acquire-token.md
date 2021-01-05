---
title: Acquérir un jeton pour appeler une API Web (applications mobiles) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment générer une application mobile qui appelle des API web. (Obtenir un jeton pour l'application).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.reviewer: brandwe
ms.custom: aaddev
ms.openlocfilehash: 85d1880479a514ffb002e4894066734690480457
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94443124"
---
# <a name="get-a-token-for-a-mobile-app-that-calls-web-apis"></a>Obtenir un jeton pour une application mobile qui appelle des API web

Pour pouvoir appeler des API web protégées, votre application doit disposer d'un jeton d'accès. Cet article décrit la procédure à suivre pour obtenir un jeton à l'aide de la bibliothèque MSAL (Microsoft Authentication Library).

## <a name="define-a-scope"></a>Définir une étendue

Lorsque vous demandez un jeton, vous devez définir une étendue. L’étendue détermine les données auxquelles votre application peut accéder.

Pour définir une étendue, l'approche la plus simple consiste à combiner l'`App ID URI` de l'API web souhaitée avec l'étendue `.default`. Cette définition indique à la plateforme d'identités Microsoft que votre application a besoin de toutes les étendues définies sur le portail.

### <a name="android"></a>Android
```Java
String[] SCOPES = {"https://graph.microsoft.com/.default"};
```

### <a name="ios"></a>iOS
```swift
let scopes = ["https://graph.microsoft.com/.default"]
```

### <a name="xamarin"></a>Xamarin
```csharp
var scopes = new [] {"https://graph.microsoft.com/.default"};
```

## <a name="get-tokens"></a>Obtenir des jetons

### <a name="acquire-tokens-via-msal"></a>Acquérir des jetons via MSAL

MSAL permet aux applications d’acquérir des jetons en mode silencieux et de manière interactive. Lorsque vous appelez `AcquireTokenSilent()` ou `AcquireTokenInteractive()`, MSAL renvoie un jeton d'accès pour les étendues demandées. La procédure appropriée consiste à effectuer une demande en mode silencieux, puis à revenir à une demande interactive.

#### <a name="android"></a>Android

```Java
String[] SCOPES = {"https://graph.microsoft.com/.default"};
PublicClientApplication sampleApp = new PublicClientApplication(
                    this.getApplicationContext(),
                    R.raw.auth_config);

// Check if there are any accounts we can sign in silently.
// Result is in the silent callback (success or error).
sampleApp.getAccounts(new PublicClientApplication.AccountsLoadedCallback() {
    @Override
    public void onAccountsLoaded(final List<IAccount> accounts) {

        if (accounts.isEmpty() && accounts.size() == 1) {
            // TODO: Create a silent callback to catch successful or failed request.
            sampleApp.acquireTokenSilentAsync(SCOPES, accounts.get(0), getAuthSilentCallback());
        } else {
            /* No accounts or > 1 account. */
        }
    }
});

[...]

// No accounts found. Interactively request a token.
// TODO: Create an interactive callback to catch successful or failed requests.
sampleApp.acquireToken(getActivity(), SCOPES, getAuthInteractiveCallback());
```

#### <a name="ios"></a>iOS

Essayez d'abord d'acquérir un jeton de manière silencieuse :

```objc

NSArray *scopes = @[@"https://graph.microsoft.com/.default"];
NSString *accountIdentifier = @"my.account.id";

MSALAccount *account = [application accountForIdentifier:accountIdentifier error:nil];

MSALSilentTokenParameters *silentParams = [[MSALSilentTokenParameters alloc] initWithScopes:scopes account:account];
[application acquireTokenSilentWithParameters:silentParams completionBlock:^(MSALResult *result, NSError *error) {

    if (!error)
    {
        // You'll want to get the account identifier to retrieve and reuse the account
        // for later acquireToken calls
        NSString *accountIdentifier = result.account.identifier;

        // Access token to call the web API
        NSString *accessToken = result.accessToken;
    }

    // Check the error
    if (error && [error.domain isEqual:MSALErrorDomain] && error.code == MSALErrorInteractionRequired)
    {
        // Interactive auth will be required, call acquireTokenWithParameters:error:
        return;
    }
}];
```

```swift

let scopes = ["https://graph.microsoft.com/.default"]
let accountIdentifier = "my.account.id"

guard let account = try? application.account(forIdentifier: accountIdentifier) else { return }
let silentParameters = MSALSilentTokenParameters(scopes: scopes, account: account)
application.acquireTokenSilent(with: silentParameters) { (result, error) in

    guard let authResult = result, error == nil else {

    let nsError = error! as NSError

    if (nsError.domain == MSALErrorDomain &&
        nsError.code == MSALError.interactionRequired.rawValue) {

            // Interactive auth will be required, call acquireToken()
            return
         }
         return
     }

    // You'll want to get the account identifier to retrieve and reuse the account
    // for later acquireToken calls
    let accountIdentifier = authResult.account.identifier

    // Access token to call the web API
    let accessToken = authResult.accessToken
}
```

Si MSAL renvoie `MSALErrorInteractionRequired`, essayez d'acquérir des jetons de manière interactive :

```objc
UIViewController *viewController = ...; // Pass a reference to the view controller that should be used when getting a token interactively
MSALWebviewParameters *webParameters = [[MSALWebviewParameters alloc] initWithAuthPresentationViewController:viewController];
MSALInteractiveTokenParameters *interactiveParams = [[MSALInteractiveTokenParameters alloc] initWithScopes:scopes webviewParameters:webParameters];
[application acquireTokenWithParameters:interactiveParams completionBlock:^(MSALResult *result, NSError *error) {
    if (!error)
    {
        // You'll want to get the account identifier to retrieve and reuse the account
        // for later acquireToken calls
        NSString *accountIdentifier = result.account.identifier;

        NSString *accessToken = result.accessToken;
    }
}];
```

```swift
let viewController = ... // Pass a reference to the view controller that should be used when getting a token interactively
let webviewParameters = MSALWebviewParameters(authPresentationViewController: viewController)
let interactiveParameters = MSALInteractiveTokenParameters(scopes: scopes, webviewParameters: webviewParameters)
application.acquireToken(with: interactiveParameters, completionBlock: { (result, error) in

    guard let authResult = result, error == nil else {
        print(error!.localizedDescription)
        return
    }

    // Get access token from result
    let accessToken = authResult.accessToken
})
```

MSAL pour iOS et macOS prend en charge différents modificateurs pour obtenir un jeton de manière interactive ou silencieuse :
* [Paramètres courants pour l'obtention d'un jeton](https://azuread.github.io/microsoft-authentication-library-for-objc/Classes/MSALTokenParameters.html#/Configuration%20parameters)
* [Paramètres permettant d'obtenir un jeton de manière interactive](https://azuread.github.io/microsoft-authentication-library-for-objc/Classes/MSALInteractiveTokenParameters.html#/Configuring%20MSALInteractiveTokenParameters)
* [Paramètres permettant d'obtenir un jeton de manière silencieuse](https://azuread.github.io/microsoft-authentication-library-for-objc/Classes/MSALSilentTokenParameters.html)

#### <a name="xamarin"></a>Xamarin

L'exemple suivant présente le code minimal à utiliser pour obtenir un jeton de manière interactive. L'exemple utilise Microsoft Graph pour lire le profil de l'utilisateur.

```csharp
string[] scopes = new string[] {"user.read"};
var app = PublicClientApplicationBuilder.Create(clientId).Build();
var accounts = await app.GetAccountsAsync();
AuthenticationResult result;
try
{
 result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
             .ExecuteAsync();
}
catch(MsalUiRequiredException)
{
 result = await app.AcquireTokenInteractive(scopes)
             .ExecuteAsync();
}
```

#### <a name="mandatory-parameters-in-msalnet"></a>Paramètres obligatoires dans MSAL.NET

`AcquireTokenInteractive` n'a qu'un seul paramètre obligatoire : `scopes`. Le paramètre `scopes` énumère les chaînes qui définissent les étendues pour lesquelles un jeton est requis. Si le jeton est destiné à Microsoft Graph, vous trouvez les étendues nécessaires dans les informations de référence de chaque API Microsoft Graph. Dans les informations de référence, accédez à la section « Autorisations ».

Par exemple, pour [répertorier les contacts de l'utilisateur](/graph/api/user-list-contacts), utilisez les étendues « User.Read », « Contacts.Read ». Pour plus d’informations, consultez la [documentation de référence sur les autorisations Microsoft Graph](/graph/permissions-reference).

Sur Android, vous pouvez spécifier l'activité parente lors de la création de l'application à l'aide de `PublicClientApplicationBuilder`. Si vous ne spécifiez pas l'activité parente à ce moment-là, vous pouvez la spécifier ultérieurement à l'aide de `.WithParentActivityOrWindow`, comme dans la section suivante. Si vous spécifiez l'activité parente, le jeton y revient après l'interaction. Si vous ne la spécifiez pas, l'appel `.ExecuteAsync()` renvoie une exception.

#### <a name="specific-optional-parameters-in-msalnet"></a>Paramètres facultatifs particuliers dans MSAL.NET

Les sections suivantes décrivent les paramètres facultatifs de MSAL.NET.

##### <a name="withprompt"></a>WithPrompt

Le paramètre `WithPrompt()` contrôle l'interactivité avec l'utilisateur en spécifiant une invite.

![Image montrant les champs de la structure de l’invite. Ces valeurs de constantes contrôlent l’interactivité avec l’utilisateur en définissant le type d’invite affiché par le paramètre WithPrompt().](https://user-images.githubusercontent.com/13203188/53438042-3fb85700-39ff-11e9-9a9e-1ff9874197b3.png)

La classe définit les constantes suivantes :

- `SelectAccount` force le service d'émission de jeton de sécurité (STS) à présenter la boîte de dialogue de sélection de compte. La boîte de dialogue contient les comptes pour lesquels l'utilisateur dispose d'une session. Vous pouvez utiliser cette option pour permettre à l'utilisateur de choisir parmi différentes identités. Cette option oblige MSAL à envoyer `prompt=select_account` au fournisseur d’identité.

    La constante `SelectAccount` est le paramètre par défaut. Elle fournit la meilleure expérience possible sur la base des informations disponibles. Les informations disponibles peuvent inclure le compte, la présence d'une session pour l'utilisateur, etc. Ne remplacez pas ce paramètre par défaut, sauf si vous avez une bonne raison de le faire.
- `Consent` vous permet de demander à l'utilisateur de donner son consentement, même si celui-ci a déjà été accordé. Dans ce cas, MSAL envoie `prompt=consent` au fournisseur d’identité.

    Vous souhaiterez peut-être utiliser la constante `Consent` dans les applications axées sur la sécurité pour lesquelles la gouvernance de l'organisation exige que les utilisateurs voient la boîte de dialogue de consentement chaque fois qu'ils utilisent l'application.
- `ForceLogin` permet au service d'inviter l'utilisateur à entrer ses informations d'identification même si l'invite n'est pas nécessaire.

    Cette option peut être utile si l'acquisition du jeton échoue et que vous souhaitez permettre à l'utilisateur de se reconnecter. Dans ce cas, MSAL envoie `prompt=login` au fournisseur d’identité. Vous souhaiterez peut-être utiliser cette option dans les applications axées sur la sécurité pour lesquelles la gouvernance de l'organisation exige que l'utilisateur se connecte chaque fois qu'il accède à des parties spécifiques de l'application.
- `Never` concerne uniquement .NET 4.5 et Windows Runtime (WinRT). Cette constante n'affichera aucune invite utilisateur, mais elle tentera d'utiliser le cookie stocké dans l'affichage web incorporé masqué. Pour plus d'informations, consultez [Utilisation de navigateurs avec MSAL.NET](./msal-net-web-browsers.md).

    Si cette option échoue, `AcquireTokenInteractive` renvoie une exception pour vous informer qu'une interaction avec l'interface utilisateur est nécessaire. Vous devez alors utiliser un autre paramètre `Prompt`.
- `NoPrompt` n'envoie pas d'invite au fournisseur d'identité.

    Cette option n'est utile que pour les stratégies de modification de profil d'Azure Active Directory B2C. Pour plus d'informations, consultez [Spécificités de B2C](https://aka.ms/msal-net-b2c-specificities).

##### <a name="withextrascopetoconsent"></a>WithExtraScopeToConsent

Utilisez le modificateur `WithExtraScopeToConsent` dans un scénario avancé où vous souhaitez que l'utilisateur fournisse son consentement préalable à plusieurs ressources. Vous pouvez utiliser ce modificateur lorsque vous ne souhaitez pas avoir recours au consentement incrémentiel, qui est normalement utilisé avec MSAL.NET ou la Plateforme d'identités Microsoft 2.0. Pour plus d’informations, consultez [Guide pratique pour obtenir le consentement préalable de l’utilisateur sur plusieurs ressources](scenario-desktop-production.md#have-the-user-consent-upfront-for-several-resources).

Exemple de code :

```csharp
var result = await app.AcquireTokenInteractive(scopesForCustomerApi)
                     .WithExtraScopeToConsent(scopesForVendorApi)
                     .ExecuteAsync();
```

##### <a name="other-optional-parameters"></a>Autres paramètres facultatifs

Pour en savoir plus sur les autres paramètres facultatifs de `AcquireTokenInteractive`, consultez la [documentation de référence relative à AcquireTokenInteractiveParameterBuilder](/dotnet/api/microsoft.identity.client.acquiretokeninteractiveparameterbuilder?view=azure-dotnet-preview#methods).

### <a name="acquire-tokens-via-the-protocol"></a>Acquérir des jetons via le protocole

Nous vous déconseillons d'utiliser directement le protocole pour obtenir des jetons. Si vous le faites, l'application ne prendra pas en charge certains scénarios impliquant l'authentification unique (SSO), la gestion des périphériques et l'accès conditionnel.

Lorsque vous utilisez le protocole pour obtenir des jetons destinés aux applications mobiles, effectuez deux demandes :

* Obtention d'un code d'autorisation
* Échange du code contre un jeton

#### <a name="get-an-authorization-code"></a>Obtention d’un code d’autorisation

```
https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=<CLIENT_ID>
&response_type=code
&redirect_uri=<ENCODED_REDIRECT_URI>
&response_mode=query
&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2F.default
&state=12345
```

#### <a name="get-access-and-refresh-the-token"></a>Obtention de l'accès et actualisation du jeton

```HTTP
POST /{tenant}/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=<CLIENT_ID>
&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&code=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq3n8b2JRLk4OxVXr...
&redirect_uri=<ENCODED_REDIRECT_URI>
&grant_type=authorization_code
```

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Appeler une API web](scenario-mobile-call-api.md).