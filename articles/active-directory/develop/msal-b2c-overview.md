---
title: Utiliser MSAL.js avec Azure AD B2C
titleSuffix: Microsoft identity platform
description: La bibliothèque d'authentification Microsoft pour JavaScript (MSAL.js) permet aux applications de travailler avec Azure AD B2C et d'acquérir des jetons pour appeler des API web sécurisées. Ces API web peuvent être Microsoft Graph, d’autres API Microsoft, des API web de tiers ou vos propres API web.
services: active-directory
author: negoe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 06/05/2020
ms.author: negoe
ms.reviewer: nacanuma
ms.custom: aaddev devx-track-js
ms.openlocfilehash: ef1c0003978251dd2637915e56dc396e85f4438f
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97107870"
---
# <a name="use-microsoft-authentication-library-for-javascript-to-work-with-azure-ad-b2c"></a>Utiliser la bibliothèque d’authentification Microsoft pour JavaScript pour travailler avec Azure AD B2C

La [bibliothèque d’authentification Microsoft pour JavaScript (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js) permet aux développeurs JavaScript d’authentifier les utilisateurs avec des identités sociales et locales à l’aide d’[Azure Active Directory B2C](../../active-directory-b2c/overview.md) (Azure AD B2C).

En utilisant Azure AD B2C en tant que service de gestion des identités, vous pouvez personnaliser et contrôler la façon dont vos clients s’inscrivent, se connectent et gèrent leurs profils lorsqu’ils utilisent vos applications. Azure AD B2C vous permet également de personnaliser l’interface utilisateur affichée par votre application lors du processus d’authentification.

Les sections ci-dessous illustrent comment :

- Protéger une API web Node.js
- Prendre en charge la connexion dans une application monopage (SPA) et appeler *cette* API web protégée
- Activer la prise en charge de la réinitialisation du mot de passe

## <a name="prerequisites"></a>Prérequis

Si vous ne l'avez pas déjà fait, créez un [locataire Azure AD B2C](../../active-directory-b2c/tutorial-create-tenant.md).

## <a name="nodejs-web-api"></a>API web Node.js

Les étapes suivantes montrent comment une **API web** peut utiliser Azure AD B2C pour se protéger et exposer des étendues sélectionnées à une application cliente.

MSAL.js pour Node est actuellement en cours de développement. Pour plus d'informations, consultez la [feuille de route](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki#roadmap) sur GitHub. En attendant, nous vous suggérons d’utiliser [passport-azure-ad](https://github.com/AzureAD/passport-azure-ad), une bibliothèque d’authentification pour Node.js développée et prise en charge par Microsoft.

### <a name="step-1-register-your-application"></a>Étape 1 : Inscrivez votre application

Pour protéger votre API web avec Azure AD B2C, vous devez commencer par l’inscrire. Pour des instructions détaillées, voir [Inscrire votre application](../../active-directory-b2c/add-web-api-application.md).

### <a name="step-2-download-the-sample-application"></a>Étape 2 : Téléchargement de l'exemple d'application

Téléchargez l’exemple en tant que fichier zip ou clonez-le à partir de GitHub :

```console
git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi.git
```

### <a name="step-3-configure-authentication"></a>Étape 3 : configurer l’authentification ;

1. Ouvrez le fichier *config.json* de l’exemple.

2. Configurez l’exemple avec les informations d’identification de l’application que vous avez obtenues précédemment lors de l’inscription de votre application. Modifiez les lignes de code suivantes en remplaçant les valeurs par le nom de votre locataire, l’ID client et le nom de la stratégie.

    ```json
         "credentials": {
             "tenantName": "<your-tenant-name>",
             "clientID": "<your-webapi-application-ID>"
         },
         "policies": {
             "policyName": "B2C_1_signupsignin1"
         },
         "resource": {
             "scope": ["demo.read"] 
         },
    ```

Pour plus d’informations, consultez cet [exemple d’API web Node.js B2C](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi).

## <a name="javascript-spa"></a>JavaScript SPA

Les étapes suivantes montrent comment une **application monopage** peut utiliser Azure AD B2C pour s’inscrire, se connecter et appeler une API web protégée.

### <a name="step-1-register-your-application"></a>Étape 1 : Inscrivez votre application

Pour implémenter l’authentification, vous devez d’abord inscrire votre application. Pour des instructions détaillées, voir [Inscrire votre application](../../active-directory-b2c/tutorial-register-applications.md).

### <a name="step-2-download-the-sample-application"></a>Étape 2 : Téléchargement de l'exemple d'application

Téléchargez l’[archive .ZIP](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp/archive/master.zip) de l’exemple de code ou clonez le référentiel GitHub :

```console
git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp.git
```

### <a name="step-3-configure-authentication"></a>Étape 3 : configurer l’authentification ;

La configuration de votre application présente deux points intéressants :

- Configurer le point de terminaison d’API et les étendues exposées
- Configurer les paramètres d’authentification et les étendues de jetons

1. Ouvrez le fichier *apiConfig.js* de l’exemple.

2. Configurez l’exemple avec les paramètres que vous avez obtenus précédemment lors de l’inscription de votre API web. Modifiez les lignes de code suivantes en remplaçant les valeurs par l’adresse de votre API web et les étendues exposées.

   ```javascript
    // The current application coordinates were pre-registered in a B2C tenant.
    const apiConfig = {
        b2cScopes: ["https://fabrikamb2c.onmicrosoft.com/helloapi/demo.read"], //API scopes you exposed during api registration
        webApi: "https://fabrikamb2chello.azurewebsites.net/hello"
    };
   ```

1. Ouvrez le fichier *authConfig.js* de l’exemple.

1. Configurez l’exemple avec les paramètres que vous avez obtenus précédemment lors de l’inscription de votre application monopage. Modifiez les lignes de code suivantes en remplaçant les valeurs par votre ID client, vos métadonnées d’autorité et vos étendues de demande de jeton.

   ```javascript
    // Config object to be passed to Msal on creation.
    const msalConfig = {
        auth: {
            clientId: "e760cab2-b9a1-4c0d-86fb-ff7084abd902",
            authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/B2C_1_signupsignin1",
            validateAuthority: false
        },
        cache: {
            cacheLocation: "localStorage", // This configures where your cache will be stored
            storeAuthStateInCookie: false // Set this to "true" to save cache in cookies
        }
    };

    // Add here scopes for id token to be used at the MS Identity Platform endpoint
    const loginRequest = {
        scopes: ["openid", "profile"],
    };
   ```

Pour plus d’informations, consultez cet [exemple d’application monopage JavaScript B2C](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp).

## <a name="support-password-reset"></a>Prendre en charge la réinitialisation du mot de passe

Dans cette section, vous étendez votre application monopage pour utiliser le flux d’utilisateur de réinitialisation du mot de passe Azure AD B2C. Bien que MSAL.js ne prenne actuellement pas en charge plusieurs flux d’utilisateurs ou stratégies personnalisées en mode natif, vous pouvez utiliser la bibliothèque pour gérer les cas d’usage courants tels que la réinitialisation du mot de passe.

Les étapes suivantes supposent que vous avez déjà suivi les étapes de la section précédente [JavaScript SPA](#javascript-spa) .

### <a name="step-1-define-the-authority-string-for-password-reset-user-flow"></a>Étape 1 : Définir la chaîne d’autorité pour le flux d’utilisateur de réinitialisation du mot de passe

1. Tout d’abord, créez un objet dans lequel vous stockez vos URI d’autorité :

    ```javascript
        const b2cPolicies = {
            names: {
                signUpSignIn: "b2c_1_susi",
                forgotPassword: "b2c_1_reset"
            },
            authorities: {
                signUpSignIn: {
                    authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_susi",
                },
                forgotPassword: {
                    authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_reset",
                },
            },
        }
    ```

1. Ensuite, initialisez votre objet MSAL avec la stratégie `signInSignUp` par défaut (consultez l’extrait de code précédent). Lorsqu’un utilisateur tente de se connecter, l’écran suivant s’affiche :

    :::image type="content" source="media/msal-b2c-overview/user-journey-01-signin.png" alt-text="Écran de connexion affiché par Azure AD B2C":::

### <a name="step-2-catch-and-handle-authentication-errors-in-your-login-method"></a>Étape 2 : Intercepter et gérer les erreurs d’authentification dans votre méthode de connexion

Lorsqu’un utilisateur sélectionne **Mot de passe oublié**, votre application génère une erreur que vous devez intercepter dans votre code, puis gérer en présentant le flux d’utilisateur approprié. Dans ce cas, le flux réinitialisation du mot de passe `b2c_1_reset`.

1. Étendez votre méthode de connexion comme suit :

    ```javascript
    function signIn() {
      myMSALObj.loginPopup(loginRequest)
        .then(loginResponse => {
            console.log("id_token acquired at: " + new Date().toString());

            if (myMSALObj.getAccount()) {
              updateUI();
            }

        }).catch(function (error) {
          console.log(error);

          // error handling
          if (error.errorMessage) {
            // check for forgot password error
            if (error.errorMessage.indexOf("AADB2C90118") > -1) {

              //call login method again with the password reset user flow
              myMSALObj.loginPopup(b2cPolicies.authorities.forgotPassword)
                .then(loginResponse => {
                  console.log(loginResponse);
                  window.alert("Password has been reset successfully. \nPlease sign-in with your new password.");
                })
            }
          }
        });
    }
    ```

1. L’extrait de code précédent montre comment afficher l’écran de réinitialisation du mot de passe après avoir intercepté l’erreur avec le code `AADB2C90118`.

    Après réinitialisation de son mot de passe, l’utilisateur est renvoyé à l’application afin de se reconnecter.

    :::image type="content" source="media/msal-b2c-overview/user-journey-02-password-reset.png" alt-text="Écrans du flux de réinitialisation du mot de passe affichés par Azure AD B2C" border="false":::

    Pour plus d’informations sur les codes d’erreur et la gestion des exceptions, consultez [Codes d’erreur et d’exception MSAL](msal-error-handling-js.md).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur ces concepts Azure AD B2C :

- [Flux d’utilisateurs](../../active-directory-b2c/tutorial-create-user-flows.md)
- [Stratégies personnalisées](../../active-directory-b2c/custom-policy-get-started.md)
- [Personnalisation de l’expérience utilisateur](../../active-directory-b2c/configure-user-input.md)
