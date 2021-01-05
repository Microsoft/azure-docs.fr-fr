---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Shopify Plus | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Shopify Plus.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/18/2020
ms.author: jeedes
ms.openlocfilehash: d7a5d160fa8a0209478fe81d2880e8956828350d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993497"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-shopify-plus"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Shopify Plus

Dans ce tutoriel, vous allez apprendre à intégrer Shopify Plus à Azure Active Directory (Azure AD). Quand vous intégrez Shopify Plus à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Shopify Plus.
* Permettre à vos utilisateurs de se connecter automatiquement à Shopify Plus avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Shopify Plus pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Shopify Plus prend en charge l’authentification unique initiée par le **fournisseur de services et le fournisseur d’identité**.

* Après avoir configuré Shopify Plus, vous pouvez appliquer le contrôle de session, qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-shopify-plus-from-the-gallery"></a>Ajout de Shopify Plus à partir de la galerie

Pour configurer l’intégration de Shopify Plus à Azure AD, vous devez ajouter Shopify Plus à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Shopify Plus** dans la zone de recherche.
1. Sélectionnez **Shopify Plus** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-shopify-plus"></a>Configurer et tester l’authentification unique Azure AD pour Shopify Plus

Configurez et testez l’authentification unique Azure AD avec Shopify Plus à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Shopify Plus associé.

Pour configurer et tester l’authentification unique Azure AD avec Shopify Plus, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Shopify Plus](#configure-shopify-plus-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Shopify Plus](#create-shopify-plus-test-user)** pour avoir un équivalent de B.Simon dans Shopify Plus lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Shopify Plus**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité**, entrez les valeurs pour les champs suivants :

    Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://accounts.shopify.com/saml/consume/organization/<ORGANIZATION_ID>`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion**, tapez l’URL : `https://shopify.plus/login`

    > [!NOTE]
    > La valeur de l’URL de réponse n’est pas réelle. Mettez à jour la valeur avec l’URL de réponse réelle. Pour obtenir cette valeur, contactez l’[équipe de support client Shopify Plus](mailto:plus-user-management@shopify.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. L’application Shopify Plus s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application Shopify Plus s’attend à ce que quelques attributs supplémentaires (présentés ci-dessous) soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source|
    | ---- | --------------- |
    | email | user.mail |

1. Remplacez le format de l’**Identificateur de nom** par **Persistant**. Sélectionnez l’option **Identificateur d’utilisateur unique (ID nom)** , puis sélectionnez le format **Identificateur de nom**. Sélectionnez **Persistant** pour cette option. Enregistrez vos modifications.
1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, sélectionnez le bouton de copie pour copier l’**URL des métadonnées de fédération d’application** et enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory**, **Utilisateurs**, puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans le champ **Nom**, entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur**, entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Shopify Plus.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Shopify Plus**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-shopify-plus-sso"></a>Configurer l’authentification unique Shopify Plus

Pour obtenir les étapes complètes, consultez la [documentation Shopify concernant la configuration des intégrations SAML](https://help.shopify.com/en/manual/shopify-plus/saml).

Si vous souhaitez configurer l’authentification unique pour **Shopify Plus**, copiez l’**URL des métadonnées de fédération d’application** à partir d’Azure Active Directory. Ensuite, connectez-vous en tant qu’[administrateur de l’organisation](https://shopify.plus), puis accédez à **Utilisateurs** > **Sécurité**. Sélectionnez **Set up configuration** (Configurer), puis collez l’URL des métadonnées de fédération d’application dans la section **Identity provider metadata URL** (URL de métadonnées du fournisseur d’identité). Pour terminer cette étape, sélectionnez **Add** (Ajouter).

### <a name="create-shopify-plus-test-user"></a>Créer un utilisateur de test Shopify Plus

Dans cette section, vous allez créer un utilisateur appelé B.Simon dans Shopify Plus. Revenez à la section **Users** (Utilisateurs) pour ajouter un utilisateur en entrant son adresse e-mail et ses autorisations. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

### <a name="enforce-saml-authentication"></a>Appliquer l’authentification SAML

> [!NOTE]
> Nous vous recommandons de tester l’intégration avec certains utilisateurs avant de l’appliquer à tous les utilisateurs.

Utilisateurs individuels :
1. Accédez à la page d’un utilisateur dans Shopify Plus avec un domaine de messagerie géré par Azure AD et vérifié dans Shopify Plus.
1. Dans la section SAML Authentication (Authentification SAML), sélectionnez **Edit** (Modifier), sélectionnez **Required** (Obligatoire), puis sélectionnez **Save** (Enregistrer).
1. Vérifiez que cet utilisateur peut se connecter à l’aide des flux lancés par le fournisseur d’identité ou le fournisseur de service.

Pour tous les utilisateurs d’un domaine de messagerie :
1. Retournez à la page **Security** (Sécurité).
1. Sélectionnez **Required** (Obligatoire) pour votre paramètre d’authentification SAML. Cela applique l’authentification SAML à tous les utilisateurs associés à ce domaine de messagerie dans Shopify Plus.
1. Sélectionnez **Enregistrer**.

> [!IMPORTANT]
> L’activation de l’authentification SAML pour tous les utilisateurs d’un domaine de messagerie affecte tous les utilisateurs qui utilisent cette application. Les utilisateurs ne pourront pas se connecter à partir de leur page de connexion standard. Ils pourront uniquement accéder à l’application par le biais d’Azure Active Directory. Shopify ne fournit pas d’URL de connexion de secours, à partir de laquelle les utilisateurs peuvent se connecter avec leur nom d’utilisateur et leur mot de passe habituels. Si nécessaire, vous pouvez contacter le support technique Shopify pour désactiver l’authentication SAML.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Shopify Plus dans le volet d’accès, vous devez être connecté automatiquement à l’application Shopify Plus pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Shopify Plus avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Shopify Plus avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)