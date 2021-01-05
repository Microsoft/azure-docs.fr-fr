---
title: 'Didacticiel : Intégration d’Azure Active Directory avec Panorama9 | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Panorama9.
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
ms.openlocfilehash: e79d8584a7c760fa0a50a614390370ee7ff8ae15
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92512478"
---
# <a name="tutorial-azure-active-directory-integration-with-panorama9"></a>Didacticiel : Intégration d’Azure Active Directory avec Panorama9

Dans ce didacticiel, vous allez apprendre à intégrer Panorama9 avec Azure Active Directory (Azure AD).
L’intégration de Panorama9 avec Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Panorama9.
* Vous pouvez permettre à vos utilisateurs d’être automatiquement connectés à Panorama9 (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec Panorama9, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Panorama9 pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Panorama9 prend en charge l’authentification unique lancée par le **fournisseur de services** .

## <a name="adding-panorama9-from-the-gallery"></a>Ajout de Panorama9 à partir de la galerie

Pour configurer l’intégration de Panorama9 avec Azure AD, vous devez ajouter Panorama9 à partir de la galerie à votre liste d’applications SaaS managées.

**Pour ajouter Panorama9 à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Panorama9** , sélectionnez **Panorama9** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Panorama9 dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Panorama9 sur un utilisateur de test nommé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur Panorama9 associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Panorama9, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Panorama9](#configure-panorama9-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Panorama9](#create-panorama9-test-user)** pour obtenir un équivalent de Britta Simon dans Panorama9 lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Panorama9, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Panorama9** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Panorama9](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion** , tapez une URL : `https://dashboard.panorama9.com/saml/access/3262`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://www.panorama9.com/saml20/<tenant-name>`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique Panorama9](https://support.panorama9.com/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la section **Certificat de signature SAML** , cliquez sur le bouton **Modifier** pour ouvrir la boîte de dialogue **Certificat de signature SAML** .

    ![Modifier le certificat de signature SAML](common/edit-certificate.png)

6. Dans la section **Certificat de signature SAML** , copiez l’ **empreinte** et enregistrez-la sur votre ordinateur.

    ![Copier la valeur de l’empreinte](common/copy-thumbprint.png)

7. Dans la section **Configurer Panorama9** , copiez l’URL ou les URL appropriées, en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-panorama9-single-sign-on"></a>Configurer l’authentification unique Panorama9

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise Panorama9 en tant qu’administrateur.

2. Dans la barre d’outils située en haut, cliquez sur **Manage** , puis sur **Extensions** .
   
    ![Extensions](./media/panorama9-tutorial/ic790023.png "Extensions")

3. Dans la boîte de dialogue **Extensions** , cliquez sur **Single Sign-On** .
   
    ![Authentification unique](./media/panorama9-tutorial/ic790024.png "Single Sign on")

4. Dans la section **Settings** , procédez comme suit :
   
    ![Paramètres](./media/panorama9-tutorial/ic790025.png "Paramètres")
   
    a. Dans la zone de texte **Identity provider URL** (URL du fournisseur d’identité), collez la valeur de l’ **URL de connexion** que vous avez copiée à partir du portail Azure.
   
    b. Dans la zone de texte **Empreinte du certificat** , collez la valeur du certificat **Empreinte** que vous avez copiée à partir du portail Azure.    
         
5. Cliquez sur **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez brittasimon@yourcompanydomain.extension. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Panorama9.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Panorama9** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Panorama9** .

    ![Lien Panorama9 dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-panorama9-test-user"></a>Créer un utilisateur de test Panorama9

Pour permettre aux utilisateurs Azure AD de se connecter à Panorama9, vous devez les provisionner dans Panorama9.  

Dans le cas de Panorama9, l’approvisionnement est une tâche manuelle.

**Pour configurer l'approvisionnement des utilisateurs, procédez comme suit :**

1. Connectez-vous au site d’entreprise **Panorama9** en tant qu’administrateur.

2. Dans le menu situé en haut, cliquez sur **Manage** , puis sur **Users** .
   
    ![Capture d’écran montrant les onglets « Manage » et « Users » sélectionnés.](./media/panorama9-tutorial/ic790027.png "Utilisateurs")

3. Dans la section Utilisateurs, cliquez sur **+** pour ajouter un nouvel utilisateur.

    ![Utilisateurs](./media/panorama9-tutorial/ic790028.png "Utilisateurs")

4. Accédez à la section Données utilisateur, puis, dans la zone de texte **E-mail** , entrez l’adresse e-mail d’un utilisateur Azure Active Directory valide que vous souhaitez approvisionner.

5. Accédez à la section Utilisateurs, puis cliquez sur **Enregistrer** .
   
    > [!NOTE]
    > Le titulaire du compte Azure Active Directory reçoit un e-mail contenant un lien à suivre pour confirmer son compte et l’activer.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Panorama9 dans le volet d’accès doit vous connecter automatiquement à l’application Panorama9 pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)