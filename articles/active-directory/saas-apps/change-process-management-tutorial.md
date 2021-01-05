---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure AD avec Change Process Management'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Change Process Management.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/07/2020
ms.author: jeedes
ms.openlocfilehash: 03c78f05566876356e4f486368dc2a5b3a29de43
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92456254"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-change-process-management"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec Change Process Management

Ce tutoriel explique comment intégrer Change Process Management avec Azure Active Directory (Azure AD). Quand vous intégrez Change Process Management avec Azure AD, vous pouvez :

* Utiliser Azure AD pour contrôler qui peut accéder à Change Process Management.
* Permettre à vos utilisateurs de se connecter automatiquement à Change Process Management avec leur compte Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS à Azure AD, consultez l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Change Process Management pour lequel l’authentification unique (SSO) est activée.

## <a name="tutorial-description"></a>Description du tutoriel

Dans ce tutoriel, vous allez configurer et tester l’authentification SSO Azure AD dans un environnement de test.

Change Process Management prend en charge l’authentification unique lancée par le fournisseur d’identité.

Une fois que vous avez configuré Change Process Management, vous pouvez appliquer le contrôle de session qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Les contrôles de session sont étendus à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="add-change-process-management-from-the-gallery"></a>Ajouter Change Process Management à partir de la Galerie

Pour configurer l’intégration de Change Process Management à Azure AD, vous devez ajouter Change Process Management à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire, ou avec un compte Microsoft personnel.
1. Sélectionnez **Azure Active Directory** dans le volet de gauche.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Change Process Management** dans la zone de recherche.
1. Sélectionnez **Change Process Management** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-change-process-management"></a>Configurer et tester l’authentification unique Azure AD pour Change Process Management

Vous allez configurer et tester l’authentification unique Azure AD avec Change Process Management à l’aide d’un utilisateur de test appelé B.Simon. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Change Process Management associé.

Pour configurer et tester l’authentification unique Azure AD avec Change Process Management, suivez les étapes générales suivantes :

1. **[Configurer l’authentification SSO Azure AD](#configure-azure-ad-sso)** , pour permettre à vos utilisateurs d’utiliser la fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD.
    1. **[Octroyer l’accès à l’utilisateur de test](#grant-access-to-the-test-user)** , pour lui permettre d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Change Process Management](#configure-change-process-management-sso)** côté application.
    1. **[Créer un utilisateur de test Change Process Management](#create-a-change-process-management-test-user)** , pour avoir un équivalent qui soit lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification SSO](#test-sso)** , pour vérifier que la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [Portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Change Process Management** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez le bouton représentant un crayon et correspondant à **Configuration SAML de base** pour modifier les paramètres :

   ![Bouton représentant un crayon pour la configuration SAML de base](common/edit-urls.png)

1. Sur la page **Configurer l’authentification unique avec SAML** , procédez comme suit :

    a. Dans la zone **Identificateur** , entrez une URL au format suivant : `https://<hostname>:8443/`

    b. Dans la zone **URL de réponse** , entrez une URL au format suivant : `https://<hostname>:8443/changepilot/saml/sso`

    > [!NOTE]
    > Les valeurs d’ **identificateur** et d’ **URL de réponse** précédentes ne sont pas les valeurs réelles que vous devez utiliser. Pour obtenir les valeurs réelles, contactez l’[équipe de support technique Change Process Management](mailto:support@realtech-us.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , sélectionnez le lien **Télécharger** correspondant à **Certificat (Base64)** pour télécharger le certificat et l’enregistrer sur votre ordinateur :

    ![Lien de téléchargement du certificat](common/certificatebase64.png)

1. Dans la section **Configurer Change Process Management** , copiez la ou les URL appropriées en fonction de vos besoins :

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test nommé B.Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** . Sélectionnez **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans la zone **Nom** , entrez **B.Simon** .  
   1. Dans la zone **Nom d’utilisateur** , entrez \<username>@\<companydomain>.\<extension>. Par exemple : `B.Simon@contoso.com`.
   1. Sélectionnez **Afficher le mot de passe** , puis notez la valeur affichée dans la zone **Mot de passe** .
   1. Sélectionnez **Create** (Créer).

### <a name="grant-access-to-the-test-user"></a>Octroyer l’accès à l’utilisateur de test

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Change Process Management.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Change Process Management** .
1. Dans la page de vue d’ensemble de l’application, dans la section **Gérer** , sélectionnez **Utilisateurs et groupes**  :

   ![Sélectionner Utilisateurs et groupes](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Sélectionnez Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B.Simon** dans la liste **Utilisateurs** , puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Affecter** .

## <a name="configure-change-process-management-sso"></a>Configurer l’authentification unique de Change Process Management

Pour configurer l’authentification unique côté Change Process Management, vous devez envoyer le certificat Base64 téléchargé et les URL appropriées copiées à partir du portail Azure à l’[équipe du support technique Change Process Management](mailto:support@realtech-us.com). Celle-ci configure la connexion SSO SAML de manière appropriée des deux côtés.

### <a name="create-a-change-process-management-test-user"></a>Créer un utilisateur de test Change Process Management
 Collaborez avec l’[équipe de support Change Process Management](mailto:support@realtech-us.com) pour ajouter un utilisateur nommé B.Simon dans Change Process Management. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification SSO Azure AD à l’aide du volet d’accès.

Lorsque vous sélectionnez la vignette Change Process Management dans le volet d’accès, vous devez être automatiquement connecté à l’instance Change Process Management pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le volet d’accès, consultez [Présentation du volet d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Change Process Management avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Change Process Management avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)