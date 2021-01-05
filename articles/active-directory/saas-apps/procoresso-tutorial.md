---
title: 'Didacticiel : Intégration d’Azure Active Directory à Procore SSO | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Procore SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/03/2019
ms.author: jeedes
ms.openlocfilehash: 13f8f1067ce7c9fe55160400d20ec0b20788c17b
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92515285"
---
# <a name="tutorial-azure-active-directory-integration-with-procore-sso"></a>Didacticiel : Intégration d’Azure Active Directory à Procore SSO

Dans ce didacticiel, vous allez apprendre à intégrer Procore SSO à Azure Active Directory (Azure AD).
L’intégration de Procore SSO dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Procore SSO.
* Vous pouvez permettre à vos utilisateurs de se connecter automatiquement à Procore SSO (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Procore SSO, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Un abonnement Procore SSO pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Procore SSO prend en charge l’authentification unique lancée par le **fournisseur d’identité**

## <a name="adding-procore-sso-from-the-gallery"></a>Ajout de Procore SSO à partir de la galerie

Pour configurer l’intégration de Procore SSO à Azure AD, vous devez ajouter Procore SSO à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Procore SSO à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Procore SSO** , sélectionnez **Procore SSO** dans le panneau des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Procore SSO dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Procore SSO au moyen d’un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Procore SSO associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Procore SSO, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Procore SSO](#configure-procore-sso-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Procore SSO](#create-procore-sso-test-user)** pour avoir dans Procore SSO un équivalent de Britta Simon lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Procore SSO, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), sur la page d’intégration de l’application **Procore SSO** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , l’utilisateur n’a rien à faire, car l’application est déjà intégrée à Azure.

    ![Informations d’authentification unique dans Domaine et URL Procore SSO](common/preintegrated.png)

5. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configurer Procore SSO** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-procore-sso-single-sign-on"></a>Configurer l’authentification unique Procore SSO

1. Pour configurer l’authentification unique sur **Procore SSO** , connectez-vous à votre site d’entreprise Procore en tant qu’administrateur.

2. Dans la liste déroulante de boîte à outils, cliquez sur **Admin** pour ouvrir la page de paramètres de l’authentification unique.

    ![Capture d’écran affichant le site d’entreprise Procore avec l’élément Directory sélectionné.](./media/procoresso-tutorial/procore_tool_admin.png)

3. Collez les valeurs dans les zones comme décrit ci-dessous :

    ![Capture d’écran montrant la boîte de dialogue Add a Person.](./media/procoresso-tutorial/procore_setting_admin.png) 

    a. Dans la zone de texte **URL de l'émetteur de l'authentification unique** , collez la valeur de l' **Identificateur Azure AD** copiée à partir du portail Azure.

    b. Dans la zone **URL de la cible de l'authentification SAML** , collez la valeur de l' **URL de connexion** copiée à partir du portail Azure.

    c. Ouvrez maintenant le fichier **XML de métadonnées de fédération** téléchargé à partir du portail Azure et copiez le certificat dans la balise nommée **X509Certificate** . Collez la valeur copiée dans la zone **Certificat x509 d’authentification unique** .

4. Cliquez sur **Enregistrer les modifications** .

5. Après ces paramètres, vous devez envoyer le **nom de domaine** (par ex. **contoso.com** ) via lequel vous connectez à Procore à [l’équipe de support technique de Procore](https://support.procore.com/), qui activera l’authentification unique fédérée pour ce domaine.

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en accordant l’accès à Procore SSO.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Procore SSO** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Procore SSO** .

    ![Lien Procore SSO dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-procore-sso-test-user"></a>Créer un utilisateur de test Procore SSO

Effectuez les étapes ci-dessous pour créer un utilisateur de test Procore côté Procore SSO.

1. Connectez-vous à votre site d’entreprise Procore en tant qu’administrateur.    

2. Dans la liste déroulante de boîte à outils, cliquez sur **Annuaire** pour ouvrir la page d’annuaire de l’entreprise.

    ![Capture d’écran affichant le site d’entreprise Procore avec l’élément Directory sélectionné dans la boîte à outils.](./media/procoresso-tutorial/Procore_sso_directory.png)

3. Cliquez sur l’option **Ajouter une personne** pour ouvrir le formulaire et réglez les options suivantes -

    ![Capture d’écran affichant la zone d’ajout d’une personne à Boylan Construction, dans laquelle vous pouvez entrer des informations sur l’utilisateur.](./media/procoresso-tutorial/Procore_user_add.png)

    a. Dans la zone de texte **First Name** , tapez le prénom de l’utilisateur, par exemple **Britta** .

    b. Dans la zone de texte **Last Name** , tapez le nom de l’utilisateur, par exemple **Simon** .

    c. Dans la zone de texte **Adresse e-mail** , tapez l’adresse e-mail de l’utilisateur, par exemple BrittaSimon@contoso.com.

    d. Sélectionnez **Modèle d’autorisation** pour **Appliquer le modèle d’autorisation plus tard** .

    e. Cliquez sur **Créer** .

4. Vérifiez et mettez à jour les détails du contact nouvellement ajouté.

    ![Capture d’écran affichant une page de modification dans laquelle vous pouvez vérifier les paramètres de l’utilisateur.](./media/procoresso-tutorial/Procore_user_check.png)

5. Cliquez sur **Enregistrer et envoyer l’invitation** (si une invitation par e-mail est exigée) ou sur **Enregistrer** (pour enregistrer directement) afin de terminer l’inscription de l’utilisateur.
    
    ![Capture d’écran affichant les paramètres actuels du projet, dans lesquels vous pouvez enregistrer et envoyer une invitation.](./media/procoresso-tutorial/Procore_user_save.png)

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Procore SSO dans le volet d’accès, vous devez être connecté automatiquement à l’application Procore SSO pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)