---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Kanbanize | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Kanbanize.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/09/2019
ms.author: jeedes
ms.openlocfilehash: 05ce0d3d3d5c66514edc07446aaf0a879657c10c
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92459235"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-kanbanize"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Kanbanize

Dans ce tutoriel, vous allez apprendre à intégrer Kanbanize à Azure Active Directory (Azure AD). Quand vous intégrez Kanbanize à Azure AD, vous pouvez :

* Contrôler qui a accès à Kanbanize
* Permettre à vos utilisateurs de se connecter automatiquement à Kanbanize avec leur compte Azure AD
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Kanbanize pour lequel l’authentification unique (SSO) est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Kanbanize prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**
* Kanbanize prend en charge l’attribution d’utilisateurs **juste-à-temps**

## <a name="adding-kanbanize-from-the-gallery"></a>Ajout de Kanbanize à partir de la galerie

Pour configurer l’intégration de Kanbanize à Azure AD, vous devez ajouter Kanbanize à votre liste d’applications SaaS gérées, à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Kanbanize** dans la zone de recherche.
1. Sélectionnez **Kanbanize** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-kanbanize"></a>Configurer et tester l’authentification unique Azure AD pour Kanbanize

Configurez et testez l’authentification unique Azure AD avec Kanbanize pour un utilisateur de test nommé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans Kanbanize.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec Kanbanize, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Kanbanize](#configure-kanbanize-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Kanbanize](#create-kanbanize-test-user)** pour avoir un équivalent de B.Simon dans Kanbanize qui soit lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Kanbanize** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

     a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<subdomain>.kanbanize.com/`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<subdomain>.kanbanize.com/saml/acs`

    c. Cliquez sur **Définir des URL supplémentaires** .

    d. Dans la zone de texte **État de relais** , tapez une URL : `/ctrl_login/saml_login`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<subdomain>.kanbanize.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez [l’équipe du support technique Kanbanize](mailto:support@ms.kanbanize.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Votre application Kanbanize s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration de vos attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut, où nameidentifier est mappé avec **user.userprincipalname** . L’application Kanbanize s’attend à ce que nameidentifier soit mappé avec **user.mail** . Vous devez donc modifier le mappage d’attribut en cliquant sur l’icône Modifier.

    ![image](common/edit-attribute.png)

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Kanbanize** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui octroyant l’accès à Kanbanize.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Kanbanize** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-kanbanize-sso"></a>Configurer l’authentification unique Kanbanize

1. Pour automatiser la configuration dans Kanbanize, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Kanbanize** pour être dirigé vers l’application Kanbanize. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Kanbanize. Cette extension de navigateur configure automatiquement l’application pour vous et automatise les étapes 3 à 7.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement Kanbanize, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Kanbanize en tant qu’administrateur et effectuez les étapes suivantes :

4. En haut à droite, cliquez sur l’icône des **paramètres** .

    ![Paramètres Kanbanize](./media/kanbanize-tutorial/tutorial-kanbanize-set.png)

5. Dans la page du volet Administration, à gauche du menu, cliquez sur **Integrations** , puis activez l’option **Single Sign-On** (Authentification unique).

    ![Capture d’écran montrant le volet Administration avec l’option Integrations sélectionnée.](./media/kanbanize-tutorial/tutorial-kanbanize-admin.png)

6. Dans la section Integrations, cliquez sur **CONFIGURE** (Configurer) pour ouvrir la page **Single Sign-On Integration** (Intégration de l’authentification unique).

    ![Configuration de Kanbanize](./media/kanbanize-tutorial/tutorial-kanbanize-config.png)

7. Dans la page **Single Sign-On Configuration** (Configuration de l’authentification unique), sous **Configurations** , effectuez les étapes suivantes :

    ![Capture d’écran montrant la page Edit Single Sign-On Integration dans laquelle vous pouvez entrer les valeurs de cette étape.](./media/kanbanize-tutorial/tutorial-kanbanize-save.png)

    a. Dans la zone de texte **Idp Entity ID** (Identificateur d’entité du fournisseur d’identité), collez l’ **Identificateur Azure AD** que vous avez copié dans le portail Azure.

    b. Dans la zone de texte **Idp Login Endpoint** (Point de terminaison de connexion du fournisseur d’identité), collez la valeur **URL de connexion** que vous avez copiée dans le Portail Azure.

    c. Dans la zone de texte **Idp Logout Endpoint** (Point de terminaison de déconnexion du fournisseur d’identité), collez la valeur **URL de déconnexion** que vous avez copiée dans le Portail Azure.

    d. Dans la zone de texte **Attribute name for Email** (Nom d’attribut pour l’e-mail), entrez cette valeur : `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`

    e. Dans la zone de texte **Attribute name for First Name** (Nom d’attribut pour le prénom), entrez cette valeur : `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`

    f. Dans la zone de texte **Attribute name for Last Name** (Nom d’attribut pour le nom de famille), entrez cette valeur : `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`

    > [!Note]
    > Vous pouvez obtenir ces valeurs en associant les valeurs de l’espace de noms et du nom de l’attribut, situées dans la section Attributs utilisateur du portail Azure.

    g. Dans le Bloc-notes, ouvrez le certificat codé en base 64 téléchargé dans le portail Azure, copiez son contenu (sans les marques de début et de fin), puis collez-le dans la zone **Idp X.509 Certificate** (Certificat X.509 du fournisseur d’identité).

    h. Cochez l’option **Enable login with both SSO and Kanbanize** (Permettre la connexion avec SSO et Kanbanize).

    i. Cliquez sur **Save Settings** .

### <a name="create-kanbanize-test-user"></a>Créer un utilisateur de test Kanbanize

Dans cette section, un utilisateur appelé B.Simon est créé dans Kanbanize. Kanbanize prend en charge l’attribution d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Kanbanize, il en est créé un après l’authentification. Si vous devez créer un utilisateur manuellement, contactez [l’équipe du support technique Kanbanize](mailto:support@ms.kanbanize.com).

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Kanbanize dans le volet d’accès, vous devez être connecté automatiquement à l’application Kanbanize pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Kanbanize avec Azure AD](https://aad.portal.azure.com/)