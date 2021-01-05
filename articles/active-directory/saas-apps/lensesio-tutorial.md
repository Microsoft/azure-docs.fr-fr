---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Lenses.io | Microsoft Docs'
description: Dans ce tutoriel, vous allez découvrir comment configurer l’authentification unique entre Azure Active Directory et Lenses.io.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/02/2020
ms.author: jeedes
ms.openlocfilehash: 181d58baf128c4848a538e776aea0e43213994dc
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92458606"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-the-lensesio-dataops-portal"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory au portail Lenses.io DataOps

Dans ce tutoriel, vous allez découvrir comment intégrer le portail DataOps [Lenses.io](https://lenses.io/) à Azure Active Directory (Azure AD). Après avoir intégré Lenses.io avec Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès au portail Lenses.io.
* Permettre à vos utilisateurs de se connecter automatiquement à Lenses avec leur compte Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS (Software as a Service) à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure AD ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Une instance d’un portail Lenses. Vous pouvez choisir parmi diverses [options de déploiement](https://lenses.io/product/deployment/).
* Une [licence](https://lenses.io/product/pricing/) Lenses.io qui prend en charge l’authentification unique

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification SSO Azure AD dans un environnement de test.

* Lenses.io prend en charge l’authentification unique initiée par le fournisseur de services.

* Après avoir configuré Lenses.io, vous pouvez appliquer le contrôle de session. Ce contrôle protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="add-lensesio-from-the-gallery"></a>Ajout de Lenses.io depuis la galerie

Pour configurer l’intégration de Lenses.io avec Azure AD, ajoutez Lenses.io à votre liste d’applications SaaS gérées :

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire, ou avec un compte personnel Microsoft.
1. Dans le volet de gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , entrez **Lenses.io** dans la zone de recherche.
1. Sélectionnez **Lenses.io** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-lensesio"></a>Configurer et tester l’authentification unique (SSO) Azure AD pour Lenses.io

Vous allez créer un utilisateur de test appelé *B.Simon* pour configurer et tester l’authentification unique Azure AD avec votre portail Lenses.io. Pour que l’authentification unique fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur associé dans Lenses.io.

Procédez comme suit :

1. [Configurer l’authentification unique Azure AD](#configure-azure-ad-sso) pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. [Créer un groupe et un utilisateur de test Azure AD](#create-an-azure-ad-test-user-and-group) pour tester l’authentification unique Azure AD avec B.Simon.
    1. [Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user) pour permettre à B.Simon d’utiliser l’authentification unique Azure AD.
1. [Configurer l’authentification unique Lenses.io](#configure-lensesio-sso) pour configurer les paramètres d’authentification unique côté application.
    1. [Créer des autorisations de groupe de test Lenses.io](#create-lensesio-test-group-permissions) pour contrôler ce à quoi B.Simon peut accéder dans Lenses.io (autorisation).
1. [Tester l’authentification unique](#test-sso) pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le portail Azure :

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Lenses.io** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Capture d’écran montrant l’icône de modification de la configuration SAML de base.](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez des valeurs dans les zones de saisie de texte suivantes :

    a. **URL d’authentification**  : Entrez une URL au format suivant : `https://<CUSTOMER_LENSES_BASE_URL>`. par exemple `https://lenses.my.company.com`.

    b. **Identificateur (ID d’entité)**  : Entrez une URL au format suivant : `https://<CUSTOMER_LENSES_BASE_URL>`. par exemple `https://lenses.my.company.com`.

    c. **URL de réponse**  : Entrez une URL au format suivant : `https://<CUSTOMER_LENSES_BASE_URL>/api/v2/auth/saml/callback?client_name=SAML2Client`. par exemple `https://lenses.my.company.com/api/v2/auth/saml/callback?client_name=SAML2Client`.

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec les valeurs réelles de l’URL d’authentification, de l’URL de réponse et de l’identificateur, en fonction de l’URL de base de votre instance de portail Lenses. Pour plus d’informations, consultez la [documentation sur l’authentification unique Lenses.io](https://docs.lenses.io/install_setup/configuration/security.html#single-sign-on-sso-saml-2-0).

1. Dans la page **Configurer l’authentification unique avec SAML** , accédez à la section **Certificat de signature SAML** . Recherchez **XML de métadonnées de fédération** , puis sélectionnez **Télécharger** pour télécharger et enregistrer le certificat sur votre ordinateur.

    ![Capture d’écran affichant le lien de téléchargement de certificat.](common/metadataxml.png)

1. Dans la section **Configurer Lenses.io** , utilisez le fichier XML que vous avez téléchargé pour configurer Lenses par rapport à votre authentification unique Azure.

### <a name="create-an-azure-ad-test-user-and-group"></a>Créer un groupe et un utilisateur de test Azure AD

Dans le portail Azure, vous allez créer un utilisateur de test appelé B.Simon. Ensuite, vous allez créer un groupe de test qui contrôle l’accès de B.Simon dans Lenses.

Pour en savoir plus sur la façon dont Lenses utilise le mappage d’appartenance de groupe, consultez la [documentation sur l’authentification unique Lenses](https://docs.lenses.io/install_setup/configuration/security.html#id3).

**Pour créer l’utilisateur de test :**

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** , en haut de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans la zone **Nom** , entrez **B.Simon** .  
   1. Dans la zone **Nom d’utilisateur** , entrez username@companydomain.extension. Par exemple : B.Simon@contoso.com.
   1. Cochez la case **Afficher le mot de passe** . Notez le mot de passe affiché dans la zone **Mot de passe** .
   1. Sélectionnez **Créer** .

**Pour créer le groupe**

1. Accédez à **Azure Active Directory** , puis sélectionnez **Groupes** .
1. En haut de l’écran, sélectionnez **Nouveau groupe** .
1. Dans les **Propriétés du groupe** , effectuez les étapes suivantes :
   1. Dans la zone **Type de groupe** , sélectionnez **Sécurité** .
   1. Dans la zone **Nom de groupe** , entrez **LensesUsers** .
   1. Sélectionnez **Créer** .
1. Sélectionnez le groupe **LensesUsers** et copiez l’ **ID d’objet** (par exemple, f8b5c1ec-45de-4abd-af5c-e874091fb5f7). Vous verrez cet ID dans Lenses pour mapper les utilisateurs de ce groupe aux [autorisations appropriées](https://docs.lenses.io/install_setup/configuration/security.html#id3).  

**Pour affecter le groupe à l’utilisateur de test**

1. Accédez à **Azure Active Directory** , puis sélectionnez **Utilisateurs** .
1. Sélectionnez l’utilisateur de test **B.Simon** .
1. Sélectionnez **Groupes** .
1. En haut de l’écran, sélectionnez **Ajouter des appartenances** .
1. Recherchez et sélectionnez **LensesUsers** .
1. Cliquez sur **Sélectionner** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Lenses.io.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Lenses.io** .
1. Dans la page de vue d’ensemble de l’application, dans la section **Gérer** , sélectionnez **Utilisateurs et groupes** .

   ![Capture d’écran affichant le lien « Utilisateurs et groupes ».](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** .

   ![Capture d’écran affichant le lien Ajouter un utilisateur.](common/add-assign-user.png)

1. Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Utilisateurs et groupes** .
1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B.Simon** dans la liste des utilisateurs. Cliquez ensuite sur le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , choisissez le rôle approprié pour l’utilisateur dans la liste. Cliquez ensuite sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-lensesio-sso"></a>Configurer l’authentification unique Lenses.io

Pour configurer l’authentification unique dans le portail **Lenses.io** , installez le **XML des métadonnées de fédération** téléchargé sur votre instance de Lenses et [configurez Lenses pour activer l’authentification unique](https://docs.lenses.io/install_setup/configuration/security.html#configure-lenses).

### <a name="create-lensesio-test-group-permissions"></a>Créer des autorisations de groupe de test Lenses.io

1. Pour créer un groupe dans Lenses, utilisez l’ **ID d’objet** du groupe **LensesUsers** . Il s’agit de l’ID que vous avez copié dans la section [Création de l’utilisateur](#create-an-azure-ad-test-user-and-group).
1. Attribuez les autorisations souhaitées pour B.Simon.

Pour plus d’informations, consultez [Mappage de groupe Azure - Lenses](https://docs.lenses.io/install_setup/configuration/security.html#azure-groups).

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester votre configuration SSO Azure AD à l’aide du volet d’accès.

Quand vous sélectionnez la vignette Lenses.io dans le volet d’accès, vous devez être connecté automatiquement au portail Lenses.io. Pour plus d’informations, consultez [Présentation du volet d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Configurer l’authentification unique dans votre instance Lenses.io](https://docs.lenses.io/install_setup/configuration/security.html#single-sign-on-sso-saml-2-0)

- [Liste de tutoriels sur l’intégration d’applications SaaS à Azure AD](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure AD ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure AD ?](../conditional-access/overview.md)

- [Essayer Lenses.io avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Lenses.io avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)