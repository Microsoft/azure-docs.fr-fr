---
title: "Tutoriel : Intégration d'Azure Active Directory à Sectigo Certificate Manager | Microsoft Docs"
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Sectigo Certificate Manager.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/15/2019
ms.author: jeedes
ms.openlocfilehash: c589db84b6221aa23868b3b49aea84f33623619f
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92673870"
---
# <a name="tutorial-azure-active-directory-integration-with-sectigo-certificate-manager"></a>Tutoriel : Intégration d'Azure Active Directory à Sectigo Certificate Manager

L’objectif de ce tutoriel est de vous apprendre à intégrer Sectigo Certificate Manager (également appelé SCM) à Azure Active Directory (Azure AD).

L’intégration de Sectigo Certificate Manager à Azure AD vous offre les avantages suivants :

* Vous pouvez utiliser Azure AD pour contrôler qui a accès à Sectigo Certificate Manager.
* Les utilisateurs peuvent être automatiquement connectés à Sectigo Certificate Manager avec leur compte Azure AD (par le biais de l’authentification unique).
* Vous pouvez centraliser la gestion de vos comptes à un seul emplacement : le Portail Azure.

Pour plus d’informations sur l’intégration d’applications software as a service (SaaS) à Azure AD, consultez l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Sectigo Certificate Manager, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’abonnement Azure AD, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Compte Sectigo Certificate Manager.

> [!NOTE]
> Sectigo exécute plusieurs instances de Sectigo Certificate Manager. L’instance principale de Sectigo Certificate Manager est **https:\//cert-manager.com**  ; c’est l’URL utilisée dans ce tutoriel.  Si votre compte se trouve sur une autre instance, vous devez ajuster les URL en conséquence.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test et intégrer Sectigo Certificate Manager à Azure AD.

Sectigo Certificate Manager prend en charge les fonctionnalités suivantes :

* **Authentification unique lancée par le fournisseur de services**
* **Authentification unique démarrée par le fournisseur d’identité**

## <a name="add-sectigo-certificate-manager-in-the-azure-portal"></a>Ajouter Sectigo Certificate Manager dans le portail Azure

Pour intégrer Sectigo Certificate Manager à Azure AD, vous devez ajouter Sectigo Certificate Manager à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans le menu de gauche, sélectionnez **Azure Active Directory** .

    ![Option Azure Active Directory](common/select-azuread.png)

1. Sélectionnez **Applications d’entreprise** > **Toutes les applications** .

    ![Volet Applications d’entreprise](common/enterprise-applications.png)

1. Pour ajouter une application, sélectionnez **Nouvelle application** .

    ![Option Nouvelle application](common/add-new-app.png)

1. Dans la zone de recherche, entrez **Sectigo Certificate Manager** . Dans les résultats de la recherche, sélectionnez **Sectigo Certificate Manager** , puis **Ajouter** .

    ![Sectigo Certificate Manager dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Sectigo Certificate Manager via un utilisateur de test appelé **Britta Simon** . Pour que l’authentification unique fonctionne, vous devez établir une relation entre l’utilisateur Azure AD et l’utilisateur Sectigo Certificate Manager.

Pour configurer et tester l’authentification unique d’Azure AD avec Sectigo Certificate Manager, vous devez suivre les indications des sections suivantes :

| Tâche | Description |
| --- | --- |
| **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** | Autorise les utilisateurs à utiliser cette fonctionnalité. |
| **[Configurer l’authentification unique de Sectigo Certificate Manager](#configure-sectigo-certificate-manager-single-sign-on)** | Configure les paramètres d’authentification unique dans l’application. |
| **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** | Teste l’authentification unique Azure AD pour un utilisateur appelé Britta Simon. |
| **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** | Active Britta Simon pour lui permettre d'utiliser l’authentification unique Azure AD. |
| **[Créer utilisateur de test Sectigo Certificate Manager](#create-a-sectigo-certificate-manager-test-user)** | Crée un équivalent de Britta Simon dans Sectigo Certificate Manager lié à la représentation d’un utilisateur Azure AD. |
| **[Tester l’authentification unique](#test-single-sign-on)** | Vérifie que la configuration fonctionne. |

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous configurez l’authentification unique Azure AD avec Sectigo Certificate Manager dans le portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), à la page d’intégration de l’application **Sectigo Certificate Manager** , sélectionnez **Authentification unique** .

    ![Configurer l’option d’authentification unique](common/select-sso.png)

1. Dans le volet **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML** ou **SAML/WS-Fed** pour activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

1. Dans le volet **Configurer l’authentification unique avec SAML** , sélectionnez **Modifier** (icône de crayon) pour ouvrir le volet **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    1. Dans la zone **Identificateur (ID d’entité)** , entrez **https:\//cert-manager.com/shibboleth** pour l’instance Sectigo Certificate Manager principale.

    1. Dans la zone **URL de réponse** , entrez **https:\//cert-manager.com/Shibboleth.sso/SAML2/POST** pour l’instance Sectigo Certificate Manager principale.
        
    > [!NOTE]
    > Bien qu’en général l’ **URL de connexion** soit obligatoire pour le mode *initié par le fournisseur de service* , il n’est pas nécessaire de se connecter à partir de Sectigo Certificate Manager.        

1. Le cas échéant, dans la section **Configuration SAML de base** , pour configurer le *mode initié par le fournisseur d’identité* et permettre l’utilisation de **Test** , effectuez les étapes suivantes :

    1. Sélectionnez **Définir des URL supplémentaires** .

    1. Dans la zone **État de relais** , entrez l’URL propre à votre client Sectigo Certificate Manager. Pour l’instance Sectigo Certificate Manager principale, entrez **https:\//cert-manager.com/customer/\<customerURI\>/idp** .

    ![Informations d’authentification unique pour le domaine et les URL de Sectigo Certificate Manager](common/idp-relay.png)

1. Dans la section **Attributs et revendications de l’utilisateur** , effectuez les étapes suivantes :

    1. Supprimez toutes les **Revendications supplémentaires** .
    
    1. Sélectionnez **Ajouter une nouvelle revendication** et ajoutez les quatre revendications suivantes :
    
        | Nom | Espace de noms | Source | Attribut source | Description |
        | --- | --- | --- | --- | --- |
        | eduPersonPrincipalName | empty | Attribut | user.userprincipalname | Doit correspondre au champ **IdP Person ID** (ID de la personne IDP) dans Sectigo Certificate Manager pour les administrateurs. |
        | mail | empty | Attribut | user.mail | Obligatoire |
        | givenName | empty | Attribut | user.givenname | Facultatif |
        | sn | empty | Attribut | user.surname | Facultatif |

       ![Sectigo Certificate Manager - Ajouter quatre nouvelles revendications](media/sectigo-certificate-manager-tutorial/additional-claims.png)

1. Dans la section **Certificat de signature SAML** , sélectionnez **Télécharger** en regard de **XML de métadonnées de fédération** . Enregistrez le fichier XML sur votre ordinateur.

    ![Option de téléchargement du fichier XML de métadonnées de fédération](common/metadataxml.png)

### <a name="configure-sectigo-certificate-manager-single-sign-on"></a>Configurer l’authentification unique de Sectigo Certificate Manager

Pour configurer l’authentification unique côté Sectigo Certificate Manager, envoyez le fichier XML de métadonnées de fédération téléchargé à l’[équipe du support technique Sectigo Certificate Manager](https://sectigo.com/support). L’équipe du support technique Sectigo Certificate Manager utilise les informations que vous lui envoyez pour vérifier que la connexion SAML à authentification unique est correctement configurée des deux côtés.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

Dans cette section, créez un utilisateur de test nommé Britta Simon dans le portail Azure.

1. Dans le portail Azure, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs** .

    ![Options Utilisateurs et Tous les utilisateurs](common/users.png)

1. Sélectionnez **Nouvel utilisateur** .

    ![Option Nouvel utilisateur](common/new-user.png)

1. Dans le volet **Utilisateur** , effectuez les étapes suivantes :

    1. Dans la zone **Nom** , entrez **BrittaSimon** .
  
    1. Dans la zone **Nom d’utilisateur** , entrez **brittasimon\@\<your-company-domain>.\<extension\>** . Par exemple, **brittasimon\@contoso.com** .

    1. Cochez la case **Afficher le mot de passe** . Prenez note de la valeur affichée dans la zone **Mot de passe** .

    1. Sélectionnez **Create** (Créer).

    ![Volet Utilisateur](common/user-properties.png)

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à accéder à Sectigo Certificate Manager et donc permettre à cet utilisateur d’utiliser l’authentification unique Azure.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications** > **Sectigo Certificate Manager** .

    ![Volet Applications d’entreprise](common/enterprise-applications.png)

1. Dans la liste des applications, sélectionnez **Sectigo Certificate Manager** .

    ![Sectigo Certificate Manager dans la liste des applications](common/all-applications.png)

1. Dans le menu, sélectionnez **Utilisateurs et groupes** .

    ![Option Utilisateurs et groupes](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** . Ensuite, dans le volet **Ajouter une attribution** , sélectionnez **Utilisateurs et groupes** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

1. Dans le volet **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste des utilisateurs. Choisissez **Select** .

1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans le volet **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste. Choisissez **Select** .

1. Dans le volet **Ajouter une attribution** , sélectionnez **Attribuer** .

### <a name="create-a-sectigo-certificate-manager-test-user"></a>Créer utilisateur de test Sectigo Certificate Manager

Dans cette section, vous créez un utilisateur appelé Britta Simon dans Sectigo Certificate Manager. Collaborez avec [l’équipe du support technique Sectigo Certificate Manager](https://sectigo.com/support) pour ajouter l'utilisateur à la plateforme Sectigo Certificate Manager. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD.

#### <a name="test-from-sectigo-certificate-manager-sp-initiated-single-sign-on"></a>Tester à partir de Sectigo Certificate Manager (Authentification unique initiée par le fournisseur de services)

Accédez à l’URL spécifique à votre client (pour l’instance Sectigo Certificate Manager principale, https:\//cert-manager.com/customer/\<customerURI\>/), puis sélectionnez le bouton en-dessous **Or Sign In With** (Ou se connecter avec).  Si la configuration est correcte, vous êtes automatiquement connecté à Sectigo Certificate Manager.

#### <a name="test-from-azure-single-sign-on-configuration-idp-initiated-single-sign-on"></a>Tester à partir de la configuration de l’authentification unique Azure (Authentification unique initiée par le fournisseur d’identité)

Dans le volet d’intégration de l’application **Sectigo Certificate Manager** , sélectionnez **Authentification unique** et sélectionnez le bouton **Tester** .  Si la configuration est correcte, vous êtes automatiquement connecté à Sectigo Certificate Manager.

#### <a name="test-by-using-the-my-apps-portal-idp-initiated-single-sign-on"></a>Tester en utilisant le portail Mes applications (Authentification unique initiée par le fournisseur d’identité)

Sélectionnez **Sectigo Certificate Manager** dans le portail Mes applications.  Si la configuration est correcte, vous êtes automatiquement connecté à Sectigo Certificate Manager. Pour plus d’informations sur le portail Mes applications, consultez [Accéder aux applications du portail Mes applications et les utiliser](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus, consultez les articles suivants :

- [Listes des tutoriels pour intégrer des applications SaaS à Azure Active Directory](./tutorial-list.md)
- [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md)
- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)