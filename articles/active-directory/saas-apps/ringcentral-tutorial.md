---
title: 'Tutoriel : Intégration d’Azure Active Directory à RingCentral | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et RingCentral.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/24/2019
ms.author: jeedes
ms.openlocfilehash: a64abfbdb33a30b41853023af14f54bb88e42c8f
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92520708"
---
# <a name="tutorial-integrate-ringcentral-with-azure-active-directory"></a>Tutoriel : Intégrer RingCentral à Azure Active Directory

Dans ce tutoriel, vous allez découvrir comment intégrer RingCentral à Azure AD (Azure Active Directory). Quand vous intégrez RingCentral à Azure AD, vous pouvez :

* Contrôler qui a accès à RingCentral dans Azure AD.
* Permettre aux utilisateurs de se connecter automatiquement à RingCentral avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement RingCentral pour lequel l’authentification unique est activée

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* RingCentral prend en charge l’authentification unique lancée par le **fournisseur d’identité**

## <a name="adding-ringcentral-from-the-gallery"></a>Ajout de RingCentral depuis la galerie

Pour configurer l’intégration de RingCentral à Azure AD, vous devez ajouter RingCentral à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **RingCentral** dans la zone de recherche.
1. Sélectionnez **RingCentral** dans le panneau de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec RingCentral à l’aide d’un utilisateur de test appelé **Britta Simon** . Pour que l’authentification unique fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur RingCentral associé.

Pour configurer et tester l’authentification unique Azure AD avec RingCentral, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique RingCentral](#configure-ringcentral-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test RingCentral](#create-ringcentral-test-user)** pour avoir un équivalent de B.Simon dans RingCentral, lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **RingCentral** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous disposez d’un **fichier de métadonnées du fournisseur de services** , suivez les étapes ci-dessous :

    1. Cliquez sur **Charger un fichier de métadonnées** .
    1. Cliquez sur le **logo du dossier** pour sélectionner le fichier de métadonnées, puis cliquez sur **Charger** .
    1. Une fois le fichier de métadonnées chargé, les valeurs **Identificateur** et **URL de réponse** sont automatiquement renseignées dans la section **Configuration SAML de base** .

    > [!Note]
    > Sur la page de configuration de l’authentification unique de RingCentral, vous obtenez le **fichier de métadonnées du fournisseur de services** , qui sera expliqué plus tard dans le didacticiel.

1. Si vous n’avez pas de **fichier de métadonnées du fournisseur de services** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL :
  
    | Identificateur |
    |--|
    |  `https://sso.ringcentral.com` |
    | `https://ssoeuro.ringcentral.com` |

    b. Dans la zone de texte **URL de réponse** , tapez l’URL au format suivant :

    | URL de réponse |
    |--|
    | `https://sso.ringcentral.com/sp/ACS.saml2` |
    | `https://ssoeuro.ringcentral.com/sp/ACS.saml2` |

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé Britta Simon dans le Portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `Britta Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `BrittaSimon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à RingCentral.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **RingCentral** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-ringcentral-sso"></a>Configurer l’authentification unique RingCentral

1. Pour automatiser la configuration dans RingCentral, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

1. Après avoir ajouté l’extension au navigateur, cliquez sur **Configurer RingCentral** pour être dirigé vers l’application RingCentral. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à RingCentral. Cette extension de navigateur configure automatiquement l’application pour vous et automatise les étapes 3 à 7.

    ![Configuration](common/setup-sso.png)

1. Si vous souhaitez configurer manuellement RingCentral, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise RingCentral en tant qu’administrateur et effectuez les étapes suivantes :

1. En haut de la page, cliquez sur **Outils** .

    ![Capture d’écran montrant les outils sélectionnés sur le site d’entreprise RingCentral.](./media/ringcentral-tutorial/ringcentral1.png)

1. Accédez à **Authentification unique** .

    ![Capture d’écran montrant l’élément Single Sign-on sélectionné dans le menu Tools.](./media/ringcentral-tutorial/ringcentral2.png)

1. Sur la page **Authentification unique** , dans la section **Configuration SSO** , cliquez sur **Modifier** dans l’ **étape 1** , puis procédez comme suit :

    ![Capture d’écran montrant la page de configuration de l’authentification unique dans laquelle vous pouvez sélectionner Edit.](./media/ringcentral-tutorial/ringcentral3.png)

1. Sur la page **Configurer l’authentification unique** , effectuez les étapes suivantes :

    ![Capture d’écran montrant la page Set up Single Sign-On, dans laquelle vous pouvez charger les métadonnées du fournisseur d’identité.](./media/ringcentral-tutorial/ringcentral4.png)

    a. Cliquez sur **Parcourir** pour charger le fichier de métadonnées que vous avez téléchargé à partir du portail Azure.

    b. Après avoir chargé les métadonnées, les valeurs sont automatiquement renseignées dans la section **SSO General Information** (Informations générales sur l’authentification unique).

    c. Dans la section **Mappage d’attributs** , définissez **Map Email Attribute to** (Mapper l’attribut d’e-mail à) sur `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`

    d. Cliquez sur **Enregistrer** .

    e. Dans l’ **étape 2** , cliquez sur **Télécharger** pour télécharger le **fichier de métadonnées du fournisseur de services** . Ensuite, chargez-le dans la section **Configuration SAML de base** afin de renseigner automatiquement les valeurs **Identificateur** et **URL de réponse** dans le portail Azure.

    ![Capture d’écran montrant la page de configuration de l’authentification unique, dans laquelle vous pouvez sélectionner Download.](./media/ringcentral-tutorial/ringcentral6.png) 

    f. Sur la même page, accédez à la section **Activer l’authentification unique** et procédez comme suit :

    ![Capture d’écran présentant la section d’activation de l’authentification unique, dans laquelle vous pouvez terminer la configuration.](./media/ringcentral-tutorial/ringcentral5.png)

    * Sélectionnez **Activer le service d’authentification unique** .

    * Sélectionnez **Allow users to log in with SSO or RingCentral credential** (Autoriser les utilisateurs à se connecter avec des informations d’identification de l’authentification unique ou RingCentral).

    * Cliquez sur **Enregistrer** .

### <a name="create-ringcentral-test-user"></a>Créer un utilisateur de test RingCentral

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans RingCentral. Collaborez avec l’[équipe de support technique RingCentral](https://success.ringcentral.com/RCContactSupp) pour ajouter les utilisateurs dans la plateforme RingCentral. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Quand vous sélectionnez la vignette RingCentral dans le volet d’accès, vous devez être connecté automatiquement à l’application RingCentral pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer RingCentral avec Azure AD](https://aad.portal.azure.com/)