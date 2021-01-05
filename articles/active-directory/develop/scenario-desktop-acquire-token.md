---
title: Acquérir un jeton pour appeler une API web (application de bureau) | Azure
titleSuffix: Microsoft identity platform
description: Découvrir comment générer une application de bureau qui appelle des API web en vue d’acquérir un jeton pour l’application
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/04/2020
ms.author: jmprieur
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: fd341a4f6e2402ce934bdffd4f024e0ef569eec1
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96340915"
---
# <a name="desktop-app-that-calls-web-apis-acquire-a-token"></a>Application de bureau qui appelle des API web : Acquérir un jeton

Après avoir créé une instance de l’application cliente publique, vous l’utilisez pour acquérir un jeton que vous utiliserez ensuite pour appeler une API web.

## <a name="recommended-pattern"></a>Modèle recommandé

L’API web est définie par ses valeurs de `scopes`. Quelle que soit l’expérience que vous fournissez dans votre application, le modèle à utiliser est le suivant :

- Systématiquement tenter d’obtenir un jeton à partir du cache de jeton en appelant `AcquireTokenSilent`
- Si cet appel échoue, utiliser le flux `AcquireToken` que vous souhaitez utiliser, ici représenté par `AcquireTokenXX`

# <a name="net"></a>[.NET](#tab/dotnet)

### <a name="in-msalnet"></a>Dans MSAL.NET

```csharp
AuthenticationResult result;
var accounts = await app.GetAccountsAsync();
IAccount account = ChooseAccount(accounts); // for instance accounts.FirstOrDefault
                                            // if the app manages is at most one account
try
{
 result = await app.AcquireTokenSilent(scopes, account)
                   .ExecuteAsync();
}
catch(MsalUiRequiredException ex)
{
  result = await app.AcquireTokenXX(scopes, account)
                    .WithOptionalParameterXXX(parameter)
                    .ExecuteAsync();
}
```

# <a name="java"></a>[Java](#tab/java)

```java

Set<IAccount> accountsInCache = pca.getAccounts().join();
// Take first account in the cache. In a production application, you would filter
// accountsInCache to get the right account for the user authenticating.
IAccount account = accountsInCache.iterator().next();

IAuthenticationResult result;
try {
    SilentParameters silentParameters =
            SilentParameters
                    .builder(SCOPE, account)
                    .build();

    // try to acquire token silently. This call will fail since the token cache
    // does not have any data for the user you are trying to acquire a token for
    result = pca.acquireTokenSilently(silentParameters).join();
} catch (Exception ex) {
    if (ex.getCause() instanceof MsalException) {

        InteractiveRequestParameters parameters = InteractiveRequestParameters
                .builder(new URI("http://localhost"))
                .scopes(SCOPE)
                .build();

        // Try to acquire a token interactively with system browser. If successful, you should see
        // the token and account information printed out to console
        result = pca.acquireToken(parameters).join();
    } else {
        // Handle other exceptions accordingly
        throw ex;
    }
}
return result;

```

# <a name="python"></a>[Python](#tab/python)

```Python
result = None

# Firstly, check the cache to see if this end user has signed in before
accounts = app.get_accounts(username=config["username"])
if accounts:
    result = app.acquire_token_silent(config["scope"], account=accounts[0])

if not result:
    result = app.acquire_token_by_xxx(scopes=config["scope"])
```

# <a name="macos"></a>[macOS](#tab/macOS)

### <a name="in-msal-for-ios-and-macos"></a>Dans MSAL pour iOS et macOS

Objective-C :

```objc
MSALAccount *account = [application accountForIdentifier:accountIdentifier error:nil];

MSALSilentTokenParameters *silentParams = [[MSALSilentTokenParameters alloc] initWithScopes:scopes account:account];
[application acquireTokenSilentWithParameters:silentParams completionBlock:^(MSALResult *result, NSError *error) {

    // Check the error
    if (error && [error.domain isEqual:MSALErrorDomain] && error.code == MSALErrorInteractionRequired)
    {
        // Interactive auth will be required, call acquireTokenWithParameters:error:
    }
}];
```
Swift :

```swift
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
}
```
---

Voici les différentes étapes permettant d’obtenir des jetons dans une application de bureau.

## <a name="acquire-a-token-interactively"></a>Acquérir un jeton de manière interactive

L’exemple suivant montre le code minimal permettant d’obtenir un jeton de manière interactive pour lire le profil de l’utilisateur avec Microsoft Graph.

# <a name="net"></a>[.NET](#tab/dotnet)
### <a name="in-msalnet"></a>Dans MSAL.NET

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

### <a name="mandatory-parameters"></a>Paramètres obligatoires

`AcquireTokenInteractive` n’a qu’un seul paramètre obligatoire, ``scopes``, qui contient une énumération de chaînes définissant les étendues pour lesquelles un jeton est nécessaire. Si le jeton est destiné à Microsoft Graph, vous trouvez les étendues nécessaires dans les informations de référence d’API de chaque API Microsoft Graph, à la section intitulée « Autorisations ». Par exemple, pour [lister les contacts de l’utilisateur](/graph/api/user-list-contacts), vous devez utiliser les étendues « User.Read », « Contacts.Read ». Pour plus d’informations, consultez la [documentation de référence sur les autorisations Microsoft Graph](/graph/permissions-reference).

Sur Android, vous devez également spécifier l’activité parente au moyen de `.WithParentActivityOrWindow`, comme indiqué, afin que le jeton retourne à cette activité parente après l’interaction. Si vous ne le précisez pas, une exception est levée lors de l’appel de `.ExecuteAsync()`.

### <a name="specific-optional-parameters-in-msalnet"></a>Paramètres facultatifs particuliers dans MSAL.NET

#### <a name="withparentactivityorwindow"></a>WithParentActivityOrWindow

L’interface utilisateur est importante, car elle est interactive. `AcquireTokenInteractive` présente un paramètre facultatif spécifique pouvant préciser, pour les plateformes qui le prennent en charge, l’interface utilisateur parente. Lorsqu’il est utilisé dans une application de bureau, `.WithParentActivityOrWindow` présente un type différent qui dépend la plateforme. Vous pouvez également omettre le paramètre facultatif de fenêtre parente pour créer une fenêtre, si vous ne souhaitez pas contrôler l’emplacement où la boîte de dialogue de connexion s’affiche à l’écran. Cela s’applique aux applications basées sur une ligne de commande, utilisées pour passer des appels à tout autre service principal et qui n’ont pas besoin de fenêtres pour l’interaction avec l’utilisateur.

```csharp
// net45
WithParentActivityOrWindow(IntPtr windowPtr)
WithParentActivityOrWindow(IWin32Window window)

// Mac
WithParentActivityOrWindow(NSWindow window)

// .Net Standard (this will be on all platforms at runtime, but only on NetStandard at build time)
WithParentActivityOrWindow(object parent).
```

Remarques :

- Sur .NET Standard, l’élément `object` attendu est `Activity` sur Android, `UIViewController` sur iOS, `NSWindow` sur MAC, et `IWin32Window` ou `IntPr` sur Windows.
- Sur Windows, vous devez appeler `AcquireTokenInteractive` à partir du thread d’interface utilisateur, afin que le navigateur intégré obtienne le contexte de synchronisation de l’interface utilisateur approprié. Ne pas appeler depuis le thread d’interface utilisateur peut occasionner des messages qui ne pompent pas correctement, et des scénarios de blocage avec l’interface utilisateur. Un moyen d’appeler les bibliothèques d’authentification Microsoft (MSAL) à partir du thread d’interface utilisateur, si vous n’êtes pas déjà sur le thread, consiste à utiliser `Dispatcher` sur WPF.
- Si vous utilisez WPF, vous pouvez vous servir de la classe `WindowInteropHelper.Handle` pour obtenir une fenêtre à partir d’un contrôle WPF. L’appel est alors le suivant, à partir d’un contrôle WPF (`this`) :

  ```csharp
  result = await app.AcquireTokenInteractive(scopes)
                    .WithParentActivityOrWindow(new WindowInteropHelper(this).Handle)
                    .ExecuteAsync();
  ```

#### <a name="withprompt"></a>WithPrompt

`WithPrompt()` permet de contrôler l’interactivité avec l’utilisateur en spécifiant une invite.

![Image montrant les champs de la structure de l’invite. Ces valeurs de constantes contrôlent l’interactivité avec l’utilisateur en définissant le type d’invite affiché par la méthode WithPrompt().](https://user-images.githubusercontent.com/13203188/53438042-3fb85700-39ff-11e9-9a9e-1ff9874197b3.png)

La classe définit les constantes suivantes :

- ``SelectAccount`` force le service d’émission de jeton de sécurité (STS) à présenter la boîte de dialogue de sélection de compte qui contient les comptes pour lesquels l’utilisateur dispose d’une session. Cette option est utile lorsque les développeurs d’applications veulent laisser les utilisateurs choisir parmi différentes identités. Cette option oblige MSAL à envoyer ``prompt=select_account`` au fournisseur d’identité. Cette option est celle par défaut. Elle remplit bien sa tâche en fournissant la meilleure expérience possible en fonction des informations disponibles, comme le compte et la présence d’une session pour l’utilisateur. Ne la remplacez pas, à moins d’avoir une excellente raison de le faire.
- ``Consent`` permet au développeur d’application de forcer l’affichage d’une invite demandant à l’utilisateur son consentement, et même si celui-ci a été accordé auparavant. Dans ce cas, MSAL envoie `prompt=consent` au fournisseur d’identité. Cette option peut être utilisée dans certaines applications axées sur la sécurité, dans lesquelles la gouvernance de l’organisation exige que l’utilisateur voie s’afficher la boîte de dialogue de consentement chaque fois que l’application est utilisée.
- ``ForceLogin`` permet au développeur d’application de faire afficher par le service une invite demandant à l’utilisateur d’entrer des informations d’identification, et même si cette invite utilisateur n’est peut-être pas nécessaire. Cette option peut s’avérer utile pour permettre à l’utilisateur de se reconnecter si l’acquisition d’un jeton échoue. Dans ce cas, MSAL envoie `prompt=login` au fournisseur d’identité. Elle est parfois utilisée dans les applications axées sur la sécurité, pour lesquelles la gouvernance de l’organisation exige que l’utilisateur se reconnecte chaque fois qu’il accède à des parties spécifiques d’une application.
- ``Never`` (pour .NET 4.5 et WinRT uniquement) n’affiche aucune invite utilisateur, mais tente à la place d’utiliser le cookie stocké dans la vue web incorporée masquée. Pour plus d’informations, consultez les vues web dans MSAL.NET. L’utilisation de cette option peut échouer. Dans ce cas, `AcquireTokenInteractive` lève une exception pour notifier qu’une interaction avec l’interface utilisateur est nécessaire. Vous devez utiliser un autre paramètre `Prompt`.
- ``NoPrompt`` n’envoie pas d’invite au fournisseur d’identité. Cette option n’est utile que pour les stratégies de modification de profil Azure Active Directory (Azure AD) B2C. Pour plus d’informations, consultez [Spécificités d’Azure Active Directory B2C](https://aka.ms/msal-net-b2c-specificities).

#### <a name="withextrascopetoconsent"></a>WithExtraScopeToConsent

Ce modificateur est utilisé dans un scénario élaboré, suivant lequel vous souhaitez obtenir le consentement préalable de l’utilisateur pour plusieurs ressources en amont, et ne souhaitez pas utiliser le consentement incrémentiel, qui est normalement utilisé avec MSAL.NET/la plateforme d’identités Microsoft. Pour plus d’informations, consultez [Guide pratique pour obtenir le consentement préalable de l’utilisateur sur plusieurs ressources](scenario-desktop-production.md#have-the-user-consent-upfront-for-several-resources).

```csharp
var result = await app.AcquireTokenInteractive(scopesForCustomerApi)
                     .WithExtraScopeToConsent(scopesForVendorApi)
                     .ExecuteAsync();
```

#### <a name="withcustomwebui"></a>WithCustomWebUi

Une interface utilisateur web est un mécanisme pour appeler un navigateur. Ce mécanisme peut être un contrôle WebBrowser dédié avec interface utilisateur ou un moyen de déléguer l’ouverture du navigateur.
MSAL fournit des implémentations d’interface utilisateur web pour la plupart des plateformes, mais il existe des cas où vous pourriez préférer héberger vous-même le navigateur :

- Pour les plateformes qui ne sont pas explicitement couvertes par MSAL, comme Blazor, Unity et Mono version bureau
- Si vous souhaitez tester l’interface utilisateur de votre application et vous servir d’un navigateur automatisé qui peut être utilisé avec Selenium
- Si le navigateur et l’application qui exécutent MSAL se trouvent dans des processus distincts

##### <a name="at-a-glance"></a>Aperçu

Pour ce faire, vous fournissez `start Url` à MSAL, qui doit s’afficher dans un navigateur de votre choix, afin que l’utilisateur final puisse entrer des informations telles que son nom d’utilisateur.
L’authentification terminée, votre application doit redonner `end Url` à MSAL, qui contient un code fourni par Azure AD.
L’hôte de `end Url` est toujours `redirectUri`. Pour intercepter `end Url`, effectuez l’une des opérations suivantes :

- Superviser les redirections du navigateur jusqu’à atteindre `redirect Url`
- Demander au navigateur de rediriger vers une URL que vous supervisez

##### <a name="withcustomwebui-is-an-extensibility-point"></a>WithCustomWebUi est un point d’extensibilité

`WithCustomWebUi` est un point d’extensibilité que vous pouvez utiliser pour fournir votre propre interface utilisateur dans les applications clientes publiques. Vous pouvez également laisser l’utilisateur passer par le point de terminaison /Authorize du fournisseur d’identité et l’autoriser à se connecter et à donner son consentement. MSAL.NET peut alors échanger le code d’authentification pour obtenir un jeton. Par exemple, il est utilisé dans Visual Studio, pour que des applications électrons (comme les commentaires Visual Studio) fournissent l’interaction web tout en laissant MSAL.NET faire le gros du travail. Vous pouvez également l’utiliser si vous voulez fournir UI automation. Dans les applications clientes publiques, MSAL.NET utilise la norme PKCE (Proof Key for Code Exchange) pour garantir le respect de la sécurité. seul MSAL.NET peut échanger le code. Pour plus d’informations, consultez [RFC 7636 - Proof Key for Code Exchange by OAuth Public Clients (RFC 7636 - Clé de preuve pour l’échange de clé par les clients publics OAuth)](https://tools.ietf.org/html/rfc7636).

  ```csharp
  using Microsoft.Identity.Client.Extensions;
  ```

##### <a name="use-withcustomwebui"></a>Utiliser WithCustomWebUi

Pour utiliser `.WithCustomWebUI`, suivez ces étapes.

  1. Implémentez l'interface `ICustomWebUi`. Pour plus d’informations, consultez [ce site web](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/blob/053a98d16596be7e9ca1ab916924e5736e341fe8/src/Microsoft.Identity.Client/Extensibility/ICustomWebUI.cs#L32-L70). Implémentez une méthode `AcquireAuthorizationCodeAsync` et acceptez l’URL de code d’autorisation calculée par MSAL.NET. Ensuite, permettez à l’utilisateur de parcourir l’interaction avec le fournisseur d’identité et de retourner l’URL par laquelle le fournisseur d’identité aurait rappelé votre implémentation avec le code d’autorisation. Si vous rencontrez des problèmes, votre implémentation doit lever une exception `MsalExtensionException` afin de pouvoir coopérer efficacement avec MSAL.
  2. Dans votre appel `AcquireTokenInteractive`, utilisez le modificateur `.WithCustomUI()` en passant l’instance de votre interface utilisateur web personnalisée.

     ```csharp
     result = await app.AcquireTokenInteractive(scopes)
                       .WithCustomWebUi(yourCustomWebUI)
                       .ExecuteAsync();
     ```

##### <a name="examples-of-implementation-of-icustomwebui-in-test-automation-seleniumwebui"></a>Exemples d’implémentation de ICustomWebUi dans l’automatisation de test : SeleniumWebUI

L’équipe de MSAL.NET a réécrit les tests d’interface utilisateur pour utiliser ce mécanisme d’extensibilité. Si vous êtes intéressé, examinez de plus près la classe [SeleniumWebUI](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/blob/053a98d16596be7e9ca1ab916924e5736e341fe8/tests/Microsoft.Identity.Test.Integration/Infrastructure/SeleniumWebUI.cs#L15-L160) dans le code source MSAL.NET.

##### <a name="provide-a-great-experience-with-systemwebviewoptions"></a>Fournir une bonne expérience avec SystemWebViewOptions

À partir de MSAL.NET 4.1 [`SystemWebViewOptions`](/dotnet/api/microsoft.identity.client.systemwebviewoptions), vous pouvez spécifier :

- L’URI vers laquelle naviguer (`BrowserRedirectError`) ou le fragment HTML à afficher (`HtmlMessageError`) en cas d’erreurs de connexion ou de consentement dans le navigateur web système.
- L’URI vers laquelle naviguer (`BrowserRedirectSuccess`) ou le fragment HTML à afficher (`HtmlMessageSuccess`) en cas de réussite de la connexion ou du consentement.
- L’action à exécuter pour démarrer le navigateur système. Vous pouvez fournir votre propre implémentation en définissant le délégué `OpenBrowserAsync`. La classe fournit également une implémentation par défaut pour deux navigateurs : `OpenWithEdgeBrowserAsync` et `OpenWithChromeEdgeBrowserAsync`, pour Microsoft Edge et [Microsoft Edge sur Chromium](https://www.windowscentral.com/faq-edge-chromium), respectivement.

Pour utiliser cette structure, écrivez quelque chose de similaire à l’exemple suivant :

```csharp
IPublicClientApplication app;
...

options = new SystemWebViewOptions
{
 HtmlMessageError = "<b>Sign-in failed. You can close this tab ...</b>",
 BrowserRedirectSuccess = "https://contoso.com/help-for-my-awesome-commandline-tool.html"
};

var result = app.AcquireTokenInteractive(scopes)
                .WithEmbeddedWebView(false)       // The default in .NET Core
                .WithSystemWebViewOptions(options)
                .Build();
```

#### <a name="other-optional-parameters"></a>Autres paramètres facultatifs

Pour en savoir plus sur tous les autres paramètres facultatifs de `AcquireTokenInteractive`, consultez [AcquireTokenInteractiveParameterBuilder](/dotnet/api/microsoft.identity.client.acquiretokeninteractiveparameterbuilder#methods).

# <a name="java"></a>[Java](#tab/java)

```java
private static IAuthenticationResult acquireTokenInteractive() throws Exception {

    // Load token cache from file and initialize token cache aspect. The token cache will have
    // dummy data, so the acquireTokenSilently call will fail.
    TokenCacheAspect tokenCacheAspect = new TokenCacheAspect("sample_cache.json");

    PublicClientApplication pca = PublicClientApplication.builder(CLIENT_ID)
            .authority(AUTHORITY)
            .setTokenCacheAccessAspect(tokenCacheAspect)
            .build();

    Set<IAccount> accountsInCache = pca.getAccounts().join();
    // Take first account in the cache. In a production application, you would filter
    // accountsInCache to get the right account for the user authenticating.
    IAccount account = accountsInCache.iterator().next();

    IAuthenticationResult result;
    try {
        SilentParameters silentParameters =
                SilentParameters
                        .builder(SCOPE, account)
                        .build();

        // try to acquire token silently. This call will fail since the token cache
        // does not have any data for the user you are trying to acquire a token for
        result = pca.acquireTokenSilently(silentParameters).join();
    } catch (Exception ex) {
        if (ex.getCause() instanceof MsalException) {

            InteractiveRequestParameters parameters = InteractiveRequestParameters
                    .builder(new URI("http://localhost"))
                    .scopes(SCOPE)
                    .build();

            // Try to acquire a token interactively with system browser. If successful, you should see
            // the token and account information printed out to console
            result = pca.acquireToken(parameters).join();
        } else {
            // Handle other exceptions accordingly
            throw ex;
        }
    }
    return result;
}
```

# <a name="python"></a>[Python](#tab/python)

MSAL Python ne fournit pas directement de méthode interactive d’acquisition de jetons. L’application doit envoyer une requête d’autorisation dans son implémentation du flux d’interaction utilisateur pour obtenir un code d’autorisation. Ce code peut ensuite être passé à la méthode `acquire_token_by_authorization_code` pour obtenir le jeton.

```Python
result = None

# Firstly, check the cache to see if this end user has signed in before
accounts = app.get_accounts(username=config["username"])
if accounts:
    result = app.acquire_token_silent(config["scope"], account=accounts[0])

if not result:
    result = app.acquire_token_by_authorization_code(
         request.args['code'],
         scopes=config["scope"])

```

# <a name="macos"></a>[macOS](#tab/macOS)

### <a name="in-msal-for-ios-and-macos"></a>Dans MSAL pour iOS et macOS

Objective-C :

```objc
MSALInteractiveTokenParameters *interactiveParams = [[MSALInteractiveTokenParameters alloc] initWithScopes:scopes webviewParameters:[MSALWebviewParameters new]];
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

Swift :

```swift
let interactiveParameters = MSALInteractiveTokenParameters(scopes: scopes, webviewParameters: MSALWebviewParameters())
application.acquireToken(with: interactiveParameters, completionBlock: { (result, error) in

    guard let authResult = result, error == nil else {
        print(error!.localizedDescription)
        return
    }

    // Get access token from result
    let accessToken = authResult.accessToken
})
```
---

## <a name="integrated-windows-authentication"></a>Authentification Windows intégrée

Pour connecter un utilisateur de domaine sur une machine jointe à Azure AD ou à un domaine, utilisez l’authentification Windows intégrée (IWA).

### <a name="constraints"></a>Contraintes

- L’authentification Windows intégrée n’est utilisable que pour les utilisateurs *fédérés+* , c’est-à-dire les utilisateurs créés dans Active Directory et reposant sur Azure AD. Les utilisateurs créés directement dans Azure AD sans appui Active Directory, appelés utilisateurs *managés*, ne peuvent pas utiliser ce flux d’authentification. Cette restriction ne concerne pas le flux de nom d’utilisateur et de mot de passe.
- L’authentification Windows intégrée est destinée aux applications écrites pour les plateformes .NET Framework, .NET Core et UWP, la plateforme Windows universelle.
- L’authentification Windows n’ignore pas l’[authentification multifacteur (MFA)](../authentication/concept-mfa-howitworks.md). Si l’authentification MFA est configurée, IWA peut échouer en cas de demande MFA exigée, car MFA a besoin d’une interaction utilisateur.
  > [!NOTE]
  > Voilà qui est délicat. L’authentification IWA est non interactive, mais MFA nécessite l’interactivité avec l’utilisateur. Vous n’avez pas le contrôle lorsque le fournisseur d’identité demande l’exécution de MFA, l’administrateur de locataire, si. D’après ce que nous avons pu observer, l’authentification MFA est demandée lorsque vous vous connectez depuis un autre pays/région alors que vous n’êtes pas connecté à un réseau d’entreprise via un VPN, et parfois même lorsque vous êtes connecté via un VPN. Ne vous attendez pas à un ensemble déterministe de règles. Azure AD utilise l’intelligence artificielle pour apprendre en continu à déterminer si l’authentification MFA est exigée. Ayez recours à une invite utilisateur de secours, comme une authentification interactive ou un flux de code d’appareil, si l’IWA échoue.

- L’autorité transmise dans `PublicClientApplicationBuilder` doit être :
  - Avec locataire sous le format `https://login.microsoftonline.com/{tenant}/`, dans lequel `tenant` est le GUID qui représente l’ID de locataire ou un domaine associé au locataire.
  - Pour les comptes professionnels et scolaires : `https://login.microsoftonline.com/organizations/`.
  - Les comptes Microsoft personnels ne sont pas pris en charge. Vous ne pouvez pas utiliser les locataires /common ou /consumers.

- Étant donné que l’authentification Windows intégrée est un flux silencieux :
  - L’utilisateur de votre application doit avoir précédemment donné son consentement pour utiliser l’application.
  - Ou bien, l’administrateur de locataires doit avoir préalablement consenti à ce que tous les utilisateurs dans le locataire utilisent l’application.
  - En d’autres termes :
    - En tant que développeur, vous avez sélectionné le bouton **Accorder** pour vous-même, dans le portail Azure.
    - Ou bien, un administrateur de locataire a sélectionné le bouton **Accorder/révoquer le consentement administrateur pour {domaine du locataire}** dans l’onglet **Autorisations de l’API** lors de l’inscription de l’application. Pour plus d’informations, consultez [Ajouter des autorisations pour accéder à votre API web](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-your-web-api).
    - Ou bien, vous avez fourni un moyen aux utilisateurs de donner leur consentement pour l’utilisation de l’application. Pour plus d’informations, consultez [Demande de consentement d’utilisateur individuel](./v2-permissions-and-consent.md#requesting-individual-user-consent).
    - Ou bien, vous avez fourni un moyen à l’administrateur de locataire de donner son consentement pour l’utilisation de l’application. Pour plus d’informations, consultez [Consentement administrateur](./v2-permissions-and-consent.md#requesting-consent-for-an-entire-tenant).

- Ce flux est activé pour les applications .NET Desktop, .NET Core et UWP.

Pour plus d’informations sur le consentement, consultez [Autorisations et consentement de la plateforme d’identités Microsoft](./v2-permissions-and-consent.md).

### <a name="learn-how-to-use-it"></a>Découvrez son utilisation

# <a name="net"></a>[.NET](#tab/dotnet)

Dans MSAL.NET, vous devez utiliser :

```csharp
AcquireTokenByIntegratedWindowsAuth(IEnumerable<string> scopes)
```

Vous n’avez normalement besoin que d’un paramètre (`scopes`). En fonction de la façon dont votre administrateur Windows a configuré les stratégies, les applications sur votre machine Windows peuvent ne pas être autorisées à rechercher l’utilisateur connecté. Dans ce cas, utilisez une deuxième méthode, `.WithUsername()`, et passez le nom d’utilisateur de l’utilisateur connecté sous un format UPN, par exemple, `joe@contoso.com`. Sur .NET Core, seule la surcharge acceptant le nom d’utilisateur est disponible, car la plateforme .NET Core ne peut pas demander le nom d’utilisateur au système d’exploitation.

L’exemple suivant met en évidence le cas le plus courant, avec des explications sur le genre d’exceptions que vous pouvez obtenir et les solutions d’atténuation.

```csharp
static async Task GetATokenForGraph()
{
 string authority = "https://login.microsoftonline.com/contoso.com";
 string[] scopes = new string[] { "user.read" };
 IPublicClientApplication app = PublicClientApplicationBuilder
      .Create(clientId)
      .WithAuthority(authority)
      .Build();

 var accounts = await app.GetAccountsAsync();

 AuthenticationResult result = null;
 if (accounts.Any())
 {
  result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
      .ExecuteAsync();
 }
 else
 {
  try
  {
   result = await app.AcquireTokenByIntegratedWindowsAuth(scopes)
      .ExecuteAsync(CancellationToken.None);
  }
  catch (MsalUiRequiredException ex)
  {
   // MsalUiRequiredException: AADSTS65001: The user or administrator has not consented to use the application
   // with ID '{appId}' named '{appName}'.Send an interactive authorization request for this user and resource.

   // you need to get user consent first. This can be done, if you are not using .NET Core (which does not have any Web UI)
   // by doing (once only) an AcquireToken interactive.

   // If you are using .NET core or don't want to do an AcquireTokenInteractive, you might want to suggest the user to navigate
   // to a URL to consent: https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id={clientId}&response_type=code&scope=user.read

   // AADSTS50079: The user is required to use multi-factor authentication.
   // There is no mitigation - if MFA is configured for your tenant and AAD decides to enforce it,
   // you need to fallback to an interactive flows such as AcquireTokenInteractive or AcquireTokenByDeviceCode
   }
   catch (MsalServiceException ex)
   {
    // Kind of errors you could have (in ex.Message)

    // MsalServiceException: AADSTS90010: The grant type is not supported over the /common or /consumers endpoints. Please use the /organizations or tenant-specific endpoint.
    // you used common.
    // Mitigation: as explained in the message from Azure AD, the authority needs to be tenanted or otherwise organizations

    // MsalServiceException: AADSTS70002: The request body must contain the following parameter: 'client_secret or client_assertion'.
    // Explanation: this can happen if your application was not registered as a public client application in Azure AD
    // Mitigation: in the Azure portal, edit the manifest for your application and set the `allowPublicClient` to `true`
   }
   catch (MsalClientException ex)
   {
      // Error Code: unknown_user Message: Could not identify logged in user
      // Explanation: the library was unable to query the current Windows logged-in user or this user is not AD or AAD
      // joined (work-place joined users are not supported).

      // Mitigation 1: on UWP, check that the application has the following capabilities: Enterprise Authentication,
      // Private Networks (Client and Server), User Account Information

      // Mitigation 2: Implement your own logic to fetch the username (e.g. john@contoso.com) and use the
      // AcquireTokenByIntegratedWindowsAuth form that takes in the username

      // Error Code: integrated_windows_auth_not_supported_managed_user
      // Explanation: This method relies on an a protocol exposed by Active Directory (AD). If a user was created in Azure
      // Active Directory without AD backing ("managed" user), this method will fail. Users created in AD and backed by
      // AAD ("federated" users) can benefit from this non-interactive method of authentication.
      // Mitigation: Use interactive authentication
   }
 }


 Console.WriteLine(result.Account.Username);
}
```

Pour la liste des modificateurs possibles sur AcquireTokenByIntegratedWindowsAuthentication, voir [AcquireTokenByIntegratedWindowsAuthParameterBuilder](/dotnet/api/microsoft.identity.client.acquiretokenbyintegratedwindowsauthparameterbuilder#methods).

# <a name="java"></a>[Java](#tab/java)

Voici un extrait des [exemples de développement MSAL Java](https://github.com/AzureAD/microsoft-authentication-library-for-java/blob/dev/src/samples/public-client/).

```Java
private static IAuthenticationResult acquireTokenIwa() throws Exception {

    // Load token cache from file and initialize token cache aspect. The token cache will have
    // dummy data, so the acquireTokenSilently call will fail.
    TokenCacheAspect tokenCacheAspect = new TokenCacheAspect("sample_cache.json");

    PublicClientApplication pca = PublicClientApplication.builder(CLIENT_ID)
            .authority(AUTHORITY)
            .setTokenCacheAccessAspect(tokenCacheAspect)
            .build();

    Set<IAccount> accountsInCache = pca.getAccounts().join();
    // Take first account in the cache. In a production application, you would filter
    // accountsInCache to get the right account for the user authenticating.
    IAccount account = accountsInCache.iterator().next();

    IAuthenticationResult result;
    try {
        SilentParameters silentParameters =
                SilentParameters
                        .builder(SCOPE, account)
                        .build();

        // try to acquire token silently. This call will fail since the token cache
        // does not have any data for the user you are trying to acquire a token for
        result = pca.acquireTokenSilently(silentParameters).join();
    } catch (Exception ex) {
        if (ex.getCause() instanceof MsalException) {

            IntegratedWindowsAuthenticationParameters parameters =
                    IntegratedWindowsAuthenticationParameters
                            .builder(SCOPE, USER_NAME)
                            .build();

            // Try to acquire a IWA. You will need to generate a Kerberos ticket.
            // If successful, you should see the token and account information printed out to
            // console
            result = pca.acquireToken(parameters).join();
        } else {
            // Handle other exceptions accordingly
            throw ex;
        }
    }
    return result;
}
```

# <a name="python"></a>[Python](#tab/python)

Ce flux n’est pas encore pris en charge dans MSAL Python.

# <a name="macos"></a>[macOS](#tab/macOS)

Ce flux ne s’applique pas à macOS.

---

## <a name="username-and-password"></a>Nom d’utilisateur et mot de passe

Vous pouvez également acquérir un jeton en fournissant le nom d’utilisateur et le mot de passe. Ce flux est limité et déconseillé, mais il existe toujours des cas d'usage où il est nécessaire.

### <a name="this-flow-isnt-recommended"></a>Ce flux n’est pas conseillé.

Ce flux est *déconseillé*, car le fait que votre application demande à un utilisateur son mot de passe n’est pas une pratique sûre. Pour plus d’informations, consultez l’article sur [la solution aux problèmes croissants associés aux mots de passe](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/). Le flux par défaut pour l’acquisition d’un jeton silencieusement sur les machines Windows jointes à un domaine est [l’authentification Windows intégrée](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Integrated-Windows-Authentication). Vous pouvez également utiliser le [flux de code d’appareil](https://aka.ms/msal-net-device-code-flow).

> [!NOTE]
> L’utilisation d’un nom d’utilisateur et d’un mot de passe est utile dans certains cas, comme dans les scénarios DevOps. Toutefois, si vous souhaitez utiliser un nom d’utilisateur et un mot de passe dans des scénarios interactifs pour lesquels vous fournissez votre propre interface utilisateur, prenez le temps de réfléchir à un moyen qui vous permettrait de l’éviter. En utilisant un nom d’utilisateur et un mot de passe, vous abandonnez un certain nombre de choses :
>
> - Les principes fondamentaux de l’identité moderne. Un mot de passe peut être récupéré et relu, car un secret partagé peut être intercepté. C’est incompatible avec l’option sans mot de passe.
> - Les utilisateurs qui doivent procéder à une authentification MFA ne peuvent pas se connecter, car il n’y a pas d’interaction.
> - Les utilisateurs ne peuvent pas disposer de l’authentification unique (SSO).

### <a name="constraints"></a>Contraintes

Les contraintes suivantes s’appliquent également :

- Le flux de nom d’utilisateur et de mot de passe n’est pas compatible avec l’accès conditionnel et l’authentification multifacteur. Par conséquent, si votre application s’exécute dans un locataire Azure AD pour lequel l’administrateur de locataires exige une authentification multifacteur, vous ne pouvez pas utiliser ce flux. De nombreuses organisations font cela.
- Ce moyen fonctionne uniquement pour les comptes professionnels et scolaires (pas les comptes MSA).
- Ce flux est disponible sur .NET Desktop et .NET Core, mais pas sur UWP.

### <a name="b2c-specifics"></a>Spécificités B2C

Pour plus d’informations, consultez [Informations d’identification du mot de passe du propriétaire de la ressource (ROPC) avec B2C](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-specifics#resource-owner-password-credentials-ropc-with-b2c).

### <a name="use-it"></a>Utilisation

# <a name="net"></a>[.NET](#tab/dotnet)

`IPublicClientApplication` contient la méthode `AcquireTokenByUsernamePassword`.

L’exemple suivant présente un cas simplifié.

```csharp
static async Task GetATokenForGraph()
{
 string authority = "https://login.microsoftonline.com/contoso.com";
 string[] scopes = new string[] { "user.read" };
 IPublicClientApplication app;
 app = PublicClientApplicationBuilder.Create(clientId)
       .WithAuthority(authority)
       .Build();
 var accounts = await app.GetAccountsAsync();

 AuthenticationResult result = null;
 if (accounts.Any())
 {
  result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
                    .ExecuteAsync();
 }
 else
 {
  try
  {
   var securePassword = new SecureString();
   foreach (char c in "dummy")        // you should fetch the password
    securePassword.AppendChar(c);  // keystroke by keystroke

   result = await app.AcquireTokenByUsernamePassword(scopes,
                                                    "joe@contoso.com",
                                                     securePassword)
                      .ExecuteAsync();
  }
  catch(MsalException)
  {
   // See details below
  }
 }
 Console.WriteLine(result.Account.Username);
}
```

L’exemple suivant met en évidence le cas le plus courant, avec des explications sur le genre d’exceptions que vous pouvez obtenir et les solutions d’atténuation.

```csharp
static async Task GetATokenForGraph()
{
 string authority = "https://login.microsoftonline.com/contoso.com";
 string[] scopes = new string[] { "user.read" };
 IPublicClientApplication app;
 app = PublicClientApplicationBuilder.Create(clientId)
                                   .WithAuthority(authority)
                                   .Build();
 var accounts = await app.GetAccountsAsync();

 AuthenticationResult result = null;
 if (accounts.Any())
 {
  result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
                    .ExecuteAsync();
 }
 else
 {
  try
  {
   var securePassword = new SecureString();
   foreach (char c in "dummy")        // you should fetch the password keystroke
    securePassword.AppendChar(c);  // by keystroke

   result = await app.AcquireTokenByUsernamePassword(scopes,
                                                    "joe@contoso.com",
                                                    securePassword)
                    .ExecuteAsync();
  }
  catch (MsalUiRequiredException ex) when (ex.Message.Contains("AADSTS65001"))
  {
   // Here are the kind of error messages you could have, and possible mitigations

   // ------------------------------------------------------------------------
   // MsalUiRequiredException: AADSTS65001: The user or administrator has not consented to use the application
   // with ID '{appId}' named '{appName}'. Send an interactive authorization request for this user and resource.

   // Mitigation: you need to get user consent first. This can be done either statically (through the portal),
   /// or dynamically (but this requires an interaction with Azure AD, which is not possible with
   // the username/password flow)
   // Statically: in the portal by doing the following in the "API permissions" tab of the application registration:
   // 1. Click "Add a permission" and add all the delegated permissions corresponding to the scopes you want (for instance
   // User.Read and User.ReadBasic.All)
   // 2. Click "Grant/revoke admin consent for <tenant>") and click "yes".
   // Dynamically, if you are not using .NET Core (which does not have any Web UI) by
   // calling (once only) AcquireTokenInteractive.
   // remember that Username/password is for public client applications that is desktop/mobile applications.
   // If you are using .NET core or don't want to call AcquireTokenInteractive, you might want to:
   // - use device code flow (See https://aka.ms/msal-net-device-code-flow)
   // - or suggest the user to navigate to a URL to consent: https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id={clientId}&response_type=code&scope=user.read
   // ------------------------------------------------------------------------

   // ------------------------------------------------------------------------
   // ErrorCode: invalid_grant
   // SubError: basic_action
   // MsalUiRequiredException: AADSTS50079: The user is required to use multi-factor authentication.
   // The tenant admin for your organization has chosen to oblige users to perform multi-factor authentication.
   // Mitigation: none for this flow
   // Your application cannot use the Username/Password grant.
   // Like in the previous case, you might want to use an interactive flow (AcquireTokenInteractive()),
   // or Device Code Flow instead.
   // Note this is one of the reason why using username/password is not recommended;
   // ------------------------------------------------------------------------

   // ------------------------------------------------------------------------
   // ex.ErrorCode: invalid_grant
   // subError: null
   // Message = "AADSTS70002: Error validating credentials.
   // AADSTS50126: Invalid username or password
   // In the case of a managed user (user from an Azure AD tenant opposed to a
   // federated user, which would be owned
   // in another IdP through ADFS), the user has entered the wrong password
   // Mitigation: ask the user to re-enter the password
   // ------------------------------------------------------------------------

   // ------------------------------------------------------------------------
   // ex.ErrorCode: invalid_grant
   // subError: null
   // MsalServiceException: ADSTS50034: To sign into this application the account must be added to
   // the {domainName} directory.
   // or The user account does not exist in the {domainName} directory. To sign into this application,
   // the account must be added to the directory.
   // The user was not found in the directory
   // Explanation: wrong username
   // Mitigation: ask the user to re-enter the username.
   // ------------------------------------------------------------------------
  }
  catch (MsalServiceException ex) when (ex.ErrorCode == "invalid_request")
  {
   // ------------------------------------------------------------------------
   // AADSTS90010: The grant type is not supported over the /common or /consumers endpoints.
   // Please use the /organizations or tenant-specific endpoint.
   // you used common.
   // Mitigation: as explained in the message from Azure AD, the authority you use in the application needs
   // to be tenanted or otherwise "organizations". change the
   // "Tenant": property in the appsettings.json to be a GUID (tenant Id), or domain name (contoso.com)
   // if such a domain is registered with your tenant
   // or "organizations", if you want this application to sign-in users in any Work and School accounts.
   // ------------------------------------------------------------------------

  }
  catch (MsalServiceException ex) when (ex.ErrorCode == "unauthorized_client")
  {
   // ------------------------------------------------------------------------
   // AADSTS700016: Application with identifier '{clientId}' was not found in the directory '{domain}'.
   // This can happen if the application has not been installed by the administrator of the tenant or consented
   // to by any user in the tenant.
   // You may have sent your authentication request to the wrong tenant
   // Cause: The clientId in the appsettings.json might be wrong
   // Mitigation: check the clientId and the app registration
   // ------------------------------------------------------------------------
  }
  catch (MsalServiceException ex) when (ex.ErrorCode == "invalid_client")
  {
   // ------------------------------------------------------------------------
   // AADSTS70002: The request body must contain the following parameter: 'client_secret or client_assertion'.
   // Explanation: this can happen if your application was not registered as a public client application in Azure AD
   // Mitigation: in the Azure portal, edit the manifest for your application and set the `allowPublicClient` to `true`
   // ------------------------------------------------------------------------
  }
  catch (MsalServiceException)
  {
   throw;
  }

  catch (MsalClientException ex) when (ex.ErrorCode == "unknown_user_type")
  {
   // Message = "Unsupported User Type 'Unknown'. Please see https://aka.ms/msal-net-up"
   // The user is not recognized as a managed user, or a federated user. Azure AD was not
   // able to identify the IdP that needs to process the user
   throw new ArgumentException("U/P: Wrong username", ex);
  }
  catch (MsalClientException ex) when (ex.ErrorCode == "user_realm_discovery_failed")
  {
   // The user is not recognized as a managed user, or a federated user. Azure AD was not
   // able to identify the IdP that needs to process the user. That's for instance the case
   // if you use a phone number
   throw new ArgumentException("U/P: Wrong username", ex);
  }
  catch (MsalClientException ex) when (ex.ErrorCode == "unknown_user")
  {
   // the username was probably empty
   // ex.Message = "Could not identify the user logged into the OS. See https://aka.ms/msal-net-iwa for details."
   throw new ArgumentException("U/P: Wrong username", ex);
  }
  catch (MsalClientException ex) when (ex.ErrorCode == "parsing_wstrust_response_failed")
  {
   // ------------------------------------------------------------------------
   // In the case of a Federated user (that is owned by a federated IdP, as opposed to a managed user owned in an Azure AD tenant)
   // ID3242: The security token could not be authenticated or authorized.
   // The user does not exist or has entered the wrong password
   // ------------------------------------------------------------------------
  }
 }

 Console.WriteLine(result.Account.Username);
}
```

Pour plus d’informations sur tous les modificateurs qui peuvent être appliqués à `AcquireTokenByUsernamePassword`, consultez [AcquireTokenByUsernamePasswordParameterBuilder](/dotnet/api/microsoft.identity.client.acquiretokenbyusernamepasswordparameterbuilder#methods).

# <a name="java"></a>[Java](#tab/java)

Voici un extrait des [exemples de développement MSAL Java](https://github.com/AzureAD/microsoft-authentication-library-for-java/blob/dev/src/samples/public-client/).

```Java
private static IAuthenticationResult acquireTokenUsernamePassword() throws Exception {

    // Load token cache from file and initialize token cache aspect. The token cache will have
    // dummy data, so the acquireTokenSilently call will fail.
    TokenCacheAspect tokenCacheAspect = new TokenCacheAspect("sample_cache.json");

    PublicClientApplication pca = PublicClientApplication.builder(CLIENT_ID)
            .authority(AUTHORITY)
            .setTokenCacheAccessAspect(tokenCacheAspect)
            .build();

    Set<IAccount> accountsInCache = pca.getAccounts().join();
    // Take first account in the cache. In a production application, you would filter
    // accountsInCache to get the right account for the user authenticating.
    IAccount account = accountsInCache.iterator().next();

    IAuthenticationResult result;
    try {
        SilentParameters silentParameters =
                SilentParameters
                        .builder(SCOPE, account)
                        .build();
        // try to acquire token silently. This call will fail since the token cache
        // does not have any data for the user you are trying to acquire a token for
        result = pca.acquireTokenSilently(silentParameters).join();
    } catch (Exception ex) {
        if (ex.getCause() instanceof MsalException) {

            UserNamePasswordParameters parameters =
                    UserNamePasswordParameters
                            .builder(SCOPE, USER_NAME, USER_PASSWORD.toCharArray())
                            .build();
            // Try to acquire a token via username/password. If successful, you should see
            // the token and account information printed out to console
            result = pca.acquireToken(parameters).join();
        } else {
            // Handle other exceptions accordingly
            throw ex;
        }
    }
    return result;
}
```

# <a name="python"></a>[Python](#tab/python)

Voici un extrait des [exemples de développement MSAL Python](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/dev/sample/).

```Python
# Create a preferably long-lived app instance which maintains a token cache.
app = msal.PublicClientApplication(
    config["client_id"], authority=config["authority"],
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache
    )

# The pattern to acquire a token looks like this.
result = None

# Firstly, check the cache to see if this end user has signed in before
accounts = app.get_accounts(username=config["username"])
if accounts:
    logging.info("Account(s) exists in cache, probably with token too. Let's try.")
    result = app.acquire_token_silent(config["scope"], account=accounts[0])

if not result:
    logging.info("No suitable token exists in cache. Let's get a new one from AAD.")
    # See this page for constraints of Username Password Flow.
    # https://github.com/AzureAD/microsoft-authentication-library-for-python/wiki/Username-Password-Authentication
    result = app.acquire_token_by_username_password(
        config["username"], config["password"], scopes=config["scope"])
```

# <a name="macos"></a>[macOS](#tab/macOS)

Ce flux n’est pas pris en charge sur MSAL pour macOS.

---

## <a name="command-line-tool-without-a-web-browser"></a>Outil en ligne de commande sans navigateur web

### <a name="device-code-flow"></a>Flux de code d’appareil

Si vous écrivez un outil en ligne de commande qui n’a pas de contrôles web, et que vous ne pouvez pas ou ne souhaitez pas utiliser les flux précédents, vous devez utiliser le flux de code d’appareil.

L’authentification interactive avec Azure AD nécessite un navigateur web. Pour plus d’informations, consultez [Utilisation de navigateurs Web](https://aka.ms/msal-net-uses-web-browser). Pour authentifier des utilisateurs sur des appareils ou des systèmes d’exploitation qui ne fournissent pas de navigateur web, le flux de code d’appareil permet à l’utilisateur de se servir d’un autre appareil, tel qu’un ordinateur ou un téléphone mobile, pour se connecter de manière interactive. En utilisant le flux de code d’appareil, l’application obtient les jetons via un processus en deux étapes, conçu pour ces appareils ou systèmes d’exploitation. De telles applications se retrouvent par exemple à s’exécuter sur IoT ou des outils en ligne de commande (CLI). L’idée est la suivante :

1. Chaque fois que l’authentification utilisateur est exigée, l’application fournit un code pour l’utilisateur. L’utilisateur est invité à utiliser un autre appareil, tel qu’un smartphone connecté à Internet, pour accéder à une URL, par exemple, `https://microsoft.com/devicelogin`. Ensuite, l’utilisateur est invité à entrer le code. Le code entré, la page web guide l’utilisateur dans un processus d’authentification normale, qui comporte des invites de consentement et une authentification multifacteur, si nécessaire.

2. Après une authentification réussie, l’application de ligne de commande reçoit les jetons demandés via un canal arrière et les utilise pour effectuer les appels d’API web dont elle a besoin.

### <a name="use-it"></a>Utilisation

# <a name="net"></a>[.NET](#tab/dotnet)

`IPublicClientApplication` contient une méthode nommée `AcquireTokenWithDeviceCode`.

```csharp
 AcquireTokenWithDeviceCode(IEnumerable<string> scopes,
                            Func<DeviceCodeResult, Task> deviceCodeResultCallback)
```

Cette méthode accepte comme paramètres :

- `scopes` qui permet de demander un jeton d’accès pour.
- Un rappel qui reçoit `DeviceCodeResult`.

  ![Propriétés DeviceCodeResult](https://user-images.githubusercontent.com/13203188/56024968-7af1b980-5d11-11e9-84c2-5be2ef306dc5.png)

L’exemple de code suivant met en évidence le cas le plus courant, avec des explications sur le genre d’exceptions que vous pouvez obtenir et les solution d’atténuation.

```csharp
private const string ClientId = "<client_guid>";
private const string Authority = "https://login.microsoftonline.com/contoso.com";
private readonly string[] scopes = new string[] { "user.read" };

static async Task<AuthenticationResult> GetATokenForGraph()
{
    IPublicClientApplication pca = PublicClientApplicationBuilder
            .Create(ClientId)
            .WithAuthority(Authority)
            .WithDefaultRedirectUri()
            .Build();

    var accounts = await pca.GetAccountsAsync();

    // All AcquireToken* methods store the tokens in the cache, so check the cache first
    try
    {
        return await pca.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
            .ExecuteAsync();
    }
    catch (MsalUiRequiredException ex)
    {
        // No token found in the cache or AAD insists that a form interactive auth is required (e.g. the tenant admin turned on MFA)
        // If you want to provide a more complex user experience, check out ex.Classification

        return await AcquireByDeviceCodeAsync(pca);
    }
}

private async Task<AuthenticationResult> AcquireByDeviceCodeAsync(IPublicClientApplication pca)
{
    try
    {
        var result = await pca.AcquireTokenWithDeviceCode(scopes,
            deviceCodeResult =>
            {
                    // This will print the message on the console which tells the user where to go sign-in using
                    // a separate browser and the code to enter once they sign in.
                    // The AcquireTokenWithDeviceCode() method will poll the server after firing this
                    // device code callback to look for the successful login of the user via that browser.
                    // This background polling (whose interval and timeout data is also provided as fields in the
                    // deviceCodeCallback class) will occur until:
                    // * The user has successfully logged in via browser and entered the proper code
                    // * The timeout specified by the server for the lifetime of this code (typically ~15 minutes) has been reached
                    // * The developing application calls the Cancel() method on a CancellationToken sent into the method.
                    //   If this occurs, an OperationCanceledException will be thrown (see catch below for more details).
                    Console.WriteLine(deviceCodeResult.Message);
                return Task.FromResult(0);
            }).ExecuteAsync();

        Console.WriteLine(result.Account.Username);
        return result;
    }
    // TODO: handle or throw all these exceptions depending on your app
    catch (MsalServiceException ex)
    {
        // Kind of errors you could have (in ex.Message)

        // AADSTS50059: No tenant-identifying information found in either the request or implied by any provided credentials.
        // Mitigation: as explained in the message from Azure AD, the authoriy needs to be tenanted. you have probably created
        // your public client application with the following authorities:
        // https://login.microsoftonline.com/common or https://login.microsoftonline.com/organizations

        // AADSTS90133: Device Code flow is not supported under /common or /consumers endpoint.
        // Mitigation: as explained in the message from Azure AD, the authority needs to be tenanted

        // AADSTS90002: Tenant <tenantId or domain you used in the authority> not found. This may happen if there are
        // no active subscriptions for the tenant. Check with your subscription administrator.
        // Mitigation: if you have an active subscription for the tenant this might be that you have a typo in the
        // tenantId (GUID) or tenant domain name.
    }
    catch (OperationCanceledException ex)
    {
        // If you use a CancellationToken, and call the Cancel() method on it, then this *may* be triggered
        // to indicate that the operation was cancelled.
        // See https://docs.microsoft.com/dotnet/standard/threading/cancellation-in-managed-threads
        // for more detailed information on how C# supports cancellation in managed threads.
    }
    catch (MsalClientException ex)
    {
        // Possible cause - verification code expired before contacting the server
        // This exception will occur if the user does not manage to sign-in before a time out (15 mins) and the
        // call to `AcquireTokenWithDeviceCode` is not cancelled in between
    }
}
```
# <a name="java"></a>[Java](#tab/java)

Voici un extrait des [exemples de développement MSAL Java](https://github.com/AzureAD/microsoft-authentication-library-for-java/blob/dev/src/samples/public-client/).

```java
private static IAuthenticationResult acquireTokenDeviceCode() throws Exception {

    // Load token cache from file and initialize token cache aspect. The token cache will have
    // dummy data, so the acquireTokenSilently call will fail.
    TokenCacheAspect tokenCacheAspect = new TokenCacheAspect("sample_cache.json");

    PublicClientApplication pca = PublicClientApplication.builder(CLIENT_ID)
            .authority(AUTHORITY)
            .setTokenCacheAccessAspect(tokenCacheAspect)
            .build();

    Set<IAccount> accountsInCache = pca.getAccounts().join();
    // Take first account in the cache. In a production application, you would filter
    // accountsInCache to get the right account for the user authenticating.
    IAccount account = accountsInCache.iterator().next();

    IAuthenticationResult result;
    try {
        SilentParameters silentParameters =
                SilentParameters
                        .builder(SCOPE, account)
                        .build();

        // try to acquire token silently. This call will fail since the token cache
        // does not have any data for the user you are trying to acquire a token for
        result = pca.acquireTokenSilently(silentParameters).join();
    } catch (Exception ex) {
        if (ex.getCause() instanceof MsalException) {

            Consumer<DeviceCode> deviceCodeConsumer = (DeviceCode deviceCode) ->
                    System.out.println(deviceCode.message());

            DeviceCodeFlowParameters parameters =
                    DeviceCodeFlowParameters
                            .builder(SCOPE, deviceCodeConsumer)
                            .build();

            // Try to acquire a token via device code flow. If successful, you should see
            // the token and account information printed out to console, and the sample_cache.json
            // file should have been updated with the latest tokens.
            result = pca.acquireToken(parameters).join();
        } else {
            // Handle other exceptions accordingly
            throw ex;
        }
    }
    return result;
}
```

# <a name="python"></a>[Python](#tab/python)

Voici un extrait des [exemples de développement MSAL Python](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/dev/sample/).

```Python
# Create a preferably long-lived app instance which maintains a token cache.
app = msal.PublicClientApplication(
    config["client_id"], authority=config["authority"],
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache
    )

# The pattern to acquire a token looks like this.
result = None

# Note: If your device-flow app does not have any interactive ability, you can
#   completely skip the following cache part. But here we demonstrate it anyway.
# We now check the cache to see if we have some end users signed in before.
accounts = app.get_accounts()
if accounts:
    logging.info("Account(s) exists in cache, probably with token too. Let's try.")
    print("Pick the account you want to use to proceed:")
    for a in accounts:
        print(a["username"])
    # Assuming the end user chose this one
    chosen = accounts[0]
    # Now let's try to find a token in cache for this account
    result = app.acquire_token_silent(config["scope"], account=chosen)

if not result:
    logging.info("No suitable token exists in cache. Let's get a new one from AAD.")

    flow = app.initiate_device_flow(scopes=config["scope"])
    if "user_code" not in flow:
        raise ValueError(
            "Fail to create device flow. Err: %s" % json.dumps(flow, indent=4))

    print(flow["message"])
    sys.stdout.flush()  # Some terminal needs this to ensure the message is shown

    # Ideally you should wait here, in order to save some unnecessary polling
    # input("Press Enter after signing in from another device to proceed, CTRL+C to abort.")

    result = app.acquire_token_by_device_flow(flow)  # By default it will block
        # You can follow this instruction to shorten the block time
        #    https://msal-python.readthedocs.io/en/latest/#msal.PublicClientApplication.acquire_token_by_device_flow
        # or you may even turn off the blocking behavior,
        # and then keep calling acquire_token_by_device_flow(flow) in your own customized loop
```

# <a name="macos"></a>[macOS](#tab/macOS)

Ce flux ne s’applique pas à macOS.

---

## <a name="file-based-token-cache"></a>Cache de jeton basé sur les fichiers

Dans MSAL.NET, un cache de jetons en mémoire est fourni par défaut.

### <a name="serialization-is-customizable-in-windows-desktop-apps-and-web-apps-or-web-apis"></a>Sérialisation personnalisable dans les applications de bureau Windows et les applications web ou les API web

Dans les cas de .NET Framework et de .NET Core, si vous ne faites rien de plus que d’habitude, le cache de jeton en mémoire dure toute la durée de l’application. Pour comprendre la raison pour laquelle la sérialisation n’est pas fournie dès le départ, il faut se souvenir que les applications de bureau MSAL .NET ou les applications .NET Core peuvent être des applications Windows ou de console (susceptibles d’avoir accès au système de fichiers), *mais également* des applications web ou des API web. Ces applications web et API web peuvent utiliser des mécanismes de cache particuliers, tels que des bases de données, des caches distribués et des caches Redis. Pour obtenir une application de cache de jeton persistante dans une application de bureau .NET ou dans .NET Core, vous devez personnaliser la sérialisation.

Les classes et interfaces concernées par la sérialisation du cache de jeton présentent les types suivants :

- ``ITokenCache``, qui définit des événements pour s’abonner à des requêtes de sérialisation du cache de jeton, et des méthodes pour sérialiser ou désérialiser le cache dans différents formats (ADAL v3.0, MSAL 2.x et MSAL 3.x = ADAL v5.0).
- ``TokenCacheCallback`` est un rappel transmis aux événements pour que vous puissiez gérer la sérialisation. Ceux-ci sont appelés avec des arguments de type ``TokenCacheNotificationArgs``.
- ``TokenCacheNotificationArgs`` fournit uniquement le ``ClientId`` d’application ainsi qu’une référence à l’utilisateur pour lequel le jeton est disponible.

  ![Diagramme de la sérialisation du cache de jeton](https://user-images.githubusercontent.com/13203188/56027172-d58d1480-5d15-11e9-8ada-c0292f1800b3.png)

> [!IMPORTANT]
> MSAL.NET crée les caches de jetons automatiquement et vous fournit le cache `IToken` quand vous appelez les propriétés `UserTokenCache` et `AppTokenCache` d’une application. Vous n’êtes pas censé implémenter l’interface vous-même. Votre responsabilité, lorsque vous implémentez une sérialisation du cache de jetons personnalisée, consiste à :
>
> - Réagir aux événements `BeforeAccess` et `AfterAccess` ou à leurs équivalents *Async*. Le délégué `BeforeAccess` est responsable de la désérialisation du cache. Le délégué `AfterAccess` est responsable de la sérialisation du cache.
> - Comprendre qu’une partie de ces événements stocke ou charge des objets blob, lesquels sont transmis par le biais de l’argument de l’événement au stockage voulu.

Les stratégies diffèrent selon que vous écrivez une sérialisation du cache de jeton pour une application cliente publique, comme une application de bureau, ou pour une application cliente confidentielle, telle qu’une application web, une API web ou une application démon.

À compter de MSAL v2.x, vous disposez de plusieurs options. Votre choix varie selon que vous souhaitez sérialiser le cache uniquement au format MSAL.NET, qui est un cache de format unifié, courant avec MSAL, mais aussi sur les plateformes. Sinon, vous pouvez également prendre en charge la sérialisation du cache de jeton [héritée](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Token-cache-serialization) d’ADAL v3.

La personnalisation de la sérialisation du cache de jeton pour partager l’état de l’authentification unique entre ADAL.NET 3.x, ADAL.NET 5.x et MSAL.NET est expliquée dans une partie de l’exemple : [active-directory-dotnet-v1-to-v2](https://github.com/Azure-Samples/active-directory-dotnet-v1-to-v2).

### <a name="simple-token-cache-serialization-msal-only"></a>Sérialisation simple du cache de jetons (MSAL.NET uniquement)

Voici un exemple d’implémentation native de la sérialisation personnalisée d’un cache de jeton pour des applications de bureau. Ici, le cache de jeton utilisateur est un fichier situé dans le même dossier que l’application.

Après avoir généré l’application, vous activez la sérialisation en appelant ``TokenCacheHelper.EnableSerialization()`` et en passant l’application `UserTokenCache`.

```csharp
app = PublicClientApplicationBuilder.Create(ClientId)
    .Build();
TokenCacheHelper.EnableSerialization(app.UserTokenCache);
```

Cette classe d’assistance ressemble à l’extrait de code suivant :

```csharp
static class TokenCacheHelper
 {
  public static void EnableSerialization(ITokenCache tokenCache)
  {
   tokenCache.SetBeforeAccess(BeforeAccessNotification);
   tokenCache.SetAfterAccess(AfterAccessNotification);
  }

  /// <summary>
  /// Path to the token cache
  /// </summary>
  public static readonly string CacheFilePath = System.Reflection.Assembly.GetExecutingAssembly().Location + ".msalcache.bin3";

  private static readonly object FileLock = new object();


  private static void BeforeAccessNotification(TokenCacheNotificationArgs args)
  {
   lock (FileLock)
   {
    args.TokenCache.DeserializeMsalV3(File.Exists(CacheFilePath)
            ? ProtectedData.Unprotect(File.ReadAllBytes(CacheFilePath),
                                      null,
                                      DataProtectionScope.CurrentUser)
            : null);
   }
  }

  private static void AfterAccessNotification(TokenCacheNotificationArgs args)
  {
   // if the access operation resulted in a cache update
   if (args.HasStateChanged)
   {
    lock (FileLock)
    {
     // reflect changesgs in the persistent store
     File.WriteAllBytes(CacheFilePath,
                         ProtectedData.Protect(args.TokenCache.SerializeMsalV3(),
                                                 null,
                                                 DataProtectionScope.CurrentUser)
                         );
    }
   }
  }
 }
```

Une préversion de sérialiseur basé sur un fichier de cache de jeton de qualité produit, pour des applications clientes publiques et des applications de bureau s’exécutant sur Windows, Mac et Linux, est disponible depuis la bibliothèque open source [Microsoft.Identity.Client.Extensions.Msal](https://github.com/AzureAD/microsoft-authentication-extensions-for-dotnet/tree/master/src/Microsoft.Identity.Client.Extensions.Msal). Vous pouvez l’inclure dans vos applications à partir du package NuGet suivant : [Microsoft.Identity.Client.Extensions.Msal](https://www.nuget.org/packages/Microsoft.Identity.Client.Extensions.Msal/).

> [!NOTE]
> Clause d’exclusion de responsabilité : La bibliothèque Microsoft.Identity.Client.Extensions.Msal est une extension sur MSAL.NET. Il est possible que les classes de ces bibliothèques soient intégrées à MSAL.NET à l’avenir, en l’état ou avec des changements cassants.

### <a name="dual-token-cache-serialization-msal-unified-cache--adal-v3"></a>Sérialisation d’un cache de jeton double (cache unifié MSAL + ADAL v3)

Le cas échéant, vous pouvez implémenter la sérialisation du cache de jeton avec le format de cache unifié. Ce format est courant sur ADAL.NET 4.x et MSAL.NET 2.x, ainsi qu’avec d’autres MSAL de la même génération ou plus ancienne, sur la même plateforme. Inspirez-vous du code suivant :

```csharp
string appLocation = Path.GetDirectoryName(Assembly.GetEntryAssembly().Location;
string cacheFolder = Path.GetFullPath(appLocation) + @"..\..\..\..");
string adalV3cacheFileName = Path.Combine(cacheFolder, "cacheAdalV3.bin");
string unifiedCacheFileName = Path.Combine(cacheFolder, "unifiedCache.bin");

IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
                                    .Build();
FilesBasedTokenCacheHelper.EnableSerialization(app.UserTokenCache,
                                               unifiedCacheFileName,
                                               adalV3cacheFileName);

```

Cette fois-ci, la classe d’assistance ressemble au code suivant :

```csharp
using System;
using System.IO;
using System.Security.Cryptography;
using Microsoft.Identity.Client;

namespace CommonCacheMsalV3
{
 /// <summary>
 /// Simple persistent cache implementation of the dual cache serialization (ADAL V3 legacy
 /// and unified cache format) for a desktop applications (from MSAL 2.x)
 /// </summary>
 static class FilesBasedTokenCacheHelper
 {
  /// <summary>
  /// Get the user token cache
  /// </summary>
  /// <param name="adalV3CacheFileName">File name where the cache is serialized with the
  /// ADAL V3 token cache format. Can
  /// be <c>null</c> if you don't want to implement the legacy ADAL V3 token cache
  /// serialization in your MSAL 2.x+ application</param>
  /// <param name="unifiedCacheFileName">File name where the cache is serialized
  /// with the Unified cache format, common to
  /// ADAL V4 and MSAL V2 and above, and also across ADAL/MSAL on the same platform.
  ///  Should not be <c>null</c></param>
  /// <returns></returns>
  public static void EnableSerialization(ITokenCache cache, string unifiedCacheFileName, string adalV3CacheFileName)
  {
   UnifiedCacheFileName = unifiedCacheFileName;
   AdalV3CacheFileName = adalV3CacheFileName;

   cache.SetBeforeAccess(BeforeAccessNotification);
   cache.SetAfterAccess(AfterAccessNotification);
  }

  /// <summary>
  /// File path where the token cache is serialized with the unified cache format
  /// (ADAL.NET V4, MSAL.NET V3)
  /// </summary>
  public static string UnifiedCacheFileName { get; private set; }

  /// <summary>
  /// File path where the token cache is serialized with the legacy ADAL V3 format
  /// </summary>
  public static string AdalV3CacheFileName { get; private set; }

  private static readonly object FileLock = new object();

  public static void BeforeAccessNotification(TokenCacheNotificationArgs args)
  {
   lock (FileLock)
   {
    args.TokenCache.DeserializeAdalV3(ReadFromFileIfExists(AdalV3CacheFileName));
    try
    {
     args.TokenCache.DeserializeMsalV3(ReadFromFileIfExists(UnifiedCacheFileName));
    }
    catch(Exception ex)
    {
     // Compatibility with the MSAL v2 cache if you used one
     args.TokenCache.DeserializeMsalV2(ReadFromFileIfExists(UnifiedCacheFileName));
    }
   }
  }

  public static void AfterAccessNotification(TokenCacheNotificationArgs args)
  {
   // if the access operation resulted in a cache update
   if (args.HasStateChanged)
   {
    lock (FileLock)
    {
     WriteToFileIfNotNull(UnifiedCacheFileName, args.TokenCache.SerializeMsalV3());
     if (!string.IsNullOrWhiteSpace(AdalV3CacheFileName))
     {
      WriteToFileIfNotNull(AdalV3CacheFileName, args.TokenCache.SerializeAdalV3());
     }
    }
   }
  }

  /// <summary>
  /// Read the content of a file if it exists
  /// </summary>
  /// <param name="path">File path</param>
  /// <returns>Content of the file (in bytes)</returns>
  private static byte[] ReadFromFileIfExists(string path)
  {
   byte[] protectedBytes = (!string.IsNullOrEmpty(path) && File.Exists(path))
       ? File.ReadAllBytes(path) : null;
   byte[] unprotectedBytes = encrypt ?
       ((protectedBytes != null) ? ProtectedData.Unprotect(protectedBytes, null, DataProtectionScope.CurrentUser) : null)
       : protectedBytes;
   return unprotectedBytes;
  }

  /// <summary>
  /// Writes a blob of bytes to a file. If the blob is <c>null</c>, deletes the file
  /// </summary>
  /// <param name="path">path to the file to write</param>
  /// <param name="blob">Blob of bytes to write</param>
  private static void WriteToFileIfNotNull(string path, byte[] blob)
  {
   if (blob != null)
   {
    byte[] protectedBytes = encrypt
      ? ProtectedData.Protect(blob, null, DataProtectionScope.CurrentUser)
      : blob;
    File.WriteAllBytes(path, protectedBytes);
   }
   else
   {
    File.Delete(path);
   }
  }

  // Change if you want to test with an un-encrypted blob (this is a json format)
  private static bool encrypt = true;
 }
}
```

## <a name="advanced-accessing-the-users-cached-tokens-in-background-apps-and-services"></a>(Avancé) Accès aux jetons mis en cache de l’utilisateur dans les applications et services en arrière-plan

[!INCLUDE [advanced-token-caching](../../../includes/advanced-token-cache.md)]

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Appeler une API web à partir de l’appareil de bureau](scenario-desktop-call-api.md).
