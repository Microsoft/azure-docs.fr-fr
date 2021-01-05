---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Pulse Secure PCS | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Pulse Secure PCS.
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
ms.openlocfilehash: d38ff5c8f33bda5b12f6267e7a8cdf477db6c7d7
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92511441"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-pulse-secure-pcs"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Pulse Secure PCS

Dans ce tutoriel, vous allez découvrir comment intégrer Pulse Secure PCS à Azure Active Directory (Azure AD). Quand vous intégrez Pulse Secure PCS à Azure AD, vous pouvez :

* Contrôler, dans Azure AD, qui a accès à Pulse Secure PCS.
* Permettre à vos utilisateurs de se connecter automatiquement à Pulse Secure PCS avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Pulse Secure PCS pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Pulse Secure PCS prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-pulse-secure-pcs-from-the-gallery"></a>Ajout de Pulse Secure PCS à partir de la galerie

Pour configurer l’intégration de Pulse Secure PCS à Azure AD, vous devez ajouter Pulse Secure PCS, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **Pulse Secure PCS** dans la zone de recherche.
1. Sélectionnez **Pulse Secure PCS** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-pulse-secure-pcs"></a>Configurer et tester l’authentification unique Azure AD pour Pulse Secure PCS

Configurez et testez l’authentification unique Azure AD avec Pulse Secure PCS pour un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir une liaison entre un utilisateur Azure AD et l’utilisateur associé dans Pulse Secure PCS.

Pour configurer et tester l’authentification unique Azure AD avec Pulse Secure PCS, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Pulse Secure PCS](#configure-pulse-secure-pcs-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Pulse Secure PCS](#create-pulse-secure-pcs-test-user)** pour avoir, dans Pulse Secure PCS, un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Pulse Secure PCS** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<FQDN of PCS>/dana-na/auth/saml-consumer.cgi`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<FQDN of PCS>/dana-na/auth/saml-endpoint.cgi?p=sp1`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://[FQDN of PCS]/dana-na/auth/saml-consumer.cgi`


    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’URL de réponse et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe du support technique Pulse Secure PCS](mailto:support@pulsesecure.net). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Pulse Secure PC** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Pulse Secure PC.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Pulse Secure PC**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

## <a name="configure-pulse-secure-pcs-sso"></a>Configurer l’authentification unique Pulse Secure PC

Cette section aborde les configurations SAML nécessaires pour configurer PCS comme fournisseur de services SAML. Les autres configurations de base comme la création de domaines et de rôles ne sont pas traitées.

**Les configurations Pulse Connect Secure sont les suivantes :**

* Configuration d’Azure AD comme fournisseur de métadonnées SAML
* Configuration d’un serveur d’authentification SAML
* Affectation à des domaines et des rôles respectifs

#### <a name="configuring-azure-ad-as-saml-metadata-provider"></a>Configuration d’Azure AD comme fournisseur de métadonnées SAML

Effectuez les étapes ci-après dans la page suivante :

![Configuration Pulse Connect Secure](./media/pulse-secure-pcs-tutorial/saml-configuration.png)

1. Connectez-vous à la console d’administration Pulse Connect Secure
1. Accédez à **System -> Configuration -> SAML** (Système -> Configuration -> SAML)
1. Cliquez sur **New Metadata Provider** (Nouveau fournisseur de métadonnées)
1. Indiquez le nom valide dans la zone de texte **Name** (Nom)
1. Chargez le fichier XML de métadonnées téléchargé à partir du portail Azure dans le **Fichier de métadonnées Azure AD**.
1. Sélectionnez **Accept Unsigned Metadata** (Accepter les métadonnées non signées)
1. Sélectionnez Roles (Rôles) comme fournisseur d’identité ( **Identity Provider** )
1. Cliquez sur **Save changes** (Enregistrer les changements)

#### <a name="steps-to-create-a-saml-auth-server"></a>Étapes à suivre pour créer un serveur d’authentification SAML :

1. Accédez à **Authentication -> Auth Servers** (Authentification -> Serveurs d’authentification)
1. Sélectionnez **New: SAML Server** (Nouveau : Serveur SAML), puis cliquez sur **New Server** (Nouveau serveur)

    ![Serveur d’authentification Pulse Connect Secure](./media/pulse-secure-pcs-tutorial/new-saml-server.png)

1. Effectuez les étapes suivantes dans la page Paramètres :

    ![Paramètre du serveur d’authentification Pulse Connect Secure](./media/pulse-secure-pcs-tutorial/server-settings.png)

    a. Indiquez le nom du serveur dans la zone de texte **Server Name** (Nom du serveur).

    b. Sélectionnez **SAML version 2.0** et **Configuration Mode** (Mode de configuration) comme métadonnées ( **Metadata** ).

    c. Copiez la valeur **Connect Secure Entity Id** (ID d’entité Connect Secure), puis collez-la dans la zone **Identifier URL** (URL d’identificateur) de la boîte de dialogue **Basic SAML Configuration** (Configuration SAML de base) du portail Azure.

    d. Sélectionnez la valeur d’ID d’entité Azure AD dans la liste déroulante **Identity Provider Entity Id** (ID d’entité du fournisseur d’identité).

    e. Sélectionnez la valeur d’URL de connexion Azure AD dans la liste déroulante **Identity Provider Single Sign-On Service URL** (URL du service d’authentification unique du fournisseur d’identité).

    f. **Single Logout** (Déconnexion unique) est un paramètre facultatif. Si cette option est sélectionnée, il demande une nouvelle authentification après la déconnexion. Si cette option n’est pas sélectionnée et que vous n’avez pas fermé le navigateur, vous pouvez vous reconnecter sans authentification.

    g. Sélectionnez **Password** (Mot de passe) comme valeur **Requested Authn Context Class** (Classe de contexte d’authentification demandée) et **exact** (exacte) comme méthode de comparaison ( **Comparison Method** ).

    h. Définissez la validité des métadonnées ( **Metadata Validity** ) en nombre de jours.
    
    i. Cliquez sur **Enregistrer les modifications**.

### <a name="create-pulse-secure-pcs-test-user"></a>Créer un utilisateur de test Pulse Secure PCS

Dans cette section, vous créez un utilisateur appelé Britta Simon dans Pulse Secure PCS. Collaborez avec l’[équipe du support technique Pulse Secure PCS](mailto:support@pulsesecure.net) pour ajouter des utilisateurs dans la plateforme Pulse Secure PCS. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

1. Cliquez sur **Tester cette application** dans le portail Azure. Cette opération redirige vers l’URL de connexion Pulse Secure PCS où vous pouvez lancer le flux de connexion. 

2. Accédez directement à l’URL de connexion Pulse Secure PCS pour lancer le flux de connexion à partir de là.

3. Vous pouvez utiliser le volet d’accès Microsoft. Le fait de cliquer sur la vignette Pulse Secure PCS dans le volet d’accès vous redirige vers l’URL de connexion Pulse Secure PCS. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Pulse Secure PCS, vous pouvez appliquer le contrôle de session qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).