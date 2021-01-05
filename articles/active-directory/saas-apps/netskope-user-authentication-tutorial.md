---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Netskope User Authentication | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Netskope User Authentication.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/01/2019
ms.author: jeedes
ms.openlocfilehash: 1ae9e39f5a23725b17451d7144c8304194d2a1d0
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92514365"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-netskope-user-authentication"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Netskope User Authentication

Ce tutoriel explique comment intégrer Netskope User Authentication à Azure Active Directory (Azure AD). En intégrant Netskope User Authentication à Azure AD, vous pouvez :

* Contrôler, dans Azure AD, qui a accès à Netskope User Authentication.
* Permettre aux utilisateurs de se connecter automatiquement à Netskope User Authentication avec leur compte Azure AD
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Netskope User Authentication pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Netskope User Authentication prend en charge l’authentification unique initiée par **le fournisseur de services et le fournisseur d’identité** .

## <a name="adding-netskope-user-authentication-from-the-gallery"></a>Ajout de Netskope User Authentication à partir de la galerie

Pour configurer l’intégration de Netskope User Authentication dans Azure AD, vous devez ajouter Netskope User Authentication à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Netskope User Authentication** dans la zone de recherche.
1. Sélectionnez **Netskope User Authentication** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-netskope-user-authentication"></a>Configurer et tester l’authentification unique Azure AD pour Netskope User Authentication

Configurez et testez l’authentification unique Azure AD auprès de Netskope User Authentication avec un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Netskope User Authentication associé.

Pour configurer et tester l’authentification unique Azure AD auprès de Netskope User Authentication, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Netskope User Authentication](#configure-netskope-user-authentication-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Netskope User Authentication](#create-netskope-user-authentication-test-user)** pour avoir un équivalent de B.Simon dans Netskope User Authentication lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Netskope User Authentication** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<tenantname>.goskope.com/<customer entered string>`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<tenantname>.goskope.com/nsauth/saml2/http-post/<customer entered string>`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de réponse réels. Ces valeurs vous seront expliquées plus loin dans le tutoriel.

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<tenantname>.goskope.com`

    > [!NOTE]
    > La valeur de l’URL de connexion n’est pas réelle. Modifiez-la selon l’URL de connexion réelle. Contactez l’[équipe du support technique de Netskope User Authentication](mailto:support@netskope.com) pour connaître la valeur de l’URL de connexion. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer Netskope User Authentication** , copiez l’URL ou les URL appropriées selon à vos besoins.

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

Dans cette section, vous autorisez B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Netskope User Authentication.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Netskope User Authentication** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-netskope-user-authentication-sso"></a>Configurer l’authentification unique Netskope User Authentication

1. Ouvrez un nouvel onglet dans votre navigateur et connectez-vous à votre site d’entreprise Netskope User Authentication en tant qu’administrateur.

1. Cliquez sur l’onglet **Active Platform** .

    ![Capture d’écran montrant l’élément Active Platform sélectionné à partir de Settings.](./media/netskope-user-authentication-tutorial/user1.png)

1. Faites défiler la page vers le bas jusqu’à **FORWARD PROXY** (Proxy de transfert) et sélectionnez **SAML** .

    ![Capture d’écran montrant l’option SAML sélectionnée à partir de Active Platform.](./media/netskope-user-authentication-tutorial/config-saml.png)

1. Sur la page **SAML Settings** , procédez comme suit :

    ![Capture d’écran montrant la zone des paramètres SAML dans laquelle vous pouvez indiquer les valeurs décrites.](./media/netskope-user-authentication-tutorial/configure-copyurls.png)

    a. Copiez la valeur **SAML Entity ID** , puis collez-la dans la zone de texte **Identificateur** de la section **Configuration SAML de base** du portail Azure.

    b. Copiez la valeur **SAML ACS URL** , puis collez-la dans la zone de texte **URL de réponse** de la section **Configuration SAML de base** du portail Azure.

1. Cliquez sur **ADD ACCOUNT** (Ajouter un compte).

    ![Capture d’écran montrant l’option ADD ACCOUNT sélectionnée dans le volet SAML.](./media/netskope-user-authentication-tutorial/config-addaccount.png)

1. Dans la page **Add SAML Account** (Ajouter un compte SAML), effectuez les étapes suivantes :

    ![Capture d’écran montrant la page Add SAML Account dans laquelle vous pouvez indiquer les valeurs décrites.](./media/netskope-user-authentication-tutorial/config-settings1.png)

    a. Dans la zone de texte **NAME** , indiquez un nom comme Azure AD.

    b. Dans la zone de texte **IDP URL** (URL du fournisseur d’identité), collez la valeur de l’ **URL de connexion** que vous avez copiée sur le portail Azure.

    c. Dans la zone de texte **IDP ENTITY ID** (ID d’entité de fournisseur d’identité), collez la valeur **Identificateur Azure AD** que vous avez copiée dans le portail Azure.

    d. Ouvrez le fichier de métadonnées téléchargé dans le Bloc-notes, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **IDP CERTIFICATE** (Certificat de fournisseur d’identité).

    e. Cliquez sur **ENREGISTRER** .

### <a name="create-netskope-user-authentication-test-user"></a>Créer un utilisateur de test Netskope User Authentication

1. Ouvrez un nouvel onglet dans votre navigateur et connectez-vous à votre site d’entreprise Netskope User Authentication en tant qu’administrateur.

1. Cliquez sur l’onglet **Settings** dans le volet de navigation de gauche.

    ![Capture d’écran montrant l’élément Settings sélectionné.](./media/netskope-user-authentication-tutorial/config-settings.png)

1. Cliquez sur l’onglet **Active Platform** .

    ![Capture d’écran montrant l’élément Active Platform sélectionné à partir de Settings.](./media/netskope-user-authentication-tutorial/user1.png)

1. Cliquez sur l’onglet **Users** (Utilisateurs).

    ![Capture d’écran montrant l’option Users sélectionnée à partir de Active Platform.](./media/netskope-user-authentication-tutorial/add-user.png)

1. Cliquez sur **ADD USERS** .

    ![Capture d’écran montrant la boîte de dialogue Users dans laquelle vous pouvez sélectionner ADD USERS.](./media/netskope-user-authentication-tutorial/user-add.png)

1. Entrez l’adresse e-mail de l’utilisateur à ajouter, puis cliquez sur **ADD** .

    ![Capture d’écran montrant la zone Add Users dans laquelle vous pouvez entrer une liste d’utilisateurs.](./media/netskope-user-authentication-tutorial/add-user-popup.png)

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Netskope User Authentication dans le volet d’accès, vous devez être connecté automatiquement à l’application Netskope User Authentication pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Netskope User Authentication avec Azure AD](https://aad.portal.azure.com/)