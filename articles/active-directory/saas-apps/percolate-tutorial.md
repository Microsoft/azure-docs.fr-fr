---
title: 'Tutoriel : Intégration d’Azure Active Directory à Percolate | Microsoft Docs'
description: Dans ce tutoriel, vous allez découvrir comment configurer l’authentification unique entre Azure Active Directory et Percolate.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/01/2019
ms.author: jeedes
ms.openlocfilehash: 913ffc0670e40e749bd28382d492a16891fdc5c9
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92522257"
---
# <a name="tutorial-azure-active-directory-integration-with-percolate"></a>Tutoriel : Intégration d’Azure Active Directory à Percolate

Dans ce didacticiel, vous allez apprendre à intégrer Percolate à Azure Active Directory (Azure AD).

Cette intégration offre les avantages suivants :

* Vous pouvez utiliser Azure AD pour contrôler qui a accès à Percolate.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Percolate (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS à Azure AD, consultez l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Percolate, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Percolate pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Percolate prend en charge l’authentification unique lancée par le fournisseur de services et le fournisseur d’identité.

## <a name="add-percolate-from-the-gallery"></a>Ajouter Percolate à partir de la galerie

Pour configurer l’intégration de Percolate dans Azure AD, vous devez ajouter Percolate à partir de la galerie à votre liste d’applications SaaS gérées.

1. Dans le [portail Azure](https://portal.azure.com), dans le volet de gauche, sélectionnez **Azure Active Directory**  :

    ![Sélectionnez Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** > **Toutes les applications**  :

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la fenêtre :

    ![Sélectionner la nouvelle application](common/add-new-app.png)

4. Entrez **Percolate** dans la zone de recherche. Dans les résultats de la recherche, sélectionnez **Percolate** , puis **Ajouter** .

     ![Résultats de la recherche](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Percolate pour un utilisateur de test appelé Britta Simon.
Pour activer l’authentification unique, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur correspondant dans Percolate.

Pour configurer et tester l’authentification unique Azure AD avec Percolate, procédez comme suit :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser la fonctionnalité.
2. **[Configurer l’authentification unique Percolate](#configure-percolate-single-sign-on)** côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour lui permettre d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Percolate](#create-a-percolate-test-user)** qui est lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** pour vérifier que la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous allez activer l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Percolate, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), sur la page d’intégration de l’application **Percolate** , sélectionnez **Authentification unique**  :

    ![Sélectionner l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** pour activer l’authentification unique :

    ![Sélectionner une méthode d’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**  :

    ![Icône Modifier](common/edit-urls.png)

4. Dans la boîte de dialogue **Configuration SAML de base** , vous n'avez rien à faire pour configurer l’application en mode lancé par le fournisseur d’identité. L’application est déjà intégrée à Azure.

    ![Informations d’authentification unique dans Domaine et URL Percolate](common/preintegrated.png)

5. Pour configurer l’application en mode lancée par le fournisseur de services, sélectionnez **Définir des URL supplémentaires** puis, dans la zone **URL de connexion** , entrez **https://percolate.com/app/login** :

   ![Capture d’écran montrant la sélection de « Définir des URL supplémentaires » avec la zone de texte « URL de connexion » mise en évidence.](common/metadata-upload-additional-signon.png)
6. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , sélectionnez l’icône **Copier** pour copier l’ **URL des métadonnées de fédération d’application** . Enregistrez cette URL.

    ![Copier l'URL des métadonnées de fédération d'application](common/copy-metadataurl.png)

7. Dans la section **Configurer Percolate** , copiez la ou les URL appropriées, selon vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    1. **URL de connexion** .

    1. **Identificateur Azure AD** .

    1. **URL de déconnexion** .

### <a name="configure-percolate-single-sign-on"></a>Configurer l’authentification unique Percolate

1. Dans une nouvelle fenêtre de navigateur web, connectez-vous à Percolate en tant qu’administrateur.

2. À gauche de la page d’accueil, sélectionnez **Paramètres**  :
    
    ![Sélection de Paramètres](./media/percolate-tutorial/configure01.png)

3. Dans le volet gauche, sélectionnez **SSO** sous **Organisation**  :

    ![Sélectionner SSO sous Organisation](./media/percolate-tutorial/configure02.png)

    1. Dans la zone **URL de connexion** , collez la valeur **URL de connexion** que vous avez copiée à partir du portail Azure.

    1. Dans la zone **ID d'entité** , collez la valeur **Identificateur Azure AD** que vous avez copiée à partir du portail Azure.

    1. Dans le Bloc-notes, ouvrez le certificat encodé en base 64 que vous avez téléchargé à partir du portail Azure. Copiez son contenu et collez-le dans la zone **Certificats x509** .

    1. Dans la zone **Attribut d'e-mail** , entrez **emailaddress** .

    1. La zone **URL des métadonnées de fournisseur d'identité** correspond à un champ facultatif. Si vous avez copié une **URL des métadonnées de fédération d'application** à partir du portail Azure, vous pouvez la coller dans cette zone.

    1. Dans la liste **Should AuthNRequests be signed?** , sélectionnez **Non** .

    1. Dans la liste **Enable SSO auto-provisioning** , sélectionnez **Non** .

    1. Sélectionnez **Enregistrer** .

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure AD en lui accordant l’accès à Percolate.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Percolate** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Percolate** .

    ![Liste des applications](common/all-applications.png)

3. Dans le volet gauche, sélectionnez **Utilisateurs et groupes**  :

    ![Capture d’écran montrant la sélection de « Utilisateurs et groupes » dans le volet gauche.](common/users-groups-blade.png)

4. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Sélectionner Utilisateurs et groupes](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste des utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste. Cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Affecter** .

### <a name="create-a-percolate-test-user"></a>Créer un utilisateur de test Percolate

Pour autoriser les utilisateurs Azure AD à se connecter à Percolate, vous devez les ajouter dans Percolate. Vous devez les ajouter manuellement.

Pour créer un compte d’utilisateur, procédez comme suit :

1. Connectez-vous à Percolate en tant qu’administrateur.

2. Dans le volet gauche, sélectionnez **Utilisateurs** sous **Organisation** . Sélectionnez **Nouveaux utilisateurs**  :

    ![Sélectionner Nouveaux utilisateurs](./media/percolate-tutorial/configure03.png)

3. Dans la page **Créer des utilisateurs** , procédez comme suit.

    ![Page Créer des utilisateurs](./media/percolate-tutorial/configure04.png)

    1. Dans la zone **E-mail** , entrez l’adresse e-mail de l’utilisateur. Par exemple : brittasimon@contoso.com.

    1. Dans la zone **Nom complet** , entrez le nom de l’utilisateur. Par exemple, **Brittasimon** .

    1. Sélectionnez **Créer des utilisateurs** .

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Vous devez maintenant tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous sélectionnez la vignette Percolate dans le volet d’accès, vous devez être connecté automatiquement à l’instance Percolate pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations, consultez [Accéder aux applications du portail Mes applications et les utiliser](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)