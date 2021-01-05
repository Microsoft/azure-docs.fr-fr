---
title: Didacticiel :Intégration d’Azure Active Directory à TalentLMS | Microsoft Docs
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et TalentLMS.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/10/2019
ms.author: jeedes
ms.openlocfilehash: a5bca193a54d63feb927070fe138464581665ac9
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92503893"
---
# <a name="tutorial-azure-active-directory-integration-with-talentlms"></a>Didacticiel : Intégration d’Azure Active Directory à TalentLMS

Dans ce didacticiel, vous allez apprendre à intégrer TalentLMS à Azure Active Directory (Azure AD).
L’intégration de TalentLMS à Azure AD vous fait bénéficier des avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à TalentLMS.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à TalentLMS (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à TalentLMS, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Abonnement TalentLMS pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* TalentLMS prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-talentlms-from-the-gallery"></a>Ajout de TalentLMS à partir de la galerie

Pour configurer l’intégration de TalentLMS à Azure AD, vous devez ajouter TalentLMS à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter TalentLMS à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **TalentLMS** , sélectionnez **TalentLMS** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![TalentLMS dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous configurez et testez l’authentification unique Azure AD avec TalentLMS pour un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans TalentLMS.

Pour configurer et tester l’authentification unique Azure AD auprès de TalentLMS, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique TalentLMS](#configure-talentlms-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test TalentLMS](#create-talentlms-test-user)** pour avoir dans TalentLMS un équivalent de Britta Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec TalentLMS, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **TalentLMS** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL TalentLMS](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<tenant-name>.TalentLMSapp.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `http://<tenant-name>.talentlms.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe de support client de TalentLMS](https://www.talentlms.com/contact). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la section **Certificat de signature SAML** , cliquez sur le bouton **Modifier** pour ouvrir la boîte de dialogue **Certificat de signature SAML** .

    ![Modifier le certificat de signature SAML](common/edit-certificate.png)

6. Dans la section **Certificat de signature SAML** , copiez l’ **EMPREINTE** et enregistrez-la sur votre ordinateur.

    ![Copier la valeur de l’empreinte](common/copy-thumbprint.png)

7. Dans la section **Configurer TalentLMS** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-talentlms-single-sign-on"></a>Configurer l’authentification unique pour TalentLMS

1. Dans une autre fenêtre de navigateur web, connectez-vous au site de votre entreprise TalentLMS en tant qu’administrateur.

1. Dans la section **Account & Settings (Compte et paramètres)** , cliquez sur l’onglet **Users (Utilisateurs)** .

    ![Accounts & Setting](./media/talentlms-tutorial/IC777296.png "Account & Settings") (Comptes et paramètres)

1. Cliquez sur **Single Sign-On (SSO)** .

1. Dans la section Single Sign-On, procédez comme suit :

    ![Authentification unique](./media/talentlms-tutorial/IC777297.png "Single Sign on")

    a. Dans la liste **SSO integration type (Type d’intégration SSO)** , sélectionnez **SAML 2.0** .

    b. Dans la zone de texte **Identity provider (IDP)** (Fournisseur d’identité IDP), collez la valeur **Azure AD Identifier** (Identificateur Azure AD) que vous avez copiée à partir du portail Azure.

    c. Collez la valeur de l’ **empreinte** à partir du portail Azure dans la zone de texte **Empreinte du certificat** .

    d.  Dans la zone de texte **Remote sign-in URL** (URL de connexion à distance), collez l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    e. Dans la zone de texte **Remote sign-out URL** (URL de déconnexion à distance), collez l’ **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    f. Renseignez les informations suivantes :

    * Dans la zone de texte **TargetedID** , tapez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`

    * Dans la zone de texte **Prénom** , tapez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`

    * Dans la zone de texte **Nom** , tapez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`

    * Dans la zone de texte **Email** , tapez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`

1. Cliquez sur **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez `brittasimon@yourcompanydomain.extension`. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à TalentLMS.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **TalentLMS** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **TalentLMS** .

    ![Lien TalentLMS dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-talentlms-test-user"></a>Créer un utilisateur de test TalentLMS

Pour permettre aux utilisateurs Azure AD de se connecter à TalentLMS, vous devez les provisionner dans TalentLMS. Dans le cas de TalentLMS, l’approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre locataire **TalentLMS** .

1. Cliquez sur **Users (Utilisateurs)** , puis sur **Add User (Ajouter un utilisateur)** .

1. Dans la boîte de dialogue **Add user** , procédez comme suit :

    ![Ajouter un utilisateur](./media/talentlms-tutorial/IC777299.png "Ajouter un utilisateur")  

    a. Dans la zone de texte **First name** , entrez le prénom de l’utilisateur, par exemple **Britta** .

    b. Dans la zone de texte **Last name** , tapez le nom de l’utilisateur, par exemple **Simon** .
 
    c. Dans la zone de texte **Email address** (Adresse e-mail), entrez l’adresse e-mail de l’utilisateur, par exemple `brittasimon\@contoso.com`.

    d. Cliquez sur **Add User** .

> [!NOTE]
> Vous pouvez utiliser tout autre outil ou n’importe quelle API de création de compte d’utilisateur fournis par TalentLMS pour provisionner des comptes d’utilisateurs Azure AD.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette TalentLMS dans le volet d’accès, vous devez être connecté automatiquement à l’application TalentLMS pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)