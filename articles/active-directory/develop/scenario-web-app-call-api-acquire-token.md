---
title: Obtenir un jeton pour une application web appelant des API web | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment acquérir un jeton pour une application web qui appelle des API web
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/25/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: c8af9ddd3649e2a5757e47ab2a3d8edca1c83929
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94442614"
---
# <a name="a-web-app-that-calls-web-apis-acquire-a-token-for-the-app"></a>Application web qui appelle des API web : Acquérir un jeton pour l’application

Vous avez généré l’objet de votre application cliente. À présent, vous allez l’utiliser pour acquérir un jeton afin d’appeler une API web. Dans ASP.NET ou ASP.NET Core, l’appel d’une API web s’effectue dans le contrôleur :

- Obtenir un jeton pour l’API web en utilisant le cache de jetons. Pour recevoir ce jeton, vous appelez la méthode MSAL `AcquireTokenSilent` (ou l’équivalent dans Microsoft.Identity.Web).
- Appelez l’API protégée, en lui transmettant le jeton d’accès en tant que paramètre.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

*Microsoft.Identity.Web* ajoute des méthodes d’extension qui fournissent des services pratiques pour appeler Microsoft Graph ou une API web en aval. Ces méthodes sont expliquées en détail dans [Application web qui appelle des API web : Appeler une API](scenario-web-app-call-api-call-api.md). Avec ces méthodes d’assistance, vous n’avez pas besoin d’acquérir manuellement un jeton.

Toutefois, si vous souhaitez acquérir manuellement un jeton, le code suivant montre un exemple d’utilisation de *Microsoft.Identity.Web* pour le faire dans un contrôleur home. Il appelle Microsoft Graph à l’aide de l’API REST (au lieu du Kit de développement logiciel (SDK) Microsoft Graph). Pour obtenir un jeton permettant d’appeler l’API en aval, injectez le service `ITokenAcquisition` par injection de dépendance dans le constructeur de votre contrôleur (ou votre constructeur de page, si vous utilisez Blazor), puis utilisez-le dans vos actions de contrôleur, en obtenant un jeton pour l’utilisateur (`GetAccessTokenForUserAsync`) ou pour l’application elle-même (`GetAccessTokenForAppAsync`) dans un scénario basé sur un démon.

Les méthodes de contrôleur sont protégées par un attribut `[Authorize]` qui veille à ce que seuls des utilisateurs authentifiés puissent utiliser l’application web.

```csharp
[Authorize]
public class HomeController : Controller
{
 readonly ITokenAcquisition tokenAcquisition;

 public HomeController(ITokenAcquisition tokenAcquisition)
 {
  this.tokenAcquisition = tokenAcquisition;
 }

 // Code for the controller actions (see code below)

}
```

Le service `ITokenAcquisition` est injecté par ASP.NET à l’aide de l’injection de dépendances.

Voici le code simplifié pour l’action du `HomeController`, qui obtient un jeton pour appeler Microsoft Graph :

```csharp
[AuthorizeForScopes(Scopes = new[] { "user.read" })]
public async Task<IActionResult> Profile()
{
 // Acquire the access token.
 string[] scopes = new string[]{"user.read"};
 string accessToken = await tokenAcquisition.GetAccessTokenForUserAsync(scopes);

 // Use the access token to call a protected web API.
 HttpClient client = new HttpClient();
 client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
 string json = await client.GetStringAsync(url);
}
```

Pour mieux comprendre le code nécessaire dans ce scénario, consultez la phase 2 ([2-1-Web App Calls Microsoft Graph](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-1-Call-MSGraph)) [Application web appelant Microsoft Graph] du tutoriel [ms-identity-aspnetcore-webapp-tutorial](https://github.com/Azure-Samples/ms-identity-aspnetcore-webapp-tutorial).

L’attribut `AuthorizeForScopes` en haut de l’action du contrôleur (ou de la page Razor si vous utilisez un modèle Razor) est fourni par Microsoft.Identity.Web. Il s’assure que l’utilisateur est invité à donner son consentement, le cas échéant, et de façon incrémentielle.

Il existe d’autres variantes complexes, telles que :

- Appels de plusieurs API.
- Traitement du consentement incrémentiel et de l’accès conditionnel.

Ces étapes avancées sont abordées dans le chapitre 3 du tutoriel [3-WebApp-multi-APIs](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/3-WebApp-multi-APIs).

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Le code pour ASP.NET est similaire au code présenté pour ASP.NET Core :

- Une action de contrôleur, protégée par un attribut [Authorize], extrait l’ID de locataire et l’identifiant utilisateur du membre `ClaimsPrincipal` du contrôleur. (ASP.NET utilise `HttpContext.User`.)
- Il génère alors un objet `IConfidentialClientApplication` MSAL.NET.
- Enfin, il appelle la méthode `AcquireTokenSilent` de l’application cliente confidentielle.
- Si une interaction est requise, l’application web doit vérifier l’utilisateur (réouverture de session) et demander des revendications supplémentaires.

L’extrait de code suivant est extrait de [HomeController.cs#L157-L192](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/257c8f96ec3ff875c351d1377b36403eed942a18/WebApp/Controllers/HomeController.cs#L157-L192) dans l’exemple de code MVC ASP.NET [ms-identity-aspnet-webapp-openidconnect](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect) :

```C#
public async Task<ActionResult> ReadMail()
{
    IConfidentialClientApplication app = MsalAppBuilder.BuildConfidentialClientApplication();
    AuthenticationResult result = null;
    var account = await app.GetAccountAsync(ClaimsPrincipal.Current.GetMsalAccountId());
    string[] scopes = { "Mail.Read" };

    try
    {
        // try to get token silently
        result = await app.AcquireTokenSilent(scopes, account).ExecuteAsync().ConfigureAwait(false);
    }
    catch (MsalUiRequiredException)
    {
        ViewBag.Relogin = "true";
        return View();
    }

    // More code here
    return View();
}
```

Pour plus d’informations, consultez le code relatif à [BuildConfidentialClientApplication()](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/master/WebApp/Utils/MsalAppBuilder.cs) et [GetMsalAccountId](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/257c8f96ec3ff875c351d1377b36403eed942a18/WebApp/Utils/ClaimPrincipalExtension.cs#L38) dans l’exemple de code.


# <a name="java"></a>[Java](#tab/java)

Dans l’exemple Java, le code qui appelle une API se trouve dans la méthode getUsersFromGraph dans [AuthPageController.java#L62](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/AuthPageController.java#L62).

La méthode tente d’appeler `getAuthResultBySilentFlow`. Si l’utilisateur a besoin de consentir à d’autres étendues, le code traite l’objet `MsalInteractionRequiredException` pour interroger l’utilisateur.

```java
@RequestMapping("/msal4jsample/graph/me")
public ModelAndView getUserFromGraph(HttpServletRequest httpRequest, HttpServletResponse response)
        throws Throwable {

    IAuthenticationResult result;
    ModelAndView mav;
    try {
        result = authHelper.getAuthResultBySilentFlow(httpRequest, response);
    } catch (ExecutionException e) {
        if (e.getCause() instanceof MsalInteractionRequiredException) {

            // If the silent call returns MsalInteractionRequired, redirect to authorization endpoint
            // so user can consent to new scopes.
            String state = UUID.randomUUID().toString();
            String nonce = UUID.randomUUID().toString();

            SessionManagementHelper.storeStateAndNonceInSession(httpRequest.getSession(), state, nonce);

            String authorizationCodeUrl = authHelper.getAuthorizationCodeUrl(
                    httpRequest.getParameter("claims"),
                    "User.Read",
                    authHelper.getRedirectUriGraph(),
                    state,
                    nonce);

            return new ModelAndView("redirect:" + authorizationCodeUrl);
        } else {

            mav = new ModelAndView("error");
            mav.addObject("error", e);
            return mav;
        }
    }

    if (result == null) {
        mav = new ModelAndView("error");
        mav.addObject("error", new Exception("AuthenticationResult not found in session."));
    } else {
        mav = new ModelAndView("auth_page");
        setAccountInfo(mav, httpRequest);

        try {
            mav.addObject("userInfo", getUserInfoFromGraph(result.accessToken()));

            return mav;
        } catch (Exception e) {
            mav = new ModelAndView("error");
            mav.addObject("error", e);
        }
    }
    return mav;
}
// Code omitted here
```

# <a name="python"></a>[Python](#tab/python)

Dans l’exemple Python, le code qui appelle Microsoft Graph se trouve dans [app.py#L53-L62](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/48637475ed7d7733795ebeac55c5d58663714c60/app.py#L53-L62).

Le code tente d’obtenir un jeton à partir du cache de jetons. Ensuite, après avoir défini l’en-tête d’autorisation, il appelle l’API web. S’il ne peut pas obtenir de jeton, il reconnecte l’utilisateur.

```python
@app.route("/graphcall")
def graphcall():
    token = _get_token_from_cache(app_config.SCOPE)
    if not token:
        return redirect(url_for("login"))
    graph_data = requests.get(  # Use token to call downstream service.
        app_config.ENDPOINT,
        headers={'Authorization': 'Bearer ' + token['access_token']},
        ).json()
    return render_template('display.html', result=graph_data)
```

---

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Appeler une API web](scenario-web-app-call-api-call-api.md).
