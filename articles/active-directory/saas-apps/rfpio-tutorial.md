---
title: 'Didacticiel : Intégration d’Azure Active Directory à RFPIO | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et RFPIO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/14/2019
ms.author: jeedes
ms.openlocfilehash: c4e838afa867a7fb1e7fa8f582bc8879c24056a9
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92506052"
---
# <a name="tutorial-azure-active-directory-integration-with-rfpio"></a>Didacticiel : Intégration d’Azure Active Directory à RFPIO

Dans ce didacticiel, vous allez apprendre à intégrer RFPIO à Azure Active Directory (Azure AD).
L’intégration de RFPIO dans Azure AD vous offre les avantages suivants :

* Dans Azure Active Directory, vous pouvez contrôler qui a accès à RFPIO.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à RFPIO (par le biais de l’authentification unique) avec leur compte Azure Active Directory.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec RFPIO, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Abonnement RFPIO pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* RFPIO prend en charge l’authentification unique lancée par le **fournisseur de services et le point de distribution d’émission**

## <a name="adding-rfpio-from-the-gallery"></a>Ajout de RFPIO depuis la galerie

Pour configurer l’intégration de RFPIO avec Azure AD, vous devez ajouter RFPIO à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter RFPIO depuis la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **RFPIO** , sélectionnez **RFPIO** dans le panneau des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![RFPIO dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure Active Directory avec RFPIO, avec un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure Active Directory et l’utilisateur RFPIO associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec RFPIO, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique RFPIO](#configure-rfpio-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test RFPIO](#create-rfpio-test-user)** pour avoir un équivalent de Britta Simon dans RFPIO qui soit lié à la représentation Azure Active Directory de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure Active Directory avec RFPIO, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), sur la page d’intégration de l’application **RFPIO** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , effectuez l’étape suivante :

    ![Capture d’écran montrant la section Configuration SAML de base, dans laquelle vous pouvez entrer un identificateur.](common/idp-identifier.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://www.rfpio.com`

    b. Cliquez sur **Définir des URL supplémentaires** .

    c. Dans la zone de texte **État de relais** , entrez une valeur de chaîne. Contactez l’[équipe de support technique de RFPIO](https://www.rfpio.com/contact/) pour obtenir cette valeur.

    ![Capture d’écran montrant Définir des URL supplémentaires.](common/idp-preintegrated-relay.png)

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    ![image](common/both-preintegrated-signon.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://www.app.rfpio.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de connexion actuels. Pour obtenir ces valeurs, contactez l’[équipe de support technique de RFPIO](https://www.rfpio.com/contact/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

6. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

7. Dans la section **Configurer RFPIO** , copiez l’URL ou les URL appropriées en fonction de vos exigences.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-rfpio-single-sign-on"></a>Configurer l’authentification unique RFPIO

1. Dans une autre fenêtre de navigateur web, connectez-vous au site web **RFPIO** en tant qu’administrateur.

1. Cliquez sur la liste déroulante située en bas à gauche.

    ![Capture d’écran montrant la flèche Bas située en bas du volet.](./media/rfpio-tutorial/app1.png)

1. Cliquez sur **Paramètres de l’organisation** . 

    ![Capture d’écran montrant la sélection de Organization Settings.](./media/rfpio-tutorial/app2.png)

1. Cliquez sur **FONCTIONNALITÉS ET INTÉGRATION** .

    ![Capture d’écran montrant la sélection de Features and Integration dans Settings.](./media/rfpio-tutorial/app4.png)

1. Dans **Configuration SAML SSO** , cliquez sur **Modifier** .

    ![Capture d’écran affichant la section SAML SSO Configuration avec le bouton Edit mis en évidence.](./media/rfpio-tutorial/app3.png)

1. Dans cette section, procédez comme suit :

    ![Capture d’écran affichant la section SAML SSO Configuration avec l’option SAML activée.](./media/rfpio-tutorial/app5.png)
    
    a. Copiez le contenu du **fichier XML de métadonnées téléchargé** et collez-le dans le champ **Configuration de l’identité** .

    > [!NOTE]
    >Pour copier le contenu du fichier **XML des métadonnées de fédération** téléchargé, utilisez **Notepad++** ou **XML Editor** .

    b. Cliquez sur **Valider** .

    c. Après avoir cliqué sur **Valider** , activez **SAML(Enabled)** .

    d. Cliquez sur **Envoyer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez `brittasimon@yourcompanydomain.extension`. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à RFPIO.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **RFPIO** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **RFPIO** .

    ![Lien RFPIO dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-rfpio-test-user"></a>Créer un utilisateur de test RFPIO

1. Connectez-vous à votre site d’entreprise RFPIO en tant qu’administrateur.

1. Cliquez sur la liste déroulante située en bas à gauche.

    ![Capture d’écran montrant la flèche Bas située en bas du volet.](./media/rfpio-tutorial/app1.png)

1. Cliquez sur **Paramètres de l’organisation** . 

    ![Capture d’écran montrant la sélection de Organization Settings.](./media/rfpio-tutorial/app2.png)

1. Cliquez sur **MEMBRES DE L’ÉQUIPE** .

    ![Capture d’écran affichant l’élément Team Members sélectionné dans Settings.](./media/rfpio-tutorial/app6.png)

1. Cliquez sur **AJOUTER DES MEMBRES** .

    ![Capture d’écran montrant le bouton Add Members.](./media/rfpio-tutorial/app7.png)

1. Dans la section **Ajouter de nouveaux membres** . Procédez comme suit :

    ![Capture d’écran montrant l’ajout de nouveaux membres, où vous pouvez indiquer les valeurs décrites.](./media/rfpio-tutorial/app8.png)

    a. Entrez l’ **adresse e-mail** dans le champ **Entrer une adresse e-mail par ligne** .

    b. Veuillez sélectionner le **Rôle** selon vos exigences.

    c. Cliquez sur **AJOUTER DES MEMBRES** .

    > [!NOTE]
    > Le titulaire du compte Azure Active Directory reçoit un e-mail contenant un lien à suivre pour confirmer son compte et l’activer.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette RFPIO dans le volet d’accès doit vous connecter automatiquement à l’application RFPIO pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)