---
title: 'Tutoriel : Intégration d’Azure Active Directory à Mimecast Admin Console | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Mimecast Admin Console.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/21/2020
ms.author: jeedes
ms.openlocfilehash: 8f0274aa15d965a4e06a096c21c0fd282ec1d12a
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92522608"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-mimecast-admin-console"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory avec Mimecast Admin Console

Dans ce tutoriel, vous allez apprendre à intégrer Mimecast Admin Console à Azure Active Directory (Azure AD). Quand vous intégrez Mimecast Admin Console à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Mimecast Admin Console.
* Permettre aux utilisateurs de se connecter automatiquement à Mimecast Admin Console avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Mimecast Admin Console pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Mimecast Admin Console prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**
* Après avoir configuré Mimecast Admin Console, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-mimecast-admin-console-from-the-gallery"></a>Ajout de Mimecast Admin Console à partir de la galerie

Pour configurer l’intégration de Mimecast Admin Console à Azure AD, vous devez ajouter Mimecast Admin Console à partir de la galerie, à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Mimecast Admin Console** dans la zone de recherche.
1. Sélectionnez **Mimecast Admin Console** dans le panneau des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-mimecast-admin-console"></a>Configurer et tester l’authentification unique Azure AD pour Mimecast Admin Console

Configurez et testez l’authentification unique Azure AD avec Mimecast Admin Console à l’aide d’un utilisateur de test nommé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Mimecast Admin Console associé.

Pour configurer et tester l’authentification unique Azure AD avec Mimecast Admin Console, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Mimecast Admin Console](#configure-mimecast-admin-console-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Mimecast Admin Console](#create-mimecast-admin-console-test-user)** pour obtenir un équivalent de B.Simon dans Mimecast Admin Console lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Mimecast Admin Console** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le fournisseur d’identité, effectuez les étapes suivantes :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant :

    | Région  |  Valeur | 
    | --------------- | --------------- |
    | Europe          | `https://eu-api.mimecast.com/sso/<accountcode>`|
    | États-Unis   | `https://us-api.mimecast.com/sso/<accountcode>`|
    | Afrique du Sud    | `https://za-api.mimecast.com/sso/<accountcode>`|
    | Australie       | `https://au-api.mimecast.com/sso/<accountcode>`|
    | Offshore        | `https://jer-api.mimecast.com/sso/<accountcode>`|

    > [!NOTE]
    > Vous trouverez la valeur `accountcode` dans le portail Mimecast Admin Console sous **Account** > **Settings** > **Account Code** (Compte > Paramètres > Code de compte). Ajoutez le `accountcode` à l’identificateur.

    b. Dans la zone de texte **URL de réponse** , tapez l’URL au format suivant : 

    | Région  |  Valeur | 
    | --------------- | --------------- | 
    | Europe          | `https://eu-api.mimecast.com/login/saml`|
    | États-Unis   | `https://us-api.mimecast.com/login/saml`|
    | Afrique du Sud    | `https://za-api.mimecast.com/login/saml`|
    | Australie       | `https://au-api.mimecast.com/login/saml`|
    | Offshore        | `https://jer-api.mimecast.com/login/saml`|

1. Si vous souhaitez configurer l’application en mode initié par le **fournisseur de service**  :

    Dans la zone de texte **URL d’authentification** , tapez l’URL : 

    | Région  |  Valeur | 
    | --------------- | --------------- | 
    | Europe          | `https://login-eu.mimecast.com/administration/app/#/administration-dashboard`|
    | États-Unis   | `https://login-us.mimecast.com/administration/app/#/administration-dashboard`|
    | Afrique du Sud    | `https://login-za.mimecast.com/administration/app/#/administration-dashboard`|
    | Australie       | `https://login-au.mimecast.com/administration/app/#/administration-dashboard`|
    | Offshore        | `https://login-jer.mimecast.com/administration/app/#/administration-dashboard`|

1. Cliquez sur **Enregistrer** .

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Mimecast Admin Console.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Mimecast Admin Console** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-mimecast-admin-console-sso"></a>Configurer l’authentification unique Mimecast Admin Console

1. Dans une autre fenêtre du navigateur web, connectez-vous à Mimecast Administration Console.

1. Accédez à **Administration** > **Services** > **Applications** .

    ![Capture d’écran montrant la fenêtre Mimecast avec l’élément Applications sélectionné.](./media/mimecast-admin-console-tutorial/services.png)

1. Cliquez sur l’onglet **Authentication Profiles** (Profils d’authentification).
    
    ![Capture d’écran montrant l’onglet Application avec les profils d’authentification sélectionnés.](./media/mimecast-admin-console-tutorial/authentication-profiles.png)

1. Cliquez sur l’onglet **New Authentication Profile** (Nouveau profil d’authentification).

    ![Capture d’écran montrant l’élément New Authentication Profile sélectionné.](./media/mimecast-admin-console-tutorial/new-authenticatio-profile.png)

1. Entrez une description valide dans la zone de texte **Description** et cochez la case **Enforce SAML Authentication for Administration Console** (Appliquer l’authentification SAML pour Administration Console).

    ![Capture d’écran montrant l’emplacement où sélectionner Enforce SAML Authentication for Administration Console.](./media/mimecast-admin-console-tutorial/selecting-admin-consle.png)

1. Dans la page **SAML Configuration for Administration Console** (Configuration SAML pour Administration Console), effectuez les étapes suivantes :

    ![Capture d’écran montrant la page SAML Configuration for Administration Console dans laquelle vous pouvez indiquer les valeurs décrites.](./media/mimecast-admin-console-tutorial/sso-settings.png)

    a. Pour **Provider** (Fournisseur), sélectionnez **Azure Active Directory** dans la liste déroulante.

    b. Dans la zone de texte **Metadata URL** (URL de métadonnées), collez la valeur **URL des métadonnées de fédération d’application** que vous avez copiée à partir du portail Azure.

    c. Cliquez sur **Importer** . Une fois l’URL des métadonnées importée, les champs sont renseignés automatiquement et aucune action n’est nécessaire sur ces champs.

    d. Veillez à décocher les cases **Use Password protected Context** (Utiliser le contexte protégé par mot de passe) et **Use Integrated Authentication Context** (Utiliser le contexte d’authentification intégré).

    e. Cliquez sur **Enregistrer** .

### <a name="create-mimecast-admin-console-test-user"></a>Créer un utilisateur de test Mimecast Admin Console

1. Dans une autre fenêtre du navigateur web, connectez-vous à Mimecast Administration Console.

1. Accédez à **Administration** > **Directories** > **Internal Directories** (Administration > Répertoires > Répertoires internes).

    ![Capture d’écran montrant la fenêtre Mimecast avec l’élément Internal Directories sélectionné.](./media/mimecast-admin-console-tutorial/internal-directories.png)

1. Sélectionnez votre domaine, s’il est mentionné ci-dessous. Sinon, créez-en un en cliquant sur **New Domain** (Nouveau domaine).

    ![Capture d’écran montrant le domaine sélectionné.](./media/mimecast-admin-console-tutorial/domain-name.png)

1. Cliquez sur l’onglet **New Address** (Nouvelle adresse).

    ![Capture d’écran montrant l’élément New Address sélectionné.](./media/mimecast-admin-console-tutorial/new-address.png)

1. Fournissez les informations utilisateur requises dans la page suivante :

    ![Capture d’écran montrant la page dans laquelle vous pouvez indiquer les valeurs décrites.](./media/mimecast-admin-console-tutorial/user-information.png)

    a. Dans la zone de texte **Email Address** , entrez l’adresse e-mail de l’utilisateur, par exemple `B.Simon@yourdomainname.com`.

    b. Dans la zone de texte **Global Name** , entrez le **nom complet** de l’utilisateur.

    c. Dans les zones de texte **Password** et **Confirm Password** , entrez le mot de passe de l’utilisateur.

    d. Cochez la case **Force Change at Login** (Forcer la modification lors de la connexion).

    e. Cliquez sur **Enregistrer** .

    f. Pour attribuer des rôles à l’utilisateur, cliquez sur **Role Edit** (Modification du rôle) et attribuez le rôle requis à l’utilisateur conformément aux exigences de votre organisation.

    ![Capture d’écran montrant la zone Address Settings où vous pouvez sélectionner Role Edit.](./media/mimecast-admin-console-tutorial/assign-role.png)

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Mimecast Admin Console dans le panneau d’accès, vous devez être connecté automatiquement à l’application Mimecast Admin Console pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Mimecast Admin Console avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Mimecast Admin Console avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)