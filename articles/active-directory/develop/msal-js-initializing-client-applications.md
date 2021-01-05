---
title: Initialiser des applications clientes MSAL.js | Azure
titleSuffix: Microsoft identity platform
description: En savoir plus sur l’initialisation d’applications clientes à l’aide de la bibliothèque d’authentification Microsoft pour JavaScript (MSAL.js).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/17/2020
ms.author: marsma
ms.reviewer: saeeda
ms.custom: aaddev, devx-track-js
ms.openlocfilehash: 6b5093c5a1a45aed3493fabd7a362b0579998171
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96343584"
---
# <a name="initialize-client-applications-using-msaljs"></a>Initialiser des applications clientes avec MSAL.js

Cet article décrit l’initialisation de la bibliothèque d’authentification Microsoft pour JavaScript (MSAL.js) avec une instance d’application agent utilisateur.

L’application agent utilisateur est une forme d'application cliente publique dans laquelle le code client est exécuté dans un agent utilisateur, par exemple un navigateur web. Ces clients ne stockent pas de secrets, car le contexte du navigateur est ouvertement accessible.

Pour plus d’informations sur les types d’application cliente et les options de configuration des applications, consultez [Applications clientes publiques et confidentielles dans MSAL](msal-client-applications.md).

## <a name="prerequisites"></a>Prérequis

Avant d’initialiser une application, vous devez d’abord l’[inscrire via le portail Azure](scenario-spa-app-registration.md), en établissant une relation d’approbation entre votre application et la plateforme d’identités Microsoft.

Une fois l’application inscrite, vous avez besoin d’une partie ou de toutes les valeurs suivantes dans le portail Azure.

| Valeur | Obligatoire | Description |
|:----- | :------: | :---------- |
| ID d’application (client) | Obligatoire | GUID qui identifie de manière unique votre application dans la plateforme d’identités Microsoft. |
| Authority | Facultatif | URL du fournisseur d’identité (*instance*) et *audience de connexion* de votre application. L’instance et l’audience de connexion, une fois concaténées, constituent l’*autorité*. |
| ID de l’annuaire (locataire) | Facultatif | Spécifiez cette valeur si vous générez une application métier seulement pour votre organisation, souvent appelée *application monolocataire*. |
| URI de redirection | Facultatif | Si vous générez une application web, `redirectUri` spécifie l’emplacement où le fournisseur d’identité (la plateforme d’identités Microsoft) doit retourner les jetons de sécurité émis. |

## <a name="initialize-msaljs-2x-apps"></a>Initialiser des applications MSAL.js 2.x

Initialisez le contexte d’authentification MSAL en instanciant un [PublicClientApplication][msal-js-publicclientapplication] avec un objet [Configuration][msal-js-configuration]. La propriété de configuration minimale nécessaire est le `clientID` de votre application. Elle est affichée sous la forme **ID d’application (client)** dans la page **Vue d’ensemble** de l’inscription de l’application au sein du portail Azure.

Voici un exemple d’objet de configuration et d’instanciation de `PublicClientApplication` :

```javascript
const msalConfig = {
    auth: {
        clientId: "11111111-1111-1111-111111111111",
        authority: "https://login.microsoftonline.com/common",
        knownAuthorities: [],
        redirectUri: "https://localhost:3001",
        postLogoutRedirectUri: "https://localhost:3001/logout",
        navigateToLoginRequestUrl: true
    },
    cache: {
        cacheLocation: "sessionStorage",
        storeAuthStateInCookie: false
    },
    system: {
        loggerOptions: {
            loggerCallback: (level: LogLevel, message: string, containsPii: boolean): void => {
                if (containsPii) {
                    return;
                }
                switch (level) {
                    case LogLevel.Error:
                        console.error(message);
                        return;
                    case LogLevel.Info:
                        console.info(message);
                        return;
                    case LogLevel.Verbose:
                        console.debug(message);
                        return;
                    case LogLevel.Warning:
                        console.warn(message);
                        return;
                }
            },
            piiLoggingEnabled: false
        },
        windowHashTimeout: 60000,
        iframeHashTimeout: 6000,
        loadFrameTimeout: 0
    }
};

// Create an instance of PublicClientApplication
const msalInstance = new PublicClientApplication(msalConfig);

// Handle the redirect flows
msalInstance.handleRedirectPromise().then((tokenResponse) => {
    // Handle redirect response
}).catch((error) => {
    // Handle redirect error
});
```

### `handleRedirectPromise`

Appelez [handleRedirectPromise][msal-js-handleredirectpromise] quand votre application utilise les flux de redirection. Durant l’utilisation des flux de redirection, `handleRedirectPromise` doit être exécuté à chaque chargement de page.

Il existe trois résultats possibles de la promesse :

- `.then` est appelé et `tokenResponse` est vrai : L’application est retournée à partir d’une opération de redirection réussie.
- `.then` est appelé et `tokenResponse` est faux (`null`) : L’application n’est pas retournée à partir d’une opération de redirection.
- `.catch` est appelé : L’application est retournée à partir d’une opération de redirection et une erreur s’est produite.

## <a name="initialize-msaljs-1x-apps"></a>Initialiser des applications MSAL.js 1.x

Initialisez le contexte d’authentification de MSAL 1.x en instanciant [UserAgentApplication][msal-js-useragentapplication] avec un objet de configuration. La propriété de configuration minimale nécessaire est le `clientID` de votre application. Elle est affichée sous la forme **ID d’application (client)** dans la page **Vue d’ensemble** de l’inscription de l’application au sein du portail Azure.

Pour les méthodes d’authentification ayant les flux de redirection ([loginRedirect][msal-js-loginredirect] et [acquireTokenRedirect][msal-js-acquiretokenredirect]) dans MSAL.js 1.2.x ou une version antérieure, vous devez explicitement inscrire un rappel de réussite ou d’erreur via la méthode `handleRedirectCallback()`. L’inscription explicite du rappel est nécessaire dans MSAL.js 1.2.x et les versions antérieures, car les flux de redirection ne retournent pas de promesses comme le font les méthodes ayant une expérience utilisateur de fenêtre contextuelle. L’inscription du rappel est *facultative* dans MSAL.js 1.3.x et les versions ultérieures.

```javascript
// Configuration object constructed
const msalConfig = {
    auth: {
        clientId: "11111111-1111-1111-111111111111"
    }
}

// Create UserAgentApplication instance
const msalInstance = new UserAgentApplication(msalConfig);

function authCallback(error, response) {
    // Handle redirect response
}

// Register a redirect callback for Success or Error (when using redirect methods)
// **REQUIRED** in MSAL.js 1.2.x and earlier
// **OPTIONAL** in MSAL.js 1.3.x and later
msalInstance.handleRedirectCallback(authCallback);
```

## <a name="single-instance-and-configuration"></a>Instance et configuration uniques

MSAL.js 1.x et 2.x sont tous deux conçus pour avoir une seule instance et une seule configuration de `UserAgentApplication` ou `PublicClientApplication`, respectivement, afin de représenter un seul contexte d’authentification.

Il n’est pas recommandé d’utiliser plusieurs instances de `UserAgentApplication` ou `PublicClientApplication`, car elles entraînent des conflits au niveau des entrées du cache et du comportement dans le navigateur.

## <a name="next-steps"></a>Étapes suivantes

Cet exemple de code MSAL.js 2.x sur GitHub illustre l’instanciation de [PublicClientApplication][msal-js-publicclientapplication] avec un objet [Configuration][msal-js-configuration] :

[Azure-Samples/ms-identity-javascript-v2](https://github.com/Azure-Samples/ms-identity-javascript-v2)

<!-- LINKS - External -->
[msal-browser]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-browser/
[msal-core]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-core/
[msal-js-acquiretokenredirect]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-core/classes/_useragentapplication_.useragentapplication.html#acquiretokenredirect
[msal-js-configuration]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-core/modules/_configuration_.html
[msal-js-handleredirectpromise]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-browser/classes/_src_app_publicclientapplication_.publicclientapplication.html#handleredirectpromise
[msal-js-loginredirect]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-core/classes/_useragentapplication_.useragentapplication.html#loginredirect
[msal-js-publicclientapplication]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-browser/classes/_src_app_publicclientapplication_.publicclientapplication.html
[msal-js-useragentapplication]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-core/modules/_useragentapplication_.html
