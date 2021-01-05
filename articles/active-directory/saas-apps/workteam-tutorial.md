---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Workteam | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Workteam.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/19/2019
ms.author: jeedes
ms.openlocfilehash: 077af1e8add63aac9f3d866c5d665ad4a47b133f
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92520045"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-workteam"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Workteam

Dans ce tutoriel, vous allez apprendre à intégrer Workteam à Azure Active Directory (Azure AD). Quand vous intégrez Workteam à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Workteam.
* Permettre à vos utilisateurs de se connecter automatiquement à Workteam avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Workteam pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Workteam prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité** .

## <a name="adding-workteam-from-the-gallery"></a>Ajout de Workteam à partir de la galerie

Pour configurer l’intégration de Workteam à Azure AD, vous devez ajouter Workteam à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Workteam** dans la zone de recherche.
1. Sélectionnez **Workteam** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-workteam"></a>Configurer et tester l’authentification unique Azure AD pour Workteam

Configurez et testez l’authentification unique Azure AD avec Workteam pour un utilisateur de test nommé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Workteam associé.

Pour configurer et tester l’authentification unique Azure AD avec Workteam, effectuez les modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Workteam](#configure-workteam-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Workteam](#create-workteam-test-user)** pour avoir un équivalent de B.Simon dans Workteam lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [Portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Workteam** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , l’application est préconfigurée en mode Lancement par le **fournisseur d’identité** et les URL nécessaires sont déjà préremplies avec Azure. L’utilisateur doit enregistrer la configuration en cliquant sur le bouton **Enregistrer** .

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL : `https://app.workte.am`

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Workteam** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Workteam.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Workteam** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="configure-workteam-sso"></a>Configurer l’authentification unique Workteam

1. Pour automatiser la configuration dans Workteam, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Workteam** pour être redirigé vers l’application Workteam. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Workteam. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 6.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement Workteam, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Workteam en tant qu’administrateur et effectuez les étapes suivantes :

4. Dans l’angle supérieur droit, cliquez sur **l’icône de profil** , puis cliquez sur **Organization settings** (Paramètres de l’organisation). 

    ![Paramètres de Workteam](./media/workteam-tutorial/tutorial_workteam_settings.png)

5. Dans la section **AUTHENTICATION** (Authentification), cliquez sur **l’icône Paramètres** .

     ![Workteam azure](./media/workteam-tutorial/tutorial_workteam_azure.png)

6. Sur la page **SAML Settings** , procédez comme suit :

     ![Workteam saml](./media/workteam-tutorial/tutorial_workteam_saml.png)

    a. Sous **SAML IdP** (Fournisseur d’identité SAML), sélectionnez **AD Azure** .

    b. Dans la zone de texte **URL du service d’authentification unique SAML** , collez la valeur de l’ **URL de connexion** que vous avez copiée à partir du Portail Azure.

    c. Dans la zone de texte **ID d’entité SAML** , collez l’ **Identificateur Azure AD** que vous avez copié dans le Portail Azure.

    d. Dans le Bloc-notes, ouvrez le **certificat codé en base 64** téléchargé à partir du Portail Azure, copiez son contenu, puis collez-le dans la zone **SAML Signing Certificate (Base64)** (Certificat de signature SAML (Base64)).

    e. Cliquez sur **OK** .

### <a name="create-workteam-test-user"></a>Créer un utilisateur de test Workteam

Pour se connecter à Workteam, les utilisateurs d’Azure AD doivent être approvisionnés dans Workteam. Dans Workteam, l’approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à Workteam en tant qu’administrateur de la sécurité.

2. En haut et au milieu de la page **Organization settings** (Paramètres de l’organisation), cliquez sur **USERS** (Utilisateurs), puis cliquez sur **NEW USER** (Nouvel utilisateur).

    ![Utilisateur Workteam](./media/workteam-tutorial/tutorial_workteam_user.png)

3. Dans la page **New employee** (Nouvel employé), effectuez les étapes suivantes :

    ![Nouvel utilisateur Workteam](./media/workteam-tutorial/tutorial_workteam_newuser.png)

    a. Dans la zone de texte **Name** (Name), entrez le nom d’un utilisateur, par exemple **B.Simon** .

    b. Dans la zone de texte **E-mail** , entrez l’adresse e-mail de l’utilisateur, comme `B.Simon\@contoso.com`.

    c. Cliquez sur **OK** .

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Workteam dans le panneau d’accès doit vous connecter automatiquement à l’application Workteam pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Workteam avec Azure AD](https://aad.portal.azure.com/)