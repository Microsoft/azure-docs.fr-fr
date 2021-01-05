---
title: 'Tutoriel : Intégration d’Azure Active Directory avec Jitbit Helpdesk | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Jitbit Helpdesk.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/14/2019
ms.author: jeedes
ms.openlocfilehash: a1f000572badcaa09c958c7ca19ed81c4d0603bb
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92459503"
---
# <a name="tutorial-azure-active-directory-integration-with-jitbit-helpdesk"></a>Tutoriel : Intégration d’Azure Active Directory avec Jitbit Helpdesk

Dans ce didacticiel, vous allez apprendre à intégrer Jitbit Helpdesk à Azure Active Directory (Azure AD).
L’intégration de Jitbit Helpdesk dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Jitbit Helpdesk.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Nomadesk (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Jitbit Helpdesk, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Jitbit Helpdesk pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Jitbit Helpdesk prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-jitbit-helpdesk-from-the-gallery"></a>Ajout de Jitbit Helpdesk à partir de la galerie

Pour configurer l’intégration de Jitbit Helpdesk à Azure AD, vous devez ajouter Jitbit Helpdesk, disponible dans la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter Jitbit Helpdesk à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Jitbit Helpdesk** , sélectionnez **Jitbit Helpdesk** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Jitbit Helpdesk dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Jitbit Helpdesk avec un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Jitbit Helpdesk associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Jitbit Helpdesk, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Jitbit Helpdesk](#configure-jitbit-helpdesk-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Jitbit Helpdesk](#create-jitbit-helpdesk-test-user)** pour obtenir un équivalent de Britta Simon dans Jitbit Helpdesk lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Jitbit Helpdesk, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Jitbit Helpdesk** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Jitbit Helpdesk](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion** , entrez une URL au format suivant :
    | |
    | ----------------------------------------|
    | `https://<hostname>/helpdesk/User/Login`|
    | `https://<tenant-name>.Jitbit.com`|
    | |
    
    > [!NOTE] 
    > Cette valeur n’est pas la valeur réelle. Mettez à jour cette valeur avec l’URL d’authentification réelle. Contactez [l’équipe de support Jitbit Helpdesk](https://www.jitbit.com/support/) pour obtenir cette valeur.

    b. Dans la zone de texte **Identificateur (ID d’entité)** , tapez une URL comme suit :`https://www.jitbit.com/web-helpdesk/`

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer Jitbit Helpdesk** , copiez la ou les URL appropriées correspondant à vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-jitbit-helpdesk-single-sign-on"></a>Configurer l’authentification unique Jitbit Helpdesk

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise Jitbit Helpdesk en tant qu’administrateur.

1. Dans la barre d’outils située dans la partie supérieure, cliquez sur **Administration** .

    ![Administration](./media/jitbit-helpdesk-tutorial/ic777681.png "Administration")

1. Cliquez sur **General settings** .

    ![Capture d’écran montrant le lien General Settings.](./media/jitbit-helpdesk-tutorial/ic777680.png "Utilisateurs, sociétés et autorisations")

1. Dans la section de configuration **Authentication settings** , procédez comme suit :

    ![Paramètres d’authentification](./media/jitbit-helpdesk-tutorial/ic777683.png "Authentication settings")

    a. Sélectionnez **Activer l’authentification unique SAML 2.0** pour vous connecter à l’aide de l’authentification unique, avec **OneLogin** .

    b. Dans la zone de texte **EndPoint URL** (URL du point de terminaison), collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.

    c. Ouvrez votre certificat codé en **base 64** dans le Bloc-notes, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **Certificat X.509** .

    d. Cliquez sur **Save changes** .

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Jitbit Helpdesk.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Jitbit Helpdesk** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Jitbit Helpdesk** .

    ![Lien Jitbit Helpdesk dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-jitbit-helpdesk-test-user"></a>Créer un utilisateur de test Jitbit Helpdesk

Pour pouvoir se connecter à Jitbit Helpdesk, les utilisateurs d’Azure Active Directory doivent être provisionnés dans Jitbit Helpdesk. Dans le cas de Jitbit Helpdesk, l’approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre locataire **Jitbit Helpdesk** .

1. Dans le menu situé dans la partie supérieure, cliquez sur **Administration** .

    ![Administration](./media/jitbit-helpdesk-tutorial/ic777681.png "Administration")

1. Cliquez sur **Users, companies and permissions** .

    ![Utilisateurs, sociétés et autorisations](./media/jitbit-helpdesk-tutorial/ic777682.png "Utilisateurs, sociétés et autorisations")

1. Cliquez sur **Add User** .

    ![Ajouter un utilisateur](./media/jitbit-helpdesk-tutorial/ic777685.png "Ajouter un utilisateur")

1. Dans la section Créer, entrez les données du compte Azure AD que vous souhaitez configurer comme suit :

    ![Créer](./media/jitbit-helpdesk-tutorial/ic777686.png "Créer")

   a. Dans la zone de texte **Username** (Nom d’utilisateur), tapez le nom de l’utilisateur, par exemple, **BrittaSimon** .

   b. Dans la zone de texte **E-mail** , entrez l’e-mail de l’utilisateur, par exemple, **BrittaSimon@contoso.com** .

   c. Dans la zone de texte **Prénom** , entrez le prénom de l’utilisateur, par exemple **Britta** .

   d. Dans la zone de texte **Nom** , entrez le nom de l’utilisateur, par exemple **Simon** .

   e. Cliquez sur **Créer** .

> [!NOTE]
> Vous pouvez d’autres outils ou API de création de compte d’utilisateur fournis par Jitbit Helpdesk pour configurer des comptes d’utilisateurs Azure AD.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Jitbit Helpdesk dans le volet d’accès, vous devez être connecté automatiquement à l’application Jitbit Helpdesk pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)