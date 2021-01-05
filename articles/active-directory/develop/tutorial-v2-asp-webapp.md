---
title: "Tutoriel : Créer une application web ASP.NET qui utilise la Plateforme d'identités Microsoft pour l'authentification | Azure"
titleSuffix: Microsoft identity platform
description: Dans ce tutoriel, vous allez créer une application web ASP.NET qui utilisera la Plateforme d'identités Microsoft et l'intergiciel OWIN pour activer la connexion de l'utilisateur.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 08/28/2019
ms.author: jmprieur
ms.custom: devx-track-csharp, aaddev, identityplatformtop40
ms.openlocfilehash: 4dca1fa0e823b482044d35f98412187af2fa72ed
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507759"
---
# <a name="tutorial-add-sign-in-to-microsoft-to-an-aspnet-web-app"></a>Tutoriel : Ajouter la connexion à Microsoft à une application web ASP.NET

Dans ce tutoriel, vous allez créer une application web ASP.NET MVC qui connecte les utilisateurs à l’aide du middleware OWIN (Open Web Interface for .NET) et de la plateforme d’identités Microsoft.

Quand vous aurez suivi ce guide, votre application sera en mesure d’accepter les connexions de comptes personnels à partir des mentions « j’aime » d’outlook.com et de live.com. De plus, les comptes professionnels et scolaires de toute entreprise ou organisation ayant intégré la plateforme d’identités Microsoft pourra se connecter à votre application.

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Créer un projet d'*application web ASP.NET* dans Visual Studio
> * Ajouter les composants de l'intergiciel Open Web Interface (OWIN) pour .NET
> * Ajouter du code pour prendre en charge la connexion et la déconnexion des utilisateurs
> * Inscrire l'application sur le portail Azure
> * Test de l'application

## <a name="prerequisites"></a>Prérequis

* Avoir installé [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) avec la charge de travail **Développement ASP.NET et web**

## <a name="how-the-sample-app-generated-by-this-guide-works"></a>Fonctionnement de l’exemple d’application de ce guide

![Fonctionnement de l’exemple d’application généré par ce tutoriel](media/active-directory-develop-guidedsetup-aspnetwebapp-intro/aspnetbrowsergeneral.svg)

L’exemple d’application que vous créez est basé sur un scénario dans lequel vous utilisez le navigateur pour accéder à un site web ASP.NET qui invite un utilisateur à s’authentifier par le biais d’un bouton de connexion. Dans ce scénario, la majorité du travail pour afficher la page web se passe côté serveur.

## <a name="libraries"></a>Bibliothèques

Ce guide utilise les bibliothèques suivantes :

|Bibliothèque|Description|
|---|---|
|[Microsoft.Owin.Security.OpenIdConnect](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/)|Intergiciel qui permet à une application d’utiliser OpenIDConnect pour l’authentification.|
|[Microsoft.Owin.Security.Cookies](https://www.nuget.org/packages/Microsoft.Owin.Security.Cookies)|Intergiciel (middleware) qui permet à une application de maintenir une session utilisateur à l’aide de cookies|
|[Microsoft.Owin.Host.SystemWeb](https://www.nuget.org/packages/Microsoft.Owin.Host.SystemWeb)|Intergiciel (middleware) qui permet aux applications basées sur OWIN de s’exécuter sur IIS (Internet Information Services) à l’aide du pipeline de requête ASP.NET|

## <a name="set-up-your-project"></a>Configuration de votre projet

Cette section montre comment installer et configurer le pipeline d’authentification par le biais du middleware OWIN sur un projet ASP.NET à l’aide d’OpenID Connect.

> Vous préférez télécharger le projet Visual Studio de cet exemple ? [Téléchargez un projet](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-DotNet/archive/master.zip), puis passez à l’étape [Inscrire votre application](#register-your-application) pour configurer l’exemple de code avant de l’exécuter.

### <a name="create-your-aspnet-project"></a>Créer votre projet ASP.NET

1. Dans Visual Studio : Accédez à **Fichier** > **Nouveau** > **Projet**.
2. Sous **Visual C#\Web**, sélectionnez **Application web ASP.NET (.NET Framework)** .
3. Donnez un nom à votre application et sélectionnez **OK**.
4. Sélectionnez **Vide**, puis cochez la case pour ajouter des références **MVC**.

## <a name="add-authentication-components"></a>Ajouter les composants d’authentification

1. Dans Visual Studio : Accédez à **Outils** > **Gestionnaire de package NuGet** > **Console du Gestionnaire de package**.
2. Ajoutez *les packages NuGet de l’intergiciel OWIN* en saisissant la commande suivante dans la fenêtre Console du gestionnaire de package :

    ```powershell
    Install-Package Microsoft.Owin.Security.OpenIdConnect
    Install-Package Microsoft.Owin.Security.Cookies
    Install-Package Microsoft.Owin.Host.SystemWeb
    ```

### <a name="about-these-libraries"></a>À propos de ces bibliothèques
Ces bibliothèques activent l’authentification unique (SSO) avec OpenID Connect à l’aide de l’authentification basée sur les cookies. Une fois que l’authentification est terminée et que le jeton qui représente l’utilisateur est envoyé à votre application, l’intergiciel OWIN crée un cookie de session. Le navigateur utilise ensuite ce cookie pour les requêtes ultérieures afin d’éviter à l’utilisateur de retaper le mot de passe, et aucune vérification supplémentaire n’est nécessaire.

## <a name="configure-the-authentication-pipeline"></a>Configurer le pipeline d’authentification

Les étapes suivantes permettent de créer une classe Startup de middleware OWIN pour configurer l’authentification OpenID Connect. Cette classe est exécutée automatiquement au démarrage de votre processus IIS.

> [!TIP]
> Si votre projet n’a pas de fichier `Startup.cs` dans le dossier racine :
> 1. Cliquez avec le bouton droit sur le dossier racine du projet, puis sélectionnez **Ajouter** > **Nouvel élément** > **Classe de démarrage OWIN**.<br/>
> 2. Nommez-la **Startup.cs**.
>
>> Vérifiez que la classe sélectionnée est une classe de démarrage OWIN et non une classe C# standard. Pour ce faire, vérifiez que vous voyez [assembly: OwinStartup(typeof({NameSpace}.Startup))] au-dessus de l’espace de noms.

1. Ajoutez les références *OWIN* et *Microsoft.IdentityModel* à Startup.cs:

    ```csharp
    using Microsoft.Owin;
    using Owin;
    using Microsoft.IdentityModel.Protocols.OpenIdConnect;
    using Microsoft.IdentityModel.Tokens;
    using Microsoft.Owin.Security;
    using Microsoft.Owin.Security.Cookies;
    using Microsoft.Owin.Security.OpenIdConnect;
    using Microsoft.Owin.Security.Notifications;
    ```

2. Remplacez la classe de démarrage par le code suivant :

    ```csharp
    public class Startup
    {
        // The Client ID is used by the application to uniquely identify itself to Microsoft identity platform.
        string clientId = System.Configuration.ConfigurationManager.AppSettings["ClientId"];

        // RedirectUri is the URL where the user will be redirected to after they sign in.
        string redirectUri = System.Configuration.ConfigurationManager.AppSettings["RedirectUri"];

        // Tenant is the tenant ID (e.g. contoso.onmicrosoft.com, or 'common' for multi-tenant)
        static string tenant = System.Configuration.ConfigurationManager.AppSettings["Tenant"];

        // Authority is the URL for authority, composed by Microsoft identity platform endpoint and the tenant name (e.g. https://login.microsoftonline.com/contoso.onmicrosoft.com/v2.0)
        string authority = String.Format(System.Globalization.CultureInfo.InvariantCulture, System.Configuration.ConfigurationManager.AppSettings["Authority"], tenant);

        /// <summary>
        /// Configure OWIN to use OpenIdConnect
        /// </summary>
        /// <param name="app"></param>
        public void Configuration(IAppBuilder app)
        {
            app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

            app.UseCookieAuthentication(new CookieAuthenticationOptions());
            app.UseOpenIdConnectAuthentication(
                new OpenIdConnectAuthenticationOptions
                {
                    // Sets the ClientId, authority, RedirectUri as obtained from web.config
                    ClientId = clientId,
                    Authority = authority,
                    RedirectUri = redirectUri,
                    // PostLogoutRedirectUri is the page that users will be redirected to after sign-out. In this case, it is using the home page
                    PostLogoutRedirectUri = redirectUri,
                    Scope = OpenIdConnectScope.OpenIdProfile,
                    // ResponseType is set to request the id_token - which contains basic information about the signed-in user
                    ResponseType = OpenIdConnectResponseType.IdToken,
                    // ValidateIssuer set to false to allow personal and work accounts from any organization to sign in to your application
                    // To only allow users from a single organizations, set ValidateIssuer to true and 'tenant' setting in web.config to the tenant name
                    // To allow users from only a list of specific organizations, set ValidateIssuer to true and use ValidIssuers parameter
                    TokenValidationParameters = new TokenValidationParameters()
                    {
                        ValidateIssuer = false // This is a simplification
                    },
                    // OpenIdConnectAuthenticationNotifications configures OWIN to send notification of failed authentications to OnAuthenticationFailed method
                    Notifications = new OpenIdConnectAuthenticationNotifications
                    {
                        AuthenticationFailed = OnAuthenticationFailed
                    }
                }
            );
        }

        /// <summary>
        /// Handle failed authentication requests by redirecting the user to the home page with an error in the query string
        /// </summary>
        /// <param name="context"></param>
        /// <returns></returns>
        private Task OnAuthenticationFailed(AuthenticationFailedNotification<OpenIdConnectMessage, OpenIdConnectAuthenticationOptions> context)
        {
            context.HandleResponse();
            context.Response.Redirect("/?errormessage=" + context.Exception.Message);
            return Task.FromResult(0);
        }
    }
    ```

> [!NOTE]
> Le paramètre `ValidateIssuer = false` est une simplification destinée aux seules fins de ce guide de démarrage rapide. Dans les applications réelles, vous devez valider l’émetteur.
> Pour découvrir comment procéder, consultez les exemples.

### <a name="more-information"></a>Informations complémentaires

Les paramètres que vous fournissez dans *OpenIDConnectAuthenticationOptions* serviront de coordonnées pour que l’application puisse communiquer avec la plateforme d’identités Microsoft. Étant donné que le middleware OpenID Connect utilise des cookies en arrière-plan, vous devez également configurer l’authentification des cookies comme le montre le code précédent. La valeur *ValidateIssuer* indique à OpenIdConnect de ne pas restreindre l’accès à une organisation spécifique.

## <a name="add-a-controller-to-handle-sign-in-and-sign-out-requests"></a>Ajouter un contrôleur pour gérer les demandes de connexion et de déconnexion

Pour créer un contrôleur afin d’exposer les méthodes de connexion et de déconnexion, effectuez les étapes suivantes :

1.  Cliquez avec le bouton droit sur le dossier **Contrôleurs**, puis sélectionnez **Ajouter** > **Contrôleur**.
2.  Sélectionnez **Contrôleur MVC (version .NET) – vide**.
3.  Sélectionnez **Ajouter**.
4.  Nommez-le **HomeController**, puis sélectionnez **Ajouter**.
5.  Ajoutez des références OWIN à la classe :

    ```csharp
    using Microsoft.Owin.Security;
    using Microsoft.Owin.Security.Cookies;
    using Microsoft.Owin.Security.OpenIdConnect;
    ```

6. Ajoutez les deux méthodes suivantes pour gérer la connexion à votre contrôleur et la déconnexion de celui-ci en lançant une demande d’authentification :

    ```csharp
    /// <summary>
    /// Send an OpenID Connect sign-in request.
    /// Alternatively, you can just decorate the SignIn method with the [Authorize] attribute
    /// </summary>
    public void SignIn()
    {
        if (!Request.IsAuthenticated)
        {
            HttpContext.GetOwinContext().Authentication.Challenge(
                new AuthenticationProperties{ RedirectUri = "/" },
                OpenIdConnectAuthenticationDefaults.AuthenticationType);
        }
    }

    /// <summary>
    /// Send an OpenID Connect sign-out request.
    /// </summary>
    public void SignOut()
    {
        HttpContext.GetOwinContext().Authentication.SignOut(
                OpenIdConnectAuthenticationDefaults.AuthenticationType,
                CookieAuthenticationDefaults.AuthenticationType);
    }
    ```

## <a name="create-the-apps-home-page-for-user-sign-in"></a>Créer la page d’accueil de l’application pour la connexion de l’utilisateur

Dans Visual Studio, créez une vue pour ajouter le bouton de connexion et afficher les informations de l’utilisateur après authentification :

1.  Faites un clic droit sur le dossier **Vues\Accueil** et sélectionnez **Ajouter une vue**.
2.  Nommez la nouvelle vue **Index**.
3.  Ajoutez le code HTML suivant, qui inclut le bouton de connexion, au fichier :

    ```html
    <html>
    <head>
        <meta name="viewport" content="width=device-width" />
        <title>Sign in with Microsoft Guide</title>
    </head>
    <body>
    @if (!Request.IsAuthenticated)
    {
        <!-- If the user is not authenticated, display the sign-in button -->
        <a href="@Url.Action("SignIn", "Home")" style="text-decoration: none;">
            <svg xmlns="http://www.w3.org/2000/svg" xml:space="preserve" width="300px" height="50px" viewBox="0 0 3278 522" class="SignInButton">
            <style type="text/css">.fil0:hover {fill: #4B4B4B;} .fnt0 {font-size: 260px;font-family: 'Segoe UI Semibold', 'Segoe UI'; text-decoration: none;}</style>
            <rect class="fil0" x="2" y="2" width="3174" height="517" fill="black" />
            <rect x="150" y="129" width="122" height="122" fill="#F35325" />
            <rect x="284" y="129" width="122" height="122" fill="#81BC06" />
            <rect x="150" y="263" width="122" height="122" fill="#05A6F0" />
            <rect x="284" y="263" width="122" height="122" fill="#FFBA08" />
            <text x="470" y="357" fill="white" class="fnt0">Sign in with Microsoft</text>
            </svg>
        </a>
    }
    else
    {
        <span><br/>Hello @System.Security.Claims.ClaimsPrincipal.Current.FindFirst("name").Value;</span>
        <br /><br />
        @Html.ActionLink("See Your Claims", "Index", "Claims")
        <br /><br />
        @Html.ActionLink("Sign out", "SignOut", "Home")
    }
    @if (!string.IsNullOrWhiteSpace(Request.QueryString["errormessage"]))
    {
        <div style="background-color:red;color:white;font-weight: bold;">Error: @Request.QueryString["errormessage"]</div>
    }
    </body>
    </html>
    ```

### <a name="more-information"></a>Informations complémentaires
Cette page ajoute un bouton de connexion au format SVG avec un arrière-plan noir :<br/>![Bouton Se connecter avec Microsoft](media/active-directory-develop-guidedsetup-aspnetwebapp-use/aspnetsigninbuttonsample.png)<br/> Pour d’autres boutons de connexion, accédez aux [Conseils pour la personnalisation](./howto-add-branding-in-azure-ad-apps.md "Instructions de personnalisation").

## <a name="add-a-controller-to-display-users-claims"></a>Ajouter un contrôleur pour afficher les revendications de l’utilisateur
Ce contrôleur démontre les utilisations de l’attribut `[Authorize]` pour protéger un contrôleur. Cet attribut limite l’accès au contrôleur en autorisant uniquement les utilisateurs authentifiés. Le code suivant tire parti de l’attribut pour afficher les revendications de l’utilisateur qui ont été récupérées dans le cadre de la connexion :

1.  Cliquez avec le bouton droit sur le dossier **Contrôleurs**, puis sélectionnez **Ajouter** > **Contrôleur**.
2.  Sélectionnez **Contrôleur MVC {version} – vide**.
3.  Sélectionnez **Ajouter**.
4.  Nommez-le **ClaimsController**.
5.  Remplacez le code de votre classe de contrôleur par le code suivant. Cette opération ajoute l’attribut `[Authorize]` à la classe :

    ```csharp
    [Authorize]
    public class ClaimsController : Controller
    {
        /// <summary>
        /// Add user's claims to viewbag
        /// </summary>
        /// <returns></returns>
        public ActionResult Index()
        {
            var userClaims = User.Identity as System.Security.Claims.ClaimsIdentity;

            //You get the user's first and last name below:
            ViewBag.Name = userClaims?.FindFirst("name")?.Value;

            // The 'preferred_username' claim can be used for showing the username
            ViewBag.Username = userClaims?.FindFirst("preferred_username")?.Value;

            // The subject/ NameIdentifier claim can be used to uniquely identify the user across the web
            ViewBag.Subject = userClaims?.FindFirst(System.Security.Claims.ClaimTypes.NameIdentifier)?.Value;

            // TenantId is the unique Tenant Id - which represents an organization in Azure AD
            ViewBag.TenantId = userClaims?.FindFirst("http://schemas.microsoft.com/identity/claims/tenantid")?.Value;

            return View();
        }
    }
    ```

### <a name="more-information"></a>Informations complémentaires
En raison de l’utilisation de l’attribut `[Authorize]`, toutes les méthodes de ce contrôleur ne peuvent être exécutées que si l’utilisateur est authentifié. Si l’utilisateur n’est pas authentifié et qu’il tente d’accéder au contrôleur, OWIN lance une demande d’authentification et oblige l’utilisateur à s’authentifier. Le code précédent examine la liste des revendications pour des attributs utilisateur spécifiques inclus dans le jeton d’ID de l’utilisateur. Ces attributs incluent le nom complet et le nom d’utilisateur de l’utilisateur, ainsi que le sujet d’identificateur d’utilisateur global. Il contient également l’*ID client*, qui représente l’identifiant de l’organisation de l’utilisateur.

## <a name="create-a-view-to-display-the-users-claims"></a>Créer une vue pour afficher les revendications de l’utilisateur

Dans Visual Studio, créez une vue pour afficher les revendications de l’utilisateur dans une page web :

1.  Cliquez avec le bouton droit sur le dossier **Views\Claims**, puis sélectionnez **Ajouter une vue**.
2.  Nommez la nouvelle vue **Index**.
3.  Ajoutez le code HTML suivant au fichier :

    ```html
    <html>
    <head>
        <meta name="viewport" content="width=device-width" />
        <title>Sign in with Microsoft Sample</title>
        <link href="@Url.Content("~/Content/bootstrap.min.css")" rel="stylesheet" type="text/css" />
    </head>
    <body style="padding:50px">
        <h3>Main Claims:</h3>
        <table class="table table-striped table-bordered table-hover">
            <tr><td>Name</td><td>@ViewBag.Name</td></tr>
            <tr><td>Username</td><td>@ViewBag.Username</td></tr>
            <tr><td>Subject</td><td>@ViewBag.Subject</td></tr>
            <tr><td>TenantId</td><td>@ViewBag.TenantId</td></tr>
        </table>
        <br />
        <h3>All Claims:</h3>
        <table class="table table-striped table-bordered table-hover table-condensed">
        @foreach (var claim in System.Security.Claims.ClaimsPrincipal.Current.Claims)
        {
            <tr><td>@claim.Type</td><td>@claim.Value</td></tr>
        }
        </table>
        <br />
        <br />
        @Html.ActionLink("Sign out", "SignOut", "Home", null, new { @class = "btn btn-primary" })
    </body>
    </html>
    ```

## <a name="register-your-application"></a>Inscrivez votre application

Pour inscrire votre application et ajouter les informations d’inscription de l’application à votre solution, deux options s’offrent à vous :

### <a name="option-1-express-mode"></a>Option 1 : Mode Express

Pour inscrire rapidement votre application, effectuez les étapes suivantes :

1. Accédez au nouveau volet [Portail Azure - Inscriptions d’applications](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/AspNetWebAppQuickstartPage/sourceType/docs).
1. Entrez un nom pour votre application, puis sélectionnez **Inscrire**.
1. Suivez les instructions permettant de télécharger et de configurer automatiquement votre nouvelle application en un seul clic.

### <a name="option-2-advanced-mode"></a>Option n°2 : Mode Avancé

Pour inscrire votre application et ajouter manuellement les informations d’inscription de l’application à votre solution, procédez comme suit :

1. Ouvrez Visual Studio, puis :
   1. Dans l’Explorateur de solutions, sélectionnez le projet et examinez la fenêtre Propriétés (si aucune fenêtre Propriétés ne s’affiche, appuyez sur F4).
   1. Définissez l’option SSL activé sur `True`.
   1. Cliquez avec le bouton droit sur le projet dans Visual Studio et sélectionnez **Propriétés**, puis l’onglet **Web**. Dans la section **Serveurs**, remplacez l’**URL du projet** par l’**URL SSL**.
   1. Copiez l’URL SSL. Vous allez ajouter cette URL à la liste des URI de redirection dans la liste des URI de redirection du portail d’inscription à l’étape suivante.<br/><br/>![Propriétés d’un projet](media/active-directory-develop-guidedsetup-aspnetwebapp-configure/vsprojectproperties.png)<br />
1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire dans lequel vous voulez inscrire une application.
1. Recherchez et sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Inscriptions d’applications** > **Nouvelle inscription**.
1. Entrez un **nom** pour votre application (par exemple, `ASPNET-Tutorial`). Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
1. Ajoutez l’URL SSL que vous avez copiée à partir de Visual Studio à l’étape 1 (par exemple, `https://localhost:44368/`) dans **URI de redirection**.
1. Sélectionnez **Inscription**.
1. Sous **Gérer**, sélectionnez **Authentification**.
1. Dans la section **Octroi implicite**, sélectionnez **Jetons d’ID**, puis sélectionnez **Enregistrer**.
1. Ajoutez le code suivant dans le fichier web.config situé dans le dossier racine, dans la section `configuration\appSettings` :

    ```xml
    <add key="ClientId" value="Enter_the_Application_Id_here" />
    <add key="redirectUri" value="Enter_the_Redirect_URL_here" />
    <add key="Tenant" value="common" />
    <add key="Authority" value="https://login.microsoftonline.com/{0}/v2.0" />
    ```

1. Remplacez `ClientId` par l’ID de l’application que vous venez d’inscrire.
1. Remplacez `redirectUri` par l’URL SSL de votre projet.

## <a name="test-your-code"></a>Test de votre code

Pour tester votre application dans Visual Studio, appuyez sur F5 afin d’exécuter votre projet. Le navigateur s’ouvre sur l’emplacement http://<span></span>localhost:{port} et le bouton **Se connecter avec Microsoft** s’affiche. Sélectionnez le bouton pour démarrer le processus de connexion.

Quand vous êtes prêt à exécuter votre test, utilisez un compte Azure AD (compte professionnel ou scolaire) ou un compte Microsoft personnel (<span>live.</span>com ou <span>outlook.</span>com) pour vous connecter.

![Bouton Se connecter avec Microsoft affiché sur la page d'ouverture de session du navigateur](media/active-directory-develop-guidedsetup-aspnetwebapp-test/aspnetbrowsersignin.png)
<br/><br/>
![Connexion à votre compte Microsoft](media/active-directory-develop-guidedsetup-aspnetwebapp-test/aspnetbrowsersignin2.png)

#### <a name="permissions-and-consent-in-the-microsoft-identity-platform-endpoint"></a>Autorisations et consentement dans le point de terminaison de la plateforme d’identités Microsoft

Les applications qui s’intègrent à la plateforme d’identité Microsoft suivent un modèle d’autorisation permettant aux utilisateurs et aux administrateurs de contrôler l’accès aux données. Une fois qu’un utilisateur s’est authentifié auprès de la plateforme d’identités Microsoft pour accéder à cette application, il est invité à donner son consentement pour les autorisations demandées par l’application (« Afficher votre profil de base » et « Conserver l’accès aux données auxquelles vous lui avez donné accès »). Après avoir accepté ces autorisations, l’utilisateur poursuit en accédant aux résultats de l’application. Toutefois, une page **Consentement de l’administrateur nécessaire** peut s’afficher à l’utilisateur si l’une des conditions suivantes se présente :

- Le développeur d’applications ajoute toutes les autorisations supplémentaires qui exigent le **consentement de l’administrateur**.
- Ou le locataire est configuré (dans **Applications d’entreprise -> Paramètres utilisateur**), ce qui empêche les utilisateurs de donner leur consentement pour les applications qui accèdent à des données de l’entreprise en leur nom.

Pour plus d’informations, consultez [Autorisations et consentement dans le point de terminaison de la plateforme d’identités Microsoft](./v2-permissions-and-consent.md).

### <a name="view-application-results"></a>Afficher les résultats de l’application

Une fois connecté, l’utilisateur est redirigé vers la page d’accueil de votre site web. La page d’accueil correspond à l’URL HTTPS qui est spécifiée dans les informations d’inscription de votre application, dans le portail d’inscription des applications de Microsoft. Cette page inclut le message de bienvenue *« Hello \<user> »* , un lien pour se déconnecter et un lien pour afficher les revendications de l’utilisateur. Le lien des revendications de l’utilisateur établit une connexion au contrôleur des revendications que vous avez créé précédemment.

### <a name="view-the-users-claims"></a>Afficher les revendications de l’utilisateur

Pour voir les revendications de l’utilisateur, sélectionnez le lien permettant d’accéder à la vue de contrôleur qui est disponible uniquement pour les utilisateurs authentifiés.

#### <a name="view-the-claims-results"></a>Afficher les résultats des revendications

Une fois que vous avez accédé à la vue de contrôleur, vous devez voir un tableau qui contient les propriétés de base de l’utilisateur :

|Propriété |Valeur |Description |
|---|---|---|
|**Nom** |Nom complet de l’utilisateur | Prénom et nom de l’utilisateur
|**Nom d’utilisateur** |utilisateur<span>@domain.com</span> | Nom d’utilisateur utilisé pour identifier l’utilisateur|
|**Subject** |Objet |Chaîne qui identifie de manière unique l’utilisateur sur le web|
|**Tenant ID** |Guid | **GUID** qui représente de manière unique l’organisation Azure AD de l’utilisateur|

En outre, vous devriez voir un tableau de toutes les revendications qui se trouvent dans la demande d’authentification. Pour plus d’informations, consultez la [liste des revendications figurant dans un jeton d’ID](./id-tokens.md).

### <a name="test-access-to-a-method-that-has-an-authorize-attribute-optional"></a>Tester l’accès à une méthode disposant d’un attribut Authorize (facultatif)

Pour tester l’accès en tant qu’utilisateur anonyme à un contrôleur qui est protégé par l’attribut `Authorize`, effectuez les étapes suivantes :

1. Sélectionnez le lien de déconnexion de l’utilisateur, puis terminez le processus de déconnexion.
2. Dans le navigateur, tapez http://<span></span>localhost:{port}/claims pour accéder à votre contrôleur qui est protégé par l’attribut `Authorize`.

#### <a name="expected-results-after-access-to-a-protected-controller"></a>Résultats attendus après l’accès à un contrôleur protégé

Vous êtes invité à vous authentifier pour utiliser la vue de contrôleur protégé.

## <a name="advanced-options"></a>Options avancées

### <a name="protect-your-entire-website"></a>Protéger l’ensemble de votre site web

Pour protéger tout votre site web, dans le fichier **Global.asax**, ajoutez l’attribut `AuthorizeAttribute` au filtre `GlobalFilters` dans la méthode `Application_Start` :

```csharp
GlobalFilters.Filters.Add(new AuthorizeAttribute());
```

### <a name="restrict-who-can-sign-in-to-your-application"></a>Restreindre les utilisateurs autorisés à se connecter à votre application

Par défaut, quand vous générez l’application créée par ce guide, celle-ci accepte les connexions de comptes personnels (dont outlook.com et live.com, entre autres) ainsi que de comptes professionnels et scolaires de n’importe quelle entreprise ou organisation intégrée à la plateforme d’identités Microsoft. Il s’agit d’une option recommandée pour les applications SaaS.

Pour restreindre les utilisateurs qui peuvent se connecter à votre application, plusieurs options sont disponibles.

#### <a name="option-1-restrict-users-from-only-one-organizations-active-directory-instance-to-sign-in-to-your-application-single-tenant"></a>Option 1 : Limiter la connexion à votre application aux utilisateurs de l’instance Active Directory d’une seule organisation (un seul locataire)

Cette option est souvent utilisée pour les *applications métier* : Si vous souhaitez que votre application accepte les connexions uniquement à partir des comptes qui appartiennent à une instance Azure AD spécifique (y compris les *comptes invités* de cette instance), suivez les étapes suivantes :

1. Dans le fichier web.config, remplacez la valeur `Common` du paramètre `Tenant` par le nom de locataire de l’organisation ; par exemple, `contoso.onmicrosoft.com`.
2. Dans la classe [OWIN Startup](#configure-the-authentication-pipeline), définissez l’argument `ValidateIssuer` sur `true`.

#### <a name="option-2-restrict-access-to-users-in-a-specific-list-of-organizations"></a>Option n°2 : Restreindre l’accès aux utilisateurs figurant dans une liste spécifique d’organisations

Vous pouvez limiter l’accès de connexion aux comptes d’utilisateurs d’une organisation Azure AD figurant dans la liste des organisations autorisées :
1. Dans la classe [OWIN Startup](#configure-the-authentication-pipeline), définissez l’argument `ValidateIssuer` sur `true`.
2. Définissez la valeur du paramètre `ValidIssuers` sur la liste des organisations autorisées.

#### <a name="option-3-use-a-custom-method-to-validate-issuers"></a>Option 3 : Utiliser une méthode personnalisée pour valider les émetteurs

Vous pouvez implémenter une méthode personnalisée pour valider les émetteurs à l’aide du paramètre **IssuerValidator**. Pour plus d’informations sur l’utilisation de ce paramètre, consultez la rubrique relative à la [classe TokenValidationParameters](/dotnet/api/microsoft.identitymodel.tokens.tokenvalidationparameters).

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Apprenez à appeler des API web protégées à partir d'applications web avec la Plateforme d'identités Microsoft :

> [!div class="nextstepaction"]
> [Applications web appelant des API web](scenario-web-app-sign-user-overview.md)
