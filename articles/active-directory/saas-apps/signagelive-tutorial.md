---
title: 'Tutoriel : Intégration d’Azure Active Directory à Signagelive | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Signagelive.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 1/11/2019
ms.author: jeedes
ms.openlocfilehash: 78324cfa58a8ac015b085052bdec7e3793befc1b
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348396"
---
# <a name="tutorial-azure-active-directory-integration-with-signagelive"></a>Tutoriel : Intégration d’Azure Active Directory à Signagelive

Au cours de ce tutoriel, vous allez apprendre à intégrer Signagelive à Azure Active Directory (Azure AD).
L’intégration de Signagelive dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Signagelive.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Signagelive (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md). Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Signagelive, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [essai d’un mois](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement à Signagelive pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Signagelive prend en charge l’authentification unique lancée par le fournisseur de services.

## <a name="add-signagelive-from-the-gallery"></a>Ajouter Signagelive à partir de la galerie

Pour configurer l’intégration de Signagelive avec Azure AD, ajoutez d’abord Signagelive à partir de la galerie à votre liste d’applications SaaS managées.

Pour ajouter Signagelive à partir de la galerie, procédez comme suit :

1. Dans le volet gauche du [portail Azure](https://portal.azure.com), sélectionnez l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Saisissez **Signagelive** dans la zone de recherche. 

     ![Signagelive dans la liste des résultats](common/search-new-app.png)

5. Sélectionnez **Signagelive** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD auprès de Signagelive avec un utilisateur de test nommé **Britta Simon**.
Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Signagelive associé.

Pour configurer et tester l’authentification unique Azure AD auprès de Signagelive, suivez d’abord les indications des modules suivants :

1. [Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on) pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. [Configurer l’authentification unique Signagelive](#configure-signagelive-single-sign-on) pour configurer les paramètres de l’authentification unique côté application.
3. [Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user) pour tester l’authentification unique Azure AD avec Britta Simon.
4. [Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user) pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. [Créer un utilisateur de test Signagelive](#create-a-signagelive-test-user) pour avoir un équivalent de Britta Simon dans Signagelive lié à la représentation Azure AD associée.
6. [Tester l’authentification unique](#test-single-sign-on) pour vérifier que la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD auprès de Signagelive, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Signagelive**, cliquez sur **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML** pour activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, procédez comme suit :

    ![Informations d’authentification unique dans Domaine et URL Signagelive](common/sp-signonurl.png)

    Dans la boîte **URL de connexion**, entrez une URL dont le modèle est le suivant : `https://login.signagelive.com/sso/<ORGANIZATIONALUNITNAME>`

    > [!NOTE]
    > Cette valeur n’est pas la valeur réelle. Mettez à jour la valeur avec l’URL de connexion réelle. Pour obtenir cette valeur, contactez [l’équipe de support client Signagelive](mailto:support@signagelive.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML**, à la section **Certificat de signature SAML**, sélectionnez **Télécharger** pour télécharger le **Certificat (brut)** en fonction des options définies. Enregistrez-le ensuite sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificateraw.png)

6. Dans la section **Configurer Signagelive**, copiez l’URL dont vous avez besoin.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-signagelive-single-sign-on"></a>Configurer l’authentification unique Signagelive

Pour configurer l’authentification unique côté Signagelive, envoyez le **certificat (brut)** téléchargé et les URL copiées depuis le portail Azure à [l’équipe du support technique Signagelive](mailto:support@signagelive.com). L’équipe vérifiera que l’authentification unique SAML est correctement configurée des deux côtés.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans la boîte de dialogue **Utilisateur**, procédez comme suit.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom**, entrez **BrittaSimon**.
  
    b. Dans le champ **Nom de l’utilisateur**, entrez « brittasimon@yourcompanydomain.extension ». Par exemple, ici, entrez « BrittaSimon@contoso.com ».

    c. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ Mot de passe.

    d. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Signagelive.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis sélectionnez **Signagelive**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Signagelive**.

    ![Lien Signagelive dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Sélectionnez le bouton **Ajouter un utilisateur**. Ensuite, dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Utilisateurs et groupes**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, dans la liste **Utilisateurs**, sélectionnez **Britta Simon**. Cliquez ensuite sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste. Cliquez ensuite sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-a-signagelive-test-user"></a>Créer un utilisateur de test Signagelive

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans Signagelive. Contactez l’[équipe du support technique Signagelive](mailto:support@signagelive.com) pour ajouter des utilisateurs dans la plateforme Signagelive. Vous devez créer et activer les utilisateurs avant d’utiliser l’authentification unique.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du portail MyApps.

Le fait de sélectionner la vignette **Signagelive** dans le portail MyApps doit vous connecter automatiquement à l’application. Pour plus d’informations sur le portail MyApps, consultez [Qu’est-ce que le portail MyApps ?](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)