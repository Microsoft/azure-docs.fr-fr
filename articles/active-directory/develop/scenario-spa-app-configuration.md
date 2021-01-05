---
title: Configurer une application monopage - Plateforme d’identités Microsoft | Azure
description: Apprendre à créer une application monopage (configuration du code de l’application)
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
ms.openlocfilehash: 30a9b710ffbf98ebc523217a3b8a7fd9a2640c49
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94443039"
---
# <a name="single-page-application-code-configuration"></a>Application monopage : Configuration de code

En savoir plus sur la configuration du code de votre application monopage (SPA).

## <a name="msal-libraries-for-spas-and-supported-authentication-flows"></a>Bibliothèques MSAL pour les SPA et les flux d’authentification pris en charge

La plateforme d’identité Microsoft fournit la bibliothèque d’authentification Microsoft suivante pour JavaScript (MSAL.js) pour prendre en charge le flux implicite et le flux de code d’autorisation avec PKCE à l’aide des pratiques de sécurité recommandées par le secteur :

| Bibliothèque MSAL | Flux | Description |
|--------------|------|-------------|
| ![MSAL.js](media/sample-v2-code/logo_js.png) <br/> [MSAL.js (2.x)](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser) | Flux du code d’autorisation (PKCE) | Bibliothèque JavaScript brute à utiliser dans n’importe quelle application web côté client qui est créée par l’intermédiaire de frameworks JavaScript ou SPA, tels que Angular, Vue.js and React.js. |
| ![MSAL.js](media/sample-v2-code/logo_js.png) <br/> [MSAL.js (1.x)](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-core) | Flux implicite | Bibliothèque JavaScript brute à utiliser dans n’importe quelle application web côté client qui est créée par l’intermédiaire de frameworks JavaScript ou SPA, tels que Angular, Vue.js and React.js. |
| ![MSAL Angular](media/sample-v2-code/logo_angular.png) <br/> [MSAL Angular](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/README.md) | Flux implicite | Wrapper de la bibliothèque MSAL.js principale pour simplifier l’utilisation dans les applications monopages qui sont créées à l’aide du framework Angular. |

## <a name="application-code-configuration"></a>Configuration du code de l’application

Dans une bibliothèque MSAL, les informations d’inscription d’application sont transmises en tant que configuration pendant l’initialisation de la bibliothèque.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
// Configuration object constructed.
const config = {
    auth: {
        clientId: 'your_client_id'
    }
};

// create UserAgentApplication instance
const userAgentApplication = new UserAgentApplication(config);
```

Pour plus d’informations sur les options configurables, consultez [Initialisation d’application avec MSAL.js](msal-js-initializing-client-applications.md).

# <a name="angular"></a>[Angular](#tab/angular)

```javascript
// App.module.ts
import { MsalModule } from '@azure/msal-angular';

@NgModule({
    imports: [
        MsalModule.forRoot({
            auth: {
                clientId: 'your_app_id'
            }
        })
    ]
})

export class AppModule { }
```

---

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Connexion et déconnexion](scenario-spa-sign-in.md).
