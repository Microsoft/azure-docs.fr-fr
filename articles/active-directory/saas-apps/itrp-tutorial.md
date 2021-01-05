---
title: 'Tutoriel : Intégration d’Azure Active Directory avec ITRP | Microsoft Docs'
description: Dans ce tutoriel, vous allez découvrir comment configurer l’authentification unique entre Azure Active Directory et ITRP.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: jeedes
ms.openlocfilehash: 2dff68f98e2922d5fc7a4fca1e6de8740bc2ae68
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "92459677"
---
# <a name="tutorial-azure-active-directory-integration-with-itrp"></a>Tutoriel : Intégration d’Azure Active Directory avec ITRP

Dans ce tutoriel, vous allez apprendre à intégrer ITRP à Azure Active Directory (Azure AD).
Cette intégration offre les avantages suivants :

* Vous pouvez utiliser Azure AD pour contrôler qui a accès à ITRP.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à ITRP (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS à Azure AD, consultez l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à ITRP, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement ITRP pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* ITRP prend en charge l’authentification unique lancée par le fournisseur de services.

## <a name="add-itrp-from-the-gallery"></a>Ajouter ITRP à partir de la galerie

Pour configurer l’intégration d’ITRP à Azure AD, vous devez ajouter ITRP à votre liste d’applications SaaS managées, à partir de la galerie.

1. Dans le [portail Azure](https://portal.azure.com), dans le volet de gauche, sélectionnez **Azure Active Directory** :

    ![Sélectionnez Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** > **Toutes les applications** :

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la fenêtre :

    ![Sélectionner la nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **ITRP**. Dans les résultats de la recherche, sélectionnez **ITRP**, puis **Ajouter**.

     ![Résultats de la recherche](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec ITRP pour un utilisateur de test appelé Britta Simon.
Pour activer l’authentification unique, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur correspondant dans ITRP.

Pour configurer et tester l’authentification unique Azure AD avec ITRP, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser la fonctionnalité.
2. **[Configurer l’authentification unique ITRP](#configure-itrp-single-sign-on)** côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour lui permettre d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test ITRP](#create-an-itrp-test-user)** qui est lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** pour vérifier que la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous allez activer l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec ITRP, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application ITRP, sélectionnez **Authentification unique** :

    ![Sélectionner l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** pour activer l’authentification unique :

    ![Sélectionner une méthode d’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** :

    ![Capture d’écran montrant la page Configurer l’authentification unique avec SAML, dans laquelle l’icône Modifier est sélectionnée.](common/edit-urls.png)

4. Dans la boîte de dialogue **Configuration SAML de base**, effectuez les étapes suivantes.

    ![Boîte de dialogue Configuration SAML de base](common/sp-identifier.png)

    1. Dans la zone **URL de connexion**, entrez une URL au format suivant :
    
       `https://<tenant-name>.itrp.com`

    1. Dans la zone **Identificateur (ID d’entité)** , entrez une URL au format suivant :

       `https://<tenant-name>.itrp.com`

    > [!NOTE]
    > Ces valeurs sont des espaces réservés. Vous devez utiliser l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe du support technique ITRP](https://www.4me.com/support/). Vous pouvez aussi vous référer aux modèles figurant dans la boîte de dialogue **Configuration SAML de base** dans le portail Azure.

5. Dans la section **Certificat de signature SAML**, sélectionnez le bouton **Modifier** pour ouvrir la boîte de dialogue **Certificat de signature SAML** :

    ![Capture d’écran montrant la page Certificat de signature SAML avec l’icône Modifier sélectionnée.](common/edit-certificate.png)

6. Dans la boîte de dialogue **Certificat de signature SAML**, copiez la valeur **Thumbprint** du certificat et enregistrez-la :

    ![Copiez la valeur Thumbprint](common/copy-thumbprint.png)

7. Dans la section **Configurer ITRP**, copiez la ou les URL appropriées, en fonction de vos besoins :

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    1. **URL de connexion**.

    1. **Identificateur Azure AD**.

    1. **URL de déconnexion**.

### <a name="configure-itrp-single-sign-on"></a>Configurer l’authentification unique ITRP

1. Dans une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise ITRP en tant qu’administrateur.

1. En haut de la fenêtre, sélectionnez l’icône **Settings** (Paramètres ) :

    ![Icône Paramètres](./media/itrp-tutorial/ic775570.png "Icône Paramètres")

1. Dans le volet gauche, sélectionnez **Single Sign-On** (Authentification unique) :

    ![Sélectionner l’authentification unique](./media/itrp-tutorial/ic775571.png "Sélectionner l’authentification unique")

1. Dans la section de configuration **Single Sign-On**, effectuez les étapes suivantes.

    ![Capture d’écran montrant la section Single Sign-On avec l’option Enabled sélectionnée.](./media/itrp-tutorial/ic775572.png "Section Single Sign-On")

    ![Capture d’écran montrant la section Single Sign-On dans laquelle vous pouvez ajouter les informations décrites à cette étape](./media/itrp-tutorial/ic775573.png "Section Single Sign-On").

    1. Sélectionnez **Enabled**.

    1. Dans la zone **URL de déconnexion distante**, collez la valeur de l’**URL de déconnexion** que vous avez copiée à partir du portail Azure.

    1. Dans la zone **URL SSO SAML**, collez la valeur de l’**URL de connexion** que vous avez copiée à partir du portail Azure.

    1. Dans la zone **Empreinte du certificat**, collez la valeur du certificat **Empreinte** que vous avez copiée à partir du portail Azure.

    1. Sélectionnez **Enregistrer**.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet de gauche du portail Azure, sélectionnez **Azure Active Directory**, **Utilisateurs**, puis **Tous les utilisateurs** :

    ![Sélectionner Tous les utilisateurs](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran :

    ![Sélectionner Nouvel utilisateur](common/new-user.png)

3. Dans la boîte de dialogue **Utilisateur**, procédez comme suit.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    1. Dans la zone **Nom**, entrez **BrittaSimon**.
  
    1. Dans la zone **Nom d’utilisateur**, entrez **BrittaSimon@\<yourcompanydomain>.\<extension>** . (Par exemple, BrittaSimon@contoso.com.)

    1. Sélectionnez **Afficher le mot de passe**, puis notez la valeur affichée dans la zone **Mot de passe**.

    1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à ITRP.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **ITRP**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **ITRP**.

    ![Liste des applications](common/all-applications.png)

3. Dans le volet gauche, sélectionnez **Utilisateurs et groupes** :

    ![Sélectionner Utilisateurs et groupes](common/users-groups-blade.png)

4. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Sélectionnez Ajouter un utilisateur](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste des utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de la fenêtre.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste. Cliquez sur le bouton **Sélectionner** en bas de la fenêtre.

7. Dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Affecter**.

### <a name="create-an-itrp-test-user"></a>Créer un utilisateur de test ITRP

Pour autoriser les utilisateurs Azure AD à se connecter à ITRP, vous devez les ajouter dans ITRP. Vous devez les ajouter manuellement.

Pour créer un compte d’utilisateur, procédez comme suit :

1. Connectez-vous à votre locataire ITRP.

1. En haut de la fenêtre, sélectionnez l’icône **Enregistrements** :

    ![Icône Enregistrements](./media/itrp-tutorial/ic775575.png "Icône Enregistrements")

1. Dans le menu, sélectionnez **Personnes** :

    ![Sélectionner des personnes](./media/itrp-tutorial/ic775587.png "Sélectionner des personnes")

1. Sélectionnez le signe plus ( **+** ) pour ajouter une nouvelle personne :

    ![Sélectionner le signe plus](./media/itrp-tutorial/ic775576.png "Sélectionner le signe plus")

1. Dans la boîte de dialogue **Ajouter une nouvelle personne**, effectuez les étapes suivantes.

    ![Boîte de dialogue Add New Person](./media/itrp-tutorial/ic775577.png "Boîte de dialogue Add New Person")

    1. Entrez l’adresse e-mail d’un compte Azure AD valide que vous souhaitez ajouter.

    1. Sélectionnez **Enregistrer**.

> [!NOTE]
> Vous pouvez utiliser n’importe quel API ou outil de création de compte d’utilisateur fourni par ITRP pour provisionner des comptes d’utilisateurs Azure AD.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Vous devez maintenant tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous sélectionnez la vignette ITRP dans le volet d’accès, vous devez être connecté automatiquement à l’instance ITRP pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le volet d’accès, consultez [Accéder aux applications du portail Mes applications et les utiliser](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)