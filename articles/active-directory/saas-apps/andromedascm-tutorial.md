---
title: 'Didacticiel : intégration d’Azure Active Directory à Andromeda | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Andromeda.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/16/2019
ms.author: jeedes
ms.openlocfilehash: e8cb939b48f8cfe311ec10c0850cfb234de04fad
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97589744"
---
# <a name="tutorial-azure-active-directory-integration-with-andromeda"></a>Didacticiel : intégration d’Azure Active Directory à Andromeda

Dans ce didacticiel, vous apprenez à intégrer Andromeda à Azure Active Directory (Azure AD).
L’intégration d’Andromeda à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Andromeda.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Andromeda (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Andromeda, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Abonnement Andromeda pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Andromeda prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité**
* Andromeda prend en charge le provisionnement d’utilisateurs **juste-à-temps**

## <a name="adding-andromeda-from-the-gallery"></a>Ajout d’Andromeda à partir de la galerie

Pour configurer l’intégration d’Andromeda à Azure AD, vous devez ajouter Andromeda, disponible dans la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter Andromeda à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, saisissez **Andromeda**, sélectionnez **Andromeda** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Andromeda dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous configurez et testez l’authentification unique Azure AD avec Andromeda à l’aide d’un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur Andromeda associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Andromeda, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Andromeda](#configure-andromeda-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Andromeda](#create-andromeda-test-user)** pour avoir un équivalent de Britta Simon dans Andromeda, associé à sa représentation dans Azure AD.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Andromeda, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), sur la page d’intégration de l’application **Andromeda**, sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité**, effectuez les étapes suivantes :

    ![Capture d’écran montrant Configuration SAML de base, où vous pouvez entrer l’identificateur, l’URL de réponse, et sélectionner Enregistrer.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `https://<tenantURL>.ngcxpress.com/`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://<tenantURL>.ngcxpress.com/SAMLConsumer.aspx`

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    ![Capture d’écran montrant Définir des URL supplémentaires, où vous pouvez entrer une URL de connexion.](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://<tenantURL>.ngcxpress.com/SAMLLogon.aspx`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Vous mettrez à jour la valeur avec l’identificateur, l’URL de réponse et l’URL de connexion réels. La procédure est expliquée plus loin dans le didacticiel.

6. L’application Andromeda attend les assertions SAML dans un format spécifique. Configurez les revendications suivantes pour cette application. Vous pouvez gérer les valeurs de ces attributs à partir de la section **Attributs utilisateur** sur la page d’intégration des applications. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur le bouton **Modifier** pour ouvrir la boîte de dialogue **Attributs utilisateur**.

    ![Capture d’écran montrant des attributs utilisateur tels que givenname user.givenname et emailaddress user.mail.](common/edit-attribute.png)

    > [!Important]
    > Effacez les définitions NameSpace lorsque vous configurez celles-ci.

7. Dans la section **Revendications des utilisateurs** de la boîte de dialogue **Attributs utilisateur**, modifiez les revendications en utilisant l’icône **Modifier** ou ajoutez des revendications en utilisant l’option **Ajouter une nouvelle revendication** pour configurer l’attribut de jeton SAML comme sur l’image ci-dessus et procédez comme suit : 

    | Name | Attribut source|
    | ------ | -----------|
    | rôle | Rôle spécifique aux applications |
    | type | Type d'application |
    | société | CompanyName |

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Ces valeurs sont fournies uniquement à des fins de démonstration ; veuillez utiliser les rôles de votre organisation.

    1. Cliquez sur le bouton **Ajouter une nouvelle revendication** pour ouvrir la boîte de dialogue **Gérer les revendications des utilisateurs**.

        ![Capture d’écran montrant Revendications des utilisateurs avec des options permettant d’ajouter une nouvelle revendication et d’enregistrer.](common/new-save-attribute.png)

        ![Capture d’écran montrant Gérer les revendications des utilisateurs, où vous pouvez entrer les valeurs décrites à cette étape.](common/new-attribute-details.png)

    1. Dans la zone de texte **Attribut**, indiquez le nom d’attribut pour cette ligne.

    1. Laissez le champ **Espace de noms** vide.

    1. Sélectionnez Source comme **Attribut**.

    1. Dans la liste **Attribut de la source**, tapez la valeur d’attribut indiquée pour cette ligne.

    1. Cliquez sur **OK**

    1. Cliquez sur **Enregistrer**.

8. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

9. Dans la section **Configurer Andromeda**, copiez la ou les URL appropriées, selon vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    1. URL de connexion

    1. Identificateur Azure AD

    1. URL de déconnexion

### <a name="configure-andromeda-single-sign-on"></a>Configurer l’authentification unique Andromeda

1. Connectez-vous à votre site d’entreprise Andromeda en tant qu’administrateur.

2. En haut de la barre de menus, cliquez sur **Admin** et accédez à **Administration**.

    ![Administrateur Andromeda](./media/andromedascm-tutorial/tutorial_andromedascm_admin.png)

3. Sur le côté gauche de la barre d’outils, dans la section **Interfaces**, cliquez sur **SAML Configuration** (Configuration SAML).

    ![SAML Andromeda](./media/andromedascm-tutorial/tutorial_andromedascm_saml.png)

4. Dans la section **SAML Configuration** (Configuration SAML), procédez comme suit :

    ![Configuration d’Andromeda](./media/andromedascm-tutorial/tutorial_andromedascm_config.png)

    1. Cochez **Enable SSO with SAML** (Activer l’authentification unique avec SAML).

    1. Dans la section **Informations Andromeda**, copiez la valeur **Identité SP** et collez-la dans la zone de texte **Identificateur** de la section **Configuration SAML de base**.

    1. Copiez la valeur **URL du consommateur** et collez-la dans la zone de texte **URL de réponse** de la section **Configuration SAML de base**.

    1. Copiez la valeur **URL d’ouverture de session** et collez-la dans la zone de texte **URL de connexion** de la section **Configuration SAML de base**.

    1. Dans la section **Fournisseur d’identité SAML**, tapez votre nom IDP.

    1. Dans la zone de texte **Point de terminaison du service d’authentification unique**, collez la valeur de l’**URL de connexion** que vous avez copiée à partir du portail Azure.

    1. Ouvrez le **certificat codé en base 64** téléchargé à partir du portail Azure dans le bloc-notes et collez-le dans la zone de texte **Certificat X.509**.

    1. Mappez les attributs suivants avec la valeur correspondante pour faciliter la connexion avec authentification unique à partir d’Azure AD. L’attribut **ID utilisateur** est requis pour la connexion. Pour l’approvisionnement, **E-mail**, **Société**, **Type d’utilisateur** et **Rôle** sont requis. Dans cette section, nous définissons le mappage des attributs (nom et valeurs) qui correspondent à ceux définis dans le portail Azure

        ![Andromeda attbmap](./media/andromedascm-tutorial/tutorial_andromedascm_attbmap.png)

    1. Cliquez sur **Enregistrer**.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom**, entrez **BrittaSimon**.

    b. Dans le champ **Nom d’utilisateur**, tapez `brittasimon@yourcompanydomain.extension`. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, autorisez Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Andromeda.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **Andromeda**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Andromeda**.

    ![Le lien Andromeda dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-andromeda-test-user"></a>Créer un utilisateur de test Andromeda

Dans cette section, un utilisateur appelé Britta Simon est créé dans Andromeda. Andromeda prend en charge le provisionnement d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Andromeda, il en est créé un après l’authentification. Si vous devez créer un utilisateur manuellement, contactez [l’équipe d’assistance client Andromeda](https://www.ngcsoftware.com/support/).

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Andromeda dans le panneau d’accès doit vous connecter automatiquement à l’application Andromeda pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)