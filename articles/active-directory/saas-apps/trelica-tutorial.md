---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Trelica | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Trelica.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/06/2020
ms.author: jeedes
ms.openlocfilehash: a674f5f653ad420ab8f28ff73c6b86f9c18b154e
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92517750"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-trelica"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Trelica

Dans ce tutoriel, vous allez apprendre à intégrer Trelica à Azure Active Directory (Azure AD).

Avec cette intégration, vous pouvez :

* Contrôler qui dans Azure AD a accès à Trelica.
* Permettre aux utilisateurs de se connecter automatiquement à Trelica avec leur compte Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS (software as a service) à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Trelica pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Trelica prend en charge l’authentification unique lancée par le fournisseur d’identité.
* Trelica prend en charge le provisionnement d’utilisateur juste-à-temps.
* Après avoir configuré Trelica, vous pouvez appliquer le contrôle de session. Ce contrôle protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-trelica-from-the-gallery"></a>Ajout de Trelica à partir de la galerie

Pour configurer l’intégration de Trelica à Azure AD, vous devez ajouter Trelica à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire, ou avec un compte personnel Microsoft.
1. Dans le volet de navigation le plus à gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , entrez **Trelica** dans la zone de recherche.
1. Sélectionnez **Trelica** dans les résultats de la recherche, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-trelica"></a>Configurer et tester l’authentification unique Azure AD pour Trelica

Configurez et testez l’authentification unique Azure AD avec Trelica à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Trelica associé.

Pour configurer et tester l’authentification unique Azure AD avec Trelica, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Trelica](#configure-trelica-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Trelica](#create-a-trelica-test-user)** pour avoir un équivalent de B.Simon dans Trelica. Cet équivalent est lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le portail Azure :

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Trelica** , puis allez à la section **Gérer** . Sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de crayon pour **Configuration SAML de base** afin de modifier les paramètres.

   ![La page Configurer l’authentification unique avec SAML, avec l’icône de crayon pour Configuration SAML de base mise en évidence](common/edit-urls.png)

1. Dans la page **Configurer l’authentification unique avec SAML** , entrez les valeurs suivantes :

    1. Dans la zone **Identificateur** , saisissez l’URL  **https://app.trelica.com** .

    1. Dans la zone **URL de réponse** , entrez une URL au format `https://app.trelica.com/Id/Saml2/<CUSTOM_IDENTIFIER>/Acs`.

    > [!NOTE]
    > La valeur de l’URL de réponse n’est pas réelle. Mettez à jour cette valeur avec l’URL de réponse réelle (également appelée ACS).
    > Vous la trouverez en vous connectant à Trelica et en accédant à la [page de configuration des fournisseurs d’identité SAML](https://app.trelica.com/Admin/Profile/SAML) (Admin > Account > SAML). Cliquez sur le bouton Copier en regard de l’ **URL Assertion Consumer Service (ACS)** pour la placer dans le Presse-papiers, afin de pouvoir la coller dans la zone de texte **URL de réponse** dans Azure AD.
    > Si vous avez des questions, lisez la [documentation d’aide de Trelica](https://docs.trelica.com/admin/saml/azure-ad) ou contactez l’[équipe de support technique de Trelica](mailto:support@trelica.com).

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Section Certificat de signature SAML, avec le bouton Copier mis en évidence en regard de l’URL des métadonnées de fédération d'application](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous créez un utilisateur de test appelé B.Simon dans le portail Azure.

1. Dans le volet le plus à gauche du portail Azure, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** , en haut de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez **B.Simon** .
   1. Dans le champ **Nom d’utilisateur** , saisissez **B.Simon@** _domaine_entreprise_ **.** _extension_ . Par exemple : B.Simon@contoso.com.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans la zone **Mot de passe** .
   1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Trelica.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Trelica** .
1. Dans la page de vue d’ensemble de l’application, accédez à la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![La section Gérer, avec Utilisateurs et groupes mis en évidence](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** . Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Utilisateurs et groupes** .

   ![La fenêtre Utilisateurs et groupes, avec Ajouter un utilisateur mis en évidence](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B.Simon** dans la liste des utilisateurs. Ensuite, choisissez le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste. Ensuite, choisissez le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Affecter** .

## <a name="configure-trelica-sso"></a>Configurer l’authentification unique Trelica

Pour configurer l’authentification unique côté **Trelica** , accédez à la [page de configuration des fournisseurs d’identité SAML](https://app.trelica.com/Admin/Profile/SAML) (Admin > Account > SAML). Cliquez sur le bouton **New** . Entrez **Azure AD** comme nom et choisissez **Metadata from url** comme type de métadonnées. Collez l’ **URL des métadonnées de fédération d’application** copiée à partir d’Azure AD dans le champ **Metadata url** dans Trelica.

Si vous avez des questions, lisez la [documentation d’aide de Trelica](https://docs.trelica.com/admin/saml/azure-ad) ou contactez l’[équipe de support technique de Trelica](mailto:support@trelica.com).

### <a name="create-a-trelica-test-user"></a>Créer un utilisateur de test Trelica

Trelica prend en charge le provisionnement d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Trelica, il en est créé un après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à partir du portail Mes applications.

Quand vous sélectionnez la vignette Trelica dans le portail Mes applications, vous vous connectez automatiquement à l’application Trelica pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le portail Mes applications, consultez [Introduction au portail Mes applications](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour intégrer des applications SaaS à Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Trelica avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Trelica avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)