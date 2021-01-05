---
title: 'Didacticiel : Intégration d’Azure Active Directory à Perception United States (Non-UltiPro) | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Perception United States (Non-UltiPro).
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
ms.openlocfilehash: 3fc16a869c176074467a0b21880937d7753602b2
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92515523"
---
# <a name="tutorial-azure-active-directory-integration-with-perception-united-states-non-ultipro"></a>Didacticiel : Intégration d’Azure Active Directory à Perception United States (Non-UltiPro)

Dans ce didacticiel, vous allez apprendre à intégrer Perception United States (Non-UltiPro) à Azure Active Directory (Azure AD).
L’intégration de Perception United States (Non-UltiPro) à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Perception United States (Non-UltiPro).
* Vous pouvez autoriser les utilisateurs à se connecter automatiquement à Perception United States (Non-UltiPro) (via l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec Perception United States (Non-UltiPro), vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Perception United States (Non-UltiPro) pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Perception United States (Non-UltiPro) prend en charge l’authentification unique lancée par le **fournisseur d’identité**

## <a name="adding-perception-united-states-non-ultipro-from-the-gallery"></a>Ajout de Perception United States (Non-UltiPro) à partir de la galerie

Pour configurer l’intégration de Perception United States (Non-UltiPro) à Azure AD, vous devez ajouter Perception United States (Non-UltiPro) à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Perception United States (Non-UltiPro) à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Perception United States (Non-UltiPro)** , sélectionnez **Perception United States (Non-UltiPro)** à partir du volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Perception United States (Non-UltiPro) dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Perception United States (Non-UltiPro) grâce à un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur associé dans Perception United States (Non-UltiPro) doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Perception United States (Non-UltiPro), vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Perception United States (Non-UltiPro)](#configure-perception-united-states-non-ultipro-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Perception United States (Non-UltiPro)](#create-perception-united-states-non-ultipro-test-user)** pour avoir dans Perception United States (Non-UltiPro) un équivalent de Britta Simon lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique d’Azure AD avec Perception United States (Non-UltiPro), procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), sur la page d’intégration de l’application **Perception United States (Non-UltiPro)** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Sur la page **Configurer l’authentification unique avec SAML** , effectuez les étapes suivantes :

    ![Informations d’authentification Domaine et URL Perception United States (Non-UltiPro)](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL : `https://perception.kanjoya.com/sp`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://perception.kanjoya.com/sso?idp=<entity_id>`

    c. L’application **Perception United States (Non-UltiPro)** requiert la valeur de l’ **identificateur Azure AD** en tant que <entity_id>, que vous obtiendrez à partir de la section **Configurer Perception United States (Non-UltiPro)** , à encoder dans l’URI. Pour obtenir la valeur encodée sous forme d’URI, utilisez le lien suivant : **http://www.url-encode-decode.com/** .

    d. Après l’obtention de la valeur encodée en URI, associez-la à l’ **URL de réponse** comme indiqué ci-dessous :

    `https://perception.kanjoya.com/sso?idp=<URI encooded entity_id>`
    
    e. Collez la valeur ci-dessus dans la zone de texte **URL de réponse** .

5. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configurer Perception United States (Non-UltiPro)** , copiez la ou les URL appropriées correspondant à vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion   

### <a name="configure-perception-united-states-non-ultipro-single-sign-on"></a>Un abonnement Perception United States (Non-UltiPro) pour lequel l’authentification unique est activée

1. Dans une autre fenêtre de navigateur, connectez-vous à votre site d’entreprise Perception United States (Non-UltiPro) en tant qu’administrateur.

2. Dans la barre d’outils principale, cliquez sur **Paramètres du compte** .

    ![Capture d’écran montrant l’élément « Paramètres du compte » sélectionné dans la barre d’outils principale.](./media/perceptionunitedstates-tutorial/tutorial_perceptionunitedstates_user.png)

3. Sur la page **Paramètres du compte** , procédez comme suit :

    ![Utilisateur Perception United States (Non-UltiPro)](./media/perceptionunitedstates-tutorial/tutorial_perceptionunitedstates_account.png)

    a. Dans la zone de texte **Nom de l’entreprise** , entrez le nom de l’ **entreprise** .
    
    b. Dans la zone de texte **Nom du compte** , entrez le nom du **compte** .

    c. Dans la zone de texte **Default Reply-To Email** (Adresse électronique de réponse par défaut), entrez une **adresse e-mail** valide.

    d. Sélectionnez le **fournisseur d’identité d’authentification unique** **SAML 2.0** .

4. Dans la page **Configuration SAML** , procédez comme suit :

    ![Configuration de l’authentification unique Perception United States (Non-UltiPro)](./media/perceptionunitedstates-tutorial/tutorial_perceptionunitedstates_ssoconfig.png)

    a. Sélectionnez le **type de NameID SAML** **E-MAIL** .

    b. Dans la zone de texte **Nom de configuration d’authentification unique** , tapez le nom de votre **configuration** .
    
    c. Dans la zone de texte **Identity Provider Name** (Nom du fournisseur d’identité), collez la valeur de l’ **identificateur Azure AD** que vous avez copiée à partir du portail Azure. 

    d. Dans la **zone de texte de domaine SAML** , entrez le domaine tel que @contoso.com.

    e. Cliquez sur **Télécharger à nouveau** pour télécharger le fichier de **XML de métadonnées** .

    f. Cliquez sur **Update** .

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Perception United States (Non-UltiPro).

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , sélectionnez **Toutes les applications** , puis sélectionnez **Perception United States (Non-UltiPro)** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Perception United States (Non-UltiPro)** .

    ![Lien Perception United States (Non-UltiPro) dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-perception-united-states-non-ultipro-test-user"></a>Créer un utilisateur de test Perception United States (Non-UltiPro)

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans Perception United States (Non-UltiPro). Collaborez avec l’[équipe de support Perception United States (Non-UltiPro)](https://www.ultimatesoftware.com/Contact/ContactUs) pour ajouter les utilisateurs dans la plateforme Perception United States (Non-UltiPro).

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Perception United States (Non-UltiPro) dans le volet d’accès, vous devez être connecté automatiquement à l’application Perception United States (Non-UltiPro) pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)