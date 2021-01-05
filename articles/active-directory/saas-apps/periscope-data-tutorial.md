---
title: 'Tutoriel : Intégration d’Azure Active Directory à Periscope Data | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Periscope Data.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/21/2019
ms.author: jeedes
ms.openlocfilehash: deac4dcde51049d744706b7479f5b5f3bbb4d9c0
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92520854"
---
# <a name="tutorial-azure-active-directory-integration-with-periscope-data"></a>Tutoriel : Intégration d’Azure Active Directory à Periscope Data

Dans ce tutoriel, vous allez apprendre à intégrer Periscope Data à Azure Active Directory (Azure AD).
L’intégration de Periscope Data à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Periscope Data.
* Vous pouvez permettre à vos utilisateurs d’être automatiquement connectés à Periscope Data (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Periscope Data, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Periscope Data pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Periscope Data prend en charge l’authentification unique (SSO) initiée par le **fournisseur de services**

## <a name="adding-periscope-data-from-the-gallery"></a>Ajout de Periscope Data depuis la galerie

Pour configurer l’intégration de Periscope Data à Azure AD, vous devez ajouter Periscope Data, disponible dans la galerie, à votre liste d’applications SaaS managées.

**Pour ajouter Periscope Data à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Periscope Data** , sélectionnez **Periscope Data** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![PeriscopeData dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD auprès de Periscope Data, à l’aide d’un utilisateur de test nommé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Periscope Data associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD auprès de Periscope Data, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique de Periscope Data](#configure-periscope-data-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer l’utilisateur de test Periscope Data](#create-periscope-data-test-user)** pour avoir dans Periscope Data un équivalent de Britta Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD auprès de Periscope Data, effectuez les étapes suivantes :

1. Dans la page d’intégration de l’application **Periscope Data** sur le [portail Azure](https://portal.azure.com/), sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique relatives au domaine et aux URL Periscope Data](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion** , tapez une de ces URL :
    
    ```https
    https://app.periscopedata.com/
    https://app.periscopedata.com/app/<SITENAME>
    ```

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://app.periscopedata.com/<SITENAME>/sso`

    > [!NOTE]
    > La valeur d’URL de connexion n’est pas réelle. Mettez à jour les valeurs avec l’URL de connexion réelle. Contactez l’ [équipe du support technique de Periscope Data](mailto:support@periscopedata.com) pour obtenir cette valeur, ainsi que celle de l’identificateur que vous allez récupérer à la section **Configurer l’authentification unique de Periscope Data** , expliquée plus loin dans le tutoriel. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

### <a name="configure-periscope-data-single-sign-on"></a>Configurer l’authentification unique de Periscope Data

1. Dans une autre fenêtre de navigateur web, connectez-vous à Periscope Data en tant qu’administrateur.

2. Ouvrez le menu engrenage en bas à gauche, puis ouvrez le menu **Billing** (Facturation) > **Security** (Sécurité) et effectuez les étapes suivantes. Seuls les administrateurs ont accès à ces paramètres.

    ![Capture d’écran montrant la boîte de dialogue « Security » avec les paramètres sélectionnés.](./media/periscope-data-tutorial/configure01.png)

    a. Copiez l’ **URL des métadonnées de fédération de l’application** que vous avez récupérée à l’étape 5 **Certificat de signature SAML** et ouvrez-la dans un navigateur. Un document XML s’ouvre.

    b. Dans la zone de texte **Single Sign-On** (Authentification unique), sélectionnez **Azure Active Directory** .

    c. Recherchez la balise **SingleSignOnService** et collez la valeur de **Location** (Emplacement) dans la zone de texte **SSO URL** (URL SSO).

    d. Recherchez la balise **SingleLogoutService** et collez la valeur de **Location** (Emplacement) dans la zone de texte **SLO URL** (URL SLO).

    e. Copiez la valeur de **Identifier** (Identificateur) pour votre instance et collez-la dans la zone de texte **Identificateur (ID d’entité)** de la section **Configuration SAML de base** du portail Azure.

    f. Recherchez la première balise du fichier XML, copiez la valeur de **entityID** et collez-la dans la zone de texte **Issuer** (Émetteur).

    g. Recherchez la balise **IDPSSODescriptor** avec le protocole SAML. Dans cette section, recherchez la balise **KeyDescriptor** avec **use=signing** . Copiez la valeur de **X509Certificate** (CertificatX509) et collez-la dans la zone de texte **Certificate** (Certificat).

    h. Les sites dotés de plusieurs espaces peuvent choisir l’espace par défaut dans la liste déroulante **Default Space** (Espace par défaut). Cet espace est celui auquel les nouveaux utilisateurs sont ajoutés lorsqu’ils se connectent à Periscope Data pour la première fois et où ils sont provisionnés par le biais de l’authentification unique Active Directory.

    i. Pour terminer, cliquez sur **Save** (Enregistrer) et **confirmez** la modification apportée aux paramètres d’authentification unique en tapant **Logout** (Déconnexion).

    ![Capture d’écran montrant la boîte de dialogue « Update SSO Configuration » avec le mot « logout » tapé dans la zone de texte mise en évidence, et le bouton « Confirm » sélectionné.](./media/periscope-data-tutorial/configure02.png)

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en accordant l’accès à Periscope Data.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Periscope Data** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Periscope Data** .

    ![Lien Periscope Data dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-periscope-data-test-user"></a>Créer l’utilisateur de test Periscope Data

Pour permettre aux utilisateurs Azure AD de se connecter à Periscope Data, vous devez les provisionner dans Periscope Data. Dans Periscope Data, le provisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à Periscope Data en tant qu’administrateur.

2. Cliquez sur l’icône **Settings** (Paramètres) dans le bas du menu à gauche et accédez à **Permissions** (Autorisations).

    ![Capture d’écran montrant le menu « Settings » avec l’élément « Permissions » sélectionné.](./media/periscope-data-tutorial/configure03.png)

3. Cliquez sur **ADD USER** (Ajouter un utilisateur) et effectuez les étapes suivantes :

      ![Informations de configuration de Periscope Data](./media/periscope-data-tutorial/configure04.png)

    a. Dans la zone de texte **Prénom** , entrez le prénom de l’utilisateur, par exemple **Britta** .

    b. Dans la zone de texte **Nom** , entrez le nom de l’utilisateur, par exemple **Simon** .

    c. Dans la zone de texte **E-mail** , entrez l’adresse e-mail de l’utilisateur, par exemple **brittasimon\@contoso.com** .

    d. Cliquez sur **ADD** (Ajouter).

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Periscope Data dans le volet d’accès doit vous connecter automatiquement à l’application Periscope Data pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)