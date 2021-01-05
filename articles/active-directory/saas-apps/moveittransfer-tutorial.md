---
title: 'Didacticiel : Intégration d’Azure Active Directory à MOVEit Transfer - Azure AD integration | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et MOVEit Transfer - Azure AD integration.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/25/2019
ms.author: jeedes
ms.openlocfilehash: c79596747be655e372958e06021c50f23ff0fe03
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92508225"
---
# <a name="tutorial-azure-active-directory-integration-with-moveit-transfer---azure-ad-integration"></a>Didacticiel : Intégration d’Azure Active Directory à MOVEit Transfer - Azure AD integration

Dans ce didacticiel, vous allez apprendre à intégrer MOVEit Transfer - Azure AD integration à Azure Active Directory (Azure AD).
L’intégration de MOVEit Transfer - Azure AD integration à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à MOVEit Transfer - Azure AD integration.
* Vous pouvez autoriser les utilisateurs à se connecter automatiquement à MOVEit Transfer - Azure AD integration (via l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration de MOVEit Transfer - Azure AD integration à Azure AD, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement MOVEit Transfer - Azure AD integration pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* MOVEit Transfer - Azure AD integration prend en charge l’authentification unique initiée par le **fournisseur de services** .

## <a name="adding-moveit-transfer---azure-ad-integration-from-the-gallery"></a>Ajout de MOVEit Transfer - Azure AD integration à partir de la galerie

Pour configurer l’intégration de MOVEit Transfer - Azure AD integration avec Azure AD, vous devez ajouter MOVEit Transfer - Azure AD integration, disponible dans la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter MOVEit Transfer - Azure AD integration à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **MOVEit Transfer - Azure AD integration** , sélectionnez **MOVEit Transfer - Azure AD integration** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![MOVEit Transfer - Azure AD integration dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec MOVEit Transfer - Azure AD integration avec un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, il faut établir une relation entre l’utilisateur Azure AD et l’utilisateur MOVEit Transfer - Azure AD integration associé.

Pour configurer et tester l’authentification unique Azure AD avec MOVEit Transfer - Azure AD integration, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique MOVEit Transfer - Azure AD integration](#configure-moveit-transfer---azure-ad-integration-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test MOVEit Transfer - Azure AD integration](#create-moveit-transfer---azure-ad-integration-test-user)** pour avoir un équivalent de Britta Simon dans MOVEit Transfer - Azure AD integration lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec MOVEit Transfer - Azure AD integration, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), sur la page d’intégration de l’application **MOVEit Transfer - Azure AD integration** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , si vous disposez d’un **fichier de métadonnées du fournisseur de services** , suivez les étapes ci-dessous :

    a. Cliquez sur **Charger un fichier de métadonnées** .

    ![Charger le fichier de métadonnées](common/upload-metadata.png)

    b. Cliquez sur le **logo du dossier** pour sélectionner le fichier de métadonnées, puis cliquez sur **Charger** .

    ![choisir le fichier de métadonnées](common/browse-upload-metadata.png)

    c. Une fois le fichier de métadonnées chargé, les valeurs **Identificateur** et **URL de réponse** sont automatiquement renseignées dans la section **Configuration SAML de base**  :

    ![Informations d’authentification unique dans Domaine et URL MOVEit Transfer - Azure AD integration](common/sp-identifier-reply.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://contoso.com`

    > [!NOTE]
    > La valeur d’ **URL de connexion** n’est pas réelle. Mettez à jour la valeur avec l’URL de connexion réelle. Contactez l’équipe du [support technique MOVEit Transfer - Azure AD integration](https://community.ipswitch.com/s/support) pour obtenir la valeur. Vous pouvez télécharger le **fichier de métadonnées du fournisseur de services** à partir de l’ **URL de métadonnées du fournisseur de services** , comme qui est expliqué plus loin dans la section **Configurer l’authentification unique MOVEit Transfer - Azure AD integration** du didacticiel. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

4. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configurer MOVEit Transfer - Azure AD integration** , copiez les URL appropriées, selon vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-moveit-transfer---azure-ad-integration-single-sign-on"></a>Configurer l’authentification unique MOVEit Transfer - Azure AD integration

1. Connectez-vous à votre client MOVEit Transfer - Azure AD integration en tant qu’administrateur.

2. Dans le volet de navigation gauche, cliquez sur **Paramètres** .

    ![Section Paramètres côté application](./media/moveittransfer-tutorial/tutorial_moveittransfer_000.png)

3. Cliquez sur le lien **Authentification unique** qui se trouve sous **Stratégies de sécurité -> Authentification des utilisateurs** .

    ![Stratégies de sécurité côté application](./media/moveittransfer-tutorial/tutorial_moveittransfer_001.png)

4. Cliquez sur le lien URL de métadonnées pour télécharger le document de métadonnées.

    ![URL de métadonnées du fournisseur de service](./media/moveittransfer-tutorial/tutorial_moveittransfer_002.png)
    
   * Vérifiez qu’ **entityID** correspond à **Identifier** dans la section **Configuration SAML de base** .
   * Vérifiez que l’URL de l’emplacement **AssertionConsumerService** correspond à **l’URL de réponse** dans la section **Configuration SAML de base** .
    
     ![Configurer l’authentification unique côté application](./media/moveittransfer-tutorial/tutorial_moveittransfer_007.png)

5. Cliquez sur **Add Identity Provider** (Ajouter un fournisseur d’identité) pour ajouter un nouveau fournisseur d’identité fédéré.

    ![Ajouter un fournisseur d’identité](./media/moveittransfer-tutorial/tutorial_moveittransfer_003.png)

6. Cliquez sur **Parcourir...** pour sélectionner le fichier de métadonnées que vous avez téléchargé à partir du portail Azure, puis cliquez sur **Ajouter un fournisseur d’identité** pour charger le fichier téléchargé.

    ![Fournisseur d’identité SAML](./media/moveittransfer-tutorial/tutorial_moveittransfer_004.png)

7. Sélectionnez **Oui** sous **Activé** dans la page **Edit Federated Identity Provider Settings** (Modifier les paramètres du fournisseur d’identité fédéré...), puis cliquez sur **Enregistrer** .

    ![Paramètres de fournisseur d’identité fédérée](./media/moveittransfer-tutorial/tutorial_moveittransfer_005.png)

8. Dans la page **Edit Federated Identity Provider User Settings** (Modifier les paramètres utilisateur du fournisseur d’identité fédérée), effectuez les actions suivantes :
    
    ![Modifier les paramètres de fournisseur d’identité fédérée](./media/moveittransfer-tutorial/tutorial_moveittransfer_006.png)
    
    a. Sélectionnez **SAML NameID** comme **Nom de connexion** .
    
    b. Sélectionnez **Autre** comme **Nom complet** et dans la zone de texte **Nom de l’attribut** , insérez la valeur : `http://schemas.microsoft.com/identity/claims/displayname`.
    
    c. Sélectionnez **Autre** comme **E-mail** et dans la zone de texte **Nom de l’attribut** , insérez la valeur : `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`.
    
    d. Sélectionnez **Oui** sous **Auto-create account on signon** (Créer automatiquement un compte lors de l’authentification).
    
    e. Cliquez sur le bouton **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon\@domainedevotreentreprise.extension** .  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à MOVEit Transfer - Azure AD integration.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **MOVEit Transfer - Azure AD integration** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **MOVEit Transfer - Azure AD integration** .

    ![Lien MOVEit Transfer - Azure AD integration dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-moveit-transfer---azure-ad-integration-test-user"></a>Créer un utilisateur de test MOVEit Transfer - Azure AD integration

L’objectif de cette section est de créer un utilisateur appelé Britta Simon dans MOVEit Transfer - Azure AD integration. MOVEit Transfer - Azure AD integration prend en charge l’approvisionnement juste-à-temps que vous avez activé. Vous n’avez aucune opération à effectuer dans cette section. Un utilisateur est créé lors d’une tentative d’accès à MOVEit Transfer - Azure AD integration s’il n’existe pas déjà.

>[!NOTE]
>Si vous devez créer un utilisateur manuellement, contactez l’[équipe du support technique MOVEit Transfer - Azure AD integration](https://community.ipswitch.com/s/support).

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la vignette MOVEit Transfer - Azure AD integration dans le volet d’accès, vous êtes connecté automatiquement à l’application MOVEit Transfer - Azure AD integration pour laquelle vous configurez l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)