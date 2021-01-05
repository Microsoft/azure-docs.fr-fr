---
title: 'Tutoriel : Intégration d’Azure Active Directory à Civic Platform | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Civic Platform.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/25/2019
ms.author: jeedes
ms.openlocfilehash: 26109d4b8875dd0b442521513dbd219dc0de06e0
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92455967"
---
# <a name="tutorial-integrate-civic-platform-with-azure-active-directory"></a>Tutoriel : Intégrer Civic Platform à Azure Active Directory

Dans ce tutoriel, vous allez découvrir comment intégrer Civic Platform à Azure Active Directory (Azure AD). Quand vous intégrez Civic Platform à Azure AD, vous pouvez :

* Contrôler qui a accès à Civic Platform dans Azure AD.
* Permettre à vos utilisateurs de se connecter automatiquement à Civic Platform avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’en avez pas, vous pouvez obtenir un essai gratuit d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Civic Platform pour lequel l’authentification unique (SSO) est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Civic Platform prend en charge l’authentification unique lancée par le **fournisseur de services**





## <a name="adding-civic-platform-from-the-gallery"></a>Ajout de Civic Platform à partir de la galerie

Pour configurer l’intégration de Civic Platform à Azure AD, vous devez ajouter Civic Platform depuis la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Civic Platform** dans la zone de recherche.
1. Sélectionnez **Civic Platform** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec Civic Platform pour un utilisateur de test nommé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Civic Platform associé.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec Civic Platform, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Civic Platform](#configure-civic-platform-sso)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Civic Platform](#create-civic-platform-test-user)** pour avoir un équivalent de B.Simon dans Civic Platform lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Civic Platform** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<SUBDOMAIN>.accela.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL : `civicplatform.accela.com`

    > [!NOTE]
    > La valeur d’URL de connexion n’est pas réelle. Mettez à jour cette valeur avec l’URL de connexion réelle. Pour obtenir cette valeur, contactez [l’équipe du support Civic Platform](mailto:skale@accela.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Capture d’écran montrant la page SAML Signing Certificate, où vous pouvez copier l’URL des métadonnées de fédération d’application.](common/copy-metadataurl.png)

1. Accédez à **Azure Active Directory** > **Inscriptions des applications** dans Azure AD et sélectionnez votre application.

1. Copier l’ **ID de l’annuaire (locataire)** et stockez-le dans le Bloc-notes.

    ![Copier l’ID du répertoire (locataire) et le stocker dans votre code d’application](media/civic-platform-tutorial/directoryid.png)

1. Copiez l’ **ID d’application** et stockez-le dans le Bloc-notes.

   ![Copier l’ID d’application (client)](media/civic-platform-tutorial/applicationid.png)

1. Accédez à **Azure Active Directory** > **Inscriptions des applications** dans Azure AD et sélectionnez votre application. Cliquez sur **Certificats et secrets** .

1. Sélectionnez **Clés secrètes client -> Nouvelle clé secrète client** .

1. Fournissez une description et la durée du secret. Quand vous avez terminé, sélectionnez **Ajouter** .

   > [!NOTE]
   > Une fois la clé secrète client enregistrée, la valeur de la clé secrète client s’affiche. Copiez cette valeur car vous ne pourrez pas récupérer la clé ultérieurement.

   ![Copiez la valeur du secret, car vous ne pourrez pas la récupérer plus tard.](media/civic-platform-tutorial/secretkey.png)

### <a name="configure-civic-platform-sso"></a>Configurer l’authentification unique Civic Platform

1. Ouvrez une nouvelle fenêtre de navigateur web et connectez-vous à votre site d’entreprise Atlassian Cloud en tant qu’administrateur.

1. Cliquez sur **Standard Choices** (Choix standard).

    ![Capture d’écran montrant le site Atlassian Cloud avec Standard Choices mis en évidence sous Administrator Tools.](media/civic-platform-tutorial/standard-choices.png)

1. Créez un choix standard **ssoconfig** .

1. Recherchez **ssoconfig** et envoyez.

    ![Capture d’écran montrant une recherche dans Standard Choices avec le nom « sso config ».](media/civic-platform-tutorial/sso-config.png)

1. Développez SSOCONFIG en cliquant sur le point rouge.

    ![Capture d’écran montrant Standard Choices Browse avec SSO CONFIG disponible.](media/civic-platform-tutorial/sso-config01.png)

1. Fournissez les informations de configuration liées à l’authentification unique à l’étape suivante :

    ![Capture d’écran montrant Standard Choices - Item Edit pour SSO CONFIG.](media/civic-platform-tutorial/sso-config02.png)

    1. Dans le champ **applicationid** , entrez la valeur de l’ **ID d’application** que vous avez copiée à partir du portail Azure.

    1. Dans le champ **clientSecret** , entrez la valeur du **Secret** que vous avez copiée à partir du portail Azure.

    1. Dans le champ **directoryId** , entrez la valeur de l’ **ID de l’annuaire (locataire)** que vous avez copiée à partir du portail Azure.

    1. Entrez le idpName. Par exemple : `Azure`.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Civic Platform.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Civic Platform** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-civic-platform-test-user"></a>Créer un utilisateur de test Civic Platform

Dans cette section, vous créez un utilisateur appelé B.Simon dans Civic Platform. Collaborez avec l’équipe de support technique Civic Platform pour ajouter des utilisateurs dans [Civic Platform](mailto:skale@accela.com). Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

### <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Civic Platform dans le volet d’accès, vous devez être automatiquement connecté à l’application Civic Platform pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)