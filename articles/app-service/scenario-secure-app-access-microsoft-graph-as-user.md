---
title: 'Tutoriel : Accès d’une application web à Microsoft Graph en tant qu’utilisateur | Azure'
description: Dans ce tutoriel, vous allez apprendre à accéder à des données dans Microsoft Graph pour le compte d’un utilisateur connecté.
services: microsoft-graph, app-service-web
author: rwike77
manager: CelesteDG
ms.service: app-service-web
ms.topic: tutorial
ms.workload: identity
ms.date: 11/30/2020
ms.author: ryanwi
ms.reviewer: stsoneff
ms.custom: azureday1
ms.openlocfilehash: e07ec17a4e14f0099d82bd444f2ee8d37abe9908
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96434989"
---
# <a name="tutorial-access-microsoft-graph-from-a-secured-app-as-the-user"></a>Tutoriel : Accéder à Microsoft Graph à partir d’une application sécurisée en tant qu’utilisateur

Découvrez comment accéder à Microsoft Graph à partir d’une application web s’exécutant sur Azure App Service.

:::image type="content" alt-text="Diagramme illustrant l’accès à Microsoft Graph." source="./media/scenario-secure-app-access-microsoft-graph/web-app-access-graph.svg" border="false":::

Vous souhaitez ajouter l’accès à Microsoft Graph à partir de votre application web et effectuer une action en tant qu’utilisateur connecté. Cette section décrit comment accorder des autorisations déléguées à l’application web et comment récupérer les informations de profil de l’utilisateur connecté à partir d’Azure Active Directory (Azure AD).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Accorder des autorisations déléguées à une application web.
> * Appeler Microsoft Graph à partir d’une application web pour le compte d’un utilisateur connecté.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* Une application web s’exécutant sur Azure App Service et pour laquelle le [module d’authentification/autorisation App Service est activé](scenario-secure-app-authentication-app-service.md)

## <a name="grant-front-end-access-to-call-microsoft-graph"></a>Accorder un accès au front-end pour appeler Microsoft Graph

Maintenant que vous avez activé l’authentification et l’autorisation sur votre application web, celle-ci est inscrite auprès de la Plateforme d’identités Microsoft et secondée par une application Azure AD. Lors de cette étape, vous allez accorder à l’application web des autorisations afin qu’elle puisse accéder à Microsoft Graph pour le compte de l’utilisateur. (Techniquement, vous allez accorder à l’application Azure AD de l’application web les autorisations nécessaires pour accéder à l’application Azure AD de Microsoft Graph pour le compte de l’utilisateur.)

Dans le menu du [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory** ou recherchez et sélectionnez **Azure Active Directory** dans n’importe quelle page.

Sélectionnez **Inscriptions d’applications** > **Applications détenues** > **Afficher toutes les applications de cet annuaire**. Sélectionnez le nom de votre application web, puis **Autorisations des API**.

Sélectionnez **Ajouter une autorisation**, puis API Microsoft et Microsoft Graph.

Sélectionnez **Autorisations déléguées**, puis **Utilisateur.Lecture** dans la liste. Sélectionnez **Ajouter des autorisations**.

## <a name="configure-app-service-to-return-a-usable-access-token"></a>Configurer App Service pour renvoyer un jeton d’accès utilisable

L’application web dispose maintenant des autorisations nécessaires pour accéder à Microsoft Graph en tant qu’utilisateur connecté. Lors de cette étape, vous allez configurer l’authentification et l’autorisation App Service afin d’obtenir un jeton d’accès utilisable pour accéder à Microsoft Graph. Pour cette étape, vous avez besoin de l’ID d’application/client du service en aval (Microsoft Graph). L’ID d’application pour Microsoft Graph est *00000003-0000-0000-c000-000000000000*.

> [!IMPORTANT]
> Si vous ne configurez pas App Service pour retourner un jeton d’accès utilisable, vous recevez une erreur ```CompactToken parsing failed with error code: 80049217``` lorsque vous appelez des API Microsoft Graph dans votre code.

Accédez à [Azure Resource Explorer](https://resources.azure.com/) et utilisez l’arborescence de ressources pour localiser votre application web. L’URL de ressource doit ressembler à `https://resources.azure.com/subscriptions/subscription-id/resourceGroups/SecureWebApp/providers/Microsoft.Web/sites/SecureWebApp20200915115914`.

Azure Resource Explorer est désormais ouvert avec votre application web sélectionnée dans l’arborescence des ressources. En haut de la page, sélectionnez **Lecture/écriture** pour permettre la modification de vos ressources Azure.

Dans le navigateur de gauche, descendez dans la hiérarchie jusqu’à **conf** > **authsettings**.

Dans la vue **authsettings**, sélectionnez **Modifier**. Définissez ```additionalLoginParams``` sur la chaîne JSON suivante, en utilisant l’ID de client que vous avez copié.

```json
"additionalLoginParams": ["response_type=code id_token","resource=00000003-0000-0000-c000-000000000000"],
```

Enregistrez vos paramètres en sélectionnant **PUT**. La prise en compte de ce paramètre peut prendre plusieurs minutes. Votre application web est maintenant configurée pour accéder à Microsoft Graph avec un jeton d’accès approprié. Si ce n’est pas le cas, Microsoft Graph retourne une erreur indiquant que le format du jeton compact est incorrect.

## <a name="call-microsoft-graph-net"></a>Appeler Microsoft Graph (.NET)

Votre application web dispose désormais des autorisations nécessaires, et ajoute également l’ID client de Microsoft Graph aux paramètres de connexion. À l’aide de la [bibliothèque Microsoft.Identity.Web](https://github.com/AzureAD/microsoft-identity-web/), l’application web obtient un jeton d’accès pour l’authentification auprès de Microsoft Graph. Dans la version 1.2.0 et ultérieures, la bibliothèque Microsoft.Identity.Web s’intègre à et peut s’exécuter parallèlement au module d’authentification/d’autorisation App Service. Microsoft.Identity.Web détecte que l’application web est hébergée dans App Service et obtient le jeton d’accès à partir du module d’authentification/d’autorisation App Service. Le jeton d’accès est ensuite transmis aux requêtes authentifiées avec l’API Microsoft Graph.

Pour voir ce code dans un exemple d’application, consultez l’[exemple sur GitHub](https://github.com/Azure-Samples/ms-identity-easyauth-dotnet-storage-graphapi/tree/main/2-WebApp-graphapi-on-behalf).

> [!NOTE]
> La bibliothèque Microsoft.Identity.Web n’est pas nécessaire dans votre application web pour l’authentification/autorisation de base, ni pour authentifier les demandes auprès de Microsoft Graph. Il est possible d’[appeler de manière sécurisée les API en aval](tutorial-auth-aad.md#call-api-securely-from-server-code) avec uniquement le module d’authentification/autorisation App Service activé.
> 
> Toutefois, l’authentification/autorisation App Service est conçue pour les scénarios d’authentification de base. Pour les scénarios plus complexes (par exemple la gestion des revendications personnalisées), vous avez besoin de la bibliothèque Microsoft.Identity.Web ou de la [bibliothèque d’authentification Microsoft](../active-directory/develop/msal-overview.md). Les tâches d’installation et de configuration sont un peu plus longues au début, mais la bibliothèque Microsoft.Identity.Web peut s’exécuter en même temps que le module d’authentification/autorisation App Service. Plus tard, lorsque votre application web devra gérer des scénarios plus complexes, vous pourrez désactiver le module d’authentification/autorisation App Service, et Microsoft.Identity.Web fera déjà partie de votre application.

### <a name="install-client-library-packages"></a>Installer des packages de bibliothèque de client

Installez les packages NuGet [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) et [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph) dans votre projet à l’aide de l’interface de ligne de commande .NET Core ou de la console du gestionnaire de package dans Visual Studio.

# <a name="command-line"></a>[Ligne de commande](#tab/command-line)

Ouvrez une ligne de commande et basculez vers le répertoire qui contient votre fichier projet.

Exécutez les commandes d’installation.

```dotnetcli
dotnet add package Microsoft.Graph

dotnet add package Microsoft.Identity.Web
```

# <a name="package-manager"></a>[Gestionnaire de package](#tab/package-manager)

Ouvrez le projet/la solution dans Visual Studio, puis ouvrez la console à l’aide de la commande **Outils** > **Gestionnaire de package NuGet** > **Console du gestionnaire de package**.

Exécutez les commandes d’installation.
```powershell
Install-Package Microsoft.Graph

Install-Package Microsoft.Identity.Web
```

---

### <a name="startupcs"></a>Startup.cs

Dans le fichier *Startup.cs*, la méthode ```AddMicrosoftIdentityWebApp``` ajoute Microsoft.Identity.Web à votre application web. La méthode ```AddMicrosoftGraph``` ajoute la prise en charge de Microsoft Graph.

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Identity.Web;
using Microsoft.AspNetCore.Authentication.OpenIdConnect;

// Some code omitted for brevity.
public class Startup
{
    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
                .AddMicrosoftIdentityWebApp(Configuration.GetSection("AzureAd"))
                    .EnableTokenAcquisitionToCallDownstreamApi()
                        .AddMicrosoftGraph(Configuration.GetSection("Graph"))
                        .AddInMemoryTokenCaches();

        services.AddRazorPages();
    }
}

```

### <a name="appsettingsjson"></a>appsettings.json

*AzureAd* spécifie la configuration de la bibliothèque Microsoft.Identity.Web. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory** dans le menu du portail, puis **Inscriptions d’applications**. Sélectionnez l’inscription d’application créée lorsque vous avez activé le module d’authentification/autorisation App Service. (L’inscription d’application doit avoir le même nom que votre application web.) Vous trouverez l’ID de locataire et l’ID client dans la page de vue d’ensemble de l’inscription d’application. Le nom de domaine est mentionné dans la page de vue d’ensemble Azure AD de votre locataire.

*Graphe* spécifie le point de terminaison Microsoft Graph et les étendues initiales nécessaires à l’application.

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "fourthcoffeetest.onmicrosoft.com",
    "TenantId": "[tenant-id]",
    "ClientId": "[client-id]",
    // To call an API
    "ClientSecret": "[secret-from-portal]", // Not required by this scenario
    "CallbackPath": "/signin-oidc"
  },

  "Graph": {
    "BaseUrl": "https://graph.microsoft.com/v1.0",
    "Scopes": "user.read"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

### <a name="indexcshtmlcs"></a>Index.cshtml.cs

L’exemple suivant montre comment appeler Microsoft Graph en tant qu’utilisateur connecté et comment récupérer des informations utilisateur. L’objet ```GraphServiceClient``` est injecté dans le contrôleur et l’authentification a été configurée pour vous par la bibliothèque Microsoft.Identity.Web.

```csharp
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Graph;
using System.IO;
using Microsoft.Identity.Web;
using Microsoft.Extensions.Logging;

// Some code omitted for brevity.

[AuthorizeForScopes(Scopes = new[] { "user.read" })]
public class IndexModel : PageModel
{
    private readonly ILogger<IndexModel> _logger;
    private readonly GraphServiceClient _graphServiceClient;

    public IndexModel(ILogger<IndexModel> logger, GraphServiceClient graphServiceClient)
    {
        _logger = logger;
        _graphServiceClient = graphServiceClient;
    }

    public async Task OnGetAsync()
    {
        try
        {
            var user = await _graphServiceClient.Me.Request().GetAsync();
            ViewData["Me"] = user;
            ViewData["name"] = user.DisplayName;

            using (var photoStream = await _graphServiceClient.Me.Photo.Content.Request().GetAsync())
            {
                byte[] photoByte = ((MemoryStream)photoStream).ToArray();
                ViewData["photo"] = Convert.ToBase64String(photoByte);
            }
        }
        catch (Exception ex)
        {
            ViewData["photo"] = null;
        }
    }
}
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez terminé ce tutoriel et que vous n’avez plus besoin de l’application web ni des ressources associées, [nettoyez les ressources que vous avez créées](scenario-secure-app-clean-up-resources.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
>
> * Accorder des autorisations déléguées à une application web.
> * Appeler Microsoft Graph à partir d’une application web pour le compte d’un utilisateur connecté.

> [!div class="nextstepaction"]
> [Accès d’un service d’application à Microsoft Graph en tant qu’application](scenario-secure-app-access-microsoft-graph-as-app.md)