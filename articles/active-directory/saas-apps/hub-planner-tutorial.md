---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Hub Planner | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Hub Planner.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/12/2020
ms.author: jeedes
ms.openlocfilehash: 2d9dc483a9d60dc395c0aff52b721690d4fc701d
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92442420"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-hub-planner"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Hub Planner

Dans ce tutoriel, vous allez apprendre à intégrer Hub Planner à Azure Active Directory (Azure AD). Quand vous intégrez Hub Planner à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Hub Planner.
* Permettre à vos utilisateurs de se connecter automatiquement à Hub Planner avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Hub Planner pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Hub Planner prend en charge l’authentification unique lancée par le **fournisseur de services** .
* Après avoir configuré Hub Planner, vous pouvez appliquer le contrôle de session, qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-hub-planner-from-the-gallery"></a>Ajout de Hub Planner à partir de la galerie

Pour configurer l’intégration de Hub Planner à Azure AD, vous devez ajouter Hub Planner à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Hub Planner** dans la zone de recherche.
1. Sélectionnez **Hub Planner** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-hub-planner"></a>Configurer et tester l’authentification unique Azure AD pour Hub Planner

Configurez et testez l’authentification unique Azure AD avec Hub Planner à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Hub Planner associé.

Pour configurer et tester l’authentification unique Azure AD avec Hub Planner, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Hub Planner](#configure-hub-planner-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer utilisateur de test Hub Planner](#create-hub-planner-test-user)** pour avoir un équivalent de B.Simon dans Hub Planner lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Hub Planner** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<SUBDOMAIN>.hubplanner.com`.

    b. Dans la zone de texte **Identificateur** , tapez une URL en utilisant le format suivant : `https://app.hubplanner.com/sso/metadata`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://app.hubplanner.com/sso/callback`

    > [!NOTE]
    > Ces valeurs sont celles que vous utiliserez. La seule modification que vous devez effectuer consiste à remplacer \<SUBDOMAIN\> dans l’ **URL de connexion** par le sous-domaine que vous avez reçu lorsque vous vous êtes inscrit à Hub Planner. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Hub Planner** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Hub Planner.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Hub Planner** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-hub-planner-sso"></a>Configurer l’authentification unique Hub Planner

Pour configurer l’authentification unique côté **Hub Planner** , vous devez vous connecter à votre compte Hub Planner et effectuer les tâches suivantes. 

### <a name="install-the-extension-in-hub-planner"></a>Installer l’extension dans Hub Planner

Pour activer la fonctionnalité d’authentification unique, vous devez d’abord activer l’extension. En tant que propriétaire du compte ou qu’utilisateur disposant d’autorisations équivalentes, effectuez les étapes suivantes :

1. Accédez à **Settings** .
1. Dans le menu latéral, sélectionnez **Gérer les extensions** > **Ajouter/supprimer des extensions** .
1. Recherchez l’extension pour l’authentification unique et l’ajout ou pour un essai gratuit.
1. Quand vous y êtes invité, acceptez les conditions générales, puis sélectionnez **Ajouter maintenant** .

### <a name="enable-sso"></a>Activer l’authentification unique

Une fois l’extension activée, vous devez activer l’authentification unique pour votre compte. 

1. Accédez à **Settings** .
1. Dans le menu latéral, sélectionnez **Authentification** .
1. Sélectionnez **Authentification unique (SSO)** .
1. Entrez des informations d’authentification supplémentaires comme indiqué dans l’image suivante, puis sélectionnez **Enregistrer** .

![Capture d’écran des paramètres d’authentification unique](media/hub-planner-tutorial/sso-settings.png)

### <a name="create-hub-planner-test-user"></a>Créer un utilisateur de test Hub Planner

Si vous voulez ajouter d’autres utilisateurs, accédez à **Paramètres** > **Gérer les ressources** , puis ajoutez des utilisateurs à partir d’ici. Veillez à ajouter leur adresse e-mail et à les inviter. Une fois invités, ils recevront un e-mail et pourront entrer par le biais de l’authentification unique. 

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Hub Planner dans le volet d’accès, vous devez être connecté automatiquement à l’application Hub Planner pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Hub Planner avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Hub Planner avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)