---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Watch by Colors | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Watch by Colors.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/09/2019
ms.author: jeedes
ms.openlocfilehash: fae276049ec3ff6c4bf96be38ff0d2a952731a15
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92636780"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-watch-by-colors"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Watch by Colors

Dans ce tutoriel, vous allez découvrir comment intégrer Watch by Colors à Azure Active Directory (Azure AD). Quand vous intégrez Watch by Colors à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Watch by Colors.
* Permettre à vos utilisateurs de se connecter automatiquement à Watch by Colors avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Watch by Colors pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Watch by Colors prend en charge l’authentification unique (SSO) lancée par **le fournisseur de services et le fournisseur d’identité**

## <a name="adding-watch-by-colors-from-the-gallery"></a>Ajout de Watch by Colors à partir de la galerie

Pour configurer l’intégration de Watch by Colors à Azure AD, vous devez ajouter Watch by Colors de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **Watch by Colors** dans la zone de recherche.
1. Sélectionnez **Watch by Colors** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-watch-by-colors"></a>Configurer et tester l’authentification unique Azure AD pour Watch by Colors

Configurez et testez l’authentification unique (SSO) Azure AD avec Watch by Colors à l’aide d’un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique (SSO) fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur Watch by Colors associé.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec Watch by Colors, suivez les modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique (SSO) Watch by Colors](#configure-watch-by-colors-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Watch by Colors](#create-watch-by-colors-test-user)** pour disposer, dans Watch by Colors, d’un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans la page d’intégration de l’application **Watch by Colors** du [portail Azure](https://portal.azure.com/), recherchez la section **Gérer** , puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , l’application est préconfigurée en mode Lancement par le **fournisseur d’identité** et les URL nécessaires sont déjà préremplies avec Azure. L’utilisateur doit enregistrer la configuration en cliquant sur le bouton **Enregistrer**.

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL : `https://app.colorscorporation.com/login`

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Watch by Colors.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Watch by Colors**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

## <a name="configure-watch-by-colors-sso"></a>Configurer l’authentification unique (SSO) Watch by Colors

1. Afin d’automatiser la configuration dans Watch by Colors, vous devez installer l’ **extension de connexion sécurisée à Mes applications** en cliquant sur **Installer l’extension**.

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Watch by Colors** pour être redirigé vers l’application Watch by Colors. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Watch by Colors. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 5.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement Watch by Colors, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Watch by Colors en tant qu’administrateur et effectuez les étapes suivantes :

4. En haut à droite de la page, cliquez sur **Profil** > **Paramètres du compte** > **Authentification unique (SSO)** .

    ![Capture d’écran montrant la page Paramètres du compte dans laquelle l’authentification unique est désactivée.](./media/watch-by-colors-tutorial/config01.png)

5. Dans la page **Authentification unique (SSO)** , effectuez les étapes suivantes :

    ![Capture d’écran montrant l’onglet Configuration SAML où vous pouvez activer SAML.](./media/watch-by-colors-tutorial/config02.png)

    a. Définissez **Activer SAML** sur **ACTIVÉ**.

    b. Dans la zone de texte **URL** , collez l’ **URL des métadonnées de fédération** que vous avez copiée à partir du portail Azure.

    c. Cliquez sur **Importer**. Les champs suivants sont alors automatiquement remplis dans la page.

    d. Cliquez sur **Enregistrer**.

### <a name="create-watch-by-colors-test-user"></a>Créer un utilisateur de test Watch by Colors

Pour permettre aux utilisateurs Azure AD de se connecter à Watch by Colors, vous devez les attribuer dans Watch by Colors. Dans Watch by Colors, le provisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à Watch by Colors en tant qu’administrateur de la sécurité.

1. En haut à droite de la page, cliquez sur **Profil** > **Utilisateurs** > **Ajouter un utilisateur**.

    ![Capture d’écran montrant la page Utilisateurs.](./media/watch-by-colors-tutorial/config03.png)

1. Dans la page **Détails de l’utilisateur** , effectuez les étapes suivantes :

    ![Capture d’écran montrant la page Détails de l’utilisateur dans laquelle vous pouvez entrer les valeurs décrites.](./media/watch-by-colors-tutorial/config04.png)

    a. Dans la zone de texte **First Name** (Prénom), tapez le prénom de l’utilisateur, par exemple **B**.

    b. Dans la zone de texte **Nom** , entrez le nom de l’utilisateur, par exemple **Simon**.

    c. Dans la zone de texte **E-mail** , entrez l’adresse e-mail de l’utilisateur, comme `B.Simon@contoso.com`.

    d. Dans la zone de texte **Mot de passe** , entrez le mot de passe.

    e. Sélectionnez **Autorisations du compte** en fonction de votre organisation.

    f. Cliquez sur **Enregistrer**.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Watch by Colors dans le volet d’accès doit vous connecter automatiquement à l’application Watch by Colors pour laquelle vous avez configuré l’authentification unique (SSO). Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Watch by Colors avec Azure AD](https://aad.portal.azure.com/)