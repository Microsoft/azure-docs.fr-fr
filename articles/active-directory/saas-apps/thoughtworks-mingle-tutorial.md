---
title: 'Didacticiel : intégration d’Azure Active Directory à Thoughtworks Mingle | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Thoughtworks Mingle.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: c3eeb1577e628965e3e5a35fa20c072224383149
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92514620"
---
# <a name="tutorial-azure-active-directory-integration-with-thoughtworks-mingle"></a>Didacticiel : Intégration d’Azure AD à Thoughtworks Mingle

Dans ce didacticiel, vous allez apprendre à intégrer Thoughtworks Mingle à Azure Active Directory (Azure AD).
L’intégration de Thoughtworks Mingle à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Thoughtworks Mingle.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Thoughtworks Mingle (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Thoughtworks Mingle, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Thoughtworks Mingle pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Thoughtworks Mingle prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-thoughtworks-mingle-from-the-gallery"></a>Ajout de Thoughtworks Mingle à partir de la galerie

Pour configurer l’intégration de Thoughtworks Mingle à Azure AD, vous devez ajouter Thoughtworks Mingle depuis la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Thoughtworks Mingle à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Thoughtworks Mingle** , sélectionnez **Thoughtworks Mingle** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Thoughtworks Mingle dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Thoughtworks Mingle avec un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Thoughtworks Mingle associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Thoughtworks Mingle, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Thoughtworks Mingle](#configure-thoughtworks-mingle-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Thoughtworks Mingle](#create-thoughtworks-mingle-test-user)** pour avoir un équivalent de Britta Simon dans Thoughtworks Mingle lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Thoughtworks Mingle, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Thoughtworks Mingle** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Thoughtworks Mingle](common/sp-signonurl.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<companyname>.mingle.thoughtworks.com`

    > [!NOTE]
    > Cette valeur n’est pas la valeur réelle. Mettez à jour la valeur avec l’URL de connexion réelle. Pour obtenir cette valeur, contactez [l’équipe de support technique Thoughtworks Mingle](https://support.thoughtworks.com/hc/categories/201743486-Mingle-Community-Support). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configurer Thoughtworks Mingle** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-thoughtworks-mingle-single-sign-on"></a>Configurer l’authentification unique Thoughtworks Mingle

1. Connectez-vous à votre site d’entreprise **Thoughtworks Mingle** en tant qu’administrateur.

2. Cliquez sur l’onglet **Admin** , puis sur **SSO Config (Configuration SSO)** .
   
    ![Onglet Admin](./media/thoughtworks-mingle-tutorial/ic785157.png "SSO Config")

3. Dans la section **SSO Config** , procédez comme suit :
   
    ![SSO Config](./media/thoughtworks-mingle-tutorial/ic785158.png "SSO Config") (Configuration de l’authentification unique)
    
    a. Pour charger le fichier de métadonnées, cliquez sur **Choose file** . 

    b. Cliquez sur **Enregistrer les modifications** .

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Thoughtworks Mingle.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Thoughtworks Mingle** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Thoughtworks Mingle** .

    ![Lien Thoughtworks Mingle dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-thoughtworks-mingle-test-user"></a>Créer un utilisateur de test Thoughtworks Mingle

Pour que les utilisateurs Azure AD puissent se connecter, ils doivent être approvisionnés dans l’application Thoughtworks Mingle à l’aide de leurs noms d’utilisateur Azure AD. Dans le cas de Thoughtworks Mingle, l’approvisionnement est une tâche manuelle.

**Pour configurer l'approvisionnement des utilisateurs, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise Thoughtworks Mingle en tant qu’administrateur.

2. Cliquez sur **Profile** .
   
    ![Your First Project](./media/thoughtworks-mingle-tutorial/ic785160.png "Votre premier projet") (Votre premier projet)

3. Cliquez sur l’onglet **Admin** , puis sur **Users (Utilisateurs)** .
   
    ![Utilisateurs](./media/thoughtworks-mingle-tutorial/ic785161.png "Utilisateurs")

4. Cliquez sur **New User** .
   
    ![Nouvel utilisateur](./media/thoughtworks-mingle-tutorial/ic785162.png "Nouvel utilisateur")

5. Dans la boîte de dialogue **New User** , procédez comme suit :
   
    ![Boîte de dialogue New User](./media/thoughtworks-mingle-tutorial/ic785163.png "Nouvel utilisateur") (Nouvel utilisateur)  
 
    a. Renseignez les zones de texte **Sign-in name** (Nom de connexion), **Display name** (Nom d’affichage), **Choose password** (Choisir le mot de passe) et **Confirm password** (Confirmer le mot de passe) du compte Azure AD valide que vous souhaitez approvisionner. 

    b. Dans **User type (Type d’utilisateur)** , sélectionnez **Full user (Utilisateur complet)** .

    c. Cliquez sur **Create This Profile** .

>[!NOTE]
>Vous pouvez utiliser tout autre outil ou n’importe quelle API de création de compte d’utilisateur fournis par Thoughtworks Mingle pour provisionner des comptes d’utilisateurs Azure AD.
> 

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Thoughtworks Mingle dans le volet d’accès, vous devez être connecté automatiquement à l’application Thoughtworks Mingle pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)