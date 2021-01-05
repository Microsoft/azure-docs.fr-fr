---
title: 'Démarrage rapide : Ajouter l’authentification à une application web Node avec MSAL Node | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce guide de démarrage rapide, vous allez découvrir comment implémenter l’authentification avec une application web Node.js et la bibliothèque d’authentification Microsoft (MSAL) pour Node.js.
services: active-directory
author: amikuma
manager: saeeda
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 10/22/2020
ms.author: amikuma
ms.custom: aaddev, scenarios:getting-started, languages:js, devx-track-js
ms.openlocfilehash: e860a33ba44743dcc5a96d7656f22fd952a1d0b8
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96762565"
---
# <a name="quickstart-sign-in-users-and-get-an-access-token-in-a-node-web-app-using-the-auth-code-flow"></a>Démarrage rapide : Connecter des utilisateurs et obtenir un jeton d’accès dans une application web Node à l’aide du flux de code d’authentification

Dans ce guide de démarrage rapide, vous téléchargez et exécutez un exemple de code qui montre comment une application web Node.js peut connecter des utilisateurs en utilisant le flux du code d’autorisation. Cet exemple de code montre également comment obtenir un jeton d’accès pour appeler l’API Microsoft Graph. 

Consultez [Fonctionnement de l’exemple](#how-the-sample-works) pour obtenir une illustration.

Ce guide de démarrage rapide utilise la bibliothèque d’authentification Microsoft pour Node.js (MSAL Node) avec le flux de code d’autorisation.

> [!IMPORTANT]
> MSAL Node [!INCLUDE [PREVIEW BOILERPLATE](../../../includes/active-directory-develop-preview.md)]

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure – [Créer un abonnement Azure gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [Node.JS](https://nodejs.org/en/download/)
* [Visual Studio Code](https://code.visualstudio.com/download) ou un autre éditeur de code

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-application"></a>Inscrire et télécharger votre application de démarrage rapide
>
> #### <a name="step-1-register-your-application"></a>Étape 1 : Inscrivez votre application
>
> 1. Connectez-vous au [portail Azure](https://portal.azure.com).
> 1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire dans lequel vous voulez inscrire une application.
> 1. Sous **Gérer**, sélectionnez **Inscriptions d’applications** > **Nouvelle inscription**.
> 1. Entrez un **nom** pour votre application. Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
> 1. Sous **Types de comptes pris en charge**, sélectionnez **Comptes dans un annuaire organisationnel et comptes personnels Microsoft**.
> 1. Choisissez `http://localhost:3000/redirect` comme valeur pour **URI de redirection**.
> 1. Sélectionnez **Inscription**. 
> 1. Dans la page **Vue d’ensemble**, notez la valeur de **ID d’application (client)** pour une utilisation ultérieure.
> 1. Sous **Gérer**, sélectionnez **Certificats et secrets** > **Nouveau secret client**.  Laissez la description vide et conservez l’expiration par défaut, puis sélectionnez **Ajouter**.
> 1. Notez la **Valeur** de la **clé secrète client** pour une utilisation ultérieure.

#### <a name="step-2-download-the-project"></a>Étape 2 : Téléchargez le projet

> [!div renderon="docs"]
> Pour exécuter le projet avec un serveur web en utilisant Node.js, [téléchargez les fichiers principaux du projet](https://github.com/Azure-Samples/ms-identity-node/archive/main.zip).

> [!div renderon="portal" class="sxs-lookup"]
> Exécuter le projet avec un serveur web en utilisant Node.js

> [!div renderon="portal" class="sxs-lookup" id="autoupdate" class="nextstepaction"]
> [Téléchargez l’exemple de code](https://github.com/Azure-Samples/ms-identity-node/archive/main.zip).

> [!div renderon="docs"]
> #### <a name="step-3-configure-your-node-app"></a>Étape 3 : Configurer votre application Node
>
> Extrayez le projet, ouvrez le dossier *ms-identity-node-main*, puis ouvrez le fichier *index.js*.
> Affectez l’**ID d’application (client)** comme valeur de `clientID`.
> Affectez la **Valeur** de la **Clé secrète client** comme valeur de `clientSecret`.
>
>```javascript
>const config = {
>    auth: {
>        clientId: "Enter_the_Application_Id_Here",
>        authority: "https://login.microsoftonline.com/common",
>        clientSecret: "Enter_the_Client_Secret_Here"
>    },
>    system: {
>        loggerOptions: {
>            loggerCallback(loglevel, message, containsPii) {
>                console.log(message);
>            },
>            piiLoggingEnabled: false,
>            logLevel: msal.LogLevel.Verbose,
>        }
>    }
>};
> ```

> [!div renderon="docs"]
>
> Modifiez les valeurs de la section `config` de la façon suivante :
>
> - `Enter_the_Application_Id_Here` est **l’ID d’application (client)** de l’application que vous avez inscrite.
> - `Enter_the_Client_Secret_Here` est la valeur **Valeur** de la **Clé secrète client** pour l’application que vous avez inscrite.
>
> La valeur `authority` par défaut représente le cloud Azure principal (global) :
>
> ```javascript
> authority: "https://login.microsoftonline.com/common",
> ```
>
> > [!TIP]
> > Pour connaître les valeurs de l’**ID d’application (client)** , consultez la page **Vue d’ensemble** de l’inscription d’application dans le portail Azure. Accédez à **Certificats et secrets** pour récupérer ou générer une nouvelle **Clé secrète client**.
>
> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Étape 3 : Votre application est configurée et prête à être exécutée
>
> [!div renderon="docs"]
>
> #### <a name="step-4-run-the-project"></a>Étape 4 : Exécuter le projet

Exécutez le projet à l’aide de Node.js :

1. Pour démarrer le serveur, exécutez les commandes suivantes dans le répertoire du projet :
    ```console
    npm install
    npm start
    ```
1. Accédez à `http://localhost:3000/`.

1. Sélectionnez **Se connecter** pour démarrer le processus de connexion.

    La première fois que vous vous connectez, vous êtes invité à autoriser l’application à accéder à votre profil et à vous connecter. Une fois que vous êtes connecté, un message de journal s’affiche sur la ligne de commande.

## <a name="more-information"></a>Informations complémentaires

### <a name="how-the-sample-works"></a>Fonctionnement de l’exemple

L’exemple, lorsqu’il est exécuté, héberge un serveur web sur localhost, port 3000. Quand un navigateur web accède à ce site, l’exemple redirige immédiatement l’utilisateur vers une page d’authentification Microsoft. Pour cette raison, l’exemple ne contient aucun élément *html* ni élément d’affichage. En cas de réussite de l’authentification, le message « OK » s’affiche.

### <a name="msal-node"></a>MSAL Node

La bibliothèque MSAL Node connecte les utilisateurs et demande les jetons utilisés pour accéder à une API protégée par la plateforme d’identités Microsoft. Vous pouvez télécharger la dernière version à l’aide du gestionnaire de package (npm) Node.js :

```console
npm install @azure/msal-node
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Ajout de l’authentification à une application web existante - Exemple de code GitHub >](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-node-samples/standalone-samples/auth-code)
