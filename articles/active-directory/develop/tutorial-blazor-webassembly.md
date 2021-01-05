---
title: Tutoriel - Connecter des utilisateurs et appeler une API protégée à partir d’une application Blazor WebAssembly
titleSuffix: Microsoft identity platform
description: Dans ce tutoriel, connectez des utilisateurs et appelez une API protégée à l’aide de la plateforme d’identités Microsoft dans une application Blazor WebAssembly (WASM).
author: knicholasa
ms.author: nichola
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.date: 10/16/2020
ms.openlocfilehash: 09e922ffddcce732d9213eb91026561528c0728a
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96169135"
---
# <a name="tutorial-sign-in-users-and-call-a-protected-api-from-a-blazor-webassembly-app"></a>Tutoriel : Connecter des utilisateurs et appeler une API protégée à partir d’une application Blazor WebAssembly

Dans ce tutoriel, vous créez une application Blazor WebAssembly qui connecte les utilisateurs et récupère les données depuis Microsoft Graph, par l’intermédiaire de la plateforme d’identités Microsoft et en inscrivant votre application dans Azure Active Directory (Azure AD).

Nous disposons également d’un [tutoriel pour Blazor Server](tutorial-blazor-server.md). 

Dans ce tutoriel :

> [!div class="checklist"]
>
> * Créer une application Blazor WebAssembly configurée qui permet d’utiliser Azure Active Directory (Azure AD) pour l’[authentification et l’autorisation](authentication-vs-authorization.md) à l’aide de la plateforme d’identités Microsoft
> * Récupérer des données à partir d’une API web protégée, ici [Microsoft Graph](/graph/overview)

## <a name="prerequisites"></a>Prérequis

* [Kit SDK .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1)
* Un locataire Azure AD dans lequel vous pouvez inscrire une application. Si vous n’avez pas accès à un locataire Azure AD, vous pouvez en obtenir un en vous inscrivant au [Programme pour les développeurs Microsoft 365](https://developer.microsoft.com/microsoft-365/dev-program) ou en créant un [compte gratuit Azure](https://azure.microsoft.com/free).

## <a name="register-the-app-in-the-azure-portal"></a>Inscrire l'application sur le portail Azure

Chaque application qui utilise Azure Active Directory (Azure AD) pour l’authentification doit être inscrite avec Azure AD. Suivez les instructions de la page [Inscrire une application](quickstart-register-app.md) avec les précisions suivantes :

- Pour les **Types de comptes pris en charge**, sélectionnez **Comptes dans cet annuaire organisationnel**.
- Laissez le champ déroulant **URI de redirection** défini sur **Web** et entrez `https://localhost:5001/authentication/login-callback`. Le port par défaut pour une application s’exécutant sur Kestrel est 5001. Si l’application est disponible sur un autre port, spécifiez ce numéro de port au lieu de `5001`.

Une fois inscrite, dans **Authentification** > **Octroi implicite**, cochez les cases pour **Jetons d’accès** et **Jetons d’ID**, puis sélectionnez le bouton **Enregistrer**.

## <a name="create-the-app-using-the-net-core-cli"></a>Créer l’application à l’aide de l’interface CLI .NET Core

Pour créer l’application, vous avez besoin des tout derniers modèles Blazor. Vous pouvez les installer pour l’interface CLI .NET Core au moyen de la commande suivante :

```dotnetcli
dotnet new --install Microsoft.AspNetCore.Components.WebAssembly.Templates::3.2.1
```

Exécutez ensuite la commande suivante pour créer l’application. Remplacez les espaces réservés dans la commande par les informations appropriées extraites de la page de présentation de votre application et exécutez la commande dans un interpréteur de commandes. L’emplacement de sortie spécifié avec l’option `-o|--output` crée un dossier de projet s’il n’existe pas et devient partie intégrante du nom de l’application.

```dotnetcli
dotnet new blazorwasm2 --auth SingleOrg --calls-graph -o {APP NAME} --client-id "{CLIENT ID}" --tenant-id "{TENANT ID}"
```

| Espace réservé   | Nom du portail Azure       | Exemple                                |
| ------------- | ----------------------- | -------------------------------------- |
| `{APP NAME}`  | &mdash;                 | `BlazorWASMSample`                         |
| `{CLIENT ID}` | ID d’application (client) | `41451fa7-0000-0000-0000-69eff5a761fd` |
| `{TENANT ID}` | ID de l’annuaire (locataire)   | `e86c78e2-0000-0000-0000-918e0565a45e` |

## <a name="test-the-app"></a>Test de l'application

Vous pouvez maintenant générer et exécuter l’application. Lorsque vous exécutez cette application modèle, vous devez spécifier l’infrastructure à exécuter à l’aide de --Framework. Ce tutoriel utilise .NET Standard 2.1, mais le modèle prend également en charge d’autres frameworks.

```dotnetcli
dotnet run --framework netstandard2.1
```

Dans votre navigateur, accédez à `https://localhost:5001` et connectez-vous avec un compte d’utilisateur Azure AD pour voir l’application s’exécuter et connecter les utilisateurs à la plateforme d’identités Microsoft.

Les composants de ce modèle qui activent les connexions avec Azure AD en utilisant la plateforme d’identités Microsoft sont expliqués dans le [document ASP.NET de cette rubrique](/aspnet/core/blazor/security/webassembly/standalone-with-azure-active-directory#authentication-package).

## <a name="retrieving-data-from-microsoft-graph"></a>Récupération de données à partir de Microsoft Graph

[Microsoft Graph](/graph/overview) propose un éventail d’API qui permettent d’accéder aux données Microsoft 365 des utilisateurs de votre locataire. En utilisant la plateforme d’identité Microsoft comme fournisseur d’identité pour votre application, vous avez plus facilement accès à ces informations puisque Microsoft Graph prend directement en charge les jetons émis par la plateforme d’identité Microsoft. Dans cette section, vous ajoutez du code qui peut afficher les e-mails de l’utilisateur connecté sur la page « Extraire les données » de l’application.

Avant de commencer, déconnectez-vous de votre application, car vous allez apporter des modifications aux autorisations requises et votre jeton actuel ne fonctionnera pas. Si vous ne l’avez pas encore fait, exécutez à nouveau votre application et sélectionnez **Se déconnecter** avant de mettre à jour le code ci-dessous.

À présent, vous allez mettre à jour l’inscription et le code de votre application pour extraire les e-mails d’un utilisateur et afficher les messages dans l’application.

Tout d’abord, ajoutez l’autorisation de l’API `Mail.Read` à l’inscription de l’application pour qu’Azure AD soit informé de la demande d’accès de l’application à la messagerie de ses utilisateurs.

1. Dans le portail Azure, sélectionnez votre application dans les **Inscriptions d’applications**.
1. Sous **Gérer**, sélectionnez **Autorisations de l’API**.
1. Sélectionnez **Ajouter une autorisation** > **Microsoft Graph**.
1. Sélectionnez **Autorisations déléguées**, puis recherchez et sélectionnez l’autorisation **mail.Read**.
1. Sélectionnez **Ajouter des autorisations**.

Ensuite, ajoutez ce qui suit au fichier *.csproj* de votre projet, dans **ItemGroup** de netstandard2.1. Cela vous permettra de créer l’objet HttpClient personnalisé à l’étape suivante.

```xml
<PackageReference Include="Microsoft.Extensions.Http" Version="3.1.7" />
```

Modifiez ensuite le code comme indiqué dans les étapes suivantes. Ces modifications ajouteront des [jetons d’accès](access-tokens.md) aux requêtes sortantes envoyées à l’API Microsoft Graph. Ce modèle est abordé plus en détail dans [Scénarios de sécurité supplémentaires Blazor WebAssembly ASP.NET Core](/aspnet/core/blazor/security/webassembly/additional-scenarios).

Tout d’abord, créez un fichier nommé *GraphAuthorizationMessageHandler.cs* avec le code suivant. Ce gestionnaire sera utilisé afin d’ajouter un jeton d’accès pour les étendues `User.Read` et `Mail.Read` aux requêtes sortantes à destination de l’API Microsoft Graph.

```csharp
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class GraphAPIAuthorizationMessageHandler : AuthorizationMessageHandler
{
    public GraphAPIAuthorizationMessageHandler(IAccessTokenProvider provider,
        NavigationManager navigationManager)
        : base(provider, navigationManager)
    {
        ConfigureHandler(
            authorizedUrls: new[] { "https://graph.microsoft.com" },
            scopes: new[] { "https://graph.microsoft.com/User.Read", "https://graph.microsoft.com/Mail.Read" });
    }
}
```

Ensuite, remplacez le contenu de la méthode `Main` dans *Program.cs* par le code suivant. Ce code utilise le nouvel élément `GraphAPIAuthorizationMessageHandler` et ajoute `User.Read` et `Mail.Read` en tant qu’étendues par défaut demandées par l’application lorsque l’utilisateur se connecte pour la première fois.

```csharp
var builder = WebAssemblyHostBuilder.CreateDefault(args);
builder.RootComponents.Add<App>("app");

builder.Services.AddScoped<GraphAPIAuthorizationMessageHandler>();

builder.Services.AddHttpClient("GraphAPI",
        client => client.BaseAddress = new Uri("https://graph.microsoft.com"))
    .AddHttpMessageHandler<GraphAPIAuthorizationMessageHandler>();

builder.Services.AddMsalAuthentication(options =>
{
    builder.Configuration.Bind("AzureAd", options.ProviderOptions.Authentication);
    options.ProviderOptions.DefaultAccessTokenScopes.Add("User.Read");
    options.ProviderOptions.DefaultAccessTokenScopes.Add("Mail.Read");
});

await builder.Build().RunAsync();
```

Finalement, remplacez le contenu de la page *FetchData.razor* par le code suivant. Ce code extrait les données de messagerie de l’utilisateur à partir de l’API Microsoft Graph et les affiche sous forme de liste. Dans `OnInitializedAsync`, le nouvel objet `HttpClient` qui utilise le jeton d’accès approprié est créé et utilisé pour effectuer la requête auprès de l’API Microsoft Graph.

```c#
@page "/fetchdata"
@using System.ComponentModel.DataAnnotations
@using System.Text.Json.Serialization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@using Microsoft.Extensions.Logging
@inject IAccessTokenProvider TokenProvider
@inject IHttpClientFactory ClientFactory
@inject IHttpClientFactory HttpClientFactory

<p>This component demonstrates fetching data from a service.</p>

@if (messages == null)
{
    <p><em>Loading...</em></p>
}
else
{
    <h1>Hello @userDisplayName !!!!</h1>
    <table class="table">
        <thead>
            <tr>
                <th>Subject</th>
                <th>Sender</th>
                <th>Received Time</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var mail in messages)
            {
                <tr>
                    <td>@mail.Subject</td>
                    <td>@mail.Sender</td>
                    <td>@mail.ReceivedTime</td>
                </tr>
            }
        </tbody>
    </table>
}

@code {

    private string userDisplayName;
    private List<MailMessage> messages = new List<MailMessage>();

    private HttpClient _httpClient;

    protected override async Task OnInitializedAsync()
    {
        _httpClient = HttpClientFactory.CreateClient("GraphAPI");
        try {
            var dataRequest = await _httpClient.GetAsync("https://graph.microsoft.com/beta/me");

            if (dataRequest.IsSuccessStatusCode)
            {
                var userData = System.Text.Json.JsonDocument.Parse(await dataRequest.Content.ReadAsStreamAsync());
                userDisplayName = userData.RootElement.GetProperty("displayName").GetString();
            }

            var mailRequest = await _httpClient.GetAsync("https://graph.microsoft.com/beta/me/messages?$select=subject,receivedDateTime,sender&$top=10");

            if (mailRequest.IsSuccessStatusCode)
            {
                var mailData = System.Text.Json.JsonDocument.Parse(await mailRequest.Content.ReadAsStreamAsync());
                var messagesArray = mailData.RootElement.GetProperty("value").EnumerateArray();

                foreach (var m in messagesArray)
                {
                    var message = new MailMessage();
                    message.Subject = m.GetProperty("subject").GetString();
                    message.Sender = m.GetProperty("sender").GetProperty("emailAddress").GetProperty("address").GetString();
                    message.ReceivedTime = m.GetProperty("receivedDateTime").GetDateTime();
                    messages.Add(message);
                }
            }
        }
        catch (AccessTokenNotAvailableException ex)
        {
            // Tokens are not valid - redirect the user to log in again
            ex.Redirect();
        }
    }

    public class MailMessage
    {
        public string Subject;
        public string Sender;
        public DateTime ReceivedTime;
    }
}
```

À présent, lancez de nouveau l’application. Vous remarquerez que vous êtes invité à donner l’accès à l’application pour lire votre courrier. C’est l’action attendue quand une application demande l’étendue `Mail.Read`.

Après avoir donné votre accord, accédez à la page « Extraire les données » pour lire quelques e-mails.

:::image type="content" source="./media/tutorial-blazor-webassembly/final-app.png" alt-text="Capture d’écran de l’application finale. Son titre indique « Bonjour Nicholas » et affiche une liste des e-mails appartenant à Nicholas.":::

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Meilleures pratiques et recommandations relatives à la plateforme d’identité Microsoft](./identity-platform-integration-checklist.md)