---
title: 'Didacticiel : Intégration d’Azure Active Directory à Help Scout | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Help Scout.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/24/2019
ms.author: jeedes
ms.openlocfilehash: 7c5e8210bc8b805d72149fd2ef3335c1d637a58f
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92445186"
---
# <a name="tutorial-azure-active-directory-integration-with-help-scout"></a>Didacticiel : Intégration d’Azure Active Directory à Help Scout

Dans ce didacticiel, vous allez apprendre à intégrer Help Scout à Azure Active Directory (Azure AD).
L’intégration de Help Scout à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Help Scout.
* Vous pouvez autoriser vos utilisateurs à se connecter automatiquement à Help Scout (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Help Scout, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Help Scout pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Help Scout prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**
* Help Scout prend en charge l’attribution d’utilisateurs **juste-à-temps**

## <a name="adding-help-scout-from-the-gallery"></a>Ajout de Help Scout à partir de la galerie

Pour configurer l’intégration de Help Scout à Azure AD, vous devez ajouter Help Scout à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Help Scout** dans la zone de recherche.
1. Sélectionnez **Help Scout** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Help Scout, à l’aide d’un utilisateur de test appelé **B.Simon** .
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur Help Scout associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Help Scout, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Help Scout](#configure-help-scout-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer l’utilisateur de test Help Scout](#create-help-scout-test-user)** pour avoir dans Help Scout un équivalent de B.Simon lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Help Scout, effectuez les étapes suivantes :

1. Dans la page d’intégration de l’application **Help Scout** sur le [portail Azure](https://portal.azure.com/), sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

1. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

1. À la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode initié par **IDP** , suivez les étapes ci-dessous :

    ![Capture d’écran montrant Configuration SAML de base, où vous pouvez entrer l’identificateur, l’URL de réponse, et sélectionner Enregistrer.](common/idp-intiated.png)

    a. **Identificateur** correspond à **Audience URI (Service Provider Entity ID)** (URI d’audience (ID d’entité de fournisseur de services)) Help Scout, qui commence par `urn:`

    b. **URL de réponse** correspond à **Post-back URL (Assertion Consumer Service URL)** (URL de publication (URL Assertion Consumer Service)) Help Scout, qui commence par `https://` 

    > [!NOTE]
    > Les valeurs de ces URL servent uniquement à des fins de démonstration. Vous devez mettre à jour ces valeurs avec l’URL de réponse et l’identificateur réels. Vous obtenez ces valeurs à partir de l’onglet **Authentification unique** sous la section Authentification, qui est décrite plus loin dans le didacticiel.

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    ![Capture d’écran montrant Définir des URL supplémentaires, où vous pouvez entrer une URL de connexion.](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL d’authentification** , tapez l’URL : `https://secure.helpscout.net/members/login/`

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Help Scout** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé B.Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **B.Simon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez **B.Simon\@yourcompanydomain.extension** .  
    Par exemple : B.Simon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Help Scout.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Help Scout** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Help Scout** .

    ![Lien Help Scout dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B.Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-help-scout-sso"></a>Configurer l’authentification unique Help Scout

1. Pour automatiser la configuration dans Help Scout, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

1. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Help Scout** pour être dirigé vers l’application Help Scout. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Help Scout. Cette extension de navigateur configure automatiquement l’application pour vous et automatise les étapes 3 à 7.

    ![Configuration](common/setup-sso.png)

1. Si vous souhaitez configurer manuellement Help Scout, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Help Scout en tant qu’administrateur, puis effectuez les étapes suivantes :

1. Cliquez sur **Manage** (Gérer) dans le menu supérieur, puis sélectionnez **Company** dans le menu déroulant.

    ![Capture d’écran montrant le menu Manage avec Company sélectionné.](./media/helpscout-tutorial/settings1.png)

1. Sélectionnez **Authentication** à partir du volet de navigation gauche.

    ![Capture d’écran montrant Authentication sélectionné.](./media/helpscout-tutorial/settings2.png)

1. Vous accédez ainsi à la section des paramètres SAML où vous effectuez les étapes suivantes :

    ![Capture d’écran montrant l’onglet Single Sign-On où vous entrez les informations spécifiées.](./media/helpscout-tutorial/settings3.png)

    a. Copiez la valeur de **Post-back URL (Assertion Consumer Service URL)** (URL de publication (URL Assertion Consumer Service)) et collez-la dans la zone de texte **URL de réponse** de la section **Configuration SAML de base** sur le portail Azure.

    b. Copiez la valeur de **Audience URI (Service Provider Entity ID)** (URL d’audience (ID d’entité de fournisseur de services)) et collez-la dans la zone de texte **Identificateur** de la section **Configuration SAML de base** sur le portail Azure.

1. Activez **Activer SAML** et effectuez les étapes suivantes :

    ![Capture d’écran montrant l’onglet Single Sign-On où vous activez SAML et ajoutez d’autres informations.](./media/helpscout-tutorial/settings4.png)

    a. Dans la zone de texte **Single Sign-on URL** (URL d’authentification unique), collez la valeur de l’ **URL de connexion** que vous avez copiée dans le portail Azure.

    b. Cliquez sur **Charger le certificat** pour charger le **certificat (en base64)** téléchargé à partir du portail Azure.

    c. Entrez les domaines de messagerie de votre organisation, par exemple `contoso.com`, dans la zone de texte **Email Domains** (Domaines de messagerie). Vous pouvez séparer plusieurs domaines par une virgule. À tout moment, un utilisateur ou administrateur Help Scout qui entre dans ce domaine spécifique dans la [page de connexion d’Help Scout](https://secure.helpscout.net/members/login/) est acheminé vers le fournisseur d’identité pour être authentifié avec ses informations d’identification.

    d. Enfin, vous pouvez activer **Force SAML Sign-on** (Forcer l’authentification SAML) si vous souhaitez que les utilisateurs se connectent uniquement à Help Scout via cette méthode. Si vous voulez toujours leur donner la possibilité de se connecter avec leurs informations d’identification Help Scout, vous pouvez laisser cette option désactivée. Même si elle est activée, un propriétaire de compte sera toujours en mesure de se connecter à Help Scout avec son mot de passe de compte.

    e. Cliquez sur **Enregistrer** .

### <a name="create-help-scout-test-user"></a>Créer l’utilisateur de test Help Scout

Dans cette section, un utilisateur appelé B.Simon est créé dans Help Scout. Help Scout prend en charge l’attribution d’utilisateurs juste-à-temps, une option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Help Scout, il en est créé un après l’authentification.

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Help Scout dans le volet d’accès doit vous connecter automatiquement à l’application Help Scout pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Help Scout avec Azure AD](https://aad.portal.azure.com/)