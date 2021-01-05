---
title: 'Didacticiel : Intégration d’Azure Active Directory avec LockPath Keylight | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et LockPath Keylight.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/14/2019
ms.author: jeedes
ms.openlocfilehash: 81949736603d22cac779d08d14bd6db65065d730
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92459082"
---
# <a name="tutorial-azure-active-directory-integration-with-lockpath-keylight"></a>Didacticiel : Intégration d’Azure Active Directory avec LockPath Keylight

Dans ce didacticiel, vous allez apprendre à intégrer LockPath Keylight avec Azure Active Directory (Azure AD).
L’intégration de LockPath Keylight avec Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à LockPath Keylight.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à LockPath Keylight (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec LockPath Keylight, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Un abonnement LockPath Keylight pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* LockPath Keylight prend en charge l’authentification unique lancée par le **fournisseur de services**
* LockPath Keylight prend en charge le provisionnement d’utilisateurs **juste-à-temps**

## <a name="adding-lockpath-keylight-from-the-gallery"></a>Ajout de LockPath Keylight à partir de la galerie

Pour configurer l’intégration de LockPath Keylight à Azure AD, vous devez ajouter LockPath Keylight à partir de la galerie à votre liste d’applications SaaS managées.

**Pour ajouter LockPath Keylight à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **LockPath Keylight** , sélectionnez **LockPath Keylight** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![LockPath Keylight dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec LockPath Keylight sur un utilisateur de test nommé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur LockPath Keylight associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec LockPath Keylight, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique LockPath Keylight](#configure-lockpath-keylight-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test LockPath Keylight](#create-lockpath-keylight-test-user)** pour avoir un équivalent de Britta Simon dans LockPath Keylight lié à sa représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec LockPath Keylight, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **LockPath Keylight** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL LockPath Keylight](common/sp-identifier-reply.png)

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<company name>.keylightgrc.com/`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<company name>.keylightgrc.com`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<company name>.keylightgrc.com/Login.aspx`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique LockPath Keylight](https://www.lockpath.com/contact/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML** , à la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Raw)** en fonction des options définies, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificateraw.png)

6. Dans la section **Configurer LockPath Keylight** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-lockpath-keylight-single-sign-on"></a>Configurer l’authentification unique LockPath Keylight

1. Pour activer l’authentification unique dans LockPath Keylight, procédez comme suit :

    a. Connectez-vous à votre compte LockPath Keylight en tant qu’administrateur.

    b. Dans le menu situé en haut, cliquez sur **Person** (Personne), puis sélectionnez **Keylight Setup** (Installation de Keylight).

    ![Capture d’écran montrant l’icône « Person » mise en évidence et l’option « Keylight Setup » sélectionnée dans la liste déroulante.](./media/keylight-tutorial/401.png)

    c. Dans l’arborescence sur la gauche, cliquez sur **SAML** .

    ![Capture d’écran montrant l’élément « SAML » sélectionné dans l’arborescence.](./media/keylight-tutorial/402.png)

    d. Dans la boîte de dialogue **SAML Settings** (Paramètres SAML), cliquez sur **Edit** (Modifier).

    ![Capture d’écran montrant la fenêtre « SAML Settings » avec le bouton « Edit » sélectionné.](./media/keylight-tutorial/404.png)

1. Dans la page de boîte de dialogue **Edit SAML Settings** (Modifier les paramètres SAML), procédez comme suit :

    ![Configure Single Sign-On](./media/keylight-tutorial/405.png)

    a. Affectez à **SAML authentication** (Authentication SAML) la valeur **Active** .

    b. Dans la zone de texte **Identity Provider Login URL** (URL de connexion du fournisseur d’identité), collez la valeur de l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    c. Dans la zone de texte **Identity Provider Logout URL** (URL de déconnexion du fournisseur d’identité), collez la valeur de l’ **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    d. Cliquez sur **Choose File** (Choisir un fichier) pour sélectionner votre certificat LockPath Keylight téléchargé, puis sur **Ouvrir** pour charger le certificat.

    e. Affectez à **SAML User Id location** (Emplacement de l’ID utilisateur SAML) la valeur **NameIdentifier element of the subject statement** (Élément NameIdentifier de l’instruction Subject).

    f. Indiquez la valeur **Keylight Service Provider** (Fournisseur de service Keylight) au format suivant : `https://<CompanyName>.keylightgrc.com`.

    g. Affectez à **Auto-provision users** (Configuration automatique des utilisateurs) la valeur **Active** .

    h. Affectez à **Auto-provision account type** (Configuration automatique du type de compte) la valeur **Full User** (Utilisateur global).

    i. Définissez **Auto-provision security role** (Configuration automatique du rôle de sécurité), sélectionnez **Standard User with SAML** (Utilisateur standard avec SAML).

    j. Définissez **Auto-provision security config** (Configuration automatique des paramètres de sécurité), sélectionnez **Standard User Configuration** (Configuration utilisateur standard).

    k. Dans la zone de texte **Email Attribute** (Attribut d’e-mail), entrez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`.

    l. Dans la zone de texte **First name attribute** (Attribut de prénom), entrez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`.

    m. Dans la zone de texte **Last name attribute** (Attribut de nom), entrez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`.

    n. Cliquez sur **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez `brittasimon@yourcompanydomain.extension`. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à LockPath Keylight.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **LockPath Keylight** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **LockPath Keylight** .

    ![Lien LockPath Keylight dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-lockpath-keylight-test-user"></a>Créer un utilisateur de test LockPath Keylight

Dans cette section, un utilisateur appelé Britta Simon est créé dans LockPath Keylight. LockPath Keylight prend en charge le provisionnement d’utilisateur juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans LockPath Keylight, il en est créé un après l’authentification. Si vous devez créer un utilisateur manuellement, contactez l’[équipe de support technique LockPath Keylight](https://www.lockpath.com/contact/).

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette LockPath Keylight dans le volet d’accès, vous devez être connecté automatiquement à l’application LockPath Keylight pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)