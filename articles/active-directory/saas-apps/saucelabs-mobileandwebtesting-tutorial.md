---
title: 'Tutoriel : Intégration d’Azure Active Directory à Sauce Labs - Mobile and Web Testing | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Sauce Labs - Mobile and Web Testing.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/22/2019
ms.author: jeedes
ms.openlocfilehash: 087e2d7f2db8f27378b54675095e97256d6aae9b
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895105"
---
# <a name="tutorial-azure-active-directory-integration-with-sauce-labs---mobile-and-web-testing"></a>Tutoriel : Intégration d’Azure Active Directory à Sauce Labs - Mobile and Web Testing

Dans ce tutoriel, découvrez comment intégrer Sauce Labs - Mobile and Web Testing à Azure Active Directory (Azure AD).
L’intégration de Sauce Labs - Mobile and Web Testing à Azure AD vous procure les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Sauce Labs - Mobile and Web Testing.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Sauce Labs - Mobile and Web Testing (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Sauce Labs - Mobile and Web Testing, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Abonnement Sauce Labs - Mobile and Web Testing avec l’authentification unique activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Sauce Labs - Mobile and Web Testing prend en charge l’authentification unique lancée par le **fournisseur de services**
* Sauce Labs - Mobile and Web Testing prend en charge l’attribution d’utilisateurs **Juste-à-temps**

## <a name="adding-sauce-labs---mobile-and-web-testing-from-the-gallery"></a>Ajout de Sauce Labs - Mobile and Web Testing à partir de la galerie

Pour configurer l’intégration de Sauce Labs - Mobile and Web Testing dans Azure AD, vous devez ajouter Sauce Labs - Mobile and Web Testing à votre liste d’applications SaaS gérées à partir de la galerie.

**Pour ajouter Sauce Labs - Mobile and Web Testing à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Sauce Labs - Mobile and Web Testing** , sélectionnez **Sauce Labs - Mobile and Web Testing** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Sauce Labs - Mobile and Web Testing dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous configurez et testez l’authentification unique Azure AD avec Sauce Labs - Mobile and Web Testing pour un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans Sauce Labs - Mobile and Web Testing.

Pour configurer et tester l’authentification unique Azure AD avec Sauce Labs - Mobile and Web Testing, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Sauce Labs - Mobile and Web Testing](#configure-sauce-labs---mobile-and-web-testing-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Sauce Labs - Mobile and Web Testing](#create-sauce-labs---mobile-and-web-testing-test-user)** pour avoir, dans Sauce Labs - Mobile and Web Testing, un équivalent de Britta Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Sauce Labs - Mobile and Web Testing, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Sauce Labs - Mobile and Web Testing** , sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , l’utilisateur n’a rien à faire, car l’application est déjà intégrée à Azure.

    ![Informations d’authentification unique pour le domaine et les URL de Sauce Labs - Mobile and Web Testing](common/preintegrated.png)

5. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configurer Sauce Labs - Mobile and Web Testing** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-sauce-labs---mobile-and-web-testing-single-sign-on"></a>Configurer l’authentification unique pour Sauce Labs - Mobile and Web Testing

1. Dans une autre fenêtre du navigateur web, ouvrez une session sur votre site d’entreprise Sauce Labs - Mobile and Web Testing en tant qu’administrateur.

2. Cliquez sur l’ **icône Utilisateur** et sélectionnez l’onglet **Gestion des équipes**.

    ![Capture d’écran montrant l’icône représentant un utilisateur et l’option « Team Management » sélectionnée](./media/saucelabs-mobileandwebtesting-tutorial/configure1.png)

3. Entrez votre **nom de domaine** dans la zone de texte.

    ![Capture d’écran montrant un exemple de nom de domaine dans la zone de texte](./media/saucelabs-mobileandwebtesting-tutorial/configure2.png)

4. Cliquez sur l’onglet **Configurer**.

    ![Capture d’écran montrant l’onglet « Configure » sélectionné sous « Single Sign On is Enabled »](./media/saucelabs-mobileandwebtesting-tutorial/configure3.png)

5. Dans la section **Configurer l’authentification unique** , procédez comme suit.

    ![Configure Single Sign-On](./media/saucelabs-mobileandwebtesting-tutorial/configure4.png)

    a. Cliquez sur **Parcourir** et chargez le fichier de métadonnées téléchargé à partir d’Azure AD.

    b. Cochez la case **AUTORISER L’APPROVISIONNEMENT JUSTE-À-TEMPS**.

    c. Cliquez sur **Enregistrer**.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon**.
  
    b. Dans le champ **Nom d’utilisateur** , tapez `brittasimon@yourcompanydomain.extension`  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Sauce Labs - Mobile and Web Testing.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Sauce Labs - Mobile and Web Testing**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Sauce Labs - Mobile and Web Testing**.

    ![Lien Sauce Labs - Mobile and Web Testing dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

### <a name="create-sauce-labs---mobile-and-web-testing-test-user"></a>Créer un utilisateur de test Sauce Labs - Mobile and Web Testing

Dans cette section, vous créez un utilisateur appelé Britta Simon dans Sauce Labs - Mobile and Web Testing. Sauce Labs - Mobile and Web Testing prend en charge l’attribution d’utilisateurs juste-à-temps, une option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. Si l’utilisateur souhaité n’existe pas déjà dans Sauce Labs - Mobile and Web Testing, il est créé après l’authentification.

> [!Note]
> Si vous devez créer un utilisateur manuellement, contactez [l’équipe de support de Sauce Labs - Mobile and Web Testing](mailto:support@saucelabs.com).

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Sauce Labs - Mobile and Web Testing dans le volet d’accès, vous devez être connecté automatiquement à l’application Sauce Labs - Mobile and Web Testing pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)