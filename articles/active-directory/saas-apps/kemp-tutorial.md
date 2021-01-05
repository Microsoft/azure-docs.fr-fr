---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Kemp LoadMaster Azure AD integration | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Kemp LoadMaster Azure AD integration.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/28/2020
ms.author: jeedes
ms.openlocfilehash: 7ac50acbf9b1b4371ac17f997828f9b8818e53b0
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92459112"
---
# <a name="tutorial-azure-active-directory-sso-integration-with-kemp-loadmaster-azure-ad-integration"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Kemp LoadMaster Azure AD integration

Dans ce tutoriel, vous allez apprendre à intégrer Kemp LoadMaster Azure AD integration à Azure Active Directory (Azure AD). Quand vous intégrez Kemp LoadMaster Azure AD integration à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Kemp LoadMaster Azure AD integration.
* Permettre à vos utilisateurs de se connecter automatiquement à Kemp LoadMaster Azure AD integration avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement à Kemp LoadMaster Azure AD integration pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Kemp LoadMaster Azure AD integration prend en charge l’authentification unique initiée par le **fournisseur d’identité**
* Après avoir configuré Kemp LoadMaster Azure AD integration, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-kemp-loadmaster-azure-ad-integration-from-the-gallery"></a>Ajout de Kemp LoadMaster Azure AD integration à partir de la galerie

Pour configurer l’intégration de Kemp LoadMaster Azure AD integration à Azure AD, vous devez ajouter Kemp LoadMaster Azure AD integration à votre liste d’applications SaaS gérées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Kemp LoadMaster Azure AD integration** dans la zone de recherche.
1. Sélectionnez **Kemp LoadMaster Azure AD integration** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-kemp-loadmaster-azure-ad-integration"></a>Configurer et tester l’authentification unique Azure AD pour Kemp LoadMaster Azure AD integration

Configurez et testez l’authentification unique Azure AD avec Kemp LoadMaster Azure AD integration pour un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans Kemp LoadMaster Azure AD integration.

Pour configurer et tester l’authentification unique Azure AD avec Kemp LoadMaster Azure AD integration, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Kemp LoadMaster Azure AD integration](#configure-kemp-loadmaster-azure-ad-integration-sso)** pour configurer les paramètres de l’authentification unique côté application.

1. **[Publication du serveur web](#publishing-web-server)**
    1. **[Créer un service virtuel](#create-a-virtual-service)**
    1. **[Certificats et sécurité](#certificates-and-security)**
    1. **[Profil SAML de Kemp LoadMaster Azure AD integration](#kemp-loadmaster-azure-ad-integration-saml-profile)**
    1. **[Vérifier les modifications](#verify-the-changes)**
1. **[Configurer l’authentification Kerberos](#configuring-kerberos-based-authentication)**
    1. **[Créer un compte de délégation Kerberos pour Kemp LoadMaster Azure AD integration](#create-a-kerberos-delegation-account-for-kemp-loadmaster-azure-ad-integration)**
    1. **[Comptes de délégation Kerberos (KCD) Kemp LoadMaster Azure AD integration](#kemp-loadmaster-azure-ad-integration-kcd-kerberos-delegation-accounts)**
    1. **[ESP Kemp LoadMaster Azure AD integration](#kemp-loadmaster-azure-ad-integration-esp)**
    1. **[Créer un utilisateur de test Kemp LoadMaster Azure AD integration](#create-kemp-loadmaster-azure-ad-integration-test-user)** pour avoir un équivalent de B.Simon dans Kemp LoadMaster Azure AD integration lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Sur le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Kemp LoadMaster Azure AD integration** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL : `https://<KEMP-CUSTOMER-DOMAIN>.com/`

    b. Dans la zone de texte **URL de réponse** , tapez l’URL : `https://<KEMP-CUSTOMER-DOMAIN>.com/`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez l’[équipe de support Kemp LoadMaster Azure AD integration](mailto:support@kemp.ax). Vous pouvez également consulter les modèles figurant à la section Configuration SAML de base dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** et **XML de métadonnées de fédération** , puis sélectionnez **Télécharger** pour télécharger le certificat et les fichiers XML de métadonnées et les enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](./media/kemp-tutorial/certificate-base-64.png)

1. Dans la section **Configurer Kemp LoadMaster Azure AD integration** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Kemp LoadMaster Azure AD integration.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Kemp LoadMaster Azure AD integration** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-kemp-loadmaster-azure-ad-integration-sso"></a>Configurer l’authentification unique Kemp LoadMaster Azure AD integration

## <a name="publishing-web-server"></a>Publication du serveur web 

### <a name="create-a-virtual-service"></a>Créer un service virtuel 

1. Accédez à l’interface utilisateur web Load Master de Kemp LoadMaster Azure AD integration > Virtual Services (Services virtuels) > Add New (Ajouter nouveau).

1. Cliquez sur Add New (Ajouter nouveau).

1. Spécifiez les paramètres pour le service virtuel.

    ![Capture d’écran montrant la page « Please Specify the Parameters for the Virtual Service » avec des exemples de valeurs dans les zones.](./media/kemp-tutorial/kemp-1.png)

    a. Virtual Address (Adresse virtuelle)
    
    b. Port
    
    c. Service Name (Optional) [Nom du service (facultatif)]
    
    d. Protocol 

1. Accédez à la section Real Servers (Serveurs réels).

1. Cliquez sur Add New (Ajouter nouveau).

1. Spécifiez les paramètres du serveur réel.
    
    ![Capture d’écran montrant la page « Please Specify the Parameters for the Real Server » avec des exemples de valeurs dans les zones.](./media/kemp-tutorial/kemp-2.png)

    a. Sélectionnez Allow Remote Addresses (Autoriser les adresses distantes)
    
    b. Entrez l’adresse du serveur réel dans le champ Real Server Address
    
    c. Port
    
    d. Forwarding method (Méthode de transfert)
    
    e. Poids
    
    f. Connection Limit (Limite de connexion)
    
    g. Cliquez sur Add This Real Server (Ajouter ce serveur réel)

## <a name="certificates-and-security"></a>Certificats et sécurité
 
### <a name="import-certificate-on-kemp-loadmaster-azure-ad-integration"></a>Importer un certificat dans Kemp LoadMaster Azure AD integration 
 
1. Accédez au portail web Kemp LoadMaster Azure AD integration > Certificates & Security (Certificats et sécurité) > SSL Certificates (Certificats SSL).

1. Sous Manage Certificates (Gérer les certificats) > Certificate Configuration (Configuration de certificat).

1. Cliquez sur Import Certificate (Importer un certificat).

1. Spécifiez le nom du fichier qui contient le certificat. Le fichier peut aussi contenir la clé privée. Si ce n’est pas le cas, le fichier contenant la clé privée doit également être spécifié. Le certificat peut être au format .PEM ou .PFX (IIS).

1. Cliquez sur Choose file (Choisir un fichier) dans Certificate File (Fichier de certificat).

1. Cliquez sur Key File (optional) [Fichier de clé (facultatif)].

1. Cliquez sur Save (Enregistrer).

### <a name="ssl-acceleration"></a>Accélération SSL
 
1. Accédez à l’interface utilisateur web Kemp LoadMaster > Virtual Services (Services virtuels) > View/Modify Services (Afficher/Modifier des services).

1. Cliquez sur Modify (Modifier) sous Operation (Opération).

1. Cliquez sur SSL Properties (Propriétés SSL) (qui fonctionne au niveau 7).
    
    ![Capture d’écran montrant la section « SSL Properties » avec l’option « SSL Acceleration - Enabled » et un exemple de certificat sélectionné.](./media/kemp-tutorial/kemp-3.png)
    
    a. Cliquez sur Enabled (Activé) dans SSL Acceleration (Accélération SSL).
    
    b. Sous Available Certificates (Certificats disponibles), sélectionnez le certificat importé, puis cliquez sur le symbole `>`.
    
    c. Une fois que le certificat SSL souhaité figure dans Assigned Certificates (Certificats attribués), cliquez sur **Set Certificates** (Définir des certificats).

    > [!NOTE]
    > Veillez à cliquer sur **Set Certificates** (Définir les certificats).

## <a name="kemp-loadmaster-azure-ad-integration-saml-profile"></a>Profil SAML de Kemp LoadMaster Azure AD integration
 
### <a name="import-idp-certificate"></a>Importer un certificat de fournisseur d’identité

Accédez à la console web Kemp LoadMaster Azure AD integration 

1. Cliquez sur Intermediate Certificates (Certificats intermédiaires) sous Certificates & Authority (Certificats et autorité).

    ![Capture d’écran montrant la section « Currently installed Intermediate Certificates » avec un exemple de certificat sélectionné.](./media/kemp-tutorial/kemp-6.png)

    a. Cliquez sur Choose File (Choisir un fichier) dans Add a new Intermediate Certificate (Ajouter un nouveau certificat intermédiaire).
    
    b. Accédez au fichier de certificat téléchargé précédemment à partir de l’application d’entreprise Azure AD.
    
    c. Cliquez sur Open (Ouvrir).
    
    d. Indiquez un nom dans Certificate Name.
    
    e. Cliquez sur Add Certificate (Ajouter le certificat).

### <a name="create-authentication-policy"></a>Créer une stratégie d’authentification 
 
Accédez à Manage SSO (Gérer l’authentification unique) sous Virtual Services (Services virtuels).

   ![Capture d’écran affichant la page « Manage SSO ».](./media/kemp-tutorial/kemp-7.png)
   
   a. Cliquez sur Add (Ajouter) sous Add new Client Side Configuration (Ajouter une nouvelle configuration côté client) après lui avoir attribué un nom.

   b. Sélectionnez SAML sous Authentication Protocol (Protocole d’authentification).

   c. Sélectionnez MetaData File (Fichier de métadonnées) sous IdP Provisioning (Provisionnement IdP). 

   d. Cliquez sur Choose File (Choisir un fichier).

   e. Accédez au fichier XML téléchargé précédemment à partir du portail Azure.

   f. Cliquez sur Open (Ouvrir), puis sur Import IdP MetaData file (Importer le fichier de métadonnées IdP).

   g. Sélectionnez le certificat intermédiaire à partir de IdP Certificate (Certificat IdP).

   h. Définissez l’ID d’entité du fournisseur de service dans le champ SP Entity ID ; il doit correspondre à l’identité créée sur le portail Azure 

   i. Cliquez sur Set SP Entity ID (Définir l’identité du fournisseur de service).

### <a name="set-authentication"></a>Définir l’authentification  
 
Dans la console web Kemp LoadMaster Azure AD integration

1. Cliquez sur Virtual Services (Services virtuels).

1. Cliquez sur View/Modify Services (Afficher/Modifier les services).

1. Cliquez sur Modify (Modifier) et accédez à ESP Options (Options ESP).
    
    ![Capture d’écran montrant la page « View/Modify Services », avec les sections « ESP Options » et « Real Servers » développées.](./media/kemp-tutorial/kemp-8.png)

    a. Cliquez sur Enable SSO (Activer l’authentification unique).
    
    b. Sélectionnez SAML dans Client Authentication Mode (Mode d’authentification du client).
    
    c. Sélectionnez l’authentification côté client créée précédemment dans SSO Domain (Domaine SSO).
    
    d. Tapez le nom d’hôte dans Allowed Virtual Hosts (Hôtes virtuels autorisés), puis cliquez sur Set Allowed Virtual Hosts (Définir les hôtes virtuels autorisés).
    
    e. Tapez /* dans Allowed Virtual Directories (Répertoires virtuels autorisés) selon les exigences d’accès, puis cliquez sur Set Allowed Directories (Définir les répertoires autorisés).

### <a name="verify-the-changes"></a>Vérifier les modifications 
 
Accédez à l’URL de l’application 

Votre page de connexion avec locataire doit s’afficher à la place de l’accès non authentifié précédent. 

![Capture d’écran montrant la page « Sign in » avec locataire.](./media/kemp-tutorial/kemp-9.png)

## <a name="configuring-kerberos-based-authentication"></a>Configurer l’authentification Kerberos 
 
### <a name="create-a-kerberos-delegation-account-for-kemp-loadmaster-azure-ad-integration"></a>Créer un compte de délégation Kerberos pour Kemp LoadMaster Azure AD integration 

1. Créez un compte d’utilisateur (dans cet exemple, AppDelegation).
    
    ![Capture d’écran montrant la fenêtre « Propriétés de l’utilisateur kcd » avec l’onglet « Compte » sélectionné.](./media/kemp-tutorial/kemp-10.png)


    a. Sélectionnez l’onglet Éditeur d’attributs.

    b. Accédez à servicePrincipalName. 

    c. Sélectionnez servicePrincipalName, puis cliquez sur Modifier.

    d. Tapez http/kcduser dans le champ Valeur à ajouter, puis cliquez sur Ajouter. 

    e. Cliquez sur Appliquer, puis sur OK. Vous devez fermer la fenêtre avant de l’ouvrir à nouveau (pour faire apparaître le nouvel onglet Délégation). 

1. Ouvrez à nouveau la fenêtre des propriétés utilisateur ; l’onglet Délégation est maintenant disponible. 

1. Sélectionnez l’onglet Délégation.

    ![Capture d’écran montrant la fenêtre « Propriétés de l’utilisateur kcd » avec l’onglet « Délégation » sélectionné.](./media/kemp-tutorial/kemp-11.png)

    a. Sélectionnez N’approuver cet utilisateur que pour la délégation aux services spécifiés.

    b. Sélectionnez Utiliser tout protocole d’authentification.

    c. Ajoutez les serveurs réels et ajoutez http comme service. 
    
    d. Cochez la case Développé. 

    e. Tous les serveurs s’affichent avec le nom d’hôte et le nom de domaine complet.
    
    f. Cliquez sur OK.

> [!NOTE]
> Définissez le SPN sur l’application ou le site web, selon le cas. Pour accéder à l’application quand l’identité du pool d’applications a été définie. Pour accéder à l’application IIS en utilisant le nom de domaine complet, accédez à l’invite de commandes du serveur réel, puis tapez SetSpn avec les paramètres nécessaires. Par exemple, Setspn –S HTTP/sescoindc.sunehes.co.in suneshes\kdcuser 

### <a name="kemp-loadmaster-azure-ad-integration-kcd-kerberos-delegation-accounts"></a>Comptes de délégation Kerberos (KCD) Kemp LoadMaster Azure AD integration 

Accédez à la console web Kemp LoadMaster Azure AD integration > Virtual Services (Services virtuels) > Manage SSO (Gérer l’authentification unique).

![Capture d’écran affichant la page « Manage SSO - Manage Domain ».](./media/kemp-tutorial/kemp-12.png)

a. Accédez aux configurations de l’authentification unique côté serveur.

b. Tapez le nom dans Add new Server-Side Configurations (Ajouter de nouvelles configurations côté serveur), puis cliquez sur Add (Ajouter).

c. Sélectionnez Kerberos Constrained Delegation (Délégation contrainte Kerberos) dans Authentication Protocol (Protocole d’authentification).

d. Tapez le nom de domaine dans Kerberos Realm (Domaine Kerberos).

e. Cliquez sur Set Kerberos realm (Définir le domaine Kerberos).

f. Tapez l’adresse IP du contrôleur de domaine dans Kerberos Key Distribution Center (Centre de distribution de clés Kerberos).

g. Cliquez sur Set Kerberos KDC (Définir le KDC Kerberos).

h. Tapez le nom d’utilisateur KCD dans Kerberos Trusted User Name (Nom d’utilisateur approuvé Kerberos).

i. Cliquez sur Set KDC trusted user name (Définir le nom d’utilisateur approuvé KDC).

j. Tapez le mot de passe dans Kerberos Trusted User Password (Mot de passe de l’utilisateur approuvé Kerberos).

k. Cliquez sur Set KCD trusted password (Définir le mot de passe de l’utilisateur approuvé KCD).

### <a name="kemp-loadmaster-azure-ad-integration-esp"></a>ESP Kemp LoadMaster Azure AD integration 

Accédez à Virtual Services (Services virtuels) > View/Modify Services (Afficher/Modifier les services).

![Serveur web Kemp LoadMaster Azure AD integration](./media/kemp-tutorial/kemp-13.png)

a. Cliquez sur Modify (Modifier) sur Nick Name (Surnom) du service virtuel.
    
b. Cliquez sur ESP Options (Options ESP).
    
c. Sous Server Authentication Mode (Mode d’authentification du serveur), sélectionnez KCD.
        
d. Sous Server-Side configuration (Configuration côté serveur), sélectionnez le profil côté serveur créé précédemment.

### <a name="create-kemp-loadmaster-azure-ad-integration-test-user"></a>Créer un utilisateur de test Kemp LoadMaster Azure AD integration

Dans cette section, vous allez créer un utilisateur appelé B.Simon dans Kemp LoadMaster Azure AD integration. Rapprochez-vous de l’[équipe de support Kemp LoadMaster Azure AD integration](mailto:support@kemp.ax) pour ajouter les utilisateurs dans la plateforme Kemp LoadMaster Azure AD integration. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Kemp LoadMaster Azure AD integration dans le volet d’accès, vous êtes connecté automatiquement à l’application Kemp LoadMaster Azure AD integration pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Kemp LoadMaster Azure AD integration avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Kemp LoadMaster Azure AD integration avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)