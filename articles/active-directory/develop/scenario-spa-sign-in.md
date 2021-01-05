---
title: Connexion et déconnexion d’une application monopage - Plateforme d’identités Microsoft | Azure
description: Apprendre à créer une application monopage (connexion)
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 02/11/2020
ms.author: nacanuma
ms.custom: aaddev
ms.openlocfilehash: 2a73af0a0488043d31722b4dc46ca19530cf34ac
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94443770"
---
# <a name="single-page-application-sign-in-and-sign-out"></a>Application monopage : Se connecter et se déconnecter

Découvrez comment ajouter une connexion au code de votre application monopage.

Pour obtenir des jetons d’accès aux API de votre application, vous avez besoin d’un contexte d’utilisateur authentifié. Vous pouvez connecter des utilisateurs à votre application dans MSAL.js de deux manières :

* [Fenêtre contextuelle](#sign-in-with-a-pop-up-window), au moyen de la méthode `loginPopup`
* [Redirection](#sign-in-with-redirect), au moyen de la méthode `loginRedirect`

Vous pouvez également passer les étendues des API, pour lesquelles l’utilisateur doit donner son consentement au moment de la connexion.

> [!NOTE]
> Si votre application a déjà accès à un contexte d’utilisateur authentifié ou à un jeton d’ID, vous pouvez ignorer l’étape de connexion et obtenir directement des jetons. Pour obtenir des informations détaillées, consultez [Authentification unique sans connexion via msal.js](msal-js-sso.md#sso-without-msaljs-login).

## <a name="choosing-between-a-pop-up-or-redirect-experience"></a>Choix entre une expérience avec fenêtre indépendante ou avec redirection

Vous ne pouvez pas utiliser conjointement ces deux méthodes de fenêtre contextuelle et de redirection dans votre application. Le choix entre la fenêtre contextuelle et la redirection dépend du flux de votre application :

* Si vous ne souhaitez pas que les utilisateurs quittent la page principale de votre application lors de l’authentification, nous vous recommandons d’utiliser la méthode avec fenêtre contextuelle. La redirection de l’authentification se produisant dans une fenêtre contextuelle, l’état de l’application principale est conservé.

* Si les utilisateurs sont soumis à des contraintes imposées par le navigateur ou par des stratégies qui bloquent les fenêtres contextuelles, vous pouvez utiliser la méthode avec redirection. Utilisez la méthode avec redirection dans le navigateur Internet Explorer, car il existe des [problèmes connus avec l’utilisation de fenêtres contextuelles dans Internet Explorer](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser).

## <a name="sign-in-with-a-pop-up-window"></a>Se connecter avec une fenêtre contextuelle


# <a name="javascript-msaljs-2x"></a>[JavaScript (MSAL.js 2.x)](#tab/javascript2)

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

let username = "";

const myMsal = new PublicClientApplication(config);

myMsal.loginPopup(loginRequest)
    .then(function (loginResponse) {
        //login success

        // In case multiple accounts exist, you can select
        const currentAccounts = myMsal.getAllAccounts();
    
        if (currentAccounts === null) {
            // no accounts detected
        } else if (currentAccounts.length > 1) {
            // Add choose account code here
        } else if (currentAccounts.length === 1) {
            username = currentAccounts[0].username;
        }
    
    }).catch(function (error) {
        //login failure
        console.log(error);
    });
```

# <a name="javascript-msaljs-1x"></a>[JavaScript (MSAL.js 1.x)](#tab/javascript1)

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

const myMsal = new UserAgentApplication(config);

myMsal.loginPopup(loginRequest)
    .then(function (loginResponse) {
        //login success
    }).catch(function (error) {
        //login failure
        console.log(error);
    });
```

# <a name="angular"></a>[Angular](#tab/angular)

Le wrapper MSAL Angular vous permet de sécuriser des itinéraires spécifiques dans votre application en ajoutant `MsalGuard` à la définition d’itinéraire. Cette protection appelle la méthode de connexion en cas d'accès à l'itinéraire.

```javascript
// In app-routing.module.ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { ProfileComponent } from './profile/profile.component';
import { MsalGuard } from '@azure/msal-angular';
import { HomeComponent } from './home/home.component';

const routes: Routes = [
  {
    path: 'profile',
    component: ProfileComponent,
    canActivate: [
      MsalGuard
    ]
  },
  {
    path: '',
    component: HomeComponent
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes, { useHash: false })],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

Pour une expérience de fenêtre contextuelle, activez l’option de configuration `popUp`. Vous pouvez également passer les étendues nécessitant un consentement comme suit :

```javascript
// In app.module.ts
@NgModule({
    imports: [
        MsalModule.forRoot({
            auth: {
                clientId: 'your_app_id',
            }
        }, {
            popUp: true,
            consentScopes: ["User.ReadWrite"]
        })
    ]
})
```
---

## <a name="sign-in-with-redirect"></a>Se connecter avec une redirection

# <a name="javascript-msaljs-2x"></a>[JavaScript (MSAL.js 2.x)](#tab/javascript2)

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

let username = "";

const myMsal = new PublicClientApplication(config);

function handleResponse(response) {
    //handle redirect response

    // In case multiple accounts exist, you can select
    const currentAccounts = myMsal.getAllAccounts();

    if (currentAccounts === null) {
        // no accounts detected
    } else if (currentAccounts.length > 1) {
        // Add choose account code here
    } else if (currentAccounts.length === 1) {
        username = currentAccounts[0].username;
    }
}

myMsal.handleRedirectPromise(handleResponse);

myMsal.loginRedirect(loginRequest);
```

# <a name="javascript-msaljs-1x"></a>[JavaScript (MSAL.js 1.x)](#tab/javascript1)

Les méthodes de redirection ne retournent pas de promesse en raison du déplacement en dehors de l’application principale. Pour traiter et accéder aux jetons retournés, vous devez enregistrer les rappels de réussite et d’erreur avant d’appeler les méthodes de redirection.

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

const myMsal = new UserAgentApplication(config);

function authCallback(error, response) {
    //handle redirect response
}

myMsal.handleRedirectCallback(authCallback);

myMsal.loginRedirect(loginRequest);
```

# <a name="angular"></a>[Angular](#tab/angular)

Ce code est identique à celui décrit plus haut, à la section traitant de la connexion au moyen d’une fenêtre contextuelle. La redirection correspond au flux par défaut.

---

## <a name="sign-out"></a>Se déconnecter

La bibliothèque MSAL fournit une méthode `logout` qui efface le cache dans le stockage de navigateur, et envoie une requête de déconnexion à Azure Active Directory (Azure AD). Après la déconnexion, la bibliothèque redirige vers la page de démarrage par défaut d’une application.

Vous pouvez configurer l’URI de redirection après la déconnexion en définissant `postLogoutRedirectUri`. Cet URI doit également être enregistré en tant qu’URI de déconnexion dans l’inscription de votre application.

# <a name="javascript-msaljs-2x"></a>[JavaScript (MSAL.js 2.x)](#tab/javascript2)

```javascript
const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const myMsal = new PublicClientApplication(config);

// you can select which account application should sign out
const logoutRequest = {
    account: myMsal.getAccountByUsername(username)
}

myMsal.logout(logoutRequest);
```

# <a name="javascript-msaljs-1x"></a>[JavaScript (MSAL.js 1.x)](#tab/javascript1)

```javascript
const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const myMsal = new UserAgentApplication(config);

myMsal.logout();
```

# <a name="angular"></a>[Angular](#tab/angular)

```javascript
//In app.module.ts
@NgModule({
    imports: [
        MsalModule.forRoot({
            auth: {
                clientId: 'your_app_id',
                postLogoutRedirectUri: "your_app_logout_redirect_uri"
            }
        })
    ]
})

// In app.component.ts
this.authService.logout();
```

---

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Acquérir un jeton pour l’application](scenario-spa-acquire-token.md).