---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Pulse Secure Virtual Traffic Manager | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Pulse Secure Virtual Traffic Manager.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/11/2020
ms.author: jeedes
ms.openlocfilehash: 1e295075a5c1ae8daa6673757770bbef01208c1d
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92505722"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-pulse-secure-virtual-traffic-manager"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Pulse Secure Virtual Traffic Manager

Dans ce tutoriel, vous allez découvrir comment intégrer Pulse Secure Virtual Traffic Manager à Azure Active Directory (Azure AD). Quand vous intégrez Pulse Secure Virtual Traffic Manager à Azure AD, vous pouvez :

* Contrôler, dans Azure AD, qui a accès à Pulse Secure Virtual Traffic Manager.
* Permettre à vos utilisateurs de se connecter automatiquement à Pulse Secure Virtual Traffic Manager avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Pulse Secure Virtual Traffic Manager pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Pulse Secure Virtual Traffic Manager prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-pulse-secure-virtual-traffic-manager-from-the-gallery"></a>Ajout de Pulse Secure Virtual Traffic Manager à partir de la galerie

Pour configurer l’intégration de Pulse Secure Virtual Traffic Manager à Azure AD, vous devez ajouter Pulse Secure Virtual Traffic Manager, disponible dans la galerie, à votre liste d’applications SaaS managées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Pulse Secure Virtual Traffic Manager** dans la zone de recherche.
1. Sélectionnez **Pulse Secure Virtual Traffic Manager** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-pulse-secure-virtual-traffic-manager"></a>Configurer et tester l’authentification unique Azure AD pour Pulse Secure Virtual Traffic Manager

Configurez et testez l’authentification unique Azure AD avec Pulse Secure Virtual Traffic Manager pour un utilisateur de test nommé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir une liaison entre un utilisateur Azure AD et l’utilisateur associé dans Pulse Secure Virtual Traffic Manager.

Pour configurer et tester l’authentification unique Azure AD avec Pulse Secure Virtual Traffic Manager, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Pulse Secure Virtual Traffic Manager](#configure-pulse-secure-virtual-traffic-manager-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Pulse Secure Virtual Traffic Manager](#create-pulse-secure-virtual-traffic-manager-test-user)** pour avoir, dans Pulse Secure Virtual Traffic Manager, un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Pulse Secure Virtual Traffic Manager** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<published virtual server FQDN>/saml/consume`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<published virtual server FQDN>/saml/metadata`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<published virtual server FQDN>/saml/consume`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe du support technique Pulse Secure Virtual Traffic Manager](mailto:support@pulsesecure.net). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Pulse Secure Virtual Traffic Manager** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Pulse Secure Virtual Traffic Manager.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Pulse Secure Virtual Traffic Manager** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .
1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .
1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-pulse-secure-virtual-traffic-manager-sso"></a>Configurer l’authentification unique Pulse Secure Virtual Traffic Manager

Cette section décrit la configuration nécessaire pour activer l’authentification SAML Azure AD sur Pulse Virtual Traffic Manager. Toutes les modifications de configuration sont apportées sur Pulse Virtual Traffic Manager à l’aide de l’interface utilisateur web d’administration. 

#### <a name="create-a-saml-trusted-identity-provider"></a>Créer un fournisseur d’identité approuvé SAML

a. Accédez à la page **Pulse Virtual Traffic Manager Appliance Admin UI > Catalog > SAML > Trusted Identity Providers Catalog** (Catalogue des fournisseurs d’identité approuvés), puis cliquez sur **Edit** (Modifier).

![page des catalogues SAML](./media/pulse-secure-virtual-traffic-manager-tutorial/saml-catalogs.png)

b. Ajoutez les détails du nouveau fournisseur d’identité approuvé SAML. Pour cela, copiez les informations de l’application Azure AD Enterprise sous la page des paramètres d’authentification unique, puis cliquez sur **Create New Trusted Identity Provider** (Créer un fournisseur d’identité approuvé).

![Créer un fournisseur d’identité approuvé](./media/pulse-secure-virtual-traffic-manager-tutorial/create-trusted-identity-provider.png)

* Dans la zone de texte **Name** (Nom), donnez un nom au fournisseur d’identité approuvé. 

* Dans la zone de texte **Entity_id** (ID d’entité), entrez la valeur **Identificateur Azure AD** que vous avez copiée dans le portail Azure.  

* Dans la zone de texte **URL** , entrez l’ **URL de connexion** que vous avez copiée dans le portail Azure. 

* Ouvrez le **Certificat** téléchargé à partir du portail Azure dans le Bloc-notes, puis collez le contenu dans la zone de texte **Certificate** (Certificat).

c. Vérifiez que le fournisseur d’identité SAML a été créé. 

![Vérifier le fournisseur d’identité approuvé](./media/pulse-secure-virtual-traffic-manager-tutorial/verify-saml-identity-provider.png)

#### <a name="configure-the-virtual-server-to-use-azure-ad-authentication"></a>Configurer le serveur virtuel pour utiliser l’authentification Azure AD

a. Accédez à la page **Pulse Virtual Traffic Manager Appliance Admin UI > Services > Virtual Servers** (Serveurs virtuels), puis cliquez sur **Edit** (Modifier) en regard du serveur virtuel créé précédemment.

![Modification des serveurs virtuels](./media/pulse-secure-virtual-traffic-manager-tutorial/virtual-servers.png)

b. Dans la section **Authentication** (Authentification), cliquez sur **Edit** (Modifier). 

![Section authentification](./media/pulse-secure-virtual-traffic-manager-tutorial/authentication.png)

c. Configurez les paramètres d’authentification suivants pour le serveur virtuel : 

1. Authentification :

    ![paramètres d’authentification du serveur virtuel](./media/pulse-secure-virtual-traffic-manager-tutorial/authentication-1.png)

    a. Dans **Auth!type** , sélectionnez **SAML Service Provider** (Fournisseur de services SAML). 

    b. Dans **Auth!verbose** , choisissez « Yes » (Oui) pour résoudre les problèmes d’authentification ; sinon, conservez la valeur par défaut (« Non »). 

2. Gestion des sessions d’authentification :

    ![Gestion des sessions d’authentification](./media/pulse-secure-virtual-traffic-manager-tutorial/authentication-session.png)

    a. Pour **Auth!session!cookie_name** , conservez la valeur par défaut (« VS_SamlSP_Auth »). 

    b. Pour **auth!session!timeout** , conservez la valeur par défaut (« 7200 »). 

    c. Dans **auth!session!log_external_state** , choisissez « Yes » (Oui) pour résoudre les problèmes d’authentification ; sinon, conservez la valeur par défaut (« Non »). 

    d. Dans **auth!session!cookie_attributes** , choisissez « HTTPOnly ». 

3. Fournisseur de services SAML :

    ![Fournisseur de services SAML](./media/pulse-secure-virtual-traffic-manager-tutorial/saml-service-provider.png)

    a. Dans la zone de texte **auth!saml!sp_entity_id** , définissez la même URL que celle utilisée comme identificateur de configuration de l’authentification unique Azure AD (ID d’entité). Par exemple, `https://pulseweb.labb.info/saml/metadata`. 

    b. Dans **auth!saml!sp_acs_url** , définissez la même URL que celle utilisée pour l’URL de relecture de la configuration de l’authentification unique Azure AD (URL Assertion Consumer Service). Par exemple, `https://pulseweb.labb.info/saml/consume`. 

    c. Dans **auth!saml!idp** , sélectionnez le **fournisseur d’identité approuvé** que vous avez créé à l’étape précédente. 

    d. Dans auth!saml!time_tolerance, laissez la valeur par défaut (« 5 » secondes). 

    e. Dans auth!saml!nameid_format, sélectionnez **unspecified** (non spécifié).

    f. Cliquez sur **Update** (Mettre à jour) en bas de la page pour appliquer les changements.
    
### <a name="create-pulse-secure-virtual-traffic-manager-test-user"></a>Créer un utilisateur de test Pulse Secure Virtual Traffic Manager

Dans cette section, vous créez un utilisateur appelé Britta Simon dans Pulse Secure Virtual Traffic Manager. Collaborez avec l’[équipe du support de Pulse Secure Virtual Traffic Manager](mailto:support@pulsesecure.net) pour ajouter des utilisateurs dans la plateforme Pulse Secure Virtual Traffic Manager. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

1. Cliquez sur **Tester cette application** dans le portail Azure. Vous êtes redirigé vers l’URL de connexion Pulse Secure Virtual Traffic Manager où vous pouvez lancer le flux de connexion. 

2. Accédez directement à l’URL de connexion Pulse Secure Virtual Traffic Manager pour lancer le flux de connexion.

3. Vous pouvez utiliser le volet d’accès Microsoft. Le fait de cliquer sur la vignette Pulse Secure Virtual Traffic Manager dans le volet d’accès vous redirige vers l’URL de connexion Pulse Secure Virtual Traffic Manager. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Pulse Secure Virtual Traffic Manager, vous pouvez appliquer le contrôle de session qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).