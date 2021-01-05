---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Cloud Academy - SSO'
description: Dans ce tutoriel, vous allez découvrir comment configurer l’authentification unique entre Azure Active Directory et Cloud Academy - SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/15/2020
ms.author: jeedes
ms.openlocfilehash: 96c4eba31013b868fa7afb41544d5d8cbcc1cdc6
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97607216"
---
# <a name="tutorial-azure-active-directory-single-sign-on-integration-with-cloud-academy---sso"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Cloud Academy - SSO

Dans ce tutoriel, vous allez découvrir comment intégrer Cloud Academy - SSO à Azure Active Directory (Azure AD). Quand vous intégrez Cloud Academy - SSO à Azure AD, vous pouvez :

* Utiliser Azure AD pour contrôler qui peut accéder à Cloud Academy - SSO.
* Permettre à vos utilisateurs de se connecter automatiquement à Cloud Academy - SSO avec leur compte Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Cloud Academy - SSO pour lequel l’authentification unique est activée.

## <a name="tutorial-description"></a>Description du tutoriel

Dans ce tutoriel, vous allez configurer et tester l’authentification SSO Azure AD dans un environnement de test.

* Cloud Academy - SSO prend en charge l’authentification unique lancée par le **fournisseur de services**
* Cloud Academy - SSO prend en charge l’attribution d’utilisateurs **juste-à-temps**

## <a name="add-cloud-academy---sso-from-the-gallery"></a>Ajouter Cloud Academy - SSO à partir de la galerie

Pour configurer l’intégration de Cloud Academy - SSO à Azure AD, vous devez ajouter Cloud Academy - SSO à votre liste d’applications SaaS managées à partir de la galerie :

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire, ou avec un compte Microsoft personnel.
1. Sélectionnez **Azure Active Directory** dans le volet de gauche.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, entrez **Cloud Academy - SSO** dans la zone de recherche.
1. Sélectionnez **Cloud Academy - SSO** dans le panneau des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-cloud-academy---sso"></a>Configurer et tester l’authentification unique Azure AD pour Cloud Academy - SSO

Vous allez configurer et tester l’authentification unique Azure AD avec Cloud Academy - SSO à l’aide d’un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Cloud Academy - SSO correspondant.

Pour configurer et tester l’authentification unique Azure AD avec Cloud Academy - SSO, vous allez effectuer les grandes étapes suivantes :

1. **[Configurer l’authentification SSO Azure AD](#configure-azure-ad-sso)** , pour permettre à vos utilisateurs d’utiliser la fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD.
    1. **[Octroyer l’accès à l’utilisateur de test](#grant-access-to-the-test-user)** , pour lui permettre d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique pour Cloud Academy - SSO](#configure-single-sign-on-for-cloud-academy)** côté application.
    1. **[Créer un utilisateur de test Cloud Academy - SSO](#create-a-cloud-academy-test-user)** comme contrepartie de la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification SSO](#test-sso)** , pour vérifier que la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le portail Azure :

1. Dans le portail Azure, dans la page d’intégration de l’application **Cloud Academy - SSO**, recherchez la section **Gérer**, puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez le bouton représentant un crayon et correspondant à **Configuration SAML de base** pour modifier les paramètres :

   ![Capture d’écran montrant le bouton avec l’icône de crayon pour la modification de la configuration SAML de base.](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, effectuez les étapes suivantes :

    a. Dans la zone de texte **URL d’authentification**, tapez l’une des URL suivantes :
    
    | URL de connexion |
    |--------------|
    | `https://cloudacademy.com/login/enterprise/` |
    | `https://app.qa.com/login/enterprise/` |
    |
    
    b. Dans la zone de texte **URL de réponse**, tapez l’une des URL suivantes :
    
    | URL de réponse |
    |--------------|
    | `https://cloudacademy.com/labs/social/complete/saml/` |
    | `https://app.qa.com/labs/social/complete/saml/` |
    |
1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, sélectionnez le bouton Copier pour copier l’**URL des métadonnées de fédération d’application**. Enregistrez l’URL.

    ![Capture d’écran du bouton Copier pour l’URL des métadonnées de fédération d’application.](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B.Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**. Sélectionnez **Utilisateurs**, puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans la zone **Nom**, entrez **B.Simon**.  
   1. Dans la zone **Nom d’utilisateur**, entrez \<username>@\<companydomain>.\<extension>. Par exemple : `B.Simon@contoso.com`.
   1. Sélectionnez **Afficher le mot de passe**, puis notez la valeur affichée dans la zone **Mot de passe**.
   1. Sélectionnez **Create** (Créer).

### <a name="grant-access-to-the-test-user"></a>Octroyer l’accès à l’utilisateur de test

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Cloud Academy - SSO.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Cloud Academy - SSO**.
1. Dans la page de vue d’ensemble de l’application, dans la section **Gérer**, sélectionnez **Utilisateurs et groupes** :
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** :
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B.Simon** dans la liste **Utilisateurs**, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Affecter**.

## <a name="configure-single-sign-on-for-cloud-academy"></a>Configurer l’authentification unique pour Cloud Academy

1. Dans une autre fenêtre de navigateur, connectez-vous à votre site d’entreprise Cloud Academy - SSO en tant qu’administrateur.

1. Sélectionnez le nom de votre société, puis sélectionnez **Settings & Integrations** (Paramètres et intégrations) dans le menu qui apparaît :

    ![Capture d’écran montrant l’option Settings & Integrations.](./media/cloud-academy-sso-tutorial/config-1.PNG)

1. Dans la page **Settings & Integrations**, accédez à l’onglet **Integrations** (Intégrations), puis sélectionnez la carte **SSO** :

    ![Capture d’écran montrant la carte SSO sous l’onglet Integrations.](./media/cloud-academy-sso-tutorial/config-2.PNG)

1. Effectuez les étapes suivantes dans cette page :

    ![Capture d’écran montrant la page Integrations > SSO.](./media/cloud-academy-sso-tutorial/config-3.PNG)

    a. Dans la zone **Entity ID URL** (URL de l’ID d’entité), collez la valeur de l’ID d’entité que vous avez copiée à partir du portail Azure.

    b. Dans la zone **SSO URL** (URL d’authentification unique), collez la valeur de l’URL de connexion que vous avez copiée à partir du portail Azure.

    c. Ouvrez le Certificat Base64 téléchargé à partir du portail Azure dans le Bloc-notes. Collez son contenu dans la zone **Certificate** (Certificat).

    d. Dans la zone de texte **Name ID Format** (Format de l’ID de nom), conservez la valeur par défaut : `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified`.

1. Sélectionnez **Enregistrer**.

    > [!NOTE]
    > Pour plus d’informations sur la configuration de Cloud Academy - SSO, consultez cet [Setting Up Single Sign-On](https://support.cloudacademy.com/hc/articles/360043908452-Setting-Up-Single-Sign-On).

### <a name="create-a-cloud-academy-test-user"></a>Créer un utilisateur de test Cloud Academy

Dans cette section, un utilisateur appelé Britta Simon est créé dans Cloud Academy - SSO. Cloud Academy - SSO prend en charge le provisionnement d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Cloud Academy - SSO, un utilisateur est créé après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

* Cliquez sur **Tester cette application** dans le portail Azure. Vous êtes redirigé vers l’URL de connexion Cloud Academy - SSO à partir de laquelle vous pouvez lancer le flux de connexion. 

* Accédez directement à l’URL de connexion Cloud Academy - SSO pour y lancer le flux de connexion.

* Vous pouvez utiliser Mes applications de Microsoft. Le fait de cliquer sur la vignette Cloud Academy - SSO dans Mes applications vous redirige vers l’URL de connexion Cloud Academy - SSO. Pour plus d’informations sur Mes applications, consultez [Présentation de Mes applications](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).


## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Cloud Academy - SSO, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).