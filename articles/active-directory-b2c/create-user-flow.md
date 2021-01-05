---
title: Créer un flux utilisateur - Azure Active Directory B2C
description: Découvrez comment créer des flux utilisateur sur le portail Azure pour permettre l’inscription, la connexion et la modification de profil utilisateur pour vos applications dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 07/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 418446e0d465b606b8d580297cebd73c466d4841
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97109010"
---
# <a name="create-a-user-flow-in-azure-active-directory-b2c"></a>Créer un flux utilisateur dans Azure Active Directory B2C

Vous pouvez créer des [flux utilisateur](user-flow-overview.md) de types différents dans votre locataire Azure Active Directory B2C (Azure AD B2C) et les utiliser dans vos applications selon les besoins. Les flux d’utilisateurs peuvent être réutilisés entre les différentes applications.

> [!IMPORTANT]
> Nous avons modifié la manière dont nous référençons les versions de flux utilisateur. Auparavant, nous proposions des versions V1 (prêtes pour la production) et des versions V1.1 et V2 (préversions). À présent, nous avons consolidé les flux utilisateur dans les versions **Recommandé** (préversion de nouvelle génération) et **Standard** (en disponibilité générale). Tous les flux utilisateur hérités V1.1 et V2 seront déconseillés d’ici le **1er août 2021**. Pour plus d’informations, consultez [Versions de flux utilisateur dans Azure AD B2C](user-flow-versions.md).

## <a name="before-you-begin"></a>Avant de commencer

- **Inscrivez l’application** que vous souhaitez utiliser pour tester le nouveau flux utilisateur. Pour obtenir un exemple, consultez le [Tutoriel : Inscrire une application web dans Azure AD B2C](tutorial-register-applications.md).
- **Ajoutez des fournisseurs d’identité externes** si vous souhaitez permettre aux utilisateurs de se connecter avec les fournisseurs que sont Azure AD, Amazon, Facebook, GitHub, LinkedIn, Microsoft ou Twitter. Pour obtenir un exemple, consultez [Tutoriel : Ajouter des fournisseurs d’identité à vos applications dans Azure AD B2C](tutorial-add-identity-providers.md).
- **Configurez le fournisseur d’identité de compte local** pour spécifier les types d’identité (e-mail, nom d’utilisateur, numéro de téléphone) que vous souhaitez prendre en charge pour les comptes locaux dans votre locataire. Vous pouvez ensuite choisir parmi ces types d’identité pris en charge quand vous créez des flux utilisateur individuels. Quand un utilisateur suit le flux utilisateur, un compte local est créé dans votre annuaire Azure AD B2C, puis votre fournisseur d’identité **Compte local** authentifie les informations de l’utilisateur. Configurez le fournisseur d’identité de compte local de votre locataire en procédant comme suit :

   1. Connectez-vous au [portail Azure](https://portal.azure.com/). 
   2. Sélectionnez le filtre **Annuaire + abonnement** dans le menu du haut, puis choisissez l’annuaire qui contient votre locataire Azure AD B2C.
   3. Dans la barre de recherche en haut du portail Azure, recherchez et sélectionnez **Azure AD B2C**.
   4. Sous **Gérer**, sélectionnez **Fournisseurs d’identité**.
   5. Dans la liste des fournisseurs d’identité, sélectionnez **Compte local**.
   6. Dans la page **Configurer l’IDP local**, sélectionnez tous les types d’identité que vous souhaitez prendre en charge. Les options que vous sélectionner ici seront disponibles pour les flux utilisateur que vous créerez ultérieurement :
      - **Téléphone** (préversion) : permet à un utilisateur d’entrer un numéro de téléphone, qui est vérifié au moment de l’inscription et devient son identifiant utilisateur.
      - **E-mail** (par défaut) : permet à un utilisateur d’entrer une adresse e-mail, qui est vérifiée au moment de l’inscription et devient son identifiant utilisateur.
      - **Nom d’utilisateur** : permet à un utilisateur de créer son propre identifiant utilisateur unique. Une adresse e-mail est collectée auprès de l’utilisateur et vérifiée.
    7. Sélectionnez **Enregistrer**.

## <a name="create-a-user-flow"></a>Créer un flux utilisateur

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez le répertoire qui contient votre locataire Azure AD B2C.

    ![Volet client B2C, répertoire et abonnement, Portail Azure](./media/create-user-flow/directory-subscription-pane.png)

3. Dans le portail Azure, recherchez et sélectionnez **Azure AD B2C**.
4. Sous **Stratégies**, sélectionnez **Flux d’utilisateurs**, puis sélectionnez **Nouveau flux d’utilisateur**.

    ![Page Flux d'utilisateur du portail avec bouton Nouveau flux d'utilisateur en surbrillance](./media/create-user-flow/signup-signin-user-flow.png)

5. Dans la page **Créer un flux d’utilisateur**, sélectionnez le type de flux utilisateur que vous souhaitez créer (voir [Flux d’utilisateur dans Azure Active Directory B2C](user-flow-overview.md) pour obtenir une vue d’ensemble).

    ![Page Créer un flux d’utilisateur avec Inscription et connexion mis en évidence](./media/create-user-flow/select-user-flow-type.png)

6. Sous **Sélectionner une version**, sélectionnez **Recommandé**, puis **Créer**. ([Apprenez-en davantage](user-flow-versions.md) sur les version de flux utilisateur.)

    ![Page Créer un flux d'utilisateur du Portail Azure avec propriétés en surbrillance](./media/create-user-flow/select-version.png)

7. Entrez un **Nom** pour le flux utilisateur (par exemple, *signupsignin1*, *profileediting1*, *passwordreset1*).
8. Sous **Fournisseurs d’identité**, choisissez les options en fonction du type de flux utilisateur que vous êtes en train de créer :

   - **Compte local**. Si vous souhaitez autoriser les utilisateurs à créer des comptes locaux dans votre locataire Azure AD B2C, sélectionnez le type d’identificateur qu’ils doivent utiliser (par exemple, e-mail, identifiant utilisateur ou téléphone). Seuls les types d’identité configurés dans les paramètres de votre [fournisseur d’identité de compte local](#before-you-begin) sont listés.

   - **Fournisseurs d’identité sociale**. Si vous souhaitez autoriser les utilisateurs à se connecter avec les [fournisseurs d’identité sociale que vous avez ajoutés](tutorial-add-identity-providers.md), comme Azure AD, Amazon, Facebook, GitHub, LinkedIn, Microsoft ou Twitter, sélectionnez les fournisseurs dans la liste.

9. Pour **Attributs utilisateur et revendications**, choisissez les revendications et les attributs à collecter et envoyer par l’utilisateur pendant l’inscription. Sélectionnez **Afficher plus**. Sélectionnez les attributs et les revendications, puis sélectionnez **OK**.

    ![Page de sélection des attributs et revendications avec trois revendications sélectionnées](./media/create-user-flow/signup-signin-attributes.png)

10. Sélectionnez **Créer** pour ajouter le flux utilisateur. Un préfixe *B2C_1* est automatiquement ajouté au nom.

### <a name="test-the-user-flow"></a>Tester le flux utilisateur

1. Sélectionnez **Stratégies** > **Flux utilisateur**, puis sélectionnez le flux utilisateur que vous avez créé. Dans la page de vue d’ensemble du flux utilisateur, sélectionnez **Exécuter le flux d’utilisateur**.
1. Pour **Application**, sélectionnez l’application web que vous avez inscrite à l’étape 1. L’**URL de réponse** doit être `https://jwt.ms`.
1. Sélectionnez **Exécuter le flux utilisateur**.
2. Selon le type de flux utilisateur que vous testez, inscrivez-vous à l’aide d’une adresse e-mail valide et suivez le processus d’inscription, ou connectez-vous à l’aide d’un compte que vous avez créé.

    ![Page Exécuter le flux d'utilisateur du portail avec bouton Exécuter le flux d'utilisateur en surbrillance](./media/create-user-flow/sign-up-sign-in-run-now.png)

1. Suivez les invites du flux utilisateur. Au terme du flux utilisateur, le jeton est retourné à `https://jwt.ms`, où vous devez le voir.

> [!NOTE]
> L’expérience « Exécuter le flux utilisateur » n’est actuellement pas compatible avec le type d’URL de réponse SPA utilisant le flux de code d’autorisation. Pour utiliser l’expérience « Exécuter le flux d’utilisateur » avec ces types d’applications, enregistrez une URL de réponse de type « Web » et activez le flux implicite, comme décrit [ici](tutorial-register-spa.md).

## <a name="next-steps"></a>Étapes suivantes

- [Ajouter l’accès conditionnel à des flux utilisateur Azure AD B2C](conditional-access-user-flow.md)
- [Personnaliser l’interface utilisateur dans un flux d’utilisateur Azure AD B2C](customize-ui-with-html.md)
