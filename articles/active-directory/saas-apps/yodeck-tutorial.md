---
title: 'Tutoriel : Intégration d’Azure Active Directory à Yodeck | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Yodeck.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/29/2019
ms.author: jeedes
ms.openlocfilehash: 68e961336d7e86f9983fea49044137894ab0f985
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895077"
---
# <a name="tutorial-azure-active-directory-integration-with-yodeck"></a>Tutoriel : Intégration d’Azure Active Directory à Yodeck

Dans ce tutoriel, vous allez découvrir comment intégrer Yodeck à Azure Active Directory (Azure AD).
L’intégration de Yodeck avec Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Yodeck
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Yodeck (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec Yodeck, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Yodeck pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Yodeck prend en charge l’authentification unique démarrée par le **fournisseur de services** et le **fournisseur d’identité**

## <a name="adding-yodeck-from-the-gallery"></a>Ajout de Yodeck à partir de la galerie

Pour configurer l’intégration de Yodeck avec Azure AD, vous devez ajouter Yodeck, à partir de la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter Yodeck à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Yodeck** , sélectionnez **Yodeck** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Yodeck dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Yodeck, avec un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur Yodeck associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Yodeck, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Yodeck](#configure-yodeck-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Yodeck](#create-yodeck-test-user)** pour avoir un équivalent de Britta Simon dans Yodeck lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Yodeck, effectuez les étapes suivantes :

1. Dans le [Portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Yodeck** , sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , effectuez l’étape suivante :

    ![Informations d’authentification unique dans Domaine et URL Yodeck](common/idp-identifier.png)

    Dans la zone de texte **Identificateur** , tapez une URL :  `https://app.yodeck.com/api/v1/account/metadata/`

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    ![Capture d’écran montrant la Configuration SAML de base avec le lien Charger un fichier de métadonnées.](common/both-preintegrated-signon.png)

    Dans la zone de texte **URL de connexion** , tapez une URL : `https://app.yodeck.com/login`

6. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

### <a name="configure-yodeck-single-sign-on"></a>Configurer l’authentification unique Yodeck

1. Pour automatiser la configuration dans **Yodeck** , vous devez installer l’ **extension de navigateur de connexion sécurisée à Mes applications** en cliquant sur **Installer l’extension**.

    ![Capture d’écran montrant le bouton Installer l’extension.](./media/target-process-tutorial/install_extension.png)

1. Une fois l’extension ajoutée au navigateur, le fait de cliquer sur **Configurer Yodeck** vous redirige vers l’application Yodeck. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Yodeck. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 5.

    ![Configuration](common/setup-sso.png)

    **Si vous souhaitez configurer l'application manuellement, procédez comme suit :**

1. Dans une autre fenêtre de navigateur web, connectez-vous au site de votre entreprise Yodeck en tant qu’administrateur.

1. Cliquez sur **User Settings** dans l’angle supérieur droit de la page et sélectionnez **Account Settings**.

    ![Capture d’écran montrant l’élément Account Settings sélectionné pour l’utilisateur.](./media/yodeck-tutorial/configure1.png)

1. Sélectionnez **SAML** et effectuez les opérations suivantes :

    ![Capture d’écran montrant l’onglet SAML où vous pouvez effectuer ces étapes.](./media/yodeck-tutorial/configure2.png)

    a. Sélectionnez **Import from URL**.

    b. Dans la zone de texte **URL** , collez la valeur **URL des métadonnées de fédération de l’application** que vous avez copiée dans le portail Azure, puis cliquez sur **Import**.
    
    c. Une fois l’ **URL des métadonnées de fédération de l’application** importée, les champs restants sont renseignés automatiquement.

    d. Cliquez sur **Enregistrer**.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon**.
  
    b. Dans le champ **Nom d’utilisateur** , tapez brittasimon@yourcompanydomain.extension. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Yodeck.

1. Dans le Portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Yodeck**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Yodeck**.

    ![Lien Yodeck dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

### <a name="create-yodeck-test-user"></a>Créer un utilisateur de test Yodeck

Pour se connecter à Yodeck, les utilisateurs d’Azure AD doivent être approvisionnés dans Yodeck. Dans le cas de Yodeck, le provisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise Yodeck en tant qu’administrateur.

2. Cliquez sur **User Settings** dans l’angle supérieur droit de la page et sélectionnez **Users**.

    ![Capture d’écran montrant l’élément Users sélectionné pour l’utilisateur.](./media/yodeck-tutorial/user1.png)

3. Cliquez sur **+User** pour ouvrir l’onglet **User Details**.

    ![Capture d’écran montrant le bouton Users.](./media/yodeck-tutorial/user2.png)

4. Dans la page de boîte de dialogue **Détails de l'utilisateur** , procédez comme suit :

    ![Capture d’écran montrant l’onglet User Details où vous pouvez effectuer ces étapes.](./media/yodeck-tutorial/user3.png)

    a. Dans la zone de texte **First Name** , entrez le prénom de l’utilisateur, par exemple **Britta**.

    b. Dans la zone de texte **Last Name** , tapez le nom de l’utilisateur, par exemple **Simon**.

    c. Dans la zone de texte **Email** (E-mail), tapez l’adresse e-mail d’un utilisateur, par exemple, brittasimon@contoso.com.

    d. Sélectionnez l’option **Account Permissions** conforme aux exigences de votre organisation.
    
    e. Cliquez sur **Enregistrer**.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la vignette Yodeck dans le volet d’accès, vous devez être connecté automatiquement à l’application Yodeck pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)