---
title: 'Tutoriel : Accorder l’accès à une API web ASP.NET'
titleSuffix: Azure AD B2C
description: Tutoriel sur l’utilisation d’Active Directory B2C pour protéger une API web ASP.NET et l’appeler à partir d’une application web ASP.NET.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 10/14/2019
ms.custom: devx-track-csharp, mvc
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: ada4226500437e7733605a29988ee7cacae40761
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994308"
---
# <a name="tutorial-grant-access-to-an-aspnet-web-api-using-azure-active-directory-b2c"></a>Tutoriel : Accorder l’accès à une API web ASP.NET à l’aide d’Azure Active Directory B2C

Ce tutoriel vous montre comment appeler une ressource d’API web protégée dans Azure Active Directory B2C (Azure AD B2C) à partir d’une application web ASP.NET.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Ajouter une application d’API web
> * Configurer des étendues pour une API web
> * Accorder des autorisations à l’API web
> * Configurer l’exemple pour utiliser l’application

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Effectuez les étapes et les prérequis du [Tutoriel : Activer l’authentification dans une application web à l’aide d’Azure Active Directory B2C](tutorial-web-app-dotnet.md).

## <a name="add-a-web-api-application"></a>Ajouter une application d’API web

Les ressources des API web doivent être inscrites auprès de votre locataire avant de pouvoir accepter et répondre aux demandes de ressources protégées émanant des applications clientes qui présentent un jeton d'accès.

Pour inscrire une application dans votre locataire Azure AD B2C, vous pouvez utiliser notre nouvelle expérience unifiée **Inscriptions d’applications** ou notre expérience héritée **Applications (héritées)** . [En savoir plus sur la nouvelle expérience](./app-registrations-training-guide.md).

#### <a name="app-registrations"></a>[Inscriptions des applications](#tab/app-reg-ga/)

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez le filtre **Annuaire et abonnement** dans le menu supérieur, puis l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans le menu de gauche, sélectionnez **Azure AD B2C**. Ou sélectionnez **Tous les services**, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Inscriptions d’applications**, puis **Nouvelle inscription**.
1. Entrez un **Nom** pour l’application. Par exemple, *webapi1*.
1. Sous **URI de redirection**, sélectionnez **Web**, puis entrez un point de terminaison où Azure AD B2C doit retourner les jetons demandés par votre application. Dans ce tutoriel, l'exemple s'exécute localement et écoute `https://localhost:44332`.
1. Sélectionnez **Inscription**.
1. Enregistrez l’**ID d’application (client)** pour l’utiliser dans une étape ultérieure.

#### <a name="applications-legacy"></a>[Applications (héritées)](#tab/applications-legacy/)

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD B2C en sélectionnant le filtre **Annuaire + abonnement** dans le menu du haut et en choisissant l’annuaire qui contient votre locataire.
3. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
4. Sélectionnez **Applications (héritées)** , puis **Ajouter**.
5. Entrez un nom pour l’application. Par exemple, *webapi1*.
6. Pour **Inclure une application/API web**, sélectionnez **Oui**.
7. Pour l’**URL de réponse**, entrez un point de terminaison où Azure AD B2C doit retourner les jetons demandés par votre application. Dans ce tutoriel, l'exemple s'exécute localement et écoute `https://localhost:44332`.
8. Pour l’**L’URI de l’ID d’application**, entrez l’identificateur utilisé pour votre API web. L’identificateur complet URI, y compris le domaine, est généré pour vous. Par exemple : `https://contosotenant.onmicrosoft.com/api`.
9. Cliquez sur **Créer**.
10. Sur la page des propriétés, enregistrez l'ID d'application que vous utiliserez pour configurer l'application web.

* * *

## <a name="configure-scopes"></a>Configurer des étendues

Les étendues permettent de gérer l'accès aux ressources protégées. Elles sont utilisées par l’API web pour implémenter le contrôle d’accès basé sur les étendues. Par exemple, les utilisateurs de l’API web peuvent avoir un accès en lecture et en écriture, ou les utilisateurs de l’API web peuvent avoir l’accès en lecture uniquement. Dans ce didacticiel, vous allez utiliser des étendues pour définir des autorisations d’accès en lecture et en écriture pour l’API web.

[!INCLUDE [active-directory-b2c-scopes](../../includes/active-directory-b2c-scopes.md)]

## <a name="grant-permissions"></a>Accorder des autorisations

Pour appeler une API web protégée à partir d'une application, vous devez accorder à cette application les autorisations d'accès à l'API. Dans le cadre du tutoriel de prérequis, vous avez créé dans Azure AD B2C une application web nommée *webapp1*. Vous allez utiliser cette application pour appeler l'API web.

[!INCLUDE [active-directory-b2c-permissions-api](../../includes/active-directory-b2c-permissions-api.md)]

Votre application est inscrite pour appeler l'API web protégée. Un utilisateur s'authentifie auprès d'Azure AD B2C pour utiliser l'application. L'application obtient d'Azure AD B2C l'autorisation d'accéder à l'API web protégée.

## <a name="configure-the-sample"></a>Configurer l'exemple

Maintenant que l’API web est inscrite et que les étendues sont définies, vous devez configurer l’API web pour utiliser votre locataire Azure AD B2C. Dans ce didacticiel, vous allez configurer un exemple d’API web. L’exemple d’API web est inclus dans le projet que vous avez téléchargé dans le tutoriel des prérequis.

L’exemple de solution contient deux projets :

* **TaskWebApp** : permettant de créer et de modifier une liste de tâches. L’exemple utilise le flux d’utilisateur d’**inscription ou de connexion** pour inscrire ou connecter des utilisateurs.
* **TaskService** : prend en charge les fonctionnalités de création, de lecture, de mise à jour et de suppression des listes de tâches. L’API est protégée par Azure AD B2C et appelée par TaskWebApp.

### <a name="configure-the-web-application"></a>Configurer l’application web

1. Ouvrez la solution **B2C-WebAPI-DotNet** dans Visual Studio.
1. Dans le projet **TaskWebApp**, ouvrez **Web.config**.
1. Pour exécuter l’API localement, utilisez le paramètre localhost pour **api:TaskServiceUrl**. Modifiez le fichier Web.config comme suit :

    ```csharp
    <add key="api:TaskServiceUrl" value="https://localhost:44332/"/>
    ```

1. Configurez l’URI de l’API. Il s’agit de l’URI que l’application web utilise pour effectuer la demande d’API. Configurez également les autorisations demandées.

    ```csharp
    <add key="api:ApiIdentifier" value="https://<Your tenant name>.onmicrosoft.com/api/" />
    <add key="api:ReadScope" value="demo.read" />
    <add key="api:WriteScope" value="demo.write" />
    ```

### <a name="configure-the-web-api"></a>Configurer l’API web

1. Dans le projet **TaskService**, ouvrez **Web.config**.
1. Configurez l’API pour utiliser votre client.

    ```csharp
    <add key="ida:AadInstance" value="https://<Your tenant name>.b2clogin.com/{0}/{1}/v2.0/.well-known/openid-configuration" />
    <add key="ida:Tenant" value="<Your tenant name>.onmicrosoft.com" />
    ```

1. Définissez l’ID client sur l’ID d’application de votre application d’API web inscrite, *webapi1*.

    ```csharp
    <add key="ida:ClientId" value="<application-ID>"/>
    ```

1. Mettez à jour le paramètre de flux utilisateur avec le nom votre flux utilisateur d’inscription et de connexion, *B2C_1_signupsignin1*.

    ```csharp
    <add key="ida:SignUpSignInPolicyId" value="B2C_1_signupsignin1" />
    ```

1. Configurez les paramètres d’étendues de sorte qu’ils correspondent à ceux que vous avez créés sur le portail.

    ```csharp
    <add key="api:ReadScope" value="demo.read" />
    <add key="api:WriteScope" value="demo.write" />
    ```

## <a name="run-the-sample"></a>Exécution de l'exemple

Vous devez exécuter les projets **TaskWebApp** et **TaskService**.

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur la solution, puis sélectionnez **Définir les projets de démarrage...** .
1. Sélectionnez **Plusieurs projets de démarrage**.
1. Pour les deux projets, définissez le paramètre **Action** sur **Démarrer**.
1. Cliquez sur **OK** pour enregistrer la configuration.
1. Appuyez sur la touche **F5** pour exécuter les deux applications. Chaque application s’ouvre dans sa propre fenêtre de navigateur.
    * `https://localhost:44316/` est l’application web.
    * `https://localhost:44332/` est l’API web.

1. Dans l’application web, sélectionnez **S’inscrire/Se connecter** pour vous connecter à l’application web. Utilisez le compte que vous avez créé précédemment.
1. Une fois connecté, sélectionnez **Liste de tâches** pour créer un élément de liste de tâches.

Lorsque vous créez un élément de liste de tâches, l’application web envoie une requête à l’API web pour le générer. L’application web que vous avez protégée appelle l’API web protégée par Azure AD B2C.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Ajouter une application d’API web
> * Configurer des étendues pour une API web
> * Accorder des autorisations à l’API web
> * Configurer l’exemple pour utiliser l’application

> [!div class="nextstepaction"]
> [Tutoriel : Ajouter des fournisseurs d’identité à vos applications dans Azure Active Directory B2C](tutorial-add-identity-providers.md)