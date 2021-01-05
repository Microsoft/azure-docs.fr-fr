---
title: 'Didacticiel : intégration d’Azure Active Directory à Tangoe Command Premium Mobile | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Tangoe Command Premium Mobile.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/10/2019
ms.author: jeedes
ms.openlocfilehash: fd2923c68aa77a4d17936eaa3a738c963458e8cb
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92514663"
---
# <a name="tutorial-azure-active-directory-integration-with-tangoe-command-premium-mobile"></a>Didacticiel : Intégration d’Azure Active Directory à Tangoe Command Premium Mobile

Dans ce didacticiel, vous allez apprendre à intégrer Tangoe Command Premium Mobile à Azure Active Directory (Azure AD).
L’intégration de Tangoe Command Premium Mobile à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez déterminer qui a accès à Tangoe Command Premium Mobile.
* Vous pouvez autoriser les utilisateurs à se connecter automatiquement à Tangoe Command Premium Mobile (via l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Tangoe Command Premium Mobile, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Un abonnement Tangoe Command Premium Mobile pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Tangoe Command Premium Mobile prend en charge l’authentification unique initiée par le **fournisseur de services**.

## <a name="adding-tangoe-command-premium-mobile-from-the-gallery"></a>Ajout de Tangoe Command Premium Mobile à partir de la galerie

Pour configurer l’intégration de Tangoe Command Premium Mobile à Azure AD, vous devez ajouter Tangoe Command Premium Mobile à votre liste d’applications SaaS gérées à partir de la galerie.

**Pour ajouter Tangoe Command Premium Mobile à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Tangoe Command Premium Mobile** , sélectionnez **Tangoe Command Premium Mobile** dans le volet des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Tangoe Command Premium Mobile dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Tangoe Command Premium Mobile pour un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique aboutisse, une liaison entre l’utilisateur Azure AD et l’utilisateur Tangoe Command Premium Mobile associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Tangoe Command Premium Mobile, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Tangoe Command Premium Mobile](#configure-tangoe-command-premium-mobile-single-sign-on)** pour définir les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Tangoe Command Premium Mobile](#create-tangoe-command-premium-mobile-test-user)** pour avoir un équivalent de Britta Simon dans Tangoe Command Premium Mobile, lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Tangoe Command Premium Mobile, procédez comme suit :

1. Dans le [portail Microsoft Azure](https://portal.azure.com/), sur la page d’intégration de l’application **Tangoe Command Premium Mobile** , cliquez sur **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL dans Tangoe Command Premium Mobile](common/sp-reply.png)

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://sso.tangoe.com/sp/startSSO.ping?PartnerIdpId=<tenant issuer>&TARGET=<target page url>`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://sso.tangoe.com/sp/ACS.saml2`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de réponse et l’URL de connexion réelles. Contact l’[équipe de support technique Tangoe Command Premium Mobile](https://www.tangoe.com/contact-us/) pour obtenir ces valeurs. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configuration de Tangoe Command Premium Mobile** , copiez l’URL ou les URL appropriées, en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-tangoe-command-premium-mobile-single-sign-on"></a>Configurer l’authentification unique dans Tangoe Command Premium Mobile

Pour configurer l’authentification unique côté **Tangoe Command Premium Mobile** , vous devez envoyer le document **XML des métadonnées de fédération** téléchargé et les URL appropriées, copiées à partir du portail Microsoft Azure, à [l’équipe du support technique de Tangoe Command Premium Mobile](https://www.tangoe.com/contact-us/). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon**.
  
    b. Dans le champ **Nom d’utilisateur** , tapez `brittasimon@yourcompanydomain.extension`. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Tangoe Command Premium Mobile.

1. Dans le portail Microsoft Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Tangoe Command Premium Mobile**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Tangoe Command Premium Mobile**.

    ![Lien Tangoe Command Premium Mobile dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

### <a name="create-tangoe-command-premium-mobile-test-user"></a>Créer un utilisateur de test Tangoe Command Premium Mobile

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans Tangoe Command Premium Mobile. Rapprochez-vous de l’[équipe de support Tangoe Command Premium Mobile](https://www.tangoe.com/contact-us/) pour ajouter les utilisateurs dans la plateforme Tangoe Command Premium Mobile. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la mosaïque Tangoe Command Premium Mobile dans le volet d’accès, vous devez être connecté automatiquement à l’application Tangoe Command Premium Mobile pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)