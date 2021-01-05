---
title: 'Tutoriel : Intégration d’Azure Active Directory à Replicon | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Replicon.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/10/2019
ms.author: jeedes
ms.openlocfilehash: dada52dcf66db2f0d0e40f3dbf6d1a16560c2754
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92515025"
---
# <a name="tutorial-integrate-replicon-with-azure-active-directory"></a>Tutoriel : Intégrer Replicon à Azure Active Directory

Dans ce didacticiel, vous allez apprendre à intégrer Replicon à Azure Active Directory (Azure AD). Quand vous intégrez Replicon à Azure AD, vous pouvez :

* Dans Azure AD, contrôlez qui a accès à Replicon.
* Permettre à vos utilisateurs de se connecter automatiquement à Replicon avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’en avez pas, vous pouvez obtenir un essai gratuit d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Replicon pour lequel l’authentification unique est activée.

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. Replicon prend en charge l’authentification unique initiée par le **SP** .

## <a name="adding-replicon-from-the-gallery"></a>Ajout de Replicon à partir de la galerie

Pour configurer l’intégration de Replicon à Azure AD, vous devez ajouter Replicon à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Replicon** dans la zone de recherche.
1. Sélectionnez **Replicon** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec Replicon pour un utilisateur de test nommé **B. Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Replicon associé.

Pour configurer et tester l’authentification unique Azure AD avec Replicon, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Replicon](#configure-replicon-sso)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Replicon](#create-replicon-test-user)** pour avoir un équivalent de B. Simon dans Replicon lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Replicon** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la boîte de dialogue **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    1. Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://global.replicon.com/!/saml2/<client name>/sp-sso/post`.

    1. Dans la zone de texte **Identificateur** , tapez une URL en utilisant le format suivant : `https://global.replicon.com/!/saml2/<client name>`

    1. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://global.replicon.com/!/saml2/<client name>/sso/post`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez [l’équipe de support technique Replicon](https://www.replicon.com/customerzone/contact-support). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Cliquez sur l’icône de modification/de crayon pour **Certificat de signature SAML** pour modifier les paramètres.

    ![Algorithme de signature](common/signing-algorithm.png)

    1. Dans **Option de signature** , sélectionnez **Signer l’assertion SAML** .

    1. Sélectionnez **SHA-256** comme **Algorithme de signature** .

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

   ![Lien Téléchargement de certificat](common/metadataxml.png)

### <a name="configure-replicon-sso"></a>Configurer l’authentification unique de Replicon

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise Replicon en tant qu’administrateur.

2. Procédez comme suit pour configurer SAML 2.0 :

    ![Activer l’authentification SAML](./media/replicon-tutorial/ic777805.png "Activer l’authentification SAML")

    a. Pour afficher la boîte de dialogue **EnableSAML Authentication2** , ajoutez ce qui suit à votre URL, après la clé de votre entreprise : `/services/SecurityService1.svc/help/test/EnableSAMLAuthentication2`

    * Voici une illustration du schéma de l’URL complète : `https://na2.replicon.com/\<YourCompanyKey\>/services/SecurityService1.svc/help/test/EnableSAMLAuthentication2`

   b. Cliquez sur **+** pour développer la section **v20Configuration** .

   c. Cliquez sur **+** pour développer la section **metaDataConfiguration** .

   d. Sélectionnez **SHA256** pour xmlSignatureAlgorithm

   e. Cliquez sur **Choose File** pour sélectionner votre fichier XML de métadonnées de fournisseur d’identité, puis cliquez sur **Submit** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `BrittaSimon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Replicon.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Replicon** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-replicon-test-user"></a>Créer un utilisateur de test Replicon

L’objectif de cette section est de créer un utilisateur appelé B. Simon dans Replicon.

**Si vous avez besoin de créer un utilisateur manuellement, procédez comme suit :**

1. Dans une fenêtre de navigateur web, connectez-vous à votre site d’entreprise Replicon en tant qu’administrateur.

2. Accédez à **Administration \> Users** .

    ![Utilisateurs](./media/replicon-tutorial/ic777806.png "Utilisateurs")

3. Cliquez sur **+Add User** .

    ![Ajouter un utilisateur](./media/replicon-tutorial/ic777807.png "Ajouter un utilisateur")

4. Dans la section **User Profile** , procédez comme suit :

    ![Profil utilisateur](./media/replicon-tutorial/ic777808.png "User Profile")

    a. Dans la zone de texte **ID de connexion** , tapez l’adresse de messagerie Azure AD de l’utilisateur Azure AD que vous souhaitez approvisionner, comme `B.Simon@contoso.com`.

    > [!NOTE]
    > Le nom de connexion doit correspondre à l’adresse e-mail utilisateur dans Azure AD

    b. Pour **Authentication Type** , sélectionnez **SSO** .

    c. Définissez l’ID de jeu d’authentification sur la même valeur que le nom de connexion (l’adresse de messagerie Azure AD de l’utilisateur)

    d. Dans la zone de texte **Department** , tapez le département de l’utilisateur.

    e. Pour **Employee Type** , sélectionnez **Administrator** .

    f. Cliquez sur **Save User Profile** .

> [!NOTE]
> Vous pouvez utiliser n’importe quel outil ou API de création de compte utilisateur, fourni par Replicon, pour provisionner des comptes utilisateur Azure AD.

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Lorsque vous sélectionnez la vignette Replicon dans le volet d’accès, vous devez être connecté automatiquement à l’application Replicon pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)