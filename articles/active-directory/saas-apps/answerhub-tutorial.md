---
title: 'Didacticiel : Intégration d’Azure Active Directory à AnswerHub | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et AnswerHub.
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
ms.openlocfilehash: 284ec65955de60f41d7bb0f97bf608faba3d4b79
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97672584"
---
# <a name="tutorial-azure-active-directory-integration-with-answerhub"></a>Didacticiel : Intégration d’Azure Active Directory à AnswerHub

Dans ce didacticiel, vous allez apprendre à intégrer AnswerHub à Azure Active Directory (Azure AD).
L’intégration d’AnswerHub à Azure AD offre ces avantages :

* Vous pouvez utiliser Azure AD pour contrôler qui a accès à AnswerHub.
* Vous pouvez autoriser les utilisateurs à se connecter automatiquement à AnswerHub avec leur compte Azure AD (par le biais de l’authentification unique).
* Vous pouvez gérer vos comptes à partir d’un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS à Azure AD, consultez l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à AnswerHub, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez commencer un [essai gratuit d’un mois](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement AnswerHub pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* AnswerHub prend en charge l’authentification unique lancée par le fournisseur de services.

## <a name="add-answerhub-from-the-gallery"></a>Ajouter AnswerHub à partir de la galerie

Pour configurer l’intégration d’AnswerHub à Azure AD, vous devez ajouter AnswerHub depuis la galerie à vos applications SaaS managées.

**Pour ajouter AnswerHub à partir de la galerie :**

1. Dans le volet gauche du [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la fenêtre.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **AnswerHub**. Sélectionnez **AnswerHub** dans la liste des résultats, puis sélectionnez **Ajouter**.

    ![AnswerHub dans la liste des résultats](common/search-new-app.png)

## <a name="set-up-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec AnswerHub pour un utilisateur de test appelé Britta Simon.
Pour l’authentification unique, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur correspondant dans AnswerHub.

Pour configurer et tester l’authentification unique Azure AD avec AnswerHub, effectuez les étapes suivantes :

1. [Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on) pour permettre à vos utilisateurs d’utiliser la fonctionnalité.
2. [Configurer l’authentification unique AnswerHub](#configure-answerhub-single-sign-on) pour configurer les paramètres de l’authentification unique côté application.
3. [Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user) appelé Britta Simon.
4. [Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user) pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. Créer un utilisateur de test AnswerHub qui correspond et est lié à l’utilisateur de test Azure AD.
6. [Tester l’authentification unique](#test-single-sign-on) pour vérifier que la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous allez configurer l’authentification unique Azure AD sur le Portail Azure.

**Pour configurer l’authentification unique Azure AD avec AnswerHub :**

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **AnswerHub**, sélectionnez **Authentification unique**.

    ![Bouton Authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** pour activer l’authentification unique.

    ![Boîte de dialogue Sélectionner une méthode d’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez l’icône Modifier pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Page Configurer l’authentification unique avec SAML](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, effectuez les étapes suivantes :

    ![Section Configuration SAML de base](common/sp-identifier.png)

    a. Dans la zone **Sign on URL** (URL de connexion), entrez une URL au format suivant : `https://<company>.answerhub.com`

    b. Dans la zone **Identifier (Entity ID)** [Identificateur (ID d’entité)], entrez une URL au format suivant : `https://<company>.answerhub.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe du support technique AnswerHub](mailto:success@answerhub.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, sélectionnez le lien **Télécharger** à côté du **certificat (Base64)** correspondant à vos besoins, puis enregistrez le certificat sur votre ordinateur.

    ![Lien de téléchargement du certificat](common/certificatebase64.png)

6. Dans la section **Configurer AnswerHub**, copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

   Vous pouvez copier ces URL :
    - URL de connexion

    - Identificateur Azure AD

    - URL de déconnexion

### <a name="configure-answerhub-single-sign-on"></a>Configurer l’authentification unique AnswerHub

Dans cette section, vous allez configurer l’authentification unique pour AnswerHub.  

**Pour configurer l’authentification unique AnswerHub :**

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise AnswerHub en tant qu’administrateur.

    > [!NOTE]
    > Si vous avez besoin d’aide pour la configuration d’AnswerHub, contactez l’[équipe du support technique AnswerHub](mailto:success@answerhub.com.).

2. Accédez à **Administration**.

3. Sous l’onglet **Users and Groups** (Utilisateurs et groupes), dans le volet de gauche, dans la section **Social Settings** (Paramètres sociaux), sélectionnez **SAML Setup** (Configurer SAML).

4. Sous l’onglet **IDP Config** (Configurer le fournisseur d’identité), effectuez les étapes suivantes :

    ![Capture d’écran montrant la page AnswerHub avec l’onglet Users & Groups sélectionné.](./media/answerhub-tutorial/ic785172.png "SAML Setup")  

    a. Dans la zone **IDP Login URL** (URL de connexion du fournisseur d’identité), collez la valeur de l’**URL de connexion** que vous avez copiée à partir du portail Azure.

    b. Dans la zone **IDP Logout URL** (URL de déconnexion du fournisseur d’identité), collez la valeur de l’**URL de déconnexion** que vous avez copiée à partir du portail Azure.

    c. Dans la zone **IDP Name Identifier Format** (Format de l’identificateur du nom IDP), entrez la valeur d’**identificateur** que vous avez sélectionnée dans la section **Attributs utilisateur** du portail Azure.

    d. Sélectionnez **Keys and Certificates** (Clés et certificats).

5. Dans la section **Keys and Certificates**, effectuez les étapes suivantes :

    ![Section Keys and Certificates](./media/answerhub-tutorial/ic785173.png "Keys and Certificates")  

    a. Dans le Bloc-notes, ouvrez le certificat Base64 que vous avez téléchargé à partir du portail Azure, copiez son contenu, puis collez-le dans la zone **IDP Public Key (x509 Format)** [Clé publique IDP (format x509)].

    b. Sélectionnez **Enregistrer**.

6. Sous l’onglet **Configurer le fournisseur d’identité** (IDP Config), sélectionnez à nouveau **Enregistrer**.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, créez un utilisateur de test nommé Britta Simon dans le portail Azure.

**Pour créer un utilisateur de test Azure AD :**

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Sélectionnez Azure Active Directory, Utilisateurs, Tous les utilisateurs](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez ces étapes.

    ![Propriétés de l’utilisateur](common/user-properties.png)

    a. Dans la zone **Nom**, entrez **BrittaSimon**.

    b. Dans la zone **Nom d’utilisateur**, entrez **brittasimon\@<votre_domaine_d’entreprise.extension>** .  
    Par exemple : BrittaSimon@contoso.com.

    c. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.

    d. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez configurer l’utilisateur Britta Simon pour utiliser l’authentification unique Azure AD en lui accordant l’accès à AnswerHub.

**Pour attribuer l’utilisateur de test Azure AD :**

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **AnswerHub**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **AnswerHub**.

    ![Liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Sélectionner Utilisateurs et groupes](common/users-groups-blade.png)

4. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Users and groups** (Utilisateurs et groupes), sélectionnez **Britta Simon** dans la liste **Users** (Utilisateurs), puis sélectionnez le bouton **Select** (Sélectionner) en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Select Role** (Sélectionner un rôle), sélectionnez le rôle approprié pour l’utilisateur dans la liste. 

7. Sélectionnez le bouton **Select** (Sélectionner) en bas de l’écran.

8. Dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Affecter**.

### <a name="create-an-answerhub-test-user"></a>Créer un utilisateur de test AnswerHub

Pour autoriser les utilisateurs Azure AD à se connecter à AnswerHub, vous devez les ajouter dans AnswerHub. Dans AnswerHub, cette opération doit être effectuée manuellement.

**Pour configurer un compte d’utilisateur :**

1. Connectez-vous à votre site d’entreprise **AnswerHub** en tant qu’administrateur.

2. Accédez à **Administration**.

3. Sélectionnez l’onglet **Users & Groups** (Utilisateurs et groupes).

4. Dans le volet de gauche, dans la section **Manage Users** (Gérer les utilisateurs), sélectionnez **Create or import users** (Créer ou importer des utilisateurs), puis sélectionnez **Users & Groups** (Utilisateurs et groupes).

    ![Capture d’écran montrant la page AnswerHub avec l’onglet Users & Groups sélectionné et le lien Create or import users activé.](./media/answerhub-tutorial/ic785175.png "Utilisateurs et groupes")

5. Dans les zones appropriées, entrez les valeurs **Email address** (Adresse e-mail), **Username** (Nom d’utilisateur) et **Password** (Mot de passe) d’un compte Azure AD valide que vous souhaitez ajouter, puis sélectionnez **Save** (Enregistrer).

> [!NOTE]
> Pour configurer des comptes d’utilisateur Azure AD, vous pouvez utiliser n’importe quel autre outil ou API de création de compte d’utilisateur fourni par AnswerHub.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous sélectionnez la vignette AnswerHub dans le volet d’accès, vous devez être connecté automatiquement à l’application AnswerHub pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le volet d’accès, consultez la page [Présentation du volet d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour intégrer des applications SaaS à Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)