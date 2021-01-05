---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec Britive | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Britive.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/25/2020
ms.author: jeedes
ms.openlocfilehash: 74f449afe2081feecfe7f0194b20429314ec572d
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92456668"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-britive"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec Britive

Dans ce tutoriel, vous allez apprendre à intégrer Britive avec Azure AD (Azure Active Directory). Quand vous intégrez Britive avec Azure AD, vous pouvez :

* Contrôler dans Azure AD l’accès à Britive
* Permettre aux utilisateurs de se connecter automatiquement à Britive avec leurs comptes Azure AD
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Britive pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Britive prend en charge l’authentification SSO lancée par le **SP (fournisseur de services)**
* Une fois que vous avez configuré Britive, vous pouvez appliquer des contrôles de session, qui protègent l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-britive-from-the-gallery"></a>Ajout de Britive à partir de la galerie

Pour configurer l’intégration de Britive avec Azure AD, vous devez ajouter Britive à votre liste d’applications SaaS managées, à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , dans la zone de recherche, tapez **Britive** .
1. Sélectionnez **Britive** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-britive"></a>Configurer et tester l’authentification unique Azure AD pour Britive

Configurez et testez l’authentification SSO Azure AD avec Britive à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification SSO fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans Britive.

Pour configurer et tester l’authentification SSO Azure AD avec Britive, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification SSO Britive](#configure-britive-sso)** , pour configurer les paramètres d’authentification unique côté application.
    1. **[Créer un utilisateur de test Britive](#create-britive-test-user)** , pour avoir un équivalent de B.Simon dans Britive, qui soit lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration d’application **Britive** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<TENANTNAME>.britive-app.com/sso`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `urn:amazon:cognito:sp:<UNIQUE_ID>`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels en suivant la procédure décrite plus loin dans ce tutoriel. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer Britive** , copiez les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez permettre à B.Simon d’utiliser l’authentification unique Azure en lui octroyant l’accès à Britive.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Britive** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-britive-sso"></a>Configurer l’authentification SSO pour Britive

1. Dans une autre fenêtre du navigateur web, connectez-vous au site web Britive en tant qu’administrateur.

1. Cliquez sur l’ **Icône Paramètres administrateur** , puis sélectionnez **Sécurité** .

    ![Capture d’écran montrant le site web Britive avec Settings et Security sélectionnés.](./media/britive-tutorial/configure1.png)

1. Sélectionnez **Configuration SSO** , puis effectuez les étapes suivantes :

    ![Capture d’écran montrant SSO Configuration, où vous entrez les informations de cette étape.](./media/britive-tutorial/configure2.png)

    a. Copiez la valeur **Audience/Entity ID** (ID d’entité/d’audience), puis collez-la dans la zone de texte **Identificateur (ID d’entité)** de la section **Configuration SAML de base** du portail Azure.

    b. Copiez la valeur **Initiate SSO URL** (URL de lancement de SSO), puis collez-la dans la zone de texte **URL de connexion** de la section **Configuration SAML de base** du portail Azure.

    c. Cliquez sur **UPLOAD SAML METADATA** (CHARGER LES MÉTADONNÉES SAML) pour charger le fichier XML de métadonnées téléchargé à partir du portail Azure. Une fois le fichier de métadonnées chargé, les valeurs ci-dessus sont indiquées automatiquement, et les changements enregistrés.

### <a name="create-britive-test-user"></a>Créer un utilisateur de test Britive

1. Dans une autre fenêtre du navigateur web, connectez-vous au site web Britive en tant qu’administrateur.

1. Cliquez sur l’ **Icône Paramètres administrateur** , puis sélectionnez **Administration des utilisateurs** .

    ![Capture d’écran montrant le site web Britive avec Settings et User Administration sélectionnés.](./media/britive-tutorial/user1.png)

1. Cliquez sur **AJOUTER UN UTILISATEUR** .

    ![Capture d’écran montrant le bouton ADD USER.](./media/britive-tutorial/user2.png)

1. Indiquez tous les détails nécessaires relatifs à l’utilisateur en fonction des exigences de votre organisation, puis cliquez sur **AJOUTER UN UTILISATEUR** .

    ![Capture d’écran montrant la page Add a User, où vous pouvez entrer des informations sur l’utilisateur.](./media/britive-tutorial/user3.png)

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Britive dans le volet d’accès, vous êtes automatiquement connecté à la plateforme Britive pour laquelle vous avez configuré l’authentification SSO. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Britive avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)