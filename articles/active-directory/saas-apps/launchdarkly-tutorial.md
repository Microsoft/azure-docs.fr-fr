---
title: 'Tutoriel : Intégration d’Azure Active Directory à LaunchDarkly | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et LaunchDarkly.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/02/2019
ms.author: jeedes
ms.openlocfilehash: 1ffa5d359e689220bd8cdbc7b9f6e305f451269a
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92458749"
---
# <a name="tutorial-azure-active-directory-integration-with-launchdarkly"></a>Tutoriel : Intégration d’Azure Active Directory à LaunchDarkly

Dans ce tutoriel, vous allez découvrir comment intégrer LaunchDarkly à Azure Active Directory (Azure AD).
L’intégration de LaunchDarkly à Azure AD offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à LaunchDarkly
* Vous pouvez permettre à vos utilisateurs de se connecter automatiquement à LaunchDarkly (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec LaunchDarkly, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement LaunchDarkly pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* LaunchDarkly prend en charge l’authentification unique initiée par le **fournisseur de services et le fournisseur d’identité**
* LaunchDarkly prend en charge l’attribution d’utilisateurs **Juste-à-temps**

## <a name="adding-launchdarkly-from-the-gallery"></a>Ajout de LaunchDarkly à partir de la galerie

Pour configurer l’intégration de LaunchDarkly à Azure AD, vous devez ajouter LaunchDarkly à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter LaunchDarkly à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **LaunchDarkly** , sélectionnez **LaunchDarkly** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![LaunchDarkly dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec [Nom de l’application] sur un utilisateur de test nommé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur [Nom de l’application] associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec [Nom de l’application], vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique LaunchDarkly](#configure-launchdarkly-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test LaunchDarkly](#create-launchdarkly-test-user)** pour avoir un équivalent de Britta Simon dans LaunchDarkly lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec [Nom de l’application], procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **LaunchDarkly** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. À la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode initié par **IDP** , suivez les étapes ci-dessous :

    ![Capture d’écran montrant Configuration SAML de base, où vous pouvez entrer l’identificateur, l’URL de réponse, et sélectionner Enregistrer.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL : `app.launchdarkly.com`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://app.launchdarkly.com/trust/saml2/acs/<customers-unique-id>`

    > [!NOTE]
    > La valeur de l’URL de réponse n’est pas réelle. Vous mettrez à jour la valeur avec l’URL de réponse réelle. La procédure est expliquée plus loin dans le didacticiel. Si vous prévoyez d’utiliser l’application en mode **IDP** , laissez le champ **URL de connexion** vide. Dans le cas contraire, vous ne pourrez pas établir la connexion à partir du **fournisseur d’identité** (IDP). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://app.launchdarkly.com`

    ![Capture d’écran montrant Définir des URL supplémentaires, où vous pouvez entrer une URL de connexion.](common/metadata-upload-additional-signon.png)

6. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

7. Dans la section **Configurer LaunchDarkly** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-launchdarkly-single-sign-on"></a>Configurer l’authentification unique LaunchDarkly

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise LaunchDarkly en tant qu’administrateur.

2. Sélectionnez **Account Settings** dans le volet de navigation gauche.

    ![Capture d’écran montrant l’élément Account Settings sélectionné sous Production.](./media/launchdarkly-tutorial/configure1.png)

3. Cliquez sur l’onglet **Security** .

    ![Capture d’écran montrant l’onglet Security de Account Settings.](./media/launchdarkly-tutorial/configure2.png)

4. Cliquez sur **ENABLE SSO** , puis sur **EDIT SAML CONFIGURATION** .

    ![Capture d’écran montrant la page Single Sign-On avec les options ENABLE SSO et EDIT SAML CONFIGURATION.](./media/launchdarkly-tutorial/configure3.png)

5. Dans la section **Edit your SAML configuration** , effectuez les étapes suivantes :

    ![Capture d’écran montrant la section Edit your SAML configuration dans laquelle vous pouvez apporter les modifications décrites ici.](./media/launchdarkly-tutorial/configure4.png)

    a. Copiez la valeur **SAML consumer service URL** pour votre instance et collez-la dans la zone de texte URL de réponse de la section **Domaine et URL LaunchDarkly** dans le portail Azure.

    b. Dans la zone de texte **URL de connexion** , collez **l’URL de connexion** que vous avez copiée à partir du portail Azure.

    c. Ouvrez le certificat téléchargé à partir du portail Azure dans le Bloc-notes, copiez le contenu et collez-le dans la zone **X.509 certificate** . Vous pouvez aussi envoyer directement le certificat en cliquant sur **upload one** .

    d. Cliquez sur **Enregistrer** .

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à LaunchDarkly.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **LaunchDarkly** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **LaunchDarkly** .

    ![Le lien LaunchDarkly dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-launchdarkly-test-user"></a>Créer un utilisateur de test LaunchDarkly

L’objectif de cette section est de créer un utilisateur nommé Britta Simon dans LaunchDarkly. LaunchDarkly prend en charge le provisionnement juste-à-temps, qui est activé par défaut. Vous n’avez aucune opération à effectuer dans cette section. Un utilisateur est créé lors d’une tentative d’accès à LaunchDarkly s’il n’existe pas encore.

> [!Note]
> Si vous devez créer un utilisateur manuellement, contactez [l’équipe de support client LaunchDarkly](mailto:support@launchdarkly.com).

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la vignette LaunchDarkly dans le volet d’accès, vous devez être connecté automatiquement à l’application LaunchDarkly pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)