---
title: 'Tutoriel : Intégration d’Azure Active Directory à Syncplicity | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Syncplicity.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/10/2019
ms.author: jeedes
ms.openlocfilehash: 792a5849c6afcdcbc32dc7de4d0d0dea23180a76
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92517104"
---
# <a name="tutorial-integrate-syncplicity-with-azure-active-directory"></a>Tutoriel : Intégrer Syncplicity à Azure Active Directory

Dans ce didacticiel, vous allez apprendre à intégrer Syncplicity à Azure Active Directory (Azure AD). Quand vous intégrez Syncplicity à Azure AD, vous pouvez :

* Dans Azure AD, contrôlez qui a accès à Syncplicity.
* Permettre à vos utilisateurs de se connecter automatiquement à Syncplicity avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’en avez pas, vous pouvez obtenir un essai gratuit d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Syncplicity pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. Syncplicity prend en charge l’authentification unique lancée par le **fournisseur de services** .

## <a name="adding-syncplicity-from-the-gallery"></a>Ajout de Syncplicity à partir de la galerie

Pour configurer l’intégration de Syncplicity à Azure AD, vous devez ajouter Syncplicity à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Syncplicity** dans la zone de recherche.
1. Sélectionnez **Syncplicity** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec Syncplicity pour un utilisateur de test nommé **B. Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Syncplicity associé.

Pour configurer et tester l’authentification unique Azure AD avec Syncplicity, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Syncplicity](#configure-syncplicity-sso)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Syncplicity](#create-syncplicity-test-user)** pour avoir un équivalent de B. Simon dans Syncplicity lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Syncplicity** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la boîte de dialogue **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<companyname>.syncplicity.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<companyname>.syncplicity.com/sp`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez [l’équipe de support technique Syncplicity](https://www.syncplicity.com/contact-us). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (Base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

   ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Syncplicity** , copiez la ou les URL appropriées en fonction de vos besoins.

   ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="configure-syncplicity-sso"></a>Configurer l’authentification unique Syncplicity

1. Connectez-vous à votre locataire **Syncplicity** .

1. Dans le menu en haut, cliquez sur **admin** , sélectionnez **settings** , puis cliquez sur **Custom domain and single sign-on** .

    ![Syncplicity](./media/syncplicity-tutorial/ic769545.png "Syncplicity")

1. Dans la boîte de dialogue **Single Sign on (SSO)** , procédez comme suit :

    ![Authentification unique \(SSO\)](./media/syncplicity-tutorial/ic769550.png "Single Sign-On \\\(SSO\\\)")

    a. Dans la zone de texte **Custom Domain** , entrez le nom de votre domaine.
  
    b. Sélectionnez **Enabled** dans **Single Sign-On Status** .

    c. Dans la zone de texte **ID d’entité** , collez la valeur **Identificateur (ID d’entité)** que vous avez copiée dans la section **Configuration SAML de base** du portail Azure.

    d. Dans la zone de texte **Sign-in page URL** (URL de la page de connexion), collez l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    e. Dans la zone de texte **Logout page URL** (URL de la page de déconnexion), collez l’ **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    f. Dans **Certificat du fournisseur d’identité** , cliquez sur **Choisir un fichier** , puis chargez le certificat que vous avez téléchargé à partir du portail Azure.

    g. Cliquez sur **ENREGISTRER LES MODIFICATIONS** .

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

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Syncplicity.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Syncplicity** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-syncplicity-test-user"></a>Créer un utilisateur de test Syncplicity

Pour que les utilisateurs Azure AD puissent se connecter, ils doivent être provisionnés pour l’application Syncplicity. Cette section explique comment créer des comptes d’utilisateurs Azure AD dans Syncplicity.

**Pour approvisionner un compte d’utilisateur dans Syncplicity, procédez comme suit :**

1. Connectez-vous à votre locataire **Syncplicity** (par exemple : `https://company.Syncplicity.com`).

1. Cliquez sur **admin** et sélectionnez **user accounts** (comptes d’utilisateur), puis cliquez sur **ADD A USER** (Ajouter un utilisateur).

    ![Gérer les utilisateurs](./media/syncplicity-tutorial/ic769764.png "Manage Users")

1. Entrez l’ **adresse e-mail** d’un compte Azure AD que vous souhaitez provisionner, sélectionnez **User** (Utilisateur) pour **Role** , puis cliquez sur **NEXT** (SUIVANT).

    ![Account Information](./media/syncplicity-tutorial/ic769765.png "Informations sur le compte") (Informations sur le compte)

    > [!NOTE]
    > Le détenteur du compte Azure AD reçoit un e-mail contenant un lien pour confirmer et activer le compte.

1. Sélectionnez un groupe dans votre société dont votre nouvel utilisateur doit devenir membre, puis cliquez sur **SUIVANT** .

    ![Group Membership](./media/syncplicity-tutorial/ic769772.png "Appartenance au groupe") (Appartenance aux groupes)

    > [!NOTE]
    > Si aucun groupe n’est répertorié, cliquez sur **SUIVANT** .

1. Sélectionnez les dossiers que vous souhaitez placer sous le contrôle de Syncplicity sur l’ordinateur de l’utilisateur, puis cliquez sur **SUIVANT** .

    ![Syncplicity Folders](./media/syncplicity-tutorial/ic769773.png "Dossiers Syncplicity") (Dossiers Syncplicity)

> [!NOTE]
> Vous pouvez utiliser tout autre outil ou n’importe quelle API de création de compte d’utilisateur fournis par Syncplicity pour provisionner des comptes d’utilisateurs Azure AD.

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Le fait de sélectionner la vignette Syncplicity dans le panneau d’accès doit vous connecter automatiquement à l’application Syncplicity pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)