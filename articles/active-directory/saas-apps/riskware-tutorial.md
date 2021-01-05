---
title: 'Tutoriel : Intégration d’Azure Active Directory avec Riskware | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Riskware.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: jeedes
ms.openlocfilehash: e33664501483280978c67b100cd56a714ef63a30
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92520759"
---
# <a name="tutorial-azure-active-directory-integration-with-riskware"></a>Tutoriel : Intégration d’Azure Active Directory à Riskware

Dans ce tutoriel, vous allez apprendre à intégrer Riskware à Azure Active Directory (Azure AD).
L’intégration de Riskware dans Azure AD offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Riskware.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Riskware (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Riskware, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Abonnement Riskware pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Riskware prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-riskware-from-the-gallery"></a>Ajout de Riskware à partir de la galerie

Pour configurer l’intégration de Riskware à Azure AD, vous devez ajouter Riskware à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Riskware à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Riskware** , sélectionnez **Riskware** dans le panneau des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Riskware dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous configurez et testez l’authentification unique Azure AD avec Riskware pour un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans Riskware.

Pour configurer et tester l’authentification unique Azure AD avec Riskware, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Riskware](#configure-riskware-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Riskware](#create-riskware-test-user)** pour avoir dans Riskware un équivalent de Britta Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Riskware, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Riskware** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Riskware](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion** , entrez une URL au format suivant :
    
    | Environnement| Modèle d’URL|
    |--|--|
    | UAT|  `https://riskcloud.net/uat?ccode=<COMPANYCODE>` |
    | PROD| `https://riskcloud.net/prod?ccode=<COMPANYCODE>` |
    | DEMO| `https://riskcloud.net/demo?ccode=<COMPANYCODE>` |
    |||

    b. Dans la zone de texte **Identificateur (ID d’entité)** , tapez l’URL :
    
    | Environnement| Modèle d’URL|
    |--|--|
    | UAT| `https://riskcloud.net/uat` |
    | PROD| `https://riskcloud.net/prod` |
    | DEMO| `https://riskcloud.net/demo` |
    |||

    > [!NOTE]
    > La valeur d’URL de connexion n’est pas réelle. Mettez à jour la valeur avec l’URL de connexion réelle. Pour obtenir cette valeur, contactez l’[équipe du support technique de Riskware](mailto:support@pansoftware.com.au). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configurer Riskware** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-riskware-single-sign-on"></a>Configurer l’authentification unique pour Riskware

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise Riskware en tant qu’administrateur.

1. Dans le coin supérieur droit, cliquez sur **Maintenance** pour ouvrir la page de maintenance.

    ![Configuration de Riskware - Maintenance](./media/riskware-tutorial/tutorial_riskware_maintain.png)

1. Dans la page de maintenance, cliquez sur **Authentication** .

    ![Configuration de Riskware - Authentification](./media/riskware-tutorial/tutorial_riskware_authen.png)

1. Dans la page **Authentication Configuration** , effectuez les étapes suivantes :

    ![Configuration de Riskware - Configuration de l’authentification](./media/riskware-tutorial/tutorial_riskware_config.png)

    a. Sélectionnez **SAML** comme **Type** d’authentification.

    b. Dans la zone de texte **Code** , tapez votre code, par exemple AZURE_UAT.

    c. Dans la zone de texte **Description** , tapez votre description, par exemple Configuration d’Azure pour l’authentification unique.

    d. Dans la zone de texte **Single Sign On page** (Page d’authentification unique), collez l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    e. Dans la zone de texte **Sign out Page** (Page de déconnexion), collez l’ **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    f. Dans la zone de texte **Post Form Field** (Champ de formulaire de notification), tapez le nom du champ présent dans Post Response qui contient SAML, comme SAMLResponse.

    g. Dans la zone de texte **XML Identity Tag Name** , tapez l’attribut qui contient l’identificateur unique dans la réponse SAML, comme NameID.

    h. Dans le Bloc-notes, ouvrez le **XML de métadonnées** téléchargé du portail Azure, copiez le certificat à partir du fichier de métadonnées, puis collez-le dans la zone de texte **Certificate**

    i. Dans la zone de texte **Consumer URL** , collez la valeur de **Reply URL** , que vous obtenez à partir de l’équipe de support.

    j. Dans la zone de texte **Issuer** , collez la valeur de **Identifier** , que vous obtenez à partir de l’équipe de support.

    > [!Note]
    > Pour obtenir ces valeurs, contactez l’[équipe de support technique de Riskware](mailto:support@pansoftware.com.au)

    k. Cochez la case **Use POST** (Utiliser POST).

    l. Cochez la case **Use SAML Request** (Utiliser une requête SAML).

    m. Cliquez sur **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez `brittasimon@yourcompanydomain.extension`  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Riskware.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Riskware** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Riskware** .

    ![Lien Riskware dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-riskware-test-user"></a>Créer un utilisateur de test Riskware

Pour se connecter à Riskware, les utilisateurs Azure AD doivent être provisionnés dans Riskware. Dans Riskware, le provisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à Riskware en tant qu’administrateur de la sécurité.

1. Dans le coin supérieur droit, cliquez sur **Maintenance** pour ouvrir la page de maintenance. 

    ![Configuration de Riskware - Maintenance](./media/riskware-tutorial/tutorial_riskware_maintain.png)

1. Dans la page de maintenance, cliquez sur **People** .

    ![Configuration de Riskware - Personnes](./media/riskware-tutorial/tutorial_riskware_people.png)

1. Sélectionnez l’onglet **Details** (Détails), puis effectuez les étapes suivantes :

    ![Configuration de Riskware - Détails](./media/riskware-tutorial/tutorial_riskware_details.png)

    a. Sélectionnez Employee comme **Person Type** .

    b. Dans la zone de texte **First name** , entrez le prénom de l’utilisateur, par exemple **Britta** .

    c. Dans la zone de texte **Surname** , tapez le nom de l’utilisateur, par exemple **Simon** .

1. Sous l’onglet **Security** , procédez comme suit :

    ![Configuration de Riskware - Sécurité](./media/riskware-tutorial/tutorial_riskware_security.png)

    a. Dans la section **Authentication** , sélectionnez le mode **Authentication** , que vous avez configuré comme Configuration d’Azure pour l’authentification unique.

    b. Sous la section **Logon Details** (Informations de connexion), dans la zone de texte **User ID** (ID utilisateur), entrez l’adresse e-mail de l’utilisateur, par exemple `brittasimon@contoso.com`.

    c. Dans la zone de texte **Password** , tapez le mot de passe de l’utilisateur.

1. Sous l’onglet **Organization** (Organisation), effectuez les étapes suivantes :

    ![Configuration de Riskware - Organisation](./media/riskware-tutorial/tutorial_riskware_org.png)

    a. Sélectionnez l’option **Level 1** (Niveau 1) pour l’organisation.

    b. Dans la section **Person’s Primary Workplace** , dans la zone de texte **Location** , tapez votre emplacement.

    c. Dans la section **Employee** , sélectionnez Casual comme **Employee Status** .

    d. Cliquez sur **Enregistrer** .

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Riskware dans le volet d’accès, vous devez être connecté automatiquement à l’application Riskware pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)