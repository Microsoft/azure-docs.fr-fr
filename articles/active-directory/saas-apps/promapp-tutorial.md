---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Nintex Promapp | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Nintex Promapp.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/30/2020
ms.author: jeedes
ms.openlocfilehash: fc31195e7f544bdce7fe2f135a39cb9992875d0a
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92505882"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-nintex-promapp"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Nintex Promapp

Dans ce tutoriel, vous allez découvrir comment intégrer Nintex Promapp à Azure Active Directory (Azure AD). Quand vous intégrez Nintex Promapp à Azure AD, vous pouvez :

* Contrôler qui dans Azure AD a accès à Nintex Promapp.
* Permettre à vos utilisateurs de se connecter automatiquement à Nintex Promapp avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Nintex Promapp pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Nintex Promapp prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**
* Nintex Promapp prend en charge le provisionnement d’utilisateurs **juste-à-temps** .

## <a name="adding-nintex-promapp-from-the-gallery"></a>Ajout de Nintex Promapp à partir de la galerie

Pour configurer l’intégration de Nintex Promapp avec Azure AD, vous devez ajouter Nintex Promapp à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Nintex Promapp** dans la zone de recherche.
1. Sélectionnez **Nintex Promapp** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-nintex-promapp"></a>Configurer et tester l’authentification unique Azure AD pour Nintex Promapp

Configurez et testez l’authentification unique Azure AD auprès de Nintex Promapp avec un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Nintex Promapp associé.

Pour configurer et tester l’authentification unique Azure AD avec Nintex Promapp, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Nintex Promapp](#configure-nintex-promapp-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Nintex Promapp](#create-nintex-promapp-test-user)** pour avoir un équivalent de B.Simon dans Nintex Promapp lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Nintex Promapp** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    1. Dans la zone **Identificateur** , entrez une URL au format suivant :

        ```https
        https://go.promapp.com/TENANTNAME/
        https://au.promapp.com/TENANTNAME/
        https://us.promapp.com/TENANTNAME/
        https://eu.promapp.com/TENANTNAME/
        https://ca.promapp.com/TENANTNAME/
        ```

       > [!NOTE]
       > L’intégration d’Azure AD à Nintex Promapp est actuellement configurée uniquement pour l’authentification lancée par le service. (Autrement dit, l’accès à une URL Nintex Promapp lance le processus d’authentification.) Mais le champ **URL de réponse** est un champ obligatoire.

    1. Dans la zone **URL de réponse** , entrez une URL au format suivant :

       `https://<DOMAINNAME>.promapp.com/TENANTNAME/saml/authenticate.aspx`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone **URL de connexion** , entrez une URL au format suivant : `https://<DOMAINNAME>.promapp.com/TENANTNAME/saml/authenticate`

    > [!NOTE]
    > Ces valeurs sont des espaces réservés. Vous devez utiliser l’identificateur, l’URL de réponse et l’URL de connexion exacts. Pour obtenir ces valeurs, contactez l’[équipe du support technique Nintex Promapp](https://www.promapp.com/about-us/contact-us/). Vous pouvez aussi vous référer aux modèles figurant dans la boîte de dialogue **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Nintex Promapp** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Nintex Promapp.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Nintex Promapp** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-nintex-promapp-sso"></a>Configurer l’authentification unique Nintex Promapp

1. Connectez-vous à votre site d’entreprise Nintex Promapp en tant qu’administrateur.

2. Dans le menu en haut de la fenêtre, sélectionnez **Administrateur**  :

    ![Sélectionnez Administrateur][12]

3. Sélectionnez **Configurer**  :

    ![Sélectionnez Configurer][13]

4. Dans la boîte de dialogue **Sécurité** , procédez comme suit.

    ![Boîte de dialogue Sécurité][14]

    1. Collez **l’URL de connexion** que vous avez copiée à partir du Portail Azure dans la zone **SSO-Login URL (SSO-URL de connexion)** .

    1. Dans la liste **Mode SSO (authentification unique)** , sélectionnez **Facultatif** . Sélectionnez **Enregistrer** .

       > [!NOTE]
       > Le mode Facultatif est utilisé à des fins de test uniquement. Lorsque vous êtes satisfait de la configuration, sélectionnez **Requis** dans la liste **Mode SSO (authentification unique)** pour forcer tous les utilisateurs à s’authentifier auprès d’Azure AD.

    1. Dans le bloc-notes, ouvrez le certificat que vous avez téléchargé à la section précédente. Copiez le contenu du certificat sans la première ligne ( **-----BEGIN CERTIFICATE-----** ) ou la dernière ligne ( **-----END CERTIFICATE-----** ). Collez le contenu du certificat dans la zone **Certificat x.509 d’authentification unique** , puis sélectionnez **Enregistrer** .

### <a name="create-nintex-promapp-test-user"></a>Créer un utilisateur de test Nintex Promapp

Dans cette section, un utilisateur appelé B.Simon est créé dans Nintex Promapp. Nintex Promapp prend en charge le provisionnement d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Nintex Promapp, il en est créé un après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Nintex Promapp dans le volet d’accès, vous devez être connecté automatiquement à l’application Nintex Promapp pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Nintex Promapp avec Azure AD](https://aad.portal.azure.com/)

<!--Image references-->

[12]: ./media/promapp-tutorial/tutorial_promapp_05.png
[13]: ./media/promapp-tutorial/tutorial_promapp_06.png
[14]: ./media/promapp-tutorial/tutorial_promapp_07.png