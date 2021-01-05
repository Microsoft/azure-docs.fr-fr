---
title: Acquérir un jeton pour appeler une API web (applications à page unique) - Plateforme d’identités Microsoft | Azure
description: Découvrir comment créer une application monopage (acquérir un jeton pour appeler une API)
services: active-directory
author: negoe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/20/2019
ms.author: negoe
ms.custom: aaddev
ms.openlocfilehash: 3d91e69d7d9eac9f8cae1a1a122a13afd19ef631
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94443872"
---
# <a name="single-page-application-acquire-a-token-to-call-an-api"></a>Application monopage : Acquérir un jeton pour appeler une API

Le modèle d’acquisition des jetons d’API avec MSAL.js consiste à tenter d’abord une demande de jeton en mode silencieux à l’aide de la méthode `acquireTokenSilent`. Lorsque cette méthode est appelée, la bibliothèque vérifie d’abord le cache dans le stockage du navigateur pour voir s’il existe un jeton valide, et le retourne. Lorsqu’il n’y a aucun jeton valide dans le cache, elle envoie une demande de jeton en mode silencieux à Azure Active Directory (Azure AD) à partir d’un iframe masqué. Cette méthode permet également à la bibliothèque de renouveler les jetons. Pour plus d’informations sur la session d’authentification unique et les valeurs de durée de vie des jetons dans Azure AD, consultez [Durée de vie des jetons](active-directory-configurable-token-lifetimes.md).

Les demandes de jeton en mode silencieux à Azure AD peuvent échouer pour certaines raisons, par exemple une session Active Directory Azure expirée ou un changement de mot de passe. Dans ce cas, vous pouvez appeler une des méthodes interactives (qui invitent l’utilisateur) pour acquérir des jetons :

* [Fenêtre contextuelle](#acquire-a-token-with-a-pop-up-window), au moyen de `acquireTokenPopup`
* [Redirection](#acquire-a-token-with-a-redirect), au moyen de `acquireTokenRedirect`

## <a name="choose-between-a-pop-up-or-redirect-experience"></a>Choisir entre une expérience avec fenêtre contextuelle ou redirection

 Vous ne pouvez pas utiliser conjointement ces deux méthodes de fenêtre contextuelle et de redirection dans votre application. Le choix entre la fenêtre contextuelle et la redirection dépend du flux de votre application :

* Si vous ne souhaitez pas que les utilisateurs quittent la page principale de votre application lors de l’authentification, nous vous recommandons d’utiliser la méthode avec fenêtre contextuelle. La redirection de l’authentification se produisant dans une fenêtre contextuelle, l’état de l’application principale est conservé.

* Si les utilisateurs sont soumis à des contraintes imposées par le navigateur ou par des stratégies qui bloquent les fenêtres contextuelles, vous pouvez utiliser la méthode avec redirection. Utilisez la méthode avec redirection dans le navigateur Internet Explorer, car il existe des [problèmes connus avec l’utilisation de fenêtres contextuelles dans Internet Explorer](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser).

Vous pouvez définir les étendues d’API qui doivent être incluses dans le jeton d’accès lors de la création de la requête de jeton d’accès. Notez que les étendues demandées sont susceptibles de ne pas être toutes accordées dans le jeton d’accès. Cela dépend du consentement de l’utilisateur.

## <a name="acquire-a-token-with-a-pop-up-window"></a>Acquérir un jeton avec une fenêtre contextuelle

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Le code suivant combine le modèle décrit précédemment aux méthodes offrant une expérience de fenêtre contextuelle :

```javascript
const accessTokenRequest = {
    scopes: ["user.read"]
}

userAgentApplication.acquireTokenSilent(accessTokenRequest).then(function(accessTokenResponse) {
    // Acquire token silent success
    // Call API with token
    let accessToken = accessTokenResponse.accessToken;
}).catch(function (error) {
    //Acquire token silent failure, and send an interactive request
    if (error.errorMessage.indexOf("interaction_required") !== -1) {
        userAgentApplication.acquireTokenPopup(accessTokenRequest).then(function(accessTokenResponse) {
            // Acquire token interactive success
        }).catch(function(error) {
            // Acquire token interactive failure
            console.log(error);
        });
    }
    console.log(error);
});
```

# <a name="angular"></a>[Angular](#tab/angular)

Le wrapper MSAL Angular fournit l’intercepteur HTTP, qui acquiert automatiquement des jetons d’accès en mode silencieux et les joint aux requêtes HTTP destinées aux API.

Vous pouvez spécifier les étendues des API dans l’option de configuration `protectedResourceMap`. `MsalInterceptor` demande ces étendues lors de l’acquisition automatique de jetons.

```javascript
// app.module.ts
@NgModule({
  declarations: [
    // ...
  ],
  imports: [
    // ...
    MsalModule.forRoot({
      auth: {
        clientId: 'Enter_the_Application_Id_Here',
      }
    },
    {
      popUp: !isIE,
      consentScopes: [
        'user.read',
        'openid',
        'profile',
      ],
      protectedResourceMap: [
        ['https://graph.microsoft.com/v1.0/me', ['user.read']]
      ]
    })
  ],
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: MsalInterceptor,
      multi: true
    }
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Pour la réussite et l’échec de l’acquisition de jetons en mode silencieux, MSAL Angular fournit des rappels auxquels vous pouvez vous abonner. Il est également important de se souvenir d’annuler l’abonnement.

```javascript
// In app.component.ts
 ngOnInit() {
    this.subscription=  this.broadcastService.subscribe("msal:acquireTokenFailure", (payload) => {
    });
}

ngOnDestroy() {
   this.broadcastService.getMSALSubject().next(1);
   if (this.subscription) {
     this.subscription.unsubscribe();
   }
 }
```

Sinon, vous pouvez explicitement acquérir des jetons à l’aide des méthodes d’acquisition de jeton, comme décrit dans la bibliothèque MSAL.js principale.

---

## <a name="acquire-a-token-with-a-redirect"></a>Acquérir un jeton avec une redirection

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Le modèle suivant est tel qu’il a été décrit précédemment, mais il est présenté avec une méthode de redirection qui permet d’acquérir les jetons de manière interactive. Vous devez inscrire le rappel de redirection, comme indiqué plus haut.

```javascript
function authCallback(error, response) {
    // Handle redirect response
}

userAgentApplication.handleRedirectCallback(authCallback);

const accessTokenRequest: AuthenticationParameters = {
    scopes: ["user.read"]
}

userAgentApplication.acquireTokenSilent(accessTokenRequest).then(function(accessTokenResponse) {
    // Acquire token silent success
    // Call API with token
    let accessToken = accessTokenResponse.accessToken;
}).catch(function (error) {
    //Acquire token silent failure, and send an interactive request
    console.log(error);
    if (error.errorMessage.indexOf("interaction_required") !== -1) {
        userAgentApplication.acquireTokenRedirect(accessTokenRequest);
    }
});
```

## <a name="request-optional-claims"></a>Demande de revendications facultatives

Vous pouvez utiliser des revendications facultatives aux fins suivantes :

- Inclure des revendications supplémentaires dans les jetons pour votre application.
- Modifier le comportement de certaines revendications retournées par Azure AD dans les jetons.
- Ajouter et accéder à des revendications personnalisées pour votre application.

Pour demander des revendications facultatives dans `IdToken`, vous pouvez envoyer un objet revendications stringifié au champ `claimsRequest` de la classe `AuthenticationParameters.ts`.

```javascript
"optionalClaims":
   {
      "idToken": [
            {
                  "name": "auth_time",
                  "essential": true
             }
      ],

var request = {
    scopes: ["user.read"],
    claimsRequest: JSON.stringify(claims)
};

myMSALObj.acquireTokenPopup(request);
```

Pour en savoir plus, consultez [Revendications facultatives](active-directory-optional-claims.md).

# <a name="angular"></a>[Angular](#tab/angular)

Ce code est identique à celui qui est décrit plus haut.

---

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Appeler une API web](scenario-spa-call-api.md).