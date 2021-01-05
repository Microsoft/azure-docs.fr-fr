---
title: 'Tutoriel : Intégration d’Azure Active Directory à Sprinklr | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Sprinklr.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/07/2019
ms.author: jeedes
ms.openlocfilehash: e76386c8c276c6c8c07751474b3718f98af18f4b
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92519450"
---
# <a name="tutorial-azure-active-directory-integration-with-sprinklr"></a>Tutoriel : Intégration d’Azure Active Directory à Sprinklr

Dans ce didacticiel, vous allez apprendre à intégrer Sprinklr à Azure Active Directory (Azure AD).
L’intégration de Sprinklr dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Sprinklr.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Sprinklr (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Sprinklr, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Sprinklr pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Sprinklr prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-sprinklr-from-the-gallery"></a>Ajout de Sprinklr à partir de la galerie

Pour configurer l’intégration de Sprinklr avec Azure AD, vous devez ajouter Sprinklr, disponible dans la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter Sprinklr à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Sprinklr** , sélectionnez **Sprinklr** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Sprinklr dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Sprinklr, avec un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Sprinklr associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Sprinklr, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Sprinklr](#configure-sprinklr-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Sprinklr](#create-sprinklr-test-user)** pour avoir un équivalent de Britta Simon dans Sprinklr lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Sprinklr, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Sprinklr** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Sprinklr](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<subdomain>.sprinklr.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<subdomain>.sprinklr.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique Sprinklr](https://www.sprinklr.com/contact-us/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer Sprinklr** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-sprinklr-single-sign-on"></a>Configurer l’authentification unique Sprinklr

1. Dans une autre fenêtre de navigateur web, connectez-vous au site de votre entreprise Sprinklr en tant qu’administrateur.

1. Accédez à **Administration \> Settings** .

    ![Administration](./media/sprinklr-tutorial/ic782907.png "Administration")

1. Accédez à **Manager Partner \> Single Sign** dans le volet gauche.

    ![Manage Partner](./media/sprinklr-tutorial/ic782908.png "Manage Partner")

1. Cliquez sur **+Add Single Sign Ons** .

    ![Capture d’écran montrant le bouton Add Single Sign Ons.](./media/sprinklr-tutorial/ic782909.png "Single Sign-Ons")

1. Dans la page **Single Sign on** , procédez comme suit :

    ![Capture d’écran montrant la page Single Sign on dans laquelle vous pouvez entrer les valeurs décrites.](./media/sprinklr-tutorial/ic782910.png "Single Sign-Ons")

    a. Dans la zone de texte **Name** (Nom), indiquez le nom de votre configuration (par exemple, *WAADSSOTest* ).

    b. Sélectionnez **Enabled** .

    c. Sélectionnez **Use new SSO Certificate** .

    d. Ouvrez votre certificat codé en base 64 dans le Bloc-notes, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **Identity provider certificate (Certificat du fournisseur d’identité)** .

    e. Dans la zone de texte **Entity Id** (ID d’entité), collez la valeur **Identificateur Azure AD** que vous avez copiée dans le portail Azure.

    f. Dans la zone de texte **Identity Provider Login URL** (URL de connexion du fournisseur d’identité), collez la valeur **URL de connexion** que vous avez copiée à partir du portail Azure.

    g. Dans la zone de texte **Identity Provider Logout URL** (URL de déconnexion du fournisseur d’identité), collez la valeur **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    h. Dans **SAML User ID Type** , sélectionnez **Assertion contains User’s sprinklr.com username** .

    i. Dans **SAML User ID Location** , sélectionnez **User ID is in the Name Identifier element of the Subject statement** .

    j. Cliquez sur **Enregistrer** .

    ![SAML](./media/sprinklr-tutorial/ic782911.png "SAML")

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon@yourcompanydomain.extension**  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Sprinklr.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Sprinklr** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Sprinklr** .

    ![Lien Sprinklr dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-sprinklr-test-user"></a>Créer un utilisateur de test Sprinklr

1. Connectez-vous à votre site d’entreprise Sprinklr en tant qu’administrateur.

1. Accédez à **Administration \> Settings** .

    ![Administration](./media/sprinklr-tutorial/ic782907.png "Administration")

1. Accédez à **Manage Client \> Users** dans le volet gauche.

    ![Capture d’écran montrant le bouton Add User dans Settings/Users.](./media/sprinklr-tutorial/ic782914.png "Paramètres")

1. Cliquez sur **Add User** .

    ![Capture d’écran montrant la boîte de dialogue Edit user dans laquelle vous pouvez entrer les valeurs décrites.](./media/sprinklr-tutorial/ic782915.png "Paramètres")

1. Dans la page **Edit User** , procédez comme suit :

    ![Modifier l’utilisateur](./media/sprinklr-tutorial/ic782916.png "Edit User")

    a. Dans les zones de texte **Email** , **First Name** et **Last Name** , saisissez les informations du compte utilisateur Azure AD que vous souhaitez approvisionner.

    b. Sélectionnez **Password Disabled** .

    c. Sélectionner une **Langue** .

    d. Sélectionnez un **Type d’utilisateur** .

    e. Cliquez sur **Update** .

    > [!IMPORTANT]
    > **Password Disabled** pour permettre à un utilisateur de se connecter par le biais d’un fournisseur d’identité. 

1. Accédez à **Role** , puis procédez comme suit :

    ![Rôles de partenaire](./media/sprinklr-tutorial/ic782917.png "Partner Roles")

    a. Dans la liste **Global** , sélectionnez **ALL_Permissions** .  

    b. Cliquez sur **Update** .

> [!NOTE]
> Vous pouvez utiliser n’importe quel outil ou API de création de compte utilisateur, fourni par Sprinklr, pour approvisionner des comptes utilisateur AAD.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Sprinklr dans le volet d’accès, vous devez être connecté automatiquement à l’application Sprinklr pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)