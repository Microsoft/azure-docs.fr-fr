---
title: 'Didacticiel : Intégration d’Azure Active Directory à Palo Alto Networks - Aperture | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Palo Alto Networks - Aperture.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/10/2020
ms.author: jeedes
ms.openlocfilehash: a70d075ae25b0059e4e1ab75c44907f584e15350
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92512816"
---
# <a name="tutorial-azure-active-directory-integration-with-palo-alto-networks---aperture"></a>Didacticiel : Intégration d’Azure Active Directory à Palo Alto Networks - Aperture

Dans ce didacticiel, vous découvrez comment intégrer Palo Alto Networks - Aperture avec Azure Active Directory (Azure AD).
L’intégration de Palo Alto Networks - Aperture avec Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Palo Alto Networks - Aperture.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Palo Alto Networks - Aperture (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD avec Palo Alto Networks - Aperture, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Palo Alto Networks - Aperture pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Palo Alto Networks - Aperture prend en charge l’authentification unique lancée par le **fournisseur de services** et le **fournisseur d’identité** .

## <a name="adding-palo-alto-networks---aperture-from-the-gallery"></a>Ajout de Palo Alto Networks - Aperture depuis la galerie

Pour configurer l’intégration de Palo Alto Networks - Aperture à Azure AD, vous devez ajouter Palo Alto Networks - Aperture à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , entrez **Palo Alto Networks - Aperture** dans la zone de recherche.
1. Sélectionnez **Palo Alto Networks - Aperture** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous configurez et vous testez l’authentification unique Azure AD avec Palo Alto Networks - Aperture sur un utilisateur de test nommé **B.Simon** .
Pour que l’authentification unique fonctionne, une relation doit être établie entre un utilisateur Azure AD et l’utilisateur Palo Alto Networks - Aperture associé.

Pour configurer et tester l’authentification unique Azure AD avec Palo Alto Networks - Aperture, procédez comme suit :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
2. **[Configurer l’authentification unique Palo Alto Networks - Aperture](#configure-palo-alto-networks---aperture-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Palo Alto Networks - Aperture](#create-palo-alto-networks---aperture-test-user)** pour obtenir un équivalent de Britta Simon dans Palo Alto Networks - Aperture lié à la représentation Azure AD associée.
3. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, dans la page d’intégration de l’application **Palo Alto Networks - Aperture** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , effectuez les étapes suivantes :

    ![Capture d’écran montrant la « Configuration SAML de base » avec les zones de texte « Identificateur » et « URL de réponse » mises en évidence, et l’action « Enregistrer » sélectionnée.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<subdomain>.aperture.paloaltonetworks.com/d/users/saml/metadata`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<subdomain>.aperture.paloaltonetworks.com/d/users/saml/auth`

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Palo Alto Networks - Aperture Domain and URLs (Domaine et URL - Palo Alto Networks - Aperture)](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<subdomain>.aperture.paloaltonetworks.com/d/users/saml/sign_in`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Contactez [l’équipe de support client de Palo Alto Networks - Aperture](https://live.paloaltonetworks.com/t5/custom/page/page-id/Support) pour obtenir ces valeurs. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

6. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

7. Dans la section **Configurer Palo Alto Networks - Aperture** , copiez les URL appropriées selon vos besoins.

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

Dans cette section, vous autorisez B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Palo Alto Networks - Aperture.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Palo Alto Networks - Aperture** (Palo Alto Networks - Aperture).
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .
1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .
1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-palo-alto-networks---aperture-sso"></a>Configurer l’authentification unique Palo Alto Networks - Aperture

1. Dans une autre fenêtre de navigateur web, connectez-vous à Palo Alto Networks - Aperture en tant qu’administrateur.

2. Dans la barre de menus supérieure, cliquez sur **PARAMÈTRES** .

    ![Onglet de paramètres](./media/paloaltonetworks-aperture-tutorial/tutorial_paloaltonetwork_settings.png)

3. Accédez à la section **APPLICATION** et cliquez sur **Authentification** dans le côté gauche du menu.

    ![Onglet d’authentification](./media/paloaltonetworks-aperture-tutorial/tutorial_paloaltonetwork_auth.png)
    
4. Sur la page **Authentification** , procédez comme suit :
    
    ![Onglet d’authentification](./media/paloaltonetworks-aperture-tutorial/tutorial_paloaltonetwork_singlesignon.png)

    a. Cochez l’option **Enable Single Sign-On(Supported SSP Providers are Okta, One login)** (Activer l’authentification unique pour ce réseau (Les fournisseurs SSP pris en charge sont Okta, Onelogin)) dans le champ **Authentification unique** .

    b. Dans la zone de texte **Identity Provider ID** (ID du fournisseur d’identité), collez la valeur de l’ **Identificateur Azure AD** , que vous avez copiée à partir du portail Azure.

    c. Cliquez sur **Choisir un fichier** pour charger le certificat obtenu à partir d’Azure AD dans le champ **Certificat du fournisseur d’identité** .

    d. Dans la zone de texte **Identity Provider SSO URL** (URL d’authentification unique du fournisseur d’identité), collez la valeur de l’ **URL de connexion** que vous avez copiée dans le portail Azure.

    e. Passez en revue les informations des fournisseurs d’identité dans la section **Aperture Info** (Informations Aperture) et téléchargez le certificat à partir du champ **Aperture Key** (Clé Aperture).

    f. Cliquez sur **Enregistrer** .


### <a name="create-palo-alto-networks---aperture-test-user"></a>Créer un utilisateur de test Palo Alto Networks - Aperture

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans Palo Alto Networks - Aperture. Collaborez avec [l’équipe de support client de Palo Alto Networks - Aperture](https://live.paloaltonetworks.com/t5/custom/page/page-id/Support) pour ajouter les utilisateurs à la plateforme Palo Alto Networks - Aperture. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

#### <a name="sp-initiated"></a>Lancée par le fournisseur de services :

* Cliquez sur **Tester cette application** dans le portail Azure. Cette opération redirige vers l’URL de connexion Palo Alto Networks - Aperture où vous pouvez lancer le processus de connexion.  

* Accédez directement à l’URL de connexion Palo Alto Networks - Aperture pour lancer le flux de connexion à partir de là.

#### <a name="idp-initiated"></a>Lancée par le fournisseur d’identité :

* Cliquez sur **Tester cette application** dans le portail Azure. Vous devriez être connecté automatiquement à l’instance Palo Alto Networks - Aperture pour laquelle vous avez configuré l’authentification unique. 

Vous pouvez aussi utiliser le panneau d’accès Microsoft pour tester l’application dans n’importe quel mode. Lorsque vous cliquez sur la vignette Palo Alto Networks - Aperture dans le volet d’accès, s’il est configuré en mode Fournisseur de services, vous êtes redirigé vers la page de connexion de l’application pour lancer le flux de connexion et, s’il est configuré en mode Fournisseur de services, vous êtes automatiquement connecté à l’instance de Palo Alto Networks - Aperture pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Palo Alto Networks - Aperture, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).