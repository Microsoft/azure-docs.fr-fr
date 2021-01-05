---
title: 'Tutoriel : Intégration d’Azure Active Directory à InsideView | Microsoft Docs'
description: Dans ce tutoriel, vous allez découvrir comment configurer l’authentification unique entre Azure Active Directory et InsideView.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/20/2019
ms.author: jeedes
ms.openlocfilehash: b8f12b6d55cb11c0cfeda5331072127ea4a696bc
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92460102"
---
# <a name="tutorial-azure-active-directory-integration-with-insideview"></a>Tutoriel : Intégration d’Azure Active Directory à InsideView

Dans ce tutoriel, vous allez apprendre à intégrer InsideView à Azure Active Directory (Azure AD).
Cette intégration offre les avantages suivants :

* Vous pouvez utiliser Azure AD pour contrôler qui a accès à InsideView.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à InsideView (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS à Azure AD, consultez l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à InsideView, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement InsideView pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* InsideView prend en charge l’authentification unique lancée par le fournisseur d’identité.

## <a name="add-insideview-from-the-gallery"></a>Ajouter InsideView à partir de la galerie

Pour configurer l’intégration d’InsideView à Azure AD, vous devez ajouter InsideView, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Dans le [portail Azure](https://portal.azure.com), dans le volet de gauche, sélectionnez **Azure Active Directory**  :

    ![Sélectionnez Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** > **Toutes les applications**  :

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la fenêtre :

    ![Sélectionner la nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **InsideView** . Dans les résultats de la recherche, sélectionnez **InsideView** , puis **Ajouter** .

    ![Résultats de la recherche](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec InsideView pour un utilisateur de test appelé Britta Simon.
Pour activer l’authentification unique, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur correspondant dans InsideView.

Pour configurer et tester l’authentification unique Azure AD avec InsideView, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser la fonctionnalité.
2. **[Configurer l’authentification unique InsideView](#configure-insideview-single-sign-on)** côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour lui permettre d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test InsideView](#create-an-insideview-test-user)** qui est lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** pour vérifier que la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous allez activer l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec InsideView, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application InsideView, sélectionnez **Authentification unique**  :

    ![Sélectionner l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** pour activer l’authentification unique :

    ![Sélectionner une méthode d’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**  :

    ![Icône Modifier](common/edit-urls.png)

4. Dans la boîte de dialogue **Configuration SAML de base** , effectuez les étapes suivantes.

    ![Boîte de dialogue Configuration SAML de base](common/idp-reply.png)

    Dans la zone **URL de réponse** , entrez une URL au format suivant :

    `https://my.insideview.com/iv/<STS Name>/login.iv`

    > [!NOTE]
    > Cette valeur est un espace réservé. Vous devez utiliser l’URL de réponse réelle. Contactez l’[équipe de support technique d’InsideView](mailto:support@insideview.com) pour obtenir la valeur. Vous pouvez aussi vous référer aux modèles figurant dans la boîte de dialogue **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , sélectionnez le lien **Télécharger** en regard du **Certificat (brut)** correspondant à vos besoins, puis enregistrez le certificat sur votre ordinateur :

    ![Lien de téléchargement du certificat](common/certificateraw.png)

6. Dans la section **Configurer InsideView** , copiez la ou les URL appropriées, en fonction de vos besoins :

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    1. **URL de connexion** .

    1. **Identificateur Azure AD** .

    1. **URL de déconnexion** .

### <a name="configure-insideview-single-sign-on"></a>Configurer l’authentification unique InsideView

1. Dans une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise InsideView en tant qu’administrateur.

1. En haut de la fenêtre, sélectionnez **Admin** , **SingleSignOn Settings** (Paramètres d’authentification unique), puis **Add SAML** (Ajouter SAML).
   
   ![Paramètres d’authentification unique SAML](./media/insideview-tutorial/ic794135.png "Paramètres d’authentification unique SAML")

1. Dans la section **Add a New SAML** (Ajouter nouveau SAML), effectuez les étapes suivantes :

    ![Section Add a New SAML](./media/insideview-tutorial/ic794136.png "Section Add a New SAML")

    1. Dans la zone de texte **STS Name** , attribuez un nom à votre configuration.

    1. Dans la zone **SamlP/WS-Fed Unsolicited EndPoint** (Point de terminaison non sollicité SamlP/WS-Fed), collez la valeur **URL de connexion** que vous avez copiée à partir du portail Azure.

    1. Ouvrez le certificat brut que vous avez téléchargé à partir du portail Azure. Copiez le contenu du certificat dans le Presse-papiers, puis collez le contenu dans la zone **STS Certificate** .

    1. Dans la zone **Crm User Id Mapping** (Mappage d’ID utilisateur Crm), entrez **`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`** .

    1. Dans la zone **Crm Email Mapping** (Mappage d’e-mail Crm), entrez **`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`** .

    1. Dans la zone **Crm First Name Mapping** (Mappage du prénom Crm), entrez **`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`** .

    1. Dans la zone **Crm lastName Mapping** (Mappage du nom Crm), entrez **`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`** .  

    1. Sélectionnez **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet de gauche du portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs**  :

    ![Sélectionner Tous les utilisateurs](common/users.png)

2. Sélectionnez **Nouvel utilisateur** en haut de la fenêtre :

    ![Sélectionner Nouvel utilisateur](common/new-user.png)

3. Dans la boîte de dialogue **Utilisateur** , procédez comme suit.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    1. Dans la zone **Nom** , entrez **BrittaSimon** .
  
    1. Dans la zone **Nom d’utilisateur** , entrez **BrittaSimon@\<yourcompanydomain>.\<extension>** . (Par exemple, BrittaSimon@contoso.com.)

    1. Sélectionnez **Afficher le mot de passe** , puis notez la valeur affichée dans la zone **Mot de passe** .

    1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à InsideView.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **InsideView** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **InsideView** .

    ![Liste des applications](common/all-applications.png)

3. Dans le volet gauche, sélectionnez **Utilisateurs et groupes**  :

    ![Sélectionner Utilisateurs et groupes](common/users-groups-blade.png)

4. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Sélectionnez Ajouter un utilisateur](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste des utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de la fenêtre.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste. Cliquez sur le bouton **Sélectionner** en bas de la fenêtre.

7. Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Affecter** .

### <a name="create-an-insideview-test-user"></a>Créer un utilisateur de test InsideView

Pour autoriser les utilisateurs Azure AD à se connecter à InsideView, vous devez les ajouter dans InsideView. Vous devez les ajouter manuellement.

Pour créer des utilisateurs ou des contacts dans InsideView, contactez l’[équipe de support InsideView](mailto:support@insideview.com).

> [!NOTE]
> Vous pouvez utiliser n’importe quel API ou outil de création de compte d’utilisateur fourni par InsideView pour provisionner des comptes d’utilisateurs Azure AD.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Vous devez maintenant tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous sélectionnez la vignette InsideView dans le volet d’accès, vous devez être connecté automatiquement à l’instance InsideView pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le volet d’accès, consultez [Accéder aux applications du portail Mes applications et les utiliser](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)