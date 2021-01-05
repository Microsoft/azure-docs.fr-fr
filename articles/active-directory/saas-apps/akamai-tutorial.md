---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Akamai | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Akamai.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/03/2020
ms.author: jeedes
ms.openlocfilehash: bbea6a0aa31034cd1c04145fb50b72432c9f8520
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92319012"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-akamai"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Akamai

Dans ce tutoriel, vous allez apprendre à intégrer Akamai à Azure Active Directory (Azure AD). Quand vous intégrez Akamai à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Akamai.
* Permettre à vos utilisateurs de se connecter automatiquement à Akamai avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

L’intégration d’Azure Active Directory et d’Akamai Enterprise Application Access permet un accès transparent aux applications héritées hébergées dans le cloud ou localement. La solution intégrée tire parti de toutes les fonctionnalités modernes d’Azure Active Directory, comme l’[accès conditionnel Azure AD](../conditional-access/overview.md), [Azure AD Identity Protection](../identity-protection/overview-identity-protection.md) et [Azure AD Identity Governance](../governance/identity-governance-overview.md), pour l’accès aux applications héritées sans installation d’agent ou modification des applications.

L’image ci-dessous décrit comment Akamai EAA s’intègre au scénario d’accès sécurisé hybride plus large.

![Akamai EAA s’intègre au scénario d’accès sécurisé hybride plus large](./media/header-akamai-tutorial/introduction01.png)

### <a name="key-authentication-scenarios"></a>Scénarios d’authentification par clé

En plus de la prise en charge de l’intégration native d’Azure Active Directory pour les protocoles d’authentification modernes que sont notamment Open ID Connect, SAML et WS-Fed, Akamai EAA étend l’accès sécurisé aux applications d’authentification héritées pour l’accès interne et externe avec Azure AD, ce qui autorise des scénarios modernes (par exemple, accès sans mot de passe) pour ces applications. notamment :

* Les applications à authentification basée sur l’en-tête
* Bureau à distance
* SSH (Secure Shell)
* Les applications à authentification Kerberos
* VNC (Virtual Network Computing)
* Les applications à authentification anonyme ou sans authentification intégrée
* Les applications à authentification NTLM (protection avec deux invites pour l’utilisateur)
* Les applications à base de formulaire (protection avec deux invites pour l’utilisateur)

### <a name="integration-scenarios"></a>Scénarios d’intégration

Le partenariat entre Microsoft et Akamai EAA offre la flexibilité nécessaire pour répondre aux besoins de votre entreprise en prenant en charge plusieurs scénarios d’intégration en fonction des besoins de votre entreprise. Ces scénarios peuvent être utilisés pour fournir une couverture immédiate de toutes les applications, et pour configurer progressivement les classifications de stratégie appropriées.

#### <a name="integration-scenario-1"></a>Scénario d’intégration 1

Akamai EAA est configuré en tant qu’application unique sur Azure AD. L’administrateur peut configurer la stratégie d’autorité de certification sur l’application, et une fois les conditions remplies les utilisateurs peuvent accéder au portail Akamai EAA.

**Avantages** :

* Vous ne devez configurer le fournisseur d’identité qu’une seule fois

**Inconvénients** :

* Les utilisateurs se retrouvent avec deux portails d’applications

* Couverture de la stratégie d’autorité de certification commune unique pour toutes les applications.

![Scénario d’intégration 1](./media/header-akamai-tutorial/scenario1.png)

#### <a name="integration-scenario-2"></a>Scénario d’intégration 2

L’application Akamai EAA est configurée individuellement dans le portail Azure AD. L’administrateur peut configurer une stratégie d’autorité de certification individuelle sur les applications, et une fois les conditions remplies, les utilisateurs peuvent être redirigés directement vers l’application spécifique.

**Avantages** :

* Vous pouvez définir des stratégies d’autorité de certification individuelles

* Toutes les applications sont représentées dans la gaufre Office 365 et le panneau myApps.microsoft.com.


**Inconvénients** :

* Vous devez configurer plusieurs fournisseurs d’identité.

![Scénario d’intégration 2](./media/header-akamai-tutorial/scenario2.png)

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Akamai pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

- Akamai prend en charge l’authentification unique lancée par le fournisseur d’identité

#### <a name="important"></a>Important

Toutes les configurations listées ci-dessous sont identiques pour le **scénario d’intégration 1** et le **scénario 2**. Pour le **scénario d’intégration 2**, vous devez configurer un fournisseur d’identité individuel dans Akamai EAA, et la propriété URL doit être modifiée pour pointer vers l’URL de l’application.

![Capture d’écran de l’onglet General pour AZURESSO-SP dans Akamai Enterprise Application Access. Le champ de l’URL de configuration de l’authentification est mis en évidence.](./media/header-akamai-tutorial/important.png)

## <a name="adding-akamai-from-the-gallery"></a>Ajout d’Akamai à partir de la galerie

Pour configurer l’intégration d’Akamai avec Azure AD, vous devez ajouter Akamai, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Akamai** dans la zone de recherche.
1. Sélectionnez **Akamai** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-akamai"></a>Configurer et tester l’authentification unique Azure AD pour Akamai

Configurez et testez l’authentification unique Azure AD avec Akamai à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Akamai associé.

Pour configurer et tester l’authentification unique Azure AD avec Akamai, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Akamai](#configure-akamai-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Configuration du fournisseur d’identité](#setting-up-idp)**
    * **[Authentification basée sur l’en-tête](#header-based-authentication)**
    * **[Bureau à distance](#remote-desktop)**
    * **[SSH](#ssh)**
    * **[Authentification Kerberos](#kerberos-authentication)**
    * **[Créer un utilisateur de test Akamai](#create-akamai-test-user)** pour avoir un équivalent de B.Simon dans Akamai, associé à sa représentation dans Azure AD.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Akamai**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `https://<Yourapp>.login.go.akamai-access.com/saml/sp/response`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https:// <Yourapp>.login.go.akamai-access.com/saml/sp/response`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique d’Akamai](https://www.akamai.com/us/en/contact-us/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer Akamai**, copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Akamai.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Akamai**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-akamai-sso"></a>Configurer l’authentification unique Akamai

### <a name="setting-up-idp"></a>Configuration du fournisseur d’identité

**Configuration du fournisseur d’identité Akamai EAA**

1. Connectez-vous à la console **Akamai Enterprise Application Access**.
1. Dans la console **Akamai EAA**, sélectionnez **Identity** > **Identity Providers** (Identité > Fournisseurs d’identité), puis cliquez sur **Add Identity Provider** (Ajouter un fournisseur d’identité).

    ![Capture d’écran de la fenêtre Identity Providers (Fournisseurs d’identité) de la console Akamai EAA. Dans la section Identity Providers (Fournisseurs d’identité) du menu Identity (Identité), sélectionnez Add Identity Provider (Ajouter un fournisseur d’identité).](./media/header-akamai-tutorial/configure01.png)

1. Dans la fenêtre **Create New Identity Provider** (Créer un fournisseur d’identité), effectuez les étapes suivantes :

    ![Capture d’écran de la boîte de dialogue Create New Identity Provider (Créer un fournisseur d’identité) dans la console Akamai EAA.](./media/header-akamai-tutorial/configure02.png)

    a. Spécifiez le nom unique (**Unique Name**).

    b. Choisissez **Third Party SAML** (SAML tiers), puis cliquez sur **Create Identity Provider and Configure** (Créer un fournisseur d’identité et configurer).

### <a name="general-settings"></a>Paramètres généraux

1. **Identity Intercept** (Interception d’identité) : spécifiez le nom de l’URL de base du fournisseur de service ; elle sera utilisée pour la configuration d’Azure AD.

    > [!NOTE]
    > Vous pouvez choisir d’avoir votre propre domaine personnalisé (nécessite une entrée DNS et un certificat). Pour cet exemple, nous allons utiliser le domaine Akamai.

1. **Akamai Cloud Zone** (Zone cloud Akamai) : sélectionnez la zone cloud appropriée.
1. **Certificate Validation** (Validation des certificats) : consultez la documentation Akamai (facultatif).

    ![Capture d’écran de l’onglet General de la console Akamai EAA montrant les paramètres pour Identity Intercept (Interception d’identité), Akamai Cloud Zone (Zone cloud Akamai) et Certificate Validation (Validation des certificats).](./media/header-akamai-tutorial/configure03.png)

### <a name="authentication-configuration"></a>Configuration de l’authentification

1. URL : spécifiez la même URL que celle d’interception d’identité (c’est là où les utilisateurs sont redirigés après l’authentification).
2. Logout URL (URL de déconnexion) : mettez à jour l’URL de déconnexion.
3. Sign SAML Request (Signer la requête SAML) : option désactivée par défaut.
4. Pour le fichier de métadonnées IDP, ajoutez l’application dans la console Azure AD.

    ![Capture d’écran de la configuration de l’authentification dans la console Akamai EAA montrant les paramètres pour URL, Logout URL (URL de déconnexion), Sign SAML Request (Signer la requête SAML) et IDP Metadata File (Fichier de métadonnées IDP).](./media/header-akamai-tutorial/configure04.png)

### <a name="session-settings"></a>Paramètres de session

Conservez les paramètres par défaut.

![Capture d’écran de la boîte de dialogue Session settings (Paramètres de session) de la console Akamai EAA.](./media/header-akamai-tutorial/sessionsettings.png)

### <a name="directories"></a>Répertoires

Ignorez la configuration de l’annuaire.

![Capture d’écran de l’onglet Directories (Annuaires) de la console Akamai EAA.](./media/header-akamai-tutorial/directories.png)

### <a name="customization-ui"></a>Personnalisation de l’interface utilisateur

Vous pouvez ajouter une personnalisation au fournisseur d’identité.

![Capture d’écran de l’onglet Customization (Personnalisation) de la console Akamai EAA montrant les paramètres pour Customize UI (Personnaliser l’interface utilisateur), Language settings (Paramètres de langue) et Themes (Thèmes).](./media/header-akamai-tutorial/customizationui.png)

### <a name="advanced-settings"></a>Paramètres avancés

Ignorez les paramètres avancés ; reportez-vous à la documentation Akamai pour plus d’informations.

![Capture d’écran de l’onglet Advanced Settings (Paramètres avancés) de la console Akamai EAA montrant les paramètres pour EAA Client (Client EAA), Advanced (Avancé) et OIDC to SAML bridging (Pontage d’OIDC à SAML).](./media/header-akamai-tutorial/advancesettings.png)

### <a name="deployment"></a>Déploiement

1. Cliquez sur Deploy Identity Provider (Déployer le fournisseur d’identité).

    ![Capture d’écran de l’onglet Deployment (Déploiement) de la console Akamai EAA montrant le bouton Deploy Identity Provider (Déployer le fournisseur d’identité).](./media/header-akamai-tutorial/deployment.png)

2. Vérifiez que déploiement a réussi.

### <a name="header-based-authentication"></a>Authentification basée sur l’en-tête

Authentification Akamai basée sur l’en-tête

1. Choisissez **Custom HTTP** (HTTP personnalisé) dans l’Assistant Ajout d’applications.

    ![Capture d’écran de l’assistant Add Applications (Ajout d’applications) de la console Akamai EAA montrant CustomHTTP dans la section Access Apps (Applications d’accès).](./media/header-akamai-tutorial/configure05.png)

2. Entrez le **nom de l’application** et une **description**.

    ![Capture d’écran d’une boîte de dialogue Custom HTTP App (Application HTTP personnalisée) montrant les paramètres pour Application Name (Nom de l’application) et Description.](./media/header-akamai-tutorial/configure06.png)

    ![Capture d’écran de l’onglet General de la console Akamai EAA montrant les paramètres généraux pour MYHEADERAPP.](./media/header-akamai-tutorial/configure07.png)

    ![Capture d’écran de la console Akamai EAA montrant les paramètres pour Certificate (Certificat) et Location (Emplacement).](./media/header-akamai-tutorial/configure08.png)

#### <a name="authentication"></a>Authentification

1. Sélectionnez l’onglet **Authentication**.

    ![Capture d’écran de la console Akamai EAA avec l’onglet Authentication (Authentification) sélectionné.](./media/header-akamai-tutorial/configure09.png)

2. Assignez le **fournisseur d’identité**.

    ![Capture d’écran de l’onglet Authentication (Authentification) de la console Akamai EAA pour MYHEADERAPP montrant Azure AD SSO défini comme fournisseur d’identité.](./media/header-akamai-tutorial/configure10.png)

#### <a name="services"></a>Services

Cliquez sur Save and Go to Authentication (Enregistrer et accéder à l’authentication).

![Capture d’écran de l’onglet Services de la console Akamai EAA pour MYHEADERAPP montrant le bouton Save and go to Advanced Settings (Enregistrer et accéder aux paramètres avancés) dans le coin inférieur droit.](./media/header-akamai-tutorial/configure11.png)

#### <a name="advanced-settings"></a>Paramètres avancés

1. Sous **Customer HTTP Headers** (En-têtes HTTP du client), spécifiez l’en-tête du client (**CustomerHeader**) et l’attribut SAML (**SAML Attribute**).

    ![Capture d’écran de l’onglet Advanced Settings (Paramètres avancés) de la console Akamai EAA montrant le champ SSO Logged URL (URL de connexion SSO) sous Authentication (Authentification).](./media/header-akamai-tutorial/configure12.png)

1. Cliquez sur le bouton **Save and go to Deployment** (Enregistrer et accéder au déploiement).

    ![Capture d’écran de l’onglet Advanced Settings (Paramètres avancés) de la console Akamai EAA montrant le bouton Save and go to Deployment (Enregistrer et accéder au déploiement) dans le coin inférieur droit.](./media/header-akamai-tutorial/configure13.png)

#### <a name="deploy-the-application"></a>Déployer l’application

1. Cliquez sur le bouton **Deploy Application** (Déployer l’application).

    ![Capture d’écran de l’onglet Deployment (Déploiement) de la console Akamai EAA montrant le bouton Deploy application (Déployer l’application).](./media/header-akamai-tutorial/configure14.png)

1. Vérifiez que l’application a été déployée correctement.

    ![Capture d’écran de l’onglet Deployment (Déploiement) de la console Akamai EAA montrant le message d’état de l’application : « Application Successfully Deployed » (Déploiement réussi de l’application).](./media/header-akamai-tutorial/configure15.png)

1. Expérience de l’utilisateur final.

    ![Capture de l’écran d’ouverture pour myapps.microsoft.com avec une image d’arrière-plan et une boîte de dialogue de connexion.](./media/header-akamai-tutorial/enduser01.png)

    ![Capture d’écran montrant une partie d’une fenêtre Applications avec des icônes pour Complément, HRWEB, Akamai - CorpApps, Dépense, Groupes et Révisions d’accès. ](./media/header-akamai-tutorial/enduser02.png)

1. Accès conditionnel.

    ![Capture d’écran du message : Approuver la demande de connexion. Nous avons envoyé une notification à votre appareil mobile. Veuillez répondre pour continuer.](./media/header-akamai-tutorial/conditionalaccess01.png)

    ![Capture d’un écran Applications montrant une icône pour MyHeaderApp.](./media/header-akamai-tutorial/conditionalaccess02.png)

#### <a name="remote-desktop"></a>Bureau à distance

1. Choisissez **RDP** dans l’Assistant Ajout d’applications.

    ![Capture d’écran de l’assistant Add Applications (Ajout d’applications) de la console Akamai EAA montrant RDP listé parmi les applications dans la section Access Apps (Applications d’accès).](./media/header-akamai-tutorial/configure16.png)

1. Entrez le **nom de l’application** et une **description**.

    ![Capture d’écran d’une boîte de dialogue RDP App (Application RDP) montrant les paramètres pour Application Name (Nom de l’application) et Description.](./media/header-akamai-tutorial/configure17.png)

    ![Capture d’écran de l’onglet General de la console Akamai EAA montrant les paramètres Application identity (Identité de l’application) pour SECRETRDPAPP.](./media/header-akamai-tutorial/configure18.png)

1. Spécifiez le connecteur qui servira à effectuer cette opération.

    ![Capture d’écran de la console Akamai EAA montrant les paramètres pour Certificate (Certificat) et Location (Emplacement). Les connecteurs associés sont définis avec USWST-CON1.](./media/header-akamai-tutorial/configure19.png)

#### <a name="authentication"></a>Authentification

Cliquez sur **Save and go to Services** (Enregistrer et accéder aux services).

![Capture d’écran de l’onglet Authentication (Authentification) de la console Akamai EAA pour SECRETRDPAPP montrant le bouton Save and go to Services (Enregistrer et accéder aux services) dans le coin inférieur droit.](./media/header-akamai-tutorial/configure20.png)

#### <a name="services"></a>Services

Cliquez sur **Save and go to Advanced Settings** (Enregistrer et accéder aux paramètres avancés).

![Capture d’écran de l’onglet Services de la console Akamai EAA pour SECRETRDPAPP montrant le bouton Save and go to Advanced Settings (Enregistrer et accéder aux paramètres avancés) dans le coin inférieur droit.](./media/header-akamai-tutorial/configure21.png)

#### <a name="advanced-settings"></a>Paramètres avancés

1. Cliquez sur **Save and go to Deployment**.

    ![Capture d’écran de l’onglet Advanced Settings (Paramètres avancés) de la console Akamai EAA pour SECRETRDPAPP montrant les paramètres pour Remote desktop configuration (Configuration du Bureau à distance).](./media/header-akamai-tutorial/configure22.png)

    ![Capture d’écran de l’onglet Advanced Settings (Paramètres avancés) de la console Akamai EAA pour SECRETRDPAPP montrant les paramètres pour Authentication (Authentification) et Health check configuration (Configuration du contrôle d’intégrité).](./media/header-akamai-tutorial/configure23.png)

    ![Capture d’écran des paramètres Custom HTTP headers (En-têtes HTTP personnalisés) de la console Akamai EAA pour SECRETRDPAPP montrant le bouton Save and go to Deployment (Enregistrer et accéder au déploiement) dans le coin inférieur droit.](./media/header-akamai-tutorial/configure24.png)

1. Expérience de l’utilisateur final

    ![Capture d’écran d’une fenêtre myapps.microsoft.com avec une image d’arrière-plan et une boîte de dialogue de connexion.](./media/header-akamai-tutorial/enduser03.png)

    ![Capture d’écran de la fenêtre Applications (myapps.microsoft.com) avec des icônes pour Complément, HRWEB, Akamai - CorpApps, Dépense, Groupes et Révisions d’accès.](./media/header-akamai-tutorial/enduser02.png)

1. Accès conditionnel

    ![Capture d’écran du message d’accès conditionnel : Approuver la demande de connexion. Nous avons envoyé une notification à votre appareil mobile. Veuillez répondre pour continuer.](./media/header-akamai-tutorial/conditionalaccess04.png)

    ![Capture d’un écran Applications montrant des icônes pour MyHeaderApp et SecretRDPApp.](./media/header-akamai-tutorial/conditionalaccess05.png)

    ![Capture de l’écran Windows Server 2012 RS montrant des icônes utilisateur génériques. Les icônes pour administrator, user0 et user1 indiquent que ces utilisateurs sont connectés.](./media/header-akamai-tutorial/conditionalaccess06.png)

1. Vous pouvez également taper directement l’URL de l’application RDP.

#### <a name="ssh"></a>SSH

1. Accédez à Add Applications (Ajouter des applications), puis choisissez **SSH**.

    ![Capture d’écran de l’Assistant Ajout d’applications de la console Akamai EAA montrant SSH listé parmi les applications dans la section Access Apps (Applications d’accès).](./media/header-akamai-tutorial/configure25.png)

1. Entrez le **nom de l’application** et une **description**.

    ![Capture d’écran d’une boîte de dialogue SSH App (Application SSH) montrant les paramètres pour Application Name (Nom de l’application) et Description.](./media/header-akamai-tutorial/configure26.png)

1. Configurer l’identité de l’application

    ![Capture d’écran de l’onglet General de la console Akamai EAA montrant les paramètres Application identity (Identité de l’application) pour SSH-SECURE.](./media/header-akamai-tutorial/configure27.png)

    a. Spécifiez le nom/la description.

    b. Spécifiez l’adresse IP/le nom de domaine complet du serveur d’applications et le port pour SSH.

    c. Spécifiez le nom d’utilisateur SSH/la phrase secrète *Consultez Akamai EAA.

    d. Spécifiez le nom d’hôte externe.

    e. Spécifiez l’emplacement du connecteur et choisissez le connecteur.

#### <a name="authentication"></a>Authentification

Cliquez sur **Save and go to Services**.

![Capture d’écran de l’onglet Authentication (Authentification) de la console Akamai EAA pour SSH-SECURE montrant le bouton Save and go to Services (Enregistrer et accéder aux services) dans le coin inférieur droit.](./media/header-akamai-tutorial/configure28.png)

#### <a name="services"></a>Services

Cliquez sur **Save and go to Advanced Settings** (Enregistrer et accéder aux paramètres avancés).

![Capture d’écran de l’onglet Services de la console Akamai EAA pour SSH-SECURE montrant le bouton Save and go to Advanced Settings (Enregistrer et accéder aux paramètres avancés) dans le coin inférieur droit.](./media/header-akamai-tutorial/configure29.png)

#### <a name="advanced-settings"></a>Paramètres avancés

Cliquez sur Save and go to Deployment.

![Capture d’écran de l’onglet Advanced Settings (Paramètres avancés) de la console Akamai EAA pour SSH-SECURE montrant les paramètres pour Authentication (Authentification) et Health check configuration (Configuration du contrôle d’intégrité).](./media/header-akamai-tutorial/configure30.png)

![Capture d’écran des paramètres Custom HTTP headers (En-têtes HTTP personnalisés) de la console Akamai EAA pour SSH-SECURE montrant le bouton Save and go to Deployment (Enregistrer et accéder au déploiement) dans le coin inférieur droit.](./media/header-akamai-tutorial/configure31.png)

#### <a name="deployment"></a>Déploiement

1. Cliquez sur **Deploy Application**.

    ![Capture d’écran de l’onglet Deployment (Déploiement) de la console Akamai EAA pour SSH-SECURE montrant le bouton Deploy application (Déployer l’application).](./media/header-akamai-tutorial/configure32.png)

1. Expérience de l’utilisateur final

    ![Capture d’écran de la boîte de dialogue de connexion dans la fenêtre myapps.microsoft.com.](./media/header-akamai-tutorial/enduser03.png)

    ![Capture d’écran de la fenêtre Applications pour myapps.microsoft.com montrant des icônes pour Complément, HRWEB, Akamai - CorpApps, Dépense, Groupes et Révisions d’accès.](./media/header-akamai-tutorial/enduser04.png)

1. Accès conditionnel

    ![Capture d’écran montrant le message : Approuver la demande de connexion. Nous avons envoyé une notification à votre appareil mobile. Veuillez répondre pour continuer.](./media/header-akamai-tutorial/conditionalaccess04.png)

    ![Capture d’un écran Applications montrant des icônes pour MyHeaderApp, SSH Secure et SecretRDPApp.](./media/header-akamai-tutorial/conditionalaccess07.png)

    ![Capture d’écran d’une fenêtre de commande pour ssh-secure-go.akamai-access.com montrant une invite de mot de passe.](./media/header-akamai-tutorial/conditionalaccess08.png)

    ![Capture d’écran d’une fenêtre de commande pour ssh-secure-go.akamai-access.com montrant des informations sur l’application et affichant une invite de commandes.](./media/header-akamai-tutorial/conditionalaccess09.png)

### <a name="kerberos-authentication"></a>Authentification Kerberos

Dans l’exemple ci-dessous, nous allons publier un serveur web interne <code>http://frp-app1.superdemo.live</code> et activer l’authentification unique à l’aide de KCD

#### <a name="general-tab"></a>Onglet Général

![Capture d’écran de l’onglet General de la console Akamai EAA pour MYKERBOROSAPP.](./media/header-akamai-tutorial/generaltab.png)

#### <a name="authentication-tab"></a>Onglet d’authentification

Assigner le fournisseur d’identité

![Capture d’écran de l’onglet Authentication (Authentification) de la console Akamai EAA pour MYKERBOROSAPP montrant Azure AD SSO défini comme fournisseur d’identité.](./media/header-akamai-tutorial/authenticationtab.png)

#### <a name="services-tab"></a>Onglet Services

![Capture d’écran de l’onglet Services de la console Akamai EAA pour MYKERBOROSAPP.](./media/header-akamai-tutorial/servicestab.png)

#### <a name="advanced-settings"></a>Paramètres avancés

![Capture d’écran de l’onglet Advanced Settings (Paramètres avancés) de la console Akamai EAA pour MYKERBOROSAPP montrant les paramètres pour Related Applications (Applications liées) et Authentication (Authentification).](./media/header-akamai-tutorial/advancesettings02.png)

> [!NOTE]
> Le nom de principal du service du serveur web est au format SPN@Domain (par exemple `HTTP/frp-app1.superdemo.live@SUPERDEMO.LIVE`, comme dans cette démonstration). Conservez le reste des paramètres par défaut.

#### <a name="deployment-tab"></a>Onglet Deployment

![Capture d’écran de l’onglet Deployment (Déploiement) de la console Akamai EAA pour MYKERBOROSAPP montrant le bouton Deploy application (Déployer l’application).](./media/header-akamai-tutorial/deploymenttab.png)

#### <a name="adding-directory"></a>Ajout de l’annuaire

1. Dans la liste déroulante, sélectionnez **AD**.

    ![Capture d’écran de la fenêtre Directories (Annuaires) de la console Akamai EAA montrant la boîte de dialogue Create New Directory (Créer un annuaire) avec AD sélectionné dans la liste déroulante Directory Type (Type d’annuaire).](./media/header-akamai-tutorial/configure33.png)

1. Fournissez les données nécessaires.

    ![Capture d’écran de la fenêtre SUPERDEMOLIVE de la console Akamai EAA avec les paramètres pour Directory Name (Nom d’annuaire), Directory Service (Service d’annuaire), Connector (Connecteur) et Attribute mapping (Mappage des attributs).](./media/header-akamai-tutorial/configure34.png)

1. Vérifiez la création de l’annuaire.

    ![Capture d’écran de la fenêtre Directories (Annuaires) de la console Akamai EAA montrant que l’annuaire superdemo.live a été ajouté.](./media/header-akamai-tutorial/directorydomain.png)

1. Ajoutez les groupes/unités d’organisation qui auraient besoin d’un accès.

    ![Capture d’écran des paramètres de pour l’annuaire superdemo.live. L’icône que vous sélectionnez pour ajouter des groupes ou des unités d’organisation est mise en évidence.](./media/header-akamai-tutorial/addgroup.png)

1. Ci-dessous, le groupe se nomme EAAGroup et compte un seul membre.

    ![Capture d’écran de la fenêtre GROUPS ON SUPERDEMOLIVE DIRECTORY (Groupes de l’annuaire SUPERDEMOLIVE) dans la console Akamai EAA. Le groupe EAAGroup avec 1 utilisateur est listé sous Groups (Groupes).](./media/header-akamai-tutorial/eaagroup.png)

1. Ajoutez l’annuaire au fournisseur d’identité en cliquant sur **Identity** > **Identity Providers** (Identité > Fournisseurs d’identité) et cliquez sur l’onglet **Directories** (Annuaires), puis sur **Assign directory** (Assigner un annuaire).

    ![Capture d’écran de la fenêtre Directories (Annuaires) de la console Akamai EAA pour Azure AD SSO montrant superdemo.live dans la liste Currently assigned directories (Anuaires actuellement affectés).](./media/header-akamai-tutorial/assigndirectory.png)

### <a name="configure-kcd-delegation-for-eaa-walkthrough"></a>Procédure pas à pas pour configurer la délégation KCD pour EAA

#### <a name="step-1-create-an-account"></a>Étape 1 : Créer un compte 

1. Dans l’exemple, nous allons utiliser un compte nommé **EAADelegation**. Vous pouvez effectuer cette opération à l’aide du composant logiciel enfichable **Utilisateurs et ordinateurs Active Directory**.

    ![Capture d’écran de l’onglet Directories (Annuaires) de la console Akamai EAA pour Azure AD SSO. L’annuaire superdemo.live est listé sous Currently assigned directories (Anuaires actuellement affectés).](./media/header-akamai-tutorial/assigndirectory.png)

    > [!NOTE]
    > Le nom d’utilisateur doit être dans un format spécifique basé sur le **nom d’interception d’identité**. Sur la figure 1, nous constatons qu’il s’agit de **corpapps.login.go.akamai-access.com**

1. Le nom d’ouverture de session de l’utilisateur sera : `HTTP/corpapps.login.go.akamai-access.com`

    ![Capture d’écran montrant les propriétés d’EAADelegation avec « EAADelegation » comme First name (Prénom) et HTTP/corpapps.login.go.akamai-access.com comme User logon name (Nom d’ouverture de session de l’utilisateur).](./media/header-akamai-tutorial/eaadelegation.png)

#### <a name="step-2-configure-the-spn-for-this-account"></a>Étape 2 : Configurer le SPN pour ce compte

1. Sur la base de cet exemple, le SPN se présentera comme suit.

2. setspn -s **Http/corpapps.login.go.akamai-access.com eaadelegation**

    ![Capture d’écran d’une invite de commandes d’administrateur montrant les résultats de la commande setspn -s Http/corpapps.login.go.akamai-access.com eaadelegation.](./media/header-akamai-tutorial/spn.png)

#### <a name="step-3-configure-delegation"></a>Étape 3 : Configurer la délégation

1. Pour le compte EAADelegation, cliquez sur l’onglet Delegation.

    ![Capture d’écran d’une invite de commandes d’administrateur montrant la commande permettant de configurer le SPN.](./media/header-akamai-tutorial/spn.png)

    * Spécifiez l’utilisation de tout protocole d’authentification.
    * Cliquez sur Add et ajoutez le compte du pool d’applications pour le site web Kerberos. Il doit être résolu automatiquement au SPN correct s’il est configuré correctement.

#### <a name="step-4-create-a-keytab-file-for-akamai-eaa"></a>Étape 4 : Créer un fichier Keytab pour AKAMAI EAA

1. Voici la syntaxe générique.

1. ktpass /out ActiveDirectorydomain.keytab  /princ `HTTP/yourloginportalurl@ADDomain.com`  /mapuser serviceaccount@ADdomain.com /pass +rdnPass  /crypto All /ptype KRB5_NT_PRINCIPAL

1. Explication de l’exemple

    | Extrait | Explication |
    | - | - |
    | Ktpass /out EAADemo.keytab | // Nom du fichier Keytab de sortie |
    | /princ HTTP/corpapps.login.go.akamai-access.com@superdemo.live | // HTTP/yourIDPName@YourdomainName |
    | /mapuser eaadelegation@superdemo.live | // Créer un compte de délégation EAA |
    | /pass RANDOMPASS | // Mot de passe du compte de délégation EAA |
    | /crypto All ptype KRB5_NT_PRINCIPAL | // consulter la documentation d’Akamai EAA |
    | | |

1. Ktpass /out EAADemo.keytab  /princ HTTP/corpapps.login.go.akamai-access.com@superdemo.live /mapuser eaadelegation@superdemo.live /pass RANDOMPASS /crypto All ptype KRB5_NT_PRINCIPAL

    ![Capture d’écran d’une invite de commandes d’administrateur montrant les résultats de la commande permettant de créer un fichier Keytab pour AKAMAI EAA.](./media/header-akamai-tutorial/administrator.png)

#### <a name="step-5-import-keytab-in-the-akamai-eaa-console"></a>Étape 5 : Importer le fichier Keytab dans la console AKAMAI EAA

1. Cliquez sur **System** > **Keytabs**.

    ![Capture d’écran de la console Akamai EAA montrant Keytabs sélectionné dans le menu System (Système).](./media/header-akamai-tutorial/keytabs.png)

1. Dans Keytab Type, choisissez **Kerberos Delegation**.

    ![Capture de l’écran EAAKEYTAB de la console Akamai EAA montrant les paramètres de Keytab. Keytab Type (Type de keytab) indique Kerberos Delegation (Délégation Kerberos).](./media/header-akamai-tutorial/keytabdelegation.png)

1. Vérifiez que le fichier Keytab apparaît comme étant déployé et vérifié.

    ![Capture de l’écran KEYTABS de la console Akamai EAA indiquant que le keytab EAA est « déployé et vérifié ».](./media/header-akamai-tutorial/keytabs02.png)

1. Expérience utilisateur

    ![Capture d’écran de la boîte de dialogue de connexion à l’adresse myapps.microsoft.com. ](./media/header-akamai-tutorial/enduser03.png)

    ![Capture d’écran de la fenêtre Applications pour myapps.microsoft.com montrant les icônes des applications.](./media/header-akamai-tutorial/enduser04.png)

1. Accès conditionnel

    ![Capture d’écran montrant un message Approuver la demande de connexion. le message.](./media/header-akamai-tutorial/conditionalaccess04.png)

    ![Capture d’un écran Applications montrant des icônes pour MyHeaderApp, SSH Secure, SecretRDPApp et myKerberosApp.](./media/header-akamai-tutorial/conditionalaccess10.png)

    ![Capture de l’écran de démarrage pour myKerberosApp. Le message « Welcome superdemo\user1 » (Bienvenue superdemo\user1 ) apparaît sur une image d’arrière-plan.](./media/header-akamai-tutorial/conditionalaccess11.png)

### <a name="create-akamai-test-user"></a>Créer un utilisateur de test Akamai

Dans cette section, vous allez créer un utilisateur appelé B.Simon dans Akamai. Collaborez avec l’[équipe du support technique Akamai](https://www.akamai.com/us/en/contact-us/) pour ajouter des utilisateurs à la plateforme Akamai. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique. 

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Akamai dans le panneau d’accès, vous devez être connecté automatiquement à l’application Akamai pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Akamai avec Azure AD](https://aad.portal.azure.com/)