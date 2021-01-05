---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Raketa | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Raketa.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/28/2020
ms.author: jeedes
ms.openlocfilehash: 649a26e474c7c4d6b4f51a035b3f8f9da35b9dd1
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92511152"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-raketa"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Raketa

Dans ce tutoriel, vous allez apprendre à intégrer Raketa à Azure Active Directory (Azure AD). Quand vous intégrez Raketa à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Raketa.
* Permettre à vos utilisateurs de se connecter automatiquement à Raketa avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Raketa pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Raketa prend en charge l’authentification unique initiée par le **fournisseur de services** .
* Après avoir configuré Raketa, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-raketa-from-the-gallery"></a>Ajout de Raketa à partir de la galerie

Pour configurer l’intégration de Raketa à Azure AD, vous devez ajouter Raketa à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** [1].

    ![rkt_1](./media/raketa-tutorial/azure-active-directory.png)

1. Accédez à **Applications d’entreprise** [2], puis sélectionnez **Toutes les applications** [3].

1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** [4]. 

    ![rkt_2](./media/raketa-tutorial/new-app.png)

1. Dans la section **Ajouter à partir de la galerie** [5], tapez **Raketa** dans la zone de recherche [6].

1. Sélectionnez **Raketa** dans le volet de résultats [7], puis cliquez sur le bouton **Ajouter** [8]. 

    ![rkt_3](./media/raketa-tutorial/add-btn.png)


## <a name="configure-and-test-azure-ad-single-sign-on-for-raketa"></a>Configurer et tester l’authentification unique Azure AD pour Raketa

Configurez et testez l’authentification unique Azure AD avec Raketa à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Raketa associé.

Pour configurer et tester l’authentification unique Azure AD avec Raketa, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Raketa](#configure-raketa-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Raketa](#create-raketa-test-user)** pour avoir un équivalent de B.Simon dans Raketa lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Sur le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Raketa** , recherchez la section **Gérer** et sélectionnez **Authentification unique** [9].

    ![rkt_4](./media/raketa-tutorial/manage-sso.png)

1. Dans la page **Sélectionner une méthode d’authentification unique** [9], sélectionnez **SAML** [10].

    ![rkt_5](./media/raketa-tutorial/saml.png)

1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** [11] pour modifier les paramètres.

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    1. Dans les zones de texte **Identificateur (ID d’entité)** [12] et **URL de connexion** [14], tapez l’URL `https://raketa.travel/`.

    1. Dans la zone de texte **URL de réponse** [13], tapez une URL au format suivant : `https://raketa.travel/sso/acs?clientId=<CLIENT_ID>`.  

    ![rkt_6](./media/raketa-tutorial/enter-urls.png)

    > [!NOTE]
    > La valeur de l’URL de réponse n’est pas réelle. Mettez à jour la valeur avec l’URL de réponse réelle. Pour obtenir cette valeur, contactez l’[équipe de support client Raketa](mailto:help@raketa.travel). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** et sélectionnez **Télécharger** [15] pour télécharger le certificat et l’enregistrer sur votre ordinateur.

1. Dans la section **Configurer Raketa** , copiez la ou les URL appropriées selon vos besoins.

    1. URL de connexion [16] : URL de la page web d’autorisation, qui est utilisée pour rediriger les utilisateurs vers le système d’authentification.

    1. Identificateur Azure AD [17] : identificateur Azure AD.

    1. URL de déconnexion [18] : URL de page web, qui est utilisée pour rediriger les utilisateurs après la déconnexion.

    ![rkt_7](./media/raketa-tutorial/copy-urls.png)


### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** [1], **Utilisateurs** [19], puis **Tous les utilisateurs** [20].

1. Sélectionnez **Nouvel utilisateur** [21] dans la partie supérieure de l’écran.

    ![rkt_8](./media/raketa-tutorial/new-user.png)

1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :

   1. Dans le champ **Nom d’utilisateur** [22], entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.

   1. Dans le champ **Nom** [23], entrez `B.Simon`.

   1. Cochez la case **Afficher le mot de passe** [25], puis notez la valeur affichée dans la zone **Mot de passe** [24].

   1. Cliquez sur **Créer** [26]. 

    ![rkt_9](./media/raketa-tutorial/create-user.png)


### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Raketa.

1. Sur le portail Azure, sélectionnez **Applications d’entreprise** [2], puis **Toutes les applications** [3].

1. Dans la liste des applications, sélectionnez **Raketa** [27].  

    ![rkt_10](./media/raketa-tutorial/add-raketa.png)

1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** [28]. 

    ![rkt_11](./media/raketa-tutorial/users-groups.png)

1. Sélectionnez **Ajouter un utilisateur** [29], puis **Utilisateurs et groupes** [30] dans la boîte de dialogue **Ajouter une attribution** .

    ![rkt_12](./media/raketa-tutorial/add-user-raketa.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B.Simon** [31] dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** [32] au bas de l’écran.

1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** [33]. 

    ![rkt_13](./media/raketa-tutorial/assign-user.png)

## <a name="configure-raketa-sso"></a>Configurer l’authentification unique Raketa

Pour configurer l’authentification unique côté **Raketa** , vous devez envoyer le **certificat (Base64)** téléchargé et les URL appropriées copiées à partir du portail Azure à l’ [équipe du support technique Raketa](mailto:help@raketa.travel). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-raketa-test-user"></a>Créer un utilisateur de test Raketa

Dans cette section, vous créez un utilisateur appelé B.Simon dans Raketa. Contactez l’[équipe de support Raketa](mailto:help@raketa.travel) pour ajouter les utilisateurs dans la plateforme Raketa. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Raketa dans le volet d’accès, vous devez être connecté automatiquement à l’application Raketa pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Raketa avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Raketa avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)