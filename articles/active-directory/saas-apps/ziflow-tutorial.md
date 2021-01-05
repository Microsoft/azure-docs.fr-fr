---
title: 'Didacticiel : Intégration d’Azure Active Directory à Ziflow | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Ziflow.
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
ms.openlocfilehash: e10ca98e0c9257187288d02483ed81915a7b321d
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92894718"
---
# <a name="tutorial-azure-active-directory-integration-with-ziflow"></a>Didacticiel : Intégration d’Azure Active Directory à Ziflow

Dans ce didacticiel, vous allez apprendre à intégrer Ziflow dans Azure Active Directory (Azure AD).
L’intégration de Ziflow dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Ziflow.
* Vous pouvez permettre à vos utilisateurs de se connecter automatiquement à Ziflow (par le biais de l'authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD dans Ziflow, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Un abonnement Ziflow pour lequel l'authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Ziflow prend en charge l'authentification unique initiée par le **fournisseur de services**

## <a name="adding-ziflow-from-the-gallery"></a>Ajout de Ziflow depuis la galerie

Pour configurer l’intégration de Ziflow dans Azure AD, vous devez ajouter Ziflow depuis la galerie dans votre liste d’applications SaaS gérées.

**Pour ajouter Ziflow à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Ziflow** , sélectionnez **Ziflow** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Ziflow dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l'authentification unique Azure AD auprès de Ziflow avec un utilisateur de test appelé **Britta Simon**.
Pour que l'authentification unique fonctionne, un lien doit être établi entre un utilisateur Azure AD et l'utilisateur Ziflow associé.

Pour configurer et tester l’authentification unique Azure AD avec Ziflow, vous devez suivre les étapes ci-dessous :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l'authentification unique Ziflow](#configure-ziflow-single-sign-on)** pour configurer les paramètres de l'authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Ziflow](#create-ziflow-test-user)** pour avoir dans Ziflow un équivalent de Britta Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l'authentification unique Azure AD auprès de Ziflow, procédez comme suit :

1. Sur le [portail Azure](https://portal.azure.com/), accédez à la page d'intégration de l'application **Ziflow** et sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Ziflow](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://ziflow-production.auth0.com/login/callback?connection=<UniqueID>`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `urn:auth0:ziflow-production:<UniqueID>`

    > [!NOTE]
    > Les valeurs ci-dessus ne sont pas réelles. Vous allez remplacer l'ID unique dans Identificateur et URL de connexion par la valeur réelle. La procédure est expliquée plus loin dans le didacticiel.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer Ziflow** , copiez les URL appropriées, selon vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-ziflow-single-sign-on"></a>Configurer l'authentification unique Ziflow

1. Ouvrez une nouvelle fenêtre dans le navigateur web, puis connectez-vous à Ziflow en tant qu'administrateur de la sécurité.

2. Dans l’angle supérieur droit, cliquez sur Avatar, puis cliquez sur **Gérer le compte**.

    ![Configuration de Ziflow - Gestion](./media/ziflow-tutorial/tutorial_ziflow_manage.png)

3. Dans le coin supérieur gauche, cliquez sur **Authentification unique**.

    ![Configuration de Ziflow - Connexion](./media/ziflow-tutorial/tutorial_ziflow_signon.png)

4. Sur la page **Authentification unique** , effectuez les opérations suivantes :

    ![Configuration de Ziflow - Authentification unique](./media/ziflow-tutorial/tutorial_ziflow_page.png)

    a. Sélectionnez **Type** dans **SAML2.0**.

    b. Dans la zone de texte **URL de connexion** , collez la valeur de l' **URL de connexion** que vous avez copiée à partir du portail Azure.

    c. Téléchargez le certificat codé en base 64 que vous avez téléchargé à partir du portail Azure, dans le **Certificat de signature X509**.

    d. Dans la zone de texte **URL de déconnexion** , collez la valeur de l' **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    e. Dans la section **Paramètres de configuration de votre fournisseur d'identificateur** , copiez l'ID unique en surbrillance et ajoutez-le à l'identificateur et à l'URL de connexion dans la section **Configuration SAML de base** du portail Azure.

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Ziflow.

1. Sur le portail Azure, sélectionnez **Applications d'entreprise** , **Toutes les applications** , puis **Ziflow**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Ziflow**.

    ![Lien Ziflow dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

### <a name="create-ziflow-test-user"></a>Créer un utilisateur de test Ziflow

Pour se connecter à Ziflow, les utilisateurs d'Azure AD doivent être approvisionnés dans Ziflow. Dans Ziflow, l’approvisionnement est une tâche manuelle.

Pour approvisionner un compte d’utilisateur, procédez comme suit :

1. Connectez-vous à Ziflow en tant qu'administrateur de la sécurité.

2. Accédez à **Personnes** en haut.

    ![Configuration de Ziflow - Personnes](./media/ziflow-tutorial/tutorial_ziflow_people.png)

3. Cliquez sur **Ajouter** , puis sur **Continuer**.

    ![Capture d’écran montrant l’option Add user sélectionnée.](./media/ziflow-tutorial/tutorial_ziflow_add.png)

4. Dans la section **Add a User (Ajouter un utilisateur)** , procédez comme suit :

    ![Capture d’écran montrant la boîte de dialogue Add a user dans laquelle vous pouvez entrer les valeurs décrites.](./media/ziflow-tutorial/tutorial_ziflow_adduser.png)

    a. Dans la zone de texte **E-mail** , entrez l’adresse e-mail de l’utilisateur, comme brittasimon@contoso.com.

    b. Dans la zone de texte **Prénom** , saisissez le prénom de l’utilisateur, par exemple Britta.

    c. Dans la zone de texte **Nom** , saisissez le nom de famille de l’utilisateur, par exemple Simon.

    d. Sélectionnez votre rôle Ziflow.

    e. Cliquez sur **Add 1 user (Ajouter un utilisateur)** .

    > [!NOTE]
    > Le titulaire du compte Azure Active Directory reçoit un e-mail contenant un lien à suivre pour confirmer son compte et l’activer.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Ziflow dans le volet d'accès doit vous connecter automatiquement à l'application Ziflow pour laquelle vous avez configuré l'authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)