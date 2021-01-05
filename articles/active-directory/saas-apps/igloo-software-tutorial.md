---
title: 'Tutoriel : Intégration d’Azure Active Directory avec Igloo Software | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Igloo Software.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/06/2019
ms.author: jeedes
ms.openlocfilehash: 033561ac3c4a510927691dc8db4f61196f54ec2a
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92460385"
---
# <a name="tutorial-azure-active-directory-integration-with-igloo-software"></a>Tutoriel : Intégration d’Azure Active Directory avec Igloo Software

Dans ce didacticiel, vous allez apprendre à intégrer Igloo Software à Azure Active Directory (Azure AD).
L’intégration d’Igloo Software à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Igloo Software.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Igloo Software (par le biais de l’authentification unique) avec leurs comptes Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Igloo Software, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Igloo Software pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Igloo Software prend en charge l’authentification unique lancée par le **fournisseur de services**
* Igloo Software prend en charge l’attribution d’utilisateurs **juste-à-temps**

## <a name="adding-igloo-software-from-the-gallery"></a>Ajout d’Igloo Software à partir de la galerie

Pour configurer l’intégration d’Igloo Software à Azure AD, vous devez ajouter Igloo Software à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Igloo Software à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Igloo Software** , sélectionnez **Igloo Software** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Igloo Software dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Igloo Software à l’aide d’un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Igloo Software associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Igloo Software, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Igloo Software](#configure-igloo-software-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Igloo Software](#create-igloo-software-test-user)** pour avoir un équivalent de Britta Simon dans Igloo Software lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Igloo Software, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Igloo Software** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Igloo Software](common/sp-identifier-reply.png)

    a. Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<company name>.igloocommmunities.com`.

    b. Dans la zone de texte **Identificateur** , tapez une URL en utilisant le format suivant : `https://<company name>.igloocommmunities.com/saml.digest`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<company name>.igloocommmunities.com/saml.digest`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez [l’équipe de support technique Igloo Software](https://www.igloosoftware.com/services/support). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer Igloo Software** , copiez les URL appropriées, selon vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-igloo-software-single-sign-on"></a>Configurer l’authentification unique Igloo Software

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise Igloo Software en tant qu’administrateur.

2. Accédez à **Control panel** .

     ![Panneau de configuration](./media/igloo-software-tutorial/ic799949.png "Control panel")

3. Sous l’onglet **Membership** (Appartenance), cliquez sur **Paramètres de connexion** .

    ![Paramètres de connexion](./media/igloo-software-tutorial/ic783968.png "Sign In Settings")

4. Dans la section SAML Configuration, cliquez sur **Configure SAML Authentication** .

    ![Configuration SAML](./media/igloo-software-tutorial/ic783969.png "Configuration SAML")

5. Dans la section **General Configuration** , procédez comme suit :

    ![Configuration générale](./media/igloo-software-tutorial/ic783970.png "General Configuration")

    a. Dans la zone de texte **Connection Name** , entrez un nom personnalisé pour votre configuration.

    b. Dans la zone de texte **IdP Login URL** (URL de connexion IdP), collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.

    c. Dans la zone de texte **IdP Logout URL** (URL de déconnexion IdP), collez la valeur de l’ **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    d. Pour **Logout Response and Request HTTP Type** (Type de réponse de déconnexion et de requête HTTP), sélectionnez **POST** .

    e. Ouvrez dans le Bloc-notes votre certificat codé en **base 64** téléchargé à partir du portail Azure, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **Certificat public** .

6. Dans **Response and Authentication Configuration** , procédez comme suit :

    ![Configuration de la réponse et de l’authentification](./media/igloo-software-tutorial/IC783971.png "Response and Authentication Configuration")
  
    a. Pour **Fournisseur d’identité** , sélectionnez **Microsoft ADFS** .

    b. Pour **Type d’identificateur** , sélectionnez **Adresse de messagerie** . 

    c. Dans la zone de texte **Email Attribute** , entrez **emailaddress** .

    d. Dans la zone de texte **Attribut de prénom** , saisissez **givenname** .

    e. Dans la zone de texte **Attribut de nom** , saisissez **surname** .

7. Pour terminer la configuration, procédez comme suit :

    ![Création de l’utilisateur lors de la connexion](./media/igloo-software-tutorial/IC783972.png "Création d’utilisateurs à l’authentification") 

    a. Pour **Création d’utilisateurs à l’authentification** , sélectionnez **Create a new user in your site when they sign in** (Créer un nouvel utilisateur sur votre site au moment de la connexion).

    b. Pour **Paramètres de connexion** , sélectionnez **Use SAML button on “Sign in” screen** (Utiliser le bouton SAML sur l’écran de connexion).

    c. Cliquez sur **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon@yourcompanydomain.extension**  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Igloo Software.

1. Dans le Portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Igloo Software** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Igloo Software** .

    ![Lien Igloo Software dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-igloo-software-test-user"></a>Créer un utilisateur de test Igloo Software

Aucun élément d’action ne vous permet de configurer l’approvisionnement des utilisateurs dans Igloo Software.  

Lorsqu’un utilisateur assigné tente de se connecter à Igloo Software à l’aide du panneau d’accès, Igloo Software vérifie si cet utilisateur existe.  Si aucun compte d’utilisateur n’est disponible, Igloo Software le crée automatiquement.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Igloo Software dans le volet d’accès, vous devez être automatiquement connecté à l’application Igloo Software pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)