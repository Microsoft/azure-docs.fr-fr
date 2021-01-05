---
title: 'Didacticiel : intégration d’Azure Active Directory avec SSO SAML pour Bitbucket de resolution GmbH | Documents Microsoft'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SSO SAML pour Bitbucket de resolution GmbH.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/27/2018
ms.author: jeedes
ms.openlocfilehash: 5e0b1d8d2cb80886131a88c01a6c24102ad7c77c
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97673538"
---
# <a name="tutorial-azure-active-directory-integration-with-saml-sso-for-bitbucket-by-resolution-gmbh"></a>Didacticiel : intégration d’Azure Active Directory avec SSO SAML pour Bitbucket de resolution GmbH

Dans ce didacticiel, vous allez apprendre à intégrer SSO SAML pour Bitbucket de resolution GmbH avec Azure Active Directory (Azure AD).
L’intégration de SSO SAML pour Bitbucket de resolution GmbH avec Azure AD vous offre les avantages suivants :

* Vous pouvez contrôler dans Azure AD qui a accès à SSO SAML pour Bitbucket de resolution GmbH.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à SSO SAML pour Bitbucket de resolution GmbH (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec SSO SAML pour Bitbucket de resolution GmbH, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement SSO SAML pour Bitbucket de resolution GmbH pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* SSO SAML pour Bitbucket de resolution GmbH prend en charge l’authentification unique démarrée par **le fournisseur de services et le fournisseur d’identité**
* SSO SAML pour Bitbucket de resolution GmbH prend en charge l’attribution d’utilisateurs **Juste-à-temps**


## <a name="adding-saml-sso-for-bitbucket-by-resolution-gmbh-from-the-gallery"></a>Ajout de SSO SAML pour Bitbucket de resolution GmbH à partir de la galerie

Pour configurer l’intégration de SSO SAML pour Bitbucket de resolution GmbH dans Azure AD, vous devez ajouter SSO SAML pour Bitbucket de resolution GmbH à votre liste d’applications SaaS gérées à partir de la galerie.

**Pour ajouter SSO SAML pour Bitbucket de resolution GmbH à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **SSO SAML pour Bitbucket de resolution GmbH**, sélectionnez **SSO SAML pour Bitbucket de resolution GmbH** dans le volet de résultats, puis cliquez sur **Ajouter** pour ajouter l’application.

    ![SSO SAML pour Bitbucket de resolution GmbH dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous configurez et testez l’authentification unique Azure AD avec SSO SAML pour Bitbucket de resolution GmbH en vous servant d’un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, un lien doit être établi entre un utilisateur Azure AD et l’utilisateur de SSO SAML pour Bitbucket de resolution GmbH associé.

Pour configurer et tester l’authentification unique Azure AD avec SSO SAML pour Bitbucket de resolution GmbH, vous devez suivre les indications des blocs suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique SSO SAML pour Bitbucket de resolution GmbH](#configure-saml-sso-for-bitbucket-by-resolution-gmbh-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un SSO SAML pour Bitbucket de resolution GmbH](#create-saml-sso-for-bitbucket-by-resolution-gmbh-test-user)** pour avoir un équivalent de Britta Simon dans SSO SAML pour Bitbucket de resolution GmbH qui soit lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec SSO SAML pour Bitbucket de resolution GmbH, effectuez les étapes suivantes :

1. Sur le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **SSO SAML pour Bitbucket de resolution GmbH**, sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, suivez les étapes ci-dessous si vous souhaitez configurer l’application en mode démarré par **IDP** :

    ![Capture d’écran montrant Basic SAML Configuration, où vous pouvez entrer Identifier, et Reply URL, et sélectionner Save.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/samlsso`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/samlsso`

    c. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    ![Capture d’écran montrant Définir des URL supplémentaires, où vous pouvez entrer une URL de connexion.](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/samlsso`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez l’[équipe de support de SSO SAML pour Bitbucket de resolution GmbH](https://marketplace.atlassian.com/apps/1217045/saml-single-sign-on-sso-bitbucket?hosting=server&tab=support). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Sur la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

### <a name="configure-saml-sso-for-bitbucket-by-resolution-gmbh-single-sign-on"></a>Configurer l’authentification unique SSO SAML pour Bitbucket de resolution GmbH

1. Connectez-vous à votre site d’entreprise SSO SAML pour Bitbucket de resolution GmbH en tant qu’administrateur.

2. À droite dans la barre d’outils principale, cliquez sur **Paramètres**.

3. Accédez à la section COMPTES, cliquez sur **SAML SingleSignOn** sur la barre de menus.

    ![Samlsingle](./media/bitbucket-tutorial/tutorial_bitbucket_samlsingle.png)

4. Sur la **page Configuration du plug-in SAML SIngleSignOn**, cliquez sur **Ajouter idp**. 

    ![Ajout d’un fournisseur d’identité (idp)](./media/bitbucket-tutorial/tutorial_bitbucket_addidp.png)

5. Sur la page **Choisir votre fournisseur d’identité SAML**, procédez comme suit :

    ![Fournisseur d’identité](./media/bitbucket-tutorial/tutorial_bitbucket_identityprovider.png)

    a. Sélectionnez **Type d’Idp** en tant qu’**AD AZURE**.

    b. Dans la zone de texte **Nom**, tapez le nom.

    c. Dans la zone de texte **Description**, tapez la description.

    d. Cliquez sur **Suivant**.

6. Sur la page **Configuration du fournisseur d’identité**, cliquez sur le bouton **Suivant**.

    ![La configuration de l’identité](./media/bitbucket-tutorial/tutorial_bitbucket_identityconfig.png)

7.  Sur la page **Importer les métadonnées SAML Idp**, cliquez sur **Charger le fichier** pour charger le fichier **METADATA XML** que vous avez téléchargé à partir du portail Azure.

    ![idpmetadata](./media/bitbucket-tutorial/tutorial_bitbucket_idpmetadata.png)

8. Cliquez sur **Suivant**.

9. Cliquez sur **Enregistrer les paramètres**.

    ![Enregistrement](./media/bitbucket-tutorial/tutorial_bitbucket_save.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom**, entrez **BrittaSimon**.

    b. Dans le champ **Nom d’utilisateur**, tapez **brittasimon\@domainedevotreentreprise.extension**.  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous activez Britta Simon pour utiliser l’authentification unique Azure en l’autorisant à accéder à SSO SAML pour Bitbucket de resolution GmbH.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **SSO SAML pour Bitbucket de resolution GmbH**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, tapez et sélectionnez **SSO SAML pour Bitbucket de resolution GmbH**.

    ![La SSO SAML pour Bitbucket de resolution GmbH a un lien dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-saml-sso-for-bitbucket-by-resolution-gmbh-test-user"></a>Créer un utilisateur de test de SSO SAML pour Bitbucket de resolution GmbH

L’objectif de cette section est de créer un utilisateur appelé Britta Simon dans SSO SAML pour Bitbucket de resolution GmbH. SSO SAML pour Bitbucket de resolution GmbH prend en charge l’approvisionnement juste-à-temps et des utilisateurs peuvent également être créés manuellement. Contactez [équipe de support SSO SAML pour Bitbucket de resolution GmbH Client](https://marketplace.atlassian.com/plugins/com.resolution.atlasplugins.samlsso-bitbucket/server/support) en fonction de vos besoins.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette SSO SAML pour Bitbucket de resolution GmbH dans le panneau d’accès, vous devez être automatiquement connecté à l’application SSO SAML pour Bitbucket de resolution GmbH pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)