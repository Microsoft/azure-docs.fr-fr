---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec Khoros Care | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Khoros Care.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/27/2020
ms.author: jeedes
ms.openlocfilehash: 5f54c61cf8ce056c79faf84b64cf6650ea1db513
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92459031"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-khoros-care"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec Khoros Care

Dans ce tutoriel, vous allez apprendre à intégrer Khoros Care avec Azure AD (Azure Active Directory). Quand vous intégrez Khoros Care avec Azure AD, vous pouvez :

* Contrôler dans Azure AD l’accès à Khoros Care
* Permettre aux utilisateurs de se connecter automatiquement à Khoros Care avec leurs comptes Azure AD
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Khoros Care pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Khoros Care prend en charge l’authentification SSO lancée par le **SP (fournisseur de services) et l’IdP (fournisseur d’identité)**
* Khoros Care prend en charge l’attribution d’utilisateurs **juste-à-temps**
* Une fois que vous avez configuré Khoros Care, vous pouvez appliquer des contrôles de session, qui protègent l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-khoros-care-from-the-gallery"></a>Ajout de Khoros Care à partir de la galerie

Pour configurer l’intégration de Khoros Care avec Azure AD, vous devez ajouter Khoros Care à votre liste d’applications SaaS managées, à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , dans la zone de recherche, tapez **Khoros Care** .
1. Sélectionnez **Khoros Care** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-khoros-care"></a>Configurer et tester l’authentification unique Azure AD pour Khoros Care

Configurez et testez l’authentification SSO Azure AD avec Khoros Care à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification SSO fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans Khoros Care.

Pour configurer et tester l’authentification SSO Azure AD avec Khoros Care, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification SSO Khoros Care](#configure-khoros-care-sso)** , pour configurer les paramètres d’authentification unique côté application.
    1. **[Créer un utilisateur de test Khoros Care](#create-khoros-care-test-user)** , pour avoir un équivalent de B.Simon dans Khoros Care, qui soit lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration d’application **Khoros Care** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<SUBDOMAIN>.response.lithium.com`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<SUBDOMAIN>.response.lithium.com/sso/saml/v2/idp_response`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<SUBDOMAIN>.response.lithium.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez l’[équipe du support technique de Khoros Care](mailto:support@khoros.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. L’application Khoros Care attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application Khoros Care s’attend à ce que quelques attributs supplémentaires repassent dans la réponse SAML, comme indiqué ci-dessous. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source|
    | ------------ | --------- |
    | FirstName | user.givenname |
    | LastName | user.surname |

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer Khoros Care** , copiez les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez permettre à B.Simon d’utiliser l’authentification unique Azure en lui octroyant l’accès à Khoros Care.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Khoros Care** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-khoros-care-sso"></a>Configurer l’authentification SSO pour Khoros Care

Pour configurer l’authentification unique côté **Khoros Care** , vous devez envoyer le **XML des métadonnées de fédération** téléchargé et les URL appropriées copiées depuis le portail Azure à l’ [équipe du support technique de Khoros Care](mailto:support@khoros.com). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-khoros-care-test-user"></a>Créer un utilisateur de test Khoros Care

Dans cette section, un utilisateur appelé Britta Simon est créé dans Khoros Care. Khoros Care prend en charge l’attribution d’utilisateurs juste-à-temps, qui est activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe aucun utilisateur dans Khoros Care, il est créé après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Khoros Care dans le volet d’accès, vous êtes automatiquement connecté à la plateforme Khoros Care pour laquelle vous avez configuré l’authentification SSO. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Khoros Care avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)