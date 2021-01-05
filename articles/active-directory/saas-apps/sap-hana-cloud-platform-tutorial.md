---
title: 'Didacticiel : Intégration d’Azure Active Directory à SAP Cloud Platform | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SAP Cloud Platform.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/17/2018
ms.author: jeedes
ms.openlocfilehash: 2d87be085331c174b58e2a475e18efa54a78cfea
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895158"
---
# <a name="tutorial-azure-active-directory-integration-with-sap-cloud-platform"></a>Didacticiel : Intégration d’Azure Active Directory avec SAP Cloud Platform

Dans ce didacticiel, vous allez apprendre à intégrer SAP Cloud Platform à Azure Active Directory (Azure AD).
L’intégration de SAP Cloud Platform à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à SAP Cloud Platform.
* Vous pouvez autoriser les utilisateurs à se connecter automatiquement à SAP Cloud Platform (via l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à SAP Cloud Platform, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement SAP Cloud Platform pour lequel l’authentification unique est activée

À l’issue de ce didacticiel, les utilisateurs d’Azure AD que vous avez affectés à SAP Cloud Platform pourront s’authentifier de manière unique dans l’application (connexion initiée par le fournisseur du service) à l’aide de la [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

>[!IMPORTANT]
>Vous devez déployer votre propre application ou vous abonner à une application sur votre compte SAP Cloud Platform pour tester l’authentification unique. Dans ce didacticiel, une application est déployée dans le compte.
> 

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* SAP Cloud Platform prend en charge l’authentification unique initiée par **SP**

## <a name="adding-sap-cloud-platform-from-the-gallery"></a>Ajout de SAP Cloud Platform à partir de la galerie

Pour configurer l’intégration de SAP Cloud Platform à Azure AD, vous devez ajouter SAP Cloud Platform disponible dans la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter SAP Cloud Platform à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **SAP Cloud Platform** , sélectionnez **SAP Cloud Platform** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![SAP Cloud Platform dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec SAP Cloud Platform avec un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur SAP Cloud Platform associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec SAP Cloud Platform, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique SAP Cloud Platform](#configure-sap-cloud-platform-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur test SAP Cloud Platform](#create-sap-cloud-platform-test-user)** pour avoir un équivalent de Britta Simon dans SAP Cloud Platform lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec SAP Cloud Platform, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **SAP Cloud Platform** , sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL SAP Cloud Platform](common/sp-identifier-reply.png)

    a. Dans la zone de texte **URL d’authentification** , entrez l’URL utilisée par vos utilisateurs pour se connecter à votre application **SAP Cloud Platform**. Il s’agit de l’URL spécifique au compte d’une ressource protégée de votre application SAP Cloud Platform. L’URL est basée sur le modèle suivant : `https://<applicationName><accountName>.<landscape host>.ondemand.com/<path_to_protected_resource>`
      
    >[!NOTE]
    >Il s’agit de l’URL de votre application SAP Cloud Platform sur laquelle l’utilisateur doit s’authentifier.
    > 

    - `https://<subdomain>.hanatrial.ondemand.com/<instancename>`
    - `https://<subdomain>.hana.ondemand.com/<instancename>`

    b. Dans la zone de texte **Identificateur** , vous indiquez le type de votre SAP Cloud Platform. Tapez une URL selon l’un des modèles suivants : 

    - `https://hanatrial.ondemand.com/<instancename>`
    - `https://hana.ondemand.com/<instancename>`
    - `https://us1.hana.ondemand.com/<instancename>`
    - `https://ap1.hana.ondemand.com/<instancename>`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant :

    - `https://<subdomain>.hanatrial.ondemand.com/<instancename>`
    - `https://<subdomain>.hana.ondemand.com/<instancename>`
    - `https://<subdomain>.us1.hana.ondemand.com/<instancename>`
    - `https://<subdomain>.dispatcher.us1.hana.ondemand.com/<instancename>`
    - `https://<subdomain>.ap1.hana.ondemand.com/<instancename>`
    - `https://<subdomain>.dispatcher.ap1.hana.ondemand.com/<instancename>`
    - `https://<subdomain>.dispatcher.hana.ondemand.com/<instancename>`

    > [!NOTE] 
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Contactez l’[équipe de support client SAP Cloud Platform](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/5dd739823b824b539eee47b7860a00be.html) pour obtenir l’URL de connexion et l’identificateur. URL de réponse que vous obtenez dans la section Gestion des approbations qui est expliquée plus loin dans le didacticiel.
    > 
4. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

### <a name="configure-sap-cloud-platform-single-sign-on"></a>Configurer l’authentification unique SAP Cloud Platform

1. Dans une autre fenêtre de navigateur web, connectez-vous à SAP Cloud Platform Cockpit à l’adresse `https://account.<landscape host>.ondemand.com/cockpit`(par exemple : https://account.hanatrial.ondemand.com/cockpit).

2. Cliquez sur l’onglet **Trust** .
   
    ![Approbation](./media/sap-hana-cloud-platform-tutorial/ic790800.png "Trust")

3. Dans la section Gestion des approbations, sous **Fournisseur de services local** , procédez comme suit :

    ![Capture d’écran montrant la section « Trust Management » avec l’onglet « Local Service Provider » sélectionné et toutes les zones de texte mises en évidence.](./media/sap-hana-cloud-platform-tutorial/ic793931.png "Gestion des approbations")
   
    a. Cliquez sur **Modifier**.

    b. Comme **Configuration Type** , sélectionnez **Custom**.

    c. Pour **Local Provider Name** , laissez la valeur par défaut. Copiez cette valeur et collez-la dans le champ **Identificateur** dans la configuration d’Azure AD pour SAP Cloud Platform.

    d. Pour générer une paire de clés **Signature Key** et **Signing Certificate** , cliquez sur **Generate Key Pair**.

    e. Pour **Principal Propagation** , sélectionnez **Disabled**.

    f. Pour **Force Authentication** , sélectionnez **Disabled**.

    g. Cliquez sur **Enregistrer**.

4. Une fois les paramètres **Fournisseur de services local** enregistrés, procédez comme suit pour obtenir l’URL de réponse :
   
    ![Obtenir les métadonnées](./media/sap-hana-cloud-platform-tutorial/ic793930.png "Get Metadata")

    a. Téléchargez le fichier de métadonnées SAP Cloud Platform en cliquant sur **Obtenir les métadonnées**.

    b. Ouvrez le fichier XML de métadonnées SAP Cloud Platform téléchargé et recherchez la balise **ns3:AssertionConsumerService** .
 
    c. Copiez la valeur de l’attribut **Emplacement** et collez-la dans le champ **URL de réponse** dans la configuration d’Azure AD pour SAP Cloud Platform.

5. Cliquez sur l’onglet **Trusted Identity Provider** , puis sur **Add Trusted Identity Provider**.
   
    ![Capture d’écran montrant la page « Trust Management » avec l’onglet « Trusted Identity Provider » sélectionné.](./media/sap-hana-cloud-platform-tutorial/ic790802.png "Gestion des approbations")
   
    >[!NOTE]
    >Pour gérer la liste de fournisseurs d’identité approuvés, vous devrez avoir choisi le type de configuration Custom dans la section Local Service Provider. Pour le type de configuration Default, vous disposez d’une approbation non modifiable et implicite au SAP ID Service. Pour None, vous n’avez aucun paramètre d’approbation.
    > 
    > 

6. Cliquez sur l’onglet **General** , puis sur **Browse** pour charger le fichier de métadonnées que vous avez téléchargé.
    
    ![Gestion des approbations](./media/sap-hana-cloud-platform-tutorial/ic793932.png "Gestion des approbations")
    
    >[!NOTE]
    >Après avoir téléchargé le fichier de métadonnées, les valeurs de **Authentification unique** , **URL de déconnexion unique** et de **Certificat de signature** sont remplies automatiquement.
    > 
     
7. Cliquez sur onglet **Attributes** .

8. Sous l’onglet **Attributes** , procédez comme suit :
    
    ![Attributs](./media/sap-hana-cloud-platform-tutorial/ic790804.png "Attributs") 

    a. Cliquez sur **Add Assertion-Based Attribute** , puis ajoutez les attributs basés sur une assertion suivants :
       
    | Assertion Attribute | Principal Attribute |
    | --- | --- |
    | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname` |firstname |
    | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname` |lastname |
    | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` |email |
   
     >[!NOTE]
     >La configuration des attributs dépend de la façon dont sont développées les applications sur SCP, en l’occurrence des attributs qu’elles attendent dans la réponse SAML et avec quel nom (Principal Attribute) elles accèdent à cet attribut dans le code.
     > 
    
    b. L’attribut **Default Attribute** de la capture d’écran ne sert qu’à des fins d’illustration. Il n’est pas nécessaire de faire fonctionner le scénario.  
 
    c. Les noms et valeurs de **Principal Attribute** dans la capture d’écran dépendent de la façon dont l’application est développée. Il est possible que votre application exige des mappages différents.

### <a name="assertion-based-groups"></a>Groupes basés sur une assertion

Une étape facultative est de configurer des groupes basés sur une assertion pour votre fournisseur d’identité Azure Active Directory.

L’utilisation de SAP Cloud Platform vous permet d’attribuer de manière dynamique un ou plusieurs utilisateurs à un ou plusieurs rôles dans vos applications SAP Cloud Platform, en fonction des valeurs des attributs de l’assertion SAML 2.0. 

Par exemple, si l’assertion contient l’attribut «  *contract=temporaire*  », vous souhaiterez peut-être que tous les utilisateurs affectés soient ajoutés au groupe «  *TEMPORAIRE*  ». Le groupe « *TEMPORAIRE* » peut contenir un ou plusieurs rôles d’une ou plusieurs applications déployées dans votre compte SAP Cloud Platform.
 
Utilisez des groupes basés sur une assertion quand vous voulez affecter simultanément plusieurs utilisateurs à un ou plusieurs rôles d’applications dans votre compte SAP Cloud Platform. Si vous ne voulez affecter qu’un seul utilisateur ou un petit nombre d’utilisateurs à des rôles spécifiques, nous vous recommandons de les affecter directement dans l’onglet « **Autorisations** » de SAP Cloud Platform Cockpit.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon**.
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon\@domainedevotreentreprise.extension**.  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à SAP Cloud Platform.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis sélectionnez **SAP Cloud Platform**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, tapez et sélectionnez **SAP Cloud Platform**.

    ![Lien SAP Cloud Platform dans la liste Applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

### <a name="create-sap-cloud-platform-test-user"></a>Créer un utilisateur test SAP Cloud Platform

Pour permettre aux utilisateurs d’Azure AD de se connecter à SAP Cloud Platform, vous devez leur attribuer des rôles dans SAP Cloud Platform.

**Pour affecter un rôle à un utilisateur, procédez comme suit :**

1. Connectez-vous à votre cockpit **SAP Cloud Platform**.

2. Procédez comme suit :
   
    ![Autorisations](./media/sap-hana-cloud-platform-tutorial/ic790805.png "Autorisations")
   
    a. Cliquez sur **Authorization**.

    b. Cliquez sur l’onglet **Users** .

    c. Dans la zone de texte **User** , tapez l’adresse de messagerie de l’utilisateur.

    d. Cliquez sur **Assign** pour attribuer l’utilisateur à un rôle.

    e. Cliquez sur **Enregistrer**.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la vignette SAP Cloud Platform dans le volet d’accès, vous devez être connecté automatiquement à l’application SAP Cloud Platform pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)