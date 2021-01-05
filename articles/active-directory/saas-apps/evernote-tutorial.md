---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Evernote | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Evernote.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/17/2019
ms.author: jeedes
ms.openlocfilehash: 86a314cd5255c06a70d0f9b28d06e3ac4156fdb6
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92453826"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-evernote"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Evernote

Dans ce tutoriel, vous allez apprendre à intégrer Evernote à Azure Active Directory (Azure AD). Quand vous intégrez Evernote à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Evernote.
* Permettre à vos utilisateurs de se connecter automatiquement à Evernote avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Evernote pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Evernote prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité**

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-evernote-from-the-gallery"></a>Ajout d’Evernote à partir de la galerie

Pour configurer l’intégration d’Evernote à Azure AD, vous devez, à partir de la galerie, ajouter Evernote à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Evernote** dans la zone de recherche.
1. Sélectionnez **Evernote** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-evernote"></a>Configurer et tester l’authentification unique Azure AD pour Evernote

Configurez et testez l’authentification unique Azure AD avec Evernote pour un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Evernote associé.

Pour configurer et tester l’authentification unique Azure AD avec Evernote, effectuez les module suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Evernote](#configure-evernote-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Evernote](#create-evernote-test-user)** pour avoir un équivalent de B.Simon dans Evernote lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [Portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Evernote** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    Dans la zone de texte **Identificateur** , tapez une URL :  `https://www.evernote.com/saml2`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL : `https://www.evernote.com/Login.action`

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

7. Pour modifier les options **Certificat de signature** , cliquez sur le bouton **Modifier** pour ouvrir la boîte de dialogue **Certificat de signature SAML** .

    ![Capture d’écran montrant la boîte de dialogue « S A M L Signing Certificate » avec le bouton « Edit » sélectionné.](common/edit-certificate.png) 

    ![image](./media/evernote-tutorial/samlassertion.png)

    a. Sélectionnez l’option **Signer la réponse et l’assertion SAML** dans **Option de signature** .

    b. Cliquez sur **Enregistrer** .

1. Dans la section **Configurer Evernote** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Evernote.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Evernote** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-evernote-sso"></a>Configurer l’authentification unique Evernote

1. Pour automatiser la configuration dans Evernote, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Evernote** pour être dirigé vers l’application Evernote. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Evernote. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 6.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement Evernote, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Evernote en tant qu’administrateur et effectuez les étapes suivantes :

4. Accédez à la **Console d’administration**

    ![Admin-Console](./media/evernote-tutorial/tutorial_evernote_adminconsole.png)

5. À partir de la **Console d’administration** , accédez à **Sécurité** et sélectionnez **Authentification unique**

    ![SSO-Setting](./media/evernote-tutorial/tutorial_evernote_sso.png)

6. Configurez les valeurs suivantes :

    ![Certificate-Setting](./media/evernote-tutorial/tutorial_evernote_certx.png)
    
    a.  **Activer l’authentification unique :** L’authentification unique est activée par défaut (cliquez sur **Désactiver l’authentification unique** pour supprimer l’exigence SSO)

    b. Collez la valeur **URL de connexion** copiée à partir du portail Azure dans la zone de texte **SAML HTTP Request URL** (URL de requête HTTP SAML).

    c. Ouvrez le certificat téléchargé à partir d’Azure AD dans un bloc-notes et copiez-en le contenu, de « BEGIN CERTIFICATE » jusqu’à « END CERTIFICATE ». Ensuite, collez-le dans la zone de texte **X.509 Certificate** (Certificat X.509). 

    d.Cliquez sur **Enregistrer les modifications**

### <a name="create-evernote-test-user"></a>Créer un utilisateur de test Evernote

Pour se connecter à Evernote, les utilisateurs d’Azure AD doivent être provisionnés dans Evernote.  
Dans le cas d’Evernote, l’approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise Evernote en tant qu’administrateur.

2. Cliquez sur la **Console d’administration** .

    ![Admin-Console](./media/evernote-tutorial/tutorial_evernote_adminconsole.png)

3. Dans la **Console d’administration** , accédez à **Ajouter des utilisateurs** .

    ![Capture d’écran montrant le menu « Users » avec l’option « Add Users » sélectionnée.](./media/evernote-tutorial/create_aaduser_0001.png)

4. Sélectionnez **Ajouter des membres de l’équipe** dans la zone de texte **E-mail** , saisissez l’adresse de messagerie du compte d’utilisateur et cliquez sur **Inviter.**

    ![Add-testUser](./media/evernote-tutorial/create_aaduser_0002.png)
    
5. Une fois l’invitation envoyée, le titulaire du compte Azure Active Directory recevra un courrier électronique pour accepter l’invitation.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Evernote dans le volet d’accès, vous devez être connecté automatiquement à l’application Evernote pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Evernote avec Azure AD](https://aad.portal.azure.com/)