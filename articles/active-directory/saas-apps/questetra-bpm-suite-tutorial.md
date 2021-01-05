---
title: "Didacticiel : Intégration d'Azure Active Directory à Questetra BPM Suite | Microsoft Docs"
description: Découvrez comment configurer l'authentification unique entre Azure Active Directory et Questetra BPM Suite.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: 13f05005529d3983e042398f5274fd7f981b8c8c
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92516588"
---
# <a name="tutorial-azure-active-directory-integration-with-questetra-bpm-suite"></a>Didacticiel : Intégration d'Azure Active Directory avec Questetra BPM Suite

Dans ce didacticiel, vous allez apprendre à intégrer Questetra BPM Suite dans Azure Active Directory (Azure AD).
L’intégration de Questetra BPM Suite à Azure AD offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Questetra BPM Suite.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Questetra BPM Suite (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l'intégration d'Azure AD avec Questetra BPM Suite, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Questetra BPM Suite pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Questetra BPM Suite prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-questetra-bpm-suite-from-the-gallery"></a>Ajout de Questetra BPM Suite à partir de la galerie

Pour configurer l'intégration de Questetra BPM Suite avec Azure AD, vous devez ajouter Questetra BPM Suite, disponible dans la galerie, à votre liste d'applications SaaS gérées.

**Pour ajouter Questetra BPM Suite à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Questetra BPM Suite** , sélectionnez **Questetra BPM Suite** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Questetra BPM Suite dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD auprès de Questetra BPM Suite, avec un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur Questetra BPM Suite associé doit être établie.

Pour configurer et tester l'authentification unique Azure AD avec Questetra BPM Suite, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Questetra BPM Suite](#configure-questetra-bpm-suite-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Questetra BPM Suite](#create-questetra-bpm-suite-test-user)** pour avoir un équivalent de Britta Simon dans Questetra BPM Suite qui soit lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Questetra BPM Suite, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Questetra BPM Suite** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Questetra BPM Suite](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<subdomain>.questetra.net/saml/SSO/alias/bpm`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<subdomain>.questetra.net/`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Vous pouvez obtenir ces valeurs dans la section **Informations SP** de votre site d’entreprise **Questetra BPM Suite** , décrit plus loin dans le didacticiel ou contacter l’ [équipe de support Questetra BPM Suite](https://www.questetra.com/contact/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer Questetra BPM Suite** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-questetra-bpm-suite-single-sign-on"></a>Configurer l’authentification unique Questetra BPM Suite

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise **Questetra BPM Suite** en tant qu’administrateur.

2. Dans le menu situé en haut, cliquez sur **Paramètres système** . 
   
    ![Capture d’écran montrant l’option des paramètres système sélectionnée à partir de votre site d’entreprise Questetra BPM Suite.][10]

3. Pour ouvrir la page **SingleSignOnSAML** , cliquez sur **SSO (SAML)** . 
   
    ![Capture d’écran montrant l’élément SSO (SAML) sélectionné.][11]

4. Sur votre site d’entreprise **Questetra BPM Suite** , dans la section **Informations SP** , effectuez les étapes suivantes :

    a. Copiez l’ **URL ACS** , puis collez-la dans la zone de texte **URL de connexion** dans la section **Configuration SAML de base** du portail Azure.
    
    b. Copiez l’ **ID d’entité** , puis collez-la dans la zone de texte **Identificateur** dans la section **Configuration SAML de base** du portail Azure.

5. Sur votre site d’entreprise **Questetra BPM Suite** , effectuez les étapes suivantes : 
   
    ![Configure Single Sign-On][15]
   
    a. Sélectionnez **Activer l'authentification unique** .
   
    b. Dans la zone de texte **Entity Id** , collez la valeur **Identificateur Azure AD** que vous avez copiée dans le portail Azure.
    
    c. Dans la zone de texte **Sign-in page URL** , collez la valeur **URL de connexion** que vous avez copiée à partir du portail Azure.
    
    d. Dans la zone de texte **Sign-out page URL** , collez la valeur **URL de déconnexion** que vous avez copiée à partir du portail Azure.
    
    e. Dans la zone de texte **Format NameID** , type `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`.

    f. Ouvrez, dans le Bloc-notes, votre certificat codé en **base 64** , téléchargé à partir du portail Azure, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **Certificat de validation** . 

    g. Cliquez sur **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez brittasimon@yourcompanydomain.extension. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Questetra BPM Suite.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Questetra BPM Suite** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Questetra BPM Suite** .

    ![Lien Questetra BPM Suite dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-questetra-bpm-suite-test-user"></a>Créer un utilisateur de test Questetra BPM Suite

L'objectif de cette section est de créer un utilisateur appelé Britta Simon dans Questetra BPM Suite.

**Pour créer un utilisateur appelé Britta Simon dans Questetra BPM Suite, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise Questetra BPM Suite en tant qu’administrateur.

2. Accédez à **Paramètres système > Liste des utilisateurs > Nouvel utilisateur** .
 
3. Dans la boîte de dialogue Nouvel utilisateur, procédez comme suit : 
   
    ![Créer un utilisateur de test][300] 
   
    a. Dans la zone de texte **Nom** , tapez le **nom** de l’utilisateur britta.simon@contoso.com.
   
    b. Dans la zone de texte **E-mail** , entrez l’ **e-mail** de l’utilisateur britta.simon@contoso.com.
   
    c. Dans la zone de texte **Mot de passe** , tapez un **mot de passe** pour l’utilisateur.
    
    d. Cliquez sur **Ajouter un nouvel utilisateur** .

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Questetra BPM Suite dans le volet d’accès, vous devez être connecté automatiquement à l’application Questetra BPM Suite pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

<!--Image references-->

[10]: ./media/questetra-bpm-suite-tutorial/questera_bpm_suite_03.png
[11]: ./media/questetra-bpm-suite-tutorial/questera_bpm_suite_04.png
[15]: ./media/questetra-bpm-suite-tutorial/questera_bpm_suite_08.png
[300]: ./media/questetra-bpm-suite-tutorial/questera_bpm_suite_11.png