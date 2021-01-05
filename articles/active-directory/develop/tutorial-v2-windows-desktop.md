---
title: 'Tutoriel : Créer une application Windows Presentation Foundation (WPF) qui utilise la plateforme d’identité Microsoft pour l’authentification | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce tutoriel, vous allez créer une application WPF qui utilise la plateforme d’identité Microsoft pour connecter les utilisateurs et obtenir un jeton d’accès pour appeler l’API Microsoft Graph en leur nom.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 12/12/2019
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 5fb7c0df653048adcffceda4d8a384be823b5c3a
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507691"
---
# <a name="tutorial-call-the-microsoft-graph-api-from-a-windows-desktop-app"></a>Tutoriel : Appeler l’API Microsoft Graph à partir d’une application de bureau Windows

Dans ce tutoriel, vous allez créer une application Windows Desktop .NET (XAML) native qui connecte des utilisateurs et obtient un jeton d’accès pour appeler l’API Microsoft Graph. 

À la fin de ce guide, votre application sera capable d’appeler une API protégée utilisant des comptes personnels (y compris outlook.com, live.com, etc.). L’application utilisera également des comptes professionnels et scolaires de toute entreprise ou organisation utilisant Azure Active Directory.

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Créez un projet *Windows Presentation Foundation (WPF)* dans Visual Studio
> * Installer la bibliothèque Microsoft Authentication Library (MSAL) pour .NET
> * Inscrire l’application dans le Portail Azure
> * Ajouter du code pour prendre en charge la connexion et la déconnexion des utilisateurs
> * Ajouter du code pour appeler l’API Microsoft Graph
> * Test de l'application

## <a name="prerequisites"></a>Prérequis

* [Visual Studio 2019](https://visualstudio.microsoft.com/vs/)

## <a name="how-the-sample-app-generated-by-this-guide-works"></a>Fonctionnement de l’exemple d’application de ce guide

![Fonctionnement de l’exemple d’application généré par ce tutoriel](./media/active-directory-develop-guidedsetup-windesktop-intro/windesktophowitworks.svg)

L’exemple d’application que vous créez avec ce guide permet à une application de bureau Windows d’interroger l’API Microsoft Graph ou une API web qui accepte les jetons d’un point de terminaison de la plateforme d’identité Microsoft. Pour ce scénario, vous ajoutez un jeton aux requêtes HTTP via l’en-tête d’autorisation. La bibliothèque d’authentification Microsoft (MSAL) gère l’acquisition et le renouvellement de jetons.

## <a name="handling-token-acquisition-for-accessing-protected-web-apis"></a>Gestion de l’acquisition de jetons pour accéder à des API web protégées

Une fois l’utilisateur authentifié, l’exemple d’application reçoit un jeton que vous pouvez utiliser pour interroger l’API Microsoft Graph ou une API web sécurisée par la plateforme d’identités Microsoft pour les développeurs.

Les API comme Microsoft Graph nécessitent un jeton d’accès pour autoriser l’accès à des ressources spécifiques. Par exemple, un jeton est nécessaire pour lire le profil d’un utilisateur, accéder au calendrier d’un utilisateur ou envoyer un e-mail. Votre application peut demander un jeton d’accès à l’aide de MSAL pour accéder à ces ressources en spécifiant les étendues d’API. Ce jeton d’accès est ensuite ajouté à l’en-tête d’autorisation HTTP pour chaque appel effectué sur la ressource protégée.

MSAL gère la mise en cache et l’actualisation des jetons d’accès pour vous, ce qui évite à votre application d’avoir à le faire.

## <a name="nuget-packages"></a>Packages NuGet

Ce guide utilise les packages NuGet suivants :

|Bibliothèque|Description|
|---|---|
|[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client)|Bibliothèque d’authentification Microsoft (MSAL.NET)|

## <a name="set-up-your-project"></a>Configuration de votre projet

Dans cette section, vous allez créer un projet pour apprendre à intégrer une application de bureau Windows .NET (XAML) avec l’option *Se connecter avec Microsoft* pour que l’application puisse interroger les API web qui nécessitent un jeton.

L’application que vous créez dans ce guide affiche un bouton pour appeler un graphique, une zone pour afficher les résultats à l’écran et un bouton de déconnexion.

> [!NOTE]
> Vous préférez télécharger le projet Visual Studio de cet exemple ? [Téléchargez un projet](https://github.com/Azure-Samples/active-directory-dotnet-desktop-msgraph-v2/archive/msal3x.zip) et passez à [l’étape Configuration](#register-your-application) pour configurer l’exemple de code avant de l’exécuter.
>

Pour créer l’application, procédez comme suit :

1. Dans Visual Studio, sélectionnez **Fichier** > **Nouveau** > **Projet**.
2. Sous **Modèles**, sélectionnez **Visual C#** .
3. Sélectionnez **Application WPF (.NET Framework)** , selon la version Visual Studio que vous utilisez.

## <a name="add-msal-to-your-project"></a>Ajouter MSAL à votre projet

1. Dans Visual Studio, sélectionnez **Outils** > **Gestionnaire de package NuGet**> **Console du gestionnaire de package**.
2. Dans la fenêtre Console du gestionnaire de package, collez la commande Azure PowerShell suivante :

    ```powershell
    Install-Package Microsoft.Identity.Client -Pre
    ```

    > [!NOTE]
    > Cette commande installe Microsoft Authentication Library. MSAL gère l’acquisition, la mise en cache et l’actualisation des jetons d’utilisateur permettant d’accéder aux API protégées par Azure Active Directory v2.0.
    >

## <a name="register-your-application"></a>Inscrivez votre application

Vous pouvez inscrire votre application de deux manières.

### <a name="option-1-express-mode"></a>Option 1 : Mode Express

Vous pouvez inscrire rapidement votre application en procédant comme suit :
1. Accédez au [portail Azure - Inscription d’applications](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/WinDesktopQuickstartPage/sourceType/docs).
1. Entrez un nom pour votre application, puis sélectionnez **Inscrire**.
1. Suivez les instructions pour télécharger et configurer automatiquement votre nouvelle application en un seul clic.

### <a name="option-2-advanced-mode"></a>Option n°2 : Mode Avancé

Pour inscrire votre application et ajouter les informations d’inscription de l’application à votre solution, procédez comme suit :
1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire dans lequel vous voulez inscrire une application.
1. Recherchez et sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Inscriptions d’applications** > **Nouvelle inscription**.
1. Entrez un **nom** pour votre application (par exemple, `Win-App-calling-MsGraph`). Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
1. Dans la section **Types de comptes pris en charge**, sélectionnez **Comptes dans un annuaire d’organisation (tout annuaire Azure AD - Multilocataire) et comptes Microsoft personnels (par exemple, Skype, Xbox)** .
1. Sélectionnez **Inscription**.
1. Sous **Gérer**, sélectionnez **Authentification** > **Ajouter une plateforme**.
1. Sélectionnez **Applications de bureau et mobiles**.
1. Dans la section **URI de redirection**, sélectionnez **https://login.microsoftonline.com/common/oauth2/nativeclient** .
1. Sélectionnez **Configurer**.
1. Accédez à Visual Studio, ouvrez le fichier *App.xaml.cs*, puis remplacez `Enter_the_Application_Id_here` dans l’extrait de code ci-dessous par l’ID de l’application que vous venez d’inscrire et de copier.

    ```csharp
    private static string ClientId = "Enter_the_Application_Id_here";
    ```

## <a name="add-the-code-to-initialize-msal"></a>Ajoutez le code pour initialiser MSAL

Dans cette étape, vous allez créer une classe pour gérer l’interaction avec MSAL, telle que la gestion des jetons.

1. Ouvrez le fichier *App.xaml.cs* et ajoutez la référence de MSAL à la classe :

    ```csharp
    using Microsoft.Identity.Client;
    ```
   <!-- Workaround for Docs conversion bug -->

2. Mettez à jour la classe d’application comme suit :

    ```csharp
    public partial class App : Application
    {
        static App()
        {
            _clientApp = PublicClientApplicationBuilder.Create(ClientId)
                .WithAuthority(AzureCloudInstance.AzurePublic, Tenant)
                .Build();
        }

        // Below are the clientId (Application Id) of your app registration and the tenant information.
        // You have to replace:
        // - the content of ClientID with the Application Id for your app registration
        // - the content of Tenant by the information about the accounts allowed to sign-in in your application:
        //   - For Work or School account in your org, use your tenant ID, or domain
        //   - for any Work or School accounts, use `organizations`
        //   - for any Work or School accounts, or Microsoft personal account, use `common`
        //   - for Microsoft Personal account, use consumers
        private static string ClientId = "0b8b0665-bc13-4fdc-bd72-e0227b9fc011";

        private static string Tenant = "common";

        private static IPublicClientApplication _clientApp ;

        public static IPublicClientApplication PublicClientApp { get { return _clientApp; } }
    }
    ```

## <a name="create-the-application-ui"></a>Créer l’interface utilisateur de l’application

La section suivante explique comment une application peut interroger un serveur principal protégé tel que Microsoft Graph.

Un fichier *MainWindow.xaml* doit être automatiquement créé dans le cadre de votre modèle de projet. Ouvrez ce fichier, puis remplacez le nœud *\<Grid>* de votre application par le code suivant :

```xml
<Grid>
    <StackPanel Background="Azure">
        <StackPanel Orientation="Horizontal" HorizontalAlignment="Right">
            <Button x:Name="CallGraphButton" Content="Call Microsoft Graph API" HorizontalAlignment="Right" Padding="5" Click="CallGraphButton_Click" Margin="5" FontFamily="Segoe Ui"/>
            <Button x:Name="SignOutButton" Content="Sign-Out" HorizontalAlignment="Right" Padding="5" Click="SignOutButton_Click" Margin="5" Visibility="Collapsed" FontFamily="Segoe Ui"/>
        </StackPanel>
        <Label Content="API Call Results" Margin="0,0,0,-5" FontFamily="Segoe Ui" />
        <TextBox x:Name="ResultText" TextWrapping="Wrap" MinHeight="120" Margin="5" FontFamily="Segoe Ui"/>
        <Label Content="Token Info" Margin="0,0,0,-5" FontFamily="Segoe Ui" />
        <TextBox x:Name="TokenInfoText" TextWrapping="Wrap" MinHeight="70" Margin="5" FontFamily="Segoe Ui"/>
    </StackPanel>
</Grid>
```

## <a name="use-msal-to-get-a-token-for-the-microsoft-graph-api"></a>Utiliser MSAL pour obtenir un jeton pour l’API Microsoft Graph

Dans cette section, vous allez utiliser MSAL pour obtenir un jeton pour l’API Microsoft Graph.

1. Dans le fichier *MainWindow.xaml.cs*, ajoutez la référence de MSAL à la classe :

    ```csharp
    using Microsoft.Identity.Client;
    ```

2. Remplacez le code de classe `MainWindow` par le suivant :

    ```csharp
    public partial class MainWindow : Window
    {
        //Set the API Endpoint to Graph 'me' endpoint
        string graphAPIEndpoint = "https://graph.microsoft.com/v1.0/me";

        //Set the scope for API call to user.read
        string[] scopes = new string[] { "user.read" };


        public MainWindow()
        {
            InitializeComponent();
        }

      /// <summary>
        /// Call AcquireToken - to acquire a token requiring user to sign-in
        /// </summary>
        private async void CallGraphButton_Click(object sender, RoutedEventArgs e)
        {
            AuthenticationResult authResult = null;
            var app = App.PublicClientApp;
            ResultText.Text = string.Empty;
            TokenInfoText.Text = string.Empty;

            var accounts = await app.GetAccountsAsync();
            var firstAccount = accounts.FirstOrDefault();

            try
            {
                authResult = await app.AcquireTokenSilent(scopes, firstAccount)
                    .ExecuteAsync();
            }
            catch (MsalUiRequiredException ex)
            {
                // A MsalUiRequiredException happened on AcquireTokenSilent.
                // This indicates you need to call AcquireTokenInteractive to acquire a token
                System.Diagnostics.Debug.WriteLine($"MsalUiRequiredException: {ex.Message}");

                try
                {
                    authResult = await app.AcquireTokenInteractive(scopes)
                        .WithAccount(accounts.FirstOrDefault())
                        .WithPrompt(Prompt.SelectAccount)
                        .ExecuteAsync();
                }
                catch (MsalException msalex)
                {
                    ResultText.Text = $"Error Acquiring Token:{System.Environment.NewLine}{msalex}";
                }
            }
            catch (Exception ex)
            {
                ResultText.Text = $"Error Acquiring Token Silently:{System.Environment.NewLine}{ex}";
                return;
            }

            if (authResult != null)
            {
                ResultText.Text = await GetHttpContentWithToken(graphAPIEndpoint, authResult.AccessToken);
                DisplayBasicTokenInfo(authResult);
                this.SignOutButton.Visibility = Visibility.Visible;
            }
        }
        }
    ```

### <a name="more-information"></a>Informations complémentaires

#### <a name="get-a-user-token-interactively"></a>Obtenir un jeton d’utilisateur de manière interactive

L’appel de la méthode `AcquireTokenInteractive` affiche une fenêtre invitant les utilisateurs à se connecter. Les applications requièrent généralement que les utilisateurs se connectent de manière interactive la première fois qu’ils cherchent à accéder à une ressource protégée. Ils peuvent également avoir besoin de se connecter en cas d’échec d’une opération en mode silencieux pour obtenir un jeton (par exemple, quand un mot de passe utilisateur a expiré).

#### <a name="get-a-user-token-silently"></a>Obtenir un jeton d’utilisateur en mode silencieux

La méthode `AcquireTokenSilent` gère les acquisitions et renouvellements de jetons sans aucune interaction de l’utilisateur. Quand `AcquireTokenInteractive` est exécuté pour la première fois, la méthode `AcquireTokenSilent` est généralement celle à utiliser pour obtenir les jetons permettant d’accéder aux ressources protégées pour les appels suivants, étant donné que les appels pour les demandes ou renouvellements de jetons se font en mode silencieux.

La méthode `AcquireTokenSilent` peut échouer. Cet échec peut être dû à une déconnexion de l’utilisateur ou à la modification de son mot de passe sur un autre appareil. Quand la bibliothèque MSAL détecte que le problème peut être résolu par une intervention interactive, elle déclenche une exception `MsalUiRequiredException`. Votre application peut gérer cette exception de deux manières :

* Elle peut appeler immédiatement `AcquireTokenInteractive`. Cet appel invite l’utilisateur à se connecter. Cette méthode est généralement employée dans les applications en ligne où aucun contenu hors connexion n’est disponible pour l’utilisateur. L’exemple généré par cette installation guidée utilise ce modèle, que vous pouvez voir en action la première fois que vous exécutez l’exemple.

* Aucun utilisateur n’ayant encore utilisé l’application, `PublicClientApp.Users.FirstOrDefault()` contient une valeur null, et une exception `MsalUiRequiredException` est levée.

* Le code de l’exemple gère ensuite cette exception en appelant `AcquireTokenInteractive`, après quoi l’utilisateur est invité à se connecter.

* Il peut également afficher à la place une indication visuelle informant les utilisateurs qu’une connexion interactive est nécessaire, pour permettre à ces derniers de sélectionner le bon moment pour se connecter. L’application peut également effectuer une nouvelle tentative de `AcquireTokenSilent` ultérieurement. Ce modèle est souvent utilisé quand les utilisateurs peuvent utiliser d’autres fonctionnalités de l’application sans interruption, par exemple, quand le contenu hors connexion est disponible dans l’application. Dans ce cas, les utilisateurs peuvent décider de se connecter pour accéder à la ressource protégée ou pour actualiser les informations obsolètes. L’application peut également décider d’effectuer une nouvelle tentative de `AcquireTokenSilent` une fois le réseau rétabli après une indisponibilité temporaire.

## <a name="call-the-microsoft-graph-api-by-using-the-token-you-just-obtained"></a>Appeler l’API Microsoft Graph à l’aide du jeton que vous venez d’obtenir

Ajoutez la nouvelle méthode suivante à votre fichier `MainWindow.xaml.cs`. Cette méthode permet d’envoyer une demande `GET` à l’API Graph à l’aide d’un en-tête d’autorisation :

```csharp
/// <summary>
/// Perform an HTTP GET request to a URL using an HTTP Authorization header
/// </summary>
/// <param name="url">The URL</param>
/// <param name="token">The token</param>
/// <returns>String containing the results of the GET operation</returns>
public async Task<string> GetHttpContentWithToken(string url, string token)
{
    var httpClient = new System.Net.Http.HttpClient();
    System.Net.Http.HttpResponseMessage response;
    try
    {
        var request = new System.Net.Http.HttpRequestMessage(System.Net.Http.HttpMethod.Get, url);
        //Add the token in Authorization header
        request.Headers.Authorization = new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", token);
        response = await httpClient.SendAsync(request);
        var content = await response.Content.ReadAsStringAsync();
        return content;
    }
    catch (Exception ex)
    {
        return ex.ToString();
    }
}
```

### <a name="more-information-about-making-a-rest-call-against-a-protected-api"></a>Informations supplémentaires sur l’envoi d’un appel REST à une API protégée

Dans cet exemple d’application, vous utiliserez la méthode `GetHttpContentWithToken` pour envoyer une requête HTTP `GET` à une ressource protégée qui requiert un jeton, puis pour retourner le contenu à l’appelant. Cette méthode ajoute le jeton acquis à l’en-tête d’autorisation HTTP. Dans cet exemple, la ressource est le point de terminaison *me* de l’API Microsoft Graph, qui affiche les informations de profil de l’utilisateur.

## <a name="add-a-method-to-sign-out-a-user"></a>Ajouter une méthode pour déconnecter un utilisateur

Pour déconnecter un utilisateur, ajoutez la méthode suivante à votre fichier `MainWindow.xaml.cs` :

```csharp
/// <summary>
/// Sign out the current user
/// </summary>
private async void SignOutButton_Click(object sender, RoutedEventArgs e)
{
    var accounts = await App.PublicClientApp.GetAccountsAsync();

    if (accounts.Any())
    {
        try
        {
            await App.PublicClientApp.RemoveAsync(accounts.FirstOrDefault());
            this.ResultText.Text = "User has signed-out";
            this.CallGraphButton.Visibility = Visibility.Visible;
            this.SignOutButton.Visibility = Visibility.Collapsed;
        }
        catch (MsalException ex)
        {
            ResultText.Text = $"Error signing-out user: {ex.Message}";
        }
    }
}
```

### <a name="more-information-about-user-sign-out"></a>Plus d'informations sur la déconnexion d’utilisateurs

La méthode `SignOutButton_Click` supprime l’utilisateur du cache utilisateur de MSAL en ordonnant à MSAL d’oublier l’utilisateur actuel pour que la demande suivante d’acquisition de jeton n’aboutisse que si elle est effectuée de manière interactive.

Bien que l’application de cet exemple ne prenne en charge qu’un seul utilisateur, MSAL autorise les scénarios où plusieurs comptes peuvent être connectés en même temps. C’est le cas, par exemple, d’une application de messagerie hébergeant plusieurs comptes d’un même utilisateur.

## <a name="display-basic-token-information"></a>Afficher les informations de base du jeton

Pour afficher les informations de base du jeton, ajoutez la méthode suivante à votre fichier *MainWindow.xaml.cs* :

```csharp
/// <summary>
/// Display basic information contained in the token
/// </summary>
private void DisplayBasicTokenInfo(AuthenticationResult authResult)
{
    TokenInfoText.Text = "";
    if (authResult != null)
    {
        TokenInfoText.Text += $"Username: {authResult.Account.Username}" + Environment.NewLine;
        TokenInfoText.Text += $"Token Expires: {authResult.ExpiresOn.ToLocalTime()}" + Environment.NewLine;
    }
}
```

### <a name="more-information"></a>Informations complémentaires

Outre le jeton d’accès qui est utilisé pour appeler l’API Microsoft Graph, MSAL obtient également un jeton d’ID une fois l’utilisateur connecté. Ce jeton contient un petit sous-ensemble d’informations pertinentes pour les utilisateurs. La méthode `DisplayBasicTokenInfo` affiche les informations de base du jeton. Par exemple, le nom affiché et l’ID de l’utilisateur, ainsi que la date d’expiration du jeton et la chaîne qui représente le jeton d’accès lui-même. Si vous cliquez plusieurs fois sur le bouton *Call Microsoft Graph API* (Appeler l’API Microsoft Graph), vous observerez que le même jeton a été réutilisé pour les demandes suivantes. Vous constatez également que la date d’expiration est différée lorsque MSAL détermine qu’il est temps de renouveler le jeton.

[!INCLUDE [5. Test and Validate](../../../includes/active-directory-develop-guidedsetup-windesktop-test.md)]

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la création d’applications de bureau qui appellent des API Web protégées dans notre série de scénarios en plusieurs parties :

> [!div class="nextstepaction"]
> [Scénario : Application de bureau qui appelle des API web](scenario-desktop-overview.md)
