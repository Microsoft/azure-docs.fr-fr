---
title: Code JavaScript de service approuvé
description: Il s’agit de la version JavaScript de la création d’un service approuvé pour Communication Services.
author: dademath
manager: nimag
services: azure-communication-services
ms.author: dademath
ms.date: 07/28/2020
ms.topic: include
ms.service: azure-communication-services
ms.openlocfilehash: daf2d675bbbee324769b6e1e8d8d34587d37c72f
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94886620"
---
## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. Pour plus d’informations, consultez [Créer un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- [Visual Studio Code](https://code.visualstudio.com/) sur l’une des [plateformes prises en charge](https://code.visualstudio.com/docs/supporting/requirements#_platforms).
- [Node.js](https://nodejs.org/), Active LTS et Maintenance LTS (version 10.14.1 recommandée). Utilisez la commande `node --version` pour vérifier la version que vous utilisez. 
- [Extension Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) pour Visual Studio Code. 
- Une chaîne de connexion et une ressource Communication Services actives. [Créer une ressource Communication Services](../../quickstarts/create-communication-resource.md)

## <a name="overview"></a>Vue d’ensemble

:::image type="content" source="../media/trusted-service-architecture.png" alt-text="Diagramme de l’architecture de service approuvé":::

Pour ce tutoriel, nous allons créer une fonction Azure qui servira de service de provisionnement de jetons approuvé. Vous pouvez utiliser ce tutoriel comme point de départ pour votre propre service de provisionnement de jetons.

Ce service est chargé d’authentifier les utilisateurs auprès d’Azure Communication Services. Les utilisateurs de vos applications Communication Services auront besoin d’un `Access Token` pour participer aux threads de conversation et aux appels VoIP. La fonction Azure servira d’intermédiaire approuvé entre l’utilisateur et Communication Services. Cela vous permet de provisionner des jetons d’accès sans exposer votre chaîne de connexion de ressource à vos utilisateurs.

Pour plus d’informations, consultez la documentation conceptuelle sur [l’architecture client-serveur](../../concepts/client-and-server-architecture.md) et [l’authentification et l’autorisation](../../concepts/authentication.md).

## <a name="setting-up"></a>Configuration

### <a name="azure-functions-set-up"></a>Configuration d’Azure Functions

Nous allons tout d’abord configurer la structure de base de notre fonction Azure. Vous trouverez ici des instructions pas à pas sur la configuration : [Créer une fonction à l’aide de Visual Studio Code](../../../azure-functions/create-first-function-vs-code-csharp.md?pivots=programming-language-javascript)

Notre fonction Azure nécessite la configuration suivante :

- Language : JavaScript
- Modèle : Déclencheur HTTP
- Niveau d’autorisation : anonyme (vous pourrez le changer ultérieurement si vous préférez un modèle d’autorisation différent)
- Nom de la fonction : défini par l’utilisateur

Après avoir suivi les [instructions relatives à Azure Functions](../../../azure-functions/create-first-function-vs-code-csharp.md?pivots=programming-language-javascript) avec la configuration ci-dessus, vous devez disposer d’un projet dans Visual Studio Code pour la fonction Azure avec un fichier `index.js` qui contient la fonction proprement dite. Le code à l’intérieur de ce fichier doit être le suivant :

```javascript

module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    const name = (req.query.name || (req.body && req.body.name));
    const responseMessage = name
        ? "Hello, " + name + ". This HTTP triggered function executed successfully."
        : "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.";

    context.res = {
        // status: 200, /* Defaults to 200 */
        body: responseMessage
    };
}

```

Nous allons maintenant procéder à l’installation des bibliothèques Azure Communication Services.

### <a name="install-communication-services-libraries"></a>Installer les bibliothèques Communication Services

Nous allons utiliser la bibliothèque `Administration` pour générer des `User Access Tokens`.

Utilisez la commande `npm install` pour installer la bibliothèque de client Administration Azure Communication Services pour JavaScript.

```console

npm install @azure/communication-administration --save

```

L’option `--save` liste la bibliothèque comme dépendance dans votre fichier **package.json**.

En haut du fichier `index.js`, importez l’interface du `CommunicationIdentityClient`.

```javascript
const { CommunicationIdentityClient } = require('@azure/communication-administration');
```

## <a name="access-token-generation"></a>Génération de jetons d’accès

Pour permettre à notre fonction Azure de générer des `User Access Tokens`, nous devons d’abord utiliser la chaîne de connexion pour notre ressource Communication Services.

Pour plus d’informations sur la récupération de votre chaîne de connexion, consultez le [guide de démarrage rapide sur le provisionnement des ressources](../../quickstarts/create-communication-resource.md).

``` javascript
const connectionString = 'INSERT YOUR RESOURCE CONNECTION STRING'
```

Nous allons ensuite modifier la fonction d’origine pour générer des `User Access Tokens`. 

Les `User Access Tokens` sont générés par la création d’un utilisateur à partir de la méthode `createUser`. Une fois l’utilisateur créé, nous pouvons utiliser la méthode `issueToken` afin de générer un jeton pour l’utilisateur retourné par la fonction Azure.

Pour cet exemple, nous allons définir `voip` comme étendue du jeton. D’autres étendues peuvent être nécessaires pour votre application. En savoir plus sur les [étendues](../../quickstarts/access-tokens.md).

```javascript
module.exports = async function (context, req) {
    let tokenClient = new CommunicationIdentityClient(connectionString);

    const user = await tokenClient.createUser();

    const userToken = await tokenClient.issueToken(user, ["voip"]);

    const response = {
        "User" : userToken.user,
        "Token": userToken.token,
        "ExpiresOn": userToken.expiresOn
    }

    context.res = {
        body: response
    };
}
```

Pour un `CommunicationUser` Communication Services existant, vous pouvez ignorer l’étape de création et simplement générer un jeton d’accès. Pour plus d’informations, consultez le [guide de démarrage rapide sur la création de jetons d’accès utilisateur](../../quickstarts/access-tokens.md).

## <a name="test-the-azure-function"></a>Tester la fonction Azure

Exécutez la fonction Azure localement à l’aide de `F5`. Cette opération initialise la fonction Azure localement et la rend accessible par le biais de `http://localhost:7071/api/FUNCTION_NAME`. Consultez la documentation supplémentaire sur l’[exécution locale](../../../azure-functions/create-first-function-vs-code-csharp.md?pivots=programming-language-javascript#run-the-function-locally).

Ouvrez l’URL dans votre navigateur. Vous devriez voir un corps de réponse avec l’ID d’utilisateur de communication, le jeton et l’expiration du jeton.

:::image type="content" source="../media/trusted-service-sample-response.png" alt-text="Capture d’écran montrant un exemple de réponse pour la fonction Azure créée.":::

## <a name="deploy-the-function-to-azure"></a>Déployer la fonction sur Azure

Pour déployer votre fonction Azure, vous pouvez suivre ces [instructions pas à pas](../../../azure-functions/create-first-function-vs-code-csharp.md?pivots=programming-language-javascript#sign-in-to-azure).

En général, vous devez :
1. Vous connecter à Azure à partir de Visual Studio.
2. Publier votre projet sur votre compte Azure. Ici, vous devrez choisir un abonnement existant.
3. Créer une ressource de fonction Azure à l’aide de l’Assistant Visual Studio ou utiliser une ressource existante. Pour une nouvelle ressource, vous devrez la configurer en fonction de la région, du runtime et de l’identificateur unique souhaités.
4. Attendre la fin du déploiement.
5. Exécuter la fonction. 🎉

## <a name="run-azure-function"></a>Exécuter la fonction Azure

Exécutez la fonction Azure à l’aide de l’URL `http://<function-appn-ame>.azurewebsites.net/api/<function-name>`.

Vous pouvez trouver l’URL en cliquant avec le bouton droit sur la fonction dans Visual Studio Code et en copiant l’URL de la fonction.

En savoir plus sur l’[exécution de votre fonction Azure](../../../azure-functions/create-first-function-vs-code-csharp.md?pivots=programming-language-javascript#run-the-function-in-azure)