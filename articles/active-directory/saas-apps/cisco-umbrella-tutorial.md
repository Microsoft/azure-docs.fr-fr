---
title: 'Tutoriel : Intégration d’Azure Active Directory à Cisco Umbrella | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Cisco Umbrella.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/17/2019
ms.author: jeedes
ms.openlocfilehash: dde618b28e004e87edc2783bc44c5e7dd9f0ebba
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97670619"
---
# <a name="tutorial-azure-active-directory-integration-with-cisco-umbrella"></a>Tutoriel : Intégration d’Azure Active Directory à Cisco Umbrella

Ce tutoriel explique comment intégrer Cisco Umbrella avec Azure Active Directory (Azure AD).
L’intégration de Cisco Umbrella avec Azure AD offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Cisco Umbrella.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Cisco Umbrella (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD avec Cisco Umbrella, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement pour lequel l’authentification unique Cisco Umbrella est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Cisco Umbrella prend en charge l’authentification unique initiée par **SP et IDP**

## <a name="adding-cisco-umbrella-from-the-gallery"></a>Ajout de Cisco Umbrella à partir de la galerie

Pour configurer l’intégration de Cisco Umbrella à Azure AD, vous devez ajouter Cisco Umbrella, disponible dans la galerie, à votre liste d’applications SaaS managées.

**Pour ajouter Cisco Umbrella à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Cisco Umbrella**, sélectionnez **Cisco Umbrella** dans le panneau des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Cisco Umbrella dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec [Nom de l’application] sur un utilisateur de test nommé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur [Nom de l’application] associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec [Nom de l’application], vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Cisco Umbrella](#configure-cisco-umbrella-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Cisco Umbrella](#create-cisco-umbrella-test-user)** pour avoir un équivalent de Britta Simon dans Cisco Umbrella, lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec [Nom de l’application], procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Cisco Umbrella**, sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, l’utilisateur n’a rien à faire, car l’application est déjà intégrée à Azure.

    ![Informations d’authentification unique dans Domaine et URL Cisco Umbrella](common/both-preintegrated-signon.png)

    a. Si vous voulez configurer l’application en **mode lancé par le fournisseur de service**, procédez comme suit :

    b. Cliquez sur **Définir des URL supplémentaires**.

    c. Dans la zone de texte **URL d’authentification**, tapez l’URL `https://login.umbrella.com/sso`

5. Sur la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le fichier **XML des métadonnées** en fonction des options définies, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configurer Cisco Umbrella**, copiez la ou les URL appropriées correspondant à vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-cisco-umbrella-single-sign-on"></a>Configurer l'authentification unique Cisco Umbrella

1. Dans une autre fenêtre de navigateur Web, connectez-vous au site de votre entreprise Cisco Umbrella en tant qu’administrateur.

2. Sur le côté gauche du menu, cliquez sur **Administrateur**, accédez à **Authentification**, puis cliquez sur **SAML**.

    ![Administrateur](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_admin.png)

3. Sélectionnez **Autre**, puis cliquez sur **SUIVANT**.

    ![Autre](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_other.png)

4. Sur la page **Métadonnées Cisco Umbrella**, cliquez sur **SUIVANT**.

    ![Métadonnées](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_metadata.png)

5. Sous l'onglet **Charger les métadonnées**, si vous avez préconfiguré SAML, sélectionnez **Cliquer ici pour les modifier**, puis suivez les étapes ci-dessous.

    ![Ensuite](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_next.png)

6. Dans l'**Option A : Charger le fichier XML**, chargez le fichier **XML des métadonnées de fédération** que vous avez téléchargé à partir du portail Azure. Après avoir chargé les métadonnées, les valeurs ci-dessous sont automatiquement renseignées. Cliquez ensuite sur **SUIVANT**.

    ![Sélection du fichier](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_choosefile.png)

7. Sous la section **Valider la configuration SAML**, cliquez sur **TESTER VOTRE CONFIGURATION SAML**.

    ![Test](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_test.png)

8. Cliquez sur **ENREGISTRER**.

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Cisco Umbrella.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis sélectionnez **Cisco Umbrella**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, tapez et sélectionnez **Cisco Umbrella**.

    ![Lien Cisco Umbrella dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-cisco-umbrella-test-user"></a>Créer un utilisateur de test Cisco Umbrella

Pour permettre aux utilisateurs Azure AD de se connecter à Cisco Umbrella, vous devez les approvisionner dans Cisco Umbrella.  
En l’occurrence, cet approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Dans une autre fenêtre de navigateur Web, connectez-vous au site de votre entreprise Cisco Umbrella en tant qu’administrateur.

2. Sur le côté gauche du menu, cliquez sur **Administrateur** et accédez à **Comptes**.

    ![Compte](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_account.png)

3. Sur la page **Comptes**, cliquez sur **Ajouter**, à droite, et procédez comme suit.

    ![L’utilisateur](./media/cisco-umbrella-tutorial/tutorial_cisco-umbrella_createuser.png)

    a. Dans le champ **Prénom** , entrez le prénom **Britta**.

    b. Dans le champ **Nom de famille**, entrez le prénom **Simon**.

    c. À partir de **Choisir un rôle d'administrateur délégué**, sélectionnez votre rôle.

    d. Dans le champ **Adresse e-mail**, entrez l’adresse e-mail de l’utilisateur, par exemple **brittasimon\@contoso.com**.

    e. Dans le champ **Mot de passe**, entrez votre mot de passe.

    f. Dans le champ **Confirmer le mot de passe**, entrez à nouveau votre mot de passe.

    g. Cliquez sur **CREATE** (Créer).

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Cisco Umbrella dans le panneau d’accès doit vous connecter automatiquement à l’application Cisco Umbrella pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)