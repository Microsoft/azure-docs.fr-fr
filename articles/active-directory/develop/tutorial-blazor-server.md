---
title: Tutoriel - Créer une application Blazor Server qui utilise la plateforme d’identité Microsoft pour l’authentification | Azure
titleSuffix: Microsoft identity platform
description: Dans ce tutoriel, vous configurez l’authentification à l’aide de la plateforme d’identité Microsoft dans une application Blazor Server.
author: knicholasa
ms.author: nichola
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.date: 09/15/2020
ms.openlocfilehash: d499a0e7d3ca7933632b52d5edbd8094a29dbcaa
ms.sourcegitcommit: f311f112c9ca711d88a096bed43040fcdad24433
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94979893"
---
# <a name="tutorial-create-a-blazor-server-app-that-uses-the-microsoft-identity-platform-for-authentication"></a>Tutoriel : Créer une application Blazor Server qui utilise la plateforme d’identité Microsoft pour l’authentification

Dans ce tutoriel, vous créez une application Blazor Server qui connecte les utilisateurs et récupère les données depuis Microsoft Graph, par l’intermédiaire de la plateforme d’identités Microsoft et en inscrivant votre application dans Azure Active Directory (Azure AD).

Nous disposons également d’un tutoriel pour [Blazor WASM](tutorial-blazor-webassembly.md).

Dans ce tutoriel :

> [!div class="checklist"]
> * Créer une application Blazor Server configurée pour utiliser Azure Active Directory (Azure AD) pour l’authentification
> * Gérer à la fois l’authentification et l’autorisation avec Microsoft.Identity.Web
> * Récupérer des données à partir d’une API web protégée, Microsoft Graph

## <a name="prerequisites"></a>Prérequis

- [Kit SDK .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1)
- Un locataire Azure AD dans lequel vous pouvez inscrire une application. Si vous n’avez pas accès à un locataire Azure AD, vous pouvez en obtenir un en vous inscrivant au [Programme pour les développeurs Microsoft 365](https://developer.microsoft.com/microsoft-365/dev-program) ou en créant un [compte gratuit Azure](https://azure.microsoft.com/free).

## <a name="register-the-app-in-the-azure-portal"></a>Inscrire l'application sur le portail Azure

Chaque application qui utilise Azure Active Directory (Azure AD) pour l’authentification doit être inscrite avec Azure AD. Suivez les instructions de la section [Enregistrer une application](quickstart-register-app.md) avec les ajouts suivants :

- Pour les **Types de comptes pris en charge**, sélectionnez **Comptes dans cet annuaire organisationnel**.
- Laissez le champ déroulant **URI de redirection** défini sur **Web** et entrez `https://localhost:5001/signin-oidc`. Le port par défaut pour une application s’exécutant sur Kestrel est 5001. Si l’application est disponible sur un autre port, spécifiez ce numéro de port au lieu de `5001`.

Dans **Authentification** > **Octroi implicite**, cochez les cases pour **Jetons d’accès** et **Jetons d’ID**, puis sélectionnez le bouton **Enregistrer**.

Enfin, étant donné que l’application appelle une API protégée (dans ce cas, Microsoft Graph), elle a besoin d’une clé secrète client afin de vérifier son identité lorsqu’elle demande un jeton d’accès pour appeler cette API.

1. Toujours dans le cadre de l’inscription de l’application, sous **Gérer**, sélectionnez **Certificats & secrets**.
2. Créez une **clé secrète client** qui n’expire jamais.
3. Prenez note de la **Valeur** de la clé secrète, car vous allez l’utiliser à l’étape suivante. Vous ne pouvez plus y accéder une fois que vous avez quitté ce volet. Toutefois, vous pouvez la recréer si nécessaire.

## <a name="create-the-app-using-the-net-cli"></a>Créer l’application à l’aide de l’interface de ligne de commande .NET

Exécutez la commande suivante pour télécharger les modèles pour Microsoft.Identity.Web, que nous utiliserons dans ce tutoriel.

```dotnetcli
dotnet new --install Microsoft.Identity.Web.ProjectTemplates::0.4.0-preview
```

Exécutez ensuite la commande suivante pour créer l’application. Remplacez les espaces réservés dans la commande par les informations appropriées extraites de la page de présentation de votre application et exécutez la commande dans un interpréteur de commandes. L’emplacement de sortie spécifié avec l’option `-o|--output` crée un dossier de projet s’il n’existe pas et devient partie intégrante du nom de l’application.

```dotnetcli
dotnet new blazorserver2 --auth SingleOrg --calls-graph -o {APP NAME} --client-id "{CLIENT ID}" --tenant-id "{TENANT ID}"
```

| Espace réservé   | Nom du portail Azure       | Exemple                                |
| ------------- | ----------------------- | -------------------------------------- |
| `{APP NAME}`  | &mdash;                 | `BlazorSample`                         |
| `{CLIENT ID}` | ID d’application (client) | `41451fa7-0000-0000-0000-69eff5a761fd` |
| `{TENANT ID}` | ID de l’annuaire (locataire)   | `e86c78e2-0000-0000-0000-918e0565a45e` |

Accédez à présent à votre nouvelle application Blazor dans votre éditeur et ajoutez la clé secrète client au fichier *appsettings.json*, en remplaçant le texte « secret-from-app-registration ».

```json
"ClientSecret": "xkAlNiG70000000_UI~d.OS4Dl.-Cy-1m3",
```

## <a name="test-the-app"></a>Test de l'application

Vous pouvez maintenant générer et exécuter l’application. Lorsque vous exécutez cette application modèle, vous devez spécifier l’infrastructure à exécuter à l’aide de --Framework. Ce tutoriel utilise le SDK .NET Core 3.1.

```dotnetcli
dotnet run --framework netcoreapp3.1
```

Dans votre navigateur, accédez à `https://localhost:5001`et connectez-vous à l’aide d’un compte d’utilisateur Azure AD pour voir l’application en cours d’exécution.

## <a name="retrieving-data-from-microsoft-graph"></a>Récupération de données à partir de Microsoft Graph

[Microsoft Graph](/graph/overview) propose un éventail d’API qui permettent d’accéder aux données Microsoft 365 de vos utilisateurs. En utilisant la plateforme d’identité Microsoft comme fournisseur d’identité pour votre application, vous avez plus facilement accès à ces informations puisque Microsoft Graph prend directement en charge les jetons émis par la plateforme d’identité Microsoft. Dans cette section, vous ajoutez du code pour afficher les e-mails de l’utilisateur connecté sur la page « Extraire les données » de l’application.

Avant de commencer, déconnectez-vous de votre application, car vous allez apporter des modifications aux autorisations requises et votre jeton actuel ne fonctionnera pas. Si vous ne l’avez pas encore fait, exécutez à nouveau votre application et sélectionnez **Se déconnecter** avant de mettre à jour le code ci-dessous.

À présent, vous allez mettre à jour l’inscription et le code de votre application pour extraire l’e-mail d’un utilisateur et afficher les messages dans l’application. Pour ce faire, vous devez d’abord étendre les autorisations d’inscription d’application dans Azure AD pour permettre l’accès aux données de messagerie. Ensuite, ajoutez du code à l’application Blazor pour récupérer et afficher ces données dans l’une des pages.

1. Dans le portail Azure, sélectionnez votre application dans les **Inscriptions d’applications**.
1. Sous **Gérer**, sélectionnez **Autorisations de l’API**.
1. Sélectionnez **Ajouter une autorisation** > **Microsoft Graph**.
1. Sélectionnez **Autorisations déléguées**, puis recherchez et sélectionnez l’autorisation **mail.Read**.
1. Sélectionnez **Ajouter des autorisations**.

Dans le fichier *appsettings.json*, mettez à jour votre code afin qu’il récupère le jeton approprié avec les autorisations adéquates. Ajoutez « mail.read » après l’étendue « user.read » sous « DownstreamAPI ». Cela permet de spécifier les étendues (ou autorisations) auxquelles l’application demandera accès.

```json
"Scopes": "user.read mail.read"
```

Ensuite, mettez à jour le code dans le fichier *FetchData.razor* pour récupérer les données de messagerie au lieu des détails de la météo par défaut (aléatoire). Remplacez le code dans ce fichier par le suivant :

```csharp
@page "/fetchdata"

@inject IHttpClientFactory HttpClientFactory
@inject Microsoft.Identity.Web.ITokenAcquisition TokenAcquisitionService

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
        _httpClient = HttpClientFactory.CreateClient();


        // get a token
        var token = await TokenAcquisitionService.GetAccessTokenForUserAsync(new string[] { "User.Read", "Mail.Read" });

        // make API call
        _httpClient.DefaultRequestHeaders.Authorization = new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", token);
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

    public class MailMessage
    {
        public string Subject;
        public string Sender;
        public DateTime ReceivedTime;
    }
}

```

Lancer l’application. Vous remarquerez que les nouvelles autorisations ajoutées vous sont demandées, ce qui indique que tout fonctionne comme prévu. Maintenant, en plus des données de base du profil utilisateur, l’application demande l’accès aux données de messagerie.

Après avoir donné votre accord, accédez à la page « Extraire les données » pour lire quelques e-mails.

:::image type="content" source="./media/tutorial-blazor-server/final-app-2.png" alt-text="Capture d’écran de l’application finale. Son titre indique « Bonjour Nicholas » et affiche une liste des e-mails appartenant à Nicholas.":::

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur la création d’applications web qui connectent les utilisateurs dans notre série de scénarios en plusieurs parties :

> [!div class="nextstepaction"]
> [Scénario : application web qui connecte les utilisateurs](scenario-web-app-sign-user-overview.md)
