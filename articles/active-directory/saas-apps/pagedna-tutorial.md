---
title: 'Tutoriel : Intégration d’Azure Active Directory à PageDNA | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et PageDNA.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/03/2019
ms.author: jeedes
ms.openlocfilehash: e1ccd337851f8242526362675a11280e33584082
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993922"
---
# <a name="tutorial-azure-active-directory-integration-with-pagedna"></a>Tutoriel : Intégration d’Azure Active Directory à PageDNA

Dans ce tutoriel, vous allez apprendre à intégrer PageDNA à Azure Active Directory (Azure AD).

L’intégration de PageDNA à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à PageDNA.
* Vous pouvez permettre à vos utilisateurs de se connecter automatiquement à PageDNA (par le biais de l'authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure.

Pour obtenir des détails sur l’intégration d’applications SaaS (software as a service) avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à PageDNA, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Un abonnement PageDNA pour lequel l'authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l'authentification unique Azure AD dans un environnement de test et intégrer PageDNA à Azure AD.

PageDNA prend en charge les fonctionnalités suivantes :

* Authentification unique initiée par le fournisseur de services.

* Attribution d’utilisateurs juste-à-temps.

## <a name="add-pagedna-from-the-azure-marketplace"></a>Ajouter PageDNA à partir de la Place de marché Azure

Pour configurer l'intégration de PageDNA à Azure AD, vous devez ajouter PageDNA, disponible sur la Place de marché Azure, à votre liste d'applications SaaS managées :

1. Connectez-vous au [portail Azure](https://portal.azure.com?azure-portal=true).
1. Sélectionnez **Azure Active Directory** dans le volet de gauche.

    ![Option Azure Active Directory](common/select-azuread.png)

1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Volet Applications d’entreprise](common/enterprise-applications.png)

1. Pour ajouter une nouvelle application, sélectionnez **+ Nouvelle application** dans la partie supérieure du volet.

    ![Option Nouvelle application](common/add-new-app.png)

1. Dans la zone de recherche, entrez **PageDNA**. Dans les résultats de recherche, sélectionnez **PageDNA**, puis **Ajouter** pour ajouter l'application.

    ![PageDNA dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l'authentification unique Azure AD auprès de PageDNA à l'aide d'un utilisateur de test appelé **Britta Simon**. Pour que l'authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l'utilisateur PageDNA associé.

Pour configurer et tester l’authentification unique Azure AD avec PageDNA, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
1. **[Configurer l'authentification unique PageDNA](#configure-pagedna-single-sign-on)** pour configurer les paramètres de l'authentification unique côté application.
1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
1. **[Créer un utilisateur de test PageDNA](#create-a-pagedna-test-user)** de sorte qu'il existe dans PageDNA un utilisateur nommé Britta Simon qui soit lié à l'utilisateur Azure AD nommé Britta Simon.
1. **[Tester l’authentification unique](#test-single-sign-on)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l'authentification unique Azure AD auprès de PageDNA, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **PageDNA**, sélectionnez **Authentification unique**.

    ![Configurer l’option d’authentification unique](common/select-sso.png)

1. Dans le volet **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** pour activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

1. Dans le volet **Configurer l’authentification unique avec SAML**, sélectionnez **Modifier** (icône de crayon) pour ouvrir le volet **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans le volet **Configuration SAML de base**, procédez comme suit :

    ![Informations d'authentification unique dans Domaine et URL PageDNA](common/sp-identifier.png)

    1. Dans la zone **URL de connexion**, entrez une URL en utilisant l'un des formats suivants :

        ```https
        https://stores.pagedna.com/<your site>
        https://<your domain>
        https://<your domain>/<your site>
        https://www.nationsprint.com/<your site>
        ```

    1. Dans la zone **Identificateur (ID d'entité)** , entrez une URL en utilisant l'un des formats suivants :

        ```https
        https://stores.pagedna.com/<your site>/saml2ep.cgi
        https://www.nationsprint.com/<your site>/saml2ep.cgi
        ```

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez [l'équipe de support technique PageDNA](mailto:success@pagedna.com). Vous pouvez aussi vous référer aux modèles figurant dans le volet **Configuration SAML de base** sur le portail Azure.

1. Dans le volet **Configurer l'authentification unique avec SAML**, accédez à la section **Certificat de signature SAML**, sélectionnez **Télécharger** pour télécharger le fichier **Certificat (brut)** à partir des options données, puis enregistrez-le sur votre ordinateur.

    ![Option de téléchargement du certificat (brut)](common/certificateraw.png)

1. Dans la section **Configurer PageDNA**, copiez les URL dont vous avez besoin :

   * **URL de connexion**
   * **Identificateur Azure AD**
   * **URL de déconnexion**

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="configure-pagedna-single-sign-on"></a>Configurer l'authentification unique PageDNA

Pour configurer l'authentification unique côté PageDNA, envoyez le certificat (brut) téléchargé et les URL copiées correspondantes du portail Azure à l'[équipe du support PageDNA](mailto:success@pagedna.com). L'équipe PageDNA vérifiera que la connexion SSO SAML est correctement configurée des deux côtés.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test nommé Britta Simon dans le portail Azure.

1. Dans le portail Azure, dans le volet gauche, sélectionnez **Azure Active Directory**   > **Utilisateurs** > **Tous les utilisateurs**.

    ![Options Utilisateurs et « Tous les utilisateurs »](common/users.png)

1. En haut de l’écran, sélectionnez **+ Nouvel utilisateur**.

    ![Option Nouvel utilisateur](common/new-user.png)

1. Dans le volet **Utilisateur**, effectuez les étapes suivantes :

    ![Volet Utilisateur](common/user-properties.png)

    1. Dans la zone **Nom**, entrez **BrittaSimon**.
  
    1. Dans la zone **Nom d’utilisateur**, entrez **BrittaSimon\@\<yourcompanydomain>.\<extension>** . Par exemple, **BrittaSimon\@contoso.com**.

    1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.

    1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser l’utilisateur Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à PageDNA.

1. Sur le portail Azure, sélectionnez **Applications d'entreprise** > **Toutes les applications** > **PageDNA**.

    ![Volet Applications d’entreprise](common/enterprise-applications.png)

1. Dans la liste des applications, sélectionnez **PageDNA**.

    ![PageDNAdans la liste des applications](common/all-applications.png)

1. Dans le volet gauche, sous **GÉRER**, sélectionnez **Utilisateurs et groupes**.

    ![Option « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **+ Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans le volet **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

1. Dans le volet **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste **Utilisateurs**, puis choisissez **Sélectionner** au bas du volet.

1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans le volet **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste. Au bas du volet, choisissez **Sélectionner**.

1. Dans le volet **Ajouter une attribution**, sélectionnez **Attribuer**.

### <a name="create-a-pagedna-test-user"></a>Créer un utilisateur de test PageDNA

Un utilisateur nommé Britta Simon est maintenant créé dans PageDNA. Aucune action n’est nécessaire pour créer cet utilisateur. PageDNA prend en charge le provisionnement d’utilisateurs juste-à-temps, option activée par défaut. S'il n'existe pas encore d'utilisateur sous le nom de Britta Simon dans PageDNA, celui-ci est créé après l'authentification.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à partir du portail Mes applications.

Lorsque vous sélectionnez **PageDNA** sur le portail Mes applications, vous devez être automatiquement connecté à l'abonnement PageDNA pour lequel vous avez configuré l'authentification unique. Pour plus d’informations sur le portail Mes applications, consultez [Accéder aux applications du portail Mes applications et les utiliser](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Liste de tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

* [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md)

* [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)