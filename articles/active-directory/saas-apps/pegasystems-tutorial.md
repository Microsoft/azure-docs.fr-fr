---
title: 'Didacticiel : Intégration d’Azure Active Directory à Pega Systems | Microsoft Docs'
description: Dans ce tutoriel, vous allez découvrir comment configurer l’authentification unique entre Azure Active Directory et Pega Systems.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/26/2019
ms.author: jeedes
ms.openlocfilehash: 4fb117b7f7b9a0c7a6a67e2714380a01cd53a4e0
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92515637"
---
# <a name="tutorial-azure-active-directory-integration-with-pega-systems"></a>Didacticiel : Intégration d’Azure Active Directory à Pega Systems

Dans ce didacticiel, vous allez apprendre à intégrer Pega Systems à Azure Active Directory (Azure AD).

Cette intégration offre les avantages suivants :

* Vous pouvez utiliser Azure AD pour contrôler qui a accès à Pega Systems.
* Vous pouvez permettre à vos utilisateurs d’être connectés automatiquement à Pega Systems (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS à Azure AD, consultez l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Pega Systems, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [essai gratuit d’un mois](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Pega Systems pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Pega Systems prend en charge l’authentification unique lancée par le fournisseur de services et le fournisseur d’identité.

## <a name="add-pega-systems-from-the-gallery"></a>Ajouter Pega Systems à partir de la galerie

Pour configurer l’intégration de Pega Systems à Azure AD, vous devez ajouter Pega Systems à partir de la galerie à votre liste d’applications SaaS managées.

1. Dans le [portail Azure](https://portal.azure.com), dans le volet de gauche, sélectionnez **Azure Active Directory**  :

    ![Sélectionnez Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** > **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la fenêtre :

    ![Sélectionner la nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Pega Systems** . Dans les résultats de la recherche, sélectionnez **Pega Systems** , puis **Ajouter** .

     ![Résultats de la recherche](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Pega Systems pour un utilisateur de test appelé Britta Simon.
Pour activer l’authentification unique, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur correspondant dans Pega Systems.

Pour configurer et tester l’authentification unique Azure AD avec Pega Systems, procédez comme suit :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser la fonctionnalité.
2. **[Configurer l’authentification unique Pega Systems](#configure-pega-systems-single-sign-on)** côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour lui permettre d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Pega Systems](#create-a-pega-systems-test-user)** qui est lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** pour vérifier que la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous allez activer l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Pega Systems, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Pega Systems** , sélectionnez **Authentification unique**  :

    ![Sélectionner l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** pour activer l’authentification unique :

    ![Sélectionner une méthode d’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**  :

    ![Icône Modifier](common/edit-urls.png)

4. Dans la boîte de dialogue **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le fournisseur d’identité, procédez comme suit.

    ![Boîte de dialogue Configuration SAML de base](common/idp-intiated.png)

    1. Dans la zone **Identificateur** , entrez une URL au format suivant :

       `https://<customername>.pegacloud.io:443/prweb/sp/<instanceID>`

    1. Dans la zone **URL de réponse** , entrez une URL au format suivant :

       `https://<customername>.pegacloud.io:443/prweb/PRRestService/WebSSO/SAML/AssertionConsumerService`

5. Si vous souhaitez configurer l’application en mode lancé par le fournisseur de services, sélectionnez **Définir des URL supplémentaires** , puis procédez comme suit.

    ![Informations d’authentification unique dans Domaine et URL Pega Systems](common/both-advanced-urls.png)

    1. Dans la zone **URL de connexion** , entrez la valeur de l’URL de connexion.

    1. Dans la zone **État de relais** , entrez une URL au format suivant : `https://<customername>.pegacloud.io/prweb/sso`

    > [!NOTE]
    > Les valeurs fournies ici sont des espaces réservés. Vous devez utiliser l’identificateur, l’URL de réponse, l’URL de connexion et l’URL d’état de relais exacts. Vous pouvez obtenir les valeurs de l’identificateur et de l’URL de réponse dans une application Pega, comme expliqué plus loin dans ce tutoriel. Pour obtenir la valeur d’état de relais, contactez le [équipe du support technique Pega Systems](https://www.pega.com/contact-us). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

6. L’application Pega Systems requiert les assertions SAML dans un format spécifique. Pour les obtenir au format qui convient, vous devez ajouter des mappages d’attributs personnalisés à la configuration d’attributs de jeton SAML. La capture d’écran suivante montre les attributs par défaut. Sélectionnez l’icône **Modifier** pour ouvrir la boîte de dialogue **Attributs d’utilisateur**  :

    ![Attributs utilisateur](common/edit-attribute.png)

7. En plus des attribués affichés dans la capture d’écran précédente, l’application Pega Systems requiert quelques attributs supplémentaires à repasser dans la réponse SAML. Dans la section **Revendications des utilisateurs** de la boîte de dialogue **Attributs utilisateur** , procédez comme suit pour ajouter ces attributs de jeton SAML :

    
   - `uid`
   - `cn`
   - `mail`
   - `accessgroup`  
   - `organization`  
   - `orgdivision`
   - `orgunit`
   - `workgroup`  
   - `Phone`

    > [!NOTE]
    > Ces valeurs sont spécifiques à votre organisation. Fournissez les valeurs appropriées.

    1. Sélectionnez **Ajouter une nouvelle revendication** pour ouvrir la boîte de dialogue **Gérer les revendications des utilisateurs**  :

    ![Sélectionner Ajouter une nouvelle revendication](common/new-save-attribute.png)

    ![Boîte de dialogue Gérer les revendications des utilisateurs](common/new-attribute-details.png)

    1. Dans la zone de texte **Nom** , entrez le nom d’attribut affiché pour cette ligne.

    1. Laissez la zone de texte **Espace de noms** vide.

    1. Pour la **Source** , sélectionnez **Attribut** .

    1. Dans la liste **Attribut de la source** , sélectionnez la valeur d’attribut indiquée pour cette ligne.

    1. Sélectionnez **OK** .

    1. Sélectionnez **Enregistrer** .

8. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , sélectionnez le lien **Télécharger** en regard de **XML de métadonnées de fédération** , selon vos besoins, puis enregistrez le certificat sur votre ordinateur :

    ![Lien de téléchargement du certificat](common/metadataxml.png)

9. Dans la section **Configurer Pega Systems** , copiez la ou les URL appropriées, selon vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    1. **URL de connexion** .

    1. **Identificateur Azure AD** .

    1. **URL de déconnexion** .

### <a name="configure-pega-systems-single-sign-on"></a>Configurer l’authentification unique Pega Systems

1. Pour configurer l’authentification unique du côté de **Pega Systems** , connectez-vous au portail Pega avec un compte administrateur dans une autre fenêtre de navigateur.

2. Sélectionnez **Créer** > **SysAdmin** > **Service d’authentification**  :

    ![Sélectionner Service d’authentification](./media/pegasystems-tutorial/tutorial_pegasystems_admin.png)
    
3. Effectuez les étapes suivantes sur l'écran **Créer le service d'authentification** .

    ![Écran Créer le service d'authentification](./media/pegasystems-tutorial/tutorial_pegasystems_admin1.png)

    1. Dans la liste **Type** , sélectionnez **SAML 2.0** .

    1. Dans la zone **Nom** , entrez un nom (par exemple **Authentification unique Azure AD** ).

    1. Dans la zone **Brève description** , entrez une description.  

    1. Sélectionnez **Créer et ouvrir** .
    
4. Dans la section **Informations sur le fournisseur d’identité (IdP)** , cliquez sur **Importer les métadonnées IdP** et explorez le fichier de métadonnées que vous avez téléchargé depuis le portail Azure. Cliquez sur **Envoyer** pour charger les métadonnées :

    ![Section d'informations sur le fournisseur d'identité (IdP)](./media/pegasystems-tutorial/tutorial_pegasystems_admin2.png)
    
    Vous renseignerez ainsi les données IdP comme illustré ici :

    ![Données IdP importées](./media/pegasystems-tutorial/tutorial_pegasystems_admin3.png)
    
6. Dans la section **Paramètres du fournisseur de services** , procédez comme suit.

    ![Paramètres du fournisseur de services](./media/pegasystems-tutorial/tutorial_pegasystems_admin4.png)

    1. Copiez la valeur **Entity Identification** , puis collez-la dans la zone **Identificateur** de la section **Configuration SAML de base** du portail Azure.

    1. Copiez la valeur **Emplacement de l'URL Assertion Consumer Service (ACS)** , puis collez-la dans la zone **URL de réponse** de la section **Configuration SAML de base** du portail Azure.

    1. Sélectionnez **Disable request signing** (Désactiver la signature de requête).

7. Sélectionnez **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet de gauche du portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs**  :

    ![Sélectionner Tous les utilisateurs](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran :

    ![Sélectionner Nouvel utilisateur](common/new-user.png)

3. Dans la boîte de dialogue **Utilisateur** , procédez comme suit.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans la zone **Nom** , entrez **BrittaSimon** .
  
    b. Dans la zone **Nom d’utilisateur** , entrez **brittasimon@\<yourcompanydomain>.\<extension>** . (Par exemple, BrittaSimon@contoso.com.)

    c. Sélectionnez **Afficher le mot de passe** , puis notez la valeur affichée dans la zone **Mot de passe** .

    d. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Pega Systems.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Pega Systems** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Pega Systems** .

    ![Liste des applications](common/all-applications.png)

3. Dans le volet gauche, sélectionnez **Utilisateurs et groupes**  :

    ![Sélectionner Utilisateurs et groupes](common/users-groups-blade.png)

4. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Sélectionnez Ajouter un utilisateur](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste des utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste. Cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Affecter** .

### <a name="create-a-pega-systems-test-user"></a>Créer un utilisateur test Pega Systems

Ensuite, vous devez créer un utilisateur nommé Britta Simon dans Pega Systems. Collaborez avec l'[équipe du support technique Pega Systems](https://www.pega.com/contact-us) pour créer des utilisateurs.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Vous devez maintenant tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous sélectionnez la vignette Pega Systems dans le volet d’accès, vous devez être connecté automatiquement à l’instance Pega Systems pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations, consultez [Accéder aux applications du portail Mes applications et les utiliser](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)