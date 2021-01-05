---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Harness | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Harness.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/02/2019
ms.author: jeedes
ms.openlocfilehash: d6a6c8b49582b34c2603e0ddf78b76736f97c183
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92445577"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-harness"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Harness

Dans ce tutoriel, vous allez apprendre à intégrer Harness à Azure Active Directory (Azure AD). Quand vous intégrez Harness à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Harness.
* Permettre à vos utilisateurs de se connecter automatiquement à Harness avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Harness pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Harness prend en charge l’authentification unique (SSO) lancée **le fournisseur de services et le fournisseur d’identité**

## <a name="adding-harness-from-the-gallery"></a>Ajout de Harness à partir de la galerie

Pour configurer l’intégration de Harness à Azure AD, vous devez ajouter Harness de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Harness** dans la zone de recherche.
1. Sélectionnez **Harness** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-harness"></a>Configurer et tester l’authentification unique Azure AD pour Harness

Configurez et testez l’authentification unique (SSO) Azure AD avec Harness à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique (SSO) fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur Harness associé.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec Harness, suivez les modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique (SSO) Harness](#configure-harness-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Harness](#create-harness-test-user)** pour disposer, dans Harness, d’un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans la page d’intégration de l’application **Harness** du [portail Azure](https://portal.azure.com/), recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://app.harness.io/gateway/api/users/saml-login?accountId=<harness_account_id>`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL : `https://app.harness.io/`

    > [!NOTE]
    > La valeur de l’URL de réponse n’est pas réelle. Vous obtiendrez l’URL de réponse réelle en suivant la procédure décrite dans la section **Configurer l’authentification unique (SSO) Harness** , plus loin dans ce tutoriel. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer Harness** , copiez la ou les URL appropriées, selon vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Harness.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Harness** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-harness-sso"></a>Configurer l’authentification unique (SSO) Harness

1. Pour automatiser la configuration dans BetterWorks, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer BetterWorks** pour être redirigé vers l’application BetterWorks. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à BetterWorks. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 6.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement BetterWorks, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise BetterWorks en tant qu’administrateur, puis effectuez les étapes suivantes :

4. En haut à droite de la page, cliquez sur **Sécurité continue** > **Gestion de l’accès** > **Paramètres d’authentification** .

    ![Capture d’écran montrant le menu « Continuous Security » avec les éléments « Access Management » et « Authentication Settings » sélectionnés.](./media/harness-tutorial/configure01.png)

5. Dans la section **Fournisseurs SSO** , cliquez sur **+ Ajouter des fournisseurs SSO** > **SAML** .

    ![Capture d’écran montrant la zone « SSO Providers » avec l’option « Add SSO Providers - SAML » sélectionnée.](./media/harness-tutorial/configure03.png)

6. Dans la fenêtre contextuelle **Fournisseur SAML** , effectuez les étapes suivantes :

    ![Capture d’écran montrant la fenêtre contextuelle « SAML Provider » avec les champs « URL » et « Display Name » mis en évidence, ainsi que les boutons « File » et « Submit » sélectionnés.](./media/harness-tutorial/configure02.png)

    a. Copiez l’instance **Dans votre fournisseur SSO, activez la connexion basée sur SAML, puis entrez l’URL suivante** , puis collez-la dans la zone de texte URL de réponse de la section **Configuration SAML de base** .

    b. Dans la zone de texte **Nom d’affichage** , tapez votre nom d’affichage.

    c. Cliquez sur **Choisir un fichier** pour charger le fichier XML de métadonnées de fédération que vous avez téléchargé à partir d’Azure AD.

    d. Cliquez sur **ENVOYER** .

### <a name="create-harness-test-user"></a>Créer un utilisateur de test Harness

Pour permettre aux utilisateurs Azure AD de se connecter à Harness, vous devez les attribuer dans Harness. Dans Harness, le provisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à Harness en tant qu’administrateur.

1. En haut à droite de la page, cliquez sur **Sécurité continue** > **Gestion de l’accès** > **Utilisateurs** .

    ![Capture d’écran montrant le menu « Continuous Security » avec les éléments « Access Management » et « Users » sélectionnés.](./media/harness-tutorial/configure04.png)

1. À droite de la page, cliquez sur **+ Ajouter un utilisateur** .

    ![Capture d’écran montrant la page « Users » avec l’action « + Add User » sélectionnée.](./media/harness-tutorial/configure05.png)

1. Dans la fenêtre contextuelle **Ajouter un utilisateur** , effectuez les étapes suivantes :

    ![Configuration de Harness](./media/harness-tutorial/configure06.png)

    a. Dans la zone de texte **Adresses e-mail** , entrez l’adresse e-mail de l’utilisateur, par exemple `B.simon@contoso.com`.

    b. Sélectionnez votre **Groupe d’utilisateurs** .

    c. Cliquez sur **Envoyer** .

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Harness dans le volet d’accès doit vous connecter automatiquement à l’application Harness pour laquelle vous avez configuré l’authentification unique (SSO). Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Harness avec Azure AD](https://aad.portal.azure.com/)