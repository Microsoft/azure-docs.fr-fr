---
title: 'Didacticiel : Intégration d’Azure Active Directory avec Picturepark | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Picturepark.
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
ms.openlocfilehash: 8a00cf11edfea2e732a18a392d465525b38ea45f
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92520844"
---
# <a name="tutorial-azure-active-directory-integration-with-picturepark"></a>Didacticiel : Intégration d’Azure Active Directory avec Picturepark

Ce didacticiel explique comment intégrer Picturepark avec Azure Active Directory (Azure AD).
L’intégration de Picturepark avec Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Picturepark.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Picturepark (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec Picturepark, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Abonnement Picturepark pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Picturepark prend en charge l’authentification unique initiée par le **fournisseur de services**

## <a name="adding-picturepark-from-the-gallery"></a>Ajout de Picturepark à partir de la galerie

Pour configurer l’intégration de Picturepark à Azure AD, vous devez ajouter Picturepark à partir de la galerie à votre liste d’applications SaaS managées.

**Pour ajouter Picturepark à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Picturepark** , sélectionnez **Picturepark** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Picturepark dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Picturepark sur un utilisateur de test nommé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur Picturepark associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Picturepark, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Picturepark](#configure-picturepark-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Picturepark](#create-picturepark-test-user)** pour avoir un équivalent de Britta Simon dans Picturepark, lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Picturepark, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), sur la page d’intégration de l’application **Picturepark** , cliquez sur **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Picturepark](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<companyname>.picturepark.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant :

    ```http
        https://<companyname>.current-picturepark.com
        https://<companyname>.picturepark.com
        https://<companyname>.next-picturepark.com
    ```

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique Picturepark](https://picturepark.com/company/picturepark-customer-support). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la section **Certificat de signature SAML** , cliquez sur le bouton **Modifier** pour ouvrir la boîte de dialogue **Certificat de signature SAML** .

    ![Modifier le certificat de signature SAML](common/edit-certificate.png)

6. Dans la section **Certificat de signature SAML** , copiez l’ **empreinte** et enregistrez-la sur votre ordinateur.

    ![Copier la valeur de l’empreinte](common/copy-thumbprint.png)

7. Dans la section **Configurer Picturepark** , copiez la ou les URL appropriées correspondant à vos besoins. Pour **Login URL (URL de connexion)** , utilisez la valeur au format suivant : `https://login.microsoftonline.com/_my_directory_id_/wsfed`

    > [!Note]
    > _my_directory_id_ est l’ID de locataire de l’abonnement Azure AD.

    ![Copier les URL de configuration](./media/picturepark-tutorial/configurls.png)

    a. Identificateur Azure AD

    b. URL de déconnexion

### <a name="configure-picturepark-single-sign-on"></a>Configurer l’authentification unique Picturepark

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise Picturepark en tant qu’administrateur.

2. Dans la barre d’outils située en haut, cliquez sur **Administrative tools** , puis sur **Management Console** .
   
    ![Console de gestion](./media/picturepark-tutorial/ic795062.png "Management Console")

3. Cliquez sur **Authentication** , puis sur **Identity providers** .
   
    ![Authentification](./media/picturepark-tutorial/ic795063.png "Authentication")

4. Dans la section **Identity provider configuration** , procédez comme suit :
   
    ![Configuration du fournisseur d’identité](./media/picturepark-tutorial/ic795064.png "Identity provider configuration")
   
    a. Cliquez sur **Ajouter** .
  
    b. Entrez un nom pour votre configuration.
   
    c. Sélectionnez **Set as default** .
   
    d. Dans la zone de texte **Issuer URI (URI de l’émetteur)** , collez la valeur **Login URL (URL de connexion)** , que vous avez copiée dans le portail Azure.
   
    e. Dans la zone de texte **Trusted Issuer Thumb Print** (Empreinte numérique de l’émetteur approuvé), collez la valeur de l’ **empreinte** que vous avez copiée à partir de la section **Certificat de signature SAML** . 

5. Cliquez sur **JoinDefaultUsersGroup** .

6. Pour définir l’attribut **Emailaddress** dans la zone de texte **Revendication** , tapez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`, puis cliquez sur **Enregistrer** .

      ![Configuration](./media/picturepark-tutorial/ic795065.png "Configuration")

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez `brittasimon@yourcompanydomain.extension`. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Picturepark.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Picturepark** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Picturepark** .

    ![Lien Picturepark dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-picturepark-test-user"></a>Créer un utilisateur de test Picturepark

Pour permettre aux utilisateurs Azure AD de se connecter à Picturepark, vous devez les approvisionner dans Picturepark. Dans le cas de Picturepark, l’approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre locataire **Picturepark** .

1. Dans la barre d’outils située en haut, cliquez sur **Administrative tools** , puis sur **Users** .
   
    ![Utilisateurs](./media/picturepark-tutorial/ic795067.png "Utilisateurs")

1. Dans l’onglet **Users overview** , cliquez sur **New** .
   
    ![Gestion des utilisateurs](./media/picturepark-tutorial/ic795068.png "User Management")

1. Dans la boîte de dialogue **Créer un utilisateur** , procédez comme suit pour un utilisateur Azure Active Directory valide que vous souhaitez approvisionner :
   
    ![Create User](./media/picturepark-tutorial/ic795069.png "Create User") (Créer un utilisateur)
   
    a. Dans la zone de texte **Email Address** (Adresse e-mail), entrez l’ **adresse e-mail** de l’utilisateur `BrittaSimon@contoso.com`.  
   
    b. Dans les zones de texte **Password** (Mot de passe) et **Confirm Password** (Confirmer le mot de passe), entrez le **mot de passe** de BrittaSimon. 
   
    c. Dans la zone de texte **First Name** (Prénom), entrez le **prénom** de l’utilisateur **Britta** . 
   
    d. Dans la zone de texte **Last Name** (Nom), entrez le **nom** de l’utilisateur **Simon** .
   
    e. Dans la zone de texte **Company** (Société), entrez le **nom de la société** de l’utilisateur. 
   
    f. Dans la zone de texte **Country** (Pays), sélectionnez le/la **pays/région** de l’utilisateur.
  
    g. Dans la zone de texte **ZIP** (Code postal), entrez le **code postal** de la ville.
   
    h. Dans la zone de texte **City** (Ville), entrez le **nom de la ville** de l’utilisateur.

    i. Sélectionnez une langue dans **Language** .
   
    j. Cliquez sur **Créer** .

>[!NOTE]
>Vous pouvez utiliser n’importe quel outil ou API de création de compte utilisateur fournis par Picturepark pour approvisionner des comptes utilisateur Azure AD.
>

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Picturepark dans le panneau d’accès doit vous connecter automatiquement à l’application Picturepark pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)