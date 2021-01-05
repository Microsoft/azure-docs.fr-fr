---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Teamphoria | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Teamphoria.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/09/2019
ms.author: jeedes
ms.openlocfilehash: a8cf58f49cba1bcc77c96a0da2061a037bb62bf2
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92504743"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-teamphoria"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Teamphoria

Dans ce tutoriel, vous allez apprendre à intégrer Teamphoria à Azure Active Directory (Azure AD). Quand vous intégrez Teamphoria à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Teamphoria.
* Permettre à vos utilisateurs de se connecter automatiquement à Teamphoria avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Teamphoria pour lequel l’authentification unique (SSO) est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Teamphoria prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-teamphoria-from-the-gallery"></a>Ajout de Teamphoria à partir de la galerie

Pour configurer l’intégration de Teamphoria à Azure AD, vous devez ajouter Teamphoria à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **Teamphoria** dans la zone de recherche.
1. Sélectionnez **Teamphoria** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-teamphoria"></a>Configurer et tester l’authentification unique Azure AD pour Teamphoria

Configurez et testez l’authentification unique Azure AD avec Teamphoria pour un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Teamphoria associé.

Pour configurer et tester l’authentification unique Azure AD avec Teamphoria, effectuez les modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Teamphoria](#configure-teamphoria-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Teamphoria](#create-teamphoria-test-user)** pour avoir un équivalent de B.Simon dans Teamphoria qui soit lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Teamphoria** , recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<sub-domain>.teamphoria.com/login`

    > [!NOTE]
    > Cette valeur n’est pas la valeur réelle. Mettez à jour la valeur avec l’URL de connexion réelle. Contactez l’[équipe de support technique Teamphoria](https://www.teamphoria.com/) pour obtenir la valeur. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Teamphoria** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Teamphoria.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Teamphoria**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

## <a name="configure-teamphoria-sso"></a>Configurer l’authentification unique Teamphoria

1. Pour automatiser la configuration dans Teamphoria, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension**.

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après avoir ajouté l’extension au navigateur, cliquez sur **Configurer Teamphoria** pour être dirigé vers l’application Teamphoria. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Teamphoria. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 6.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement Teamphoria, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Teamphoria en tant qu’administrateur et effectuez les étapes suivantes :

4. Accédez à l’option **ADMIN SETTINGS** (Paramètres d’administration) dans la barre d’outils gauche puis, sous l’onglet Configure (Configurer), cliquez sur **SINGLE SIGN-ON** (Authentification unique) pour ouvrir la fenêtre de configuration de l’authentification unique (SSO).

    ![Capture d’écran montrant ADMIN SETTINGS, où vous pouvez sélectionner SINGLE SIGN-ON](./media/teamphoria-tutorial/admin_sso_configure.png)

5. Cliquez sur l’option **ADD NEW IDENTITY PROVIDER** (Ajouter un nouveau fournisseur d’identité) dans le coin supérieur droit pour ouvrir le formulaire permettant d’ajouter les paramètres pour l’authentification unique.

    ![Capture d’écran montrant où vous pouvez sélectionner ADD NEW IDENTITY PROVIDER](./media/teamphoria-tutorial/add_new_identity_provider.png)

6. Entrez les informations dans les champs comme indiqué ci-dessous :

    ![Capture d’écran montrant la page dans laquelle vous pouvez indiquer les valeurs décrites.](./media/teamphoria-tutorial/Teamphoria_sso_save.png)

    a. **NOM D'AFFICHAGE**  : entrez le nom d'affichage du plug-in sur la page d'administration.

    b. **NOM DU BOUTON**  : nom de l'onglet qui apparaîtra sur la page de connexion via l'authentification unique.

    c. **CERTIFICAT**  : dans le Bloc-notes, ouvrez le certificat téléchargé précédemment à partir du portail Azure, copiez le contenu de celui-ci, puis collez-le dans ce champ.

    d. **POINT D'ENTRÉE**  : Collez l’ **URL de connexion** copiée précédemment à partir du portail Azure.

    e. Définissez l’option sur **ON** , puis cliquez sur **SAVE** (Enregistrer).

### <a name="create-teamphoria-test-user"></a>Créer un utilisateur de test Teamphoria

Pour se connecter à Teamphoria, les utilisateurs d’Azure AD doivent être provisionnés dans Teamphoria. Dans le cas de Teamphoria, l’approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous au site d’entreprise Teamphoria en tant qu’administrateur.

1. Cliquez sur les paramètres **ADMIN** dans la barre d’outils gauche puis, sous l’onglet **MANAGE** (Gérer), cliquez sur **USERS** (Utilisateurs) afin d’ouvrir la page d’administration pour les utilisateurs.

    ![Ajouter un employé](./media/teamphoria-tutorial/admin_manage_users.png)

1. Cliquez sur l’option **MANUAL INVITE** (Inviter manuellement).

    ![Capture d’écran montrant l’option MANUAL INVITE](./media/teamphoria-tutorial/admin_manage_add_users.png)

1. Dans cette page, effectuez l’action suivante.

    ![Capture d’écran montrant la page MANUAL USER INVITE, dans laquelle vous pouvez entrer le nom et l’adresse e-mail](./media/teamphoria-tutorial/manual_user_invite.png)

    a. Dans la zone de texte **EMAIL ADDRESS** , entrez l’ **adresse e-mail** de l’utilisateur, par exemple B.Simon.

    b. Dans la zone de texte **FIRST NAME** , entrez le prénom de l’utilisateur, par exemple **B**.

    c. Dans la zone de texte **LAST NAME** , tapez le nom de l’utilisateur, par exemple **Simon**.

    d. Cliquez sur **INVITE 1 USER** (Inviter l’utilisateur 1). L’utilisateur doit accepter l’invitation pour être créé dans le système.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Teamphoria dans le volet d’accès, vous devez être connecté automatiquement à l’application Teamphoria pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Teamphoria avec Azure AD](https://aad.portal.azure.com/)