---
title: 'Tutoriel : Créer une application monopage JavaScript qui utilise le flux de code d’authentification | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce tutoriel, vous allez créer une application monopage JavaScript qui peut connecter des utilisateurs et utiliser le flux de code d’authentification pour obtenir un jeton d’accès à partir de la plateforme d’identités Microsoft et appeler l’API Microsoft Graph.
services: active-directory
author: hahamil
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 07/17/2020
ms.author: hahamil
ms.custom: aaddev, devx-track-js
ms.openlocfilehash: 79fe821e2da494ab8c9e4cb407e2c2b025f75568
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96169101"
---
# <a name="tutorial-sign-in-users-and-call-the-microsoft-graph-api-from-a-javascript-single-page-app-spa-using-auth-code-flow"></a>Tutoriel : Connecter les utilisateurs et appeler l’API Microsoft Graph à partir d’une application monopage (SPA) JavaScript à l’aide du flux de code d’authentification

Dans ce tutoriel, vous créez une application monopage (SPA) JavaScript qui connecte les utilisateurs et appelle Microsoft Graph en utilisant le flux du code d’autorisation avec PKCE. L’application monopage que vous créez utilise la bibliothèque d’authentification Microsoft (MSAL) pour JavaScript v2.0.

Dans ce tutoriel :
> [!div class="checklist"]
> * Effectuer le flux de code d’autorisation OAuth 2.0 avec PKCE.
> * Connecter des comptes personnels Microsoft, ainsi que des comptes professionnels et scolaires.
> * Obtenir un jeton d’accès
> * Appeler l’API Microsoft Graph ou votre propre API qui exige des jetons d’accès obtenus à partir du point de terminaison de la plateforme d’identités Microsoft.

MSAL.js 2.0 offre une amélioration par rapport à MSAL.js 1.0 en prenant en charge le flux de code d’autorisation dans le navigateur au lieu du flux d’octroi implicite. MSAL.js 2.0 ne prend **PAS** en charge le flux implicite.

## <a name="prerequisites"></a>Prérequis

* [Node.js](https://nodejs.org/en/download/) pour l’exécution d’un serveur web local
* [Visual Studio Code](https://code.visualstudio.com/download) ou un autre éditeur de code

## <a name="how-the-tutorial-app-works"></a>Fonctionnement de l’application du tutoriel

:::image type="content" source="media/tutorial-v2-javascript-auth-code/diagram-01-auth-code-flow.png" alt-text="Diagramme montrant le flux de code d’autorisation dans une application monopage":::

L’application que vous créez dans ce tutoriel permet à une application monopage JavaScript d’interroger l’API Microsoft Graph en faisant l’acquisition de jetons de sécurité provenant du point de terminaison de la plateforme d’identités Microsoft. Dans ce scénario, une fois l’utilisateur connecté, un jeton d’accès est demandé et ajouté aux requêtes HTTP dans l’en-tête d’autorisation. L’acquisition et le renouvellement des jetons sont gérés par la bibliothèque d’authentification Microsoft pour JavaScript (MSAL.js).

Ce tutoriel utilise la bibliothèque suivante :

Package de navigateur de la bibliothèque d’authentification Microsoft pour JavaScript v2.0 [msal.js](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

## <a name="get-the-completed-code-sample"></a>Obtenir l’exemple de code complet

Vous préférez plutôt télécharger l’exemple de projet complet de ce tutoriel ? Pour exécuter le projet en utilisant un serveur web local, par exemple Node.js, clonez le dépôt [ms-identity-javascript-v2](https://github.com/Azure-Samples/ms-identity-javascript-v2) :

`git clone https://github.com/Azure-Samples/ms-identity-javascript-v2`

Ensuite, pour configurer l’exemple de code avant de l’exécuter, passez à l’[étape de configuration](#register-your-application).

Pour poursuivre le tutoriel et générer vous-même l’application, passez à la section suivante, [Prérequis](#prerequisites).

## <a name="create-your-project"></a>Créer votre projet

Une fois que vous avez installé [Node.js](https://nodejs.org/en/download/), créez un dossier pour héberger votre application, par exemple *msal-spa-tutorial*.

Ensuite, implémentez un petit serveur web [Express](https://expressjs.com/) pour traiter votre fichier *index.html*.

1. Tout d’abord, accédez à votre répertoire de projet dans le terminal, puis exécutez les commandes `npm` suivantes :
    ```console
    npm init -y
    npm install @azure/msal-browser
    npm install express
    npm install morgan
    npm install yargs
    ```
2. Ensuite, créez un fichier nommé *server.js* et ajoutez le code suivant :

   ```JavaScript
   const express = require('express');
   const morgan = require('morgan');
   const path = require('path');
   const argv = require('yargs')
      .usage('Usage: $0 -p [PORT]')
      .alias('p', 'port')
      .describe('port', '(Optional) Port Number - default is 3000')
      .strict()
      .argv;

   const DEFAULT_PORT = 3000;

   //initialize express.
   const app = express();

   // Initialize variables.
   let port = DEFAULT_PORT; // -p {PORT} || 3000;
   if (argv.p) {
      port = argv.p;
   }

   // Configure morgan module to log all requests.
   app.use(morgan('dev'));

   // Set the front-end folder to serve public assets.
   app.use("/lib", express.static(path.join(__dirname, "../../lib/msal-browser/lib")));

   // Setup app folders
   app.use(express.static('app'));

   // Set up a route for index.html.
   app.get('*', function (req, res) {
      res.sendFile(path.join(__dirname + '/index.html'));
   });

   // Start the server.
   app.listen(port);
   console.log(`Listening on port ${port}...`);
    ```

Vous disposez maintenant d’un petit serveur web pour traiter votre application monopage. Une fois le tutoriel terminé, la structure des fichiers et des dossiers de votre projet doit ressembler à ce qui suit :

```
msal-spa-tutorial/
├── app
│   ├── authConfig.js
│   ├── authPopup.js
│   ├── authRedirect.js
│   ├── graphConfig.js
│   ├── graph.js
│   ├── index.html
│   └── ui.js
└── server.js
```

## <a name="create-the-spa-ui"></a>Créer l’interface utilisateur de SPA

1. Créez un dossier *app* dans le répertoire de votre projet, puis créez-y un fichier *index.html* pour votre application monopage JavaScript. Ce fichier implémente une interface utilisateur générée avec le **framework Bootstrap 4**, et importe des fichiers de script pour la configuration, l’authentification et les appels d’API.

    Dans le fichier *index.html*, ajoutez le code suivant :

    ```html
    <!DOCTYPE html>
    <html lang="en">
      <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0, shrink-to-fit=no">
        <title>Tutorial | MSAL.js JavaScript SPA</title>

        <!-- IE support: add promises polyfill before msal.js  -->
        <script type="text/javascript" src="//cdn.jsdelivr.net/npm/bluebird@3.7.2/js/browser/bluebird.min.js"></script>
        <script type="text/javascript" src="https://alcdn.msauth.net/browser/2.0.0-beta.4/js/msal-browser.js" integrity="sha384-7sxY2tN3GMVE5jXH2RL9AdbO6s46vUh9lUid4yNCHJMUzDoj+0N4ve6rLOmR88yN" crossorigin="anonymous"></script>

        <!-- adding Bootstrap 4 for UI components  -->
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
        <link rel="SHORTCUT ICON" href="https://c.s-microsoft.com/favicon.ico?v2" type="image/x-icon">
      </head>
      <body>
        <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
          <a class="navbar-brand" href="/">Microsoft identity platform</a>
          <div class="btn-group ml-auto dropleft">
              <button type="button" id="SignIn" class="btn btn-secondary" onclick="signIn()">
                Sign In
              </button>
          </div>
        </nav>
        <br>
        <h5 class="card-header text-center">JavaScript SPA calling Microsoft Graph API with MSAL.js</h5>
        <br>
        <div class="row" style="margin:auto" >
        <div id="card-div" class="col-md-3" style="display:none">
        <div class="card text-center">
          <div class="card-body">
            <h5 class="card-title" id="WelcomeMessage">Please sign-in to see your profile and read your mails</h5>
            <div id="profile-div"></div>
            <br>
            <br>
            <button class="btn btn-primary" id="seeProfile" onclick="seeProfile()">See Profile</button>
            <br>
            <br>
            <button class="btn btn-primary" id="readMail" onclick="readMail()">Read Mail</button>
          </div>
        </div>
        </div>
        <br>
        <br>
          <div class="col-md-4">
            <div class="list-group" id="list-tab" role="tablist">
            </div>
          </div>
          <div class="col-md-5">
            <div class="tab-content" id="nav-tabContent">
            </div>
          </div>
        </div>
        <br>
        <br>

        <!-- importing bootstrap.js and supporting js libraries -->
        <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js" integrity="sha384-J6qa4849blE2+poT4WnyKhv5vZF5SrPo0iEjwBvKU7imGFAV0wwj1yYfoRSJoZ+n" crossorigin="anonymous"></script>
        <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossorigin="anonymous"></script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js" integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6" crossorigin="anonymous"></script>

        <!-- importing app scripts (load order is important) -->
        <script type="text/javascript" src="./authConfig.js"></script>
        <script type="text/javascript" src="./graphConfig.js"></script>
        <script type="text/javascript" src="./ui.js"></script>

        <!-- <script type="text/javascript" src="./authRedirect.js"></script>   -->
        <!-- uncomment the above line and comment the line below if you would like to use the redirect flow -->
        <script type="text/javascript" src="./authPopup.js"></script>
        <script type="text/javascript" src="./graph.js"></script>
      </body>
    </html>
    ```

2. Ensuite, toujours dans le dossier *app*, créez un fichier nommé *ui.js* et ajoutez le code suivant. Ce fichier accèdera aux éléments DOM et les mettra à jour.

    ```JavaScript
    // Select DOM elements to work with
    const welcomeDiv = document.getElementById("WelcomeMessage");
    const signInButton = document.getElementById("SignIn");
    const cardDiv = document.getElementById("card-div");
    const mailButton = document.getElementById("readMail");
    const profileButton = document.getElementById("seeProfile");
    const profileDiv = document.getElementById("profile-div");

    function showWelcomeMessage(account) {
        // Reconfiguring DOM elements
        cardDiv.style.display = 'initial';
        welcomeDiv.innerHTML = `Welcome ${account.username}`;
        signInButton.setAttribute("onclick", "signOut();");
        signInButton.setAttribute('class', "btn btn-success")
        signInButton.innerHTML = "Sign Out";
    }

    function updateUI(data, endpoint) {
        console.log('Graph API responded at: ' + new Date().toString());

        if (endpoint === graphConfig.graphMeEndpoint) {
            const title = document.createElement('p');
            title.innerHTML = "<strong>Title: </strong>" + data.jobTitle;
            const email = document.createElement('p');
            email.innerHTML = "<strong>Mail: </strong>" + data.mail;
            const phone = document.createElement('p');
            phone.innerHTML = "<strong>Phone: </strong>" + data.businessPhones[0];
            const address = document.createElement('p');
            address.innerHTML = "<strong>Location: </strong>" + data.officeLocation;
            profileDiv.appendChild(title);
            profileDiv.appendChild(email);
            profileDiv.appendChild(phone);
            profileDiv.appendChild(address);

        } else if (endpoint === graphConfig.graphMailEndpoint) {
            if (data.value.length < 1) {
                alert("Your mailbox is empty!")
            } else {
                const tabList = document.getElementById("list-tab");
                tabList.innerHTML = ''; // clear tabList at each readMail call
                const tabContent = document.getElementById("nav-tabContent");

                data.value.map((d, i) => {
                    // Keeping it simple
                    if (i < 10) {
                        const listItem = document.createElement("a");
                        listItem.setAttribute("class", "list-group-item list-group-item-action")
                        listItem.setAttribute("id", "list" + i + "list")
                        listItem.setAttribute("data-toggle", "list")
                        listItem.setAttribute("href", "#list" + i)
                        listItem.setAttribute("role", "tab")
                        listItem.setAttribute("aria-controls", i)
                        listItem.innerHTML = d.subject;
                        tabList.appendChild(listItem)

                        const contentItem = document.createElement("div");
                        contentItem.setAttribute("class", "tab-pane fade")
                        contentItem.setAttribute("id", "list" + i)
                        contentItem.setAttribute("role", "tabpanel")
                        contentItem.setAttribute("aria-labelledby", "list" + i + "list")
                        contentItem.innerHTML = "<strong> from: " + d.from.emailAddress.address + "</strong><br><br>" + d.bodyPreview + "...";
                        tabContent.appendChild(contentItem);
                    }
                });
            }
        }
    }
    ```

## <a name="register-your-application"></a>Inscrivez votre application

Suivez les étapes décrites dans [Application monopage : inscription d’application](scenario-spa-app-registration.md) pour créer une inscription d’application pour votre SPA.

À l’étape [URI de redirection : MSAL.js 2.0 avec flux de code d’authentification](scenario-spa-app-registration.md#redirect-uri-msaljs-20-with-auth-code-flow), entrez `http://localhost:3000`, l’emplacement par défaut où s’exécute l’application de ce tutoriel.

Si vous souhaitez utiliser un autre port, entrez `http://localhost:<port>`, où `<port>` est votre numéro de port TCP par défaut. Si vous spécifiez un numéro de port autre que `3000`, mettez également à jour *server.js* avec le numéro de port de votre choix.

### <a name="configure-your-javascript-spa"></a>Configurer une application SPA JavaScript

Créez un fichier nommé *authConfig.js* dans le dossier *app* qui contiendra vos paramètres de configuration pour l’authentification, puis ajoutez le code suivant :

```javascript
const msalConfig = {
  auth: {
    clientId: "Enter_the_Application_Id_Here",
    authority: "Enter_the_Cloud_Instance_Id_Here/Enter_the_Tenant_Info_Here",
    redirectUri: "Enter_the_Redirect_Uri_Here",
  },
  cache: {
    cacheLocation: "sessionStorage", // This configures where your cache will be stored
    storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
  }
};

// Add scopes here for ID token to be used at Microsoft identity platform endpoints.
const loginRequest = {
 scopes: ["openid", "profile", "User.Read"]
};

// Add scopes here for access token to be used at Microsoft Graph API endpoints.
const tokenRequest = {
 scopes: ["User.Read", "Mail.Read"]
};
```

Modifiez les valeurs de la section `msalConfig` de la façon suivante :

- `Enter_the_Application_Id_Here`: **ID d’application (client)** de l’application que vous avez inscrite.
- `Enter_the_Cloud_Instance_Id_Here`: instance cloud Azure dans laquelle votre application est inscrite.
  - Pour le cloud Azure principal (ou *mondial*), entrez `https://login.microsoftonline.com`.
  - Pour les clouds **nationaux** (par exemple la Chine), vous trouverez les valeurs appropriées dans [Clouds nationaux](authentication-national-cloud.md).
- `Enter_the_Tenant_info_here` doit prendre l’une des valeurs suivantes :
  - Si votre application prend en charge les *comptes dans cet annuaire organisationnel*, remplacez cette valeur par l’**ID de locataire** ou le **Nom du locataire**. Par exemple : `contoso.microsoft.com`.
  - Si votre application prend en charge les *comptes dans un annuaire organisationnel*, remplacez cette valeur par `organizations`.
  - Si votre application prend en charge les *comptes dans un annuaire organisationnel et comptes personnels Microsoft*, remplacez cette valeur par `common`.
  - Pour limiter la prise en charge aux *comptes Microsoft personnels uniquement*, remplacez cette valeur par `consumers`.
- `Enter_the_Redirect_Uri_Here` a la valeur `http://localhost:3000`.

La valeur `authority` dans votre fichier *authConfig.js* doit ressembler à ce qui suit si vous utilisez le cloud Azure mondial :

```javascript
authority: "https://login.microsoftonline.com/common",
```

Toujours dans le dossier *app*, créez un fichier nommé *graphConfig.js*. Ajoutez le code suivant pour fournir à votre application les paramètres de configuration pour l’appel de l’API Microsoft Graph :

```javascript
// Add the endpoints here for Microsoft Graph API services you'd like to use.
const graphConfig = {
    graphMeEndpoint: "Enter_the_Graph_Endpoint_Here/v1.0/me",
    graphMailEndpoint: "Enter_the_Graph_Endpoint_Here/v1.0/me/messages"
};
```

Modifiez les valeurs de la section `graphConfig` comme décrit ci-dessous :

- `Enter_the_Graph_Endpoint_Here` est l’instance de l’API Microsoft Graph avec laquelle l’application doit communiquer.
  - Pour le point de terminaison de l’API Microsoft Graph **mondial**, remplacez les deux instances de cette chaîne par `https://graph.microsoft.com`.
  - Pour les points de terminaison dans les déploiements dans des clouds **nationaux**, consultez [Déploiements dans des clouds nationaux](/graph/deployments) dans la documentation Microsoft Graph.

Les valeurs `graphMeEndpoint` et `graphMailEndpoint` dans votre fichier *graphConfig.js* doivent ressembler à ce qui suit si vous utilisez le point de terminaison mondial :

```javascript
graphMeEndpoint: "https://graph.microsoft.com/v1.0/me",
graphMailEndpoint: "https://graph.microsoft.com/v1.0/me/messages"
```

## <a name="use-microsoft-authentication-library-msal-to-sign-in-user"></a>Utiliser la bibliothèque d’authentification Microsoft (MSAL) pour connecter l’utilisateur

### <a name="pop-up"></a>Fenêtre contextuelle

Dans le dossier *app*, créez un fichier nommé *authPopup.js*, puis ajoutez le code d’authentification et d’acquisition de jeton suivant pour la fenêtre contextuelle de connexion :

```JavaScript
// Create the main myMSALObj instance
// configuration parameters are located at authConfig.js
const myMSALObj = new msal.PublicClientApplication(msalConfig);

let username = "";

function loadPage() {
    /**
     * See here for more info on account retrieval:
     * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-common/docs/Accounts.md
     */
    const currentAccounts = myMSALObj.getAllAccounts();
    if (currentAccounts === null) {
        return;
    } else if (currentAccounts.length > 1) {
        // Add choose account code here
        console.warn("Multiple accounts detected.");
    } else if (currentAccounts.length === 1) {
        username = currentAccounts[0].username;
        showWelcomeMessage(currentAccounts[0]);
    }
}

function handleResponse(resp) {
    if (resp !== null) {
        username = resp.account.username;
        showWelcomeMessage(resp.account);
    } else {
        loadPage();
    }
}

function signIn() {
    myMSALObj.loginPopup(loginRequest).then(handleResponse).catch(error => {
        console.error(error);
    });
}

function signOut() {
    const logoutRequest = {
        account: myMSALObj.getAccountByUsername(username)
    };

    myMSALObj.logout(logoutRequest);
}

function getTokenPopup(request) {
    /**
     * See here for more info on account retrieval:
     * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-common/docs/Accounts.md
     */
    request.account = myMSALObj.getAccountByUsername(username);
    return myMSALObj.acquireTokenSilent(request).catch(error => {
        console.warn("silent token acquisition fails. acquiring token using redirect");
        if (error instanceof msal.InteractionRequiredAuthError) {
            // fallback to interaction when silent call fails
            return myMSALObj.acquireTokenPopup(request).then(tokenResponse => {
                console.log(tokenResponse);

                return tokenResponse;
            }).catch(error => {
                console.error(error);
            });
        } else {
            console.warn(error);
        }
    });
}

function seeProfile() {
    getTokenPopup(loginRequest).then(response => {
        callMSGraph(graphConfig.graphMeEndpoint, response.accessToken, updateUI);
        profileButton.classList.add('d-none');
        mailButton.classList.remove('d-none');
    }).catch(error => {
        console.error(error);
    });
}

function readMail() {
    getTokenPopup(tokenRequest).then(response => {
        callMSGraph(graphConfig.graphMailEndpoint, response.accessToken, updateUI);
    }).catch(error => {
        console.error(error);
    });
}

loadPage();
```

### <a name="redirect"></a>Rediriger

Créez un fichier nommé *authRedirect.js* dans le dossier *app*, puis ajoutez le code d’authentification et d’acquisition de jeton suivant pour la redirection de la connexion :

```javascript
// Create the main myMSALObj instance
// configuration parameters are located at authConfig.js
const myMSALObj = new msal.PublicClientApplication(msalConfig);

let accessToken;
let username = "";

// Redirect: once login is successful and redirects with tokens, call Graph API
myMSALObj.handleRedirectPromise().then(handleResponse).catch(err => {
    console.error(err);
});

function handleResponse(resp) {
    if (resp !== null) {
        username = resp.account.username;
        showWelcomeMessage(resp.account);
    } else {
        /**
         * See here for more info on account retrieval:
         * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-common/docs/Accounts.md
         */
        const currentAccounts = myMSALObj.getAllAccounts();
        if (currentAccounts === null) {
            return;
        } else if (currentAccounts.length > 1) {
            // Add choose account code here
            console.warn("Multiple accounts detected.");
        } else if (currentAccounts.length === 1) {
            username = currentAccounts[0].username;
            showWelcomeMessage(currentAccounts[0]);
        }
    }
}

function signIn() {
    myMSALObj.loginRedirect(loginRequest);
}

function signOut() {
    const logoutRequest = {
        account: myMSALObj.getAccountByUsername(username)
    };

    myMSALObj.logout(logoutRequest);
}

function getTokenRedirect(request) {
    /**
     * See here for more info on account retrieval:
     * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-common/docs/Accounts.md
     */
    request.account = myMSALObj.getAccountByUsername(username);
    return myMSALObj.acquireTokenSilent(request).catch(error => {
            console.warn("silent token acquisition fails. acquiring token using redirect");
            if (error instanceof msal.InteractionRequiredAuthError) {
                // fallback to interaction when silent call fails
                return myMSALObj.acquireTokenRedirect(request);
            } else {
                console.warn(error);
            }
        });
}

function seeProfile() {
    getTokenRedirect(loginRequest).then(response => {
        callMSGraph(graphConfig.graphMeEndpoint, response.accessToken, updateUI);
        profileButton.classList.add('d-none');
        mailButton.classList.remove('d-none');
    }).catch(error => {
        console.error(error);
    });
}

function readMail() {
    getTokenRedirect(tokenRequest).then(response => {
        callMSGraph(graphConfig.graphMailEndpoint, response.accessToken, updateUI);
    }).catch(error => {
        console.error(error);
    });
}
```

### <a name="how-the-code-works"></a>Fonctionnement du code

Quand un utilisateur sélectionne le bouton **Sign In** pour la première fois, la méthode `signIn` appelle `loginPopup` pour le connecter. La méthode `loginPopup` ouvre une fenêtre contextuelle avec le *point de terminaison de la plateforme d’identités Microsoft* afin de demander et de valider les informations d’identification de l’utilisateur. Après une connexion réussie, *msal.js* lance le [flux du code d’autorisation](v2-oauth2-auth-code-flow.md).

Un code d’autorisation protégé par PKCE est alors envoyé au point de terminaison de jeton protégé par CORS en vue de l’échange de jetons. Un jeton d’ID, un jeton d’accès et un jeton d’actualisation sont reçus par votre application et traités par *msal.js*, et les informations contenues dans le jeton sont mises en cache.

Le jeton d’ID contient des informations générales sur l’utilisateur, comme son nom d’affichage. Si vous envisagez d’utiliser les données fournies par le jeton d’ID, votre serveur back-end *doit* le valider afin de garantir qu’il a été émis pour un utilisateur autorisé de votre application.

Le jeton d’accès a une durée de vie limitée et expire au bout de 24 heures. Le jeton d’actualisation peut être utilisé pour acquérir de nouveaux jetons d’accès en mode silencieux.

L’application monopage que vous avez créée dans ce tutoriel appelle `acquireTokenSilent` et/ou `acquireTokenPopup` pour acquérir un *jeton d’accès* utilisé pour demander les informations de profil utilisateur à l’API Microsoft Graph. Pour obtenir un exemple de validation du jeton d’ID, consultez l’exemple d’application [active-directory-javascript-singlepageapp-dotnet-webapi-v2](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi-v2) sur GitHub. Cet exemple utilise une API web ASP.NET pour la validation du jeton.

#### <a name="get-a-user-token-interactively"></a>Obtenir un jeton d’utilisateur de manière interactive

Après la première connexion, votre application ne doit pas demander aux utilisateurs de se réauthentifier (autrement dit, demander un jeton) chaque fois qu’ils ont besoin d’accéder à une ressource protégée. Pour empêcher ces requêtes de réauthentification, appelez `acquireTokenSilent`. Il existe toutefois certaines situations dans lesquelles vous aurez à forcer les utilisateurs à interagir avec le point de terminaison de la plateforme d’identités Microsoft. Par exemple :

- Les utilisateurs doivent entrer à nouveau leurs informations d’identification car le mot de passe a expiré.
- Votre application demande l’accès à une ressource et vous avez besoin du consentement de l’utilisateur.
- Une authentification à 2 facteurs est demandée.

L’appel de `acquireTokenPopup` ouvre une fenêtre contextuelle (ou `acquireTokenRedirect` redirige les utilisateurs vers le point de terminaison de la plateforme d’identités Microsoft). Dans cette fenêtre, les utilisateurs doivent interagir en confirmant leurs informations d’identification, en donnant leur consentement pour la ressource demandée ou en effectuant l’authentification à 2 facteurs.

#### <a name="get-a-user-token-silently"></a>Obtenir un jeton d’utilisateur en mode silencieux

La méthode `acquireTokenSilent` gère l’acquisition et le renouvellement de jetons sans aucune interaction utilisateur. Quand la méthode `loginPopup` (ou `loginRedirect`) est exécutée pour la première fois, c’est en général la méthode `acquireTokenSilent` qui est utilisée pour obtenir les jetons permettant d’accéder aux ressources protégées pour les appels suivants. (Les appels pour les demandes ou les renouvellements de jetons sont effectués en mode silencieux.) `acquireTokenSilent` risque d’échouer dans certains cas. Par exemple, le mot de passe de l’utilisateur a peut-être expiré. Votre application peut gérer cette exception de deux manières :

1. Effectuez immédiatement un appel à `acquireTokenPopup` pour déclencher une invite de connexion utilisateur. Cette méthode est couramment employée avec les applications en ligne dans lesquelles aucun contenu non authentifié n’est disponible pour l’utilisateur. L’exemple créé avec cette installation guidée utilise ce modèle.
1. Signalez visuellement à l’utilisateur qu’une connexion interactive est requise afin qu’il puisse choisir le bon moment pour se connecter ou que l’application puisse réessayer d’exécuter `acquireTokenSilent` ultérieurement. Cette technique est fréquemment retenue quand l’utilisateur peut se servir d’autres fonctionnalités de l’application sans être perturbé. Par exemple, un contenu non authentifié peut être disponible dans l’application. Dans ce cas, l’utilisateur peut décider quand se connecter pour accéder à la ressource protégée ou pour actualiser les informations obsolètes.

> [!NOTE]
> Ce tutoriel utilise les méthodes `loginPopup` et `acquireTokenPopup` par défaut. Si vous utilisez Internet Explorer, nous vous recommandons d’utiliser les méthodes `loginRedirect` et `acquireTokenRedirect` en raison d’un [problème connu](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues) avec Internet Explorer et les fenêtres contextuelles. Pour obtenir un exemple d’obtention du même résultat à l’aide de méthodes de redirection, consultez [*authRedirect.js*](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/blob/quickstart/JavaScriptSPA/authRedirect.js) sur GitHub.

## <a name="call-the-microsoft-graph-api"></a>Appeler l’API Microsoft Graph

Créez un fichier nommé *graph.js* dans le dossier *app*, puis ajoutez le code suivant pour effectuer des appels REST à l’API Microsoft Graph :

```javascript
// Helper function to call Microsoft Graph API endpoint
// using authorization bearer token scheme
function callMSGraph(endpoint, token, callback) {
    const headers = new Headers();
    const bearer = `Bearer ${token}`;

    headers.append("Authorization", bearer);

    const options = {
        method: "GET",
        headers: headers
    };

    console.log('request made to Graph API at: ' + new Date().toString());

    fetch(endpoint, options)
        .then(response => response.json())
        .then(response => callback(response, endpoint))
        .catch(error => console.log(error));
}
```

Dans l’exemple d’application créé dans ce tutoriel, la méthode `callMSGraph()` est utilisée pour effectuer une requête HTTP `GET` sur une ressource protégée qui exige un jeton. La requête retourne ensuite le contenu à l’appelant. Cette méthode ajoute le jeton acquis dans l’*en-tête d’autorisation HTTP*. Dans l’exemple d’application créé dans ce tutoriel, la ressource protégée est le point de terminaison *me* de l’API Microsoft Graph, qui affiche les informations de profil de l’utilisateur connecté.

## <a name="test-your-application"></a>Tester votre application

Vous avez terminé la création de l’application, et êtes maintenant prêt à lancer le serveur web Node.js et à tester les fonctionnalités de l’application.

1. Démarrez le serveur web Node.js en exécutant la commande suivante à partir de la racine de votre dossier de projet :

   ```console
   npm start
   ```
1. Dans votre navigateur, accédez à `http://localhost:3000` ou `http://localhost:<port>`, où `<port>` est le port sur lequel votre serveur web est à l’écoute. Vous devez voir apparaître le contenu de votre fichier *index.html*, ainsi que le bouton **Se connecter**.

### <a name="sign-in-to-the-application"></a>Se connecter à l’application

Une fois que le navigateur a chargé votre fichier *index.html*, sélectionnez **Se connecter**. Vous êtes invité à vous connecter avec le point de terminaison de la plateforme d’identités Microsoft :

:::image type="content" source="media/tutorial-v2-javascript-auth-code/spa-01-signin-dialog.png" alt-text="Navigateur web affichant la boîte de dialogue de connexion":::

### <a name="provide-consent-for-application-access"></a>Accorder les droits d’accès à l’application

La première fois que vous vous connectez à votre application, vous êtes invité à lui accorder l’accès à votre profil et à vous connecter :

:::image type="content" source="media/tutorial-v2-javascript-auth-code/spa-02-consent-dialog.png" alt-text="Boîte de dialogue de contenu affichée dans le navigateur web":::

Si vous consentez à accorder les autorisations demandées, l’application web affiche votre nom d’utilisateur, ce qui signifie que la connexion a réussi :

:::image type="content" source="media/tutorial-v2-javascript-auth-code/spa-03-signed-in.png" alt-text="Résultats d’une connexion réussie dans le navigateur web":::

### <a name="call-the-graph-api"></a>Appeler l’API Graph

Une fois connecté, sélectionnez **See Profile** pour afficher les informations de profil utilisateur retournées dans la réponse à l’appel à l’API Microsoft Graph :

:::image type="content" source="media/tutorial-v2-javascript-auth-code/spa-04-see-profile.png" alt-text="Informations de profil de Microsoft Graph affichées dans le navigateur":::

### <a name="more-information-about-scopes-and-delegated-permissions"></a>Informations supplémentaires sur les étendues et les autorisations déléguées

L’API Microsoft Graph nécessite l’étendue *user.read* pour lire le profil d’un utilisateur. Par défaut, cette étendue est automatiquement ajoutée à toutes les applications inscrites dans le Portail Azure. D’autres API pour Microsoft Graph ainsi que des API personnalisées pour votre serveur principal peuvent nécessiter des étendues supplémentaires. Par exemple, l’API Microsoft Graph nécessite l’étendue *Mail.Read* afin de lister l’e-mail de l’utilisateur.

À mesure que vous ajoutez des étendues, vos utilisateurs peuvent être invités à accorder un consentement supplémentaire pour les étendues ajoutées.

Si une API back-end ne nécessite pas d’étendue (ce qui n’est pas recommandé), vous pouvez utiliser `clientId` en tant qu’étendue dans les appels pour acquérir des jetons.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Si vous souhaitez approfondir le développement d’applications monopages JavaScript sur la plateforme d’identités Microsoft, consultez notre série de scénarios en plusieurs parties :

> [!div class="nextstepaction"]
> [Scénario : Application monopage](scenario-spa-overview.md)