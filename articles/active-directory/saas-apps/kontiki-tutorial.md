---
title: 'Didacticiel : Intégration d’Azure Active Directory avec Kontiki | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Kontiki.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/14/2019
ms.author: jeedes
ms.openlocfilehash: 9e0b438e0cfa729300099463086248780c69484c
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95019838"
---
# <a name="tutorial-azure-active-directory-integration-with-kontiki"></a>Didacticiel : Intégration d’Azure Active Directory à Kontiki

Dans ce didacticiel, vous allez apprendre à intégrer Kontiki à Azure AD (Azure Active Directory).

L’intégration de Kontiki à Azure AD vous offre les avantages suivants :

* Vous pouvez utiliser Azure AD pour contrôler qui a accès à Kontiki.
* Les utilisateurs peuvent être automatiquement connectés à Kontiki avec leur compte Azure AD (par le biais de l'authentification unique).
* Vous pouvez centraliser la gestion de vos comptes à un seul emplacement : le Portail Azure.

Pour plus d’informations sur l’intégration d’applications software as a service (SaaS) à Azure AD, consultez l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec Kontiki, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’abonnement Azure AD, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Un abonnement Kontiki pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test et intégrer Kontiki à Azure AD.

Kontiki prend en charge les fonctionnalités suivantes :

* **Authentification unique lancée par le fournisseur de services**
* **Attribution d’utilisateurs juste-à-temps**

## <a name="add-kontiki-in-the-azure-portal"></a>Ajouter Kontiki dans le portail Azure

Pour intégrer Kontiki à Azure AD, vous devez ajouter Kontiki à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans le menu de gauche, sélectionnez **Azure Active Directory**.

    ![Option Azure Active Directory](common/select-azuread.png)

1. Sélectionnez **Applications d’entreprise** > **Toutes les applications**.

    ![Volet Applications d’entreprise](common/enterprise-applications.png)

1. Pour ajouter une application, sélectionnez **Nouvelle application**.

    ![Option Nouvelle application](common/add-new-app.png)

1. Dans la zone de recherche, entrez **Kontiki**. Dans les résultats de la recherche, sélectionnez **Kontiki**, puis **Ajouter**.

    ![Kontiki dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Kontiki à l’aide d’un utilisateur de test appelé **Britta Simon**. Pour que l’authentification unique fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur Kontiki associé.

Pour configurer et tester l’authentification unique Azure AD avec Kontiki, vous devez suivre les indications des sections suivantes :

| Tâche | Description |
| --- | --- |
| **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** | Autorise les utilisateurs à utiliser cette fonctionnalité. |
| **[Configurer l’authentification unique Kontiki](#configure-kontiki-single-sign-on)** | Configure les paramètres d’authentification unique dans l’application. |
| **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** | Teste l’authentification unique Azure AD pour un utilisateur appelé Britta Simon. |
| **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** | Active Britta Simon pour lui permettre d'utiliser l’authentification unique Azure AD. |
| **[Créer un utilisateur de test Kontiki](#create-a-kontiki-test-user)** | Crée un équivalent de Britta Simon dans Kontiki lié à la représentation Azure AD associée. |
| **[Tester l’authentification unique](#test-single-sign-on)** | Vérifie que la configuration fonctionne. |

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous allez configurer l’authentification unique Azure AD avec Kontiki dans le portail Azure.

1. Sur le [portail Azure](https://portal.azure.com/), dans le volet d’intégration de l’application **Kontiki**, sélectionnez **Authentification unique**.

    ![Configurer l’option d’authentification unique](common/select-sso.png)

1. Dans le volet **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML** ou **SAML/WS-Fed** pour activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

1. Dans le volet **Configurer l’authentification unique avec SAML**, sélectionnez **Modifier** (icône de crayon) pour ouvrir le volet **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans le volet **Configuration SAML de base**, dans la zone de texte **URL de connexion**, entrez une URL au format suivant : `https://<companyname>.mc.eval.kontiki.com`

    ![Informations d’authentification unique dans Domaine et URL Kontiki](common/sp-signonurl.png)

    > [!NOTE]
    > Pour obtenir la valeur à utiliser, contactez l’[équipe du support technique de Kontiki](https://kollective.com/support/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans le volet **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, sélectionnez **Télécharger** en regard de **XML de métadonnées de fédération**. Sélectionnez une option de téléchargement en fonction de vos exigences. Enregistrez le certificat sur votre ordinateur.

    ![Option de téléchargement du certificat XML de métadonnées de fédération](common/metadataxml.png)

1. Dans la section **Configurer Kontiki**, copiez les URL suivantes en fonction de vos besoins :

    * URL de connexion
    * Identificateur Azure AD
    * URL de déconnexion

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="configure-kontiki-single-sign-on"></a>Configurer l’authentification unique Kontiki

Pour configurer l’authentification unique côté Kontiki, envoyez le fichier XML des métadonnées de fédération téléchargé et les URL appropriées copiées dans le portail Azure à l’[équipe du support technique Kontiki](https://kollective.com/support/). L’équipe du support technique Kontiki utilise les informations que vous lui envoyez pour vérifier que la connexion SAML à authentification unique est correctement configurée des deux côtés.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

Dans cette section, créez un utilisateur de test nommé Britta Simon dans le portail Azure.

1. Dans le portail Azure, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs**.

    ![Options Utilisateurs et Tous les utilisateurs](common/users.png)

1. Sélectionnez **Nouvel utilisateur**.

    ![Option Nouvel utilisateur](common/new-user.png)

1. Dans le volet **Utilisateur**, effectuez les étapes suivantes :

    1. Dans la zone **Nom**, entrez **BrittaSimon**.
  
    1. Dans la zone **Nom d’utilisateur**, entrez **brittasimon\@\<your-company-domain>.\<extension>** . Par exemple, **brittasimon\@contoso.com**.

    1. Cochez la case **Afficher le mot de passe**. Notez la valeur affichée dans la zone **Mot de passe**.

    1. Sélectionnez **Create** (Créer).

    ![Volet Utilisateur](common/user-properties.png)

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez Britta Simon à accéder à Kontiki pour lui permettre d'utiliser l’authentification unique Azure.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications** > **Kontiki**.

    ![Volet Applications d’entreprise](common/enterprise-applications.png)

1. Dans la liste des applications, sélectionnez **Kontiki**.

    ![Kontiki dans la liste des applications](common/all-applications.png)

1. Dans le menu, sélectionnez **Utilisateurs et groupes**.

    ![Option Utilisateurs et groupes](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**. Ensuite, dans le volet **Ajouter une attribution**, sélectionnez **Utilisateurs et groupes**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

1. Dans le volet **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste des utilisateurs. Choisissez **Select**.

1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans le volet **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste. Choisissez **Select**.

1. Dans le volet **Ajouter une attribution**, sélectionnez **Attribuer**.

### <a name="create-a-kontiki-test-user"></a>Créer un utilisateur de test Kontiki

Aucun élément d'action ne vous permet de configurer l’approvisionnement des utilisateurs dans Kontiki. Lorsqu’un utilisateur tente de se connecter à Kontiki à l’aide du portail Mes applications, Kontiki vérifie si cet utilisateur existe. Si aucun compte d’utilisateur n’est trouvé, Kontiki crée automatiquement le compte d’utilisateur.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à partir du portail Mes applications.

Après avoir configuré l’authentification unique, lorsque vous sélectionnez **Kontiki** dans le portail Mes applications, vous êtes automatiquement connecté à Kontiki. Pour plus d’informations sur le portail Mes applications, consultez [Accéder aux applications du portail Mes applications et les utiliser](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus, consultez les articles suivants :

- [Listes des tutoriels pour intégrer des applications SaaS à Azure Active Directory](./tutorial-list.md)
- [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md)
- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)