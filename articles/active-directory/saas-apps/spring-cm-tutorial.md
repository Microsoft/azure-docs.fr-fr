---
title: 'Tutoriel : Intégration d’Azure Active Directory à SpringCM | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SpringCM.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/08/2019
ms.author: jeedes
ms.openlocfilehash: 9cfc48e3fdb96ba5b63b28288a801095f7b36f43
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97589829"
---
# <a name="tutorial-azure-active-directory-integration-with-springcm"></a>Tutoriel : Intégration d’Azure Active Directory à SpringCM

Dans ce didacticiel, vous allez apprendre à intégrer SpringCM à Azure Active Directory (Azure AD).
L’intégration de SpringCM dans Azure AD offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à SpringCM.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à SpringCM (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à SpringCM, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Abonnement SpringCM pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* SpringCM prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-springcm-from-the-gallery"></a>Ajout de SpringCM à partir de la galerie

Pour configurer l’intégration de SpringCM avec Azure AD, vous devez ajouter SpringCM, disponible dans la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter SpringCM à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **SpringCM**, sélectionnez **SpringCM** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![SpringCM dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec SpringCM à l’aide d’un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur SpringCM associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec SpringCM, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique SpringCM](#configure-springcm-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test SpringCM](#create-springcm-test-user)** pour avoir un équivalent de Britta Simon dans SpringCM lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec SpringCM, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **SpringCM**, sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL SpringCM](common/sp-signonurl.png)

    Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://na11.springcm.com/atlas/SSO/SSOEndpoint.ashx?aid=<identifier>`

    > [!NOTE]
    > Cette valeur n’est pas la valeur réelle. Mettez à jour la valeur avec l’URL de connexion réelle. Pour obtenir la valeur, contactez l’[équipe du support client SpringCM](https://knowledge.springcm.com/support). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

4. Dans la page **Configurer l’authentification unique avec SAML**, à la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Raw)** en fonction des options définies, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificateraw.png)

6. Dans la section **Configurer SpringCM**, copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-springcm-single-sign-on"></a>Configurer l’authentification unique SpringCM

1. Dans une autre fenêtre de navigateur web, connectez-vous au site de votre entreprise **SpringCM** en tant qu’administrateur.

1. Dans le menu en haut, cliquez sur **GO TO**, sur **Preferences** puis, dans la section **Account Preferences**, cliquez sur **SAML SSO**.

    ![Authentification unique SAML](./media/spring-cm-tutorial/ic797051.png "SAML SSO")

1. Dans la section Identity Provider Configuration, procédez comme suit :

    ![Configuration du fournisseur d’identité](./media/spring-cm-tutorial/ic797052.png "Identity Provider Configuration")

    a. Pour charger votre certificat Azure Active Directory téléchargé, cliquez sur **Select Issuer Certificate** ou **Change Issuer Certificate**.

    b. Dans la zone de texte **Issuer** (Émetteur), collez l’**Identificateur Azure AD** que vous avez copié à partir du portail Azure.

    c. Dans la zone de texte **Service Provider (SP) Initiated Endpoint** (Point de terminaison lancé par le fournisseur de services), collez l’**URL de connexion** que vous avez copiée à partir du portail Azure.

    d. Sélectionnez **Enable** pour **SAML Enabled**.

    e. Cliquez sur **Enregistrer**.

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à SpringCM.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **SpringCM**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **SpringCM**.

    ![Lien SpringCM dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-springcm-test-user"></a>Créer un utilisateur de test SpringCM

Pour se connecter à SpringCM, les utilisateurs Azure Active Directory doivent être provisionnés dans SpringCM. Dans le cas de SpringCM, l’approvisionnement est une tâche manuelle.

> [!NOTE]
> Pour plus d’informations, consultez [Créer et modifier un utilisateur SpringCM](http://community.springcm.com/s/article/Create-and-Edit-a-SpringCM-User-1619481053). 

**Pour approvisionner un compte d’utilisateur dans SpringCM, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise **SpringCM** en tant qu’administrateur.

1. Cliquez sur **GOTO** puis sur **ADDRESS BOOK**.

    ![Create User](./media/spring-cm-tutorial/ic797054.png "Create User") (Créer un utilisateur)

1. Cliquez sur **Créer l’utilisateur**.

1. Sélectionnez un rôle d’utilisateur dans **User Role**.

1. Sélectionnez **Send Activation Email**.

1. Indiquez le prénom, le nom et l’adresse e-mail du compte Azure AD valide que vous souhaitez approvisionner dans les zones de texte correspondantes.

1. Ajoutez l’utilisateur à un groupe de sécurité dans **Security group**.

1. Cliquez sur **Enregistrer**.

   > [!NOTE]
   > Vous pouvez utiliser tout autre outil ou API de création de compte d’utilisateur SpringCM fourni par SpringCM pour provisionner des comptes d’utilisateur Azure AD.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette SpringCM dans le panneau d’accès doit vous connecter automatiquement à l’application SpringCM pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)