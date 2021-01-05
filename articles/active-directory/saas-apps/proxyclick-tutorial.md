---
title: 'Didacticiel : Intégration d’Azure Active Directory à Proxyclick | Microsoft Docs'
description: Dans ce tutoriel, vous allez découvrir comment configurer l’authentification unique entre Azure Active Directory et Proxyclick.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: 23ae1a2c1371cda9435ea76f02cebc79c141c904
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92522238"
---
# <a name="tutorial-azure-active-directory-integration-with-proxyclick"></a>Didacticiel : Intégration d’Azure Active Directory à Proxyclick

Ce didacticiel vous montrera comment intégrer Proxyclick à Azure Active Directory (Azure AD).
Cette intégration offre les avantages suivants :

* Vous pouvez utiliser Azure AD pour contrôler qui a accès à Proxyclick.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Proxyclick (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS à Azure AD, consultez l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Proxyclick, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [essai gratuit d’un mois](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Proxyclick pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Proxyclick prend en charge l’authentification unique lancée par le fournisseur de services et le fournisseur d’identité.

## <a name="add-proxyclick-from-the-gallery"></a>Ajouter Proxyclick à partir de la galerie

Pour configurer l’intégration de Proxyclick à Azure AD, vous devez ajouter Proxyclick, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Dans le [portail Azure](https://portal.azure.com), dans le volet de gauche, sélectionnez **Azure Active Directory**  :

    ![Sélectionnez Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** > **Toutes les applications**  :

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la fenêtre :

    ![Sélectionner la nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Proxyclick** . Dans les résultats de la recherche, sélectionnez **Proxyclick** , puis **Ajouter** .

     ![Résultats de la recherche](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Proxyclick pour un utilisateur de test appelé Britta Simon.
Pour activer l’authentification unique, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur correspondant dans Proxyclick.

Pour configurer et tester l’authentification unique Azure AD avec Proxyclick, procédez comme suit :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser la fonctionnalité.
2. **[Configurer l’authentification unique Proxyclick](#configure-proxyclick-single-sign-on)** côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour lui permettre d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Proxyclick](#create-a-proxyclick-test-user)** qui est lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** pour vérifier que la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous allez activer l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Proxyclick, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application Proxyclick, sélectionnez **Authentification unique**  :

    ![Sélectionner l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** pour activer l’authentification unique :

    ![Sélectionner une méthode d’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**  :

    ![Icône Modifier](common/edit-urls.png)

4. Dans la boîte de dialogue **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le fournisseur d’identité, procédez comme suit.

    ![Boîte de dialogue Configuration SAML de base](common/idp-intiated.png)

    1. Dans la zone **Identificateur** , entrez une URL au format suivant :
   
       `https://saml.proxyclick.com/init/<companyId>`

    1. Dans la zone **URL de réponse** , entrez une URL au format suivant :

       `https://saml.proxyclick.com/consume/<companyId>`

5. Si vous voulez configurer l’application en mode lancé par le fournisseur de services, sélectionnez **Définir des URL supplémentaires** . Dans la zone **URL de connexion** , entrez une URL au format suivant :
   
   `https://saml.proxyclick.com/init/<companyId>`

    ![Informations d’authentification unique dans Domaine et URL Proxyclick](common/metadata-upload-additional-signon.png)

    

    > [!NOTE]
    > Ces valeurs sont des espaces réservés. Vous devez utiliser l’identificateur, l’URL de réponse et l’URL de connexion exacts. Vous trouverez plus loin dans ce tutoriel les étapes permettant d’obtenir ces valeurs.

6. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , sélectionnez le lien **Télécharger** en regard du **Certificat (Base64)** correspondant à vos besoins, puis enregistrez le certificat sur votre ordinateur :

    ![Lien de téléchargement du certificat](common/certificatebase64.png)

7. Dans la section **Configurer Proxyclick** , copiez la ou les URL appropriées, selon vos besoins :

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    1. **URL de connexion** .

    1. **Identificateur Azure AD** .

    1. **URL de déconnexion** .

### <a name="configure-proxyclick-single-sign-on"></a>Configurer l’authentification unique Proxyclick

1. Dans une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Proxyclick en tant qu’administrateur.

2. Sélectionnez **Compte et paramètres**  :

    ![Sélectionner Compte et paramètres](./media/proxyclick-tutorial/configure1.png)

3. Faites défiler jusqu'à la section **Intégrations** , puis sélectionnez **SAML**  :

    ![Sélectionner SAML](./media/proxyclick-tutorial/configure2.png)

4. Dans la section **SAML** , procédez comme suit.

    ![Section SAML](./media/proxyclick-tutorial/configure3.png)

    1. Copiez la valeur de l’ **URL de consommateur SAML** , puis collez-la dans la zone **URL de réponse** de la boîte de dialogue **Configuration SAML de base** du portail Azure.

    1. Copiez la valeur de l’ **URL de redirection d’authentification unique SAML** , puis collez-la dans les zones **URL de connexion** et **Identificateur** de la boîte de dialogue **Configuration SAML de base** du portail Azure.

    1. Dans la liste **Méthode de requête SAML** , sélectionnez **Redirection HTTP** .

    1. Dans la zone **Émetteur** , collez la valeur **Identificateur Azure AD** copiée à partir du portail Azure.

    1. Dans la zone **URL de point de terminaison SAML 2.0** , collez la valeur de l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    1. Dans le bloc-notes, ouvrez le fichier de certificat que vous avez téléchargé à partir du portail Azure. Collez le contenu de ce fichier dans la zone **Certificat** .

    1. Sélectionnez **Enregistrer les modifications** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet de gauche du portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs**  :

    ![Sélectionner Tous les utilisateurs](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran :

    ![Sélectionner Nouvel utilisateur](common/new-user.png)

3. Dans la boîte de dialogue **Utilisateur** , procédez comme suit.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    1. Dans la zone **Nom** , entrez **BrittaSimon** .
  
    1. Dans la zone **Nom d’utilisateur** , entrez **BrittaSimon@\<yourcompanydomain>.\<extension>** . (Par exemple, BrittaSimon@contoso.com.)

    1. Sélectionnez **Afficher le mot de passe** , puis notez la valeur affichée dans la zone **Mot de passe** .

    1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Proxyclick.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Proxyclick** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Proxyclick** .

    ![Liste des applications](common/all-applications.png)

3. Dans le volet gauche, sélectionnez **Utilisateurs et groupes**  :

    ![Sélectionner Utilisateurs et groupes](common/users-groups-blade.png)

4. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Sélectionnez Ajouter un utilisateur](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste des utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de la fenêtre.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste. Cliquez sur le bouton **Sélectionner** en bas de la fenêtre.

7. Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Affecter** .

### <a name="create-a-proxyclick-test-user"></a>Créer un utilisateur de test Proxyclick

Pour autoriser les utilisateurs Azure AD à se connecter à Proxyclick, vous devez les ajouter dans Proxyclick. Vous devez les ajouter manuellement.

Pour créer un compte d’utilisateur, procédez comme suit :

1. Connectez-vous à votre site d’entreprise Proxyclick en tant qu’administrateur.

1. Sélectionnez **Collègues** en haut de la fenêtre :

    ![Sélectionner Collègues](./media/proxyclick-tutorial/user1.png)

1. Sélectionnez **Ajouter un collègue** :

    ![Sélectionner Ajouter un collègue](./media/proxyclick-tutorial/user2.png)

1. Dans la section **Ajouter un collègue** , procédez comme suit.

    ![Section Ajouter un collègue](./media/proxyclick-tutorial/user3.png)

    1. Dans la zone **E-mail** , entrez l’adresse e-mail de l’utilisateur. Dans ce cas, **brittasimon\@contoso.com** .

    1. Dans la boîte **Prénom** , entrez le prénom de l’utilisateur. Dans ce cas, **Britta** .

    1. Dans la boîte **Nom** , entrez le nom de l’utilisateur. Dans ce cas, **Simon** .

    1. Sélectionnez **Add User** (Ajouter un utilisateur).

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Vous devez maintenant tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous sélectionnez la vignette Proxyclick dans le volet d’accès, vous devez être connecté automatiquement à l’instance Proxyclick pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le volet d’accès, consultez [Accéder aux applications du portail Mes applications et les utiliser](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)