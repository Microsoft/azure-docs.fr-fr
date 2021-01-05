---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à PureCloud by Genesys | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et PureCloud by Genesys.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/03/2019
ms.author: jeedes
ms.openlocfilehash: d7aa43a94c7b49eef5b3273617ad9038fd1202e4
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92505695"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-purecloud-by-genesys"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à PureCloud by Genesys

Dans ce tutoriel, vous allez apprendre à intégrer PureCloud by Genesys à Azure Active Directory (Azure AD). Ensuite, vous pourrez :

* Utiliser Azure AD pour contrôler les utilisateurs qui peuvent accéder à PureCloud by Genesys.
* Autoriser les utilisateurs à se connecter automatiquement à PureCloud by Genesys avec leur compte Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’en avez pas, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement PureCloud by Genesys pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* PureCloud by Genesys prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité** .

> [!NOTE]
> L’ID de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-purecloud-by-genesys-from-the-gallery"></a>Ajout de PureCloud by Genesys à partir de la galerie

Pour configurer l’intégration de PureCloud by Genesys à Azure AD, vous devez ajouter PureCloud by Genesys à partir de la galerie à votre liste d’applications SaaS managées. Pour ce faire, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com), soit avec un compte professionnel ou scolaire, soit avec un compte Microsoft personnel.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **PureCloud by Genesys** dans la zone de recherche.
1. Sélectionnez **PureCloud by Genesys** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-purecloud-by-genesys"></a>Configurer et tester l’authentification unique Azure AD pour PureCloud by Genesys

Configurez et testez l’authentification unique Azure AD avec PureCloud by Genesys pour un utilisateur de test nommé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans PureCloud by Genesys.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec PureCloud by Genesys, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique PureCloud by Genesys](#configure-purecloud-by-genesys-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test PureCloud by Genesys](#create-purecloud-by-genesys-test-user)** pour avoir un équivalent de B.Simon dans PureCloud by Genesys lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Pour activer l’authentification unique Azure AD dans le portail Azure, effectuez les étapes suivantes :

1. Sur le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **PureCloud by Genesys** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône de stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous voulez configurer l’application en mode **lancé par le fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone **Identificateur** , entrez une URL qui correspond à votre région :

    ```http
    https://login.mypurecloud.com/saml
    https://login.mypurecloud.de/saml
    https://login.mypurecloud.jp/saml
    https://login.mypurecloud.ie/saml
    https://login.mypurecloud.au/saml
    ```

    b. Dans la zone **URL de réponse** , entrez une URL qui correspond à votre région :

    ```http
    https://login.mypurecloud.com/saml
    https://login.mypurecloud.de/saml
    https://login.mypurecloud.jp/saml
    https://login.mypurecloud.ie/saml
    https://login.mypurecloud.com.au/saml
    ```

1. Sélectionnez **Définir des URL supplémentaires** et effectuez l’étape suivante si vous souhaitez configurer l’application en mode lancé par le **fournisseur de services**  :

    Dans la zone **URL de connexion** , entrez une URL qui correspond à votre région :
    
    ```http
    https://login.mypurecloud.com
    https://login.mypurecloud.de
    https://login.mypurecloud.jp
    https://login.mypurecloud.ie
    https://login.mypurecloud.com.au
    ```

1. Votre application PureCloud by Genesys attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration des attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut :

    ![image](common/default-attributes.png)

1. En outre, l’application PureCloud by Genesys s’attend également à ce que quelques attributs de plus soient transmis dans la réponse SAML, comme indiqués dans le tableau suivant. Ces attributs sont également préremplis, mais vous pouvez les examiner en fonction de vos besoins.

    | Nom | Attribut source|
    | ---------------| --------------- |
    | E-mail | user.userprincipalname |
    | OrganizationName | `Your organization name` |

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (Base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer PureCloud by Genesys** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test nommé B.Simon dans le portail Azure :

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom d’utilisateur** , entrez le nom d’utilisateur au format suivant : username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez configurer l’utilisateur B.Simon afin qu’il utilise l’authentification unique Azure en lui accordant l’accès à PureCloud by Genesys.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **PureCloud by Genesys** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B.Simon** dans la liste Utilisateurs, puis choisissez le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis choisissez le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-purecloud-by-genesys-sso"></a>Configurer l’authentification unique PureCloud by Genesys

1. Dans une autre fenêtre de navigateur web, connectez-vous à PureCloud by Genesys en tant qu’administrateur.

1. Sélectionnez **Admin** en haut, puis accédez à **Single Sign-on** (Authentification unique) sous **Integrations** .

    ![Capture d’écran montrant la fenêtre PureCloud Admin, où vous pouvez sélectionner Single Sign-on.](./media/purecloud-by-genesys-tutorial/configure01.png)

1. Basculez vers l’onglet **ADFS/Azure AD(Premium)** , puis effectuez les étapes suivantes :

    ![Capture d’écran montrant la page Integrations dans laquelle vous pouvez indiquer les valeurs décrites.](./media/purecloud-by-genesys-tutorial/configure02.png)

    a. Dans **ADFS Certificate** (Certificat ADFS), sélectionnez **Browse** (Parcourir) pour charger le certificat codé en base 64 que vous avez téléchargé du portail Azure.

    b. Dans la zone **ADFS Issuer URI** (URI de l’émetteur SAML), collez la valeur **Identificateur Azure AD** que vous avez copiée à partir du portail Azure.

    c. Dans la zone **Target URI** (URI cible), collez la valeur **URL de connexion** que vous avez copiée du portail Azure.

    d. Pour la valeur **Relying Party Identifier** (Identificateur de partie de confiance), accédez au portail Azure puis, dans la page d’intégration de l’application **PureCloud by Genesys** , sélectionnez l’onglet **Propriétés** et copiez la valeur **ID de l’application** . Collez ensuite cette valeur dans la zone **Relying Party Identifier** .

    ![Capture d’écran affichant le volet Propriétés dans lequel vous pouvez trouver la valeur de l’ID de l’application.](./media/purecloud-by-genesys-tutorial/configure06.png)

    e. Sélectionnez **Enregistrer** .

### <a name="create-purecloud-by-genesys-test-user"></a>Créer un utilisateur de test PureCloud by Genesys

Pour permettre aux utilisateurs Azure AD de se connecter à PureCloud by Genesys, vous devez les provisionner dans PureCloud by Genesys. Dans PureCloud by Genesys, le provisionnement se fait manuellement.

**Pour configurer un compte d’utilisateur, effectuez ces étapes :**

1. Connectez-vous à PureCloud by Genesys en tant qu’administrateur.

1. Sélectionnez **Admin** en haut de la fenêtre, puis accédez à la section **People** (Personnes) sous **People & Permissions** (Personnes & Autorisations).

    ![Capture d’écran montrant la fenêtre PureCloud Admin, dans laquelle vous pouvez sélectionner People.](./media/purecloud-by-genesys-tutorial/configure03.png)

1. Dans la page **People** , sélectionnez **Add Person** (Ajouter une personne).

    ![Capture d’écran affichant la page People dans laquelle vous pouvez ajouter une personne.](./media/purecloud-by-genesys-tutorial/configure04.png)

1. Dans la boîte de dialogue **Add People to the Organization** (Ajouter des personnes à l’organisation), effectuez les étapes suivantes :

    ![Capture d’écran montrant la page dans laquelle vous pouvez indiquer les valeurs décrites.](./media/purecloud-by-genesys-tutorial/configure05.png)

    a. Dans la zone **Full Name** (Nom complet), entrez le nom d’un utilisateur. Par exemple : **B.simon** .

    b. Dans la zone **Email** , entrez l’adresse e-mail de l’utilisateur. Par exemple : **b.simon\@contoso.com** .

    c. Sélectionnez **Create** (Créer).

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous sélectionnez la vignette **PureCloud by Genesys** dans le volet d’accès, vous devez être connecté automatiquement au compte PureCloud by Genesys pour lequel vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste des tutoriels sur l’intégration d’applications SaaS avec Azure AD](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure AD ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure AD ?](../conditional-access/overview.md)

- [Essayer PureCloud by Genesys avec Azure AD](https://aad.portal.azure.com/)