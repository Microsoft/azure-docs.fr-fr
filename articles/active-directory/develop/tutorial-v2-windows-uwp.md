---
title: 'Tutoriel : Créer une application de plateforme Windows universelle (UWP) qui utilise la plateforme d’identité Microsoft pour l’authentification | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce tutoriel, vous allez créer une application UWP qui utilise la plateforme d’identité Microsoft pour connecter les utilisateurs et obtenir un jeton d’accès pour appeler l’API Microsoft Graph en leur nom.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 12/13/2019
ms.author: jmprieur
ms.custom: devx-track-csharp, aaddev, identityplatformtop40
ms.openlocfilehash: dce2cd0d77ff0a98d4d68e1c99edb472e61ce8a5
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97509459"
---
# <a name="tutorial-call-the-microsoft-graph-api-from-a-universal-windows-platform-uwp-application"></a>Tutoriel : Appeler l’API Microsoft Graph à partir d’une application de plateforme Windows universelle (UWP)

Dans ce tutoriel, vous allez créer une application Plateforme Windows universelle (UWP) native qui connecte les utilisateurs et obtient un jeton d’accès pour appeler l’API Microsoft Graph. 

À la fin de ce guide, votre application appelle une API protégée à l’aide de comptes personnels. Outlook.com, live.com, etc. en sont des exemples. Votre application appelle également des comptes professionnels et scolaires de toute entreprise ou organisation qui utilise Azure Active Directory (Azure AD).

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Créer un projet de *plateforme Windows universelle (UWP)* dans Visual Studio
> * Inscrire l’application dans le Portail Azure
> * Ajouter du code pour prendre en charge la connexion et la déconnexion des utilisateurs
> * Ajouter du code pour appeler l’API Microsoft Graph
> * Test de l'application

## <a name="prerequisites"></a>Prérequis

* [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) avec la charge de travail [plateforme Windows universelle de développement](/windows/uwp/get-started/get-set-up) installée

## <a name="how-this-guide-works"></a>Fonctionnement de ce guide

![Fonctionnement de l’exemple d’application généré par ce tutoriel](./media/tutorial-v2-windows-uwp/uwp-intro.svg)

Ce guide crée un exemple d’application UWP qui interroge l’API Microsoft Graph. Pour ce scénario, un jeton est ajouté à des requêtes HTTP via l’en-tête d’autorisation. La bibliothèque d’authentification Microsoft gère les acquisitions et renouvellements de jetons.

## <a name="nuget-packages"></a>Packages NuGet

Ce guide utilise le package NuGet suivant :

|Bibliothèque|Description|
|---|---|
|[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client)|Microsoft Authentication Library|
|[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph)|Bibliothèque de client Microsoft Graph|

## <a name="set-up-your-project"></a>Configuration de votre projet

Cette section fournit des instructions pas à pas pour intégrer une application Windows Desktop .NET (XAML) à la connexion avec Microsoft. L’application peut alors interroger les API web qui nécessitent un jeton, par exemple l’API Microsoft Graph.

Ce guide crée une application qui affiche un bouton permettant d’interroger l’API Microsoft Graph et un bouton permettant de se déconnecter. Elle affiche aussi des zones de texte qui contiennent les résultats des appels.

> [!NOTE]
> Vous préférez télécharger le projet Visual Studio de cet exemple au lieu de le créer ? [Téléchargez un projet](https://github.com/Azure-Samples/active-directory-dotnet-native-uwp-v2/archive/msal3x.zip), puis passez à l’étape d’[inscription d’application](#register-your-application "étape d’inscription d’application") pour configurer l’exemple de code avant son exécution.

### <a name="create-your-application"></a>Créer votre application

1. Ouvrez Visual Studio et sélectionnez **Créer un projet**.
1. Dans **Créer un nouveau projet**, choisissez **Application vide (Universal Windows)**  pour C#, puis sélectionnez **Suivant**.
1. Dans **Configurer votre nouveau projet**, nommez l’application, puis sélectionnez **Créer**.
1. Si vous y êtes invité, dans **Nouveau projet de plateforme Windows universelle**, sélectionnez une version pour les versions **Cible** et **Minimale**, puis sélectionnez **OK**.

   ![Versions minimale et cible](./media/tutorial-v2-windows-uwp/select-uwp-target-minimum.png)

### <a name="add-microsoft-authentication-library-to-your-project"></a>Ajouter Microsoft Authentication Library à votre projet

1. Dans Visual Studio, sélectionnez **Outils** > **Gestionnaire de package NuGet** > **Console du gestionnaire de package**.
1. Copiez et collez les commandes suivantes dans la fenêtre **Console du Gestionnaire de package**:

    ```powershell
    Install-Package Microsoft.Identity.Client
    Install-Package Microsoft.Graph
    ```

   > [!NOTE]
   > La première commande installe la [bibliothèque d’authentification Microsoft (MSAL.NET)](https://aka.ms/msal-net). MSAL.NET acquiert, met en cache et actualise les jetons utilisateur accédant aux API qui sont protégées par la plateforme d’identité Microsoft. La deuxième commande installe la [bibliothèque de client .NET Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-dotnet) pour authentifier les requêtes soumises à Microsoft Graph et effectuer des appels au service.

### <a name="create-your-applications-ui"></a>Créer l’interface utilisateur de votre application

Visual Studio crée *MainPage.xaml* dans votre modèle de projet. Ouvrez ce fichier, puis remplacez le nœud **Grid** de votre application par le code suivant :

```xml
<Grid>
    <StackPanel Background="Azure">
        <StackPanel Orientation="Horizontal" HorizontalAlignment="Right">
            <Button x:Name="CallGraphButton" Content="Call Microsoft Graph API" HorizontalAlignment="Right" Padding="5" Click="CallGraphButton_Click" Margin="5" FontFamily="Segoe Ui"/>
            <Button x:Name="SignOutButton" Content="Sign-Out" HorizontalAlignment="Right" Padding="5" Click="SignOutButton_Click" Margin="5" Visibility="Collapsed" FontFamily="Segoe Ui"/>
        </StackPanel>
        <TextBlock Text="API Call Results" Margin="2,0,0,-5" FontFamily="Segoe Ui" />
        <TextBox x:Name="ResultText" TextWrapping="Wrap" MinHeight="120" Margin="5" FontFamily="Segoe Ui"/>
        <TextBlock Text="Token Info" Margin="2,0,0,-5" FontFamily="Segoe Ui" />
        <TextBox x:Name="TokenInfoText" TextWrapping="Wrap" MinHeight="70" Margin="5" FontFamily="Segoe Ui"/>
    </StackPanel>
</Grid>
```

### <a name="use-microsoft-authentication-library-to-get-a-token-for-the-microsoft-graph-api"></a>Utiliser la bibliothèque d’authentification Microsoft afin d’obtenir un jeton pour l’API Microsoft Graph

Cette section montre comment utiliser la bibliothèque d’authentification Microsoft afin d’obtenir un jeton pour l’API Microsoft Graph. Apportez des modifications au fichier *MainPage.xaml.cs*.

1. Dans *MainPage.xaml.cs*, ajoutez les références suivantes :

    ```csharp
    using Microsoft.Identity.Client;
    using Microsoft.Graph;
    using System.Diagnostics;
    using System.Threading.Tasks;
    ```

1. Remplacez votre classe `MainPage` par le code suivant :

    ```csharp
    public sealed partial class MainPage : Page
    {

        //Set the scope for API call to user.read
        private string[] scopes = new string[] { "user.read" };

        // Below are the clientId (Application Id) of your app registration and the tenant information.
        // You have to replace:
        // - the content of ClientID with the Application Id for your app registration
        private const string ClientId = "[Application Id pasted from the application registration portal]";

        private const string Tenant = "common"; // Alternatively "[Enter your tenant, as obtained from the Azure portal, e.g. kko365.onmicrosoft.com]"
        private const string Authority = "https://login.microsoftonline.com/" + Tenant;

        // The MSAL Public client app
        private static IPublicClientApplication PublicClientApp;

        private static string MSGraphURL = "https://graph.microsoft.com/v1.0/";
        private static AuthenticationResult authResult;

        public MainPage()
        {
            this.InitializeComponent();
        }

        /// <summary>
        /// Call AcquireTokenAsync - to acquire a token requiring user to sign in
        /// </summary>
        private async void CallGraphButton_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                // Sign in user using MSAL and obtain an access token for Microsoft Graph
                GraphServiceClient graphClient = await SignInAndInitializeGraphServiceClient(scopes);

                // Call the /me endpoint of Graph
                User graphUser = await graphClient.Me.Request().GetAsync();

                // Go back to the UI thread to make changes to the UI
                await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
                {
                    ResultText.Text = "Display Name: " + graphUser.DisplayName + "\nBusiness Phone: " + graphUser.BusinessPhones.FirstOrDefault()
                                      + "\nGiven Name: " + graphUser.GivenName + "\nid: " + graphUser.Id
                                      + "\nUser Principal Name: " + graphUser.UserPrincipalName;
                    DisplayBasicTokenInfo(authResult);
                    this.SignOutButton.Visibility = Visibility.Visible;
                });
            }
            catch (MsalException msalEx)
            {
                await DisplayMessageAsync($"Error Acquiring Token:{System.Environment.NewLine}{msalEx}");
            }
            catch (Exception ex)
            {
                await DisplayMessageAsync($"Error Acquiring Token Silently:{System.Environment.NewLine}{ex}");
                return;
            }
        }
                /// <summary>
        /// Signs in the user and obtains an access token for Microsoft Graph
        /// </summary>
        /// <param name="scopes"></param>
        /// <returns> Access Token</returns>
        private static async Task<string> SignInUserAndGetTokenUsingMSAL(string[] scopes)
        {
            // Initialize the MSAL library by building a public client application
            PublicClientApp = PublicClientApplicationBuilder.Create(ClientId)
                .WithAuthority(Authority)
                .WithUseCorporateNetwork(false)
                .WithRedirectUri("https://login.microsoftonline.com/common/oauth2/nativeclient")
                 .WithLogging((level, message, containsPii) =>
                 {
                     Debug.WriteLine($"MSAL: {level} {message} ");
                 }, LogLevel.Warning, enablePiiLogging: false, enableDefaultPlatformLogging: true)
                .Build();

            // It's good practice to not do work on the UI thread, so use ConfigureAwait(false) whenever possible.
            IEnumerable<IAccount> accounts = await PublicClientApp.GetAccountsAsync().ConfigureAwait(false);
            IAccount firstAccount = accounts.FirstOrDefault();

            try
            {
                authResult = await PublicClientApp.AcquireTokenSilent(scopes, firstAccount)
                                                  .ExecuteAsync();
            }
            catch (MsalUiRequiredException ex)
            {
                // A MsalUiRequiredException happened on AcquireTokenSilentAsync. This indicates you need to call AcquireTokenAsync to acquire a token
                Debug.WriteLine($"MsalUiRequiredException: {ex.Message}");

                authResult = await PublicClientApp.AcquireTokenInteractive(scopes)
                                                  .ExecuteAsync()
                                                  .ConfigureAwait(false);

            }
            return authResult.AccessToken;
        }
    }
    ```

#### <a name="get-a-user-token-interactively"></a>Obtenir un jeton utilisateur de manière interactive<a name="more-information"></a>

La méthode `AcquireTokenInteractive` affiche une fenêtre qui invite les utilisateurs à se connecter. Les applications imposent généralement aux utilisateurs de se connecter de manière interactive la première fois qu’ils accèdent à une ressource protégée. Ils peuvent également avoir besoin de se connecter en cas d’échec d’une opération en mode silencieux pour obtenir un jeton, par exemple quand le mot de passe d’un utilisateur est arrivé à expiration.

#### <a name="get-a-user-token-silently"></a>Obtenir un jeton d’utilisateur en mode silencieux

La méthode `AcquireTokenSilent` gère les acquisitions et renouvellements de jetons sans aucune interaction de l’utilisateur. Après que la méthode `AcquireTokenInteractive` s’est exécutée pour la première fois et a demandé à l’utilisateur des informations d’identification, utilisez la méthode `AcquireTokenSilent` pour demander des jetons pour les appels ultérieurs. Cette méthode acquiert les jetons en mode silencieux. La bibliothèque d’authentification Microsoft gère le cache et le renouvellement des jetons.

La méthode `AcquireTokenSilent` peut échouer. Cet échec peut notamment être dû à une déconnexion de l’utilisateur ou à la modification de son mot de passe sur un autre appareil. Quand la bibliothèque d’authentification Microsoft détecte que le problème nécessite une action interactive, une exception `MsalUiRequiredException` est levée. Votre application peut gérer cette exception de deux manières :

* Votre application appelle immédiatement `AcquireTokenInteractive`. Cet appel invite l’utilisateur à se connecter. Normalement, cette approche est réservée aux applications en ligne qui n’ont pas de contenu hors connexion à la disposition de l’utilisateur. L’exemple créé avec cette installation guidée suit ce modèle. Vous le découvrez en action lors de la première exécution de l’exemple.

   Comme aucun utilisateur n’a encore utilisé l’application, `accounts.FirstOrDefault()` contient une valeur null et lève une exception `MsalUiRequiredException`.

   Le code contenu dans l’exemple gère alors l’exception en appelant `AcquireTokenInteractive`. Cet appel invite l’utilisateur à se connecter.

* Votre application présente une indication visuelle aux utilisateurs qu’ils doivent se connecter. Ils peuvent alors sélectionner le moment opportun pour se connecter. L’application peut réessayer `AcquireTokenSilent` ultérieurement. Employez cette approche quand les utilisateurs peuvent utiliser d’autres fonctionnalités de l’application sans interruption de service, par exemple lorsque le contenu hors connexion est disponible dans l’application. Dans ce cas, les utilisateurs peuvent décider à quel moment ils se connectent. L’application peut réessayer `AcquireTokenSilent` à la suite d’une indisponibilité temporaire du réseau.

### <a name="instantiate-the-microsoft-graph-service-client-by-obtaining-the-token-from-the-signinuserandgettokenusingmsal-method"></a>Instancier le client du service Microsoft Graph en obtenant le jeton à partir de la méthode SignInUserAndGetTokenUsingMSAL

Ajoutez la nouvelle méthode suivante à *MainPage.xaml.cs* :

```csharp
      /// <summary>
     /// Sign in user using MSAL and obtain a token for Microsoft Graph
     /// </summary>
     /// <returns>GraphServiceClient</returns>
     private async static Task<GraphServiceClient> SignInAndInitializeGraphServiceClient(string[] scopes)
     {
         GraphServiceClient graphClient = new GraphServiceClient(MSGraphURL,
             new DelegateAuthenticationProvider(async (requestMessage) =>
             {
                 requestMessage.Headers.Authorization = new AuthenticationHeaderValue("bearer", await SignInUserAndGetTokenUsingMSAL(scopes));
             }));

         return await Task.FromResult(graphClient);
     }
```

#### <a name="more-information-on-making-a-rest-call-against-a-protected-api"></a>Plus d’informations sur l’envoi d’un appel REST à une API protégée

Dans cet exemple d’application, la méthode `GetGraphServiceClient` instancie `GraphServiceClient` à l’aide d’un jeton d’accès. `GraphServiceClient` est ensuite utilisé pour obtenir les informations de profil de l’utilisateur à partir du point de terminaison **me**.

### <a name="add-a-method-to-sign-out-the-user"></a>Ajouter une méthode pour déconnecter l’utilisateur

Pour déconnecter l’utilisateur, ajoutez la méthode suivante au fichier *MainPage.xaml.cs* :

```csharp
/// <summary>
/// Sign out the current user
/// </summary>
private async void SignOutButton_Click(object sender, RoutedEventArgs e)
{
    IEnumerable<IAccount> accounts = await PublicClientApp.GetAccountsAsync().ConfigureAwait(false);
    IAccount firstAccount = accounts.FirstOrDefault();

    try
    {
        await PublicClientApp.RemoveAsync(firstAccount).ConfigureAwait(false);
        await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
        {
            ResultText.Text = "User has signed out";
            this.CallGraphButton.Visibility = Visibility.Visible;
                this.SignOutButton.Visibility = Visibility.Collapsed;
            });
        }
        catch (MsalException ex)
        {
            ResultText.Text = $"Error signing out user: {ex.Message}";
        }
    }
```

> [!NOTE]
> MSAL.NET utilise des méthodes asynchrones pour acquérir des jetons ou manipuler des comptes. Vous devez prendre en charge les actions d’interface utilisateur dans le thread d’interface utilisateur. C’est ce qui explique la raison de l’appel `Dispatcher.RunAsync` et des précautions à observer pour appeler `ConfigureAwait(false)`.

#### <a name="more-information-about-signing-out"></a>Informations supplémentaires sur la déconnexion<a name="more-information-on-sign-out"></a>

La méthode `SignOutButton_Click` supprime l’utilisateur du cache utilisateur de la bibliothèque d’authentification Microsoft. Cette méthode indique à la bibliothèque d’authentification Microsoft qu’elle doit oublier l’utilisateur actuel. Une demande ultérieure d’acquisition d’un jeton n’aboutit que si elle est interactive.

L’application de cet exemple prend en charge un seul utilisateur, La bibliothèque d’authentification Microsoft prend en charge les scénarios où l’utilisateur peut se connecter à plusieurs comptes. C’est le cas, par exemple, d’une application de messagerie hébergeant plusieurs comptes d’un même utilisateur.

### <a name="display-basic-token-information"></a>Afficher les informations de base du jeton

Pour afficher les informations de base du jeton, ajoutez la méthode suivante au fichier *MainPage.xaml.cs* :

```csharp
/// <summary>
/// Display basic information contained in the token. Needs to be called from the UI thread.
/// </summary>
private void DisplayBasicTokenInfo(AuthenticationResult authResult)
{
    TokenInfoText.Text = "";
    if (authResult != null)
    {
        TokenInfoText.Text += $"User Name: {authResult.Account.Username}" + Environment.NewLine;
        TokenInfoText.Text += $"Token Expires: {authResult.ExpiresOn.ToLocalTime()}" + Environment.NewLine;
    }
}
```

#### <a name="more-information"></a>Informations complémentaires<a name="more-information-1"></a>

Les jetons d’ID acquis en utilisant **OpenID Connect** contiennent aussi une petite partie des informations pertinentes pour l’utilisateur. `DisplayBasicTokenInfo` affiche les informations de base contenues dans le jeton, Ces informations se composent du nom d’affichage et de l’ID de l’utilisateur. Elles comprennent aussi la date d’expiration du jeton et la chaîne qui représente le jeton d’accès proprement dit. Si vous cliquez plusieurs fois sur le bouton **Call Microsoft Graph API** (Appeler l’API Microsoft Graph), vous pouvez constater que le même jeton a été réutilisé pour les demandes suivantes. Vous pouvez également voir la date d’expiration différée quand la bibliothèque d’authentification Microsoft détermine qu’il est temps de renouveler le jeton.

### <a name="display-message"></a>Afficher un message

Ajoutez la nouvelle méthode suivante à *MainPage.xaml.cs* :

```csharp
/// <summary>
/// Displays a message in the ResultText. Can be called from any thread.
/// </summary>
private async Task DisplayMessageAsync(string message)
{
     await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal,
         () =>
         {
             ResultText.Text = message;
         });
     }
```

## <a name="register-your-application"></a>Inscrivez votre application

Vous devez maintenant inscrire votre application :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire dans lequel vous voulez inscrire une application.
1. Recherchez et sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Inscriptions d’applications** > **Nouvelle inscription**.
1. Entrez un **nom** pour votre application (par exemple, `UWP-App-calling-MSGraph`). Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
1. Sous **Types de comptes pris en charge**, sélectionnez **Comptes dans un annuaire organisationnel (tout annuaire Azure AD mutualisé) et comptes Microsoft personnels (par exemple, Skype, Xbox)** . 
1. Sélectionnez **Inscription**.
1. Dans la page Vue d’ensemble, recherchez la valeur **ID d’application (client)** et copiez-la. Revenez dans Visual Studio, ouvrez *MainPage.xaml.cs* et remplacez la valeur de `ClientId` par cette valeur.

Configurez l’authentification pour votre application :

1. De retour dans le [portail Azure](https://portal.azure.com), sous **Gérer**, sélectionnez **Authentification** > **Ajouter une plateforme**, puis sélectionnez **Applications de bureau et mobiles**.
1. Dans la section **URI de redirection**, cochez **https://login.microsoftonline.com/common/oauth2/nativeclient** .
1. Sélectionnez **Configurer**.

Configurez des autorisations d’API pour votre application :

1. Sous **Gérer**, sélectionnez **Autorisations de l’API** > **Ajouter une autorisation**.
1. Sélectionnez **Microsoft Graph**.
1. Sélectionnez **Autorisations déléguées**, recherchez *User.Read*, puis vérifiez que **User.Read** est sélectionné.
1. Si vous avez apporté des modifications, sélectionnez **Ajouter des autorisations** pour les enregistrer.

## <a name="enable-integrated-authentication-on-federated-domains-optional"></a>Activer l’authentification intégrée sur des domaines fédérés (facultatif)

Pour activer l’authentification Windows intégrée avec un domaine Azure AD fédéré, le manifeste de l’application doit activer des fonctionnalités supplémentaires. Revenez dans votre application dans Visual Studio.

1. Ouvrez *Package.appxmanifest*.
1. Sélectionnez **Capacités**, puis activez les paramètres suivants :

   * **Authentification en entreprise**
   * **Réseaux privés (client et serveur)**
   * **Certificats utilisateur partagés**

> [!IMPORTANT]
> L’[Authentification Windows intégrée](https://aka.ms/msal-net-iwa) n’est pas configurée par défaut pour cet exemple. Les applications qui demandent des capacités `Enterprise Authentication` ou `Shared User Certificates` nécessitent un niveau de vérification supérieur de Microsoft Store. Tous les développeurs ne souhaitent pas effectuer la vérification de niveau supérieur. Activez ce paramètre uniquement si vous avez besoin d’utiliser l’authentification Windows intégrée avec un domaine Azure AD fédéré.

## <a name="alternate-approach-to-using-withdefaultredirecturi"></a>Autre approche de l’utilisation de WithDefaultRedirectURI()

Dans l’exemple actuel, la méthode `WithRedirectUri("https://login.microsoftonline.com/common/oauth2/nativeclient")` est utilisée. Pour utiliser `WithDefaultRedirectURI()`, effectuez les étapes suivantes :

1. Dans *MainPage.XAML.cs*, remplacez `WithRedirectUri` par `WithDefaultRedirectUri` :

   **Code actuel**

   ```csharp

   PublicClientApp = PublicClientApplicationBuilder.Create(ClientId)
       .WithAuthority(Authority)
       .WithUseCorporateNetwork(false)
       .WithRedirectUri("https://login.microsoftonline.com/common/oauth2/nativeclient")
       .WithLogging((level, message, containsPii) =>
        {
            Debug.WriteLine($"MSAL: {level} {message} ");
        }, LogLevel.Warning, enablePiiLogging: false, enableDefaultPlatformLogging: true)
       .Build();

   ```

   **Code mis à jour**

   ```csharp

   PublicClientApp = PublicClientApplicationBuilder.Create(ClientId)
       .WithAuthority("https://login.microsoftonline.com/common")
       .WithUseCorporateNetwork(false)
       .WithDefaultRedirectUri()
       .WithLogging((level, message, containsPii) =>
        {
            Debug.WriteLine($"MSAL: {level} {message} ");
        }, LogLevel.Warning, enablePiiLogging: false, enableDefaultPlatformLogging: true)
       .Build();
   ```

2.  Recherchez l’URI de rappel de votre application en ajoutant le champ `redirectURI` dans *MainPage.xaml.cs* et en définissant un point d’arrêt sur celui-ci :

    ```csharp

    public sealed partial class MainPage : Page
    {
            ...

            string redirectURI = Windows.Security.Authentication.Web.WebAuthenticationBroker
                                .GetCurrentApplicationCallbackUri().ToString();
            public MainPage()
            {
                ...
            }
           ...
    }

    ```

    Exécutez l’application, puis copiez la valeur de `redirectUri` quand le point d’arrêt est atteint. La valeur doit ressembler à la valeur suivante : `ms-app://s-1-15-2-1352796503-54529114-405753024-3540103335-3203256200-511895534-1429095407/`

    Vous pouvez ensuite supprimer la ligne de code, car elle n’est nécessaire qu’une seule fois, pour extraire la valeur.

3. Dans le portail d’inscription des applications, ajoutez la valeur retournée dans **RedirectUri** dans le volet **Authentification**.

## <a name="test-your-code"></a>Test de votre code

Pour tester votre application, appuyez sur la touche **F5** afin d’exécuter votre projet dans Visual Studio. Votre fenêtre principale s’affiche :

![Interface utilisateur de l’application](./media/tutorial-v2-windows-uwp/testapp-ui-vs2019.png)

Lorsque vous êtes prêt à passer aux tests, sélectionnez **Call Microsoft Graph API** (Appeler l’API Microsoft Graph). Utilisez ensuite un compte professionnel Azure AD ou un compte Microsoft (par exemple, live.com ou outlook.com) pour vous connecter. La première fois qu’un utilisateur exécute ce test, l’application affiche une fenêtre lui demandant de se connecter.

### <a name="consent"></a>Consentement

La première fois que vous vous connectez à votre application, un écran de consentement semblable l’image suivante s’affiche. Sélectionnez **Oui** pour donner explicitement votre consentement pour l’accès :

![Écran de consentement d’accès](./media/tutorial-v2-windows-uwp/consentscreen-vs2019.png)

### <a name="expected-results"></a>Résultats attendus

Les informations de profil utilisateur retournées par l’appel de l’API Microsoft Graph s’affichent dans l’écran **API Call Results** (Résultats de l’appel d’API) :

![Écran API Call Results (Résultats de l’appel d’API)](./media/tutorial-v2-windows-uwp/uwp-results-screen-vs2019.png)

Des informations de base sur le jeton obtenu via `AcquireTokenInteractive` ou `AcquireTokenSilent` s’affichent également dans la zone **Token Info** (Informations sur le jeton) :

|Propriété  |Format  |Description |
|---------|---------|---------|
|`Username` |`user@domain.com` |Nom qui identifie l’utilisateur.|
|`Token Expires` |`DateTime` |L’heure d’expiration du jeton. La bibliothèque d’authentification Microsoft repousse la date d’expiration en renouvelant le jeton si nécessaire.|

### <a name="more-information-about-scopes-and-delegated-permissions"></a>Informations supplémentaires sur les étendues et les autorisations déléguées

L’API Microsoft Graph nécessite l’étendue `user.read` pour lire le profil d’un utilisateur. Par défaut, cette étendue est ajoutée à toutes les applications inscrites sur le portail d’inscription de l’application. D’autres API pour Microsoft Graph et des API personnalisées pour votre serveur back-end peuvent nécessiter des étendues supplémentaires. Par exemple, l’API Microsoft Graph nécessite l’étendue `Calendars.Read` pour lister les calendriers de l’utilisateur.

Pour accéder aux calendriers de l’utilisateur dans le contexte d’une application, ajoutez l’autorisation déléguée `Calendars.Read` aux informations d’inscription de l’application. Ajoutez ensuite l’étendue `Calendars.Read` à l’appel `acquireTokenSilent`.

> [!NOTE]
> Les utilisateurs peuvent être invités à donner des consentements supplémentaires lorsque vous augmentez le nombre d’étendues.

## <a name="known-issues"></a>Problèmes connus

### <a name="issue-1"></a>Problème 1

Vous recevez l’un des messages d’erreur suivants quand vous vous connectez à votre application dans un domaine Azure AD fédéré :

* « Aucun certificat client valide trouvé dans la demande. »
* « Aucun certificat valide trouvé dans le magasin de certificats de l’utilisateur. »
* « Réessayez en choisissant une autre méthode d’authentification. »

**Cause :** les fonctionnalités d’entreprise et de certificats ne sont pas activées.

**Solution :** Suivez les étapes décrites dans [Activer l’authentification intégrée sur des domaines fédérés (facultatif)](#enable-integrated-authentication-on-federated-domains-optional).

### <a name="issue-2"></a>Problème 2

Vous activez [l’authentification intégrée sur des domaines fédérés](#enable-integrated-authentication-on-federated-domains-optional) et vous essayez d’utiliser Windows Hello sur un ordinateur Windows 10 pour vous connecter à un environnement dans lequel l’authentification multifacteur est configurée. La liste des certificats s’affiche. Si vous choisissez d’utiliser votre code confidentiel, la fenêtre correspondante ne s’affiche jamais.

**Cause :** Ce problème est une limitation connue du répartiteur d’authentification web dans les applications UWP qui s’exécutent sur des ordinateurs de bureau Windows 10. Il fonctionne correctement sur Windows 10 Mobile.

**Solution de contournement :** sélectionnez **Connexion avec d’autres options**, puis **Sign in with a username and password** (Se connecter avec un nom d’utilisateur et un mot de passe). Sélectionnez **Provide your password** (Indiquer votre mot de passe), puis passez au processus d’authentification par téléphone.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur l’utilisation de la Bibliothèque d’authentification Microsoft (MSAL) pour l’autorisation et l’authentification dans les applications .NET :

> [!div class="nextstepaction"]
> [Vue d’ensemble de Microsoft Authentication Library (MSAL)](msal-overview.md)
