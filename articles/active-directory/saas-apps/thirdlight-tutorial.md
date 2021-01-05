---
title: 'Tutoriel : Intégration d’Azure Active Directory à Thirdlight | Microsoft Docs'
description: Dans ce tutoriel, vous allez découvrir comment configurer l’authentification unique entre Azure Active Directory et ThirdLight.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: jeedes
ms.openlocfilehash: f15d00f1050177c6255fb5528f03314153c3ed9a
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92514659"
---
# <a name="tutorial-azure-active-directory-integration-with-thirdlight"></a>Tutoriel : Intégration d’Azure Active Directory à Thirdlight

L’objectif de ce didacticiel est de vous apprendre à intégrer ThirdLight avec Azure Active Directory (Azure AD). Cette intégration offre les avantages suivants :

* Vous pouvez utiliser Azure AD pour contrôler qui a accès à ThirdLight.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à ThirdLight (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS à Azure AD, consultez l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à ThirdLight, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement ThirdLight pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* ThirdLight prend en charge l’authentification unique lancée par le fournisseur de services.

## <a name="add-thirdlight-from-the-gallery"></a>Ajouter ThirdLight à partir de la galerie

Pour configurer l’intégration de ThirdLight avec Azure AD, vous devez ajouter ThirdLight, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Dans le [portail Azure](https://portal.azure.com), dans le volet de gauche, sélectionnez **Azure Active Directory**  :

    ![Sélectionnez Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** > **Toutes les applications**  :

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la fenêtre :

    ![Sélectionner la nouvelle application](common/add-new-app.png)

4. Dans la zone Rechercher, entrez **ThirdLight** . Dans les résultats de la recherche, sélectionnez **ThirdLight** , puis **Ajouter** .

     ![Résultats de la recherche](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec ThirdLight pour un utilisateur de test appelé Britta Simon.
Pour activer l’authentification unique, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur correspondant dans ThirdLight.

Pour configurer et tester l’authentification unique Azure AD avec ThirdLight, procédez comme suit :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser la fonctionnalité.
2. **[Configurer l’authentification unique ThirdLight](#configure-thirdlight-single-sign-on)** côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour lui permettre d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test ThirdLight](#create-a-thirdlight-test-user)** qui est lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** pour vérifier que la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous allez activer l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec ThirdLight, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application ThirdLight, cliquez sur **Authentification unique**  :

    ![Sélectionner l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** pour activer l’authentification unique :

    ![Sélectionner une méthode d’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**  :

    ![Icône Modifier](common/edit-urls.png)

4. Dans la boîte de dialogue **Configuration SAML de base** , procédez comme suit.

    ![Boîte de dialogue Configuration SAML de base](common/sp-identifier.png)

    1. Dans la zone **URL de connexion** , entrez une URL au format suivant :
    
          `https://<subdomain>.thirdlight.com/`

    1. Dans la zone **Identificateur (ID d’entité)** , entrez une URL au format suivant :

       `https://<subdomain>.thirdlight.com/saml/sp`

       > [!NOTE]
       > Ces valeurs sont des espaces réservés. Vous devez utiliser l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe du support technique ThirdLight](https://www.thirdlight.com/support). Vous pouvez aussi vous référer aux modèles figurant dans la boîte de dialogue **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , sélectionnez le lien **Télécharger** en regard de **XML de métadonnées de fédération** , selon vos besoins, puis enregistrez le fichier sur votre ordinateur :

    ![Lien de téléchargement du certificat](common/metadataxml.png)

6. Dans la section **Configurer ThirdLight** , copiez la ou les URL appropriées, selon vos besoins :

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    1. **URL de connexion** .

    1. **Identificateur Azure AD** .

    1. **URL de déconnexion** .

### <a name="configure-thirdlight-single-sign-on"></a>Configurer l’authentification unique ThirdLight

1. Dans une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise ThirdLight en tant qu’administrateur.

1. Accédez **Configuration** > **Administration système** > **SAML2**  :

    ![Administration système](./media/thirdlight-tutorial/ic805843.png "Administration de système")

1. Dans la section de configuration de SAML2, procédez comme suit.
  
    ![Section de configuration de SAML2](./media/thirdlight-tutorial/ic805844.png "Section de configuration de SAML2")

    1. Sélectionnez **Enable SAML2 Single Sign-On** .

    1. Sous **Source for IdP Metadata** , sélectionnez **Load IdP Metadata from XML** .

    1. Ouvrez le fichier de métadonnées que vous avez téléchargé à partir du portail Azure dans la section précédente. Copiez le contenu du fichier et collez-le dans la zone **IdP Metadata XML** .

    1. Sélectionnez **Save SAML2 settings** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet de gauche du portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs**  :

    ![Sélectionner Tous les utilisateurs](common/users.png)

2. Sélectionnez **Nouvel utilisateur** en haut de la fenêtre :

    ![Sélectionner Nouvel utilisateur](common/new-user.png)

3. Dans la boîte de dialogue **Utilisateur** , procédez comme suit.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    1. Dans la zone **Nom** , entrez **BrittaSimon** .
  
    1. Dans la zone **Nom d’utilisateur** , entrez **BrittaSimon@\<yourcompanydomain>.\<extension>** . (Par exemple, BrittaSimon@contoso.com.)

    1. Sélectionnez **Afficher le mot de passe** , puis notez la valeur affichée dans la zone **Mot de passe** .

    1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à ThirdLight.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **ThirdLight** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **ThirdLight** .

    ![Liste des applications](common/all-applications.png)

3. Dans le volet gauche, sélectionnez **Utilisateurs et groupes**  :

    ![Sélectionner Utilisateurs et groupes](common/users-groups-blade.png)

4. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Sélectionnez Ajouter un utilisateur](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste des utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de la fenêtre.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste. Cliquez sur le bouton **Sélectionner** en bas de la fenêtre.

7. Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Affecter** .

### <a name="create-a-thirdlight-test-user"></a>Créer un utilisateur de test ThirdLight

Pour autoriser les utilisateurs Azure AD à se connecter à ThirdLight, vous devez les ajouter dans ThirdLight. Vous devez les ajouter manuellement.

Pour créer un compte d’utilisateur, procédez comme suit :

1. Connectez-vous à votre site d’entreprise ThirdLight en tant qu’administrateur.

1. Accédez à l’onglet **Users** .

1. Sélectionnez **Users and Groups** .

1. Sélectionnez **Add new User** .

1. Entrez le nom d’utilisateur, un nom ou une description ainsi que l’adresse électronique d’un compte Azure AD valide que vous souhaitez approvisionner. Choisissez un paramètre prédéfini ou un groupe de nouveaux membres.

1. Sélectionnez **Create** (Créer).

> [!NOTE]
> Vous pouvez utiliser n’importe quel API ou outil de création de compte d’utilisateur fourni par ThirdLight pour provisionner des comptes d’utilisateurs Azure AD.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Vous devez maintenant tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de sélectionner la vignette ThirdLight dans le panneau d’accès doit vous connecter automatiquement à l’instance ThirdLight pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le volet d’accès, consultez [Accéder aux applications du portail Mes applications et les utiliser](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)