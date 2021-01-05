---
title: "Tutoriel : Intégration d'Azure Active Directory à Workspot Control | Microsoft Docs"
description: Découvrez comment configurer l’authentification unique pour Azure Active Directory et Workspot Control.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 3/11/2019
ms.author: jeedes
ms.openlocfilehash: 65e424c6275f9990a5d5a42a54bc4131300a45bf
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895033"
---
# <a name="tutorial-azure-active-directory-integration-with-workspot-control"></a>Tutoriel : Intégration d'Azure Active Directory à Workspot Control

Dans ce tutoriel, vous allez apprendre à intégrer Workspot Control à Azure Active Directory (Azure AD). Quand vous intégrez Workspot Control à Azure AD, vous pouvez :

* utiliser Azure AD pour contrôler qui a accès à Workspot Control ;
* autoriser les utilisateurs à se connecter automatiquement à Workspot Control (par le biais de l’authentification unique) avec leur compte Azure AD ;
* gérer vos comptes à un emplacement central : le portail Azure.

Pour plus d’informations sur l’intégration d’applications SaaS à Azure AD, consultez l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure AD](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Workspot Control, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/).

* Un abonnement Workspot Control pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

> [!Note]
> Workspot Control prend en charge l’authentification unique lancée par le fournisseur de services et le fournisseur d’identité.


## <a name="adding-workspot-control-from-the-gallery"></a>Ajout de Workspot Control depuis la galerie

Pour configurer l’intégration de Workspot Control à Azure AD, vous devez ajouter Workspot Control à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Workspot Control à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet gauche du [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .

    ![Volet Applications d’entreprise](common/enterprise-applications.png)

3. Sélectionnez **Nouvelle application** en haut de la fenêtre.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Workspot Control** , sélectionnez **Workspot Control** dans le volet de résultats, puis sélectionnez **Ajouter** .

     ![Fenêtre « Ajouter à partir de la galerie »](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous configurez et testez l’authentification unique Azure AD avec Workspot Control pour un utilisateur de test nommé Britta Simon.
Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans Workspot Control.

Pour configurer et tester l’authentification unique Azure AD avec Workspot Control, vous devez effectuer les tâches suivantes :

1. [Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on) pour permettre à vos utilisateurs d’utiliser la fonctionnalité.
2. [Configurer l’authentification unique Workspot Control](#configure-workspot-control-single-sign-on) pour configurer les paramètres de l’authentification unique côté application.
3. [Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user) pour tester l’authentification unique Azure AD pour Britta Simon.
4. [Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user) pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. [Créer un utilisateur de test Workspot Control](#create-a-workspot-control-test-user) pour avoir dans Workspot Control un équivalent de Britta Simon lié à la représentation Azure AD associée.
6. [Tester l’authentification unique](#test-single-sign-on) pour vérifier que la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Workspot Control, effectuez les étapes suivantes :

1. Dans le **portail Azure** , dans la page d’intégration de l’application [Workspot Control](https://portal.azure.com/), sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la fenêtre **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML** pour activer l’authentification unique.

    ![Fenêtre Sélectionner une méthode d’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône **Modifier** (crayon) pour accéder à **Configuration SAML de base** .

    ![Icône Modifier en surbrillance dans « Configuration SAML de base »](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le fournisseur d’identité, effectuez ces étapes :

    ![Capture d’écran montrant Configuration SAML de base, où vous pouvez entrer l’identificateur, l’URL de réponse, et sélectionner Enregistrer.](common/idp-intiated.png)

    1. Dans la zone de texte **Identificateur** , entrez une URL au format suivant :<br/>
    **_https://<<i></i>NOMINSTANCE>-saml.workspot.com/saml/metadata_* _

    1. Dans la zone de texte _*URL de réponse* *, entrez une URL au format suivant :<br/>
    **_https://< <i></i>NOMINSTANCE>-saml.workspot.com/saml/assertion_*_

5. Si vous voulez configurer l’application en mode initié par le fournisseur de services, sélectionnez _*Définir des URL supplémentaires**.

    ![Capture d’écran montrant Définir des URL supplémentaires, où vous pouvez entrer une URL de connexion.](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion** , entrez une URL au format suivant :<br/>
    **_https://< <i></i>NOMINSTANCE>-saml.workspot.com/_* _

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Remplacez ces valeurs par l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez l’[équipe du support technique Workspot Control](mailto:support@workspot.com). Vous pouvez également consulter les modèles présentés dans la section _ *Configuration SAML de base* * du portail Azure.

6. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , sélectionnez **Télécharger** pour télécharger le **certificat (Base64)** approprié parmi les options disponibles. Enregistrez-le sur votre ordinateur.

    ![Lien de téléchargement du certificat (en base64)](common/certificatebase64.png)

7. Dans la section **Configurer Workspot Control** , copiez les URL appropriées en fonction de vos besoins :

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    - **URL de connexion**

    - **Identificateur Azure AD**

    - **URL de déconnexion**

### <a name="configure-workspot-control-single-sign-on"></a>Configurer l’authentification unique pour Workspot Control

1. Ouvrez une autre fenêtre de navigateur web, puis connectez-vous à Workspot Control en tant qu’administrateur de la sécurité.

2. Dans la barre d’outils en haut de la page, sélectionnez **Setup** (Configuration), puis **SAML** .

    ![Options d’installation](./media/workspotcontrol-tutorial/tutorial_workspotcontrol_setup.png)

3. Dans la fenêtre **Security Assertion Markup Language Configuration** (Configuration SAML), effectuez ces étapes :
 
    ![Fenêtre Security Assertion Markup Language Configuration](./media/workspotcontrol-tutorial/tutorial_workspotcontrol_saml.png)

    1. Dans la zone **Entity ID** (ID d’entité), collez la valeur **Azure AD Identifier** (Identificateur Azure AD) que vous avez copiée à partir du portail Azure.

    1. Dans la zone **Signon Service URL** (URL du service de connexion), collez l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    1. Dans la zone **Logout Service URL** (URL du service de déconnexion), collez l’ **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    1. Sélectionnez **Update File** (Mettre à jour le fichier) pour charger le certificat X.509 encodé en base 64 que vous avez téléchargé à partir du portail Azure.

    1. Sélectionnez **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous créez un utilisateur de test dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** en haut de la fenêtre.

    ![Bouton « Nouvel utilisateur »](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez ces étapes :

    ![Fenêtre des propriétés de l’utilisateur](common/user-properties.png)

    1. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    1. Dans le champ **Nom d’utilisateur** , entrez * *brittasimon@* yourcompanydomain.extension***. Par exemple, entrez **BrittaSimon@contoso. <i></i>com** .

    1. Cochez la case **Afficher le mot de passe** . Notez ensuite la valeur affichée dans la zone **Mot de passe** .

    1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez Britta Simon à accéder à Workspot Control pour lui permettre d’utiliser l’authentification unique Azure.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Workspot Control** .

    ![Volet Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Workspot Control** .

    ![Lien Workspot Control dans la liste des applications](common/all-applications.png)

3. Dans le menu situé à gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Sélectionnez le bouton **Ajouter un utilisateur** . Ensuite, sélectionnez **Utilisateurs et groupes** dans la fenêtre **Ajouter une attribution** .

    ![Fenêtre « Ajouter une attribution »](common/add-assign-user.png)

5. Dans la fenêtre **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste **Utilisateurs** . Puis cliquez sur **Sélectionner** .

6. Si vous attendez une valeur de rôle dans l’assertion SAML, sélectionnez le rôle approprié pour l’utilisateur dans la liste de la fenêtre **Sélectionner un rôle** . Cliquez ensuite sur **Sélectionner** en bas.

7. Dans la fenêtre **Ajouter une affectation** , sélectionnez **Affecter** .

### <a name="create-a-workspot-control-test-user"></a>Créer un utilisateur de test Workspot Control

Pour pouvoir se connecter à Workspot Control, les utilisateurs Azure AD doivent être provisionnés dans Workspot Control. Le provisionnement est une tâche manuelle.

**Pour configurer un compte d’utilisateur, effectuez ces étapes :**

1. Connectez-vous à Workspot Control en tant qu’administrateur de la sécurité.

2. Dans la barre d’outils en haut de la page, sélectionnez **Users** (Utilisateurs), puis **Add User** (Ajouter un utilisateur).

    ![Options « Utilisateurs »](./media/workspotcontrol-tutorial/tutorial_workspotcontrol_adduser.png)

3. Dans la fenêtre **Add a New User** (Ajouter un nouvel utilisateur), effectuez ces étapes :

    ![Fenêtre « Add a New User »](./media/workspotcontrol-tutorial/tutorial_workspotcontrol_addnewuser.png)

    1. Dans la zone **First name** (Prénom), entrez le prénom d’un utilisateur, par exemple **Britta** .

    1. Dans la zone de texte **Last name** (Nom), entrez le nom de famille de l’utilisateur, par exemple **Simon** .

    1. Dans la zone **Email** (E-mail), entrez l’adresse e-mail de l’utilisateur, par exemple **Brittasimon@contoso. <i></i>com** .

    1. Sélectionnez le rôle d’utilisateur approprié dans la liste déroulante **Role** (Rôle).

    1. Sélectionnez le groupe d’utilisateurs approprié dans la liste déroulante **Group** (Groupe).

    1. Sélectionnez **Add User** (Ajouter un utilisateur).

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous testez la configuration de l’authentification unique Azure AD par le biais du *volet d’accès* .

Quand vous cliquez sur la vignette **Workspot Control** dans le volet d’accès, vous devez être connecté automatiquement à l’application Workspot Control pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations, consultez [Présentation du volet d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)