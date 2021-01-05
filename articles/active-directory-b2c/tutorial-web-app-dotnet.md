---
title: 'Tutoriel : Activer l’authentification dans une application web'
titleSuffix: Azure AD B2C
description: Didacticiel sur l’utilisation d’Azure Active Directory B2C pour fournir une connexion utilisateur pour une application web ASP.NET.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 10/02/2020
ms.custom: devx-track-csharp, mvc
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: 9c3c63b6116e02e8a742b69e90c11e182d72ab2e
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94953026"
---
# <a name="tutorial-enable-authentication-in-a-web-application-using-azure-active-directory-b2c"></a>Tutoriel : Activer l’authentification dans une application web à l’aide d’Azure Active Directory B2C

Ce tutoriel vous montre comment utiliser Azure Active Directory B2C (Azure AD B2C) pour connecter et inscrire des utilisateurs dans une application web ASP.NET. Azure AD B2C permet à vos applications de s’authentifier auprès de comptes de réseaux sociaux, de comptes d’entreprise et de comptes Azure Active Directory à l’aide de protocoles standards ouverts.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Mettre à jour l’application dans Azure AD B2C
> * Configurer l’exemple pour utiliser l’application
> * S’inscrire à l’aide du flux utilisateur

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

> [!NOTE]
> Ce tutoriel utilise un exemple d’application web ASP.NET. Pour d’autres exemples d’applications (y compris ASP.NET Core, Node.js, Python, etc.), consultez [Exemples de code Azure Active Directory B2C](code-samples.md).

## <a name="prerequisites"></a>Prérequis

* [Créer des flux d’utilisateur](tutorial-create-user-flows.md) pour activer les expériences utilisateur dans votre application
* Installer [Visual Studio 2019](https://www.visualstudio.com/downloads/) avec la charge de travail **Développement ASP.NET et web**.

## <a name="update-the-application-registration"></a>Mettre à jour l’inscription d’application

Au cours du tutoriel que vous avez effectué dans le cadre des prérequis, vous avez inscrit une application web dans Azure AD B2C. Pour permettre la communication avec l’exemple de ce tutoriel, vous devez ajouter un URI de redirection et créer une clé secrète client pour l’application inscrite.

### <a name="add-a-redirect-uri-reply-url"></a>Ajoutez un URI de redirection (URL de réponse)

Pour mettre à jour une application dans votre locataire Azure AD B2C, vous pouvez utiliser notre nouvelle expérience unifiée **Inscriptions d’applications** ou notre expérience héritée **Applications (héritées)** . [En savoir plus sur la nouvelle expérience](./app-registrations-training-guide.md).

#### <a name="app-registrations"></a>[Inscriptions des applications](#tab/app-reg-ga/)

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez le filtre **Annuaire et abonnement** dans le menu supérieur, puis l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans le menu de gauche, sélectionnez **Azure AD B2C**. Ou sélectionnez **Tous les services**, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Inscriptions d’applications**, sélectionnez l’onglet **Applications détenues**, puis l’application *webapp1*.
1. Sous **Web**, sélectionnez le lien **Ajouter un URI**, entrez `https://localhost:44316`, puis sélectionnez **Enregistrer**.
1. Sélectionnez **Vue d’ensemble**.
1. Enregistrez **l’ID d’application (client)** que vous utiliserez ultérieurement pour configurer l'application web.

#### <a name="applications-legacy"></a>[Applications (héritées)](#tab/applications-legacy/)

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD B2C en sélectionnant le filtre **Annuaire + abonnement** dans le menu du haut et en choisissant l’annuaire qui contient votre locataire.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Applications (héritées)** , puis l’application *webapp1*.
1. Sous **URL de réponse**, ajoutez `https://localhost:44316`.
1. Sélectionnez **Enregistrer**.
1. Sur la page des propriétés, enregistrez l'ID d'application que vous utiliserez ultérieurement pour configurer l'application web.

* * *

### <a name="create-a-client-secret"></a>Créer une clé secrète client

Ensuite, créez une clé secrète client pour l’application web inscrite. L’exemple de code d’application web l’utilise pour prouver son identité lors de la requête de jetons.

[!INCLUDE [active-directory-b2c-client-secret](../../includes/active-directory-b2c-client-secret.md)]

## <a name="configure-the-sample"></a>Configurer l'exemple

Dans ce tutoriel, vous allez configurer un exemple que vous pouvez télécharger à partir de GitHub. L’exemple utilise ASP.NET pour fournir une liste de tâches simple. L’exemple utilise des [composants middleware (intergiciel) Microsoft OWIN](/aspnet/aspnet/overview/owin-and-katana/). [Téléchargez un fichier zip ](https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi/archive/master.zip) ou clonez l’exemple à partir de GitHub. Assurez-vous que vous extrayez l’exemple de fichier dans un dossier où la longueur totale du chemin d’accès est inférieure à 260 caractères.

```
git clone https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi.git
```

L’exemple de solution contient les deux projets suivants :

* **TaskWebApp** : permettant de créer et de modifier une liste de tâches. L’exemple utilise le flux utilisateur d’**inscription ou de connexion** pour inscrire et connecter des utilisateurs.
* **TaskService** : prend en charge les fonctionnalités de création, de lecture, de mise à jour et de suppression des listes de tâches. L’API est protégée par Azure AD B2C et appelée par TaskWebApp.

Vous devez modifier l’exemple pour qu’il utilise l’application qui est inscrite dans votre locataire, ce qui inclut l’ID d’application et la clé que vous avez enregistrés précédemment. Vous devez également configurer les flux d’utilisateur que vous avez créés. L’exemple définit les valeurs de configuration en tant que paramètres dans le fichier *Web.config*.

Mettez à jour les paramètres dans le fichier Web. config pour qu’ils fonctionnent avec votre flux utilisateur :

1. Ouvrez la solution **B2C-WebAPI-DotNet** dans Visual Studio.
1. Dans le projet **TaskWebApp**, ouvrez le fichier **Web.config**.
    1. Mettez à jour la valeur de `ida:Tenant` et `ida:AadInstance` avec le nom du locataire Active AD B2C que vous avez créé. Par exemple, remplacez `fabrikamb2c` par `contoso`.
    1. Remplacez la valeur de `ida:TenantId` par l’ID de répertoire, que vous trouverez dans les propriétés de votre locataire Azure B2C (dans le portail Azure sous **Azure Active Directory** > **Propriétés** > **ID de répertoire**).
    1. Remplacez la valeur de `ida:ClientId` par l’ID d’application que vous avez enregistré.
    1. Remplacez la valeur de `ida:ClientSecret` avec la clé que vous avez enregistrée. Si la clé secrète client contient des entités XML prédéfinies, par exemple inférieur à (`<`), supérieur à (`>`), perluète (`&`) ou guillemet double (`"`), vous devez placer ces caractères dans une séquence d’échappement en encodant au format XML la clé secrète client avant de l’ajouter à votre fichier Web.config.
    1. Remplacez la valeur de `ida:SignUpSignInPolicyId` par `b2c_1_signupsignin1`.
    1. Remplacez la valeur de `ida:EditProfilePolicyId` par `b2c_1_profileediting1`.
    1. Remplacez la valeur de `ida:ResetPasswordPolicyId` par `b2c_1_passwordreset1`.

## <a name="run-the-sample"></a>Exécution de l'exemple

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **TaskWebApp**, puis cliquez sur **Définir en tant que projet de démarrage**.
1. Appuyez sur **F5**. Le navigateur par défaut se lance à l’adresse du site web local `https://localhost:44316/`.

### <a name="sign-up-using-an-email-address"></a>S’inscrire au moyen d’une adresse e-mail

1. Sélectionnez **S’inscrire/Se connecter** pour vous inscrire en tant qu’utilisateur de l’application. Le flux d’utilisateur **b2c_1_signupsignin1** est utilisé.
1. Azure AD B2C présente une page de connexion avec un lien pour l’abonnement. Si vous ne disposez pas encore d’un compte, sélectionnez **Inscrivez-vous maintenant**. Le flux de travail d’abonnement présente une page pour collecter et vérifier l’identité de l’utilisateur à l’aide d’une adresse e-mail. Le flux de travail d’inscription collecte également le mot de passe et les attributs demandés, qui sont définis dans le flux d’utilisateur.
1. Utilisez une adresse e-mail valide et validez à l’aide d’un code de vérification. Définissez un mot de passe. Entrez des valeurs pour les attributs requis.

    ![Page d’inscription affichée dans le cadre du flux de travail de connexion/inscription](./media/tutorial-web-app-dotnet/sign-up-workflow.PNG)

1. Sélectionnez **Créer** pour créer un compte local dans le locataire Azure AD B2C.

L’utilisateur de l’application peut maintenant utiliser son adresse e-mail pour se connecter à l’application web et l’utiliser.

Cependant, la fonctionnalité **Liste de tâches** ne fonctionne pas tant que vous n’avez pas terminé le tutoriel suivant de la série [Tutoriel : Utiliser Azure AD B2C pour protéger une API web ASP.NET](tutorial-web-api-dotnet.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Mettre à jour l’application dans Azure AD B2C
> * Configurer l’exemple pour utiliser l’application
> * S’inscrire à l’aide du flux utilisateur

Passez maintenant au tutoriel suivant pour activer la fonctionnalité **Liste de tâches** de l’application web. Vous y inscrirez une application d’API web de votre propre locataire Azure AD B2C et modifierez l’exemple de code de façon à utiliser votre locataire pour l’authentification de l’API.

> [!div class="nextstepaction"]
> [Tutoriel : Utiliser Azure Active Directory B2C pour protéger une API web ASP.NET >](tutorial-web-api-dotnet.md)