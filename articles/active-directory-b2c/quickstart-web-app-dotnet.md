---
title: 'Démarrage rapide : Configurer la connexion pour une application web ASP.NET'
titleSuffix: Azure AD B2C
description: Dans ce guide de démarrage rapide, vous allez exécuter un exemple d’application web ASP.NET qui utilise Azure Active Directory B2C pour la connexion aux comptes.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.topic: quickstart
ms.custom: devx-track-csharp, mvc
ms.date: 09/12/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 379ea9a0c2a49e92fa1c4882ea6e783b3b91e32f
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88163915"
---
# <a name="quickstart-set-up-sign-in-for-an-aspnet-application-using-azure-active-directory-b2c"></a>Démarrage rapide : Configurer la connexion pour une application ASP.NET à l’aide d’Azure Active Directory B2C

Azure Active Directory B2C (Azure AD B2C) assure la gestion des identités de cloud pour préserver la protection de votre application, de votre entreprise et de vos clients. Azure AD B2C permet à vos applications de s’authentifier auprès de comptes des réseaux sociaux et de comptes d’entreprise à l’aide de protocoles standard ouverts. Dans ce démarrage rapide, vous utilisez un exemple d’application ASP.NET pour vous connecter à l’aide d’un fournisseur d’identité sociale et appeler une API web protégée par Azure AD B2C.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

- [Visual Studio 2019](https://www.visualstudio.com/downloads/) avec la charge de travail **Développement ASP.NET et web**.
- Un compte de réseau social Facebook, Google ou Microsoft.
- [Téléchargez un fichier zip](https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi/archive/master.zip) ou clonez l’exemple d’application web à partir de GitHub.

    ```
    git clone https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi.git
    ```

    L’exemple de solution comprend deux projets :

    - **TaskWebApp** : application web qui crée et modifie une liste de tâches. L’application web utilise le flux d’**inscription ou de connexion** d’utilisateur pour inscrire ou connecter des utilisateurs.
    - **TaskService** : API web qui prend en charge les fonctionnalités de création, de lecture, de mise à jour et de suppression des listes de tâches. L’API web est protégée par Azure AD B2C et appelée par l’application web.

## <a name="run-the-application-in-visual-studio"></a>Exécuter l’application dans Visual Studio

1. Dans l’exemple de dossier de projet de l’application, ouvrez la solution **B2C-WebAPI-DotNet.sln** dans Visual Studio.
2. Pour ce guide de démarrage rapide, vous exécutez les projets **TaskWebApp** et **TaskService** simultanément. Cliquez sur la solution **B2C-WebAPI-DotNet** avec le bouton droit de la souris dans l’Explorateur de solutions, puis sélectionnez **Définir les projets de démarrage**.
3. Sélectionnez **Plusieurs projets de démarrage** et remplacez l’**action** pour les deux projets par **Démarrer**.
4. Cliquez sur **OK**.
5. Appuyez sur la touche **F5** pour déboguer les deux applications. Chaque application s’ouvre dans son propre onglet de navigateur :

    - `https://localhost:44316/` : application web ASP.NET. Dans ce démarrage rapide, vous allez interagir directement avec cette application.
    - `https://localhost:44332/` : API web appelée par l’application web ASP.NET.

## <a name="sign-in-using-your-account"></a>Connectez-vous à l’aide de votre compte

1. Cliquez sur le lien **S’inscrire/Se connecter** dans l’application web ASP.NET pour démarrer le workflow.

    ![Exemple d’application web ASP.NET dans un navigateur avec le lien s’inscrire/se connecter en surbrillance](./media/quickstart-web-app-dotnet/web-app-sign-in.png)

    L’exemple prend en charge plusieurs options d’inscription, notamment l’utilisation d’un fournisseur d’identité sociale ou la création d’un compte local à l’aide d’une adresse e-mail. Pour ce guide de démarrage rapide, utilisez un compte de fournisseur d’identité sociale provenant de Facebook, Google ou Microsoft.

2. Azure AD B2C présente une page de connexion d’une entreprise fictive appelée Fabrikam pour l’exemple d’application web. Pour vous inscrire au moyen d’un fournisseur d’identité sociale, cliquez sur le bouton en regard du fournisseur d’identité que vous souhaitez utiliser.

    ![Page de connexion ou d’inscription affichant des boutons de fournisseur d’identité](./media/quickstart-web-app-dotnet/sign-in-or-sign-up-web.png)

    Vous vous authentifiez (vous connectez) avec les informations d’identification de votre compte de réseau social et autorisez l’application à lire les informations de votre compte de réseau social. En accordant l’accès, l’application peut extraire des informations de profil du compte de réseau social, telles que votre nom et votre ville.

3. Terminez le processus de connexion pour le fournisseur d’identité.

## <a name="edit-your-profile"></a>Modifier votre profil

Azure Active Directory B2C fournit des fonctionnalités permettant aux utilisateurs de mettre à jour leurs profils. L’exemple d’application web utilise un flux utilisateur de modification du profil Azure AD B2C pour le workflow.

1. Dans la barre de menus de l’application, cliquez sur le nom de votre profil et sélectionnez **Modifier le profil** pour modifier le profil que vous avez créé.

    ![Exemple d’application web dans un navigateur avec le lien Modifier le profil en surbrillance](./media/quickstart-web-app-dotnet/edit-profile-web.png)

2. Modifiez votre **nom d'affichage** ou **ville**, puis cliquez sur **Continuer** pour mettre à jour votre profil.

    Le nouveau nom d’affichage s’affiche dans la partie supérieure droite de la page d’accueil de l’application web.

## <a name="access-a-protected-api-resource"></a>Accéder à une ressource d’API protégée

1. Cliquez sur **Liste de tâches** pour entrer et modifier des éléments de votre liste des tâches.

2. Entrez du texte dans la zone de texte **Nouvel élément**. Cliquez sur **Ajouter** pour appeler l’API web protégée par Azure AD B2C qui ajoute un élément de liste des tâches.

    ![Exemple d’application web dans un navigateur affichant l’option Ajouter un élément de la liste des tâches](./media/quickstart-web-app-dotnet/add-todo-item-web.png)

    L’application web ASP.NET inclut un jeton d’accès Azure AD dans la requête envoyée à la ressource de l’API web protégée pour effectuer des opérations sur les éléments de la liste des tâches de l’utilisateur.

Vous avez utilisé votre compte d’utilisateur Azure AD B2C pour effectuer un appel autorisé à une API web protégée par Azure AD B2C.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez utiliser votre locataire Azure AD B2C si vous envisagez d’effectuer d’autres didacticiels ou démarrages rapides Azure AD B2C. Si vous n’en avez plus besoin, vous pouvez [supprimer votre client Azure AD B2C](faq.md#how-do-i-delete-my-azure-ad-b2c-tenant).

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez utilisé un exemple d’application ASP.NET pour :

* Vous connecter avec une page de connexion personnalisée
* Vous connecter avec un fournisseur d’identité sociale
* Créer un compte Azure AD B2C
* Appeler une API web protégée par Azure AD B2C

Passez maintenant à la création de votre propre locataire Azure AD B2C.

> [!div class="nextstepaction"]
> [Créer un locataire Azure Active Directory B2C dans le portail Azure](tutorial-create-tenant.md)
