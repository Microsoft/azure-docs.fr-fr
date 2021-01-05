---
title: 'Tutoriel : Intégration d’Azure Active Directory à HighGear | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et HighGear.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/16/2019
ms.author: jeedes
ms.openlocfilehash: b3806d768e8335801e2dd097eac7a440b157de38
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92444677"
---
# <a name="tutorial-azure-active-directory-integration-with-highgear"></a>Tutoriel : Intégration d’Azure Active Directory à HighGear

Dans ce tutoriel, vous pouvez apprendre à intégrer HighGear à Azure Active Directory (Azure AD).
L’intégration de HighGear à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à HighGear.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à HighGear (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à HighGear, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un système HighGear avec une licence Enterprise ou Unlimited

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous pouvez apprendre à configurer et à tester l’authentification unique Azure AD dans un environnement de test.

* HighGear prend en charge l’authentification unique démarrée par **le fournisseur de services et le fournisseur d’identité**

## <a name="adding-highgear-from-the-gallery"></a>Ajout de HighGear à partir de la galerie

Pour configurer l’intégration de HighGear à Azure AD, vous devez ajouter HighGear, disponible dans la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter HighGear à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **HighGear** , sélectionnez **HighGear** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![HighGear dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous pouvez apprendre à configurer et à tester l’authentification unique Azure AD auprès de votre système HighGear à l’aide d’un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur associé dans votre système HighGear doit être établie.

Pour configurer et tester l’authentification unique Azure AD auprès de votre système HighGear, vous devez effectuer les modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique HighGear](#configure-highgear-single-sign-on)** pour configurer les paramètres de l’authentification unique du côté de l’application HighGear.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer utilisateur de test HighGear](#create-highgear-test-user)** pour avoir un équivalent de Britta Simon dans HighGear lié à la représentation Azure AD associée. 
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous pouvez apprendre à activer l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD auprès de votre système HighGear, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **HighGear** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Capture d’écran montrant Basic SAML Configuration, où vous pouvez entrer Identifier, et Reply URL, et sélectionner Save.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , collez la valeur du champ **Service Provider Entity ID** (ID d’entité du fournisseur de services) figurant dans la page Single Sign-On Settings (Paramètres d’authentification unique) au sein de votre système HighGear.

    ![Champ Service Provider Entity ID (ID d’entité du fournisseur de services)](media/highgear-tutorial/service-provider-entity-id-field.png)
    
    > [!NOTE]
    > Vous devez vous connecter à votre système HighGear pour accéder à la page Single Sign-On Settings (Paramètres d’authentification unique). Une fois que vous êtes connecté, déplacez votre souris sur l’onglet Administration dans HighGear et cliquez sur l’élément de menu Single Sign-On Settings (Paramètres d’authentification unique).
    
    ![Élément de menu Single Sign-On Settings (Paramètres d’authentification unique)](media/highgear-tutorial/single-sign-on-settings-menu-item.png)

    b. Dans la zone de texte **URL de réponse** , collez la valeur du champ **Assertion Consumer Service (ACS) URL** (URL du service ACS) à partir de la page Single Sign-On Settings (Paramètres d’authentification unique) de votre système HighGear.

    ![Champ Assertion Consumer Service (ACS) URL (URL du service ACS)](media/highgear-tutorial/assertion-consumer-service-url-field.png)

    c. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

     ![Capture d’écran montrant Set additional URLs, où vous pouvez entrer une URL de connexion.](common/metadata-upload-additional-signon.png)

     Dans la zone de texte **URL de connexion** , collez la valeur du champ **Service Provider Entity ID** (ID d’entité du fournisseur de services) figurant dans la page Single Sign-On Settings (Paramètres d’authentification unique) au sein de votre système HighGear. (Cet ID d’entité est également l’URL de base du système HighGear qui doit être utilisée pour l’authentification démarrée par le fournisseur de services.)

    ![Champ Service Provider Entity ID (ID d’entité du fournisseur de services)](media/highgear-tutorial/service-provider-entity-id-field.png)

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels à partir de la page **Single Sign-On Settings** (Paramètres d’authentification unique) de votre système HighGear. Si vous avez besoin d’aide, contactez l’[équipe du support technique HighGear](mailto:support@highgear.com).

4. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** , puis enregistrez-le sur votre ordinateur. Vous en aurez besoin à une étape ultérieure de la configuration de l’authentification unique.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer HighGear** , notez l’emplacement des URL suivantes.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion. Vous aurez besoin de cette valeur à l’étape 2 de la procédure **Configurer l’authentification unique HighGear** ci-après.

    b. Identificateur Azure AD. Vous aurez besoin de cette valeur à l’étape 3 de la procédure **Configurer l’authentification unique HighGear** ci-après.

    c. URL de déconnexion. Vous aurez besoin de cette valeur à l’étape 4 de la procédure **Configurer l’authentification unique HighGear** ci-après.

### <a name="configure-highgear-single-sign-on"></a>Configurer l’authentification unique HighGear

Pour configurer HighGear pour l’authentification unique, veuillez vous connecter à votre système HighGear. Une fois que vous êtes connecté, déplacez votre souris sur l’onglet Administration dans HighGear et cliquez sur l’élément de menu Single Sign-On Settings (Paramètres d’authentification unique).

![Élément de menu Single Sign-On Settings (Paramètres d’authentification unique)](media/highgear-tutorial/single-sign-on-settings-menu-item.png)

1. Dans la zone **Identity Provider Name** (Nom du fournisseur d’identité), tapez une brève description qui apparaîtra sur le bouton Single Sign-On (Authentification unique) de HighGear dans la page Login (Connexion). Par exemple : Azure AD

2. Dans le champ **Single Sign-On (SSO) URL** (URL d’authentification unique) dans HighGear, collez la valeur du champ **URL de connexion** de la section **Configurer HighGear** dans Azure.

3. Dans le champ **Identity Provider Entity ID** (ID d’entité du fournisseur d’identité) dans HighGear, collez la valeur du champ **Identificateur Azure AD** de la section **Configurer HighGear** dans Azure.

4. Dans le champ **Single Logout (SLO) URL** (URL de déconnexion unique) dans HighGear, collez la valeur du champ **URL de déconnexion** de la section **Configurer HighGear** dans Azure.

5. Utilisez le Bloc-notes pour ouvrir le certificat que vous avez téléchargé à partir de la section **Certificat de signature SAML** dans Azure. Vous devez avoir téléchargé le format **Certificat (Base64)** . Copiez le contenu du certificat à partir du Bloc-notes et collez-le dans le champ **Identity Provider Certificate** (Certificat du fournisseur d’identité) de HighGear.

6. Envoyez un e-mail à l’[équipe du support technique HighGear](mailto:support@highgear.com) pour demander votre certificat HighGear. Suivez les instructions indiquées dans sa réponse pour renseigner les champs **HighGear Certificate** (Certificat HighGear) et **HighGear Certificate Password** (Mot de passe du certificat HighGear).

7. Cliquez sur le bouton **Enregistrer** pour enregistrer la configuration de l’authentification unique HighGear.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon\@domainedevotreentreprise.extension** .  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à HighGear.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **HighGear** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **HighGear** .

    ![Lien HighGear dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-highgear-test-user"></a>Créer un utilisateur de test HighGear

Pour créer un utilisateur de test HighGear afin de tester la configuration de l’authentification unique, connectez-vous à votre système HighGear.

1. Cliquez sur le bouton **Create New Contact** .

    ![Bouton Create New Contact](media/highgear-tutorial/create-new-contact-button.png)

    Un menu s’affiche vous permettant de choisir le type de contact que vous souhaitez créer.

2. Cliquez sur l’élément de menu **Individual** pour créer un utilisateur HighGear.

    Dans le volet qui apparaît sur la droite, tapez les informations du nouvel utilisateur.  
    ![Formulaire de nouveau contact](media/highgear-tutorial/new-contact-form.png)

3. Dans le champ **Name** , tapez un nom pour le contact. Par exemple : Britta Simon

4. Cliquez sur le menu **More Options** et sélectionnez l’élément de menu **Account Info** .

    ![Clic sur l’élément de menu Account Info](media/highgear-tutorial/account-info-menu-item.png)

5. Définissez le champ **User Can Log In** (L’utilisateur peut se connecter) sur Yes.

    Le champ **Enable Single Sign-On** (Activer l’authentification unique) est aussi automatiquement défini sur Yes.

6. Dans le champ **Single Sign-On User Id** (ID d’utilisateur pour l’authentification unique), tapez l’identifiant de l’utilisateur. Par exemple : BrittaSimon@contoso.com

    La section Account Info doit maintenant ressembler à ceci :  
    ![Section Account Info terminée](media/highgear-tutorial/finished-account-info-section.png)

7. Pour enregistrer le contact, cliquez sur le bouton **Save** en bas du volet.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette HighGear dans le panneau d’accès doit vous connecter automatiquement à l’application HighGear pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)