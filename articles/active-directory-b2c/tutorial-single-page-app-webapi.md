---
title: 'Tutoriel : Protéger une API web Node.js avec Azure AD B2C et accorder l’accès à une application monopage'
titleSuffix: Azure AD B2C
description: Dans ce tutoriel, découvrez comment utiliser Active Directory B2C pour protéger une API web Node.js et l’appeler depuis une application monopage.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 04/04/2020
ms.custom: mvc, devx-track-js
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: 737810a7d07d0d97b2e42acffa17fdd32986c48b
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93421088"
---
# <a name="tutorial-protect-and-grant-access-to-a-nodejs-web-api-from-a-single-page-application-with-azure-ad-b2c"></a>Tutoriel : Protéger et accorder l’accès à une API web Node.js à partir d’une application monopage avec Azure AD B2C

Ce tutoriel vous montre comment appeler une API web Node.js protégée par Azure Active Directory B2C (Azure AD B2C) depuis une application monopage.

Dans ce tutoriel, qui est le deuxième d’une série de deux, vous allez :

> [!div class="checklist"]
> * Créer une inscription d’application d’API web dans votre locataire Azure AD B2C
> * Configurer des étendues pour l’API web
> * Accorder des autorisations à l’API web
> * Modifier un exemple de code d’API web pour qu’il fonctionne avec votre locataire

Dans le [premier tutoriel](tutorial-single-page-app.md) de cette série, vous avez téléchargé l’exemple de code et vous l’avez modifié pour connecter des utilisateurs avec un workflow utilisateur dans votre locataire Azure AD B2C.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* Effectuez les étapes et les prérequis du [Tutoriel : Activer l’authentification dans une application monopage avec Azure AD B2C](tutorial-single-page-app.md)
* [Visual Studio Code](https://code.visualstudio.com/) ou un autre éditeur de code
* [Node.JS](https://nodejs.org/en/download/)

## <a name="add-a-web-api-application"></a>Ajouter une application d’API web

[!INCLUDE [active-directory-b2c-appreg-webapi](../../includes/active-directory-b2c-appreg-webapi.md)]

## <a name="configure-scopes"></a>Configurer des étendues

Les étendues permettent de gérer l'accès aux ressources protégées. Elles sont utilisées par l’API web pour implémenter le contrôle d’accès basé sur les étendues. Par exemple, certains utilisateurs peuvent bénéficier d’un accès en lecture et en écriture tandis que d’autres peuvent disposer d’autorisations d’accès en lecture seule. Dans ce tutoriel, vous définissez des autorisations d’accès en lecture et en écriture pour l’API web.

[!INCLUDE [active-directory-b2c-scopes](../../includes/active-directory-b2c-scopes.md)]

Enregistrez la valeur sous **Étendues** pour l’étendue `demo.read` à utiliser plus tard, quand vous configurez l’application monopage. La valeur d’étendue complète est similaire à `https://contosob2c.onmicrosoft.com/api/demo.read`.

## <a name="grant-permissions"></a>Accorder des autorisations

Pour appeler une API web protégée à partir d’une autre application, vous devez accorder à cette application les autorisations d’accès à l’API web.

Dans le tutoriel sur les prérequis, vous avez créé une application monopage nommée *spaapp1*. Dans ce tutoriel, vous configurez cette application pour appeler l’API web que vous avez créée dans une section précédente, *spaapp1*.

[!INCLUDE [active-directory-b2c-permissions-api](../../includes/active-directory-b2c-permissions-api.md)]

Votre application monopage a maintenant reçu des autorisations sur l’API web protégée pour les étendues spécifiées. Un utilisateur s’authentifie auprès d’Azure AD B2C pour utiliser l’application monopage. L’application monopage obtient un jeton d’accès d’Azure AD B2C pour accéder à l’API web protégée.

## <a name="configure-the-sample"></a>Configurer l'exemple

Maintenant que l’API web est inscrite et que vous avez défini des étendues, configurez le code de l’API web pour qu’il fonctionne avec votre locataire Azure AD B2C. Dans ce tutoriel, vous configurez un exemple d’API web Node.js que vous pouvez télécharger à partir de GitHub.

[Téléchargez une \*archive .zip](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi/archive/master.zip) ou clonez l’exemple de projet d’API web à partir de GitHub. Vous pouvez également accéder directement au projet [Azure-Samples/active-directory-b2c-javascript-nodejs-webapi](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi) sur GitHub.

```console
git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi.git
```

### <a name="configure-the-web-api"></a>Configurer l’API web

1. Ouvrez le fichier *config.json* dans votre éditeur de code.
1. Modifiez les valeurs des variables de façon à refléter celles de l’inscription de l’application que vous avez créée précédemment. Mettez également à jour le `policyName` avec le workflow utilisateur que vous avez créé dans le cadre des prérequis. Par exemple, *B2C_1_signupsignin1*.
    
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

#### <a name="enable-cors"></a>Activez CORS

Pour autoriser votre application monopage à appeler l’API web Node.js, vous devez activer [CORS](https://expressjs.com/en/resources/middleware/cors.html) dans l’API web. Dans une application de production, vous devez être attentif au domaine qui effectue la demande, mais pour ce tutoriel, autorisez les demandes provenant de n’importe quel domaine.

Pour activer CORS, utilisez l’intergiciel (middleware) suivant. Dans l’exemple de code de l’API Web Node.js de ce tutoriel, il a déjà été ajouté au fichier *index.js*.

```javascript
app.use((req, res, next) => {
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "Authorization, Origin, X-Requested-With, Content-Type, Accept");
    next();
});
```

### <a name="configure-the-single-page-application"></a>Configurer l’application monopage

L’application monopage du [tutoriel précédent](tutorial-single-page-app.md) de la série utilise Azure AD B2C pour l’inscription et la connexion des utilisateurs, et par défaut, elle appelle l’API web Node.js protégée par le locataire de démonstration *fabrikamb2c*.

Dans cette section, vous mettez à jour l’application monopage pour appeler l’API web Node.js protégée par *votre* locataire Azure AD B2C (et que vous exécutez sur votre machine locale).

Pour modifier les paramètres dans l’application monopage :

1. Dans le projet [active-directory-b2c-javascript-msal-singlepageapp][github-js-spa] que vous avez téléchargé ou cloné dans le tutoriel précédent, ouvrez le fichier *apiConfig.js* dans le projet *JavaScriptSPA*.
1. Configurez l’exemple avec l’URI pour l’étendue *demo.read* que vous avez créée précédemment et l’URL de l’API web.
    1. Dans la définition de `apiConfig`, remplacez la valeur de `b2cScopes` par l’URI complet de l’étendue *demo.read* (la valeur **Étendue** que vous avez notée précédemment).
    1. Remplacez le domaine dans le valeur `webApi` par l’URI de redirection que vous avez ajouté lors de l’inscription de l’application d’API web dans une étape précédente.

    Étant donné que l’API est accessible au point de terminaison `/hello`, laissez */hello* dans l’URI.

    La définition de `apiConfig` doit être similaire au bloc de code suivant, mais avec le nom de votre locataire B2C à la place de `<your-tenant-name>` :

    ```javascript
    // The current application coordinates were pre-registered in a B2C tenant.
    const apiConfig = {
      b2cScopes: ["https://<your-tenant-name>.onmicrosoft.com/api/demo.read"],
      webApi: "http://localhost:5000/hello" // '/hello' should remain in the URI
    };
    ```

## <a name="run-the-spa-and-web-api"></a>Exécuter l’application monopage et l’API web

Vous êtes maintenant prêt à tester l’accès délimité de l’application monopage à l’API. Enfin, vous exécutez à la fois l’API web Node.js et l’exemple d’application monopage JavaScript sur votre machine locale. Ensuite, vous vous connectez à l’application monopage et vous sélectionnez le bouton **Call API** (Appeler l’API) pour envoyer une demande à l’API protégée.

Bien que les deux applications s’exécutent localement dans ce tutoriel, vous les avez configurées pour qu’elles utilisent Azure AD B2C pour sécuriser l’inscription/la connexion et pour accorder l’accès à l’API web protégée.

### <a name="run-the-nodejs-web-api"></a>Exécuter l’API web Node.js

1. Ouvrez une fenêtre de console et accédez au répertoire contenant l’exemple d’API web Node.js. Par exemple :

    ```console
    cd active-directory-b2c-javascript-nodejs-webapi
    ```

1. Exécutez les commandes suivantes :

    ```console
    npm install && npm update
    node index.js
    ```

    La fenêtre de console affiche le numéro de port de l’endroit où est hébergée l’application.

    ```console
    Listening on port 5000...
    ```

### <a name="run-the-single-page-app"></a>Exécuter l’application monopage

1. Ouvrez une autre fenêtre de console et accédez au répertoire contenant l’exemple d’application monopage JavaScript. Par exemple :

    ```console
    cd ms-identity-b2c-javascript-spa
    ```

1. Exécutez les commandes suivantes :

    ```console
    npm install && npm update
    npm start
    ```

    La fenêtre de console affiche le numéro de port de l’endroit où est hébergée l’application.

    ```console
    Listening on port 6420...
    ```

1. Accédez à `http://localhost:6420` dans votre navigateur pour voir l’application.

    ![Exemple d’application monopage affichée dans un navigateur](./media/tutorial-single-page-app-webapi/tutorial-01-sample-app-browser.png)

1. Connectez-vous en utilisant l’adresse e-mail et le mot de passe que vous avez utilisés dans le [tutoriel précédent](tutorial-single-page-app.md). Une fois la connexion établie, vous voyez normalement le message `User 'Your Username' logged-in`.
1. Sélectionnez le bouton **Appeler l’API**. L’application monopage obtient un octroi d’autorisation d’Azure AD B2C, puis accède à l’API web protégée pour afficher le nom de l’utilisateur connecté :

    ![Application monopage dans le navigateur montrant le résultat JSON avec le nom d’utilisateur retourné par l’API](./media/tutorial-single-page-app-webapi/tutorial-02-call-api.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez :

> [!div class="checklist"]
> * Créé une inscription d’application d’API web dans votre locataire Azure AD B2C
> * Configuré des étendues pour l’API web
> * Accordé des autorisations à l’API web
> * Modifié un exemple de code d’API web pour qu’il fonctionne avec votre locataire

Maintenant que vous avez vu une application monopage demander une ressource à partir d’une API web protégée, vous pouvez mieux comprendre comment ces types d’application interagissent entre elles et avec Azure AD B2C.

> [!div class="nextstepaction"]
> [Types d’applications pouvant être utilisés dans Azure Active Directory B2C >](application-types.md)

<!-- Links - EXTERNAL -->
[github-js-spa]: https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp
