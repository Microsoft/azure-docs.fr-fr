---
title: 'Tutoriel : Intégration d’Azure Active Directory à Peakon | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Peakon.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/19/2019
ms.author: jeedes
ms.openlocfilehash: a84d4d71e5190c455441d1e627381be86ef5e129
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97608525"
---
# <a name="tutorial-azure-active-directory-integration-with-peakon"></a>Tutoriel : Intégration d’Azure Active Directory à Peakon

Dans ce tutoriel, vous allez apprendre à intégrer Peakon à Azure Active Directory (Azure AD).
L’intégration de Peakon à Azure AD offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Peakon.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Peakon (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Peakon, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Peakon pour lequel l’authentification unique est activée

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Peakon prend en charge l’authentification unique lancée par le **fournisseur de services** et le **fournisseur d’identité**

## <a name="adding-peakon-from-the-gallery"></a>Ajout de Peakon à partir de la galerie

Pour configurer l’intégration de Peakon à Azure AD, vous devez ajouter Peakon, disponible dans la galerie, à votre liste d’applications SaaS managées.

**Pour ajouter Peakon à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Peakon**, sélectionnez **Peakon** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Peakon dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Peakon à l’aide d’un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Peakon associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Peakon, vous devez effectuer les modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Peakon](#configure-peakon-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Peakon](#create-peakon-test-user)** pour avoir un équivalent de Britta Simon dans Peakon lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Peakon, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Peakon**, sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité**, effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL de Peakon](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `https://app.peakon.com/saml/<companyid>/metadata`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://app.peakon.com/saml/<companyid>/assert`

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    ![Capture d’écran montrant l’option « Définir des URL supplémentaires » sélectionnée, avec la zone de texte « URL de connexion » mise en évidence.](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion**, tapez une URL : `https://app.peakon.com/login`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de réponse réels, procédure expliquée plus loin dans le tutoriel. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

6. Dans la page **Configurer l’authentification unique avec SAML**, à la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Raw)** en fonction des options définies, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificateraw.png)

7. Dans la section **Configurer Peakon**, copiez l’URL ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-peakon-single-sign-on"></a>Configurer l’authentification unique Peakon

1. Dans une autre fenêtre de navigateur web, connectez-vous à Peakon en tant qu’administrateur.

2. Dans la barre de menus située à gauche de la page, cliquez sur **Configuration**, puis accédez à **Integrations**.

    ![Configuration](./media/peakon-tutorial/tutorial_peakon_config.png)

3. Dans la page **Integrations** (Intégrations), cliquez sur **Single Sign-On** (Authentification unique).

    ![Authentification unique](./media/peakon-tutorial/tutorial_peakon_single.png)

4. Sous la section **Single Sign-On** (Authentification unique), cliquez sur **Enable** (Activer).

    ![Activation](./media/peakon-tutorial/tutorial_peakon_enable.png)

5. Dans la section **Single sign-on for employees using SAML** (Authentification unique pour les employés avec SAML), effectuez les étapes suivantes :

    ![SAML](./media/peakon-tutorial/tutorial_peakon_saml.png)

    a. Dans la zone de texte **SSO Login URL** (URL de connexion avec authentification unique), collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.

    b. Dans la zone de texte **SSO Logout URL** (URL de déconnexion avec authentification unique), collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.

    c. Cliquez sur **Choose a File** (Choisir un fichier) pour charger le certificat que vous avez téléchargé à partir du portail Azure dans la zone Certificate (Certificat).

    d. Cliquez sur l’**icône** pour copier l’**ID d’entité** et collez-le dans la zone de texte **Identificateur** de la section **Configuration SAML de base** du portail Azure.

    e. Cliquez sur l’**icône** pour copier l’**URL de réponse (ACS)** et collez-la dans la zone de texte **URL de réponse** de la section **Configuration SAML de base** du portail Azure.

    f. Cliquez sur **Enregistrer**.

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Peakon.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **Peakon**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Peakon**.

    ![Lien Peakon dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-peakon-test-user"></a>Créer un utilisateur de test Peakon

Pour se connecter à Peakon, les utilisateurs Azure AD doivent être provisionnés dans Peakon.  
Dans le cas de Peakon, le provisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise Peakon en tant qu’administrateur.

2. Dans la barre de menus située à gauche de la page, cliquez sur **Configuration**, puis accédez à **Employees** (Employés).

    ![Employé](./media/peakon-tutorial/tutorial_peakon_employee.png)

3. En haut à droite de la page, cliquez sur **Add employee** (Ajouter un employé).

    ![Ajouter un employé](./media/peakon-tutorial/tutorial_peakon_addemployee.png)

3. Dans la boîte de dialogue **New employee** (Nouvel employé), effectuez les étapes suivantes :

    ![Nouvel employé](./media/peakon-tutorial/tutorial_peakon_create.png)

    1. Dans la zone de texte **Name** (Nom), entrez **Britta** comme prénom et **simon** comme nom.

    1. Dans la zone de texte **Email** (Adresse e-mail), tapez l’adresse e-mail, par exemple **Brittasimon\@contoso.com**.

    1. Cliquez sur **Créer un employé**.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Peakon dans le volet d’accès, vous devez être connecté automatiquement à l’application Peakon pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)