---
title: 'Tutoriel : Accorder l’accès à une API web Node.js à partir d’une application de bureau'
description: Tutoriel sur l’utilisation d’Active Directory B2C pour protéger une API web Node.js et l’appeler à partir d’une application de bureau .NET.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 10/12/2019
ms.custom: mvc
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: 8eca6297a704e1ef478c09baf227cf622d6890bb
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94953149"
---
# <a name="tutorial-grant-access-to-a-nodejs-web-api-from-a-desktop-app-using-azure-active-directory-b2c"></a>Tutoriel : Accorder l’accès à une API web Node.js depuis une application de bureau à l’aide d’Azure Active Directory B2C

Ce didacticiel vous montre comment appeler une API web Node.js protégée par Azure Active Directory B2C (Azure AD B2C) à partir d’une application de bureau Windows Presentation Foundation (WPF) également protégée par Azure AD B2C.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Ajouter une application d’API web
> * Configurer des étendues pour une API web
> * Accorder des autorisations à l’API web
> * Mettre à jour l’exemple pour utiliser l’application

## <a name="prerequisites"></a>Prérequis

Effectuez les étapes et les prérequis du [Tutoriel : Authentifiez les utilisateurs dans un client de bureau natif](tutorial-desktop-app.md).

## <a name="add-a-web-api-application"></a>Ajouter une application d’API web

[!INCLUDE [active-directory-b2c-appreg-webapi](../../includes/active-directory-b2c-appreg-webapi.md)]

## <a name="configure-scopes"></a>Configurer des étendues

Les étendues permettent de gérer l'accès aux ressources protégées. Elles sont utilisées par l’API web pour implémenter le contrôle d’accès basé sur les étendues. Par exemple, certains utilisateurs peuvent bénéficier d’un accès en lecture et en écriture tandis que d’autres peuvent disposer d’autorisations d’accès en lecture seule. Dans ce didacticiel, vous allez définir des autorisations d’accès en lecture et en écriture pour l’API web.

[!INCLUDE [active-directory-b2c-scopes](../../includes/active-directory-b2c-scopes.md)]

Notez la valeur de **ÉTENDUES** de l’étendue `demo.read` pour l’utiliser plus tard, quand vous configurerez l’application de bureau. La valeur d’étendue complète est similaire à `https://contosob2c.onmicrosoft.com/api/demo.read`.

## <a name="grant-permissions"></a>Accorder des autorisations

Pour appeler une API web protégée à partir d’une application client native, vous devez octroyer à l’application cliente native inscrite des autorisations sur l’API web que vous avez inscrite dans Azure AD B2C.

Dans le didacticiel des conditions requises, vous avez inscrit une application cliente native appelée *nativeapp1*. Les étapes suivantes configurent l’inscription de cette application native avec les étendues d’API que vous avez exposées pour *webapi1* dans la section précédente. Cela permet à l’application de bureau d’obtenir un jeton d’accès de la part d’Azure AD B2C que l’API web peut utiliser pour vérifier et fournir un accès délimité à ses ressources. Vous configurez et exécutez les exemples de code de l’application de bureau et de l’API web plus loin dans le didacticiel.

Pour inscrire une application dans votre locataire Azure AD B2C, vous pouvez utiliser notre nouvelle expérience unifiée **Inscriptions d’applications** ou notre expérience héritée **Applications (héritées)** . [En savoir plus sur la nouvelle expérience](./app-registrations-training-guide.md).

#### <a name="app-registrations"></a>[Inscriptions des applications](#tab/app-reg-ga/)

1. Sélectionnez **Inscriptions d’applications**, puis sélectionnez l’application cliente native qui doit avoir accès à l’API. Par exemple, *nativeapp1*.
1. Sous **Gérer**, sélectionnez **Autorisations de l’API**.
1. Sous **Autorisations configurées**, sélectionnez **Ajouter une autorisation**.
1. Sélectionnez l’onglet **Mes API**.
1. Sélectionnez l’API à laquelle l’application cliente native doit être autorisée à accéder. Par exemple, *webapi1*.
1. Sous **Autorisation**, développez **demo**, puis sélectionnez les étendues que vous avez définies auparavant. Par exemple, *demo.read* et *demo.write*.
1. Sélectionnez **Ajouter des autorisations**. Comme vous l’indiquent les instructions, patientez quelques minutes avant de passer à l’étape suivante.
1. Sélectionnez **Accorder le consentement de l’administrateur pour (nom de votre abonné)** .
1. Sélectionnez le compte administrateur actuellement connecté ou connectez-vous avec un compte de votre locataire Azure AD B2C qui possède au minimum le rôle *Administrateur d’application cloud*.
1. Sélectionnez **Accepter**.
1. Sélectionnez **Actualiser**, puis vérifiez que la mention « Accordé pour ... » apparaît dans **État** pour les deux étendues. La propagation des autorisations peut prendre quelques minutes.

#### <a name="applications-legacy"></a>[Applications (héritées)](#tab/applications-legacy/)

1. Sélectionnez **Applications (héritées)** , puis *nativeapp1*.
1. Sélectionnez **Accès aux API**, puis **Ajouter**.
1. Dans la liste déroulante **Sélectionner une API**, sélectionnez *webapi1*.
1. Dans la liste déroulante **Sélectionner des étendues**, sélectionnez les étendues que vous avez définies précédemment. Par exemple, *demo.read* et *demo.write*.
1. Sélectionnez **OK**.

* * *

Un utilisateur s’authentifie auprès d’Azure AD B2C pour utiliser l’application de bureau WPF. L’application de bureau obtient un octroi d’autorisation d’Azure AD B2C pour accéder à l’API web protégée.

## <a name="configure-the-samples"></a>Configurer les exemples

L’API web étant désormais inscrite et les étendues et autorisations configurées, vous pouvez configurer les exemples de l’application de bureau et de l’API web pour qu’ils utilisent votre locataire Azure AD B2C.

### <a name="update-the-desktop-application"></a>Mettre à jour l’application de bureau

Dans les prérequis pour cet article, vous avez modifié une [application de bureau WPF](https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop) de manière à activer la connexion avec un flux d’utilisateur dans votre locataire Azure AD B2C. Dans cette section, vous modifiez cette même application pour qu’elle fasse référence à l’API web que vous avez inscrite précédemment, *webapi1*.

1. Ouvrez la solution **active-directory-b2c-wpf** (`active-directory-b2c-wpf.sln`) dans Visual Studio.
1. Dans le projet **active-directory-b2c-wpf**, ouvrez le fichier *App.xaml.cs* et recherchez les définitions de variables suivantes.
    1. Remplacez la valeur de la variable `ApiScopes` par la valeur notée précédemment lorsque vous avez défini l’étendue **demo.read**.
    1. Remplacez la valeur de la variable `ApiEndpoint` par l’**URI de redirection** que vous avez notée précédemment lorsque vous avez inscrit l’API web (par exemple, *webapi1*) dans votre locataire.

    Voici un exemple :

    ```csharp
    public static string[] ApiScopes = { "https://contosob2c.onmicrosoft.com/api/demo.read" };
    public static string ApiEndpoint = "http://localhost:5000";
    ```

### <a name="get-and-update-the-nodejs-api-sample"></a>Obtenir et mettre à jour l’exemple de l’API Node.js

Obtenez ensuite l’exemple de code de l’API web Node.js auprès de GitHub et configurez-le de façon à utiliser l’API web que vous avez inscrite dans votre locataire Azure AD B2C.

[Téléchargez un fichier zip ](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi/archive/master.zip) ou clonez l’exemple d’application web à partir de GitHub.

```console
git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi.git
```

L’exemple d’API web Node.js utilise la bibliothèque Passport.js pour activer Azure AD B2C afin de protéger les appels vers l’API.

1. Ouvrez le fichier `index.js` .
1. Mettez à jour ces définitions de variables avec les valeurs suivantes. Remplacez `<web-API-application-ID>` par l’**ID d’application (client)** de l’API web que vous avez inscrite précédemment (*webapi1*). Remplacez `<your-b2c-tenant>` par le nom de votre locataire Azure AD B2C.

    ```nodejs
    var clientID = "<web-API-application-ID>";
    var b2cDomainHost = "<your-b2c-tenant>.b2clogin.com";
    var tenantIdGuid = "<your-b2c-tenant>.onmicrosoft.com";
    var policyName = "B2C_1_signupsignin1";
    ```
1. Étant donné que vous exécutez l’API localement, indiquez `/` pour le chemin d’accès dans l’itinéraire pour la méthode GET au lieu de l’emplacement de l’application de démonstration `/hello` :

    ```nodejs
    app.get("/",
    ```

## <a name="run-the-samples"></a>Exécuter des exemples

### <a name="run-the-nodejs-web-api"></a>Exécuter l’API web Node.js

1. Lancez une invite de commande Node.js.
2. Accédez au répertoire contenant l’exemple Node.js. Par exemple `cd c:\active-directory-b2c-javascript-nodejs-webapi`
3. Exécutez les commandes suivantes :
    ```console
    npm install && npm update
    ```
    ```console
    node index.js
    ```

### <a name="run-the-desktop-application"></a>Exécuter l’application de bureau

1. Ouvrez la solution **active directory b2c wpf** dans Visual Studio.
2. Appuyez sur **F5** pour exécuter l’application de bureau.
3. Connectez-vous à l’aide de l’adresse e-mail et du mot de passe utilisés dans le [didacticiel Activer l’authentification d’application de bureau avec des comptes à l’aide d’Azure Active Directory B2C](tutorial-desktop-app.md).
4. Sélectionnez le bouton **Appeler l’API**.

L’application de bureau envoie une requête à l’API web exécutée localement et, après vérification d’un jeton d’accès valide, affiche le nom complet de l’utilisateur connecté.

![Nom complet affiché dans le volet supérieur de l’application de bureau WPF](./media/tutorial-desktop-app-webapi/desktop-app-01-post-api-call.png)

Votre application de bureau, protégée par Azure AD B2C, appelle l’API web exécutée localement, qui est également protégée par Azure AD B2C.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Ajouter une application d’API web
> * Configurer des étendues pour une API web
> * Accorder des autorisations à l’API web
> * Mettre à jour l’exemple pour utiliser l’application

> [!div class="nextstepaction"]
> [Tutoriel : Ajouter des fournisseurs d’identité à vos applications dans Azure Active Directory B2C](tutorial-add-identity-providers.md)