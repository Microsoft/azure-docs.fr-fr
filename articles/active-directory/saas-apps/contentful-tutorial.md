---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Contentful | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Contentful.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/20/2020
ms.author: jeedes
ms.openlocfilehash: b664a7e4aac571359fb1494cf69d5bbe3e841b4d
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92455310"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-contentful"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Contentful

Dans ce tutoriel, vous allez découvrir comment intégrer Contentful à Azure Active Directory (Azure AD). Quand vous intégrez Contentful à Azure AD, vous pouvez :

* Contrôler qui a accès à Contentful dans Azure AD.
* Permettre à vos utilisateurs de se connecter automatiquement à Contentful avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Contentful pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Contentful prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**
* Contentful prend en charge le provisionnement d’utilisateur **juste-à-temps**
* Après avoir configuré Contentful, vous pouvez appliquer le contrôle de session qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

> [!NOTE]
> L’identificateur de cette application est une valeur de chaîne fixe. Une seule instance peut être configurée dans un locataire.

## <a name="adding-contentful-from-the-gallery"></a>Ajout de Contentful à partir de la galerie

Pour configurer l’intégration de Contentful à Azure AD, vous devez ajouter Contentful à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le volet de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Contentful** dans la zone de recherche.
1. Sélectionnez **Contentful** dans les résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-contentful"></a>Configurer et tester l’authentification unique Azure AD pour Contentful

Configurez et testez l’authentification unique Azure AD avec Contentful à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Contentful associé.

Pour configurer et tester l’authentification unique Azure AD avec Contentful, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Contentful](#configure-contentful-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Contentful](#create-contentful-test-user)** pour avoir un équivalent de B.Simon dans Contentful lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Contentful** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous voulez configurer l’application en mode lancé par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    - Dans la zone de texte **URL de réponse** , copiez l’URL ACS (Assertion Consumer Service) à partir de la page de configuration de l’authentification unique dans Contentful. Elle se présente comme suit : `https://be.contentful.com/sso/<organization_id>/consume`

1. Cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes si vous voulez configurer l’application en mode lancé par le **fournisseur de services**  :

    - Dans la zone de texte **URL de connexion** , copiez la même URL ACS (Assertion Consumer Service). Elle se présente comme suit : `https://be.contentful.com/sso/<organization_id>/login`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de réponse et l’URL de connexion réelles en copiant l’URL ACS (Assertion Consumer Service) à partir de la page de configuration de l’authentification unique dans Contentful.

1. Votre application Contentful s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration des attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application Contentful s’attend à ce que quelques attributs supplémentaires (présentés ci-dessous) soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.
    
    | Nom |  Attribut source|
    | --------------- | --------- |
    | email | user.userprincipalname |

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (Base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Contentful** , copiez l’URL de connexion pour configurer l’authentification unique Contentful.

    ![Copier les URL de configuration](media/contentful-tutorial/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Contentful.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Contentful** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B.Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de la page.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de la page.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-contentful-sso"></a>Configurer l’authentification unique Contentful

Effectuez les étapes suivantes pour configurer l’authentification unique côté **Contentful** .

1. Dans [Contentful](https://app.contentful.com), accédez à la page de configuration de l’authentification unique dans **Organization Settings** (Paramètres de l’organisation).
1. Cliquez sur **Set up SSO** (Configurer l’authentification unique).
1. Copiez et collez l’URL de connexion à partir de la section **Configurer Contentful** dans Azure AD.
1. Copiez et collez le certificat à partir du fichier de certificat base64 que vous avez téléchargé à partir d’Azure AD.
1. Configurez un nom SSO pour la connexion lancée par le fournisseur de service.
1. Cliquez sur **Enable SSO** (Activer l’authentification unique).

Si cela ne fonctionne pas, contactez l’[équipe de support technique Contentful](mailto:support@contentful.com).

### <a name="create-contentful-test-user"></a>Créer un utilisateur de test Contentful

Dans cette section, un utilisateur appelé B.Simon est créé dans Contentful. Contentful prend en charge le provisionnement d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Contentful, il en est créé un après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Contentful dans le volet d’accès, vous devez être connecté automatiquement à l’application Contentful pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Contentful avec Azure AD](https://aad.portal.azure.com/)