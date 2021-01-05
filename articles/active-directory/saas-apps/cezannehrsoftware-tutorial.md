---
title: 'Didacticiel : Intégration d’Azure Active Directory avec Cezanne HR Software | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Cezanne HR Software.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/12/2019
ms.author: jeedes
ms.openlocfilehash: 0aa0dab7b512c85fbbdf374c962e6ee8e1c7d616
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92456275"
---
# <a name="tutorial-azure-active-directory-integration-with-cezanne-hr-software"></a>Didacticiel : Intégration d’Azure Active Directory avec Cezanne HR Software

Dans ce didacticiel, vous allez apprendre à intégrer Cezanne HR Software à Azure Active Directory (Azure AD).
L’intégration de Cezanne HR Software dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Cezanne HR Software.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Cezanne HR Software (via l’authentification unique) avec leurs comptes Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec Cezanne HR Software, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Cezanne HR Software pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Cezanne HR Software prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-cezanne-hr-software-from-the-gallery"></a>Ajout de Cezanne HR Software à partir de la galerie

Pour configurer l’intégration de Cezanne HR Software avec Azure AD, vous devez ajouter Cezanne HR Software, disponible dans la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter Cezanne HR Software à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Cezanne HR Software** , sélectionnez **Cezanne HR Software** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Cezanne HR Software dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Cezanne HR Software pour un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Cezanne HR Software associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Cezanne HR Software, vous avez besoin de suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Cezanne HR Software](#configure-cezanne-hr-software-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Cezanne HR Software](#create-cezanne-hr-software-test-user)** pour avoir un équivalent de Britta Simon dans Cezanne HR Software lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Cezanne HR Software, effectuez les étapes suivantes :

1. Dans le [Portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Cezanne HR Software** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Cezanne HR Software](common/sp-identifier-reply.png)

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://w3.cezanneondemand.com/CezanneOnDemand/-/<tenantidentifier>`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , tapez l’URL suivante : `https://w3.cezanneondemand.com/CezanneOnDemand/`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://w3.cezanneondemand.com:443/cezanneondemand/-/<tenantidentifier>/Saml/samlp`
    
    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de réponse et l’URL de connexion réelles. Pour obtenir ces valeurs, contactez [l’équipe du support Cezanne HR Software](https://cezannehr.com/services/support/).

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer Cezanne HR Software** , copiez les URL appropriées, selon vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-cezanne-hr-software-single-sign-on"></a>Configurer l’authentification unique Cezanne HR Software

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre client Cezanne HR Software en tant qu’administrateur.

2. Dans le volet de navigation gauche, cliquez sur **Configuration système** . Accédez aux **Paramètres de sécurité** . Accédez ensuite à **Configuration de l’authentification unique** .

    ![Capture d’écran montrant le locataire Cezanne HR avec Security Settings et Single Sign-On Configuration sélectionnés.](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_000.png)

3. Dans le panneau **Autoriser les utilisateurs à se connecter à l’aide de l’authentification unique** , cochez la case **SAML 2.0** et sélectionnez l’option **Configuration avancée** .

    ![Capture d’écran montrant le volet Allow users avec SAML 2.0 et Advanced Configuration sélectionnés.](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_001.png)

4. Cliquez sur le bouton **Ajouter nouveau** .

    ![Capture d’écran montrant le bouton Add New.](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_002.png)

5. Dans la section **SAML 2.0 IDENTITY PROVIDERS** , procédez comme suit.

    ![Capture d’écran montrant un volet dans lequel vous pouvez entrer les valeurs décrites à cette étape.](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_003.png)

    a. Entrez le nom de votre fournisseur d’identité en tant que **Nom d’affichage** .

    b. Dans la zone de texte **Identificateur (ID d’entité)** , collez la valeur de l’ **identificateur Azure AD** que vous avez copiée à partir du Portail Azure.

    c. Modifiez la **Liaison SAML** sur POST.

    d. Dans la zone de texte **Security Token Service Endpoint** (Point de terminaison de service d’émission de jeton de sécurité), collez la valeur de l’ **URL de connexion** que vous avez copiée à partir du Portail Azure.

    e. Dans la zone de texte User ID Attribute Name (Nom de l’attribut de l’identificateur d’utilisateur), entrez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`.

    f. Cliquez sur **Télécharger** pour charger le certificat obtenu auprès du portail Azure.

    g. Cliquez sur le bouton **OK** .

6. Cliquez sur le bouton **Enregistrer** .

    ![Capture d’écran montrant le bouton Save pour Single Sign-on Configuration.](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_004.png)

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Cezanne HR Software.

1. Dans le Portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Cezanne HR Software** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Cezanne HR Software** .

    ![Lien Cezanne HR Software dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-cezanne-hr-software-test-user"></a>Créer un utilisateur de test Cezanne HR Software

Pour permettre aux utilisateurs Azure AD de se connecter à Cezanne HR Software, vous devez les approvisionner dans Cezanne HR Software. Dans le cas de Cezanne HR Software, cet approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise Cezanne HR Software en tant qu’administrateur.

2. Dans le volet de navigation gauche, cliquez sur **Configuration système** . Accédez à **Gérer les utilisateurs** . Accédez ensuite à **Ajouter un nouvel utilisateur** .

    ![Capture d’écran montrant le locataire Cezanne HR Software avec Manage Users et Add New User sélectionnés.](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_005.png "Nouvel utilisateur")

3. Dans la section **DÉTAILS DE LA PERSONNE** , effectuez les étapes suivantes :

    ![Capture d’écran montrant la section PERSON DETAILS, dans laquelle vous pouvez entrer les valeurs décrites à cette étape.](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_006.png "Nouvel utilisateur")

    a. Désactivez l’option **Utilisateur interne** .

    b. Dans la zone de texte **First Name** (Prénom), tapez le prénom de l’utilisateur, par exemple **Britta** .  

    c. Dans la zone de texte **Last Name** (Nom de famille), tapez le nom de l’utilisateur, par exemple **Simon** .

    d. Dans la zone de texte **Email** (E-mail), tapez l’adresse e-mail d’un utilisateur, par exemple, Brittasimon@contoso.com.

4. À la section **Informations sur le compte** , procédez comme suit :

    ![Capture d’écran montrant ACCOUNT INFORMATION, où vous pouvez entrer les valeurs décrites à cette étape.](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_007.png "Nouvel utilisateur")

    a. Dans la zone de texte **Username** (Nom d’utilisateur), tapez l’e-mail d’un utilisateur, par exemple, Brittasimon@contoso.com.

    b. Dans la zone de texte **Password** (Mot de passe), tapez le mot de passe de l’utilisateur.

    c. Sélectionnez **Professionnel des RH** en tant que **Rôle de sécurité** .

    d. Cliquez sur **OK** .

5. Accédez à l’onglet **Authentification unique** et sélectionnez **Ajouter nouveau** dans la zone **Identificateurs SAML 2.0** .

    ![Capture d’écran montrant l’onglet Single Sign-On, où vous pouvez sélectionner Add New.](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_008.png "Utilisateur")

6. Choisissez votre **fournisseur d’identité** et dans la zone de texte **Identificateur d’utilisateur** , entrez l’adresse de messagerie du compte de Britta Simon.

    ![Capture d’écran montrant les identificateurs SAML 2.0, où vous pouvez sélectionner votre fournisseur d’identité et votre identificateur d’utilisateur.](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_009.png "Utilisateur")

7. Cliquez sur le bouton **Enregistrer** .

    ![Capture d’écran montrant le bouton Save pour User Settings.](./media/cezannehrsoftware-tutorial/tutorial_cezannehrsoftware_010.png "Utilisateur")

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Cezanne HR Software dans le volet d’accès, vous devez être automatiquement connecté à l’application Cezanne HR Software pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)