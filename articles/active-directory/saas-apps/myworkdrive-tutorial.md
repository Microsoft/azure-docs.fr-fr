---
title: 'Tutoriel : Intégration d’Azure Active Directory à MyWorkDrive | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et MyWorkDrive.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/27/2019
ms.author: jeedes
ms.openlocfilehash: b639ee45bfa6356054d93a1b6fcc8a7959173acd
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92516084"
---
# <a name="tutorial-integrate-myworkdrive-with-azure-active-directory"></a>Tutoriel : Intégrer MyWorkDrive à Azure Active Directory

Dans ce tutoriel, vous allez apprendre à intégrer MyWorkDrive à Azure Active Directory (Azure AD). Quand vous intégrez MyWorkDrive à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à MyWorkDrive.
* Permettre à vos utilisateurs de se connecter automatiquement à MyWorkDrive avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’en avez pas, vous pouvez obtenir un essai gratuit d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement MyWorkDrive pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. MyWorkDrive prend en charge l’authentification unique démarrée par **le fournisseur de services** et le **fournisseur d’identité**

## <a name="adding-myworkdrive-from-the-gallery"></a>Ajout de MyWorkDrive à partir de la galerie

Pour configurer l’intégration de MyWorkDrive à Azure AD, vous devez ajouter MyWorkDrive à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **MyWorkDrive** dans la zone de recherche.
1. Sélectionnez **MyWorkDrive** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec MyWorkDrive à l’aide d’un utilisateur de test appelé **Britta Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur MyWorkDrive associé.

Pour configurer et tester l’authentification unique Azure AD avec MyWorkDrive, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique MyWorkDrive](#configure-myworkdrive-sso)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test MyWorkDrive](#create-myworkdrive-test-user)** pour avoir un équivalent de Britta Simon dans MyWorkDrive lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **MyWorkDrive** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la page **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , entrez les valeurs pour le champ suivant :

    Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<SERVER.DOMAIN.COM>/SAML/AssertionConsumerService.aspx`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<SERVER.DOMAIN.COM>/Account/Login-saml`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de réponse et l’URL de connexion réelles. Entrez le nom d’hôte du serveur MyWorkDrive de votre propre société : p. ex.
    > 
    > URL de réponse : `https://yourserver.yourdomain.com/SAML/AssertionConsumerService.aspx`
    > 
    > URL de connexion :`https://yourserver.yourdomain.com/Account/Login-saml`
    > 
    > Contactez l’[équipe de support MyWorkDrive](mailto:support@myworkdrive.com) si vous avez des doutes sur la façon de configurer vos propres nom d’hôte et certificat TLS/SSL pour ces valeurs.

1. Dans la page **Configurer l’authentification unique avec SAML** , accédez à la section **Certificat de signature SAML** et cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** dans le Presse-papiers.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

### <a name="configure-myworkdrive-sso"></a>Configurer l’authentification unique MyWorkDrive

1. Pour automatiser la configuration dans MyWorkDrive, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

1. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer MyWorkDrive** pour être dirigé vers l’application MyWorkDrive. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à MyWorkDrive. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 et 4.

    ![Configuration](common/setup-sso.png)

1. Si vous souhaitez configurer MyWorkDrive manuellement, ouvrez une autre fenêtre de navigateur web et connectez-vous à MyWorkDrive en tant qu’administrateur de la sécurité.

1. Sur le serveur MyWorkDrive, accédez au panneau d'administration, cliquez sur **ENTREPRISE** et procédez comme suit :

    ![Administrateur](./media/myworkdrive-tutorial/tutorial_myworkdrive_admin.png)

    a. Activez **SAML/ADFS SSO** .

    b. Sélectionnez **SAML - Azure AD**

    c. Dans la zone de texte **URL des métadonnées de fédération d'application Azure** , collez la valeur de l' **URL des métadonnées de fédération d'application** que vous avez copiée à partir du portail Azure.

    d. Cliquez sur **Enregistrer** .

    > [!NOTE]
    > Pour plus d'informations, consultez l'[article de support MyWorkDrive Azure AD](https://www.myworkdrive.com/support/saml-single-sign-on-azure-ad/).

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à MyWorkDrive.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **MyWorkDrive** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-myworkdrive-test-user"></a>Créer un utilisateur de test MyWorkDrive

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans MyWorkDrive. Collaborez avec [l’équipe du support technique de MyWorkDrive](mailto:support@myworkdrive.com) pour ajouter des utilisateurs dans la plateforme MyWorkDrive. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Quand vous sélectionnez la vignette MyWorkDrive dans le volet d’accès, vous devez être connecté automatiquement à l’application MyWorkDrive pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)