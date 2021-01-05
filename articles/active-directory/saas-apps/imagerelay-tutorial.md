---
title: 'Didacticiel : Intégration d’Azure Active Directory à Image Relay | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Image Relay.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/20/2019
ms.author: jeedes
ms.openlocfilehash: b76c2e346adb6c2afd146b0d73c8f20165145bac
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92460357"
---
# <a name="tutorial-azure-active-directory-integration-with-image-relay"></a>Didacticiel : Intégration d’Azure Active Directory à ImageRelay

Dans ce didacticiel, vous allez apprendre à intégrer Image Relay avec Azure Active Directory (Azure AD).
L’intégration d’Image Relay à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Image Relay.
* Vous pouvez permettre à vos utilisateurs d’être automatiquement connectés à Image Relay (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec Image Relay, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Image Relay pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Image Relay prend en charge l’authentification unique initiée par le **fournisseur de services**

## <a name="adding-image-relay-from-the-gallery"></a>Ajout d’Image Relay à partir de la galerie

Pour configurer l’intégration d’Image Relay avec Azure AD, vous devez ajouter Image Relay à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Image Relay à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Image Relay** , sélectionnez **Image Relay** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Image Relay dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD auprès d’Image Relay, à l’aide d’un utilisateur de test nommé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur Image Relay associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Image Relay, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique d’Image Relay](#configure-image-relay-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer l’utilisateur de test Image Relay](#create-image-relay-test-user)** pour avoir dans Image Relay un équivalent de Britta Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD auprès d’Image Relay, effectuez les étapes suivantes :

1. Dans la page d’intégration de l’application [Image Relay](https://portal.azure.com/) sur le **portail Azure** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique relatives au domaine et aux URL Image Relay](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<companyname>.imagerelay.com/`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<companyname>.imagerelay.com/sso/metadata`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique d’Image Relay](http://support.imagerelay.com/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

4. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer Image Relay** , copiez l’URL ou les URL appropriées, en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-image-relay-single-sign-on"></a>Configurer l’authentification unique d’Image Relay

1. Dans une autre fenêtre de navigateur, connectez-vous à votre site d’entreprise Image Relay en tant qu’administrateur.

2. Dans la barre d’outils du haut, cliquez sur la charge de travail **Utilisateurs et autorisations** .

    ![Capture d’écran montrant Users & Permissions sélectionné à partir de la barre d’outils.](./media/imagerelay-tutorial/tutorial_imagerelay_06.png) 

3. Cliquez sur **Créer une nouvelle autorisation** .

    ![Capture d’écran montrant une zone de texte pour entrer le titre de l’autorisation et une option pour choisir le type d’autorisation.](./media/imagerelay-tutorial/tutorial_imagerelay_08.png)

4. Dans la charge de travail **Paramètres d’authentification unique** , sélectionnez la case à cocher **Ce groupe peut se connecter uniquement via l’authentification unique** , puis cliquez sur **Enregistrer** .

    ![Capture d’écran montrant Single Sign On Settings, où vous pouvez sélectionner l’option.](./media/imagerelay-tutorial/tutorial_imagerelay_09.png) 

5. Accédez à **Paramètres du compte** .

    ![Capture d’écran montrant l’option de la barre d’outils Account Settings.](./media/imagerelay-tutorial/tutorial_imagerelay_10.png) 

6. Accédez à la charge de travail **Paramètres d’authentification unique** .

    ![Capture d’écran montrant l’option de menu Single Sign On Settings.](./media/imagerelay-tutorial/tutorial_imagerelay_11.png)

7. Dans la boîte de dialogue **SAML Settings** (Paramètres SAML), procédez comme suit :

    ![Capture d’écran montrant la boîte de dialogue SAML Settings où vous pouvez entrer les informations.](./media/imagerelay-tutorial/tutorial_imagerelay_12.png)

    a. Dans la zone de texte **Login URL** (URL de connexion), collez la valeur **URL de connexion** que vous avez copiée dans le Portail Azure.

    b. Dans la zone de texte **Logout URL** (URL de déconnexion), collez la valeur de l’ **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    c. Sous **Name Id Format** (Format d’ID de nom), sélectionnez **urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress** .

    d. Sous **Binding Options for Requests from the Service Provider (Image Relay)** (Options de liaison pour les demandes provenant du fournisseur de services (ImageRelay)), sélectionnez **POST Binding** (Liaison POST).

    e. Sous **Certificat x.509** , cliquez sur **Mettre à jour le certificat** .

    ![Capture d’écran montrant l’option de mise à jour du certificat.](./media/imagerelay-tutorial/tutorial_imagerelay_17.png)

    f. Ouvrez le certificat que vous avez téléchargé dans le Bloc-notes, copiez son contenu, puis collez-le dans la zone de texte **Certificat x.509** .

    ![Capture d’écran montrant le certificat x.509.](./media/imagerelay-tutorial/tutorial_imagerelay_18.png)

    g. Dans la section **Approvisionnement juste à temps des utilisateurs** , sélectionnez la case à cocher **Activer l’approvisionnement juste à temps des utilisateurs** .

    ![Capture d’écran montrant la section Just-In-Time User Provisioning avec Enable control sélectionné.](./media/imagerelay-tutorial/tutorial_imagerelay_19.png)

    h. Sélectionnez le groupe d’autorisations (par exemple, **SSO de base** ) qui sera autorisé à se connecter uniquement via l’authentification unique.

    ![Capture d’écran montrant la section Just-In-Time User Provisioning avec SSO Basic sélectionné.](./media/imagerelay-tutorial/tutorial_imagerelay_20.png)

    i. Cliquez sur **Enregistrer** .

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Image Relay.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Image Relay** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Image Relay** .

    ![Lien Image Relay dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-image-relay-test-user"></a>Créer l’utilisateur de test Image Relay

L’objectif de cette section est de créer un utilisateur appelé Britta Simon dans Image Relay.

**Pour créer un utilisateur appelé Britta Simon dans Image Relay, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise Image Relay tant qu’administrateur.

2. Accédez à **Utilisateurs et autorisations** et sélectionnez **Créer un utilisateur SSO** .

    ![Capture d’écran montrant Create SSO User sélectionné dans le menu.](./media/imagerelay-tutorial/tutorial_imagerelay_21.png) 

3. Renseignez les champs **E-mail** , **Prénom** , **Nom** et **Société** de l’utilisateur que vous souhaitez configurer, puis sélectionnez le groupe d’autorisations (par exemple authentification unique de base), qui peut se connecter uniquement via l’authentification unique.

    ![Capture d’écran montrant la page Create a SSO User où vous pouvez entrer les informations requises.](./media/imagerelay-tutorial/tutorial_imagerelay_22.png)

4. Cliquez sur **Créer** .

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Image Relay dans le volet d’accès doit vous connecter automatiquement à l’application Image Relay pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)