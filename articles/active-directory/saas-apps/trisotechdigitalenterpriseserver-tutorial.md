---
title: 'Tutoriel : intégration d’Azure Active Directory à Trisotech Digital Enterprise Server | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Trisotech Digital Enterprise Server.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/28/2019
ms.author: jeedes
ms.openlocfilehash: 05978b7300c401a0658d6c3316f37735fcb70c20
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92521932"
---
# <a name="tutorial-azure-active-directory-integration-with-trisotech-digital-enterprise-server"></a>Tutoriel : intégration d’Azure Active Directory à Trisotech Digital Enterprise Server

Dans ce tutoriel, vous allez apprendre à intégrer Trisotech Digital Enterprise Server avec Azure Active Directory (Azure AD).
L’intégration de Trisotech Digital Enterprise Server à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Trisotech Digital Enterprise Server.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Trisotech Digital Enterprise Server (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Trisotech Digital Enterprise Server, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Trisotech Digital Enterprise Server pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Trisotech Digital Enterprise Server prend en charge l’authentification unique lancée par le **fournisseur de services**

* Trisotech Digital Enterprise Server prend en charge le provisionnement d’utilisateurs **juste-à-temps**

## <a name="adding-trisotech-digital-enterprise-server-from-the-gallery"></a>Ajout de Trisotech Digital Enterprise Server à partir de la galerie

Pour configurer l’intégration de Trisotech Digital Enterprise Server à Azure AD, vous devez ajouter Trisotech Digital Enterprise Server, disponible dans la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter Trisotech Digital Enterprise Server à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Trisotech Digital Enterprise Server** , sélectionnez **Trisotech Digital Enterprise Server** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Trisotech Digital Enterprise Server dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Trisotech Digital Enterprise Server grâce à un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Trisotech Digital Enterprise Server associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Trisotech Digital Enterprise Server, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Trisotech Digital Enterprise Server](#configure-trisotech-digital-enterprise-server-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Trisotech Digital Enterprise Server](#create-trisotech-digital-enterprise-server-test-user)** pour avoir un équivalent de Britta Simon dans Trisotech Digital Enterprise Server, lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Trisotech Digital Enterprise Server, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Trisotech Digital Enterprise Server** , sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique de domaine et d’URL Trisotech Digital Enterprise Server](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<companyname>.trisotech.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<companyname>.trisotech.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Contactez l’[équipe de support Trisotech Digital Enterprise Server](mailto:support@trisotech.com) pour obtenir ces valeurs. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

4. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

### <a name="configure-trisotech-digital-enterprise-server-single-sign-on"></a>Configurer l’authentification unique Trisotech Digital Enterprise Server

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise Trisotech Digital Enterprise Server Configuration en tant qu’administrateur.

2. Cliquez sur l’ **icône de menu** , puis sélectionnez **Administration**.

    ![Capture d’écran montrant l’icône Administration dans Microsoft Digital Enterprise Server](./media/trisotechdigitalenterpriseserver-tutorial/user1.png)

3. Sélectionnez **User Provider (Fournisseur d’utilisateur)** .

    ![Capture d’écran montrant User Provider sélectionné dans le menu](./media/trisotechdigitalenterpriseserver-tutorial/user2.png)

4. Dans la section **User Provider Configuration** , effectuez les étapes suivantes :

    ![Capture d’écran montrant User Provider Configurations où vous pouvez entrer les valeurs décrites](./media/trisotechdigitalenterpriseserver-tutorial/user3.png)

    a. Sélectionnez **Secured Assertion Markup Language 2 (SAML 2)** dans la liste déroulante dans **Authentication Method**.

    b. Dans la zone de texte **URL des métadonnées** , collez la valeur **URL des métadonnées de fédération de l’application** que vous avez copiée sur le portail Azure.

    c. Dans la zone de texte **Application ID** , entrez l’URL au format suivant : `https://<companyname>.trisotech.com`.

    d. Cliquez sur **Enregistrer**.

    e. Entrez le nom de domaine dans la zone de texte **Allowed Domains (empty means everyone)** . Des licences sont affectées automatiquement pour les utilisateurs correspondant aux domaines autorisés.

    f. Cliquez sur **Enregistrer**.

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Trisotech Digital Enterprise Server.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **IBM Trisotech Digital Enterprise Server**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Trisotech Digital Enterprise Server**.

    ![Le lien Trisotech Digital Enterprise Server dans la liste des Applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

### <a name="create-trisotech-digital-enterprise-server-test-user"></a>Créer un utilisateur de test Trisotech Digital Enterprise Server

Dans cette section, un utilisateur appelé Britta Simon est créé dans Trisotech Digital Enterprise Server. Trisotech Digital Enterprise Server prend en charge le provisionnement d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Trisotech Digital Enterprise Server, il en est créé un après l’authentification.

>[!Note]
>Si vous avez besoin de créer un utilisateur manuellement, contactez l’[équipe de support Trisotech Digital Enterprise Server](mailto:support@trisotech.com).

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Trisotech Digital Enterprise Server dans le volet d’accès, vous devez être connecté automatiquement à l’application Trisotech Digital Enterprise Server pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)