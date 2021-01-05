---
title: Écrire une application web qui connecte/déconnecte des utilisateurs – Plateforme Microsoft Identity | Azure
description: Apprenez à générer une application web qui connecte/déconnecte des utilisateurs (connexion)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/14/2020
ms.author: jmprieur
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: e7397f6d02d71a6344953b8210b0349b9ee26360
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94443549"
---
# <a name="web-app-that-signs-in-users-sign-in-and-sign-out"></a>Application web qui connecte les utilisateurs : Se connecter et se déconnecter

Découvrez comment ajouter une connexion au code de votre application web pour connecter les utilisateurs. Ensuite, découvrez comment les autoriser à se déconnecter.

## <a name="sign-in"></a>Connexion

La connexion comprend deux parties :

- Le bouton de connexion dans la page HTML
- L’action de connexion dans le code-behind du contrôleur

### <a name="sign-in-button"></a>Bouton de connexion

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Dans ASP.NET Core, pour les applications de plateforme d’identité Microsoft, le bouton **Se connecter** est exposé dans `Views\Shared\_LoginPartial.cshtml` (pour une application MVC) ou `Pages\Shared\_LoginPartial.cshtm` (pour une application Razor). Elles s’affichent uniquement quand l’utilisateur n’est pas authentifié. Autrement dit, il s’affiche lorsque l’utilisateur ne s’est pas encore connecté ou qu’il s’est déconnecté. Au contraire, le bouton **Se déconnecter** s’affiche lorsque l’utilisateur est déjà connecté. Notez que le contrôleur de compte est défini dans le **package NuGet Microsoft.Identity.Web.UI**, dans la zone nommée **MicrosoftIdentity**

```html
<ul class="navbar-nav">
  @if (User.Identity.IsAuthenticated)
  {
    <li class="nav-item">
        <span class="navbar-text text-dark">Hello @User.Identity.Name!</span>
    </li>
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="MicrosoftIdentity" asp-controller="Account" asp-action="SignOut">Sign out</a>
    </li>
  }
  else
  {
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="MicrosoftIdentity" asp-controller="Account" asp-action="SignIn">Sign in</a>
    </li>
  }
</ul>
```

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Dans ASP.NET MVC, le bouton de déconnexion est exposé dans `Views\Shared\_LoginPartial.cshtml`. Cela s’affiche uniquement lorsqu’il y a un compte authentifié. Autrement dit, cela s’affiche lorsque l’utilisateur s’est connecté précédemment.

```html
@if (Request.IsAuthenticated)
{
 // Code omitted code for clarity
}
else
{
    <ul class="nav navbar-nav navbar-right">
        <li>@Html.ActionLink("Sign in", "SignIn", "Account", routeValues: null, htmlAttributes: new { id = "loginLink" })</li>
    </ul>
}
```

# <a name="java"></a>[Java](#tab/java)

Dans notre guide de démarrage rapide Java, le bouton de déconnexion se trouve dans le fichier [main/resources/templates/index.html](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/master/msal-java-webapp-sample/src/main/resources/templates/index.html).

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>HomePage</title>
</head>
<body>
<h3>Home Page</h3>

<form action="/msal4jsample/secure/aad">
    <input type="submit" value="Login">
</form>

</body>
</html>
```

# <a name="python"></a>[Python](#tab/python)

Dans le guide de démarrage rapide Python, il n’y a aucun bouton de connexion. Le code-behind invite automatiquement l’utilisateur à se connecter lorsqu’il atteint la racine de l’application web. Consultez [app.py#L14-L18](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/0.1.0/app.py#L14-L18).

```Python
@app.route("/")
def index():
    if not session.get("user"):
        return redirect(url_for("login"))
    return render_template('index.html', user=session["user"])
```

---

### <a name="signin-action-of-the-controller"></a>Action `SignIn` du contrôleur

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Dans ASP.NET, sélectionner le bouton **Connexion** de l’application web déclenche l’action `SignIn` sur le contrôleur `AccountController`. Dans les versions précédentes des modèles ASP.NET Core, le contrôleur `Account` était incorporé avec l’application web. Ce n’est plus le cas, car le contrôleur fait maintenant partie du package NuGet **Microsoft.Identity.Web.UI**. Pour plus d’informations, consultez [AccountController.cs](https://github.com/AzureAD/microsoft-identity-web/blob/master/src/Microsoft.Identity.Web.UI/Areas/MicrosoftIdentity/Controllers/AccountController.cs).

Ce contrôleur gère également les applications Azure AD B2C.

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Dans ASP.NET, la déconnexion est déclenchée par la méthode `SignOut()` sur un contrôleur (par exemple, [AccountController.cs#L16-L23](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/a2da310539aa613b77da1f9e1c17585311ab22b7/WebApp/Controllers/AccountController.cs#L16-L23)). Cette méthode ne fait pas partie de l’infrastructure ASP.NET (contrairement à ASP.NET Core). Envoi d’une demande de connexion OpenID après avoir proposé un URI de redirection.

```csharp
public void SignIn()
{
    // Send an OpenID Connect sign-in request.
    if (!Request.IsAuthenticated)
    {
        HttpContext.GetOwinContext().Authentication.Challenge(new AuthenticationProperties { RedirectUri = "/" }, OpenIdConnectAuthenticationDefaults.AuthenticationType);
    }
}
```

# <a name="java"></a>[Java](#tab/java)

Dans Java, la déconnexion est gérée en appelant directement le point de terminaison `logout` de la plateforme d’identités Microsoft et en fournissant la valeur `post_logout_redirect_uri`. Pour plus d’informations, consultez [AuthPageController.java#L30-L48](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/AuthPageController.java#L30-L48).

```Java
@Controller
public class AuthPageController {

    @Autowired
    AuthHelper authHelper;

    @RequestMapping("/msal4jsample")
    public String homepage(){
        return "index";
    }

    @RequestMapping("/msal4jsample/secure/aad")
    public ModelAndView securePage(HttpServletRequest httpRequest) throws ParseException {
        ModelAndView mav = new ModelAndView("auth_page");

        setAccountInfo(mav, httpRequest);

        return mav;
    }

    // More code omitted for simplicity
```

# <a name="python"></a>[Python](#tab/python)

Contrairement à d’autres plateformes, MSAL Python s’occupe de permettre à l’utilisateur de se connecter sur la page de connexion. Consultez [app.py#L20-L28](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/e03be352914bfbd58be0d4170eba1fb7a4951d84/app.py#L20-L28).

```Python
@app.route("/login")
def login():
    session["state"] = str(uuid.uuid4())
    auth_url = _build_msal_app().get_authorization_request_url(
        app_config.SCOPE,  # Technically we can use an empty list [] to just sign in
                           # Here we choose to also collect user consent up front
        state=session["state"],
        redirect_uri=url_for("authorized", _external=True))
    return "<a href='%s'>Login with Microsoft Identity</a>" % auth_url
```

La méthode `_build_msal_app()` est définie dans [app.py#L81-L88](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/e03be352914bfbd58be0d4170eba1fb7a4951d84/app.py#L81-L88) comme suit :

```Python
def _load_cache():
    cache = msal.SerializableTokenCache()
    if session.get("token_cache"):
        cache.deserialize(session["token_cache"])
    return cache

def _save_cache(cache):
    if cache.has_state_changed:
        session["token_cache"] = cache.serialize()

def _build_msal_app(cache=None):
    return msal.ConfidentialClientApplication(
        app_config.CLIENT_ID, authority=app_config.AUTHORITY,
        client_credential=app_config.CLIENT_SECRET, token_cache=cache)

def _get_token_from_cache(scope=None):
    cache = _load_cache()  # This web app maintains one cache per session
    cca = _build_msal_app(cache)
    accounts = cca.get_accounts()
    if accounts:  # So all accounts belong to the current signed-in user
        result = cca.acquire_token_silent(scope, account=accounts[0])
        _save_cache(cache)
        return result

```

---

Une fois que l’utilisateur est connecté à votre application, vous voudrez leur donner la possibilité de se déconnecter.

## <a name="sign-out"></a>Se déconnecter

La déconnexion d’une application web ne consiste pas seulement à supprimer les informations sur le compte connecté de l’état de l’application web.
L’application web doit également rediriger l’utilisateur vers le point de terminaison `logout` de la plateforme d’identités Microsoft pour suivre la procédure de déconnexion.

Lorsque votre application web redirige l’utilisateur vers le point de terminaison `logout`, ce dernier efface la session de l’utilisateur dans le navigateur. Si votre application n’a pas atteint le point de terminaison `logout`, l’utilisateur se réauthentifie auprès de votre application sans entrer à nouveau ses informations d’identification. La raison en est qu’il disposera d’une session d’authentification unique valide avec le point de terminaison de la plateforme d’identité Microsoft.

Pour en savoir plus, consultez la section [Envoi d’une demande de déconnexion](v2-protocols-oidc.md#send-a-sign-out-request) dans la documentation conceptuelle de la [plateforme d’identités Microsoft et du protocole OpenID Connect](v2-protocols-oidc.md).

### <a name="application-registration"></a>Inscription de l’application

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Lors de l’inscription de l’application, vous inscrivez une URI Post-déconnexion. Dans notre didacticiel, vous avez inscrit `https://localhost:44321/signout-oidc` dans le champ **URL de déconnexion** de la section **Paramètres avancés** sur la page **Authentification**. Pour plus d’informations, voir [Inscrire l’application web](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-1-MyOrg#register-the-webapp-app-webapp).

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Lors de l’inscription de l’application, vous inscrivez une URI Post-déconnexion. Dans notre didacticiel, vous avez inscrit `https://localhost:44308/Account/EndSession` dans le champ **URL de déconnexion** de la section **Paramètres avancés** sur la page **Authentification**. Pour plus d’informations, voir [Inscrire l’application web](https://github.com/Azure-Samples/active-directory-dotnet-web-single-sign-out#register-the-service-app-webapp-distributedsignout-dotnet).

# <a name="java"></a>[Java](#tab/java)

Lors de l’inscription de l’application, vous inscrivez une URI Post-déconnexion. Dans notre didacticiel, vous avez inscrit `http://localhost:8080/msal4jsample/sign_out` dans le champ **URL de déconnexion** de la section **Paramètres avancés** sur la page **Authentification**.

# <a name="python"></a>[Python](#tab/python)

Au cours de l’inscription de l’application, vous n’avez pas besoin d’inscrire d’URL de déconnexion supplémentaire. L’application est rappelée sur son URL principale.

---

### <a name="sign-out-button"></a>Bouton Déconnexion

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Dans ASP.NET, sélectionner le bouton **Déconnexion** de l’application web déclenche l’action `SignOut` sur le contrôleur `AccountController` (voir ci-dessous)

```html
<ul class="navbar-nav">
  @if (User.Identity.IsAuthenticated)
  {
    <li class="nav-item">
        <span class="navbar-text text-dark">Hello @User.Identity.Name!</span>
    </li>
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="MicrosoftIdentity" asp-controller="Account" asp-action="SignOut">Sign out</a>
    </li>
  }
  else
  {
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="MicrosoftIdentity" asp-controller="Account" asp-action="SignIn">Sign in</a>
    </li>
  }
</ul>
```

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Dans ASP.NET MVC, le bouton de déconnexion est exposé dans `Views\Shared\_LoginPartial.cshtml`. Cela s’affiche uniquement lorsqu’il y a un compte authentifié. Autrement dit, cela s’affiche lorsque l’utilisateur s’est connecté précédemment.

```html
@if (Request.IsAuthenticated)
{
    <text>
        <ul class="nav navbar-nav navbar-right">
            <li class="navbar-text">
                Hello, @User.Identity.Name!
            </li>
            <li>
                @Html.ActionLink("Sign out", "SignOut", "Account")
            </li>
        </ul>
    </text>
}
else
{
    <ul class="nav navbar-nav navbar-right">
        <li>@Html.ActionLink("Sign in", "SignIn", "Account", routeValues: null, htmlAttributes: new { id = "loginLink" })</li>
    </ul>
}
```

# <a name="java"></a>[Java](#tab/java)

Dans notre guide de démarrage rapide Java, le bouton Déconnexion se trouve dans le fichier main/resources/templates/auth_page.html.

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<body>

<form action="/msal4jsample/sign_out">
    <input type="submit" value="Sign out">
</form>
...
```

# <a name="python"></a>[Python](#tab/python)

Dans le guide de démarrage rapide Python, le bouton de déconnexion se trouve dans le fichier [templates/index.html#L10](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/e03be352914bfbd58be0d4170eba1fb7a4951d84/templates/index.html#L10).

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
</head>
<body>
    <h1>Microsoft Identity Python web app</h1>
    Welcome {{ user.get("name") }}!
    <li><a href='/graphcall'>Call Microsoft Graph API</a></li>
    <li><a href="/logout">Logout</a></li>
</body>
</html>
```

---

### <a name="signout-action-of-the-controller"></a>Action `SignOut` du contrôleur

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Dans les versions précédentes des modèles ASP.NET Core, le contrôleur `Account` était incorporé avec l’application web. Ce n’est plus le cas, car le contrôleur fait maintenant partie du package NuGet **Microsoft.Identity.Web.UI**. Pour plus d’informations, consultez [AccountController.cs](https://github.com/AzureAD/microsoft-identity-web/blob/master/src/Microsoft.Identity.Web.UI/Areas/MicrosoftIdentity/Controllers/AccountController.cs).

- Définir un URI de redirection OpenID sur `/Account/SignedOut` afin que le contrôleur soit rappelé quand Azure AD a effectué la déconnexion.
- Appeler `Signout()`, ce qui permet à l’intergiciel OpenID Connect de contacter le point de terminaison `logout` de la plateforme d’identités Microsoft. Ensuite, le point de terminaison :

  - efface le cookie de session du navigateur.
  - Rappelle l’URL de déconnexion. Par défaut, l’URL de déconnexion affiche la page de vue déconnectée [SignedOut.cshtml.cs](https://github.com/AzureAD/microsoft-identity-web/blob/master/src/Microsoft.Identity.Web.UI/Areas/MicrosoftIdentity/Pages/Account/SignedOut.cshtml.cs). Cette page est également fournie dans le cadre de Microsoft.Identity.Web.

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Dans ASP.NET, la déconnexion est déclenchée par la méthode `SignOut()` sur un contrôleur (par exemple, [AccountController.cs#L25-L31](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/a2da310539aa613b77da1f9e1c17585311ab22b7/WebApp/Controllers/AccountController.cs#L25-L31)). Cette méthode ne fait pas partie de l’infrastructure ASP.NET, contrairement à ASP.NET Core. Elle effectue les actions suivantes :

- Elle envoie une demande de déconnexion OpenID.
- Elle efface le cache.
- Effectue une redirection vers la page souhaitée.

```csharp
/// <summary>
/// Send an OpenID Connect sign-out request.
/// </summary>
public void SignOut()
{
 HttpContext.GetOwinContext()
            .Authentication
            .SignOut(CookieAuthenticationDefaults.AuthenticationType);
 Response.Redirect("/");
}
```

# <a name="java"></a>[Java](#tab/java)

Dans Java, la déconnexion est gérée en appelant directement le point de terminaison `logout` de la plateforme d’identités Microsoft et en fournissant la valeur `post_logout_redirect_uri`. Pour plus d’informations, consultez [AuthPageController.java#L50-L60](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/AuthPageController.java#L50-L60).

```Java
@RequestMapping("/msal4jsample/sign_out")
    public void signOut(HttpServletRequest httpRequest, HttpServletResponse response) throws IOException {

        httpRequest.getSession().invalidate();

        String endSessionEndpoint = "https://login.microsoftonline.com/common/oauth2/v2.0/logout";

        String redirectUrl = "http://localhost:8080/msal4jsample/";
        response.sendRedirect(endSessionEndpoint + "?post_logout_redirect_uri=" +
                URLEncoder.encode(redirectUrl, "UTF-8"));
    }
```

# <a name="python"></a>[Python](#tab/python)

Le code qui déconnecte l’utilisateur se trouve dans [app.py#L46-L52](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/48637475ed7d7733795ebeac55c5d58663714c60/app.py#L47-L48).

```Python
@app.route("/logout")
def logout():
    session.clear()  # Wipe out the user and the token cache from the session
    return redirect(  # Also need to log out from the Microsoft Identity platform
        "https://login.microsoftonline.com/common/oauth2/v2.0/logout"
        "?post_logout_redirect_uri=" + url_for("index", _external=True))
```

---

### <a name="intercepting-the-call-to-the-logout-endpoint"></a>Interception de l’appel à au point de terminaison `logout`

L’URI Post-déconnexion permet aux applications de participer à la déconnexion globale.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

L’intergiciel OpenID Connect ASP.NET Core permet à votre application d’intercepter l’appel au point de terminaison `logout` de la plateforme d’identités Microsoft en fournissant un événement OpenID Connect nommé `OnRedirectToIdentityProviderForSignOut`. Cela est géré automatiquement par Microsoft.Identity.Web (qui efface les comptes dans le cas où votre application web appelle des API web)

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Dans ASP.NET, vous déléguez à l’intergiciel (middleware) l’exécution de la déconnexion, en effaçant le cookie de session :

```csharp
public class AccountController : Controller
{
 ...
 public void EndSession()
 {
  Request.GetOwinContext().Authentication.SignOut();
  Request.GetOwinContext().Authentication.SignOut(Microsoft.AspNet.Identity.DefaultAuthenticationTypes.ApplicationCookie);
  this.HttpContext.GetOwinContext().Authentication.SignOut(CookieAuthenticationDefaults.AuthenticationType);
 }
}
```

# <a name="java"></a>[Java](#tab/java)

Dans le guide de démarrage rapide Java, l’URI de redirection post-déconnexion affiche uniquement la page index.html.

# <a name="python"></a>[Python](#tab/python)

Dans notre guide de démarrage rapide Python, l’URI de redirection post-déconnexion affiche uniquement la page index.html.

---

## <a name="protocol"></a>Protocol

Pour en savoir plus sur la déconnexion, voir la documentation du protocole, disponible dans [Open ID Connect](./v2-protocols-oidc.md).

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Passer en production](scenario-web-app-sign-user-production.md).