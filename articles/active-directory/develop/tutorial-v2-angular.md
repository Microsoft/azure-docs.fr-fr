---
title: 'Tutoriel : Créer une application Angular qui utilise la plateforme d’identités Microsoft pour l’authentification | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce tutoriel, vous allez créer une application monopage Angular qui utilise la plateforme d’identités Microsoft pour connecter les utilisateurs et obtenir un jeton d’accès pour appeler l’API Microsoft Graph en leur nom.
services: active-directory
author: hamiltonha
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 03/05/2020
ms.author: hahamil
ms.custom: aaddev, identityplatformtop40, devx-track-js
ms.openlocfilehash: c4c7d021c7c3a5a32d537a50fa45449fdee7e817
ms.sourcegitcommit: f311f112c9ca711d88a096bed43040fcdad24433
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94979927"
---
# <a name="tutorial-sign-in-users-and-call-the-microsoft-graph-api-from-an-angular-single-page-application"></a>Tutoriel : Connecter des utilisateurs et appeler l’API Microsoft Graph à partir d’une application monopage Angular

Dans ce tutoriel, vous créez une application monopage (SPA) Angular qui connecte les utilisateurs et appelle l’API Microsoft Graph.

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Créer un projet angulaire avec `npm`.
> * Inscrire l’application dans le Portail Azure
> * Ajouter du code pour prendre en charge la connexion et la déconnexion des utilisateurs
> * Ajouter du code pour appeler l’API Microsoft Graph
> * Test de l'application

## <a name="prerequisites"></a>Prérequis

* [Node.js](https://nodejs.org/en/download/) pour l’exécution d’un serveur web local
* [Visual Studio Code](https://code.visualstudio.com/download) ou un autre éditeur pour la modification des fichiers projet

## <a name="how-the-sample-app-works"></a>Fonctionnement de l’exemple d’application

![Schéma illustrant le fonctionnement de l’exemple d’application généré dans ce tutoriel](./media/tutorial-v2-angular/diagram-auth-flow-spa-angular.svg)

L’exemple d’application créé dans ce tutoriel permet à une application monopage Angular d’interroger l’API Microsoft Graph ou une API web qui accepte des jetons émis par la plateforme d’identités Microsoft. Elle utilise la bibliothèque d’authentification Microsoft (MSAL) pour Angular, wrapper de la bibliothèque MSAL.js principale. MSAL Angular permet aux applications Angular 6+ d’authentifier les utilisateurs en entreprise avec Azure Active Directory (Azure AD) ainsi que les utilisateurs qui ont des comptes d’utilisateur Microsoft et des identités de réseaux sociaux tels que Facebook, Google et LinkedIn. La bibliothèque permet aussi aux applications d’accéder aux services cloud Microsoft et à Microsoft Graph.

Dans ce scénario, une fois qu’un utilisateur s’est connecté, un jeton d’accès est demandé et ajouté aux requêtes HTTP par le biais de l’en-tête d’autorisation. Les opérations d’acquisition et de renouvellement de jetons sont gérées par MSAL.

### <a name="libraries"></a>Bibliothèques

Ce tutoriel utilise la bibliothèque suivante :

|Bibliothèque|Description|
|---|---|
|[msal.js](https://github.com/AzureAD/microsoft-authentication-library-for-js)|Wrapper de la bibliothèque d’authentification Microsoft pour JavaScript Angular|

Vous trouverez le code source de la bibliothèque MSAL.js dans le dépôt [AzureAD/microsoft-authentication-library-for-js](https://github.com/AzureAD/microsoft-authentication-library-for-js) sur GitHub.

## <a name="create-your-project"></a>Créer votre projet

Générez une nouvelle application Angular à l’aide des commandes npm suivantes :

```bash
npm install -g @angular/cli@8                    # Install the Angular CLI
ng new my-application --routing=true --style=css # Generate a new Angular app
cd my-application                                # Change to the app directory
npm install @angular/material@8 @angular/cdk@8   # Install the Angular Material component library (optional, for UI)
npm install msal @azure/msal-angular             # Install MSAL and MSAL Angular in your application
ng generate component page-name                  # To add a new page (such as a home or profile page)
```

## <a name="register-your-application"></a>Inscrivez votre application

Suivez les [instructions pour inscrire une application monopage](./scenario-spa-app-registration.md) sur le portail Azure.

Dans la page **Vue d’ensemble** de votre inscription, notez la valeur de **ID d’application (client)** pour une utilisation ultérieure.

Inscrivez la valeur de votre **URI de redirection** sous la forme **http://localhost:4200/** et activez les paramètres d’octroi implicite.

## <a name="configure-the-application"></a>Configuration de l'application

1. Dans le dossier *src/app*, modifiez *app.module.ts* et ajoutez `MSALModule` à `imports` ainsi que la constante `isIE` :

    ```javascript
    const isIE = window.navigator.userAgent.indexOf('MSIE ') > -1 || window.navigator.userAgent.indexOf('Trident/') > -1;
    @NgModule({
      declarations: [
        AppComponent
      ],
      imports: [
        BrowserModule,
        AppRoutingModule,
        MsalModule.forRoot({
          auth: {
            clientId: 'Enter_the_Application_Id_here', // This is your client ID
            authority: 'Enter_the_Cloud_Instance_Id_Here'/'Enter_the_Tenant_Info_Here', // This is your tenant ID
            redirectUri: 'Enter_the_Redirect_Uri_Here'// This is your redirect URI
          },
          cache: {
            cacheLocation: 'localStorage',
            storeAuthStateInCookie: isIE, // Set to true for Internet Explorer 11
          },
        }, {
          popUp: !isIE,
          consentScopes: [
            'user.read',
            'openid',
            'profile',
          ],
          unprotectedResources: [],
          protectedResourceMap: [
            ['https://graph.microsoft.com/v1.0/me', ['user.read']]
          ],
          extraQueryParameters: {}
        })
      ],
      providers: [],
      bootstrap: [AppComponent]
    })
    ```

    Remplacez les valeurs suivantes :

    |Nom de la valeur|À propos|
    |---------|---------|
    |Enter_the_Application_Id_Here|Dans la page **Vue d’ensemble** de l’inscription de votre application, il s’agit de la valeur de votre **ID d’application (client)** . |
    |Enter_the_Cloud_Instance_Id_Here|Il s’agit de l’instance du cloud Azure. Pour le cloud Azure principal ou mondial, entrez **https://login.microsoftonline.com** . Pour les clouds nationaux (par exemple Chine), consultez [Clouds nationaux](./authentication-national-cloud.md).|
    |Enter_the_Tenant_Info_Here| Définissez cette valeur sur une des options suivantes : Si votre application prend en charge les *comptes dans cet annuaire organisationnel*, remplacez cette valeur par l’ID de l’annuaire (locataire) ou le nom du locataire (par exemple, **contoso.microsoft.com**). Si votre application prend en charge les *Comptes dans un annuaire organisationnel*, remplacez cette valeur par **organizations**. Si votre application prend en charge les *Comptes dans un annuaire organisationnel et comptes personnels Microsoft*, remplacez cette valeur par **common**. Pour limiter la prise en charge aux *Comptes Microsoft personnels uniquement*, remplacez cette valeur par **consumers**. |
    |Enter_the_Redirect_Uri_Here|À remplacer par **http://localhost:4200** .|

    Pour plus d’informations sur les options configurables disponibles, consultez [Initialiser les applications clientes](msal-js-initializing-client-applications.md).

2. En haut du même fichier, ajoutez l’instruction import suivante :

    ```javascript
    import { MsalModule, MsalInterceptor } from '@azure/msal-angular';
    ```

3. Ajoutez les instructions import suivantes en haut du fichier `src/app/app.component.ts` :

    ```javascript
    import { MsalService, BroadcastService } from '@azure/msal-angular';
    import { Component, OnInit } from '@angular/core';
    ```
## <a name="sign-in-a-user"></a>Connecter un utilisateur

Ajoutez le code suivant à `AppComponent` pour connecter un utilisateur :

```javascript
export class AppComponent implements OnInit {
    constructor(private broadcastService: BroadcastService, private authService: MsalService) { }

    ngOnInit() { }

    login() {
        const isIE = window.navigator.userAgent.indexOf('MSIE ') > -1 || window.navigator.userAgent.indexOf('Trident/') > -1;

        if (isIE) {
          this.authService.loginRedirect({
            extraScopesToConsent: ["user.read", "openid", "profile"]
          });
        } else {
          this.authService.loginPopup({
            extraScopesToConsent: ["user.read", "openid", "profile"]
          });
        }
    }
}
```

> [!TIP]
> Nous vous recommandons d’utiliser `loginRedirect` pour les utilisateurs d’Internet Explorer.

## <a name="acquire-a-token"></a>Acquérir un jeton

### <a name="angular-interceptor"></a>Classe Interceptor d’Angular

MSAL Angular fournit une classe `Interceptor` qui acquiert automatiquement des jetons pour les demandes sortantes qui utilisent le client `http` Angular pour les ressources connues protégées.

Tout d’abord, incluez la classe `Interceptor` en tant que fournisseur dans votre application :

```javascript
import { MsalInterceptor, MsalModule } from "@azure/msal-angular";
import { HTTP_INTERCEPTORS, HttpClientModule } from "@angular/common/http";

@NgModule({
    // ...
    providers: [
        {
            provide: HTTP_INTERCEPTORS,
            useClass: MsalInterceptor,
            multi: true
        }
    ]
}
```

Ensuite, fournissez une carte des ressources protégées à `MsalModule.forRoot()` en tant que `protectedResourceMap` et incluez ces étendues dans `consentScopes`. Les URL que vous fournissez dans la collection `protectedResourceMap` sont sensibles à la casse.

```javascript
@NgModule({
  // ...
  imports: [
    // ...
    MsalModule.forRoot({
      auth: {
        clientId: 'Enter_the_Application_Id_here', // This is your client ID
        authority: 'https://login.microsoftonline.com/Enter_the_Tenant_Info_Here', // This is your tenant info
        redirectUri: 'Enter_the_Redirect_Uri_Here' // This is your redirect URI
      },
      cache: {
        cacheLocation: 'localStorage',
        storeAuthStateInCookie: isIE, // Set to true for Internet Explorer 11
      },
    },
    {
      popUp: !isIE,
      consentScopes: [
        'user.read',
        'openid',
        'profile',
      ],
      unprotectedResources: [],
      protectedResourceMap: [
        ['https://graph.microsoft.com/v1.0/me', ['user.read']]
      ],
      extraQueryParameters: {}
    })
  ],
});
```

Enfin, récupérez le profil d’un utilisateur avec une requête HTTP :

```JavaScript
const graphMeEndpoint = "https://graph.microsoft.com/v1.0/me";

getProfile() {
  this.http.get(graphMeEndpoint).toPromise()
    .then(profile => {
      this.profile = profile;
    });
}
```

### <a name="acquiretokensilent-acquiretokenpopup-acquiretokenredirect"></a>acquireTokenSilent, acquireTokenPopup, acquireTokenRedirect
MSAL utilise trois méthodes pour acquérir des jetons : `acquireTokenRedirect`, `acquireTokenPopup` et `acquireTokenSilent`. Nous vous recommandons cependant d’utiliser plutôt la classe `MsalInterceptor` pour les applications Angular, comme indiqué dans la section précédente.

#### <a name="get-a-user-token-silently"></a>Obtenir un jeton d’utilisateur en mode silencieux

La méthode `acquireTokenSilent` gère les acquisitions et le renouvellement de jetons sans interaction de l’utilisateur. Après la première exécution de la méthode `loginRedirect` ou `loginPopup`, c’est en général la méthode `acquireTokenSilent` qui est utilisée pour obtenir les jetons permettant d’accéder aux ressources protégées dans les appels ultérieurs. Les appels pour les demandes ou les renouvellements de jetons sont effectués en mode silencieux.

```javascript
const requestObj = {
    scopes: ["user.read"]
};

this.authService.acquireTokenSilent(requestObj).then(function (tokenResponse) {
    // Callback code here
    console.log(tokenResponse.accessToken);
}).catch(function (error) {
    console.log(error);
});
```

Dans ce code, `scopes` contient les étendues qui doivent être retournées dans le jeton d’accès pour l’API.

Par exemple :

* `["user.read"]` pour Microsoft Graph
* `["<Application ID URL>/scope"]` pour les API web personnalisées (c’est-à-dire `api://<Application ID>/access_as_user`)

#### <a name="get-a-user-token-interactively"></a>Obtenir un jeton d’utilisateur de manière interactive

Parfois, vous avez besoin que l’utilisateur interagisse avec le point de terminaison de la plateforme d’identité Microsoft. Par exemple :

* Les utilisateurs doivent réentrer leurs informations d’identification, car leur mot de passe a expiré.
* Votre application demande l’accès à des étendues de ressources supplémentaires pour laquelle l’utilisateur doit donner son consentement.
* Une authentification à 2 facteurs est demandée.

Le modèle recommandé pour la plupart des applications est de d’abord appeler `acquireTokenSilent`, puis d’intercepter l’exception, puis d’appeler `acquireTokenPopup` (ou `acquireTokenRedirect`) pour démarrer une demande interactive.

L’appel de `acquireTokenPopup` fait apparaître une fenêtre de connexion contextuelle. Une alternative est que `acquireTokenRedirect` redirige les utilisateurs vers le point de terminaison de la plateforme d’identités Microsoft. Dans cette fenêtre, les utilisateurs doivent confirmer leurs informations d’identification, donner leur consentement pour la ressource demandée ou effectuer une authentification à deux facteurs.

```javascript
  const requestObj = {
      scopes: ["user.read"]
  };

  this.authService.acquireTokenPopup(requestObj).then(function (tokenResponse) {
      // Callback code here
      console.log(tokenResponse.accessToken);
  }).catch(function (error) {
      console.log(error);
  });
```

> [!NOTE]
> Ce démarrage rapide utilise les méthodes `loginRedirect` et `acquireTokenRedirect` avec Microsoft Internet Explorer en raison d’un [problème connu](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues) lié à la gestion des fenêtres contextuelles par Internet Explorer.

## <a name="log-out"></a>Se déconnecter

Ajoutez le code suivant pour déconnecter un utilisateur :

```javascript
logout() {
  this.authService.logout();
}
```

## <a name="add-ui"></a>Ajouter une interface utilisateur
Pour obtenir un exemple de la façon dont une interface utilisateur est ajoutée en utilisant la bibliothèque de composants Angular Material, consultez l’[exemple d’application](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-angular).

## <a name="test-your-code"></a>Test de votre code

1.  Démarrez le serveur web pour écouter sur le port en exécutant les commandes suivantes sur une invite de ligne de commande à partir du dossier de l’application :

    ```bash
    npm install
    npm start
    ```
1. Dans votre navigateur, entrez **http://localhost:4200** ou **http://localhost:{port}** , où *port* est le port sur lequel votre serveur web est à l’écoute.


### <a name="provide-consent-for-application-access"></a>Accorder les droits d’accès à l’application

Quand vous vous connectez pour la première fois à votre application, vous êtes invité à lui accorder l’accès à votre profil et à l’autoriser à vous connecter :

![La fenêtre « Autorisations nécessaires »](media/active-directory-develop-guidedsetup-javascriptspa-test/javascriptspaconsent.png)

## <a name="add-scopes-and-delegated-permissions"></a>Ajouter des étendues et des autorisations déléguées

L’API Microsoft Graph nécessite l’étendue *user.read* pour lire le profil d’un utilisateur. Par défaut, cette étendue est automatiquement ajoutée à toutes les applications inscrites dans le portail d’inscription. D’autres API pour Microsoft Graph ainsi que des API personnalisées pour votre serveur principal peuvent nécessiter des étendues supplémentaires. Par exemple, l’API Microsoft Graph nécessite l’étendue *Calendars.Read* pour pouvoir lister les calendriers de l’utilisateur.

Pour accéder aux calendriers de l’utilisateur dans le contexte d’une application, ajoutez l’autorisation déléguée *Calendars.Read* aux informations d’inscription de l’application. Ajoutez ensuite l’étendue *Calendars.Read* à l’appel `acquireTokenSilent`.

>[!NOTE]
>L’utilisateur peut être invité à donner des consentements supplémentaires à mesure que vous augmentez le nombre d’étendues.

Si une API back-end ne nécessite pas d’étendue (non recommandé), vous pouvez utiliser *clientId* comme étendue dans les appels pour acquérir des jetons.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Approfondissez le développement d’applications monopages sur la plateforme d’identités Microsoft grâce à notre série d’articles.

> [!div class="nextstepaction"]
> [Scénario : Application monopage](scenario-spa-overview.md)
