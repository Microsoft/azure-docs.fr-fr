---
title: 'Didacticiel : Intégration d’Azure Active Directory avec eDigitalResearch | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et eDigitalResearch.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/01/2019
ms.author: jeedes
ms.openlocfilehash: de794a94e23bda67d617e5d3d6f4634690b78e3e
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92454303"
---
# <a name="tutorial-azure-active-directory-integration-with-edigitalresearch"></a>Didacticiel : Intégration d’Azure Active Directory avec eDigitalResearch

Dans ce didacticiel, vous allez apprendre à intégrer eDigitalResearch dans Azure Active Directory (Azure AD).
L’intégration d’eDigitalResearch à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à eDigitalResearch.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à eDigitalResearch (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec eDigitalResearch, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Un abonnement eDigitalResearch pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* eDigitalResearch prend en charge l’authentification unique initiée par le **fournisseur d’identité**

## <a name="adding-edigitalresearch-from-the-gallery"></a>Ajout d’eDigitalResearch à partir de la galerie

Pour configurer l’intégration d’eDigitalResearch avec Azure AD, vous devez ajouter eDigitalResearch, disponible dans la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter eDigitalResearch à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **eDigitalResearch** , sélectionnez **eDigitalResearch** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![eDigitalResearch dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec eDigitalResearch à l’aide d’un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, vous devez établir une relation entre l’utilisateur Azure AD et l’utilisateur eDigitalResearch associé.

Pour configurer et tester l’authentification unique Azure AD avec eDigitalResearch, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique eDigitalResearch](#configure-edigitalresearch-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test eDigitalResearch](#create-edigitalresearch-test-user)** pour avoir dans eDigitalResearch un équivalent de Britta Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec eDigitalResearch, effectuez les étapes suivantes :

1. Sur le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **eDigitalResearch** , sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Sur la page **Configurer l’authentification unique avec SAML** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans eDigitalResearch Domain and URLs (Domaine et URL eDigitalResearch)](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<company-name>.edigitalresearch.com`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<company-name>.edigitalresearch.com/login/consume`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez l’[équipe du support eDigitalResearch](https://www.maruedr.com/contact). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer eDigitalResearch** , copiez la ou les URL appropriées correspondant à vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-edigitalresearch-single-sign-on"></a>Configurer l’authentification unique eDigitalResearch

Pour configurer l’authentification unique côté **eDigitalResearch** , vous devez envoyer le **certificat (Base64)** téléchargé et les URL correspondantes copiées à partir du portail Azure à l’ [équipe du support technique eDigitalResearch](https://www.maruedr.com/contact). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon**.
  
    b. Dans le champ **Nom d’utilisateur** , tapez `brittasimon@yourcompanydomain.extension`. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à eDigitalResearch.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **eDigitalResearch**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **eDigitalResearch**.

    ![Lien eDigitalResearch dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

### <a name="create-edigitalresearch-test-user"></a>Créer un utilisateur de test eDigitalResearch

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans eDigitalResearch. Contactez l’[équipe du support eDigitalResearch](https://www.maruedr.com/contact) pour ajouter des utilisateurs à la plateforme eDigitalResearch. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

 > [!NOTE]
 > Le titulaire du compte Azure Active Directory reçoit un e-mail contenant un lien à suivre pour confirmer son compte et l’activer.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette eDigitalResearch dans le panneau d’accès doit vous connecter automatiquement à l’application eDigitalResearch pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)