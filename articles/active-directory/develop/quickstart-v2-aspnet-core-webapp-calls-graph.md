---
title: 'Démarrage rapide : Application web ASP.NET Core qui connecte des utilisateurs et appelle Microsoft Graph | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce guide de démarrage rapide, vous découvrez comment une application tire parti de Microsoft.Identity.Web pour implémenter la connexion Microsoft dans une application web ASP.NET Core à l’aide d’OpenID Connect et appeler Microsoft Graph.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 12/10/2020
ms.author: jmprieur
ms.custom: devx-track-csharp, aaddev, scenarios:getting-started, languages:aspnet-core
ms.openlocfilehash: 95a0dbd61e44e9c5b1f2854f8b3215aa4ef9403c
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97438318"
---
# <a name="quickstart-aspnet-core-web-app-that-signs-in-users-and-calls-microsoft-graph-on-their-behalf"></a>Démarrage rapide : Application web ASP.NET Core qui connecte des utilisateurs et appelle Microsoft Graph en leur nom

Dans ce guide de démarrage rapide, vous téléchargez et exécutez un exemple de code qui montre comment une application web ASP.NET Core peut connecter des utilisateurs à partir de n’importe quelle organisation Azure Active Directory (Azure AD) et appeler Microsoft Graph.  

Consultez [Fonctionnement de l’exemple](#how-the-sample-works) pour obtenir une illustration.

> [!div renderon="docs"]
> ## <a name="prerequisites"></a>Prérequis
>
> * [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) ou [Visual Studio Code](https://code.visualstudio.com/)
> * [SDK .NET Core 3.1+](https://dotnet.microsoft.com/download)
>
> ## <a name="register-and-download-the-quickstart-app"></a>Inscrire et télécharger l’application de démarrage rapide
> Vous disposez de deux options pour démarrer votre application de démarrage rapide :
> * [Express] [Option 1 : Inscrire et configurer automatiquement votre application, puis télécharger votre exemple de code](#option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample)
> * [Manuel] [Option 2 : Inscrire et configurer manuellement vos application et exemple de code](#option-2-register-and-manually-configure-your-application-and-code-sample)
>
> ### <a name="option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>Option 1 : Inscrire et configurer automatiquement votre application, puis télécharger votre exemple de code
>
> 1. Accédez au [portail Azure - Inscriptions d’applications](https://aka.ms/aspnetcore-webapp-calls-graph-quickstart-v2).
> 1. Entrez un nom pour votre application, puis sélectionnez **Inscrire**.
> 1. Suivez les instructions pour télécharger et configurer automatiquement votre nouvelle application pour vous en un seul clic.
>
> ### <a name="option-2-register-and-manually-configure-your-application-and-code-sample"></a>Option n°2 : Inscrire et configurer manuellement vos application et exemple de code
>
> #### <a name="step-1-register-your-application"></a>Étape 1 : Inscrivez votre application
> Pour inscrire votre application et ajouter manuellement les informations d’inscription de l’application à votre solution, procédez comme suit :
>
> 1. Connectez-vous au [portail Azure](https://portal.azure.com).
> 1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire dans lequel vous voulez inscrire une application.
> 1. Recherchez et sélectionnez **Azure Active Directory**.
> 1. Sous **Gérer**, sélectionnez **Inscriptions d’applications** > **Nouvelle inscription**.
> 1. Entrez un **nom** pour votre application (par exemple, `AspNetCoreWebAppCallsGraph-Quickstart`). Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
> 1. Entrez l’**URI de redirection** `https://localhost:44321/signin-oidc`
> 1. Sélectionnez **Inscription**.
> 1. Sous **Gérer**, sélectionnez **Authentification**.
> 1. Entrez l’**URL de déconnexion** `https://localhost:44321/signout-oidc`
> 1. Sélectionnez **Enregistrer**.
> 1. Sous **Gérer**, sélectionnez **Certificats et secrets** > **Nouveau secret client**.
> 1. Entrez une **description**, telle que `clientsecret1`.
> 1. Sélectionnez **Dans 1 an** pour l’expiration du secret.
> 1. Sélectionnez **Ajouter** et enregistrez immédiatement la **valeur** du secret, en vue de l’utiliser dans une étape ultérieure. Cette valeur de secret ne sera *plus jamais affichée* et ne sera récupérable par aucun autre moyen. Enregistrez-la dans un emplacement sécurisé, comme vous le feriez pour n’importe quel mot de passe.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>Étape 1 : Configurer votre application dans le portail Azure
> Pour que l’exemple de code de ce guide de démarrage rapide fonctionne, vous devez ajouter des URL de réponse telles que `https://localhost:44321/signin-oidc` et ajouter l’URL de déconnexion `https://localhost:44321/signout-oidc`.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Apporter cette modification pour moi]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Déjà configuré](media/quickstart-v2-aspnet-webapp/green-check.png) Votre application est configurée avec ces attributs.

#### <a name="step-2-download-the-aspnet-core-project"></a>Étape 2 : Télécharger le projet ASP.NET Core

> [!div renderon="docs"]
> [Télécharger la solution ASP.NET Core](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/archive/aspnetcore3-1-callsgraph.zip)

> [!div renderon="portal" class="sxs-lookup"]
> Exécutez le projet.

> [!div renderon="portal" class="sxs-lookup" id="autoupdate" class="nextstepaction"]
> [Téléchargez l’exemple de code](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/archive/aspnetcore3-1-callsgraph.zip).

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Étape 3 : Votre application est configurée et prête à être exécutée
> Nous avons configuré votre projet avec les valeurs des propriétés de votre application et il est prêt à être exécuté.
> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`
> [!div renderon="docs"]
> #### <a name="step-3-configure-your-aspnet-core-project"></a>Étape 3 : Configurer votre projet ASP.NET Core
> 1. Extrayez l’archive .zip dans un dossier local proche de la racine de votre lecteur. Par exemple, dans *C:\Azure-Samples*.
> 1. Ouvrez la solution dans Visual Studio 2019.
> 1. Ouvrez le fichier *appsettings.json* pour modifier ce qui suit :
>
>    ```json
>    "ClientId": "Enter_the_Application_Id_here",
>    "TenantId": "common",
>    "clientSecret": "Enter_the_Client_Secret_Here"
>    ```
>
>    - Remplacez `Enter_the_Application_Id_here` par l’**ID d’application (client)** de l’application que vous avez inscrite dans le portail Azure. Vous trouverez l’**ID d’application (client)** dans la page **Vue d’ensemble** de l’application.
>    - Remplacez `common` par l’un des éléments suivants :
>       - Si votre application prend en charge les **Comptes dans cet annuaire organisationnel uniquement**, remplacez cette valeur par l’**ID de l’annuaire (locataire)** (un GUID) ou par le **nom du locataire** (par exemple, `contoso.onmicrosoft.com`). L’**ID de l’annuaire (locataire)** se trouve dans la page **Vue d’ensemble** de l’application.
>       - Si votre application prend en charge **Comptes dans un annuaire organisationnel**, remplacez cette valeur par `organizations`
>       - Si votre application prend en charge **Tous les utilisateurs de compte Microsoft**, conservez la valeur `common`.
>    - Remplacez `Enter_the_Client_Secret_Here` par le **secret client** que vous avez créé et enregistré dans une étape précédente.
> 
> Pour ce guide de démarrage rapide, ne modifiez pas les autres valeurs du fichier *appsettings.json*.
>
> #### <a name="step-4-build-and-run-the-application"></a>Étape 4 : Générer et exécuter l’application
>
> Générez puis exécutez l’application dans Visual Studio en sélectionnant le menu **Débogage** > **Démarrer le débogage**, ou en appuyant sur la touche `F5`.
>
> Vous êtes invité à entrer vos informations d’identification, puis à donner votre consentement pour les autorisations dont a besoin votre application. Sélectionnez **Accepter** dans l’invite de consentement.
>
> :::image type="content" source="media/quickstart-v2-aspnet-core-webapp-calls-graph/webapp-01-consent.png" alt-text="Boîte de dialogue de consentement montrant les autorisations que l’application demande à l’utilisateur":::
>
> Après avoir donné son consentement aux autorisations demandées, l’application indique que vous avez réussi à vous connecter avec vos informations d’identification Azure Active Directory et vous verrez votre adresse e-mail dans la section « Résultat API » de la page. Elle a été extraite à l’aide de Microsoft Graph.
>
> :::image type="content" source="media/quickstart-v2-aspnet-core-webapp-calls-graph/webapp-02-signed-in.png" alt-text="Navigateur web montrant l’application web en cours d’exécution et l’utilisateur connecté":::

## <a name="about-the-code"></a>À propos du code

Cette section fournit une vue d’ensemble du code requis pour la connexion des utilisateurs et l’appel de l’API Microsoft Graph en leur nom. Cette vue d’ensemble peut être utile pour comprendre comment le code fonctionne, les principaux arguments et également si vous souhaitez ajouter une connexion à une application ASP.NET Core existante et appeler Microsoft Graph. Elle utilise [Microsoft.Identity.Web](microsoft-identity-web.md), qui est un wrapper autour de [MSAL.NET](msal-overview.md).

### <a name="how-the-sample-works"></a>Fonctionnement de l’exemple
![Fonctionnement de l’exemple d’application généré par ce guide de démarrage rapide](media/quickstart-v2-aspnet-core-webapp-calls-graph/aspnetcorewebapp-intro.svg)

### <a name="startup-class"></a>Classe de démarrage

Le middleware (intergiciel) *Microsoft.AspNetCore.Authentication* utilise une classe `Startup` qui s’exécute lors de l’initialisation du processus d’hébergement :

```csharp
  // Get the scopes from the configuration (appsettings.json)
  var initialScopes = Configuration.GetValue<string>("DownstreamApi:Scopes")?.Split(' ');

  public void ConfigureServices(IServiceCollection services)
  {
      // Add sign-in with Microsoft
      services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
        .AddMicrosoftIdentityWebApp(Configuration.GetSection("AzureAd"))

            // Add the possibility of acquiring a token to call a protected web API
            .EnableTokenAcquisitionToCallDownstreamApi(initialScopes)

                // Enables controllers and pages to get GraphServiceClient by dependency injection
                // And use an in memory token cache
                .AddMicrosoftGraph(Configuration.GetSection("DownstreamApi"))
                .AddInMemoryTokenCaches();

      services.AddControllersWithViews(options =>
      {
          var policy = new AuthorizationPolicyBuilder()
              .RequireAuthenticatedUser()
              .Build();
          options.Filters.Add(new AuthorizeFilter(policy));
      });

      // Enables a UI and controller for sign in and sign out.
      services.AddRazorPages()
          .AddMicrosoftIdentityUI();
  }
```

La méthode `AddAuthentication()` configure le service pour ajouter une authentification basée sur les cookies, qui est utilisée dans les scénarios de navigateur, et pour définir la demande d’authentification sur OpenID Connect.

La ligne contenant `.AddMicrosoftIdentityWebApp` ajoute à votre application l’authentification auprès de la plateforme d’identités Microsoft. Elle est fournie par [Microsoft.Identity.Web](microsoft-identity-web.md). Elle est ensuite configurée pour se connecter à l’aide du point de terminaison de la plateforme d’identités Microsoft en fonction des informations contenues dans la section `AzureAD` du fichier de configuration *appsettings.json* :

| Clé *appsettings.json* | Description                                                                                                                                                          |
|------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ClientId`             | **ID d’application (client)** de l’application inscrite dans le portail Azure.                                                                                       |
| `Instance`             | Point de terminaison du service d’émission de jeton de sécurité (STS) pour l’utilisateur à authentifier. Cette valeur est généralement `https://login.microsoftonline.com/`, ce qui indique le cloud public Azure. |
| `TenantId`             | Nom de votre locataire ou de son ID (GUID), ou *common* pour connecter les utilisateurs avec des comptes professionnels ou scolaires, ou des comptes personnels Microsoft.                             |

La méthode `EnableTokenAcquisitionToCallDownstreamApi` permet à votre application d’acquérir un jeton pour appeler des API web protégées. `AddMicrosoftGraph` permet à vos contrôleurs ou pages Razor de tirer parti directement de `GraphServiceClient` (par injection de dépendances) et les méthodes `AddInMemoryTokenCaches` permettent à votre application de tirer parti d’un cache de jeton.

La méthode `Configure()` contient deux méthodes importantes, `app.UseAuthentication()` et `app.UseAuthorization()`, qui activent leurs fonctionnalités nommées : Également dans la méthode `Configure()`, vous devez inscrire les routes de Microsoft Identity Web avec au moins un appel à `endpoints.MapControllerRoute()` ou un appel à `endpoints.MapControllers()`.

```csharp
app.UseAuthentication();
app.UseAuthorization();

app.UseEndpoints(endpoints =>
{

    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
    endpoints.MapRazorPages();
});

// endpoints.MapControllers(); // REQUIRED if MapControllerRoute() isn't called.
```

### <a name="protect-a-controller-or-a-controllers-method"></a>Protéger un contrôleur ou la méthode d’un contrôleur

Vous pouvez protéger un contrôleur ou ses méthodes en appliquant l’attribut `[Authorize]` à la classe du contrôleur ou à une ou plusieurs de ses méthodes. Cet attribut `[Authorize]` limite l’accès en autorisant uniquement les utilisateurs authentifiés. Si l’utilisateur n’est pas déjà authentifié, une demande d’authentification peut être démarrée pour accéder au contrôleur. Dans ce guide de démarrage rapide, les étendues sont lues à partir du fichier de configuration :

```CSharp
[AuthorizeForScopes(ScopeKeySection = "DownstreamApi:Scopes")]
public async Task<IActionResult> Index()
{
    var user = await _graphServiceClient.Me.Request().GetAsync();
    ViewData["ApiResult"] = user.DisplayName;

    return View();
}
 ```

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Le dépôt GitHub qui contient l’exemple de code ASP.NET Core référencé dans ce guide de démarrage rapide inclut des instructions et d’autres exemples de code qui vous montrent comment :

- Ajouter l’authentification à une nouvelle application web ASP.NET Core
- Appeler Microsoft Graph, d’autres API Microsoft ou vos propres API web
- Ajouter une autorisation
- Connecter des utilisateurs dans des clouds nationaux ou avec des identités sociales

> [!div class="nextstepaction"]
> [Tutoriels concernant les applications web ASP.NET Core sur GitHub](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/)
