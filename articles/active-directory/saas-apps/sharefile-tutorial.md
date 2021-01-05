---
title: 'Didacticiel : Intégration d’Azure Active Directory à Citrix ShareFile | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Citrix ShareFile.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/04/2020
ms.author: jeedes
ms.openlocfilehash: e0cabcbaaf8e84ac1bdfd13c0ef4ab14f326fcf7
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96182283"
---
# <a name="tutorial-azure-active-directory-integration-with-citrix-sharefile"></a>Didacticiel : Intégration d’Azure Active Directory à Citrix ShareFile

Dans ce didacticiel, vous allez apprendre à intégrer Citrix ShareFile à Azure Active Directory (Azure AD).
L’intégration de Citrix ShareFile à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Citrix ShareFile.
* Vous pouvez autoriser les utilisateurs à se connecter automatiquement à Citrix ShareFile (via l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Citrix ShareFile, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Citrix ShareFile pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Citrix ShareFile prend en charge l’authentification unique (SSO) initiée par le **fournisseur de services**

## <a name="adding-citrix-sharefile-from-the-gallery"></a>Ajout de Citrix ShareFile à partir de la galerie

Pour configurer l’intégration de Citrix ShareFile avec Azure AD, vous devez ajouter Citrix ShareFile, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Citrix ShareFile** dans la zone de recherche.
1. Sélectionnez **Citrix ShareFile** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Citrix ShareFile avec un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur Citrix ShareFile associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Citrix ShareFile, procédez comme suit :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
2. **[Configurer l’authentification unique Citrix ShareFile](#configure-citrix-sharefile-sso)** pour configurer les paramètres d’authentification unique côté application.
    * **[Créer un utilisateur de test Citrix ShareFile](#create-citrix-sharefile-test-user)** pour avoir un équivalent de Britta Simon dans Citrix ShareFile, lié à la représentation Azure AD associée.
3. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Citrix ShareFile**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://<tenant-name>.sharefile.com/saml/login`.

    b. Dans la zone de texte **Identificateur (ID d’entité)**, entrez une URL au format suivant :

    - `https://<tenant-name>.sharefile.com`
    - `https://<tenant-name>.sharefile.com/saml/info`
    - `https://<tenant-name>.sharefile1.com/saml/info`
    - `https://<tenant-name>.sharefile1.eu/saml/info`
    - `https://<tenant-name>.sharefile.eu/saml/info`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : 
    
    - `https://<tenant-name>.sharefile.com/saml/acs`
    - `https://<tenant-name>.sharefile.eu/saml/<URL path>`
    - `https://<tenant-name>.sharefile.com/saml/<URL path>`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez [l’équipe du support Citrix ShareFile](https://www.citrix.co.in/products/citrix-content-collaboration/support.html). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

4. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer Citrix ShareFile**, copiez la ou les URL appropriées correspondant à vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Citrix ShareFile.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Citrix ShareFile**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-citrix-sharefile-sso"></a>Configurer l’authentification unique Citrix ShareFile

1. Dans une fenêtre de navigateur web, connectez-vous à votre site d'entreprise **Citrix ShareFile** en tant qu'administrateur.

1. Dans le tableau de bord (**Dashboard**), cliquez sur **Settings** (Paramètres), puis sélectionnez **Admin Settings** (Paramètres d’administration).

    ![Administration](./media/sharefile-tutorial/settings.png)

1. Dans les paramètres d’administration, accédez à **Security** -> **Login & Security Policy** (Stratégie de connexion et de sécurité).
   
    ![Administration des comptes](./media/sharefile-tutorial/settings-security.png "Administration des comptes")

1. Dans la page de boîte de dialogue **Authentification unique/Configuration de SAML 2.0** sous **Paramètres de base**, procédez comme suit :
   
    ![Authentification unique](./media/sharefile-tutorial/saml-configuration.png "Authentification unique")
   
    a. Sélectionnez **OUI** dans **Activer SAML**.

    b. Copiez la valeur **Émetteur / ID d’entité ShareFile** et collez-la dans la zone **URL d’identificateur** de la boîte de dialogue **Configuration SAML de base** du portail Azure.
    
    c. Dans la zone de texte **Your IDP Issuer/Entity ID** (Votre émetteur IDP/identificateur d’entité), collez **l’Identificateur Azure AD** que vous avez copié sur le Portail Azure.

    d. Cliquez sur **Change** (Modifier) en regard du champ **X.509 Certificate** (Certificat X.509), puis chargez le certificat que vous avez téléchargé à partir du portail Azure.
    
    e. Dans la zone de texte **Login URL** (URL de connexion), collez la valeur **URL de connexion** que vous avez copiée dans le Portail Azure.
    
    f. Dans la zone de texte **Logout URL** (URL de déconnexion), collez la valeur **URL de déconnexion** que vous avez copiée dans le Portail Azure.

5. Cliquez sur **Enregistrer** sur le portail de gestion Citrix ShareFile.

## <a name="create-citrix-sharefile-test-user"></a>Créer un utilisateur de test Citrix ShareFile

1. Connectez-vous à votre client **Citrix ShareFile**.

2. Cliquez sur **People** -> **Manage Users Home** -> **Create New Users** -> **Create Employee**.
   
    ![Créer un employé](./media/sharefile-tutorial/create-user.png "Créer un employé")

3. Dans la section **Basic Information**, procédez comme suit :
   
    ![Informations de base](./media/sharefile-tutorial/user-form.png "Informations de base")
   
    a. Dans la zone de texte **First Name**, entrez le **prénom** de l’utilisateur. Ici, il s’agit de **Britta**.
   
    b.  Dans la zone de texte **Last Name**, entrez le **nom** de l’utilisateur. Ici, il s’agit de **Simon**.
   
    c. Dans la zone de texte **Email Address** (Adresse e-mail), tapez l’adresse e-mail du compte de Britta Simon au format suivant : **brittasimon\@contoso.com**.

4. Cliquez sur **Add User**.
  
    >[!NOTE]
    >Le titulaire du compte Azure AD va recevoir un e-mail contenant un lien lui permettant de confirmer son compte avant que celui-ci ne soit activé. Vous pouvez utiliser n’importe quel autre outil de création de compte d’utilisateur Citrix ShareFile ou toute autre API fournie par Citrix ShareFile pour approvisionner des comptes d’utilisateur Azure AD.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

1. Cliquez sur **Tester cette application** dans le portail Azure. Cette opération redirige vers l’URL de connexion Citrix ShareFile, où vous pouvez lancer le processus de connexion. 

2. Accédez directement à l’URL de connexion Citrix ShareFile pour lancer le processus de connexion.

3. Vous pouvez utiliser le volet d’accès Microsoft. Le fait de cliquer sur la vignette Citrix ShareFile dans le volet d’accès vous redirige vers l’URL de connexion Citrix ShareFile. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Citrix ShareFile, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).