---
title: 'Tutoriel : Intégration d’Azure Active Directory à Clarizen | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Clarizen.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/21/2019
ms.author: jeedes
ms.openlocfilehash: 38b2ff6909dae15ff0f836316d5d12140ecc331a
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97672936"
---
# <a name="tutorial-azure-active-directory-integration-with-clarizen"></a>Tutoriel : Intégration d’Azure Active Directory à Clarizen

Dans ce didacticiel, vous allez apprendre à intégrer Clarizen à Azure Active Directory (Azure AD).
L’intégration de Clarizen dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Clarizen.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Clarizen (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Clarizen, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Clarizen pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Clarizen prend en charge l’authentification unique lancée par le **fournisseur d’identité**

## <a name="adding-clarizen-from-the-gallery"></a>Ajout de Clarizen à partir de la galerie

Pour configurer l’intégration de Clarizen à Azure AD, vous devez ajouter Clarizen à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Clarizen à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Clarizen**, sélectionnez **Clarizen** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Clarizen dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD auprès de Clarizen sur un utilisateur de test nommé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Clarizen associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Clarizen, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Clarizen](#configure-clarizen-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Clarizen](#create-clarizen-test-user)** pour avoir un équivalent de Britta Simon dans Clarizen, associé à sa représentation dans Azure AD.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD auprès de Clarizen, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Clarizen**, sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Sur la page **Configurer l’authentification unique avec SAML**, effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Clarizen](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur**, tapez une valeur : `Clarizen`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://.clarizen.com/Clarizen/Pages/Integrations/SAML/SamlResponse.aspx`

    > [!NOTE]
    > Il ne s’agit pas des valeurs réelles. Vous devrez utiliser les véritables valeurs d’identificateur et d’URL de réponse. Nous vous suggérons d’utiliser ici la valeur unique d’une chaîne en guise d’identificateur. Pour obtenir les valeurs réelles, contactez [l’équipe de support technique Clarizen](https://success.clarizen.com/hc/en-us/requests/new).

4. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer Clarizen**, copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-clarizen-single-sign-on"></a>Configurer l’authentification unique Clarizen

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise Clarizen en tant qu’administrateur.

1. Cliquez sur votre nom d’utilisateur, puis sur **Settings** (Paramètres).

    ![Sélection de l’option « Settings » (Paramètres) sous votre nom d’utilisateur](./media/clarizen-tutorial/tutorial_clarizen_001.png "Paramètres")

1. Cliquez sur l’onglet **Global Settings** (Paramètres globaux). En regard de la zone **Federated Authentication** (Authentification fédérée), cliquez sur **edit** (modifier).

    ![Onglet « Global Settings »](./media/clarizen-tutorial/tutorial_clarizen_002.png "Paramètres globaux")

1. Dans la boîte de dialogue **Federated Authentication** (Authentification fédérée), procédez comme suit :

    ![Boîte de dialogue « Federated Authentication »](./media/clarizen-tutorial/tutorial_clarizen_003.png "Federated Authentication")

    a. Sélectionnez **Activer l'authentification fédérée**.

    b. Cliquez sur **Upload** pour charger votre certificat téléchargé.

    c. Dans la zone **URL de connexion**, entrez la valeur de l’**URL de connexion** indiquée dans la fenêtre de configuration de l’application Azure AD.

    d. Dans la zone **URL de déconnexion**, entrez la valeur de l’**URL de déconnexion** indiquée dans la fenêtre de configuration de l’application Azure AD.

    e. Sélectionnez **Use POST**.

    f. Cliquez sur **Enregistrer**.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom**, entrez **BrittaSimon**.

    b. Dans le champ **Nom d’utilisateur**, tapez **brittasimon\@domainedevotreentreprise.extension**.  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Clarizen.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **Clarizen**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Clarizen**.

    ![Lien Clarizen dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-clarizen-test-user"></a>Créer un utilisateur de test Clarizen

L'objectif de cette section est de créer un utilisateur appelé Britta Simon dans Clarizen.

**Si vous avez besoin de créer un utilisateur manuellement, procédez comme suit :**

Pour permettre aux utilisateurs Azure AD de se connecter à Clarizen, vous devez approvisionner des comptes d’utilisateurs. Dans le cas de Clarizen, l’approvisionnement est une tâche manuelle.

1. Connectez-vous à votre site d’entreprise Clarizen en tant qu’administrateur.

2. Cliquez sur **People**.

    ![Clic sur « People »](./media/clarizen-tutorial/create_aaduser_001.png "Personnes")

3. Cliquez sur **Invite User**.

    ![Bouton « Invite User »](./media/clarizen-tutorial/create_aaduser_002.png "Inviter des utilisateurs")

1. Dans la boîte de dialogue **Invite People** (Inviter un contact), procédez comme suit :

    ![Boîte de dialogue « Invite People »](./media/clarizen-tutorial/create_aaduser_003.png "Inviter des personnes")

    a. Dans la zone **Email** (E-mail), tapez l’adresse e-mail du compte de Britta Simon.

    b. Cliquez sur **Invite**.

    > [!NOTE]
    > Le titulaire du compte Azure Active Directory reçoit un message électronique contenant un lien à suivre pour confirmer son compte et l’activer.


### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Clarizen dans le volet d’accès, vous devez vous connecter automatiquement à l’application Clarizen pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)