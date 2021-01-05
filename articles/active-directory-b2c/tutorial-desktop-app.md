---
title: 'Tutoriel : Authentifier les utilisateurs dans une application cliente native'
titleSuffix: Azure AD B2C
description: Tutoriel sur l’utilisation d’Azure Active Directory B2C pour fournir une connexion utilisateur à une application de bureau .NET.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 10/12/2019
ms.custom: mvc
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: 06d27c3a3daa4702653a2063d0ac70fd094e2d74
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "78186197"
---
# <a name="tutorial-authenticate-users-in-a-native-desktop-client-using-azure-active-directory-b2c"></a>Tutoriel : Authentifier les utilisateurs dans un client de bureau natif à l’aide d’Azure Active Directory B2C

Ce didacticiel vous montre comment utiliser Azure Active Directory B2C (Azure AD B2C) pour connecter et inscrire des utilisateurs dans une application de bureau Windows Presentation Foundation (WPF). Azure AD B2C permet à vos applications de s’authentifier auprès de comptes de réseaux sociaux, de comptes d’entreprise et de comptes Azure Active Directory à l’aide de protocoles standards ouverts.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Ajouter l’application cliente native
> * Configurer l’exemple pour utiliser l’application
> * S’inscrire à l’aide du flux utilisateur

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

- [Créer des flux d’utilisateur](tutorial-create-user-flows.md) pour activer les expériences utilisateur dans votre application
- Installez [Visual Studio 2019](https://www.visualstudio.com/downloads/) avec les charges de travail **Développement .NET Desktop** et **Développement web et ASP.NET**.

## <a name="add-the-native-client-application"></a>Ajouter l’application cliente native

[!INCLUDE [active-directory-b2c-appreg-native](../../includes/active-directory-b2c-appreg-native.md)]

Enregistrez l’**ID d’application (client)** pour l’utiliser dans une étape ultérieure.

## <a name="configure-the-sample"></a>Configurer l'exemple

Dans ce tutoriel, vous allez configurer un exemple que vous pouvez télécharger à partir de GitHub. L’exemple d’application de bureau WPF illustre l’inscription et la connexion, et peut appeler une API web protégée dans Azure AD B2C. [Téléchargez un fichier zip ](https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop/archive/msalv3.zip), [parcourez le référentiel](https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop) ou clonez l’exemple à partir de GitHub.

```
git clone https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop.git
```

Pour mettre à jour l’application afin qu’elle fonctionne avec votre locataire Azure AD B2C et appeler ses flux utilisateur au lieu de ceux du locataire de démonstration par défaut :

1. Ouvrez la solution **active-directory-b2c-wpf** (`active-directory-b2c-wpf.sln`) dans Visual Studio.
2. Dans le projet **active-directory-b2c-wpf**, ouvrez le fichier *App.xaml.cs* et recherchez les définitions de variables suivantes. Remplacez `{your-tenant-name}` par le nom de votre locataire Azure AD B2C et `{application-ID}` par l’ID d’application que vous avez enregistré précédemment.

    ```csharp
    private static readonly string Tenant = "{your-tenant-name}.onmicrosoft.com";
    private static readonly string AzureAdB2CHostname = "{your-tenant-name}.b2clogin.com";
    private static readonly string ClientId = "{application-ID}";
    ```

3. Mettez à jour les variables de nom de stratégie avec les noms des flux d’utilisateurs que vous avez créés dans le cadre des conditions préalables. Par exemple :

    ```csharp
    public static string PolicySignUpSignIn = "B2C_1_signupsignin1";
    public static string PolicyEditProfile = "B2C_1_profileediting1";
    public static string PolicyResetPassword = "B2C_1_passwordreset1";
    ```

## <a name="run-the-sample"></a>Exécution de l'exemple

Appuyez sur **F5** pour générer et exécuter l’exemple.

### <a name="sign-up-using-an-email-address"></a>S’inscrire au moyen d’une adresse e-mail

1. Sélectionnez **Se connecter** pour vous inscrire en tant qu’utilisateur. Le flux d’utilisateur **B2C_1_signupsignin1** est utilisé.
2. Azure AD B2C présente une page de connexion avec un lien **S'inscrire maintenant**. Étant donné que vous n’avez pas encore de compte, sélectionnez le lien **Inscrivez-vous maintenant**.
3. Le flux de travail d’abonnement présente une page pour collecter et vérifier l’identité de l’utilisateur à l’aide d’une adresse e-mail. Le flux de travail d’inscription collecte également le mot de passe et les attributs demandés, qui sont définis dans le flux d’utilisateur.

    Utilisez une adresse e-mail valide et validez à l’aide d’un code de vérification. Définissez un mot de passe. Entrez des valeurs pour les attributs requis.

    ![Page d’inscription affichée dans le cadre du flux de travail de connexion/inscription](./media/tutorial-desktop-app/azure-ad-b2c-sign-up-workflow.png)

4. Sélectionnez **Créer** pour créer un compte local dans le locataire Azure AD B2C.

L’utilisateur peut à présent utiliser son adresse e-mail pour se connecter et utiliser l’application de bureau. Après une inscription ou une connexion réussie, les détails du jeton s’affichent dans le volet inférieur de l’application WPF.

![Détails du jeton affichés dans le volet inférieur de l’application de bureau WPF](./media/tutorial-desktop-app/desktop-app-01-post-signin.png)

Si vous sélectionnez le bouton **Appel d’API**, un **message d’erreur** s’affiche. Vous rencontrez l’erreur car, dans son état actuel, l’application tente d’accéder à une API protégée par le locataire de démonstration, `fabrikamb2c.onmicrosoft.com`. Étant donné que votre jeton d’accès n’est valide que pour votre locataire Azure AD B2C, l’appel d’API n’est pas autorisé.

Passez au didacticiel suivant pour inscrire une API web protégée dans votre propre locataire et activer les fonctionnalités **Appel API**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Ajouter l’application cliente native
> * Configurer l’exemple pour utiliser l’application
> * S’inscrire à l’aide du flux utilisateur

Ensuite, pour activer la fonctionnalité du bouton **Appel d’API**, accordez à l’application de bureau WPF l’accès à une API web inscrite dans votre propre locataire Azure AD B2C :

> [!div class="nextstepaction"]
> [Tutoriel : Accorder l’accès à une API web Node.js à partir d’une application de bureau >](tutorial-desktop-app-webapi.md)
