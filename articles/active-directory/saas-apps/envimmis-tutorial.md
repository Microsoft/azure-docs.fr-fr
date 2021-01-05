---
title: 'Didacticiel : intégration d’Azure Active Directory à Envi MMIS | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Envi MMIS.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/06/2019
ms.author: jeedes
ms.openlocfilehash: 6ccf755a73cafa4b855f602aa18246d710e5e1ff
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92454009"
---
# <a name="tutorial-azure-active-directory-integration-with-envi-mmis"></a>Didacticiel : intégration d’Azure Active Directory à Envi MMIS

Dans ce didacticiel, vous allez apprendre à intégrer Envi MMIS à Azure Active Directory (Azure AD).
L’intégration d’Envi MMIS à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Envi MMIS.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Dynamic Signal (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Envi MMIS, vous devez disposer des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Envi MMIS pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Envi MMIS prend en charge l’authentification unique lancée par le **fournisseur de services** et le **fournisseur d’identité**

## <a name="adding-envi-mmis-from-the-gallery"></a>Ajout d’Envi MMIS à partir de la galerie

Pour configurer l’intégration d’Envi MMIS à Azure AD, vous devez ajouter Envi MMIS à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Envi MMIS à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Envi MMIS** , sélectionnez **Envi MMIS** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Envi MMIS dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Envi MMIS et un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Envi MMIS associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Envi MMIS, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Envi MMIS](#configure-envi-mmis-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Envi MMIS](#create-envi-mmis-test-user)** pour avoir un équivalent de Britta Simon dans Envi MMIS, lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Envi MMIS, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Envi MMIS** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. À la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode initié par **IDP** , suivez les étapes ci-dessous :

    ![Capture d’écran montrant Configuration SAML de base avec les boutons « Identificateur », « URL de réponse » et « Enregistrer » mis en évidence.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://www.<CUSTOMER DOMAIN>.com/Account`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://www.<CUSTOMER DOMAIN>.com/Account/Acs`

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    ![Informations d’authentification unique dans la section Domaine et URL Envi MMIS](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://www.<CUSTOMER DOMAIN>.com/Account`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez [l’équipe du support technique d’Envi MMIS](mailto:support@ioscorp.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

6. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

7. Dans la section **Configurer Envi MMIS** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-envi-mmis-single-sign-on"></a>Configurer l’authentification unique Envi MMIS

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site Envi MMIS en tant qu’administrateur.

2. Cliquez sur l’onglet **My Domain** (Mon domaine).

    ![Capture d’écran montrant le menu « User » avec « My Domain » sélectionné.](./media/envimmis-tutorial/configure1.png)

3. Cliquez sur **Modifier** .

    ![Capture d’écran montrant le bouton « Edit » sélectionné.](./media/envimmis-tutorial/configure2.png)

4. Cochez la case **Use remote authentication** (Utiliser l’authentification distante), puis sélectionnez **HTTP Redirect** (Redirection HTTP) dans la liste déroulante **Authentication Type** (Type d’authentification).

    ![Capture d’écran montrant l’onglet « Details » avec l’option « Use remote authentication » activée et « HTTP Redirect » sélectionné.](./media/envimmis-tutorial/configure3.png)

5. Sélectionnez l’onglet **Resources** (Ressources), puis cliquez sur **Upload Metadata** (Charger les métadonnées).

    ![Capture d’écran montrant l’onglet « Resources » avec l’action « Upload Metadata » sélectionnée.](./media/envimmis-tutorial/configure4.png)

6. Dans la section **Upload Metadata** (Charger les métadonnées), procédez comme suit :

    ![Capture d’écran montrant la fenêtre contextuelle « Upload Metadata » avec l’option « File » sélectionnée et l’icône de choix de fichier et le bouton « OK » mis en évidence.](./media/envimmis-tutorial/configure5.png)

    a. Dans la liste déroulante **Upload From** (Charger à partir de), sélectionnez **File** (Fichier).

    b. Chargez le fichier de métadonnées téléchargé à partir du Portail Azure en sélectionnant **l’icône de choix de fichier** .

    c. Cliquez sur **OK** .

7. Une fois que vous avez chargé le fichier de métadonnées téléchargé, les champs sont automatiquement renseignés. Cliquez sur **Mettre à jour**

    ![Bouton Enregistrer de la page Configurer l’authentification unique](./media/envimmis-tutorial/configure6.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon\@domainedevotresociété.extension** .  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Envi MMIS.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Envi MMIS** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Envi MMIS** .

    ![Lien Envi MMIS dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-envi-mmis-test-user"></a>Créer un utilisateur de test Envi MMIS

Pour se connecter à Envi MMIS, les utilisateurs Azure AD doivent être provisionnés dans Envi MMIS. Dans le cas d’Envi MMIS, l’approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise Envi MMIS en tant qu’administrateur.

2. Cliquez sur l’onglet **User List** (Liste d’utilisateurs).

    ![Capture d’écran montrant le menu « User » avec « User List » sélectionné.](./media/envimmis-tutorial/user1.png)

3. Cliquez sur le bouton **Add User** (Ajouter un utilisateur).

    ![Capture d’écran montrant la section « Users » avec le bouton « Add User » sélectionné.](./media/envimmis-tutorial/user2.png)

4. Dans la section **Add User** , procédez comme suit :

    ![Ajouter un employé](./media/envimmis-tutorial/user3.png)

    a. Dans la zone de texte **User Name** (Nom d’utilisateur), tapez le nom d’utilisateur du compte de Britta Simon, comme **brittasimon\@contoso.com** .
    
    b. Dans la zone de texte **First Name** (Prénom), tapez le prénom de Britta Simon, tel que **Britta** .

    c. Dans la zone de texte **Last Name** (Nom de famille), tapez le nom de Britta Simon, par exemple **Simon** .

    d. Dans la zone de texte **Title** (Titre), entrez le titre de l’utilisateur.
    
    e. Dans la zone de texte **Email Address** (Adresse e-mail), tapez l’adresse e-mail du compte de Britta Simon, comme **brittasimon\@contoso.com** .

    f. Dans la zone de texte **SSO User Name** (Nom d’utilisateur SSO), tapez le nom d’utilisateur du compte de Britta Simon, comme **brittasimon\@contoso.com** .

    g. Cliquez sur **Enregistrer** .

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Envi MMIS dans le volet d’accès, vous devez être connecté automatiquement à l’application Envi MMIS pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)