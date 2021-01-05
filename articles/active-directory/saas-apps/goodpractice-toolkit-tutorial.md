---
title: 'Tutoriel : Intégration d’Azure Active Directory à Mind Tools Toolkit | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Mind Tools Toolkit.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/12/2020
ms.author: jeedes
ms.openlocfilehash: 57598ea22874cfba772f6cf0e6b2ba779ef7c482
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92448385"
---
# <a name="tutorial-azure-active-directory-integration-with-mind-tools-toolkit"></a>Tutoriel : Intégration d’Azure Active Directory à Mind Tools Toolkit

Dans ce tutoriel, vous allez apprendre à intégrer Mind Tools Toolkit à Azure Active Directory (Azure AD).

Avec cette intégration, vous pouvez :

* Dans Azure AD, contrôlez qui a accès à Mind Tools Toolkit.
* Permettez à vos utilisateurs de se connecter automatiquement à Mind Tools Toolkit (authentification unique) avec leur compte Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS (software as a service) à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration de Mind Tools Toolkit à Azure AD, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Mind Tools Toolkit avec l’authentification unique (SSO) activé.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Mind Tools Toolkit prend en charge l’authentification unique lancée par le fournisseur de services.
* Mind Tools Toolkit prend en charge l’attribution d’utilisateurs juste-à-temps.
* Après avoir configuré Mind Tools Toolkit, vous pouvez appliquer le contrôle de session. Ce contrôle protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="add-mind-tools-toolkit-from-the-gallery"></a>Ajouter Mind Tools Toolkit depuis la galerie

Pour configurer l’intégration de Mind Tools Toolkit dans Azure AD, vous devez ajouter Mind Tools Toolkit à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire, ou avec un compte personnel Microsoft.
1. Dans le volet de navigation le plus à gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , saisissez **Mind Tools Toolkit** dans la zone de recherche.
1. Sélectionnez **Mind Tools Toolkit** dans les résultats de la recherche, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Mind Tools Toolkit, à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur Mind Tools Toolkit associé.

Pour configurer et tester l’authentification unique Azure AD avec Mind Tools Toolkit, suivez les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Mind Tools Toolkit](#configure-mind-tools-toolkit-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créez un utilisateur de test Mind Tools Toolkit](#create-a-mind-tools-toolkit-test-user)** pour avoir un équivalent de B. Simon dans Mind Tools Toolkit. Cet équivalent est lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous configurez l’authentification unique Azure AD avec Mind Tools Toolkit, en procédant comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Mind Tools Toolkit** , sélectionnez **Authentification unique** .

    ![La section Gérer avec Authentification unique en évidence](common/select-sso.png)

1. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML/WS-Fed** pour activer l’authentification unique.

    ![La boîte de dialogue Sélectionner une méthode d’authentification unique, avec SAML mis en évidence](common/select-saml-option.png)

1. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône de crayon pour **Configuration SAML de base** afin de modifier les paramètres.

    ![La page Configurer l’authentification unique avec SAML, avec l’icône de crayon pour Configuration SAML de base mise en évidence](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , dans la zone **URL de connexion** , entrez une URL au format `https://app.goodpractice.net/#/<subscriptionUrl>/s/<locationId>`.

    > [!NOTE]
    > La valeur d’ **URL de connexion** n’est pas réelle. Mettez à jour la valeur avec l’URL de connexion réelle. Pour obtenir cette valeur, contactez l’[équipe du support technique Mind Tools Toolkit](mailto:support@goodpractice.com).

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** . À droite de **XML de métadonnées de fédération** , sélectionnez **Télécharger** pour télécharger le texte XML et l’enregistrer sur votre ordinateur. Le contenu XML dépend des options que vous sélectionnez.

    ![Section Certificat de signature SAML, avec Télécharger mis en évidence en regard de XML de métadonnées de fédération](common/metadataxml.png)

1. Dans **Configurer Mind Tools Toolkit** , copiez les URL dont vous avez besoin parmi les suivantes.

    * **URL de connexion**

    * **Identificateur Azure AD**

    * **URL de déconnexion**

    ![La section Configurer Mind Tools Toolkit, avec les URL de configuration mises en évidence](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous créez un utilisateur de test appelé B.Simon dans le portail Azure :

1. Dans le volet le plus à gauche du portail Azure, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** , en haut de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez **B.Simon** .  
   1. Dans le champ **Nom d’utilisateur** , saisissez **B.Simon@** _domaine_entreprise_ **.** _extension_ . Par exemple : B.Simon@contoso.com.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans la zone **Mot de passe** .
   1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Mind Tools Toolkit.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Mind Tools Toolkit** .
1. Dans la page de vue d’ensemble de l’application, accédez à la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![La section Gérer, avec Utilisateurs et groupes mis en évidence](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** . Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Utilisateurs et groupes** .

   ![La fenêtre Utilisateurs et groupes, avec Ajouter un utilisateur mis en évidence](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B.Simon** dans la liste des utilisateurs. Ensuite, choisissez le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste. Ensuite, choisissez le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Affecter** .

## <a name="configure-mind-tools-toolkit-sso"></a>Configurer l’authentification unique Mind Tools Toolkit

Pour configurer l’authentification unique du côté de **Mind Tools Toolkit** , envoyez le **XML de métadonnées de fédération** téléchargé et les URL précédemment copiées à l’ [équipe de support technique Mind Tools Toolkit](mailto:support@goodpractice.com). Celle-ci configure ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-a-mind-tools-toolkit-test-user"></a>Créer un utilisateur de test Mind Tools Toolkit

Dans cette section, vous créez un utilisateur appelé B. Simon dans Mind Tools Toolkit.

Mind Tools Toolkit prend en charge le provisionnement juste-à-temps, une option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. Si un utilisateur n’existe pas déjà dans Mind Tools Toolkit, un nouvel utilisateur est créé lorsque vous tentez d’accéder à Mind Tools Toolkit.

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à partir du portail Mes applications.

Quand vous sélectionnez la vignette Mind Tools Toolkit dans le portail Mes applications, vous vous connectez automatiquement à l’application Mind Tools Toolkit pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le portail Mes applications, consultez [Introduction au portail Mes applications](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour intégrer des applications SaaS à Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Mind Tools Toolkit avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Mind Tools Toolkit avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)