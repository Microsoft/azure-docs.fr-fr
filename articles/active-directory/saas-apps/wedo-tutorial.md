---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory avec WEDO | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et WEDO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/22/2020
ms.author: jeedes
ms.openlocfilehash: 529c4e6433d16f9d70530ba516b5ec1426806984
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92519229"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-wedo"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à WEDO

Dans ce tutoriel, vous allez apprendre à intégrer WEDO à Azure Active Directory (Azure AD). Quand vous intégrez WEDO à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à WEDO.
* Permettre à vos utilisateurs de se connecter automatiquement à WEDO avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement WEDO pour lequel l’authentification unique est activée. Contactez l’équipe de [support client WEDO](mailto:info@wedo.swiss) pour obtenir un abonnement pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* WEDO prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**

* [Après avoir configuré WEDO, vous pouvez appliquer des contrôles de session qui protègent l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Les contrôles de session sont étendus à partir de l’accès conditionnel. Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-wedo-from-the-gallery"></a>Ajout de WEDO à partir de la galerie

Pour configurer l’intégration de WEDO avec Azure AD, vous devez ajouter WEDO à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **WEDO** dans la zone de recherche.
1. Sélectionnez **WEDO** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-wedo"></a>Configurer et tester l’authentification unique Azure AD pour WEDO

Configurez et testez l’authentification unique Azure AD avec WEDO pour un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur WEDO associé.

Pour configurer et tester l’authentification unique Azure AD avec WEDO, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique WEDO](#configure-wedo-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test WEDO](#create-wedo-test-user)** pour avoir un équivalent de B.Simon dans WEDO, associé à sa représentation dans Azure AD.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **WEDO** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<SUBDOMAIN>.wedo.swiss/sp/acs`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<SUBDOMAIN>.wedo.swiss/sp/acs`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<SUBDOMAIN>.wedo.swiss/`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez l’[équipe de support client de WEDO](mailto:info@wedo.swiss). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Votre application WEDO s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration des attributs de jetons SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    | Nom | Attribut source|
    | ------------ | --------- |
    | email | user.email |
    | firstName | user.firstName |
    | lastName | user.lasttName |
    | userName | user.userName |

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer WEDO** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à WEDO.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **WEDO** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-wedo-sso"></a>Configurer l’authentification unique WEDO

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans WEDO.

1. Connectez-vous à [WEDO](https://login.wedo.swiss/). Vous devez disposer du **rôle d’administrateur** .
1. Dans les paramètres de profil, sélectionnez le menu **Authentication** (Authentification) dans la section **Network settings** (Paramètres réseau).
1. Dans la page **SAML Authentication** , effectuez les étapes suivantes :

   ![Lien SAML Authentication](media/wedo-tutorial/network-security-authentification.png)

   a. Activez **SAML Authentication** .

   b. Sélectionnez l’onglet **Identity Provider metadata (XML)** (Métadonnées du fournisseur d’identité (XML)).

   c. Dans le Bloc-notes, ouvrez le **XML de métadonnées de fédération** téléchargé à partir du portail Azure, copiez son contenu et collez-le dans la zone de texte **X.509 Certificate** .

   d. Dans le menu Paramètres, cliquez sur **Enregistrer**

### <a name="create-wedo-test-user"></a>Créer un utilisateur de test WEDO

Dans cette section, vous allez créer un utilisateur de test dans WEDO appelé Bob Simon. Les informations doivent être identiques à celles de *Créer un utilisateur de test Azure AD* .

1. Dans les paramètres de profil dans WEDO, sélectionnez **Users** (Utilisateurs) dans la section *Network settings* .
1. Cliquez sur **Add User** .
1. Dans la fenêtre contextuelle Add user, renseignez les informations de l’utilisateur

    a. Prénom `B`.

    b. Nom `Simon`.

    c. Entrez l’adresse e-mail `username@companydomain.extension`. Par exemple : `B.Simon@contoso.com`. Il est obligatoire d’avoir un e-mail avec le même domaine que le nom abrégé de votre société.

    d. Type d’utilisateur `User`.

    e. Cliquez sur **Create User** (Créer un utilisateur).

    f. Dans la page *Select teams* (Sélectionner les équipes), cliquez sur **Save** .

    g.  Dans la page *Invite user* (Inviter l’utilisateur), cliquez sur **Yes** .

1. Valider l’utilisateur à l’aide du lien que vous avez reçu par e-mail

> [!NOTE]
> Si vous souhaitez créer un utilisateur factice (l’e-mail ci-dessus n’existe pas sur votre réseau), contactez [notre équipe de support](mailto:info@wedo.swiss) pour valider l’utilisateur*.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette WEDO dans le volet d’accès, vous devez être connecté automatiquement à l’application WEDO pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer WEDO avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger WEDO avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)