---
title: 'Tutoriel : Intégration d’Azure Active Directory à Mitel Connect | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Mitel Connect.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/31/2020
ms.author: jeedes
ms.openlocfilehash: b89ae74a1f4ead568a5454c9494d4a82b0eeea3e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015177"
---
# <a name="tutorial-azure-active-directory-integration-with-mitel-micloud-connect-or-cloudlink-platform"></a>Tutoriel : Intégration d'Azure Active Directory à Mitel MiCloud Connect ou à la plateforme CloudLink

Dans ce tutoriel, vous allez apprendre à utiliser l'application Mitel Connect pour intégrer Azure Active Directory (Azure AD) à Mitel MiCloud Connect ou à la plateforme CloudLink. L'application Mitel Connect est disponible dans la galerie Azure. L'intégration d'Azure AD à MiCloud Connect ou à la plateforme CloudLink vous offre les avantages suivants :

* Vous pouvez contrôler l'accès des utilisateurs aux applications MiCloud Connect et CloudLink dans Azure AD à l'aide de leurs informations d'identification d'entreprise.
* Vous pouvez permettre aux utilisateurs de votre compte de se connecter automatiquement à MiCloud Connect ou CloudLink (par le biais de l'authentification unique) à l'aide de leur compte Azure AD.

Pour obtenir des informations sur l'intégration des applications SaaS à Azure AD, consultez [Qu'est-ce que l'accès aux applications et l'authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

Si vous n'avez pas d'abonnement Azure, [créez un compte gratuit](https://azure.microsoft.com/free/) avant d'entamer l'intégration d'Azure AD à Mitel MiCloud Connect ou à la plateforme CloudLink.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration de MiCloud Connect à Azure AD, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/).
* Un compte Mitel MiCloud Connect ou Mitel CloudLink, en fonction de l'application que vous souhaitez configurer

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD.

* Mitel Connect prend en charge l’authentification unique lancée par le **fournisseur de services**
* Après avoir configuré Mitel Connect, vous pouvez appliquer le contrôle de session, qui protège contre l'exfiltration et l'infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="add-mitel-connect-from-the-gallery"></a>Ajouter Mitel Connect à partir de la galerie

Pour configurer l’intégration de Mitel Connect à Azure AD, vous devez ajouter Mitel Connect, à partir de la galerie, à votre liste d’applications SaaS managées dans le portail Azure.

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Sélectionnez **Nouvelle application**.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Tapez **Mitel Connect** dans le champ de recherche, sélectionnez **Mitel Connect** dans le volet de résultats, puis sélectionnez **Ajouter**.

     ![Mitel Connect dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l'authentification unique Azure AD auprès de MiCloud Connect ou de la plateforme CloudLink avec un utilisateur de test appelé **_Britta Simon_**. Pour que l'authentification unique fonctionne, un lien doit être établi entre l'utilisateur du portail Azure AD et l'utilisateur correspondant de la plateforme Mitel. Reportez-vous aux sections suivantes pour obtenir des informations sur la configuration et le test de l'authentification unique Azure AD avec MiCloud Connect ou la plateforme CloudLink.
* Configurer et tester l'authentification unique Azure AD avec MiCloud Connect
* Configurer et tester l'authentification unique Azure AD avec la plateforme CloudLink

## <a name="configure-and-test-azure-ad-sso-with-micloud-connect"></a>Configurer et tester l'authentification unique Azure AD avec MiCloud Connect

Pour configurer et tester l'authentification unique Azure AD avec MiCloud Connect :

1. **[Configurer MiCloud Connect pour l’authentification unique avec Azure AD](#configure-micloud-connect-for-sso-with-azure-ad)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité et pour configurer les paramètres d’authentification unique côté application.
2. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
3. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
4. **[Créer un utilisateur de test Mitel MiCloud Connect](#create-a-mitel-micloud-connect-test-user)** pour avoir un équivalent de Britta Simon dans votre compte MiCloud Connect lié à la représentation Azure AD de l’utilisateur.
5. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

## <a name="configure-micloud-connect-for-sso-with-azure-ad"></a>Configurer MiCloud Connect pour l’authentification unique auprès d’Azure AD

Dans cette section, vous allez activer l’authentification unique Azure AD pour MiCloud Connect dans le portail Azure, et configurer votre compte MiCloud Connect pour autoriser l’authentification unique à l’aide d’Azure AD.

Pour configurer MiCloud Connect avec l’authentification unique pour Azure AD, il est plus facile d’ouvrir le portail Azure et le portail de compte Mitel côte à côte. Vous devrez copier certaines informations du portail Azure vers le portail de compte Mitel, et d’autres du portail de compte Mitel vers le portail Azure.


1. Pour ouvrir la page de configuration sur le [portail Azure](https://portal.azure.com/) :

    1. Sur la page d'intégration de l'application **Mitel Connect**, sélectionnez **Authentification unique**.

       ![Lien Configurer l’authentification unique](common/select-sso.png)

    1. Dans la boîte de dialogue **Sélectionner une méthode d'authentification unique**, sélectionnez **SAML**.
    
       ![Mode de sélection de l’authentification unique](common/select-saml-option.png)
    
       La page d’authentification basée sur SAML s’affiche.

2. Pour ouvrir la boîte de dialogue de configuration sur le portail de compte Mitel :

    1. Dans le menu **Système téléphonique**, sélectionnez **Fonctionnalités supplémentaires**.

    1. À droite de **Authentification unique**, sélectionnez **Activer** ou **Paramètres**.
    
    La boîte de dialogue Connect Single Sign-On Settings (Paramètres d’authentification unique Connect) s’affiche.
    
3. Cochez la case **Enable Single Sign-On** (Activer l’authentification unique).
    
    ![Capture d'écran montrant la page des paramètres d'authentification unique de Mitel Connect, avec la case Activer l'authentification unique cochée.](./media/mitel-connect-tutorial/mitel-connect-enable.png)

4. Sur le portail Azure, sélectionnez l'icône **Modifier** dans la section **Configuration SAML de base**.
   
    ![Capture d’écran montrant la page Configurer l’authentification unique avec SAML, dans laquelle l’icône Modifier est sélectionnée.](common/edit-urls.png)

    La boîte de dialogue Configuration SAML de base s’affiche.

5.  Copiez l’URL du champ **Mitel Identifier (Entity ID)** dans le portail de compte Mitel et collez-la dans le champ **Identificateur (ID d’entité)** dans le portail Azure.

6. Copiez l’URL du champ **Reply URL (Assertion Consumer Service URL)** dans le portail de compte Mitel et collez-la dans le champ **URL de réponse (URL Assertion Consumer Service)** dans le portail Azure.

   ![Capture d’écran montrant la page Configuration SAML de base dans le portail Azure, et la section Set Up Identity Provider dans le portail du compte Mitel ; des traits signalent la relation existante entre eux.](./media/mitel-connect-tutorial/mitel-azure-basic-configuration.png)

7. Dans la zone de texte **URL d’authentification**, tapez l’une des URL suivantes :

    1. **https://portal.shoretelsky.com** pour utiliser le portail de compte Mitel en tant qu’application Mitel par défaut.
    1. **https://teamwork.shoretel.com** pour utiliser Teamwork en tant qu’application Mitel par défaut.

    > [!NOTE]
    > L'application Mitel par défaut est celle à laquelle un utilisateur accède lorsqu'il sélectionne la vignette Mitel Connect dans le volet d'accès. Il s’agit également de l’application sollicitée lors d’une configuration de test à partir d’Azure AD.

8. Sélectionnez **Enregistrer** dans la boîte de dialogue **Configuration SAML de base** du portail Azure.

9. Dans la section **Certificat de signature SAML** de la page **Authentification basée sur SAML** du portail Azure, sélectionnez **Télécharger** en regard de **Certificat (Base64)** pour télécharger le **Certificat de signature** et l'enregistrer sur votre ordinateur.

    ![Capture d’écran montrant le volet Certificat de signature SAML dans lequel vous pouvez télécharger un certificat.](./media/mitel-connect-tutorial/azure-signing-certificate.png)

10. Ouvrez le fichier de certificat de signature dans un éditeur de texte, copiez toutes les données contenues dans le fichier, puis collez-les dans le champ **Signing Certificate** (Certificat de signature) dans le portail de compte Mitel. 

      ![Capture d’écran montrant le champ Signing Certificate.](./media/mitel-connect-tutorial/mitel-connect-signing-certificate.png)

11. Dans la section **Configurer Mitel Connect** de la page **Authentification basée sur SAML** du portail Azure :

     1. Copiez l’URL du champ **URL de connexion** et collez-la dans le champ **Sign-in URL** dans le portail de compte Mitel.

     1. Copiez l’URL du champ **Identificateur Azure AD** et collez-la dans le champ **Entity ID** dans le portail de compte Mitel.
         
         ![Capture d’écran montrant la relation existant entre la page d’authentification basée sur SAML du portail Azure et le portail du compte Mitel.](./media/mitel-connect-tutorial/mitel-azure-set-up-connect.png)

12. Sélectionnez **Enregistrer** dans la boîte de dialogue **Paramètres d'authentification unique Connect** du portail de compte Mitel.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

Dans cette section, vous allez créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans la boîte de dialogue Propriétés de l'utilisateur, procédez comme suit :

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    1. Dans le champ **Nom**, tapez **BrittaSimon**.
  
    1. Dans le champ **Nom d'utilisateur**, tapez brittasimon@\<yourcompanydomain\>.\<extension\>.  Par exemple : BrittaSimon@contoso.com.

    1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.

    1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Mitel Connect.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Mitel Connect**.

    ![Lien Mitel Connect dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste **Utilisateurs**, puis choisissez **Sélectionner** en bas de l'écran.

6. Si vous attendez une valeur de rôle dans l'assertion SAML, sélectionnez le rôle approprié pour l'utilisateur dans la liste dans la boîte de dialogue **Sélectionner un rôle**, puis choisissez **Sélectionner** en bas de l'écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Affecter**.

### <a name="create-a-mitel-micloud-connect-test-user"></a>Créer un utilisateur de test Mitel MiCloud Connect

Dans cette section, vous allez créer un utilisateur nommé Britta Simon dans votre compte MiCloud Connect. Les utilisateurs doivent être créés et activés avant l’utilisation de l’authentification unique.

Pour plus d’informations sur l’ajout d’utilisateurs dans le portail de compte Mitel, consultez l’article [Adding a User](https://shoretelcommunity.force.com/s/article/Adding-Users-092815) (Ajout d’un utilisateur) dans la Base de connaissances Mitel.

Créez un utilisateur dans votre compte MiCloud Connect avec les détails suivants :

* **Nom :** Britta Simon
* **Adresse e-mail professionnelle :** `brittasimon@<yourcompanydomain>.<extension>`   
  (Par exemple [brittasimon@contoso.com](mailto:brittasimon@contoso.com))
* **Nom d’utilisateur :** `brittasimon@<yourcompanydomain>.<extension>`  
  (Par exemple [brittasimon@contoso.com](mailto:brittasimon@contoso.com). Le nom d’utilisateur est généralement identique à l’adresse e-mail professionnelle de l’utilisateur)

> [!NOTE]
> Le nom d’utilisateur de l’utilisateur MiCloud Connect doit être identique à son adresse e-mail dans Azure.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous sélectionnez la vignette Mitel Connect dans le volet d'accès, vous devez être automatiquement redirigé pour vous connecter à l'application MiCloud Connect que vous avez configurée comme application par défaut dans le champ **URL de connexion**. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="configure-and-test-azure-ad-sso-with-cloudlink-platform"></a>Configurer et tester l'authentification unique Azure AD avec la plateforme CloudLink

Cette section explique comment activer l'authentification unique Azure AD pour la plateforme CloudLink sur le portail Azure et comment configurer le compte dont vous disposez sur la plateforme CloudLink pour permettre l'authentification unique à l'aide d'Azure AD.

Pour configurer la plateforme CloudLink avec l'authentification unique pour Azure AD, il est recommandé d'ouvrir le portail Azure et le portail des comptes CloudLink côte à côte, car vous allez devoir copier des informations du portail Azure vers le portail des comptes CloudLink et vice versa.

1. Pour ouvrir la page de configuration sur le [portail Azure](https://portal.azure.com/) :

    1. Sur la page d'intégration de l'application **Mitel Connect**, sélectionnez **Authentification unique**.

       ![Lien Configurer l’authentification unique](common/select-sso.png)

    1. Dans la boîte de dialogue **Sélectionner une méthode d'authentification unique**, sélectionnez **SAML**.

       ![Mode de sélection de l’authentification unique](common/select-saml-option.png)
    
       La page **Authentification basée sur SAML** s'ouvre et affiche la section **Configuration SAML de base**.

       ![Capture d’écran montrant la page Authentification basée sur SAML, avec la configuration SAML de base.](./media/mitel-connect-tutorial/mitel-azure-saml-settings.png)

2. Pour accéder au panneau de configuration **Authentification unique Azure AD** sur le portail des comptes CloudLink :

    1. Accédez à la page **Informations sur le compte** du compte client avec lequel vous souhaitez activer l'intégration.

    1. Dans la section **Intégrations**, sélectionnez **+ Ajouter une nouvelle intégration**. Un écran contextuel affiche le panneau **Intégrations**.

    1. Sélectionnez l'onglet **Tiers**. La liste des applications tierces prises en charge s'affiche. Sélectionnez le bouton **Ajouter** associé à l'**authentification unique Azure AD**, puis sélectionnez **Terminé**.

       ![Capture d’écran montrant la page « Integrations » dans laquelle vous pouvez ajouter l’authentification unique Azure AD.](./media/mitel-connect-tutorial/mitel-cloudlink-integrations.png)

       L'**authentification unique Azure AD** est activée pour le compte client et ajoutée à la section **Intégrations** de la page **Informations sur le compte**.   

   1. Sélectionnez **Terminer la configuration**.
    
      ![Capture d’écran montrant l’option Complete Setup pour l’authentification unique Azure AD.](./media/mitel-connect-tutorial/mitel-cloudlink-complete-setup.png)
      
      Le panneau de configuration **Authentification unique Azure AD** s'ouvre.
      
       ![Capture d’écran montrant la configuration de l’authentification unique Azure AD.](./media/mitel-connect-tutorial/mitel-cloudlink-sso-setup.png)
       
       Mitel recommande de ne pas cocher la case **Activer les informations d'identification Mitel (facultatif)** de la section **Informations d'identification Mitel facultatives**. Cochez cette case uniquement si vous souhaitez que l'utilisateur se connecte à l'application CloudLink à l'aide des informations d'identification Mitel en plus de l'option d'authentification unique.

3. Sur le portail Azure, accédez à la page **Authentification basée sur SAML** et sélectionnez l'icône **Modifier** de la section **Configuration SAML de base**. Le panneau **Configuration SAML de base** s'ouvre.

    ![Capture d’écran montrant le volet Configuration SAML de base avec l’icône Modifier sélectionnée.](./media/mitel-connect-tutorial/mitel-azure-saml-basic.png)
 
 4. Copiez l'URL figurant dans le champ **Identificateur Mitel (ID d'entité)** du portail des comptes CloudLink et collez-la dans le champ **Identificateur (ID d'entité)** du portail Azure.

 5. Copiez l'URL figurant dans le champ **URL de réponse (URL Assertion Consumer Service)** du portail des comptes CloudLink et collez-la dans le champ **URL de réponse (URL Assertion Consumer Service)** du portail Azure.  
    
    ![Capture d’écran montrant la relation existant entre la page du portail des comptes CloudLink et celle du portail Azure.](./media/mitel-connect-tutorial/mitel-cloudlink-saml-mapping.png) 

 6. Dans la zone de texte **URL de connexion**, entrez l'URL `https://accounts.mitel.io` afin d'utiliser le portail des comptes CloudLink comme application Mitel par défaut.
     
     ![Capture d’écran montrant la zone de texte URL de connexion.](./media/mitel-connect-tutorial/mitel-cloudlink-sign-on-url.png)
  
     > [!NOTE]
     > L'application Mitel par défaut est celle qui s'ouvre lorsqu'un utilisateur sélectionne la vignette Mitel Connect dans le volet d'accès. Il s'agit également de l'application à laquelle l'utilisateur accède lorsqu'il crée une configuration de test à partir d'Azure AD.

7. Sélectionnez **Enregistrer** dans la boîte de dialogue **Configuration SAML de base**.

8. Dans la section **Certificat de signature SAML** de la page **Authentification basée sur SAML** du portail Azure, sélectionnez **Télécharger** en regard de **Certificat (Base64)** pour télécharger le **Certificat de signature**. Enregistrez le certificat sur votre ordinateur.
  
    ![Capture d’écran montrant la section Certificat de signature SAML dans laquelle vous pouvez télécharger un certificat en base64.](./media/mitel-connect-tutorial/mitel-cloudlink-save-certificate.png)

9. Ouvrez le fichier de certificat de signature dans un éditeur de texte, copiez toutes les données qu'il contient, puis collez-les dans le champ **Certificat de signature** du portail des comptes CloudLink.  

    > [!NOTE]
    > Si vous disposez de plusieurs certificats, nous vous recommandons de les coller l'un après l'autre. 
       
    ![Capture d’écran montrant l’étape 2 de la procédure où vous renseignez des valeurs à partir de votre intégration Azure AD.](./media/mitel-connect-tutorial/mitel-cloudlink-enter-certificate.png)

10. Dans la section **Configurer Mitel Connect** de la page **Authentification basée sur SAML** du portail Azure :

     1. Copiez l'URL figurant dans le champ **URL de connexion** et collez-la dans le champ **URL de connexion** du portail des comptes CloudLink.

     1. Copiez l'URL figurant dans le champ **Identificateur Azure AD** et collez-la dans le champ **Identificateur de l'IDP (ID d'entité)** du portail des comptes CloudLink.
     
        ![Capture d’écran affichant la source des valeurs décrites ici dans Mitel Connect.](./media/mitel-connect-tutorial/mitel-cloudlink-copy-settings.png)

11. Sélectionnez **Enregistrer** dans le panneau **Authentification unique Azure AD** du portail des comptes CloudLink.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

Dans cette section, vous allez créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans la boîte de dialogue Propriétés de l'utilisateur, procédez comme suit :

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    1. Dans le champ **Nom**, tapez **BrittaSimon**.
  
    1. Dans le champ **Nom d'utilisateur**, tapez brittasimon@\<yourcompanydomain\>.\<extension\>.  Par exemple : BrittaSimon@contoso.com.

    1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.

    1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Mitel Connect.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Mitel Connect**.

    ![Lien Mitel Connect dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste **Utilisateurs**, puis choisissez **Sélectionner** en bas de l'écran.

6. Si vous attendez une valeur de rôle dans l'assertion SAML, sélectionnez le rôle approprié pour l'utilisateur dans la liste dans la boîte de dialogue **Sélectionner un rôle**, puis choisissez **Sélectionner** en bas de l'écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Affecter**.

### <a name="create-a-cloudlink-test-user"></a>Créer un utilisateur de test CloudLink

Cette section vous explique comment créer un utilisateur de test nommé **_Britta Simon_** sur votre plateforme CloudLink. Les utilisateurs doivent être créés et activés avant de pouvoir utiliser l'authentification unique.

Pour plus d'informations sur l'ajout d'utilisateurs sur le portail des comptes CloudLink, consultez la section **_Gérer les utilisateurs_** de la [documentation relative aux comptes CloudLink](https://www.mitel.com/document-center/technology/cloudlink/all-releases/en/cloudlink-accounts-html).

Créez un utilisateur sur le portail des comptes CloudLink, avec les détails suivants :

* Nom : Britta Simon
* Prénom : Britta
* Nom : Simon
* E-mail : BrittaSimon@contoso.com

> [!NOTE]
> L'adresse e-mail CloudLink de l'utilisateur doit être identique au **Nom d'utilisateur principal** figurant sur le portail Azure.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l'authentification unique Azure AD à l'aide du volet d'accès.

Lorsque vous sélectionnez la vignette Mitel Connect dans le volet d'accès, vous êtes automatiquement redirigé pour vous connecter à l'application CloudLink que vous avez configurée comme application par défaut dans le champ **URL de connexion**. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)