---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec GitHub Enterprise Cloud – Enterprise Account | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et GitHub Enterprise Cloud – Enterprise Account.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/29/2020
ms.author: jeedes
ms.openlocfilehash: d88cbb79b42637721412dd0a35c231782a896721
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96029844"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-github-enterprise-cloud---enterprise-account"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec GitHub Enterprise Cloud – Enterprise Account

Ce didacticiel explique comment intégrer le compte GitHub Enterprise Cloud-Enterprise avec Azure Active Directory (Azure AD). L’intégration de GitHub Enterprise Cloud – Enterprise Account avec Azure AD vous permet d’effectuer les opérations suivantes :

* Contrôler qui dans Azure AD a accès à un compte GitHub Enterprise et à toute organisation au sein du compte Enterprise.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un [compte GitHub Enterprise](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-your-enterprise/about-enterprise-accounts)
* Un compte d’utilisateur GitHub qui est propriétaire d’un compte Enterprise. 

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* GitHub Enterprise Cloud – Enterprise Account prend en charge l’authentification unique initiée par un **fournisseur de services** et un **fournisseur d’ID**.
* GitHub Enterprise Cloud – Enterprise Account prend en charge l’approvisionnement **juste-à-temps** d’utilisateurs.
* Une fois que vous avez configuré GitHub Enterprise Cloud – Enterprise Account, vous pouvez appliquer un contrôle de session qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-github-enterprise-cloud---enterprise-account-from-the-gallery"></a>Ajouter GitHub Enterprise Cloud – Enterprise Account à partir de la galerie

Pour configurer l’intégration de GitHub Enterprise Cloud – Enterprise Account dans Azure AD, vous devez ajouter GitHub Enterprise Cloud – Enterprise Account à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **GitHub Enterprise Cloud – Enterprise Account** dans la zone de recherche.
1. Sélectionnez **GitHub Enterprise Cloud – Enterprise Account** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-github-enterprise-cloud---enterprise-account"></a>Configurer et tester l’authentification unique Azure AD pour GitHub Enterprise Cloud – Enterprise Account

Configurez et testez l’authentification unique Azure AD avec GitHub Enterprise Cloud – Enterprise Account avec un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur associé dans GitHub Enterprise Cloud – Enterprise Account.

Pour configurer et tester l’authentification unique Azure AD avec GitHub Enterprise Cloud – Enterprise Account, vous devez accomplir les opérations suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter votre utilisateur Azure AD et le compte d’utilisateur de test à l’application GitHub](#assign-the-azure-ad-test-user)** pour permettre à votre compte d’utilisateur et à l’utilisateur de test `B.Simon` d’utiliser l’authentification unique Azure AD.
1. **[Activer et tester SAML pour le compte Enterprise et ses organisations](#enable-and-test-saml-for-the-enterprise-account-and-its-organizations)** afin de configurer les paramètres d’authentification unique côté application.
    1. **[Tester l’authentification unique avec un autre compte de membre de l’organisation ou propriétaire de compte Enterprise](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **GitHub Enterprise Cloud – Enterprise Account**, recherchez la section **Gérer**, puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://github.com/enterprises/<ENTERPRISE-SLUG>`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://github.com/enterprises/<ENTERPRISE-SLUG>/saml/consume`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

     Dans la zone de texte **URL de connexion**, entrez une URL au format suivant :  `https://github.com/enterprises/<ENTERPRISE-SLUG>/sso`

    > [!NOTE]
    > Remplacez `<ENTERPRISE-SLUG>` par le nom réel de votre compte GitHub Enterprise.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificateBase64.png)

1. Dans la section **Configurer GitHub Enterprise Cloud – Enterprise Account**, copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé `B.Simon` dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory**, **Utilisateurs**, puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans le champ **Nom**, entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur**, entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

<a name="assign-the-azure-ad-test-user"></a>

### <a name="assign-your-azure-ad-user-and-the-test-user-account-to-the-github-app"></a>Affecter votre utilisateur Azure AD et le compte d’utilisateur de test à l’application GitHub

Dans cette section, vous allez autoriser `B.Simon` et votre compte d’utilisateur à utiliser l’authentification unique Azure en accordant l’accès au compte GitHub Enterprise Cloud - Enterprise.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **GitHub Enterprise Cloud – Enterprise Account**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** et votre compte d’utilisateur dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="enable-and-test-saml-for-the-enterprise-account-and-its-organizations"></a>Activer et tester SAML pour le compte Enterprise et ses organisations

Pour configurer l’authentification unique côté **compte GitHub Enterprise Cloud - Enterprise**, suivez les étapes listées dans [cette documentation GitHub](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-your-enterprise/enforcing-security-settings-in-your-enterprise-account#enabling-saml-single-sign-on-for-organizations-in-your-enterprise-account). 
1. Connectez-vous à GitHub.com avec un compte d’utilisateur qui est [propriétaire d’un compte Enterprise](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-your-enterprise/roles-in-an-enterprise#enterprise-owner). 
1. Copiez la valeur du champ `Login URL` de l’application à partir du portail Azure et collez-la dans le champ `Sign on URL` des paramètres SAML du compte GitHub Enterprise. 
1. Copiez la valeur du champ `Azure AD Identifier` de l’application à partir du portail Azure et collez-la dans le champ `Issuer` des paramètres SAML du compte GitHub Enterprise. 
1. Copiez le contenu du fichier de **certificat (en base64)** que vous avez téléchargé lors des étapes précédentes à partir du portail Azure, puis collez-le dans le champ approprié des paramètres SAML du compte GitHub Enterprise. 
1. Cliquez sur `Test SAML configuration` et confirmez que vous pouvez correctement vous authentifier à partir du compte GitHub Enterprise auprès d’Azure AD.
1. Une fois le test réussi, enregistrez les paramètres. 
1. Après l’authentification via SAML pour la première fois à partir du compte GitHub Enterprise, une _identité externe liée_ est créée dans le compte GitHub Enterprise qui associe le compte d’utilisateur GitHub connecté au compte d’utilisateur Azure AD.  
 
Une fois que vous avez activé l’authentification unique SAML pour votre compte GitHub Enterprise, l’authentification unique SAML est activée par défaut pour toutes les organisations appartenant à votre compte Enterprise. Tous les membres devront s’authentifier à l’aide de l’authentification unique SAML pour accéder aux organisations dont ils sont membres, et les propriétaires de compte Enterprise devront s’authentifier à l’aide de l’authentification unique SAML lors de l’accès à un compte Enterprise.

<a name="test-sso"></a>

## <a name="test-sso-with-another-enterprise-account-owner-or-organization-member-account"></a>Tester l’authentification unique avec un autre compte de membre de l’organisation ou propriétaire de compte Enterprise

Une fois l’intégration SAML configurée pour le compte GitHub Enterprise (qui s’applique également aux organisations GitHub dans le compte Enterprise), les autres propriétaires de compte Enterprise qui sont affectés à l’application dans Azure AD doivent pouvoir accéder à l’URL du compte GitHub Enterprise (`https://github.com/enterprises/<enterprise account>`), s’authentifier via SAML et accéder aux stratégies et aux paramètres sous le compte GitHub Enterprise. 

Le propriétaire d’une organisation dans un compte Enterprise doit être en mesure d’[inviter un utilisateur à rejoindre son organisation GitHub](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-organizations-and-teams/inviting-users-to-join-your-organization). Connectez-vous à GitHub.com avec le compte d’un propriétaire de l’organisation et suivez les étapes décrites dans l’article pour inviter `B.Simon` à rejoindre l’organisation. Un compte d’utilisateur GitHub doit être créé pour `B.Simon` s’il n’en existe pas déjà un. 

Pour tester l’accès à l’organisation GitHub sous le compte Enterprise avec le compte d’utilisateur de test `B.Simon` :
1. Invitez `B.Simon` à rejoindre une organisation sous le compte Enterprise en tant que propriétaire de l’organisation. 
1. Connectez-vous à GitHub.com avec le compte d’utilisateur que vous souhaitez lier au compte d’utilisateur Azure AD de `B.Simon`.
1. Connectez-vous à Azure AD avec le compte d’utilisateur de `B.Simon`.
1. Accédez à l’organisation GitHub. L’utilisateur doit être invité à s’authentifier via SAML. Après une authentification SAML réussie, `B.Simon` doit être en mesure d’accéder aux ressources de l’organisation. 

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer GitHub Enterprise Cloud – Enterprise Account avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Comment protéger GitHub Enterprise Cloud – Enterprise Account avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)
