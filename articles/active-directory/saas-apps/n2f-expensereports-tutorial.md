---
title: 'Didacticiel : Intégration d’Azure Active Directory à N2F - Note de frais | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et N2F - Note de frais.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/01/2019
ms.author: jeedes
ms.openlocfilehash: b260e51321e14a6ea1d1ee75f88ca7564b83d492
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92516747"
---
# <a name="tutorial-azure-active-directory-integration-with-n2f---expense-reports"></a>Didacticiel : Intégration d’Azure Active Directory à N2F - Note de frais

Dans le cadre de ce didacticiel, vous allez apprendre à intégrer N2F - Note de frais à Azure Active Directory (Azure AD).
L’intégration de N2F - Note de frais à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à N2F - Note de frais.
* Vous pouvez autoriser vos utilisateurs à se connecter automatiquement à N2F - Note de frais (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à N2F - Note de frais, vous devez disposer des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement N2F - Note de frais pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* N2F - Note de frais prend en charge l’authentification unique lancée par le **fournisseur de services** et le **fournisseur d’identité**

## <a name="adding-n2f---expense-reports-from-the-gallery"></a>Ajout de N2F - Note de frais à partir de la galerie

Pour configurer l’intégration de N2F - Note de frais à Azure AD, vous devez ajouter N2F - Note de frais à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter N2F - Note de frais à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **N2F - Note de frais** , sélectionnez **N2F - Note de frais** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![N2F - Note de frais dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec N2F - Note de frais pour un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur N2F - Note de frais associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec N2F - Note de frais, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique N2F - Note de frais](#configure-n2f---expense-reports-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test N2F - Note de frais](#create-n2f---expense-reports-test-user)** pour disposer dans N2F - Note de frais d’un équivalent de Britta Simon qui soit lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec N2F - Note de frais, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **N2F - Note de frais** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , l’utilisateur n’a rien à faire, car l’application est pré-intégrée à Azure.

    ![Capture d’écran montrant la page Authentification basée sur SAML, avec la configuration SAML de base.](common/preintegrated.png)

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    ![Capture d’écran montrant la page Intégrations dans laquelle vous pouvez ajouter l’authentification unique Azure AD.](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion** , tapez une URL : `https://www.n2f.com/app/`

6. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

7. Dans la section **Configurer N2F - Note de frais** , copiez les URL appropriées, selon vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-n2f---expense-reports-single-sign-on"></a>Configurer l’authentification unique N2F - Note de frais

1. Dans une autre fenêtre de navigateur web, connectez-vous au site d’entreprise N2F - Note de frais en tant qu’administrateur.

2. Cliquez sur **Paramètres** , puis sélectionnez **Paramètres avancés** dans la liste déroulante.

    ![Capture d’écran montrant l’élément Advanced Settings sélectionné.](./media/n2f-expensereports-tutorial/configure1.png)

3. Sélectionnez l’onglet **Paramètres du compte** .

    ![Capture d’écran montrant l’élément Account settings sélectionné.](./media/n2f-expensereports-tutorial/configure2.png)

4. Sélectionnez **Authentification** , puis sélectionnez l’onglet **+ Add an authentication method** (Ajouter une méthode d’authentification).

    ![Capture d’écran montrant la section Account Setting Authentication dans laquelle vous pouvez ajouter une méthode d’authentification.](./media/n2f-expensereports-tutorial/configure3.png)

5. Sélectionnez **SAML Microsoft Office 365** comme méthode d’authentification.

    ![Capture d’écran montrant la zone Authentication method avec l’option SAML Microsoft Office 365 sélectionnée.](./media/n2f-expensereports-tutorial/configure4.png)

6. Dans la section **Méthode d’authentification** , procédez comme suit :

    ![Capture d’écran montrant la section Authentication method dans laquelle vous pouvez entrer les valeurs décrites.](./media/n2f-expensereports-tutorial/configure5.png)

    a. Dans la zone de texte **ID d’entité** , collez l’ **Identificateur Azure AD** que vous avez copié sur le portail Azure.

    b. Dans la zone de texte **URL de métadonnées** , collez la valeur **URL des métadonnées de fédération d’application** que vous avez copiée à partir du Portail Azure.

    c. Cliquez sur **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon\@domainedevotreentreprise.extension** .  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à N2F - Note de frais.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **N2F - Note de frais** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **N2F - Note de frais** .

    ![Lien N2F - Note de frais dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-n2f---expense-reports-test-user"></a>Créer un utilisateur de test N2F - Note de frais

Pour permettre aux utilisateurs Azure AD de se connecter à N2F - Note de frais, vous devez les approvisionner dans N2F - Note de frais. Dans le cas de N2F - Note de frais, l’approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous au site d’entreprise N2F - Note de frais en tant qu’administrateur.

2. Cliquez sur **Paramètres** , puis sélectionnez **Paramètres avancés** dans la liste déroulante.

    ![Capture d’écran montrant l’élément Advanced Settings sélectionné.](./media/n2f-expensereports-tutorial/configure1.png)

3. Sélectionnez l’onglet **Utilisateurs** dans le volet de navigation de gauche.

    ![Capture d’écran montrant l’élément Users sélectionné.](./media/n2f-expensereports-tutorial/user1.png)

4. Sélectionnez l’onglet **+ Nouvel utilisateur** .

    ![Capture d’écran montrant l’option New user.](./media/n2f-expensereports-tutorial/user2.png)

5. Dans la section **Utilisateur** , procédez comme suit :

    ![Capture d’écran affichant la section dans laquelle vous pouvez indiquer les valeurs décrites.](./media/n2f-expensereports-tutorial/user3.png)

    a. Dans la zone de texte **Email address** (Adresse e-mail), entrez l’adresse e-mail de l’utilisateur, par exemple **brittasimon\@contoso.com** .

    b. Dans la zone de texte **First name** , entrez le prénom de l’utilisateur, par exemple **Britta** .

    c. Dans la zone de texte **Nom** , entrez le nom d’un utilisateur, par exemple **Britta Simon** .

    d. Choisissez **Role, Direct manager (N+1)** (Rôle, Supérieur hiérarchique [N+1]) et **Division** (Division) conformément aux exigences de votre organisation.

    e. Cliquez sur **Validate and send invitation** (Valider et envoyer une invitation).

    > [!NOTE]
    > Si vous rencontrez des problèmes lors de l’ajout de l’utilisateur, contactez [l’équipe du support technique de N2F - Note de frais](mailto:support@n2f.com).

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette N2F - Note de frais dans le volet d’accès, vous devez vous connecter automatiquement à l’application N2F - Note de frais pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)