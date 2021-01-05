---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à SiteIntel | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SiteIntel.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/02/2020
ms.author: jeedes
ms.openlocfilehash: 0c8224bf84a2235086d941df7d02ab6f458f3f16
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92509962"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-siteintel"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à SiteIntel

Dans ce tutoriel, vous allez apprendre à intégrer SiteIntel à Azure Active Directory (Azure AD). Quand vous intégrez SiteIntel à Azure AD, vous pouvez :

* Contrôler, dans Azure AD, qui a accès à SiteIntel.
* Permettre aux utilisateurs de se connecter automatiquement à SiteIntel avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central, le portail Azure.

Pour en savoir plus sur l’intégration d’applications software as a service (SaaS) à Azure AD, consultez l’article [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement SiteIntel pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* SiteIntel prend en charge l’authentification unique lancée par le fournisseur de services et le fournisseur d’identité.
* Après avoir configuré SiteIntel, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="add-siteintel-from-the-gallery"></a>Ajouter SiteIntel à partir de la galerie

Pour configurer l’intégration de SiteIntel à Azure AD, vous devez ajouter SiteIntel à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire, ou avec un compte personnel Microsoft.
1. Sélectionnez **Azure Active Directory** dans le volet de gauche.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la zone **Ajouter à partir de la galerie** , entrez **SiteIntel**.
1. Dans la liste des résultats, sélectionnez **SiteIntel** , puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-siteintel"></a>Configurer et tester l’authentification unique Azure AD pour SiteIntel

Configurez et testez l’authentification unique Azure AD avec SiteIntel à l’aide d’un utilisateur de test nommé *B.Simon*. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur SiteIntel associé.

Pour configurer et tester l’authentification unique Azure AD avec SiteIntel, suivez les indications des modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.  

    a. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec l’utilisateur B.Simon.  

    b. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à l’utilisateur B.Simon d’utiliser l’authentification unique Azure AD.

1. **[Configurer l’authentification unique SiteIntel](#configure-siteintel-sso)** pour configurer les paramètres d’authentification unique côté application.

    * **[Créer un utilisateur de test SiteIntel](#create-a-siteintel-test-user)** pour avoir, dans SiteIntel, un équivalent de l’utilisateur B.Simon qui soit lié à la représentation Azure AD de l’utilisateur.

1. **[Tester l’authentification SSO](#test-sso)** , pour vérifier que la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Pour activer l’authentification unique Azure AD dans le portail Azure, effectuez les étapes suivantes :

1. Dans la page d’intégration de l’application **SiteIntel** du [portail Azure](https://portal.azure.com/), recherchez la section **Gérer** , puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , en regard de **Configuration SAML de base** , sélectionnez l’icône **Modifier** (icône de stylet).

   ![Capture d’écran du volet « Configurer l’authentification unique avec SAML »](common/edit-urls.png)

1. Pour configurer l’application en mode lancé par le fournisseur d’identité, dans la section **Configuration SAML de base** , effectuez les opérations suivantes :

    a. Dans la zone **Identificateur** , tapez une URL au format suivant : `urn:amazon:cognito:sp:<REGION>_<USERPOOLID>`

    b. Dans la zone **URL de réponse** , tapez une URL au format suivant : `https://<CLIENT>.auth.siteintel.com/saml2/idpresponse`

    c. Dans la zone **État de relais** , tapez une URL au format suivant : `https://<CLIENT>.siteintel.com`

1. Pour configurer l’application en mode lancé par le fournisseur de services, sélectionnez **Définir des URL supplémentaires** , puis effectuez les opérations suivantes :

   * Dans la zone **URL de connexion** , tapez une URL au format suivant : `https://<CLIENT>.siteintel.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez-les à jour avec l’identificateur, l’URL de réponse, l’URL de connexion et l’état de relais réels. Pour obtenir ces valeurs, contactez l’[équipe du support technique de SiteIntel](mailto:support@intalytics.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la section **Certificat de signature SAML** de la page **Configurer l’authentification unique avec SAML** , sélectionnez le bouton **Copier** pour copier l’URL mentionnée dans la zone **URL des métadonnées de fédération d’application**.

    ![Capture d’écran du bouton de copie de l’« URL des métadonnées de fédération d’application »](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon sur le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** en haut du volet.
1. Dans les propriétés de l’ **Utilisateur** , effectuez les opérations suivantes :

   a. Dans la zone **Nom** , entrez **B.Simon**.  

   b. Dans la zone **Nom d’utilisateur** , entrez le nom d’utilisateur au format suivant : `username@companydomain.extension` (par exemple, `B.Simon@contoso.com`).

   c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe**.

   d. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez l’utilisateur B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à SiteIntel.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications**.
1. Dans la liste **Applications** , sélectionnez **SiteIntel**.
1. Dans la page de vue d’ensemble de l’application, dans la section **Gérer** , sélectionnez **Utilisateurs et groupes**.

   ![Capture d’écran du lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** puis, dans le volet **Ajouter une attribution** , sélectionnez **Utilisateurs et groupes**.

    ![Capture d’écran du bouton « Ajouter un utilisateur »](common/add-assign-user.png)

1. Dans le volet **Utilisateurs et groupes** , sélectionnez **B.Simon** , puis sélectionnez le bouton **Sélectionner** dans la partie inférieure de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans le volet **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis choisissez le bouton **Sélectionner**.
1. Dans le volet **Ajouter une attribution** , sélectionnez le bouton **Attribuer**.

## <a name="configure-siteintel-sso"></a>Configurer l’authentification unique SiteIntel

Pour configurer l’authentification unique côté SiteIntel, envoyez l’URL que vous avez copiée dans la zone **URL des métadonnées de fédération d’application** au membres de l’ [équipe de support technique de SiteIntel](mailto:support@intalytics.com). Ils définissent cette valeur pour établir correctement la connexion d’authentification unique (SSO) SAML des deux côtés.

### <a name="create-a-siteintel-test-user"></a>Créer un utilisateur de test SiteIntel

Dans cette section, vous créez un utilisateur appelé *Britta Simon* dans SiteIntel. Contactez l’[équipe du support technique de SiteIntel](mailto:support@intalytics.com) pour ajouter les utilisateurs dans la plateforme SiteIntel. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous sélectionnez la vignette **SiteIntel** dans le volet d’accès, vous devez être automatiquement connecté à l’application SiteIntel pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur les procédures d’intégration d’applications SaaS à Azure Active Directory](./tutorial-list.md)
- [Que sont l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)
- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)
- [Essayer SiteIntel avec Azure AD](https://aad.portal.azure.com/)
- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)
- [Guide pratique pour protéger SiteIntel avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)