---
title: 'Tutoriel : Intégration d’Azure Active Directory à ScreenSteps | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et ScreenSteps.
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
ms.openlocfilehash: 3816239798290318404980ded388b726d8134395
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999788"
---
# <a name="tutorial-azure-active-directory-integration-with-screensteps"></a>Tutoriel : Intégration d’Azure AD à ScreenSteps

Dans ce didacticiel, vous allez apprendre à intégrer ScreenSteps à Azure Active Directory (Azure AD).
L’intégration de ScreenSteps à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à ScreenSteps.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à ScreenSteps (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à ScreenSteps, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement ScreenSteps pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* ScreenSteps prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-screensteps-from-the-gallery"></a>Ajout de ScreenSteps à partir de la galerie

Pour configurer l’intégration de ScreenSteps à Azure AD, vous devez ajouter ScreenSteps à partir de la galerie à votre liste d’applications SaaS managées.

**Pour ajouter ScreenSteps à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **ScreenSteps**, sélectionnez **ScreenSteps** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![ScreenSteps dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec ScreenSteps avec un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur ScreenSteps associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec ScreenSteps, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique ScreenSteps](#configure-screensteps-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test ScreenSteps](#create-screensteps-test-user)** pour avoir un équivalent de Britta Simon dans ScreenSteps lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec ScreenSteps, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **ScreenSteps**, sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL ScreenSteps](common/sp-signonurl.png)

    Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://<tenantname>.ScreenSteps.com`

    > [!NOTE]
    > Cette valeur n’est pas la valeur réelle. Vous devez remplacer cette valeur par l’URL de connexion réelle. La procédure est expliquée plus loin dans le didacticiel.

5. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer ScreenSteps**, copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-screensteps-single-sign-on"></a>Configurer l’authentification unique ScreenSteps

1. Dans une autre fenêtre de navigateur web, connectez-vous au site de votre entreprise ScreenSteps en tant qu’administrateur.

1. Cliquez sur **Account Settings** (Paramètres du compte).

    ![Gestion du compte](./media/screensteps-tutorial/ic778523.png "Account management")

1. Cliquez sur **Single Sign-on**.

    ![Capture d’écran montrant l’option « Single Sign-on » sélectionnée.](./media/screensteps-tutorial/ic778524.png "Remote Authentication")

1. Cliquez sur **Créer un point de terminaison d’authentification unique**.

    ![Authentification à distance](./media/screensteps-tutorial/ic778525.png "Remote Authentication")

1. Dans la section **Créer un point de terminaison d’authentification unique**, procédez comme suit :

    ![Créer un point de terminaison d’authentification](./media/screensteps-tutorial/ic778526.png "Create an Authentication Endpoint")

    a. Dans la zone de texte **Title** , tapez un titre.

    b. Dans la liste **Mode**, sélectionnez **SAML**.

    c. Cliquez sur **Créer**.

1. Cliquez sur **Edit** (Modifier) pour modifier le nouveau point de terminaison.

    ![Modifier le point de terminaison](./media/screensteps-tutorial/ic778528.png "Ajouter un point de terminaison")

1. Dans la section **Edit Single Sign-on Endpoint (Modifier un point de terminaison d’authentification unique)** , procédez comme suit :

    ![Point de terminaison d’authentification distant](./media/screensteps-tutorial/ic778527.png "Remote Authentication Endpoint")

    a. Cliquez sur **Upload new SAML Certificate file** (Charger le nouveau fichier de certificat SAML), puis chargez le certificat que vous avez téléchargé à partir du portail Azure.

    b. Collez la valeur **URL de connexion**, que vous avez copiée dans le portail Azure, dans la zone de texte **Remote Login URL** (URL d’ouverture de session à distance).

    c. Collez la valeur **URL de déconnexion** que vous avez copiée à partir du portail Azure dans la zone de texte **Log out URL** (URL de déconnexion).

    d. Sélectionnez un **groupe** auquel attribuer des utilisateurs lorsque ces derniers sont approvisionnés.

    e. Cliquez sur **Update**.

    f. Copiez la valeur **SAML Consumer URL** (URL du consommateur SAML) dans le Presse-papiers, puis collez-la dans la zone de texte **URL de connexion** située dans la section **Configuration SAML de base** sur le portail Azure.

    g. Revenez à **Edit Single Sign-on Endpoint (Modifier un point de terminaison d’authentification unique)** .

    h. Cliquez sur le bouton **Make default for account** (Configurer par défaut pour le compte) pour utiliser ce point de terminaison pour tous les utilisateurs qui se connectent à ScreenSteps. Vous pouvez également cliquer sur le bouton **Ajouter au site** pour utiliser ce point de terminaison pour des sites spécifiques dans **ScreenSteps**.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom**, entrez **BrittaSimon**.
  
    b. Dans le champ **Nom d’utilisateur**, tapez **brittasimon@yourcompanydomain.extension**  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à ScreenSteps.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis sélectionnez **ScreenSteps**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **ScreenSteps**.

    ![Lien ScreenSteps dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-screensteps-test-user"></a>Créer un utilisateur de test ScreenSteps

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans ScreenSteps. Pour ajouter des utilisateurs à la plateforme ScreenSteps, contactez [l’équipe du support ScreenSteps](https://www.screensteps.com/contact). Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette ScreenSteps dans le panneau d’accès doit vous connecter automatiquement à l’application ScreenSteps pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)