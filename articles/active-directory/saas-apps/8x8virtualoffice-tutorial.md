---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à 8x8 | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et 8x8.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/28/2020
ms.author: jeedes
ms.openlocfilehash: 81a7efea268600e661981b35f79149fe814ef084
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96180674"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-8x8"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à 8x8

Dans ce tutoriel, vous allez apprendre à intégrer 8x8 à Azure Active Directory (Azure AD). Quand vous intégrez 8x8 à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à 8x8.
* Permettre à vos utilisateurs de se connecter automatiquement à 8x8 avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement 8x8.

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* 8x8 prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité**.

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-8x8-from-the-gallery"></a>Ajout de 8x8 depuis la galerie

Pour configurer l’intégration de 8x8 à Azure AD, vous devez ajouter 8x8 à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **8x8** dans la zone de recherche.
1. Sélectionnez **8x8** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-8x8"></a>Configurer et tester l’authentification unique Azure AD pour 8x8

Configurez et testez l’authentification unique Azure AD avec 8x8 pour un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans 8x8.

Pour configurer et tester l’authentification unique Azure AD avec 8x8, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique 8x8](#configure-8x8-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test 8x8](#create-8x8-test-user)** pour avoir un équivalent de B.Simon dans 8x8, associé à sa représentation dans Azure AD.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Sur le portail Azure, dans la page d’intégration de l’application **8x8**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, effectuez les étapes suivantes :

    a. Dans la zone de texte **Identificateur**, tapez une URL : `https://sso.8x8.com/saml2`

    b. Dans la zone de texte **URL de réponse**, tapez l’URL : `https://sso.8x8.com/saml2`

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur. Vous utiliserez le certificat plus tard dans le tutoriel dans la section **Configurer l’authentification unique 8x8**.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer 8x8**, copiez la ou les URL et vous utiliserez ces valeurs d’URL plus tard dans le tutoriel.

    ![Copier les URL de configuration](./media/8x8virtualoffice-tutorial/copy-configuration-urls.png)

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à 8x8.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **8x8**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-8x8-sso"></a>Configurer l’authentification unique 8x8

La partie suivante de ce tutoriel dépend du type d’abonnement que vous avez avec 8x8.

* Pour les clients 8x8 Editions et X Series qui utilisent le Gestionnaire de configuration pour l’administration, reportez-vous à [Configurer la Console d’administration 8x8](#configure-8x8-admin-console).
* Pour les clients Virtual Office qui utilisent le responsable de compte pour l’administration, reportez-vous à [Configurer le responsable de compte 8x8](#configure-8x8-account-manager).

### <a name="configure-8x8-admin-console"></a>Configurer la Console d’administration 8x8

1. Pour automatiser la configuration dans 8x8, vous devez installer l’**extension de navigateur de connexion sécurisée à Mes applications** en cliquant sur **Installer l’extension**.

    ![Extension My apps](common/install-myappssecure-extension.png)

1. Une fois l’extension ajoutée au navigateur, le fait de cliquer sur **Configurer 8x8** vous redirige vers l’application 8x8. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à 8x8. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 6.

    ![Configuration](common/setup-sso.png)

1. Si vous voulez configurer 8x8 manuellement, connectez-vous à la [Console d’administration](https://admin.8x8.com/) 8x8 en tant qu’administrateur.

1. Dans la page d’accueil, cliquez sur **Identity Management** (Gestion des identités).

    ![Capture d’écran qui met en évidence la vignette Identity Management.](./media/8x8virtualoffice-tutorial/configure1.png)

1. Activez la case à cocher **Single Sign On (SSO)** (Authentification unique), puis sélectionnez **Microsoft Azure AD**.

    ![Capture d’écran qui met en évidence les options Single Sign on (SSO) et Microsoft Azure AD.](./media/8x8virtualoffice-tutorial/configure2.png)

1. Copiez les trois URL et le certificat de signature à partir de la page **Configurer l’authentification unique avec SAML** dans Azure AD dans la section **Microsoft Azure AD SAML Settings** (Paramètres SAML de Microsoft Azure AD) dans la Console d’administration 8x8.

    ![Console d’administration 8x8](./media/8x8virtualoffice-tutorial/configure3.png)

    a. Copiez l’**URL de connexion** dans **IDP Login URL** (URL de connexion IDP).

    b. Copiez l’**Identificateur Azure AD** dans **IDP Issuer URL/URN** (URL/URN de l’émetteur IDP).

    c. Copiez l’**URL de déconnexion** dans **IDP Logout URL** (URL de déconnexion IDP).

    d. Téléchargez le **certificat (Base64)** et chargez-le dans **Certificate** (Certificat).

    e. Cliquez sur **Enregistrer**.

### <a name="configure-8x8-account-manager"></a>Configurer le responsable de compte 8x8

1. Connectez-vous à votre client 8x8 Virtual Office en tant qu’administrateur.

1. Sélectionnez **Virtual Office Account Mgr** (Gestionnaire de compte Virtual Office) dans le volet Applications.

    ![Capture d’écran qui met en évidence la vignette Virtual Office Account Mgr.](./media/8x8virtualoffice-tutorial/tutorial_8x8virtualoffice_001.png)

1. Sélectionnez le compte **Entreprise** à gérer, puis cliquez sur le bouton **Se connecter**.

    ![Capture d’écran qui met en évidence l’option Business et le bouton Sign In.](./media/8x8virtualoffice-tutorial/tutorial_8x8virtualoffice_002.png)

1. Cliquez sur l’onglet **COMPTES** dans la liste de menu.

    ![Capture d’écran qui met en évidence l’onglet ACCOUNTS dans la liste de menus.](./media/8x8virtualoffice-tutorial/tutorial_8x8virtualoffice_003.png)

1. Cliquez sur **Authentification unique** dans la liste des comptes.

    ![Capture d’écran qui met en évidence l’option Single Sign On.](./media/8x8virtualoffice-tutorial/tutorial_8x8virtualoffice_004.png)

1. Sélectionnez **Authentification unique** sous la section Méthodes d’authentification, puis cliquez sur **SAML**.

    ![Capture d’écran qui met en évidence SAML sous Single Sign On.](./media/8x8virtualoffice-tutorial/tutorial_8x8virtualoffice_005.png)

1. Dans la section **Authentification unique SAML**, procédez comme suit :

    ![Configurer l’authentification côté application](./media/8x8virtualoffice-tutorial/tutorial_8x8virtualoffice_006.png)

    a. Dans la zone de texte **URL de connexion**, collez l’**URL de connexion** que vous avez copiée sur le portail Azure.

    b. Dans la zone de texte **URL de déconnexion**, collez l’**URL de déconnexion** que vous avez copiée sur le portail Azure.

    c. Dans la zone de texte **URL de l’émetteur**, collez l’**Identificateur Azure AD** que vous avez copié sur le portail Azure.

    d. Cliquez sur le bouton **Parcourir** pour charger le certificat que vous avez téléchargé à partir du portail Azure.

    e. Cliquez sur le bouton **Enregistrer** .

### <a name="create-8x8-test-user"></a>Créer un utilisateur de test 8x8

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans 8x8. Travaillez en collaboration avec l’[équipe du support technique 8x8](https://www.8x8.com/about-us/contact-us) pour ajouter les utilisateurs dans la plateforme 8x8. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

#### <a name="sp-initiated"></a>Lancée par le fournisseur de services :

* Cliquez sur **Tester cette application** dans le portail Azure. Cette opération redirige vers l’URL de connexion 8x8, où vous pouvez lancer le processus de connexion.  

* Accédez directement à l’URL de connexion 8x8 pour lancer le processus de connexion.

#### <a name="idp-initiated"></a>Lancée par le fournisseur d’identité :

* Cliquez sur **Tester cette application** dans le portail Azure : vous devez être connecté automatiquement à l’instance de 8x8 pour laquelle vous avez configuré l’authentification unique. 

Vous pouvez aussi utiliser le panneau d’accès Microsoft pour tester l’application dans n’importe quel mode. Si, quand vous cliquez sur la vignette 8x8 dans le volet d’accès, le mode Fournisseur de services est configuré, vous êtes redirigé vers la page de connexion de l’application pour lancer le processus de connexion ; s’il s’agit du mode Fournisseur d’identité, vous êtes automatiquement connecté à l’instance de 8x8 pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez configuré 8x8, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).