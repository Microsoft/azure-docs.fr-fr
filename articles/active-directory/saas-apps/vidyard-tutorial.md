---
title: 'Didacticiel : Intégration d’Azure Active Directory à Vidyard | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Vidyard.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/28/2019
ms.author: jeedes
ms.openlocfilehash: 4176c92d48b67b9f9207f22ebd8939b5ec1437ee
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92636746"
---
# <a name="tutorial-azure-active-directory-integration-with-vidyard"></a>Didacticiel : Intégration d’Azure AD à Vidyard

Dans ce didacticiel, vous allez apprendre à intégrer Vidyard à Azure Active Directory (Azure AD).
L’intégration de Vidyard à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Vidyard.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Vidyard (authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Vidyard, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Abonnement Vidyard avec l’authentification unique activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Vidyard prend en charge l’authentification unique (SSO) lancée par le **fournisseur de services** et le **fournisseur d’identité**

* Vidyard prend en charge le provisionnement d’utilisateurs **Juste-à-temps**

## <a name="adding-vidyard-from-the-gallery"></a>Ajout de Vidyard depuis la galerie

Pour configurer l’intégration de Vidyard avec Azure AD, vous devez ajouter Vidyard à votre liste d’applications SaaS gérées depuis la galerie.

**Pour ajouter Vidyard depuis la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Vidyard** , sélectionnez **Vidyard** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Vidyard dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous configurez et testez l’authentification unique Azure AD avec Vidyard à l’aide d’un utilisateur de test, **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur Vidyard associé doit être établie.

Pour configurer et tester l'authentification unique Azure AD avec Vidyard, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Vidyard](#configure-vidyard-single-sign-on)**  : pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Vidyard](#create-vidyard-test-user)**  : pour avoir un équivalent de Britta Simon dans Vidyard lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Vidyard, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Vidyard** , sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , effectuez les étapes suivantes :

    ![Capture d’écran montrant Configuration SAML de base, où vous pouvez entrer l’identificateur, l’URL de réponse, et sélectionner Enregistrer.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://secure.vidyard.com/sso/saml/<unique id>/metadata`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://secure.vidyard.com/sso/saml/<unique id>/consume`

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    ![Capture d’écran montrant Définir des URL supplémentaires, où vous pouvez entrer une URL de connexion.](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://secure.vidyard.com/sso/saml/<unique id>/login`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Vous mettrez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. La procédure est expliquée plus loin dans le didacticiel. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

6. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

7. Dans la section **Configurer Vidyard** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-vidyard-single-sign-on"></a>Configurer l’authentification unique Vidyard

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise Vidyard Software comme administrateur.

2. Dans le tableau de bord Vidyard, sélectionnez **Groupe** > **Sécurité**

    ![Capture d’écran montrant Security sélectionné à partir de Group sur le site Vidyard Software.](./media/vidyard-tutorial/configure1.png)

3. Cliquez sur l’onglet **Nouveau profil**.

    ![Capture d’écran montrant le bouton Nouveau profil.](./media/vidyard-tutorial/configure2.png)

4. Dans la section **Configuration de SAML** , procédez comme suit :

    ![Capture d’écran montrant la section SAML Configuration où vous pouvez entrer les valeurs décrites.](./media/vidyard-tutorial/configure3.png)

    a. Entrez le nom du profil général dans la zone de texte **Nom du profil**.

    b. Copiez la valeur **SSO User Login Page** (Page de connexion utilisateur SSO), collez-la dans la zone de texte **URL de connexion** dans la section **Configuration SAML de base** dans le portail Azure.

    c. Copiez la valeur **URL ACS** et collez-la dans la zone de texte **URL de réponse** de la section **Configuration SAML de base** du portail Azure.

    d. Copiez la valeur **URL des métadonnées/de l’émetteur** et collez-la dans la zone de texte **Identificateur** de la section **Configuration SAML de base** du portail Azure.

    e. Dans le Bloc-notes, ouvrez le fichier de certificat que vous avez téléchargé sur le portail Azure, copiez son contenu, puis collez-le dans la zone de texte **Certificat X.509**.

    f. Dans la zone de texte **URL de point de terminaison SAML** , collez la valeur de l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    g. Cliquez sur **Confirmer**.

5. Dans l’onglet Authentification unique, sélectionnez **Attribuer** en regard d’un profil existant

    ![Capture d’écran montrant le bouton Attribuer pour le profil Authentification unique Azure AD.](./media/vidyard-tutorial/configure4.png)

    > [!NOTE]
    > Une fois le profil SSO créé, attribuez-le aux groupes auxquels les utilisateurs devront accéder via Azure. Si un utilisateur n’existe pas dans le groupe auquel il est attribué, Vidyard créer automatiquement un compte d’utilisateur et attribue le rôle en temps réel.

6. Sélectionnez le groupe de votre organisation, qui est visible dans **Groups Available to Assign** (Groupes disponibles pour l’attribution).

    ![Capture d’écran montrant la section Assign SAML Configuration to Organizations où vous pouvez sélectionner votre groupe.](./media/vidyard-tutorial/configure5.png)

7. Vous pouvez voir les groupes attribués sous **Groups Currently Assigned** (Groupes actuellement attribués). Sélectionnez un rôle pour le groupe de votre organisation et cliquez sur **Confirmer**.

    ![Capture d’écran montrant la section Assign SAML Configuration to Organizations où vous pouvez sélectionner un rôle.](./media/vidyard-tutorial/configure6.png)

    > [!NOTE]
    > Pour plus d’informations, consultez [ce document](https://knowledge.vidyard.com/hc/articles/360009990033-SAML-based-Single-Sign-On-SSO-in-Vidyard).

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon**.
  
    b. Dans le champ **Nom d’utilisateur** , tapez brittasimon@yourcompanydomain.extension. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Vidyard.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis sélectionnez **Vidyard**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Vidyard**.

    ![Lien Vidyard dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

### <a name="create-vidyard-test-user"></a>Créer un utilisateur de test Vidyard

Dans cette section, un utilisateur appelé Britta Simon est créé dans Vidyard. Vidyard prend en charge le provisionnement d’utilisateurs juste-à-temps, activé par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Vidyard, un nouveau est créé après l’authentification.

>[!Note]
>Si vous devez créer un utilisateur manuellement, contactez [l’équipe de support Vidyard](mailto:support@vidyard.com).

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Vidyard dans le panneau d’accès, vous devez être automatiquement connecté à l’application Vidyard pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)