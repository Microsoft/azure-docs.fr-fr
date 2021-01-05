---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à AwareGo | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et AwareGo.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/07/2020
ms.author: jeedes
ms.openlocfilehash: 6599abd6282d0d1eb7cb81002c34ddd5158dab6b
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97511057"
---
# <a name="tutorial-azure-active-directory-single-sign-on-integration-with-awarego"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à AwareGo

Dans ce tutoriel, vous découvrez comment intégrer AwareGo à Azure Active Directory (Azure AD). Quand vous intégrez AwareGo à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à AwareGo
* Permettre à vos utilisateurs de se connecter automatiquement à AwareGo avec leur compte Azure AD
* Gérer vos comptes à un emplacement central, le portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement AwareGo pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. AwareGo prend en charge l’authentification unique lancée par le fournisseur de services.


## <a name="adding-awarego-from-the-gallery"></a>Ajout d’AwareGo à partir de la galerie

Pour configurer l’intégration d’AwareGo à Azure AD, vous devez ajouter AwareGo, disponible dans la galerie, à votre liste d’applications SaaS (Software as a Service) managées.

1. Connectez-vous au portail Azure avec un compte professionnel, un compte scolaire ou un compte personnel Microsoft.
1. Dans le volet de gauche, sélectionnez le service **Azure Active Directory**.
1. Sélectionnez **Applications d’entreprise** > **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **AwareGo** dans la zone de recherche.
1. Dans le volet de résultats, sélectionnez **AwareGo**, puis ajoutez l’application. Après quelques secondes, l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-awarego"></a>Configurer et tester l’authentification unique Azure AD pour AwareGo

Configurez et testez l’authentification unique Azure AD avec AwareGo à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur AwareGo associé.

Pour configurer et tester l’authentification unique Azure AD avec AwareGo, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.  

    a. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec l’utilisateur B.Simon.  
    b. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à l’utilisateur B.Simon d’utiliser l’authentification unique Azure AD.  

1. **[Configurer l’authentification unique AwareGo](#configure-awarego-sso)** pour configurer les paramètres de l’authentification unique côté application.

    a. **[Créer un utilisateur de test AwareGo](#create-an-awarego-test-user)** pour avoir un équivalent de B.Simon dans AwareGo lié à la représentation Azure AD associée.  
    b. **[Tester l’authentification SSO](#test-sso)** , pour vérifier que la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Pour activer l’authentification unique Azure AD dans le portail Azure, effectuez les étapes suivantes :

1. Dans le portail Azure, dans la page d’intégration de l’application **AwareGo**, sous **Gérer**, sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Pour modifier les paramètres, dans le volet **Configurer l’authentification unique avec SAML**, sélectionnez le bouton **Modifier**.

   ![Capture d’écran du bouton Modifier pour la configuration SAML de base.](common/edit-urls.png)

1. Dans le volet de modification, sous **Configuration SAML de base**, effectuez les étapes suivantes :

    a. Dans la zone **URL de connexion**, entrez l’une des URL suivantes :

    * `https://lms.awarego.com/auth/signin/` 
    * `https://my.awarego.com/auth/signin/`

    b. Dans la zone **Identificateur (ID d’entité)** , entrez une URL au format suivant : `https://<SUBDOMAIN>.awarego.com`

    c. Dans la zone **URL de réponse**, entrez une URL au format suivant : `https://<SUBDOMAIN>.awarego.com/auth/sso/callback`

    > [!NOTE]
    > Les valeurs ci-dessus ne sont pas réelles. Mettez-les à jour avec l’identificateur et les URL de réponse réels. Pour obtenir les valeurs, contactez l’[équipe de support client AwareGo](mailto:support@awarego.com). Vous pouvez également consulter les exemples figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, en regard de **Certificat (Base64)** , sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Capture d’écran du lien « Télécharger » du certificat dans le volet Certificat de signature SAML.](common/certificatebase64.png)

1. Dans la section **Configurer AwareGo**, copiez une ou plusieurs URL en fonction de vos besoins.

    ![Capture d’écran du volet « Configurer AwareGo » pour la copie des URL de configuration.](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon sur le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, puis **Utilisateurs** > **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans le volet des propriétés de l’**Utilisateur**, effectuez les étapes suivantes :

   a. Dans la zone **Nom**, entrez **B.Simon**.  
   b. Dans la zone **Nom d’utilisateur**, entrez le nom d’utilisateur au format suivant : `<username>@<companydomain>.<extension>` (par exemple, B.Simon@contoso.com).  
   c. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans la zone **Mot de passe** pour une utilisation ultérieure.  
   d. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à AwareGo.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications**.
1. Dans la liste **Applications**, sélectionnez **AwareGo**.
1. Dans la page de vue d’ensemble de l’application, dans la section **Gérer**, sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur** puis, dans le volet **Ajouter une attribution**, sélectionnez **Utilisateurs et groupes**.
1. Dans le volet **Utilisateurs et groupes**, dans la liste **Utilisateurs**, sélectionnez **B.Simon**, puis sélectionnez le bouton **Sélectionner**.
1. Si vous prévoyez d’attribuer un rôle aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle**. Si aucun rôle n’a été configuré pour cette application, le rôle *Accès par défaut* est sélectionné.
1. Dans le volet **Ajouter une attribution**, sélectionnez le bouton **Attribuer**.

## <a name="configure-awarego-sso"></a>Configurer l’authentification unique AwareGo

Pour configurer l’authentification unique côté **AwareGo**, envoyez le certificat **Certificat (base64)** téléchargé précédemment et les URL copiées plus tôt à partir du portail Azure à l’[équipe du support technique AwareGo](mailto:support@awarego.com). L’équipe de support crée ce paramètre pour établir correctement la connexion d’authentification unique SAML des deux côtés.

### <a name="create-an-awarego-test-user"></a>Créer un utilisateur de test AwareGo

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans AwareGo. Contactez l’[équipe du support technique AwareGo](mailto:support@awarego.com) pour ajouter des utilisateurs à la plateforme AwareGo. Vous devez créer et activer les utilisateurs avant d’utiliser l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous pouvez tester votre configuration de l’authentification unique Azure AD en procédant de l’une des façons suivantes : 

* Dans le portail Azure, sélectionnez **Tester cette application**. Vous êtes redirigé vers la page de connexion AwareGo où vous pouvez lancer le flux de connexion. 

* Accédez directement à la page de connexion AwareGo et lancez le flux de connexion.

* Accédez à Mes applications Microsoft. Quand vous sélectionnez la vignette **AwareGo** dans Mes applications, vous êtes redirigé vers la page de connexion AwareGo. Pour plus d’informations, consultez [Se connecter et démarrer des applications à partir du portail Mes applications](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).


## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré AwareGo, vous pouvez appliquer le contrôle de session qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir du contrôle d’application par accès conditionnel. Pour plus d’informations, consultez [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).


