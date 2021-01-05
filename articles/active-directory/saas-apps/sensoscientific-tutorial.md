---
title: 'Didacticiel : Intégration d’Azure Active Directory avec Système de surveillance de température sans fil SensoScientific | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Système de surveillance de température sans fil SensoScientific.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/10/2019
ms.author: jeedes
ms.openlocfilehash: 27512ac694f20544f1fdd5b79b27d7bf2cabdd66
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92675491"
---
# <a name="tutorial-azure-active-directory-integration-with-sensoscientific-wireless-temperature-monitoring-system"></a>Didacticiel : Intégration d’Azure Active Directory avec Système de surveillance de température sans fil SensoScientific

Dans ce didacticiel, vous allez découvrir comment intégrer Système de surveillance de température sans fil SensoScientific avec Azure Active Directory (Azure AD).
L’intégration de Système de surveillance de température sans fil SensoScientific avec Azure AD vous offre les avantages suivants :

* Vous pouvez contrôler dans Azure AD qui a accès à SensoScientific Wireless Temperature Monitoring System.
* Vous pouvez autoriser vos utilisateurs à se connecter automatiquement à SensoScientific Wireless Temperature Monitoring System (via l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec Système de surveillance de température sans fil SensoScientific, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Un abonnement à SensoScientific Wireless Temperature Monitoring System pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* SensoScientific Wireless Temperature Monitoring System prend en charge l’authentification unique initiée par le **fournisseur d’identité** .

## <a name="adding-sensoscientific-wireless-temperature-monitoring-system-from-the-gallery"></a>Ajout de Système de surveillance de température sans fil SensoScientific à partir de la galerie

Pour configurer l’intégration de Système de surveillance de température sans fil SensoScientific dans Azure AD, vous devez ajouter Système de surveillance de température sans fil SensoScientific à partir de la galerie dans votre liste d’applications SaaS gérées.

**Pour ajouter Système de surveillance de température sans fil SensoScientific à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **SensoScientific Wireless Temperature Monitoring System** , sélectionnez **SensoScientific Wireless Temperature Monitoring System** dans le volet de résultats, puis cliquez sur **Ajouter** pour ajouter l’application.

    ![SensoScientific Wireless Temperature Monitoring System dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec SensoScientific Wireless Temperature Monitoring System à l’aide d’un utilisateur de test appelé **Britta Simon** .
Pour l’authentification unique, une relation doit être établie entre un utilisateur Azure AD et un utilisateur SensoScientific Wireless Temperature Monitoring System associé.

Pour configurer et tester l’authentification unique Azure AD avec Système de surveillance de température sans fil SensoScientific, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique SensoScientific Wireless Temperature Monitoring System](#configure-sensoscientific-wireless-temperature-monitoring-system-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Wireless Temperature Monitoring System](#create-sensoscientific-wireless-temperature-monitoring-system-test-user)** pour disposer d’un équivalent de Britta Simon dans SensoScientific Wireless Temperature Monitoring System qui soit lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec SensoScientific Wireless Temperature Monitoring System, effectuez les étapes suivantes :

1. Sur le [portail Azure](https://portal.azure.com/), dans la page d’intégration d’application **SensoScientific Wireless Temperature Monitoring System** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , l’utilisateur n’a rien à faire, car l’application est déjà intégrée à Azure.

    ![Informations d’authentification unique du domaine et des URL SensoScientific Wireless Temperature Monitoring System](common/preintegrated.png)

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer SensoScientific Wireless Temperature Monitoring System** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-sensoscientific-wireless-temperature-monitoring-system-single-sign-on"></a>Configurer l’authentification unique pour SensoScientific Wireless Temperature Monitoring System

1. Connectez-vous à votre application Système de surveillance de température sans fil SensoScientific en tant qu’administrateur.

1. Dans le menu de navigation en haut, cliquez sur **Configuration** , accédez à **Configure** sous **Single Sign On** (Authentification unique) pour ouvrir les paramètres de l’authentification unique, puis effectuez les étapes suivantes :

    ![Configure Single Sign-On](./media/sensoscientific-tutorial/tutorial_sensoscientificwtms_admin.png)

    a. Sélectionnez **Nom de l’émetteur** comme Azure AD.

    b. Dans la zone de texte **Issuer URL** (URL de l’émetteur), collez l’ **identificateur Azure AD** que vous avez copié à partir du portail Azure.

    c. Dans la zone de texte **Single Sign-On Service URL** (URL du service de d’authentification unique), collez l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    d. Dans la zone de texte **Single Sign-Out Service URL** (URL du service de déconnexion unique), collez l’ **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    e. Accédez au certificat que vous avez téléchargé à partir du portail Azure et chargez-le ici.

    f. Cliquez sur **Enregistrer** .

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Système de surveillance de température sans fil SensoScientific.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **SensoScientific Wireless Temperature Monitoring System** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Système de surveillance de température sans fil SensoScientific** .

    ![Lien SensoScientific Wireless Temperature Monitoring System dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-sensoscientific-wireless-temperature-monitoring-system-test-user"></a>Créer un utilisateur de test SensoScientific Wireless Temperature Monitoring System

Pour que les utilisateurs d’Azure AD puissent se connecter à SensoScientific Wireless Temperature Monitoring System, ils doivent être provisionnés dans SensoScientific Wireless Temperature Monitoring System. Travaillez avec l’[équipe de support technique de Système de surveillance de température sans fil SensoScientific](https://www.sensoscientific.com/contact-us/) pour ajouter des utilisateurs dans la plateforme Système de surveillance de température sans fil SensoScientific. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la vignette SensoScientific Wireless Temperature Monitoring System dans le panneau d’accès, vous êtes automatiquement connecté à l’application SensoScientific Wireless Temperature Monitoring System. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)