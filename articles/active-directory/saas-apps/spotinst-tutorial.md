---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Spotinst | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Spotinst.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/03/2020
ms.author: jeedes
ms.openlocfilehash: d9c9ac87e655cf9178d58f04b502f7e781d1fef7
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96181405"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-spotinst"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Spotinst

Dans ce tutoriel, vous allez apprendre à intégrer Spotinst à Azure Active Directory (Azure AD). Quand vous intégrez Spotinst à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Spotinst.
* Permettre à vos utilisateurs de se connecter automatiquement à Spotinst avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Spotinst pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Spotinst prend en charge l’authentification unique initiée par **le fournisseur de services et le fournisseur d’identité**

## <a name="adding-spotinst-from-the-gallery"></a>Ajout de Spotinst à partir de la galerie

Pour configurer l’intégration de Spotinst à Azure AD, vous devez ajouter Spotinst, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Spotinst** dans la zone de recherche.
1. Sélectionnez **Spotinst** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-spotinst"></a>Configurer et tester l’authentification unique Azure AD pour Spotinst

Configurez et testez l’authentification unique Azure AD avec Spotinst à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Spotinst associé.

Pour configurer et tester l’authentification unique Azure AD avec Spotinst, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Spotinst](#configure-spotinst-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Spotinst](#create-spotinst-test-user)** pour avoir un équivalent de B.Simon dans Spotinst lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Spotinst**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, si vous souhaitez configurer l’application en mode Lancé par le fournisseur d’identité, effectuez les étapes suivantes :

   1. Vérifiez que l’**URL de réponse** est définie sur : https://console.spotinst.com/auth/saml.
   1. Dans **État de relais**, entrez votre ID d’organisation Spotinst, que vous pouvez également vérifier sous l’onglet **SSO**.
   1. L’**URL de connexion** doit être vide.

1. Cliquez sur **Enregistrer**.

1. L’application Spotinst s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application Spotinst s’attend à ce que quelques attributs supplémentaires (présentés ci-dessous) soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source|
    | -----| --------------- |
    | E-mail | user.mail |
    | FirstName | user.givenname |
    | LastName | user.surname |

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer Spotinst**, copiez l’URL ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Spotinst.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Spotinst**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-spotinst-sso"></a>Configurer l’authentification unique Spotinst

1. Dans une autre fenêtre de navigateur web, connectez-vous à Spotinst en tant qu’administrateur de la sécurité.

2. Cliquez sur **l’icône de l’utilisateur** dans le coin supérieur droit de l’écran et cliquez sur **Settings** (Paramètres).

    ![Capture d’écran montrant l’élément Settings sélectionné à partir de l’icône User.](./media/spotinst-tutorial/tutorial_spotinst_settings.png)

3. Cliquez sur l’onglet **SECURITY** (SÉCURITÉ) en haut, puis sélectionnez **Identity Providers** (Fournisseurs d’identité) et suivez ces étapes :

    ![Sécurité de Spotinst](./media/spotinst-tutorial/tutorial_spotinst_security.png)

    a. Copiez la valeur de **Relay State** (État de relais) et collez-la dans la zone de texte **État de relais** de la section **Configuration SAML de base** dans le Portail Azure.

    b. Cliquez sur **BROWSE** (PARCOURIR) pour charger le fichier xml de métadonnées que vous avez téléchargé du portail Azure.

    c. Cliquez sur **ENREGISTRER**.

### <a name="create-spotinst-test-user"></a>Créer un utilisateur de test Spotinst

L’objectif de cette section est de créer un utilisateur appelé Britta Simon dans Spotinst.

1. Si vous avez configuré l’application en mode initié par le **fournisseur de service**, procédez comme suit :

   a. Dans une autre fenêtre de navigateur web, connectez-vous à Spotinst en tant qu’administrateur de la sécurité.

   b. Cliquez sur **l’icône de l’utilisateur** dans le coin supérieur droit de l’écran et cliquez sur **Settings** (Paramètres).

    ![Capture d’écran montrant l’élément Settings sélectionné à partir de l’icône User.](./media/spotinst-tutorial/tutorial_spotinst_settings.png)

    c. Cliquez sur **Users** (Utilisateurs) et sélectionnez **ADD USER** (AJOUTER UN UTILISATEUR).

    ![Capture d’écran montrant l’élément ADD USER sélectionné à partir de Users.](./media/spotinst-tutorial/adduser1.png)

    d. Dans la section d’ajout d’un utilisateur, procédez comme suit :

    ![Capture d’écran affichant la section Add user dans laquelle vous pouvez indiquer les valeurs décrites.](./media/spotinst-tutorial/adduser2.png)

    * Dans la zone de texte **Full Name** (Nom complet), entrez le nom entier d’un utilisateur, par exemple **Britta Simon**.

    * Dans la zone de texte **Email** (E-mail), entrez l’adresse e-mail de l’utilisateur, par exemple `brittasimon\@contoso.com`.

    * Sélectionnez les détails propres à votre organisation pour **Organization Role (Rôle de l’organisation), Account Role (Rôle de compte) et Accounts (Comptes)**.

2. Si vous avez configuré l’application en mode initié par le **fournisseur d’identité**, vous n’avez aucune opération à effectuer dans cette section. Spotinst prend en charge le provisionnement juste-à-temps, option qui est activée par défaut. S’il n’existe pas déjà, un utilisateur est créé lors d’une tentative d’accès à Spotinst.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Workpath dans le panneau d’accès doit vous connecter automatiquement à l’application Workpath pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Spotinst avec Azure AD](https://aad.portal.azure.com/)