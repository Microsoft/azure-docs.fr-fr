---
title: 'Didacticiel : Intégration d’Azure Active Directory dans Lifesize Cloud | Documentation Microsoft'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Lifesize Cloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 1/4/2019
ms.author: jeedes
ms.openlocfilehash: 483377d0376a5a98d822392af1fa2d84ad8498e0
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92458589"
---
# <a name="tutorial-azure-active-directory-integration-with-lifesize-cloud"></a>Didacticiel : Intégration d’Azure Active Directory à Lifesize Cloud

Dans ce didacticiel, vous allez apprendre à intégrer Lifesize Cloud à Azure Active Directory (Azure AD).
L’intégration de Lifesize Cloud dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Lifesize Cloud.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Lifesize Cloud (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Lifesize Cloud, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Lifesize Cloud pour lequel l’authentification unique est activée

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Lifesize Cloud prend en charge l’authentification unique initiée par le **fournisseur de services**

* Lifesize Cloud prend en charge l’ **attribution d’utilisateurs automatique** .

## <a name="adding-lifesize-cloud-from-the-gallery"></a>Ajout de Lifesize Cloud à partir de la galerie

Pour configurer l’intégration de Lifesize Cloud à Azure AD, vous devez ajouter Lifesize Cloud de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Lifesize Cloud à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Lifesize Cloud** , sélectionnez **Lifesize Cloud** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Lifesize Cloud dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Lifesize Cloud, avec un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur associé dans Lifesize Cloud doit être établie.

Pour configurer et tester l’authentification unique avec Azure AD avec Lifesize Cloud, vous devez compléter les blocs de construction suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Lifesize Cloud](#configure-lifesize-cloud-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Lifesize Cloud](#create-lifesize-cloud-test-user)** pour avoir un équivalent de Britta Simon dans Lifesize Cloud qui est lié à la représentation d’un utilisateur Azure AD.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Lifesize Cloud, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Lifesize Cloud** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Lifesize Cloud](common/sp-identifier-relay.png)

    a. Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://login.lifesizecloud.com/ls/?acs`.

    b. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://login.lifesizecloud.com/<companyname>`

    c. Cliquez sur **Définir des URL supplémentaires** .

    d. Dans la zone de texte **État de relais** , entrez une URL en utilisant le modèle suivant : `https://webapp.lifesizecloud.com/?ent=<identifier>`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’état de relais réels. Pour obtenir les valeurs d’URL de connexion et d’identificateur, contactez l’[équipe du support technique Lifesize Cloud](https://www.lifesize.com/en/support). Vous pouvez également obtenir la valeur de l’état de relais à partir de la configuration SSO, expliquée plus loin dans ce tutoriel. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer Lifesize Cloud** , copiez la ou les URL appropriées correspondant à vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-lifesize-cloud-single-sign-on"></a>Configurer l’authentification unique Lifesize Cloud

1. Pour configurer SSO pour votre application, connectez-vous à l’application Lifesize Cloud avec les privilèges d’administrateur.

2. Dans le coin supérieur droit, cliquez sur votre nom, puis cliquez sur **Paramètres avancés** .

    ![Capture d’écran montrant l’élément de menu Advanced Settings.](./media/lifesize-cloud-tutorial/tutorial_lifesizecloud_06.png)

3. Dans les Paramètres avancés, cliquez maintenant sur le lien **Configuration de l’authentification unique** . La page Configuration SSO de l’instance s’ouvre.

    ![Capture d’écran montrant la zone des paramètres avancés dans laquelle vous pouvez sélectionner « SSO configuration ».](./media/lifesize-cloud-tutorial/tutorial_lifesizecloud_07.png)

4. Maintenant, configurez les valeurs suivantes dans l’interface utilisateur de configuration de l’authentification unique.

    ![Capture d’écran montrant la page SSO Configuration dans laquelle vous pouvez indiquer les valeurs décrites.](./media/lifesize-cloud-tutorial/tutorial_lifesizecloud_08.png)

    a. Dans la zone de texte **Identity Provider Issuer** (Émetteur du fournisseur d’identité), collez la valeur de **l’identificateur Azure AD** que vous avez copié à partir du portail Azure.

    b.  Dans la zone de texte **Login URL** (URL de connexion), collez la valeur **URL de connexion** que vous avez copiée dans le Portail Azure.

    c. Ouvrez dans le Bloc-notes votre certificat codé en base 64 téléchargé à partir du portail Azure, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **Certificat X.509** .
  
    d. Dans la zone de texte First Name (Prénom) des mappages d’attribut SAML, entrez la valeur `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`

    e. Dans la zone de texte **Last Name** (Nom) des mappages d’attribut SAML, entrez la valeur `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`

    f. Dans la zone de texte **Email** des mappages d’attribut SAML, entrez la valeur `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`

5. Pour vérifier la configuration, vous pouvez cliquer sur le bouton **Test** .

    >[!NOTE]
    >Pour que les tests se déroulent correctement, vous devez exécuter l’assistant de configuration dans Azure AD et également fournir l’accès aux utilisateurs ou groupes autorisés à effectuer le test.

6. Activez l’authentification unique en appuyant sur le bouton **Activer l’authentification unique** .

7. Maintenant, cliquez sur le bouton **Mettre à jour** afin que tous les paramètres soient enregistrés. Cela génère la valeur RelayState. Copiez la valeur RelayState générée dans la zone de texte, puis collez-la dans la zone de texte **État de relais** , sous la section **Domaine et URL Lifesize Cloud** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon\@domainedevotreentreprise.extension** .  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Lifesize Cloud.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Lifesize Cloud** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Lifesize Cloud** .

    ![Lien Lifesize Cloud dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-lifesize-cloud-test-user"></a>Créer un utilisateur de test Lifesize Cloud

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans Lifesize Cloud. Lifesize Cloud ne prend pas en charge l’approvisionnement automatique des utilisateurs. Après une authentification réussie à Azure AD, l’utilisateur sera automatiquement approvisionné dans l’application.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

En cliquant sur la vignette Lifesize Cloud dans le Panneau d’accès, vous allez en principe être connecté automatiquement à votre application Lifesize Cloud. Une fois que vous avez entré votre nom d’utilisateur, vous êtes redirigé vers la page d’accueil de l’application.

Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)