---
title: Guide relatif à l’authentification des clients du service Azure SignalR Service
description: Découvrez comment implémenter votre propre authentification et l’intégrer à Azure SignalR Service en suivant l’exemple e2e.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.date: 11/13/2019
ms.author: zhshang
ms.custom: devx-track-js, devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: 8dd3c60c3d1b714ab75b496a94ba4bd5aec4e43d
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96558468"
---
# <a name="azure-signalr-service-authentication"></a>Authentification par Azure SignalR Service

Ce didacticiel s’appuie sur l’application de salle de conversation présentée dans le démarrage rapide. Si vous n’avez pas effectué l’exercice de [création de salle de conversation avec le service SignalR](signalr-quickstart-dotnet-core.md), exécutez-le dans un premier temps.

Dans ce didacticiel, vous allez découvrir comment implémenter votre propre authentification et l’intégrer dans le service Microsoft Azure SignalR.

L’authentification utilisée initialement dans l’application de salle de conversation du démarrage rapide est trop simple pour les scénarios concrets. L’application permet à chaque client de revendiquer son identité, et le serveur accepte cet état de fait. Cette approche n’est pas très utile dans les applications concrètes, où un utilisateur non autorisé pourrait usurper l’identité d’un autre utilisateur pour accéder à des données sensibles.

[GitHub](https://github.com/) fournit des API d’authentification basées sur un protocole standard populaire appelé [OAuth](https://oauth.net/). Ces API autorisent des applications tierces à authentifier des comptes GitHub. Dans ce didacticiel, vous utiliserez ces API pour implémenter l’authentification via un compte GitHub, avant de prendre en charge les connexions clientes sur l’application de salle de conversation. Une fois le compte GitHub authentifié, ses données sont ajoutées sous forme de cookie à mettre à profit pour l’authentification du client web.

Pour plus d’informations sur les API d’authentification OAuth fournies via GitHub, voir [Principes fondamentaux de l’authentification](https://developer.github.com/v3/guides/basics-of-authentication/).

Vous pouvez utiliser l’éditeur de code de votre choix pour exécuter les étapes de ce démarrage rapide. Toutefois, [Visual Studio Code](https://code.visualstudio.com/) est une excellente option disponible sur les plateformes Windows, macOS et Linux.

Le code de ce didacticiel est disponible au téléchargement dans le [référentiel AzureSignalR-samples GitHub](https://github.com/aspnet/AzureSignalR-samples/tree/master/samples/GitHubChat).

![Flux OAuth complet hébergé dans Azure](media/signalr-concept-authenticate-oauth/signalr-oauth-complete-azure.png)

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Enregistrer une nouvelle application OAuth avec votre compte GitHub
> * Ajouter un contrôleur d’authentification pour prendre en charge l’authentification GitHub
> * Déployer votre application web ASP.NET Core sur Azure

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous devez disposer des éléments suivants :

- Compte créé sur [GitHub](https://github.com/)
- [Git](https://git-scm.com/)
- [Kit de développement logiciel (SDK) .NET Core](https://www.microsoft.com/net/download/windows)
- [Azure Cloud Shell](/azure/cloud-shell/quickstart) configuré pour l’environnement Bash
- Télécharger ou cloner le référentiel GitHub [AzureSignalR-sample](https://github.com/aspnet/AzureSignalR-samples).

## <a name="create-an-oauth-app"></a>Créer une application OAuth

1. Ouvrez un navigateur web, puis accédez à `https://github.com` et connectez-vous à votre compte.

2. Pour votre compte, accédez à **Paramètres** > **Paramètres de développeur**, puis cliquez sur **Register a new application (Inscrire une nouvelle application)** ou sur **New OAuth App (Nouvelle application OAuth)** sous *OAuth Apps (Applications OAuth)* .

3. Utilisez les paramètres suivants pour la nouvelle application OAuth, puis cliquez sur **Inscrire l’application** :

    | Nom de paramètre | Valeur suggérée | Description |
    | ------------ | --------------- | ----------- |
    | Nom de l'application | *Conversation Azure SignalR* | L’utilisateur GitHub devrait être en mesure de reconnaître l’application avec laquelle il s’authentifie et de lui faire confiance.   |
    | URL de la page d’accueil | `http://localhost:5000/home` | |
    | Description de l’application | *Un exemple de salle de conversation utilisant le service Azure SignalR avec l’authentification GitHub* | Une description utile de l’application qui aidera les utilisateurs de votre application à comprendre le contexte de l’authentification utilisée. |
    | URL de rappel d’autorisation | `http://localhost:5000/signin-github` | Ce paramètre est le paramètre essentiel de votre application OAuth. Il s’agit de l’URL de rappel renvoyée par GitHub à l’utilisateur une fois l’authentification créée. Dans ce didacticiel, vous devez utiliser l’URL de rappel par défaut associée au package *AspNet.Security.OAuth.GitHub*, */signin-github*.  |

4. Une fois le nouvel enregistrement d’application OAuth terminé, ajoutez l’*ID client* et la *Question secrète du client* à l’outil Secret Manager à l’aide des commandes suivantes. Remplacez les paramètres *Your_GitHub_Client_Id* et *Your_GitHub_Client_Secret* par les valeurs de votre application OAuth.

    ```dotnetcli
    dotnet user-secrets set GitHubClientId Your_GitHub_Client_Id
    dotnet user-secrets set GitHubClientSecret Your_GitHub_Client_Secret
    ```

## <a name="implement-the-oauth-flow"></a>Implémenter le flux OAuth

### <a name="update-the-startup-class-to-support-github-authentication"></a>Mettre à jour la classe de démarrage pour prendre en charge l’authentification GitHub

1. Ajoutez une référence aux derniers packages *Microsoft.AspNetCore.Authentication.Cookies* et *AspNet.Security.OAuth.GitHub*, puis restaurez l’ensemble des packages.

    ```dotnetcli
    dotnet add package Microsoft.AspNetCore.Authentication.Cookies -v 2.1.0-rc1-30656
    dotnet add package AspNet.Security.OAuth.GitHub -v 2.0.0-rc2-final
    dotnet restore
    ```

1. Ouvrez *Startup.cs*, puis ajoutez les instructions `using` pour les espaces de noms suivants :

    ```csharp
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Security.Claims;
    using Microsoft.AspNetCore.Authentication.Cookies;
    using Microsoft.AspNetCore.Authentication.OAuth;
    using Newtonsoft.Json.Linq;
    ```

2. En haut de la classe `Startup`, ajoutez des constantes à associer aux clés Secret Manager qui contiennent les secrets d’application GitHub OAuth.

    ```csharp
    private const string GitHubClientId = "GitHubClientId";
    private const string GitHubClientSecret = "GitHubClientSecret";
    ```

3. Ajoutez le code suivant à la méthode `ConfigureServices` afin de prendre en charge l’authentification avec l’application GitHub OAuth :

    ```csharp
    services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
        .AddCookie()
        .AddGitHub(options =>
        {
            options.ClientId = Configuration[GitHubClientId];
            options.ClientSecret = Configuration[GitHubClientSecret];
            options.Scope.Add("user:email");
            options.Events = new OAuthEvents
            {
                OnCreatingTicket = GetUserCompanyInfoAsync
            };
        });
    ```

4. Ajoutez la méthode d’aide `GetUserCompanyInfoAsync` à la classe `Startup`.

    ```csharp
    private static async Task GetUserCompanyInfoAsync(OAuthCreatingTicketContext context)
    {
        var request = new HttpRequestMessage(HttpMethod.Get, context.Options.UserInformationEndpoint);
        request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", context.AccessToken);

        var response = await context.Backchannel.SendAsync(request,
            HttpCompletionOption.ResponseHeadersRead, context.HttpContext.RequestAborted);

        var user = JObject.Parse(await response.Content.ReadAsStringAsync());
        if (user.ContainsKey("company"))
        {
            var company = user["company"].ToString();
            var companyIdentity = new ClaimsIdentity(new[]
            {
                new Claim("Company", company)
            });
            context.Principal.AddIdentity(companyIdentity);
        }
    }
    ```

5. Mettez à jour la méthode `Configure` de la classe de démarrage avec la ligne suivante de code, puis enregistrez le fichier.

    ```csharp
    app.UseAuthentication();
    ```

### <a name="add-an-authentication-controller"></a>Ajoutez un contrôleur d’authentification

Dans cette section, vous allez implémenter une API `Login` qui authentifie les clients à l’aide de l’application GitHub OAuth. Une fois authentifiée, l’API ajoute un cookie à la réponse du client web avant de rediriger le client vers l’application de conversation. Ce cookie sert ensuite à identifier le client.

1. Ajoutez un nouveau fichier de code de contrôleur au répertoire *chattest\Controllers*. Nommez le fichier *AuthController.cs*.

2. Ajoutez le code suivant pour le contrôleur d’authentification. Assurez-vous de mettre à jour l’espace de noms, si votre répertoire de projet n’était pas *chattest* :

    ```csharp
    using AspNet.Security.OAuth.GitHub;
    using Microsoft.AspNetCore.Authentication;
    using Microsoft.AspNetCore.Mvc;

    namespace chattest.Controllers
    {
        [Route("/")]
        public class AuthController : Controller
        {
            [HttpGet("login")]
            public IActionResult Login()
            {
                if (!User.Identity.IsAuthenticated)
                {
                    return Challenge(GitHubAuthenticationDefaults.AuthenticationScheme);
                }

                HttpContext.Response.Cookies.Append("githubchat_username", User.Identity.Name);
                HttpContext.SignInAsync(User);
                return Redirect("/");
            }
        }
    }
    ```

3. Enregistrez vos modifications.

### <a name="update-the-hub-class"></a>Mettre à jour la classe de concentrateur

Par défaut, lorsqu’un client tente de se connecter au service SignalR, la connexion est accordée en fonction d’un jeton d’accès fourni en interne. Ce jeton d’accès n’est associé à aucune identité authentifiée. Il s’agit en réalité d’un accès anonyme.

Dans cette section, vous allez activer l’authentification réelle en ajoutant l’attribut `Authorize` à la classe de concentrateur, et en ajoutant les méthodes du concentrateur afin de lire le nom d’utilisateur à partir de la réclamation de l’utilisateur authentifié.

1. Ouvrez *Hub\Chat.cs* et ajoutez des références à ces espaces de noms :

    ```csharp
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Authorization;
    ```

2. Mettez à jour le code de concentrateur, tel qu’indiqué ci-dessous. Ce code ajoute l’attribut `Authorize` à la classe `Chat` et utilise l’identité authentifiée de l’utilisateur dans les méthodes du concentrateur. En outre, la méthode `OnConnectedAsync` est ajoutée. Elle consigne un message système dans la salle de conversation à chaque connexion de nouveau client.

    ```csharp
    [Authorize]
    public class Chat : Hub
    {
        public override Task OnConnectedAsync()
        {
            return Clients.All.SendAsync("broadcastMessage", "_SYSTEM_", $"{Context.User.Identity.Name} JOINED");
        }

        // Uncomment this line to only allow user in Microsoft to send message
        //[Authorize(Policy = "Microsoft_Only")]
        public void BroadcastMessage(string message)
        {
            Clients.All.SendAsync("broadcastMessage", Context.User.Identity.Name, message);
        }

        public void Echo(string message)
        {
            var echoMessage = $"{message} (echo from server)";
            Clients.Client(Context.ConnectionId).SendAsync("echo", Context.User.Identity.Name, echoMessage);
        }
    }
    ```

3. Enregistrez vos modifications.

### <a name="update-the-web-client-code"></a>Mettre à jour le code du client web

1. Ouvrez *wwwroot\index.html*, puis remplacez le code d’invite du nom d’utilisateur par le code prenant en charge l’utilisation du cookie renvoyé par le contrôleur d’authentification.

    Supprimez le code suivant de *index.html* :

    ```javascript
    // Get the user name and store it to prepend to messages.
    var username = generateRandomName();
    var promptMessage = 'Enter your name:';
    do {
        username = prompt(promptMessage, username);
        if (!username || username.startsWith('_') || username.indexOf('<') > -1 || username.indexOf('>') > -1) {
            username = '';
            promptMessage = 'Invalid input. Enter your name:';
        }
    } while(!username)
    ```

    Ajoutez le code suivant en lieu et place du code ci-dessus pour utiliser le cookie :

    ```javascript
    // Get the user name cookie.
    function getCookie(key) {
        var cookies = document.cookie.split(';').map(c => c.trim());
        for (var i = 0; i < cookies.length; i++) {
            if (cookies[i].startsWith(key + '=')) return unescape(cookies[i].slice(key.length + 1));
        }
        return '';
    }
    var username = getCookie('githubchat_username');
    ```

2. Juste sous la ligne de code ajoutée pour utiliser le cookie, ajoutez la définition suivante pour la fonction `appendMessage` :

    ```javascript
    function appendMessage(encodedName, encodedMsg) {
        var messageEntry = createMessageEntry(encodedName, encodedMsg);
        var messageBox = document.getElementById('messages');
        messageBox.appendChild(messageEntry);
        messageBox.scrollTop = messageBox.scrollHeight;
    }
    ```

3. Mettez à jour les fonctions `bindConnectionMessage` et `onConnected` avec le code suivant afin d’utiliser `appendMessage`.

    ```javascript
    function bindConnectionMessage(connection) {
        var messageCallback = function(name, message) {
            if (!message) return;
            // Html encode display name and message.
            var encodedName = name;
            var encodedMsg = message.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
            appendMessage(encodedName, encodedMsg);
        };
        // Create a function that the hub can call to broadcast messages.
        connection.on('broadcastMessage', messageCallback);
        connection.on('echo', messageCallback);
        connection.onclose(onConnectionError);
    }

    function onConnected(connection) {
        console.log('connection started');
        document.getElementById('sendmessage').addEventListener('click', function (event) {
            // Call the broadcastMessage method on the hub.
            if (messageInput.value) {
                connection
                    .invoke('broadcastMessage', messageInput.value)
                    .catch(e => appendMessage('_BROADCAST_', e.message));
            }

            // Clear text box and reset focus for next comment.
            messageInput.value = '';
            messageInput.focus();
            event.preventDefault();
        });
        document.getElementById('message').addEventListener('keypress', function (event) {
            if (event.keyCode === 13) {
                event.preventDefault();
                document.getElementById('sendmessage').click();
                return false;
            }
        });
        document.getElementById('echo').addEventListener('click', function (event) {
            // Call the echo method on the hub.
            connection.send('echo', messageInput.value);

            // Clear text box and reset focus for next comment.
            messageInput.value = '';
            messageInput.focus();
            event.preventDefault();
        });
    }
    ```

4. Dans la partie inférieure de *index.html*, mettez à jour le gestionnaire d’erreurs `connection.start()` tel qu’indiqué ci-dessous afin d’inviter l’utilisateur à se connecter.

    ```javascript
    connection.start()
        .then(function () {
            onConnected(connection);
        })
        .catch(function (error) {
            if (error) {
                if (error.message) {
                    console.error(error.message);
                }
                if (error.statusCode && error.statusCode === 401) {
                    appendMessage('_BROADCAST_', 'You\'re not logged in. Click <a href="/login">here</a> to login with GitHub.');
                }
            }
        });
    ```

5. Enregistrez vos modifications.

## <a name="build-and-run-the-app-locally"></a>Générer et exécuter l’application localement

1. Enregistrez les modifications sur l’ensemble des fichiers.

2. Générez l’application à l’aide de l’interface CLI .NET Core, exécutez la commande suivante dans l’interpréteur de commandes :

    ```dotnetcli
    dotnet build
    ```

3. Une fois la version terminée, exécutez la commande suivante afin d’exécuter l’application web en local :

    ```dotnetcli
    dotnet run
    ```

    Par défaut, l’application est hébergée en local sur le port 5000 :

    ```output
    E:\Testing\chattest>dotnet run
    Hosting environment: Production
    Content root path: E:\Testing\chattest
    Now listening on: http://localhost:5000
                    Application started. Press Ctrl+C to shut down.
    ```

4. Lancez une fenêtre de navigateur, puis accédez à `http://localhost:5000`. Cliquez sur le lien **ici** dans la partie supérieure afin de vous connecter avec GitHub.

    ![Flux OAuth complet hébergé dans Azure](media/signalr-concept-authenticate-oauth/signalr-oauth-complete-azure.png)

    Vous serez invité à autoriser l’accès de l’application de conversation à votre compte GitHub. Cliquez sur le bouton **Autoriser**.

    ![Autoriser l’application OAuth](media/signalr-concept-authenticate-oauth/signalr-authorize-oauth-app.png)

    Vous serez redirigé vers l’application de conversation et connecté avec votre nom de compte GitHub. L’application web a identifié le nom de votre compte en vous authentifiant à l’aide de la nouvelle authentification ajoutée.

    ![Compte identifié](media/signalr-concept-authenticate-oauth/signalr-oauth-account-identified.png)

    Maintenant que l’application de conversation exécute l’authentification avec GitHub et stocke les données d’authentification sous forme de cookies, vous avez intérêt à la déployer sur Azure afin de permettre à d’autres utilisateurs de s’authentifier avec leurs comptes et de communiquer à partir d’autres stations de travail.

## <a name="deploy-the-app-to-azure"></a>Déploiement de l’application dans Azure

Préparer votre environnement pour Azure CLI :

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Dans cette section, vous allez utiliser l’interface de ligne de commande (CLI) Azure afin de créer une application web dans [Azure App Service](../app-service/index.yml) pour héberger votre application ASP.NET dans Azure. L’application web sera configurée pour l’utilisation du déploiement local Git. L’application web sera également configurée avec votre chaîne de connexion SignalR, les secrets d’application GitHub OAuth et un utilisateur de déploiement.

Lorsque vous créez les ressources suivantes, veillez à utiliser le groupe de ressources hébergeant la ressource de votre service SignalR. Cette approche simplifie grandement l’étape de nettoyage que vous exécuterez au moment de la suppression de l’ensemble des ressources. Les exemples fournis supposent que vous avez utilisé le nom de groupe recommandé dans les didacticiels précédents, *SignalRTestResources*.

### <a name="create-the-web-app-and-plan"></a>Créer l’application web et le plan

Copiez le texte dédié aux commandes ci-dessous et mettez à jour les paramètres. Collez le script mis à jour dans l’instance Azure Cloud Shell, puis appuyez sur **Entrée** afin de créer un plan App Service et une application web.

```azurecli-interactive
#========================================================================
#=== Update these variable for your resource group name.              ===
#========================================================================
ResourceGroupName=SignalRTestResources

#========================================================================
#=== Update these variable for your web app.                          ===
#========================================================================
WebAppName=myWebAppName
WebAppPlan=myAppServicePlanName

# Create an App Service plan.
az appservice plan create --name $WebAppPlan --resource-group $ResourceGroupName \
    --sku FREE

# Create the new Web App
az webapp create --name $WebAppName --resource-group $ResourceGroupName \
    --plan $WebAppPlan
```

| Paramètre | Description |
| -------------------- | --------------- |
| ResourceGroupName | Ce nom de groupe de ressources a été conseillé dans les didacticiels précédents. Il est judicieux de grouper l’ensemble des ressources de didacticiels. Utilisez le groupe de ressources employé dans les didacticiels précédents. |
| WebAppPlan | Saisissez un nouveau nom unique de plan App Service. |
| WebAppName | Ce sera le nom de la nouvelle application web et une portion de l’URL. Utilisez un nom unique. Par exemple, signalrtestwebapp22665120.   |

### <a name="add-app-settings-to-the-web-app"></a>Ajouter des paramètres d’application à l’application web

Dans cette section, vous allez ajouter des paramètres d’application aux composants suivants :

* Chaîne de connexion de la ressource du service SignalR
* ID client de l’application GitHub OAuth
* Clé secrète du client de l’application GitHub OAuth

Copiez le texte dédié aux commandes ci-dessous et mettez à jour les paramètres. Collez le script mis à jour dans l’instance Azure Cloud Shell, puis appuyez sur **Entrée** pour ajouter les paramètres de l’application :

```azurecli-interactive
#========================================================================
#=== Update these variables for your GitHub OAuth App.                ===
#========================================================================
GitHubClientId=1234567890
GitHubClientSecret=1234567890

#========================================================================
#=== Update these variables for your resources.                       ===
#========================================================================
ResourceGroupName=SignalRTestResources
SignalRServiceResource=mySignalRresourcename
WebAppName=myWebAppName

# Get the SignalR primary connection string 
primaryConnectionString=$(az signalr key list --name $SignalRServiceResource \
  --resource-group $ResourceGroupName --query primaryConnectionString -o tsv)

#Add an app setting to the web app for the SignalR connection
az webapp config appsettings set --name $WebAppName \
    --resource-group $ResourceGroupName \
    --settings "Azure__SignalR__ConnectionString=$primaryConnectionString"

#Add the app settings to use with GitHub authentication
az webapp config appsettings set --name $WebAppName \
    --resource-group $ResourceGroupName \
    --settings "GitHubClientId=$GitHubClientId"
az webapp config appsettings set --name $WebAppName \
    --resource-group $ResourceGroupName \
    --settings "GitHubClientSecret=$GitHubClientSecret"
```

| Paramètre | Description |
| -------------------- | --------------- |
| GitHubClientId | Affectez à cette variable l’ID secret du client pour votre application GitHub OAuth. |
| GitHubClientSecret | Affectez à cette variable le mot de passe secret pour votre application GitHub OAuth. |
| ResourceGroupName | Mettez à jour cette variable afin qu’il s’agisse du nom du groupe de ressources utilisé dans la section précédente. |
| SignalRServiceResource | Mettez à jour cette variable avec le nom de la ressource du service SignalR créée dans le démarrage rapide. Par exemple, signalrtestsvc48778624. |
| WebAppName | Mettez à jour cette variable avec le nom de l’application web créée dans la section précédente. |

### <a name="configure-the-web-app-for-local-git-deployment"></a>Configurer l’application web pour le déploiement Git local

Dans l’instance Azure Cloud Shell, collez le script suivant. Ce script crée un nom d’utilisateur et un mot de passe de déploiement à utiliser lors du déploiement de votre code sur l’application web avec Git. Le script configure également l’application web pour le déploiement avec un référentiel local Git, et renvoie l’URL du déploiement Git.

```azurecli-interactive
#========================================================================
#=== Update these variables for your resources.                       ===
#========================================================================
ResourceGroupName=SignalRTestResources
WebAppName=myWebAppName

#========================================================================
#=== Update these variables for your deployment user.                 ===
#========================================================================
DeploymentUserName=myUserName
DeploymentUserPassword=myPassword

# Add the desired deployment user name and password
az webapp deployment user set --user-name $DeploymentUserName \
    --password $DeploymentUserPassword

# Configure Git deployment and note the deployment URL in the output
az webapp deployment source config-local-git --name $WebAppName \
    --resource-group $ResourceGroupName \
    --query [url] -o tsv
```

| Paramètre | Description |
| -------------------- | --------------- |
| DeploymentUserName | Choisissez un nouveau nom d’utilisateur de déploiement. |
| DeploymentUserPassword | Choisissez un mot de passe pour le nouvel utilisateur de déploiement. |
| ResourceGroupName | Utilisez le nom du groupe de ressources employé dans la section précédente. |
| WebAppName | Ce sera le nom de la nouvelle application web créée précédemment. |

Notez l’URL du déploiement Git renvoyée par cette commande. Vous l’utiliserez ultérieurement.

### <a name="deploy-your-code-to-the-azure-web-app"></a>Déployer votre code sur l’application web Azure

Pour déployer votre code, exécutez les commandes suivantes dans l’interpréteur de commandes Git.

1. Accédez à la racine du répertoire de votre projet. Si vous ne possédez pas le projet initialisé avec un référentiel Git, exécutez la commande suivante :

    ```bash
    git init
    ```

2. Ajoutez une instance distante pour l’URL de déploiement Git notée précédemment :

    ```bash
    git remote add Azure <your git deployment url>
    ```

3. Indexez l’ensemble des fichiers dans le référentiel initialisé et ajoutez une validation.

    ```bash
    git add -A
    git commit -m "init commit"
    ```

4. Déployez votre code sur l’application web dans Azure.

    ```bash
    git push Azure main
    ```

    Vous serez invité à vous authentifier pour déployer le code sur Azure. Saisissez le nom d’utilisateur et le mot de passe de l’utilisateur de déploiement créé plus haut.

### <a name="update-the-github-oauth-app"></a>Mettre à jour l’application GitHub OAuth

Pour finir, vous devez mettre à jour l’**URL de la page d’accueil** et l’**URL de rappel d’autorisation** de l’application GitHub OAuth afin de pointer vers la nouvelle application hébergée.

1. Ouvrez [https://github.com](https://github.com) dans un navigateur et dans votre compte, accédez à **Paramètres** > **Paramètres de développeur** > **Oauth Apps (Applications OAuth)** .

2. Cliquez sur votre page d’authentification et mettez à jour l’**URL de la page d’accueil** et l’**URL de rappel d’autorisation** tel que représenté ci-dessous :

    | Paramètre | Exemple |
    | ------- | ------- |
    | URL de la page d’accueil | `https://signalrtestwebapp22665120.azurewebsites.net/home` |
    | URL de rappel d’autorisation | `https://signalrtestwebapp22665120.azurewebsites.net/signin-github` |

3. Accédez à l’URL de votre application web, puis testez l’application.

    ![Flux OAuth complet hébergé dans Azure](media/signalr-concept-authenticate-oauth/signalr-oauth-complete-azure.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez d’exécuter le tutoriel suivant, vous pouvez conserver les ressources créées dans ce démarrage rapide.

Sinon, si l’exemple d’application de démarrage rapide était votre dernière opération, vous pouvez supprimer les ressources Azure créées dans ce démarrage rapide afin d’éviter tout frais.

> [!IMPORTANT]
> La suppression d’un groupe de ressources est définitive ; le groupe de ressources et l’ensemble des ressources qu’il contient sont supprimés de manière permanente. Veillez à ne pas supprimer accidentellement des ressources ou un groupe de ressources incorrects. Si vous avez créé les ressources pour l’hébergement de cet exemple dans un groupe de ressources existant contenant des ressources que vous souhaitez conserver, vous pouvez supprimer chaque ressource individuellement à partir de son panneau respectif, au lieu de supprimer l’intégralité du groupe de ressources.

Connectez-vous au [Portail Azure](https://portal.azure.com) et cliquez sur **Groupes de ressources**.

Dans la zone de texte **Filtrer par nom.** , saisissez le nom de votre groupe de ressources. Les instructions relatives à cet article utilisaient un groupe de ressources nommé *SignalRTestResources*. Sur votre groupe de ressources dans la liste des résultats, cliquez sur **...** , puis sur **Supprimer le groupe de ressources**.

![DELETE](./media/signalr-concept-authenticate-oauth/signalr-delete-resource-group.png)

Il vous sera demandé de confirmer la suppression du groupe de ressources. Saisissez le nom de votre groupe de ressources pour confirmer, puis cliquez sur **Supprimer**.

Après quelques instants, le groupe de ressources et toutes les ressources qu’il contient sont supprimés.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez ajouté l’authentification avec OAuth afin de fournir une meilleure approche de l’authentification avec le service Azure SignalR. Pour en savoir plus sur l’utilisation d’Azure SignalR Server, accédez aux exemples Azure CLI pour le service SignalR.

> [!div class="nextstepaction"]
> [Exemples CLI Azure SignalR](./signalr-reference-cli.md)
