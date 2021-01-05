---
title: 'Didacticiel : Intégration d’Azure Active Directory à SmartRecruiters | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SmartRecruiters.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/07/2019
ms.author: jeedes
ms.openlocfilehash: b7de874cc074faa63203c58638fb04cc5cb2282b
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92514926"
---
# <a name="tutorial-azure-active-directory-integration-with-smartrecruiters"></a>Didacticiel : Intégration d’Azure Active Directory à SmartRecruiters

Dans ce didacticiel, vous apprenez à intégrer SmartRecruiters à Azure Active Directory (Azure AD).
L’intégration de SmartRecruiters à Azure AD vous offre les avantages suivants :

* Vous pouvez contrôler dans Azure AD qui a accès à SmartRecruiters.
* Vous pouvez permettre à vos utilisateurs de se connecter automatiquement à SmartRecruiters (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à SmartRecruiters, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement SmartRecruiters pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* SmartRecruiters prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité** .

## <a name="adding-smartrecruiters-from-the-gallery"></a>Ajout de SmartRecruiters à partir de la galerie

Pour configurer l’intégration de SmartRecruiters à Azure AD, vous devez ajouter SmartRecruiters à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter SmartRecruiters à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **SmartRecruiters** , sélectionnez **SmartRecruiters** dans le volet des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![SmartRecruiters dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous configurez et vous testez l’authentification unique Azure AD avec SmartRecruiters, avec un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur SmartRecruiters associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec SmartRecruiters, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique SmartRecruiters](#configure-smartrecruiters-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test SmartRecruiters](#create-smartrecruiters-test-user)** pour avoir un équivalent de Britta Simon dans SmartRecruiters lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec SmartRecruiters, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **SmartRecruiters** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. À la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode initié par **IDP** , suivez les étapes ci-dessous :

    ![Capture d’écran montrant Configuration SAML de base, où vous pouvez entrer l’identificateur, l’URL de réponse, et sélectionner Enregistrer.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://www.smartrecruiters.com/web-sso/saml/<companyname>`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://www.smartrecruiters.com/web-sso/saml/<companyname>/callback`

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    ![Capture d’écran montrant Définir des URL supplémentaires, où vous pouvez entrer une URL de connexion.](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://www.smartrecruiters.com/web-sso/saml/<companyname>/login`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez [l’équipe de support technique SmartRecruiters](https://www.smartrecruiters.com/about-us/contact-us/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

6. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

7. Dans la section **Configurer SmartRecruiters** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-smartrecruiters-single-sign-on"></a>Configurer l’authentification unique de SmartRecruiters

1. Dans une autre fenêtre de navigateur web, connectez-vous au site de votre entreprise SmartRecruiters en tant qu’administrateur.

1. Accédez à **Paramètres / Administrateur** .

    ![Capture d’écran montrant l’élément Settings / Admin sélectionné à partir d’un menu.](./media/smartrecruiters-tutorial/configure.png)

1. Dans la section **Configuration** , cliquez sur **Authentification unique web** .

    ![Capture d’écran montrant la sélection de Web SSO dans Configuration.](./media/smartrecruiters-tutorial/configure1.png)

1. Choisissez **Activer l’authentification unique web** .

    ![La capture d’écran montre le contrôle Enable Web SSO.](./media/smartrecruiters-tutorial/configure2.png)

1. Dans la section **Configuration du fournisseur d’identité** , procédez comme suit :

    ![Capture d’écran montrant la section Identity Provider Configuration, dans laquelle vous pouvez indiquer les valeurs décrites.](./media/smartrecruiters-tutorial/configure4.png)

    a. Dans la zone de texte **URL du fournisseur d’identité** , collez la valeur de l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    b. Ouvrez le **Certificat (en base64)** que vous avez téléchargé à partir du portail Azure dans le Bloc-notes, copiez son contenu et collez-le dans la zone de texte **Certificat du fournisseur d’identité** .

1. Cliquez sur **Enregistrer la configuration de l’authentification unique web** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon@yourcompanydomain.extension**  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à SmartRecruiters.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis sélectionnez **SmartRecruiters** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **SmartRecruiters** .

    ![Lien SmartRecruiters dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-smartrecruiters-test-user"></a>Créer un utilisateur test de SmartRecruiters

Dans cette section, vous créez un utilisateur appelé Britta Simon dans SmartRecruiters. Contactez [l’équipe de support technique de SmartRecruiters](https://www.smartrecruiters.com/about-us/contact-us/) pour ajouter les utilisateurs sur la plateforme SmartRecruiters. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette SmartRecruiters dans le volet d’accès, vous devez être connecté automatiquement à l’application SmartRecruiters pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)