---
title: 'Tutoriel : Intégration d’Azure Active Directory à EverBridge | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Everbridge.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/18/2019
ms.author: jeedes
ms.openlocfilehash: 732362ef7099e93697320d8e47180c1207e8cb32
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995894"
---
# <a name="tutorial-azure-active-directory-integration-with-everbridge"></a>Tutoriel : Intégration d’Azure Active Directory à EverBridge

Dans ce didacticiel, vous allez apprendre à intégrer EverBridge à Azure Active Directory (Azure AD).
Lorsque vous intégrez EverBridge à Azure AD, vous pouvez :

* Contrôler qui a accès à EverBridge dans Azure AD.
* Permettre aux utilisateurs de se connecter automatiquement à EverBridge avec leur compte Azure AD. Ce contrôle d’accès porte le nom d’authentification unique (SSO).
* Gérer vos comptes à partir d’un emplacement centralisé à l’aide du portail Azure.
Pour plus d’informations sur l’intégration d’applications SaaS (software as a service) à Azure AD, consultez la page [Qu’est-ce que l’accès aux applications et l’authentification unique à Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Everbridge, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement EverBridge qui utilise l’authentification unique.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* EverBridge prend en charge l’authentification unique lancée par le fournisseur d’identité.

## <a name="add-everbridge-from-the-azure-marketplace"></a>Ajouter EverBridge à partir de la Place de marché Azure

Pour configurer l’intégration d’EverBridge à Azure AD, ajoutez EverBridge à partir de la Place de marché Azure à votre liste d’applications SaaS gérées.

Pour ajouter EverBridge à partir de la Place de marché Azure, effectuez les étapes suivantes.

1. Dans le panneau de navigation gauche du [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **EverBridge**. Sélectionnez **EverBridge** dans le volet des résultats, puis choisissez **Ajouter**.

     ![EverBridge dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec EverBridge à l’aide de l’utilisateur de test Britta Simon.
Pour que l’authentification unique fonctionne, établissez une relation entre un utilisateur Azure AD et l’utilisateur EverBridge associé.

Pour configurer et tester l’authentification unique Azure AD avec EverBridge, suivez les indications des sections ci-dessous :

- [Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on) pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
- [Configurer EverBridge comme authentification unique au portail de manager EverBridge](#configure-everbridge-as-everbridge-manager-portal-single-sign-on) pour configurer les paramètres de l’authentification unique côté application.
- [Configurer EverBridge comme authentification unique au portail de membre EverBridge](#configure-everbridge-as-everbridge-member-portal-single-sign-on) pour configurer les paramètres de l’authentification unique côté application.
- [Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user) pour tester l’authentification unique Azure AD avec Britta Simon.
- [Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user) pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
- [Créer un utilisateur de test EverBridge](#create-an-everbridge-test-user) pour avoir un équivalent de Britta Simon dans EverBridge lié à la représentation Azure AD de l’utilisateur.
- [Tester l’authentification unique](#test-single-sign-on) pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec EverBridge, effectuez les étapes suivantes.

1. Dans le [portail Azure](https://portal.azure.com/), sur la page d’intégration de l’application **EverBridge**, sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** pour activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

    >[!NOTE]
    >Configurez l’application en tant que portail de manager *ou* en tant que portail de membre à la fois sur le portail Azure et sur le portail EverBridge.

4. Pour configurer l’application **EverBridge** en tant que **portail de manager EverBridge**, dans la section **Configuration SAML de base**, procédez comme suit :

    ![Informations d’authentification unique du domaine et des URL EverBridge](common/idp-intiated.png)

    a. Dans la zone **Identificateur**, entrez une URL en suivant le modèle `https://sso.everbridge.net/<API_Name>`

    b. Dans la zone **URL de réponse**, entrez une URL en suivant le modèle `https://manager.everbridge.net/saml/SSO/<API_Name>/alias/defaultAlias`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez [l’équipe de support technique EverBridge](mailto:support@everbridge.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Pour configurer l’application **EverBridge** en tant que **portail de membre EverBridge**, dans la section **Configuration SAML de base**, procédez comme suit :

  * Si vous voulez configurer l’application en mode lancé par le fournisseur d’identité, procédez comme suit :

     ![Informations d’authentification unique du domaine et des URL EverBridge en mode lancé par le fournisseur d’identité](common/idp-intiated.png)

    a. Dans la zone **Identificateur**, entrez une URL en suivant le modèle `https://sso.everbridge.net/<API_Name>/<Organization_ID>`

    b. Dans la zone **URL de réponse**, entrez une URL en suivant le modèle `https://member.everbridge.net/saml/SSO/<API_Name>/<Organization_ID>/alias/defaultAlias`

   * Si vous voulez configurer l’application en mode lancé par le fournisseur de services, sélectionnez **Définir des URL supplémentaires**, puis procédez comme suit :

     ![Informations d’authentification unique du domaine et des URL EverBridge en mode lancé par le fournisseur de services](common/both-signonurl.png)

     a. Dans la zone **URL de connexion**, entrez une URL en suivant le modèle `https://member.everbridge.net/saml/login/<API_Name>/<Organization_ID>/alias/defaultAlias?disco=true`

     > [!NOTE]
     > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL d’authentification, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez [l’équipe de support technique EverBridge](mailto:support@everbridge.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

6. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, sélectionnez **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération**. Enregistrez-le sur votre ordinateur.

    ![Lien de téléchargement du certificat](common/metadataxml.png)

7. Dans la section **Configurer EverBridge**, copiez les URL dont vous avez besoin :

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    - URL de connexion
    - Identificateur Azure AD
    - URL de déconnexion

### <a name="configure-everbridge-as-everbridge-manager-portal-single-sign-on"></a>Configurer EverBridge comme authentification unique au portail de manager EverBridge

Pour configurer l’authentification unique sur **EverBridge** comme une application de **portail de manager EverBridge**, procédez comme suit.
 
1. Dans une autre fenêtre de navigateur web, connectez-vous à EverBridge en tant qu’administrateur.

1. Dans le menu situé en haut, cliquez sur l’onglet **Paramètres**. Sous **Sécurité**, sélectionnez **Authentification unique**.
   
     ![Configurer l’authentification unique](./media/everbridge-tutorial/tutorial_everbridge_002.png)
   
     a. Dans la zone **Nom**, entrez le nom du fournisseur d’identité. Exemple : le nom de votre société.
   
     b. Dans la zone **Nom de l’API**, entrez le nom de l’API.
   
     c. Sélectionnez **Choisir un fichier** pour charger le fichier de métadonnées que vous avez téléchargé sur le Portail Azure.
   
     d. Pour **Emplacement de l’identité SAML**, sélectionnez **L’identité est dans l’élément NameIdentifier de l’instruction Subject**.
   
     e. Dans la zone **URL de connexion du fournisseur identité**, collez la valeur de l’**URL de connexion** que vous avez copiée à partir du portail Azure.
   
     f. Pour **Liaison de demande initiée par le fournisseur de service**, sélectionnez **Redirection HTTP**.

     g. Sélectionnez **Enregistrer**.

### <a name="configure-everbridge-as-everbridge-member-portal-single-sign-on"></a>Configurer EverBridge comme authentification unique au portail de membre EverBridge

Pour configurer l’authentification unique sur **EverBridge** en tant que **portail de membre EverBridge**, envoyez le fichier **XML des métadonnées de fédération** à l’[équipe de support technique EverBridge](mailto:support@everbridge.com). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

Pour créer l’utilisateur test Britta Simon dans le portail Azure, procédez comme suit.

1. Dans le portail Azure, dans le volet gauche, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs**.

    ![Liens Utilisateurs et Tous les utilisateurs](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans la boîte de dialogue **Utilisateur**, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans la zone **Nom**, entrez **BrittaSimon**.
  
    b. Dans la zone **Nom d’utilisateur**, entrez `brittasimon@yourcompanydomain.extension`. par exemple BrittaSimon@contoso.com.

    c. Cochez la case **Afficher le mot de passe**. Notez la valeur affichée dans la zone **Mot de passe**.

    d. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Autorisez Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à EverBridge.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications** >**EverBridge**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Everbridge**.

    ![Lien EverBridge dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien Utilisateurs et groupes](common/users-groups-blade.png)

4. Sélectionnez **Ajouter un utilisateur**. Dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Utilisateurs et groupes**.

    ![Boîte de dialogue Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste d’utilisateurs. Choisissez **Sélectionner** au bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste. Choisissez **Sélectionner** au bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Affecter**.

### <a name="create-an-everbridge-test-user"></a>Créer un utilisateur de test EverBridge

Dans cette section, vous allez créer l’utilisateur de test appelé Britta Simon dans EverBridge. Pour ajouter des utilisateurs dans la plateforme EverBridge, contactez l’[équipe de support technique EverBridge](mailto:support@everbridge.com). Les utilisateurs doivent être créés et activés dans EverBridge pour que vous puissiez utiliser l’authentification unique. 

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Testez la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette EverBridge dans le volet d’accès, vous devez être connecté automatiquement au compte EverBridge pour lequel vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)
- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)
- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)