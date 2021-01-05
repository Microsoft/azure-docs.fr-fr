---
title: 'Tutoriel : Créer une application monopage JavaScript qui utilise la Plateforme d’identités Microsoft pour l’authentification | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce tutoriel, vous allez créer une application monopage JavaScript qui utilise la plateforme d’identités Microsoft pour connecter les utilisateurs et obtenir un jeton d’accès pour appeler l’API Microsoft Graph en leur nom.
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 08/06/2020
ms.author: nacanuma
ms.custom: aaddev, identityplatformtop40, devx-track-js
ms.openlocfilehash: 4eb3c2905f3c1ccfa63da1bb4a8c81decdbc2f2b
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507725"
---
# <a name="tutorial-sign-in-users-and-call-the-microsoft-graph-api-from-a-javascript-single-page-application-spa"></a>Tutoriel : Connecter les utilisateurs et appeler l’API Microsoft Graph à partir d’une application monopage (SPA) JavaScript

Dans ce tutoriel, vous créez une application monopage (SPA) JavaScript qui connecte les utilisateurs et appelle Microsoft Graph en utilisant le flux implicite. L’application monopage que vous créez utilise la bibliothèque d’authentification Microsoft (MSAL) pour JavaScript v1.0.

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Créer un projet JavaScript avec `npm`.
> * Inscrire l’application dans le Portail Azure
> * Ajouter du code pour prendre en charge la connexion et la déconnexion des utilisateurs
> * Ajouter du code pour appeler l’API Microsoft Graph
> * Test de l'application

>[!TIP]
> Ce tutoriel utilise MSAL.js v1.x, qui est limité à l’utilisation du flux d’octroi implicite pour les applications monopages. Nous recommandons que toutes les nouvelles applications utilisent plutôt [MSAL.js 2.x et le flux de code d’autorisation avec la prise en charge de PKCE et CORS](tutorial-v2-javascript-auth-code.md).

## <a name="prerequisites"></a>Prérequis

* [Node.js](https://nodejs.org/en/download/) pour l’exécution d’un serveur web local
* [Visual Studio Code](https://code.visualstudio.com/download) ou un autre éditeur pour la modification des fichiers projet
* Un navigateur web moderne. **Internet Explorer** n’est **pas pris en charge** par l’application que vous générez dans ce tutoriel, en raison de l’utilisation par l’application des conventions [ES6](http://www.ecma-international.org/ecma-262/6.0/).

## <a name="how-the-sample-app-generated-by-this-guide-works"></a>Fonctionnement de l’exemple d’application de ce guide

![Fonctionnement de l’exemple d’application généré par ce tutoriel](media/active-directory-develop-guidedsetup-javascriptspa-introduction/javascriptspa-intro.svg)

L’exemple d’application créé par ce guide permet à une application monopage (SPA) JavaScript d’interroger l’API Microsoft Graph ou une API web qui accepte les jetons provenant du point de terminaison de la plateforme d’identités Microsoft. Dans ce scénario, une fois qu’un utilisateur s’est connecté, un jeton d’accès est demandé et ajouté aux requêtes HTTP par le biais de l’en-tête d’autorisation. Ce jeton sera utilisé pour acquérir le profil et les e-mails de l’utilisateur via l’**API MS Graph**.

L’acquisition et le renouvellement des jetons sont gérés par la [bibliothèque d’authentification Microsoft (MSAL) pour JavaScript](https://github.com/AzureAD/microsoft-authentication-library-for-js).

## <a name="set-up-your-web-server-or-project"></a>Configurer le serveur web ou projet

> Vous préférez télécharger le projet de cet exemple ? [Téléchargez les fichiers du projet](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/quickstart.zip).
>
> Pour configurer l’exemple de code avant de l’exécuter, passez à l’[étape de configuration](#register-your-application).

## <a name="create-your-project"></a>Créer votre projet

Vérifiez que [Node.js](https://nodejs.org/en/download/) est installé, puis créez un dossier pour héberger votre application. Nous allons y implémenter un serveur web [Express](https://expressjs.com/) simple pour servir votre fichier `index.html`.

1. À l’aide d’un terminal (par exemple, le terminal intégré de Visual Studio Code), localisez votre dossier de projet, puis tapez :

   ```console
   npm init
   ```

2. Installez ensuite les dépendances nécessaires :

   ```console
   npm install express --save
   npm install morgan --save
   ```

1. À présent, créez un fichier .js nommé `server.js`, puis ajoutez le code suivant :

   ```JavaScript
   const express = require('express');
   const morgan = require('morgan');
   const path = require('path');

   //initialize express.
   const app = express();

   // Initialize variables.
   const port = 3000; // process.env.PORT || 3000;

   // Configure morgan module to log all requests.
   app.use(morgan('dev'));

   // Set the front-end folder to serve public assets.
   app.use(express.static('JavaScriptSPA'))

   // Set up a route for index.html.
   app.get('*', function (req, res) {
       res.sendFile(path.join(__dirname + '/index.html'));
   });

   // Start the server.
   app.listen(port);
   console.log('Listening on port ' + port + '...');
   ```

Vous disposez maintenant d’un serveur simple pour servir votre application monopage. La structure de dossiers prévue à la fin de ce tutoriel est la suivante :

![description textuelle de la structure prévue des dossiers d’application monopage](./media/tutorial-v2-javascript-spa/single-page-application-folder-structure.png)

## <a name="create-the-spa-ui"></a>Créer l’interface utilisateur de SPA

1. Créez un fichier `index.html` pour votre application SPA JavaScript. Ce fichier implémente une interface utilisateur générée avec l’**infrastructure Bootstrap 4** et importe des fichiers de script pour la configuration, l’authentification et l’appel d’API.

   Dans le fichier `index.html`, ajoutez le code suivant :

   ```html
   <!DOCTYPE html>
   <html lang="en">
     <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0, shrink-to-fit=no">
       <title>Quickstart | MSAL.JS Vanilla JavaScript SPA</title>

       <!-- msal.js with a fallback to backup CDN -->
       <script type="text/javascript" src="https://alcdn.msauth.net/lib/1.2.1/js/msal.js" integrity="sha384-9TV1245fz+BaI+VvCjMYL0YDMElLBwNS84v3mY57pXNOt6xcUYch2QLImaTahcOP" crossorigin="anonymous"></script>
       <script type="text/javascript">
         if(typeof Msal === 'undefined')document.write(unescape("%3Cscript src='https://alcdn.msftauth.net/lib/1.2.1/js/msal.js' type='text/javascript' integrity='sha384-m/3NDUcz4krpIIiHgpeO0O8uxSghb+lfBTngquAo2Zuy2fEF+YgFeP08PWFo5FiJ' crossorigin='anonymous'%3E%3C/script%3E"));
       </script>

       <!-- adding Bootstrap 4 for UI components  -->
       <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
     </head>
     <body>
       <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
         <a class="navbar-brand" href="/">MS Identity Platform</a>
         <div class="btn-group ml-auto dropleft">
           <button type="button" id="signIn" class="btn btn-secondary" onclick="signIn()">Sign In</button>
           <button type="button" id="signOut" class="btn btn-success d-none" onclick="signOut()">Sign Out</button>
       </div>
       </nav>
       <br>
       <h5 class="card-header text-center">Vanilla JavaScript SPA calling MS Graph API with MSAL.JS</h5>
       <br>
       <div class="row" style="margin:auto" >
       <div id="card-div" class="col-md-3 d-none">
       <div class="card text-center">
         <div class="card-body">
           <h5 class="card-title" id="welcomeMessage">Please sign-in to see your profile and read your mails</h5>
           <div id="profile-div"></div>
           <br>
           <br>
           <button class="btn btn-primary" id="seeProfile" onclick="seeProfile()">See Profile</button>
           <br>
           <br>
           <button class="btn btn-primary d-none" id="readMail" onclick="readMail()">Read Mails</button>
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

       <!-- replace next line with authRedirect.js if you would like to use the redirect flow -->
       <!-- <script type="text/javascript" src="./authRedirect.js"></script>   -->
       <script type="text/javascript" src="./authPopup.js"></script>
       <script type="text/javascript" src="./graph.js"></script>
     </body>
   </html>
   ```

   > [!TIP]
   > Vous pouvez remplacer la version de MSAL.js dans le script précédent par la dernière version publiée sous [Publications MSAL.js](https://github.com/AzureAD/microsoft-authentication-library-for-js/releases).

2. À présent, créez un fichier .js nommé `ui.js`, qui accédera aux éléments DOM et les mettra à jour, puis ajoutez le code suivant :

   ```JavaScript
   // Select DOM elements to work with
   const welcomeDiv = document.getElementById("welcomeMessage");
   const signInButton = document.getElementById("signIn");
   const signOutButton = document.getElementById('signOut');
   const cardDiv = document.getElementById("card-div");
   const mailButton = document.getElementById("readMail");
   const profileButton = document.getElementById("seeProfile");
   const profileDiv = document.getElementById("profile-div");

   function showWelcomeMessage(account) {
     // Reconfiguring DOM elements
     cardDiv.classList.remove('d-none');
     welcomeDiv.innerHTML = `Welcome ${account.name}`;
     signInButton.classList.add('d-none');
     signOutButton.classList.remove('d-none');
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

Avant de poursuivre l’authentification, inscrivez votre application sur **Azure Active Directory**.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire dans lequel vous voulez inscrire une application.
1. Recherchez et sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Inscriptions d’applications** > **Nouvelle inscription**.
1. Entrez un **nom** pour votre application. Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
1. Sous **Types de comptes pris en charge**, sélectionnez **Comptes dans un annuaire organisationnel et comptes personnels Microsoft**.
1. Dans la section **URI de redirection**, sélectionnez la plateforme **Web** dans la liste déroulante, puis définissez la valeur sur l’URL de l’application en fonction de votre serveur web.
1. Sélectionnez **Inscription**.
1. Dans la page **Vue d’ensemble**, notez la valeur de **ID d’application (client)** pour une utilisation ultérieure.
1. Sous **Gérer**, sélectionnez **Authentification**.
1. Dans la section **Octroi implicite**, sélectionnez **Jetons d’ID** et **Jetons d’accès**. Des jetons d’ID et des jetons d’accès sont nécessaires car cette application doit connecter des utilisateurs et appeler une API.
1. Sélectionnez **Enregistrer**.

> ### <a name="set-a-redirect-url-for-nodejs"></a>Définir d’une URL de redirection pour Node.js
>
> Pour Node.js, vous pouvez définir le port du serveur web dans le fichier *server.js*. Ce tutoriel utilise le port 3000, mais vous pouvez utiliser tout autre port disponible.
>
> Pour configurer une URL de redirection dans les informations d’inscription de l’application, retournez dans le volet **Inscription d’application** et effectuez l’une des opérations suivantes :
>
> - Définissez *`http://localhost:3000/`* en tant qu’**URL de redirection**.
> - Si vous utilisez un port TCP personnalisé, utilisez *`http://localhost:<port>/`* (où *\<port>* correspond au numéro du port TCP personnalisé).
>   1. Copiez la valeur d’**URL**.
>   1. Basculez vers le volet **Inscriptions des applications**, collez la valeur en tant **qu’URL de redirection**.
>

### <a name="configure-your-javascript-spa"></a>Configurer une application SPA JavaScript

Créez un fichier .js nommé `authConfig.js`, qui contiendra vos paramètres de configuration pour l’authentification, puis ajoutez le code suivant :

```javascript
  const msalConfig = {
    auth: {
      clientId: "Enter_the_Application_Id_Here",
      authority: "Enter_the_Cloud_Instance_Id_HereEnter_the_Tenant_Info_Here",
      redirectUri: "Enter_the_Redirect_Uri_Here",
    },
    cache: {
      cacheLocation: "sessionStorage", // This configures where your cache will be stored
      storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
    }
  };

  // Add here scopes for id token to be used at MS Identity Platform endpoints.
  const loginRequest = {
    scopes: ["openid", "profile", "User.Read"]
  };

  // Add here scopes for access token to be used at MS Graph API endpoints.
  const tokenRequest = {
    scopes: ["Mail.Read"]
  };
```

 Où :
 - *\<Enter_the_Application_Id_Here>* est l’**ID d’application (client)** de l’application que vous avez inscrite.
 - *\<Enter_the_Cloud_Instance_Id_Here>* est l’instance du cloud Azure. Pour le cloud Azure principal ou mondial, entrez simplement *https://login.microsoftonline.com* . Pour les clouds **nationaux** (par exemple, Chine), consultez [Clouds nationaux](./authentication-national-cloud.md).
 - *\<Enter_the_Tenant_info_here>* est défini sur l’une des options suivantes :
   - Si votre application prend en charge les *Comptes dans cet annuaire organisationnel*, remplacez cette valeur par l’**ID de locataire** ou le **nom du locataire** (par exemple, *contoso.microsoft.com*).
   - Si votre application prend en charge les *Comptes dans un annuaire organisationnel*, remplacez cette valeur par **organizations**.
   - Si votre application prend en charge les *Comptes dans un annuaire organisationnel et comptes personnels Microsoft*, remplacez cette valeur par **common**. Pour limiter la prise en charge aux *Comptes Microsoft personnels uniquement*, remplacez cette valeur par **consumers**.


## <a name="use-the-microsoft-authentication-library-msal-to-sign-in-the-user"></a>Utiliser Microsoft Authentication Library (MSAL) pour connecter l’utilisateur

Créez un fichier .js nommé `authPopup.js`, qui contiendra votre logique d’authentification et d’acquisition de jeton, puis ajoutez le code suivant :

   ```JavaScript
   const myMSALObj = new Msal.UserAgentApplication(msalConfig);

   function signIn() {
     myMSALObj.loginPopup(loginRequest)
       .then(loginResponse => {
         console.log('id_token acquired at: ' + new Date().toString());
         console.log(loginResponse);

         if (myMSALObj.getAccount()) {
           showWelcomeMessage(myMSALObj.getAccount());
         }
       }).catch(error => {
         console.log(error);
       });
   }

   function signOut() {
     myMSALObj.logout();
   }

   function callMSGraph(theUrl, accessToken, callback) {
       var xmlHttp = new XMLHttpRequest();
       xmlHttp.onreadystatechange = function () {
           if (this.readyState == 4 && this.status == 200) {
              callback(JSON.parse(this.responseText));
           }
       }
       xmlHttp.open("GET", theUrl, true); // true for asynchronous
       xmlHttp.setRequestHeader('Authorization', 'Bearer ' + accessToken);
       xmlHttp.send();
   }

   function getTokenPopup(request) {
     return myMSALObj.acquireTokenSilent(request)
       .catch(error => {
         console.log(error);
         console.log("silent token acquisition fails. acquiring token using popup");

         // fallback to interaction when silent call fails
           return myMSALObj.acquireTokenPopup(request)
             .then(tokenResponse => {
               return tokenResponse;
             }).catch(error => {
               console.log(error);
             });
       });
   }

   function seeProfile() {
     if (myMSALObj.getAccount()) {
       getTokenPopup(loginRequest)
         .then(response => {
           callMSGraph(graphConfig.graphMeEndpoint, response.accessToken, updateUI);
           profileButton.classList.add('d-none');
           mailButton.classList.remove('d-none');
         }).catch(error => {
           console.log(error);
         });
     }
   }

   function readMail() {
     if (myMSALObj.getAccount()) {
       getTokenPopup(tokenRequest)
         .then(response => {
           callMSGraph(graphConfig.graphMailEndpoint, response.accessToken, updateUI);
         }).catch(error => {
           console.log(error);
         });
     }
   }
   ```

### <a name="more-information"></a>Informations complémentaires

Quand un utilisateur sélectionne le bouton **Se connecter** pour la première fois, la méthode `signIn` appelle `loginPopup` pour le connecter. Cette méthode ouvre une fenêtre contextuelle avec le *point de terminaison de la plateforme d’identités Microsoft* afin de demander et de valider les informations d’identification de l’utilisateur. Après une connexion réussie, l’utilisateur est redirigé vers la page *index.html* d’origine. Un jeton est reçu, traité par `msal.js`, puis les informations contenues dans le jeton sont mises en cache. Ce jeton est le *jeton d’ID* et contient des informations de base sur l’utilisateur, telles que son nom d’affichage. Si vous envisagez d’utiliser les données fournies par ce jeton à toutes fins que ce soit, vous devez vous assurer que ce jeton est validé par le serveur principal afin de garantir qu’il a été émis pour un utilisateur valide de votre application.

L’application SPA créée dans ce guide appelle `acquireTokenSilent` et/ou `acquireTokenPopup` pour acquérir un *jeton d’accès* utilisé pour demander les informations de profil utilisateur à l’API Microsoft Graph. Si vous avez besoin d’un exemple qui valide le jeton d’ID, consultez [cet exemple d’application](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi-v2 "Exemple GitHub active-directory-javascript-singlepageapp-dotnet-webapi-v2") dans GitHub. Cet exemple utilise une API web ASP.NET pour la validation du jeton.

#### <a name="get-a-user-token-interactively"></a>Obtenir un jeton d’utilisateur de manière interactive

Après la première connexion, il n’est pas souhaitable de demander aux utilisateurs de se réauthentifier chaque fois qu’ils ont besoin d’un jeton pour accéder à une ressource. Par conséquent, *acquireTokenSilent* doit être utilisé pour acquérir des jetons dans la plupart des cas. Il existe toutefois des situations dans lesquelles vous devez forcer les utilisateurs à interagir avec le point de terminaison de la plateforme d’identités Microsoft. Voici quelques exemples :

- Les utilisateurs doivent entrer à nouveau leurs informations d’identification car le mot de passe a expiré.
- Votre application demande l’accès à une ressource et vous avez besoin du consentement de l’utilisateur.
- Une authentification à 2 facteurs est demandée.

L’appel d’*acquireTokenPopup* ouvre une fenêtre contextuelle (ou *acquireTokenRedirect* redirige les utilisateurs vers le point de terminaison de la plateforme d’identités Microsoft). Dans cette fenêtre, les utilisateurs doivent interagir en confirmant leurs informations d’identification, en donnant leur consentement pour la ressource demandée ou en effectuant l’authentification à 2 facteurs.

#### <a name="get-a-user-token-silently"></a>Obtenir un jeton d’utilisateur en mode silencieux

La méthode `acquireTokenSilent` gère l’acquisition et le renouvellement de jetons sans aucune interaction utilisateur. Quand la méthode `loginPopup` (ou `loginRedirect`) est exécutée pour la première fois, c’est en général la méthode `acquireTokenSilent` qui est utilisée pour obtenir les jetons permettant d’accéder aux ressources protégées pour les appels suivants. (Les appels pour les demandes ou les renouvellements de jetons sont effectués en mode silencieux.) `acquireTokenSilent` risque d’échouer dans certains cas. Par exemple, le mot de passe de l’utilisateur a peut-être expiré. Votre application peut gérer cette exception de deux manières :

1. Effectuez immédiatement un appel à `acquireTokenPopup`, ce qui déclenche une invite de connexion utilisateur. Cette méthode est couramment employée avec les applications en ligne dans lesquelles aucun contenu non authentifié n’est disponible pour l’utilisateur. L’exemple créé avec cette installation guidée utilise ce modèle.

1. Les applications peuvent également signaler à l’utilisateur qu’une connexion interactive est requise afin qu’il puisse choisir le bon moment pour se connecter ou que l’application puisse réessayer d’exécuter `acquireTokenSilent` ultérieurement. Cette solution est fréquemment retenue quand l’utilisateur peut se servir d’autres fonctionnalités de l’application sans être perturbé. Par exemple, un contenu non authentifié peut être disponible dans l’application. Dans ce cas, l’utilisateur peut décider quand se connecter pour accéder à la ressource protégée ou pour actualiser les informations obsolètes.

> [!NOTE]
> Ce guide de démarrage rapide utilise les méthodes `loginPopup` et `acquireTokenPopup` par défaut. Si vous utilisez Internet Explorer comme navigateur, il est recommandé d’utiliser les méthodes `loginRedirect` et `acquireTokenRedirect`, en raison d’un [problème connu](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues) lié à la façon dont Internet Explorer gère les fenêtres indépendantes. Si vous souhaitez voir comment obtenir le même résultat à l’aide de `Redirect methods`, suivez [ce lien](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/blob/quickstart/JavaScriptSPA/authRedirect.js).

## <a name="call-the-microsoft-graph-api-by-using-the-token-you-just-acquired"></a>Appeler l’API Microsoft Graph à l’aide du jeton que vous venez d’acquérir

1. Tout d’abord, créez un fichier .js nommé `graphConfig.js`, qui stockera vos points de terminaison REST. Ajoutez le code suivant :

   ```JavaScript
      const graphConfig = {
        graphMeEndpoint: "Enter_the_Graph_Endpoint_Herev1.0/me",
        graphMailEndpoint: "Enter_the_Graph_Endpoint_Herev1.0/me/messages"
      };
   ```

   Où :
   - *\<Enter_the_Graph_Endpoint_Here>* est l’instance de l’API MS Graph. Pour utiliser le point de terminaison global de l’API MS Graph, remplacez simplement cette chaîne par `https://graph.microsoft.com`. Pour des déploiements sur les clouds nationaux, consultez la [documentation de l’API Graph](/graph/deployments).

1. Ensuite, créez un fichier .js nommé `graph.js`, qui effectuera un appel REST à l’API Microsoft Graph, et ajoutez le code suivant :

   ```javascript
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
       .catch(error => console.log(error))
   }
   ```

### <a name="more-information-about-making-a-rest-call-against-a-protected-api"></a>Informations supplémentaires sur l’envoi d’un appel REST à une API protégée

Dans l’exemple d’application créé par ce guide, la méthode `callMSGraph()` est utilisée pour effectuer une requête HTTP `GET` sur une ressource protégée qui exige un jeton. La requête retourne ensuite le contenu à l’appelant. Cette méthode ajoute le jeton acquis dans l’*en-tête d’autorisation HTTP*. Dans l’exemple d’application créé par ce guide, la ressource est le point de terminaison *me* de l’API Microsoft Graph, qui affiche les informations de profil de l’utilisateur.

## <a name="test-your-code"></a>Test de votre code

1. Configurez le serveur pour écouter un port TCP basé sur l’emplacement de votre fichier *index.html*. Pour Node.js, démarrez le serveur web afin d’écouter le port en exécutant les commandes suivantes dans une invite de ligne de commande à partir du dossier d’application :

   ```bash
   npm install
   npm start
   ```
1. Dans votre navigateur, entrez **http://localhost:3000** ou **http://localhost:{port}** , où *port* est le port que votre serveur web écoute. Vous devez voir apparaître le contenu de votre fichier *index.html*, ainsi que le bouton **Se connecter**.

Une fois que le navigateur a chargé votre fichier *index.html*, sélectionnez **Se connecter**. Vous êtes invité à vous connecter avec le point de terminaison de la plateforme d’identités Microsoft :

![La fenêtre de connexion au compte JavaScript SPA](media/active-directory-develop-guidedsetup-javascriptspa-test/javascriptspascreenshot1.png)

### <a name="provide-consent-for-application-access"></a>Accorder les droits d’accès à l’application

La première fois que vous vous connectez à votre application, vous êtes invité à lui accorder l’accès à votre profil et à vous connecter :

![La fenêtre « Autorisations nécessaires »](media/active-directory-develop-guidedsetup-javascriptspa-test/javascriptspaconsent.png)

### <a name="view-application-results"></a>Afficher les résultats de l’application

Une fois connecté, vos informations de profil utilisateur sont retournées dans la réponse de l’API Microsoft Graph qui s’affiche :

![Résultats de l’appel à l’API Graph](media/active-directory-develop-guidedsetup-javascriptspa-test/javascriptsparesults.png)

### <a name="more-information-about-scopes-and-delegated-permissions"></a>Informations supplémentaires sur les étendues et les autorisations déléguées

L’API Microsoft Graph nécessite l’étendue *user.read* pour lire le profil d’un utilisateur. Par défaut, cette étendue est automatiquement ajoutée à toutes les applications inscrites dans le portail d’inscription. D’autres API pour Microsoft Graph ainsi que des API personnalisées pour votre serveur principal peuvent nécessiter des étendues supplémentaires. Par exemple, l’API Microsoft Graph nécessite l’étendue *Mail.Read* afin de lister les e-mails de l’utilisateur.

> [!NOTE]
> L’utilisateur peut être invité à donner des consentements supplémentaires à mesure que vous augmentez le nombre d’étendues.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Approfondissez le développement d’applications monopages sur la plateforme d’identités Microsoft grâce à notre série de scénarios en plusieurs parties.

> [!div class="nextstepaction"]
> [Scénario : Application monopage](scenario-spa-overview.md)
