---
title: 'Didacticiel : Intégration d’Azure Active Directory avec Pingboard | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Pingboard.
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
ms.openlocfilehash: d7b090eb5228a449212ceebd6cd299c5f8860089
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92520793"
---
# <a name="tutorial-azure-active-directory-integration-with-pingboard"></a>Didacticiel : Intégration d’Azure Active Directory avec Pingboard

Ce didacticiel explique comment intégrer Pingboard avec Azure Active Directory (Azure AD).
L’intégration de Pingboard avec Azure AD offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Pingboard.
* Vous pouvez permettre à vos utilisateurs d’être connectés automatiquement à Pingboard (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec Pingboard, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Pingboard pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Pingboard prend en charge l’authentification unique (SSO) initiée par le **fournisseur de services** et le **fournisseur d’identité**

* Pingboard prend en charge l’[attribution d’utilisateurs automatisée](./pingboard-provisioning-tutorial.md). 

## <a name="adding-pingboard-from-the-gallery"></a>Ajout de Pingboard à partir de la galerie

Pour configurer l’intégration de Pingboard avec Azure AD, vous devez ajouter Pingboard, disponible dans la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter Pingboard à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Pingboard** , sélectionnez **Pingboard** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Pingboard dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Pingboard sur un utilisateur de test nommé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur Pingboard associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Pingboard, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Pingboard](#configure-pingboard-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Pingboard](#create-pingboard-test-user)** - pour avoir un équivalent de Britta Simon dans Pingboard lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Pingboard, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Pingboard** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , effectuez les étapes suivantes :

    ![Capture d’écran montrant la « Configuration SAML de base » avec les zones de texte « Identificateur » et « URL de réponse » mises en évidence, et le bouton « Enregistrer » sélectionné.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL : `http://app.pingboard.com/sp`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<entity-id>.pingboard.com/auth/saml/consume`

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Pingboard](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<sub-domain>.pingboard.com/sign_in`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de réponse et l’URL de connexion réelles. Pour obtenir ces valeurs, contactez l’[équipe de support technique Pingboard](https://support.pingboard.com/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

6. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

7. Dans la section **Configurer Pingboard** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-pingboard-single-sign-on"></a>Configurer l’authentification unique Pingboard

1. Pour configurer l’authentification unique côté Pingboard, ouvrez une nouvelle fenêtre de navigateur et connectez-vous à votre compte Pingboard. Pour configurer l’authentification unique, vous devez être un administrateur Pingboard.

2. Dans le menu supérieur, sélectionnez **Applications > Intégrations**

    ![Configure Single Sign-On](./media/pingboard-tutorial/Pingboard_integration.png)

3. Sur la page **Intégrations** , recherchez la mosaïque **« Azure Active Directory »** , puis cliquez dessus.

    ![Intégration de l’authentification unique Pingboard](./media/pingboard-tutorial/Pingboard_aad.png)

4. Dans la fenêtre modale qui suit, cliquez sur **« Configurer »**

    ![Bouton de configuration Pingboard](./media/pingboard-tutorial/Pingboard_configure.png)

5. Sur la page suivante, vous remarquez que « l’intégration Azure SSO est activée ». Ouvrez le fichier XML de métadonnées téléchargé dans un bloc-notes puis collez le contenu dans **IDP Metadata** (métadonnées du fournisseur d’identité).

    ![Écran de configuration SSO de Pingboard](./media/pingboard-tutorial/Pingboard_sso_configure.png)

6. Le fichier est validé et, si tout est correct, l’authentification unique est alors activée.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez brittasimon@yourcompanydomain.extension. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Pingboard.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Pingboard** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Pingboard** .

    ![Lien correspondant à Pingboard dans la liste Applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-pingboard-test-user"></a>Créer un utilisateur de test Pingboard

L’objectif de cette section est de créer un utilisateur appelé Britta Simon dans Pingboard. Pingboard prend en charge l’attribution automatique d’utilisateurs, qui est activée par défaut. Vous trouverez plus d’informations [ici](pingboard-provisioning-tutorial.md) sur la façon de configurer l’attribution automatique d’utilisateurs.

**Si vous avez besoin de créer un utilisateur manuellement, procédez comme suit :**

1. Connectez-vous au site d’entreprise Pingboard en tant qu’administrateur.

2. Cliquez sur le bouton **« Ajouter un employé »** sur la page **Annuaire** .

    ![Ajouter un employé](./media/pingboard-tutorial/create_testuser_add.png)

3. Dans la boîte de dialogue **Ajouter un employé** , procédez comme suit :

    ![Inviter des personnes](./media/pingboard-tutorial/create_testuser_name.png)

    a. Dans la zone de texte **Nom complet** , tapez le nom complet d’un utilisateur, par exemple, **Britta Simon** .

    b. Dans la zone de texte **E-mail** , tapez l’adresse e-mail d’un utilisateur, par exemple, **brittasimon@contoso.com** .

    c. Dans la zone de texte **Poste** , tapez le poste de Britta Simon.

    d. Dans la liste déroulante **Emplacement** , sélectionnez l’emplacement de Britta Simon.

    e. Cliquez sur **Ajouter** .

4. Un écran de confirmation s’affiche pour confirmer l’ajout de l’utilisateur.

    ![Confirmer](./media/pingboard-tutorial/create_testuser_confirm.png)

    > [!NOTE]
    > Le titulaire du compte Azure Active Directory reçoit un e-mail contenant un lien à suivre pour confirmer son compte et l’activer.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Pingboard dans le panneau d’accès doit vous connecter automatiquement à l’application Pingboard pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Configurer l’approvisionnement de l’utilisateur](./pingboard-provisioning-tutorial.md)