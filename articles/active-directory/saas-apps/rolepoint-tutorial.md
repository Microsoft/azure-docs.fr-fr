---
title: 'Didacticiel : Intégration d’Azure Active Directory avec RolePoint | Microsoft Docs'
description: Dans ce tutoriel, vous allez découvrir comment configurer l’authentification unique entre Azure Active Directory et RolePoint.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/15/2019
ms.author: jeedes
ms.openlocfilehash: 3225aa9eaff5c3cd0acca99261935feb9774810f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010263"
---
# <a name="tutorial-azure-active-directory-integration-with-rolepoint"></a>Didacticiel : Intégration d’Azure Active Directory avec RolePoint

Dans ce tutoriel, vous allez apprendre à intégrer RolePoint avec Azure Active Directory (Azure AD).
Cette intégration offre les avantages suivants :

* Vous pouvez utiliser Azure AD pour contrôler qui a accès à RolePoint.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à RolePoint (authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS à Azure AD, consultez l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à RolePoint, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement RolePoint pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* RolePoint prend en charge l’authentification unique lancée par le fournisseur de services

## <a name="add-rolepoint-from-the-gallery"></a>Ajouter RolePoint à partir de la galerie

Pour configurer l’intégration de RolePoint avec Azure AD, vous devez ajouter RolePoint, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Dans le [portail Azure](https://portal.azure.com), dans le volet de gauche, sélectionnez **Azure Active Directory** :

    ![Sélectionnez Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** > **Toutes les applications** :

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la fenêtre :

    ![Sélectionner la nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **RolePoint**. Dans les résultats de la recherche, sélectionnez **RolePoint**, puis **Ajouter**.

     ![Résultats de la recherche](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec RolePoint pour un utilisateur de test appelé Britta Simon.
Pour activer l’authentification unique, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur correspondant dans RolePoint.

Pour configurer et tester l’authentification unique Azure AD avec RolePoint, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser la fonctionnalité.
2. **[Configurer l’authentification unique RolePoint](#configure-rolepoint-single-sign-on)** côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour lui permettre d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test RolePoint](#create-a-rolepoint-test-user)** qui est lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** pour vérifier que la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous allez activer l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec RolePoint, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application RolePoint, sélectionnez **Authentification unique** :

    ![Sélectionner l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** pour activer l’authentification unique :

    ![Sélectionner une méthode d’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** :

    ![Icône Modifier](common/edit-urls.png)

4. Dans la boîte de dialogue **Configuration SAML de base**, effectuez les étapes suivantes.

    ![Boîte de dialogue Configuration SAML de base](common/sp-identifier.png)

    1. Dans la zone **URL de connexion**, entrez une URL au format suivant :

       `https://<subdomain>.rolepoint.com/login`

    1. Dans la zone **Identificateur (ID d’entité)** , entrez une URL au format suivant :

       `https://app.rolepoint.com/<instancename>`

    > [!NOTE]
    > Ces valeurs sont des espaces réservés. Vous devez utiliser l’URL de connexion et l’identificateur réels. Nous vous suggérons d’utiliser une valeur de chaîne unique dans l’identificateur. Pour obtenir ces valeurs, contactez l’[équipe de support de RolePoint](mailto:info@rolepoint.com). Vous pouvez aussi vous référer aux modèles figurant dans la boîte de dialogue **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, sélectionnez le lien **Télécharger** en regard de **XML de métadonnées de fédération**, en fonction de vos besoins, puis enregistrez le certificat sur votre ordinateur.

    ![Lien de téléchargement du certificat](common/metadataxml.png)

6. Dans la section **Configurer RolePoint**, copiez la ou les URL appropriées, en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    1. **URL de connexion**.

    1. **Identificateur Azure AD**.

    1. **URL de déconnexion**.


### <a name="configure-rolepoint-single-sign-on"></a>Configurer l’authentification unique RolePoint

Pour configurer l’authentification unique côté RolePoint, vous devez collaborer avec l’[équipe de support de RolePoint](mailto:info@rolepoint.com). Envoyez-lui le fichier XML de métadonnées de fédération et les URL que vous avez obtenues à partir du portail Azure. Elle configurera RolePoint pour garantir que l’authentification unique SAML est correctement configurée des deux côtés.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet de gauche du portail Azure, sélectionnez **Azure Active Directory**, **Utilisateurs**, puis **Tous les utilisateurs** :

    ![Sélectionner Tous les utilisateurs](common/users.png)

2. Sélectionnez **Nouvel utilisateur** en haut de la fenêtre :

    ![Sélectionner Nouvel utilisateur](common/new-user.png)

3. Dans la boîte de dialogue **Utilisateur**, procédez comme suit.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    1. Dans la zone **Nom**, entrez **BrittaSimon**.
  
    1. Dans la zone **Nom d’utilisateur**, entrez **BrittaSimon@\<yourcompanydomain>.\<extension>** . (Par exemple, BrittaSimon@contoso.com.)

    1. Sélectionnez **Afficher le mot de passe**, puis notez la valeur affichée dans la zone **Mot de passe**.

    1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à RolePoint.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **RolePoint**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **RolePoint**.

    ![Liste des applications](common/all-applications.png)

3. Dans le volet gauche, sélectionnez **Utilisateurs et groupes** :

    ![Sélectionner Utilisateurs et groupes](common/users-groups-blade.png)

4. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Sélectionnez Ajouter un utilisateur](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste des utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de la fenêtre.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste. Cliquez sur le bouton **Sélectionner** en bas de la fenêtre.

7. Dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Affecter**.

### <a name="create-a-rolepoint-test-user"></a>Créer un utilisateur de test RolePoint

Ensuite, vous devez créer un utilisateur nommé Britta Simon dans RolePoint. Rapprochez-vous de l’[équipe de support RolePoint](mailto:info@rolepoint.com) pour ajouter des utilisateurs à RolePoint. Les utilisateurs doivent d’abord être créés et activés pour que vous puissiez utiliser l’authentification unique.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Vous devez maintenant tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous sélectionnez la vignette RolePoint dans le volet d’accès, vous devez être automatiquement connecté à l’instance de RolePoint pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le volet d’accès, consultez [Accéder aux applications du portail Mes applications et les utiliser](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)