---
title: 'Tutoriel : Intégration d’Azure Active Directory à Uberflip | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Uberflip.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/28/2019
ms.author: jeedes
ms.openlocfilehash: 6bf3eccec70619bf73397e315c6d851f753ec52f
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92521830"
---
# <a name="tutorial-azure-active-directory-integration-with-uberflip"></a>Tutoriel : Intégration d’Azure Active Directory à Uberflip

Dans ce tutoriel, vous allez apprendre à intégrer Uberflip à Azure Active Directory (Azure AD).

L’intégration d’Uberflip à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Uberflip.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Uberflip (avec l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure.

Pour obtenir des détails sur l’intégration d’applications SaaS (software as a service) avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Uberflip, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Un abonnement Uberflip pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

Uberflip prend en charge les fonctionnalités suivantes :

* Authentification unique initiée par le fournisseur de services (SP) et par le fournisseur d’identité (IDP).
* Attribution d’utilisateurs juste-à-temps.

## <a name="add-uberflip-from-the-azure-marketplace"></a>Ajouter Uberflip à partir de la Place de marché Azure

Pour configurer l’intégration d’Uberflip dans Azure AD, vous devez ajouter Uberflip à votre liste d’applications SaaS gérées à partir de la Place de marché Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez **Azure Active Directory** dans le volet de gauche.

   ![Option Azure Active Directory](common/select-azuread.png)

1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

   ![Volet Applications d’entreprise](common/enterprise-applications.png)

1. Pour ajouter une nouvelle application, sélectionnez **+ Nouvelle application** dans la partie supérieure du volet.

   ![Option Nouvelle application](common/add-new-app.png)

1. Dans la zone de recherche, entrez **Uberflip**. Dans les résultats de recherche, sélectionnez **Uberflip** , puis **Ajouter** pour ajouter l’application.

   ![Uberflip dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Uberflip sur un utilisateur de test nommé **B Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur correspondant dans Uberflip.

Pour configurer et tester l’authentification unique Azure AD avec Uberflip, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
1. **[Configurer l’authentification unique Uberflip](#configure-uberflip-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Créer un utilisateur de test Uberflip](#create-an-uberflip-test-user)** de sorte qu’il existe un utilisateur nommé B. Simon dans Uberflip qui soit lié à l’utilisateur Azure AD nommé B. Simon.
1. **[Tester l’authentification unique](#test-single-sign-on)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Uberflip, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Uberflip** , sélectionnez **Authentification unique**.

    ![Configurer l’option d’authentification unique](common/select-sso.png)

1. Dans le volet **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** pour activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

1. Dans le volet **Configurer l’authentification unique avec SAML** , sélectionnez **Modifier** (icône de crayon) pour ouvrir le volet **Configuration SAML de base**.

   ![Capture d’écran montrant Configuration SAML de base où vous pouvez entrer une URL de réponse](common/edit-urls.png)

1. Dans le volet **Configuration SAML de base** , effectuez l’une des étapes suivantes, selon le mode d’authentification unique que vous souhaitez configurer :

   * Pour configurer l’application en mode d’authentification unique initiée par le fournisseur d’identité (IDP), dans la zone **URL de réponse (URL Assertion Consumer Service)** , entrez une URL selon le modèle suivant :

     `https://app.uberflip.com/sso/saml2/<IDPID>/<ACCOUNTID>`

     ![Informations d’authentification unique dans Domaine et URL Uberflip](common/both-replyurl.png)

     > [!NOTE]
     > Cette valeur n’est pas réelle. Mettez à jour la valeur avec l’URL de réponse réelle. Pour obtenir la valeur réelle, contactez l’[équipe de support technique Uberflip](mailto:support@uberflip.com). Vous pouvez aussi vous référer aux modèles figurant dans le volet **Configuration SAML de base** sur le portail Azure.

   * Pour configurer l’application en mode d’authentification unique initiée par le fournisseur de services (SP), sélectionnez **Définir des URL supplémentaires** puis, dans la zone **URL de connexion** , entrez cette URL :

     `https://app.uberflip.com/users/login`

     ![Capture d’écran montrant Définir des URL supplémentaires, où vous pouvez entrer une URL de connexion.](common/both-signonurl.png)

1. Dans le volet **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , sélectionnez **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** à partir des options données et enregistrez-le sur votre ordinateur.

   ![Option de téléchargement du fichier XML de métadonnées de fédération](common/metadataxml.png)

1. Dans le volet **Configurer Uberflip** , copiez la ou les URL dont vous avez besoin :

   * **URL de connexion**
   * **Identificateur Azure AD**
   * **URL de déconnexion**

   ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="configure-uberflip-single-sign-on"></a>Configurer l’authentification unique Uberflip

Pour configurer l’authentification unique côté Uberflip, vous devez envoyer le fichier XML des métadonnées de fédération téléchargé et les URL appropriées, copiées à partir du portail Azure, à l’[équipe de support Uberflip](mailto:support@uberflip.com). L’équipe Uberflip vérifiera que la connexion avec l’authentification unique SAML est correctement configurée des deux côtés.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, créez un utilisateur de test nommé B. Simon dans le portail Azure.

1. Dans le portail Azure, dans le volet gauche, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs**.

    ![Options Utilisateurs et « Tous les utilisateurs »](common/users.png)

1. En haut de l’écran, sélectionnez **+ Nouvel utilisateur**.

    ![Option Nouvel utilisateur](common/new-user.png)

1. Dans le volet **Utilisateur** , effectuez les étapes suivantes :

    ![Volet Utilisateur](common/user-properties.png)

    1. Dans la zone **Nom** , entrez **BSimon**.
  
    1. Dans la zone **Nom d’utilisateur** , entrez **BSimon\@\<yourcompanydomain>.\<extension>** . Par exemple **BSimon\@contoso.com**.

    1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe**.

    1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Uberflip.

1. Sur le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications** > **Uberflip**.

    ![Volet Applications d’entreprise](common/enterprise-applications.png)

1. Dans la liste des applications, sélectionnez **Uberflip**.

    ![Uberflip dans la liste des applications](common/all-applications.png)

1. Dans le volet gauche, sous **GÉRER** , sélectionnez **Utilisateurs et groupes**.

    ![Option « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **+ Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans le volet **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

1. Dans le volet **Utilisateurs et groupes** , sélectionnez **B Simon** dans la liste **Utilisateurs** , puis choisissez **Sélectionner** au bas du volet.

1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans le volet **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste. Au bas du volet, choisissez **Sélectionner**.

1. Dans le volet **Ajouter une attribution** , sélectionnez **Attribuer**.

### <a name="create-an-uberflip-test-user"></a>Créer un utilisateur de test Uberflip

Un utilisateur nommé B. Simon est maintenant créé dans Uberflip. Aucune action n’est nécessaire pour créer cet utilisateur. Uberflip prend en charge le provisionnement d’utilisateurs juste-à-temps, activé par défaut. S’il n’existe pas déjà d’utilisateur sous le nom de B. Simon dans Uberflip, un nouveau est créé après l’authentification.

> [!NOTE]
> Si vous devez créer un utilisateur manuellement, contactez [l’équipe de support technique Uberflip](mailto:support@uberflip.com).

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à partir du portail Mes applications.

Quand vous sélectionnez **Uberflip** sur le portail Mes applications, vous devez être connecté automatiquement à l’abonnement Uberflip pour lequel vous avez configuré l’authentification unique. Pour plus d’informations sur le portail Mes applications, consultez [Accéder aux applications du portail Mes applications et les utiliser](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Liste de tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

* [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)