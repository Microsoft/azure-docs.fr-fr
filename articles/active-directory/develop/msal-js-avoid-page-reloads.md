---
title: Éviter les rechargements de pages (MSAL.js) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment éviter les rechargements de page lors de l’acquisition et du renouvellement de jetons en mode silencieux à l’aide de Microsoft Authentication Library pour JavaScript (MSAL.js).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 05/29/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 5eb30f7dcf4b459b0af0bd8de965971fbbe44863
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85477649"
---
# <a name="avoid-page-reloads-when-acquiring-and-renewing-tokens-silently-using-msaljs"></a>Éviter les rechargements de page lors de l’acquisition et du renouvellement de jetons en mode silencieux à l'aide de MSAL.js
Microsoft Authentication Library pour JavaScript (MSAL.js) utilise des éléments `iframe` masqués pour acquérir et renouveler des jetons en mode silencieux en arrière-plan. Azure AD renvoie le jeton à l'URI redirect_uri spécifié dans la requête de jeton (par défaut, il s'agit de la page racine de l’application). La réponse étant une réponse 302, le HTML correspondant à `redirect_uri` est chargé dans `iframe`. En règle générale, l'URI `redirect_uri` de l'application correspond à la page racine, ce qui permet son rechargement.

Mais parfois, l'accès à la page racine de l’application requiert une authentification, ce qui peut donner lieu à des éléments `iframe` imbriqués ou à une erreur `X-Frame-Options: deny`.

MSAL.js n'étant pas en mesure de faire disparaître la réponse 302 émise par Azure AD et devant traiter le jeton renvoyé, il n'est pas en mesure d'empêcher le chargement de `redirect_uri` dans `iframe`.

Pour éviter le rechargement de l'application toute entière et des autres erreurs qui en résultent, suivez ces solutions de contournement.

## <a name="specify-different-html-for-the-iframe"></a>Spécifier un HTML différent pour l'iframe

Définissez la propriété `redirect_uri` sur une seule page, ce qui ne requiert pas d’authentification. Vous devez vous assurer qu’elle correspond à l'URI `redirect_uri` inscrit dans le portail Azure. Cela n'affecte pas l'expérience de connexion de l’utilisateur car MSAL enregistre la page de démarrage lorsque l’utilisateur entame le processus de connexion, et redirige vers l’emplacement exact une fois la connexion établie.

## <a name="initialization-in-your-main-app-file"></a>Initialisation dans votre fichier d’application principal

Si votre application est structurée de telle manière qu'un fichier Javascript central définit l’initialisation, le routage, etc. de l'application, vous pouvez charger de manière conditionnelle les modules de votre application selon que l'application se charge dans un `iframe` ou non. Par exemple :

Dans AngularJS : app.js

```javascript
// Check that the window is an iframe and not popup
if (window !== window.parent && !window.opener) {
angular.module('todoApp', ['ui.router', 'MsalAngular'])
    .config(['$httpProvider', 'msalAuthenticationServiceProvider','$locationProvider', function ($httpProvider, msalProvider,$locationProvider) {
        msalProvider.init(
            // msal configuration
        );

        $locationProvider.html5Mode(false).hashPrefix('');
    }]);
}
else {
    angular.module('todoApp', ['ui.router', 'MsalAngular'])
        .config(['$stateProvider', '$httpProvider', 'msalAuthenticationServiceProvider', '$locationProvider', function ($stateProvider, $httpProvider, msalProvider, $locationProvider) {
            $stateProvider.state("Home", {
                url: '/Home',
                controller: "homeCtrl",
                templateUrl: "/App/Views/Home.html",
            }).state("TodoList", {
                url: '/TodoList',
                controller: "todoListCtrl",
                templateUrl: "/App/Views/TodoList.html",
                requireLogin: true
            })

            $locationProvider.html5Mode(false).hashPrefix('');

            msalProvider.init(
                // msal configuration
            );
        }]);
}
```

Dans Angular : app.module.ts

```javascript
// Imports...
@NgModule({
  declarations: [
    AppComponent,
    MsalComponent,
    MainMenuComponent,
    AccountMenuComponent,
    OsNavComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    HttpClientModule,
    ServiceWorkerModule.register('ngsw-worker.js', { enabled: environment.production }),
    MsalModule.forRoot(environment.MsalConfig),
    SuiModule,
    PagesModule
  ],
  providers: [
    HttpServiceHelper,
    {provide: HTTP_INTERCEPTORS, useClass: MsalInterceptor, multi: true},
    AuthService
  ],
  entryComponents: [
    AppComponent,
    MsalComponent
  ]
})
export class AppModule {
  constructor() {
    console.log('APP Module Constructor!');
  }

  ngDoBootstrap(ref: ApplicationRef) {
    if (window !== window.parent && !window.opener)
    {
      console.log("Bootstrap: MSAL");
      ref.bootstrap(MsalComponent);
    }
    else
    {
    //this.router.resetConfig(RouterModule);
      console.log("Bootstrap: App");
      ref.bootstrap(AppComponent);
    }
  }
}
```

MsalComponent :

```javascript
import { Component} from '@angular/core';
import { MsalService } from '@azure/msal-angular';

// This component is used only to avoid Angular reload
// when doing acquireTokenSilent()

@Component({
  selector: 'app-root',
  template: '',
})
export class MsalComponent {
  constructor(private Msal: MsalService) {
  }
}
```

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur la [création d’une application monopage (SPA)](scenario-spa-overview.md) à l'aide de MSAL.js.