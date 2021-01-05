---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à uniFLOW Online | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et uniFLOW Online.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/02/2019
ms.author: jeedes
ms.openlocfilehash: fc1317ffce7c958f092ad01feff473e123581439
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92521762"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-uniflow-online"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à uniFLOW Online

Dans ce tutoriel, vous allez apprendre à intégrer uniFLOW Online à Azure Active Directory (Azure AD). Quand vous intégrez uniFLOW Online à Azure AD, vous pouvez :

* Contrôler qui dans Azure AD a accès à uniFLOW Online.
* Permettre à vos utilisateurs de se connecter à uniFLOW Online avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un locataire uniFLOW Online.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* uniFLOW Online prend en charge l’authentification unique initiée par le **fournisseur de services**

## <a name="adding-uniflow-online-from-the-gallery"></a>Ajout d’uniFLOW Online à partir de la galerie

Pour configurer l’intégration d’uniFLOW Online à Azure AD, vous devez ajouter uniFLOW Online à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **uniFLOW Online** dans la zone de recherche.
1. Sélectionnez **uniFLOW Online** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-uniflow-online"></a>Configurer et tester l’authentification unique Azure AD pour uniFLOW Online

Configurez et testez l’authentification unique Azure AD auprès d’uniFLOW Online pour un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur uniFLOW Online associé.

Pour configurer et tester l’authentification unique Azure AD avec uniFLOW Online, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
   1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
   1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique uniFLOW Online](#configure-uniflow-online-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Se connecter à uniFLOW Online à l’aide de l’utilisateur de test créé](#sign-in-to-uniflow-online-using-the-created-test-user)** pour tester la connexion de l’utilisateur côté application.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Sur le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **uniFLOW Online** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , entrez une URL au format suivant :

    - `https://<tenant_domain_name>.eu.uniflowonline.com`
    - `https://<tenant_domain_name>.us.uniflowonline.com`
    - `https://<tenant_domain_name>.sg.uniflowonline.com`
    - `https://<tenant_domain_name>.jp.uniflowonline.com`
    - `https://<tenant_domain_name>.au.uniflowonline.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant :

    - `https://<tenant_domain_name>.eu.uniflowonline.com`
    - `https://<tenant_domain_name>.us.uniflowonline.com`
    - `https://<tenant_domain_name>.sg.uniflowonline.com`
    - `https://<tenant_domain_name>.jp.uniflowonline.com`
    - `https://<tenant_domain_name>.au.uniflowonline.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez [l’équipe de support uniFLOW Online](mailto:support@nt-ware.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure ou examiner l’URL de réponse affichée dans votre locataire uniFLOW Online.

1. L’application uniFLOW Online attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application uniFLOW Online s’attend à ce que quelques attributs supplémentaires (présentés ci-dessous) soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom |  Attribut source|
    | -----------| --------------- |
    | displayname | user.displayname |
    | nickname | user.onpremisessamaccountname |

   > [!NOTE]
   > L’attribut `user.onpremisessamaccountname` contient une valeur uniquement si vos utilisateurs Azure AD sont synchronisés à partir d’un annuaire Windows Active Directory local.

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

Dans cette section, vous allez permettre à B.Simon d’utiliser l’authentification unique Azure en lui accordant l’accès à uniFLOW Online.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **uniFLOW Online**.
1. Dans la page de vue d’ensemble de l’application, accédez à la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

   ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

> [!NOTE]
> Pour autoriser tous les utilisateurs à accéder à l’application sans attribution manuelle, accédez à la section **Gérer** et sélectionnez **Propriétés**. Ensuite, affectez **NON** au paramètre **Affectation de l’utilisateur obligatoire**.

## <a name="configure-uniflow-online-sso"></a>Configurer l’authentification unique uniFLOW Online

1. Dans une autre fenêtre de navigateur web, connectez-vous au site web uniFLOW Online en tant qu’administrateur.

1. Dans le volet de navigation de gauche, sélectionnez l’onglet **User** (Utilisateur).

    ![Capture d’écran montrant Users sélectionné sur le site uniflow Online.](./media/uniflow-online-tutorial/configure1.png)

1. Cliquez sur **Fournisseurs d’identité**.

    ![Capture d’écran montrant l’élément Identity Providers sélectionné.](./media/uniflow-online-tutorial/configure2.png)

1. Cliquez sur **Add identity provider** (Ajouter un fournisseur d’identité).

    ![Capture d’écran montrant l’option Add identity provider sélectionnée.](./media/uniflow-online-tutorial/configure3.png)

1. Dans la section **ADD IDENTITY PROVIDER** , effectuez les étapes suivantes :

    ![Capture d’écran montrant la section ADD IDENTITY PROVIDER où vous pouvez entrer les valeurs décrites.](./media/uniflow-online-tutorial/configure4.png)

    a. Entrez le nom d’affichage dans le champ « Display name », par exemple : *AzureAD SSO*.

    b. Pour **Provider type** (Type de fournisseur), sélectionnez l’option **WS-Fed** dans la liste déroulante.

    c. Pour **WS-Fed type** , sélectionnez l’option **Azure Active Directory** dans la liste déroulante.

    d. Cliquez sur **Enregistrer**.

1. Sous l’onglet **General** , effectuez les étapes suivantes :

    ![Capture d’écran montrant l’onglet General où vous pouvez entrer les valeurs décrites.](./media/uniflow-online-tutorial/configure5.png)

    a. Entrez le nom d’affichage dans le champ « Display name », par exemple : *AzureAD SSO*.

    b. Sélectionnez l’option **From URL** (À partir d’une URL) pour **ADFS Federation Metadata** (Métadonnées de fédération ADFS).

    c. Dans la zone de texte **Federation Metadata URL** (URL des métadonnées de fédération), collez la valeur du champ **URL des métadonnées de fédération d’application** que vous avez copiée à partir du portail Azure.

    d. Pour **Identity provider** (Fournisseur d’identité), cochez la case **Enabled** (Activé).

    e. Pour **Automatic user registration** (Inscription automatique des utilisateurs), cochez la case **Activated** (Activé).

    f. Cliquez sur **Enregistrer**.

### <a name="sign-in-to-uniflow-online-using-the-created-test-user"></a>Se connecter à uniFLOW Online à l’aide de l’utilisateur de test créé

1. Dans une autre fenêtre de navigateur web, accédez à l’URL uniFLOW Online pour votre locataire.

1. Sélectionnez le fournisseur d’identité créé précédemment pour vous connecter par le biais de votre instance Azure AD.

1. Connectez-vous à l’aide de l’utilisateur de test.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer uniFLOW Online avec Azure AD](https://aad.portal.azure.com/)