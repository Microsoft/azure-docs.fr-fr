---
title: 'Didacticiel : Intégration d’Azure Active Directory à Zscaler Beta | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Zscaler Beta.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/24/2019
ms.author: jeedes
ms.openlocfilehash: 1fec471e4047707f2b7cbaa55fdc7f2256bebe93
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92519863"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-beta"></a>Didacticiel : Intégration d’Active Directory à Zscaler Beta

Dans ce didacticiel, vous allez découvrir comment intégrer Zscaler Beta à Azure Active Directory (Azure AD).
Quand vous intégrez Zscaler Beta à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Zscaler Beta.
* Permettre aux utilisateurs de se connecter automatiquement à Zscaler Beta avec leur compte Azure AD. Ce contrôle d’accès porte le nom d’authentification unique (SSO).
* Gérer vos comptes à partir d’un emplacement centralisé à l’aide du portail Azure.

Pour plus d’informations sur l’intégration d’applications SaaS (software as a service) à Azure AD, consultez la page [Qu’est-ce que l’accès aux applications et l’authentification unique à Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Zscaler Beta, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Zscaler Beta qui utilise l’authentification unique.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Zscaler Beta prend en charge l’authentification unique lancée par le fournisseur de services.
* Zscaler Beta prend en charge le provisionnement d’utilisateurs Juste-à-temps.

## <a name="add-zscaler-beta-from-the-azure-marketplace"></a>Ajouter Zscaler Beta à partir de la Place de marché Azure

Pour configurer l’intégration de Zscaler Beta à Azure AD, ajoutez Zscaler Beta, disponible à partir de la Place de marché, à votre liste d’applications SaaS gérées.

Pour ajouter Zscaler Beta à partir de la Place de marché Azure, effectuez les étapes suivantes.

1. Dans le panneau de navigation gauche du [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Zscaler Beta** . Sélectionnez **Zscaler Beta** dans le panneau de résultats, puis sélectionnez **Ajouter** .

     ![Zscaler Beta dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Zscaler Beta à l’aide de l’utilisateur de test Britta Simon.
Pour que l’authentification unique fonctionne, établissez une relation entre un utilisateur Azure AD et l’utilisateur Zscaler Beta associé.

Pour configurer et tester l’authentification unique Azure AD avec Zscaler Beta, suivez les indications des sections suivantes :

- [Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on) pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
- [Configurer l’authentification unique Zscaler Beta](#configure-zscaler-beta-single-sign-on) pour configurer les paramètres de l’authentification unique côté application.
- [Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user) pour tester l’authentification unique Azure AD avec Britta Simon.
- [Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user) pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
- [Créer un utilisateur de test Zscaler Beta](#create-a-zscaler-beta-test-user) pour avoir un équivalent de Britta Simon dans Zscaler Beta qui est lié à la représentation de l’utilisateur Azure AD.
- [Tester l’authentification unique](#test-single-sign-on) pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Zscaler Beta, effectuez les étapes suivantes.

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Zscaler Beta** , sélectionnez **Authentification unique** .

    ![Configurer le lien d’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** pour activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez cette étape :

    ![Informations d’authentification unique dans Domaine et URL Zscaler Beta](common/sp-intiated.png)

    - Dans la zone **URL de connexion** , entrez l’URL utilisée par vos utilisateurs pour se connecter à l’application Zscaler Beta.

    > [!NOTE]
    > Cette valeur n’est pas réelle. Remplacez-la par la valeur d’URL de connexion réelle. Pour obtenir la valeur, contactez l’[équipe de support technique de Zscaler Beta](https://www.zscaler.com/company/contact).

5. L’application Zscaler Beta attend les assertions SAML dans un format spécifique. L’utilisateur doit ajouter des mappages d’attributs personnalisés à la configuration d’attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut. Sélectionnez **Modifier** pour ouvrir la boîte de dialogue **Attributs utilisateur** .

    ![Boîte de dialogue Attributs utilisateur](common/edit-attribute.png)

6. L’application Zscaler Beta s’attend à ce que quelques attributs supplémentaires soient passés dans la réponse SAML. Dans la section **Revendications des utilisateurs** de la boîte de dialogue **Attributs utilisateur** , effectuez les étapes suivantes pour ajouter le jeton SAML comme indiqué dans le tableau ci-dessous.
    
    | Name | Attribut source | 
    | ---------------| --------------- |
    | memberOf  | user.assignedroles |

    a. Sélectionnez **Ajouter une nouvelle revendication** pour ouvrir la boîte de dialogue **Gérer les revendications des utilisateurs** .

    ![Boîte de dialogue des revendications des utilisateurs](common/new-save-attribute.png)

    ![Boîte de dialogue Gérer les revendications des utilisateurs](common/new-attribute-details.png)

    b. Dans la zone de texte **Nom** , entrez le nom d’attribut affiché pour cette ligne.

    c. Laissez la zone **Espace de noms** vide.

    d. Dans **Source** , sélectionnez **Attribut** .

    e. Dans la liste **Attribut de la source** , entrez la valeur d’attribut affichée pour cette ligne.

    f. Sélectionnez **OK** .

    g. Sélectionnez **Enregistrer** .

    > [!NOTE]
    > Pour découvrir comment configurer des rôles dans Azure AD, consultez [Configurer la revendication de rôle](../develop/active-directory-enterprise-app-role-management.md).

7. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , sélectionnez **Télécharger** pour télécharger le **Certificat (Base64)** . Enregistrez-le sur votre ordinateur.

    ![Lien de téléchargement du certificat](common/certificatebase64.png)

8. Dans la section **Configurer Zscaler Beta** , copiez les URL dont vous avez besoin :

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    - URL de connexion
    - Identificateur Azure AD
    - URL de déconnexion

### <a name="configure-zscaler-beta-single-sign-on"></a>Configurer l’authentification unique Zscaler Beta

1. Pour automatiser la configuration dans Zscaler Beta, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en sélectionnant **Install the extension** (Installer l’extension).

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Une fois que vous avez ajouté l’extension au navigateur, la sélection de **Configurer Zscaler Beta** vous dirige vers l’application Zscaler Beta. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Zscaler Beta. L’extension de navigateur configure automatiquement l’application pour vous et automatise les étapes 3 à 6.

    ![Configuration](common/setup-sso.png)

3. Pour configurer manuellement Zscaler Beta, ouvrez une nouvelle fenêtre de navigateur web. Connectez-vous à votre site d’entreprise Zscaler Beta en tant qu’administrateur et effectuez les étapes suivantes.

4. Accédez à **Administration** > **Authentication** > **Authentication Settings** et effectuez les étapes suivantes.
   
    ![Administration](./media/zscaler-beta-tutorial/ic800206.png "Administration")

    a. Sous **Authentication Type** (Type d’authentification), sélectionnez **SAML** .

    b. Sélectionnez **Configure SAML** (Configurer SAML).

5. Dans la fenêtre **Edit SAML** (Modifier SAML), effectuez les étapes suivantes : 
            
    ![Gérer les utilisateurs et l’authentification](./media/zscaler-beta-tutorial/ic800208.png "Manage Users & Authentication")
    
    a. Dans la zone **SAML Portal URL** (URL du portail SAML), collez la valeur de l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    b. Dans la zone de texte **Login Name Attribute** (Attribut de nom de connexion), entrez **NameID** .

    c. Dans la zone **Public SSL Certificate** (Certificat SSL public), sélectionnez **Upload** pour charger le certificat de signature SAML Azure que vous avez téléchargé à partir du portail Azure.

    d. Cliquez sur le bouton bascule **Enable SAML Auto-Provisioning** (Activer le provisionnement automatique SAML).

    e. Dans la zone **User Display Name Attribute** (Attribut du nom d’affichage de l’utilisateur), entrez **displayName** si vous souhaitez activer le provisionnement automatique SAML pour les attributs displayName.

    f. Dans la zone **Group Name Attribute** (Attribut du nom de groupe), entrez **memberOf** si vous souhaitez activer le provisionnement automatique SAML pour les attributs memberOf.

    g. Dans **Department Name Attribute** (Attribut du nom du service), entrez **department** si vous souhaitez activer le provisionnement automatique SAML pour les attributs department.

    h. Sélectionnez **Enregistrer** .

6. Dans la page **Configure User Authentication** (Configurer l’authentification utilisateur), effectuez les étapes suivantes :

    ![Menu Activation et bouton Activer](./media/zscaler-beta-tutorial/ic800207.png)

    a. Passez la souris sur le menu **Activation** en bas à gauche.

    b. Sélectionnez **Activer** .

## <a name="configure-proxy-settings"></a>Configuration des paramètres de proxy
Pour configurer les paramètres de proxy dans Internet Explorer, effectuez les étapes suivantes :

1. Démarrez **Internet Explorer** .

2. Dans le menu **Outils** , sélectionnez **Options Internet** pour ouvrir la boîte de dialogue **Options Internet** . 
    
     ![Boîte de dialogue Options Internet](./media/zscaler-beta-tutorial/ic769492.png "Options Internet")

3. Sélectionnez l’onglet **Connexions** . 
  
     ![Onglet Connexions](./media/zscaler-beta-tutorial/ic769493.png "Connexions")

4. Sélectionnez **Paramètres réseau** pour ouvrir la boîte de dialogue **Paramètres du réseau local** .

5. Dans la section **Serveur proxy** , effectuez les étapes suivantes : 
   
    ![Section Serveur proxy](./media/zscaler-beta-tutorial/ic769494.png "Serveur proxy")

    a. Cochez la case **Utiliser un serveur proxy pour le réseau local** .

    b. Dans la zone **Adresse** , entrez **gateway.Zscaler Beta.net** .

    c. Dans la zone **Port** , entrez **80** .

    d. Cochez la case **Ne pas utiliser de serveur proxy pour les adresses locales** .

    e. Sélectionnez **OK** pour fermer la boîte de dialogue **Paramètres du réseau local** .

6. Sélectionnez **OK** pour fermer la boîte de dialogue **Options Internet** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

Créez un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le portail Azure, dans le volet gauche, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs** .

    ![Liens Utilisateurs et Tous les utilisateurs](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans la boîte de dialogue **Utilisateur** , effectuez les étapes suivantes :

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans la zone **Nom** , entrez **BrittaSimon** .
  
    b. Dans la zone **Nom d’utilisateur** , entrez `brittasimon@yourcompanydomain.extension`. par exemple BrittaSimon@contoso.com.

    c. Cochez la case **Afficher le mot de passe** . Notez la valeur affichée dans la zone **Mot de passe** .

    d. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Autorisez Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Zscaler Beta.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications** > **Zscaler Beta** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, entrez et sélectionnez **Zscaler Beta** .

    ![Lien Zscaler Beta dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien Utilisateurs et groupes](common/users-groups-blade.png)

4. Sélectionnez **Ajouter un utilisateur** . Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Utilisateurs et groupes** .

    ![Bouton Ajouter un utilisateur](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez l’utilisateur **Britta Simon** dans la liste. Choisissez **Sélectionner** au bas de l’écran.

    ![Boîte de dialogue Utilisateurs et groupes](./media/zscaler-beta-tutorial/tutorial_zscalerbeta_users.png)

6. Dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle d’utilisateur approprié dans la liste. Choisissez **Sélectionner** au bas de l’écran.

    ![Sélectionner la boîte de dialogue Rôle](./media/zscaler-beta-tutorial/tutorial_zscalerbeta_roles.png)

7. Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Affecter** .

    ![Boîte de dialogue Ajouter une attribution](./media/zscaler-beta-tutorial/tutorial_zscalerbeta_assign.png)

### <a name="create-a-zscaler-beta-test-user"></a>Créer un utilisateur de test Zscaler Beta

Dans cette section, l’utilisateur Britta Simon est créé dans Zscaler Beta. Zscaler Beta prend en charge l’ **attribution d’utilisateurs juste-à-temps** , option activée par défaut. Vous n’avez rien à faire dans cette section. S’il n’existe pas encore d’utilisateur dans Zscaler Beta, il en est créé un après l’authentification.

>[!Note]
>Pour créer un utilisateur manuellement, contactez l’[équipe de support technique Zscaler Beta](https://www.zscaler.com/company/contact).

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Testez la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Zscaler Beta dans le volet d’accès, vous devez être connecté automatiquement à l’application Zscaler Beta pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)
- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)
- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)