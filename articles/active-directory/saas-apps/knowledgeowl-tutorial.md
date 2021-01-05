---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à KnowledgeOwl | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et KnowledgeOwl.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/14/2019
ms.author: jeedes
ms.openlocfilehash: 367f6713102912786ce258c471278373636f7326
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92458912"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-knowledgeowl"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à KnowledgeOwl

Dans ce tutoriel, vous allez découvrir comment intégrer KnowledgeOwl à Azure Active Directory (Azure AD). Quand vous intégrez KnowledgeOwl à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à KnowledgeOwl.
* Permettre à vos utilisateurs de se connecter automatiquement à KnowledgeOwl avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement KnowledgeOwl pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* KnowledgeOwl prend en charge l’authentification unique lancée par **le fournisseur de services et le point de distribution d’émission**
* KnowledgeOwl prend en charge l’attribution d’utilisateurs **Juste-à-temps**

## <a name="adding-knowledgeowl-from-the-gallery"></a>Ajout de KnowledgeOwl à partir de la galerie

Pour configurer l’intégration de KnowledgeOwl à Azure AD, vous devez ajouter KnowledgeOwl à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **KnowledgeOwl** dans la zone de recherche.
1. Sélectionnez **KnowledgeOwl** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-knowledgeowl"></a>Configurer et tester l’authentification unique Azure AD pour KnowledgeOwl

Configurez et testez l’authentification unique Azure AD avec KnowledgeOwl à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur KnowledgeOwl associé.

Pour configurer et tester l’authentification unique Azure AD avec KnowledgeOwl, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique KnowledgeOwl](#configure-knowledgeowl-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test KnowledgeOwl](#create-knowledgeowl-test-user)** pour avoir un équivalent de B.Simon dans KnowledgeOwl, lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **KnowledgeOwl** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant :
    
    ```http
    https://app.knowledgeowl.com/sp
    https://app.knowledgeowl.com/sp/id/<unique ID>
    ```

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant :
    
    ```http
    https://subdomain.knowledgeowl.com/help/saml-login
    https://subdomain.knowledgeowl.com/docs/saml-login
    https://subdomain.knowledgeowl.com/home/saml-login
    https://privatedomain.com/help/saml-login
    https://privatedomain.com/docs/saml-login
    https://privatedomain.com/home/saml-login
    ```

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL d’authentification** , tapez une URL au format suivant :
    
    ```http
    https://subdomain.knowledgeowl.com/help/saml-login
    https://subdomain.knowledgeowl.com/docs/saml-login
    https://subdomain.knowledgeowl.com/home/saml-login
    https://privatedomain.com/help/saml-login
    https://privatedomain.com/docs/saml-login
    https://privatedomain.com/home/saml-login
    ```

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Vous devez changer cette valeur pour l’identificateur, l’URL de réponse et l’URL de connexion réels. Ceci est expliqué plus loin dans le tutoriel.

1. L’application KnowledgeOwl attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jetons SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application KnowledgeOwl s’attend à ce que quelques attributs supplémentaires soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source | Espace de noms |
    | ------------ | -------------------- | -----|
    | ssoid | user.mail | `http://schemas.xmlsoap.org/ws/2005/05/identity/claims`|

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (brut)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificateraw.png)

1. Dans la section **Configurer KnowledgeOwl** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à KnowledgeOwl.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **KnowledgeOwl** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-knowledgeowl-sso"></a>Configurer l’authentification unique KnowledgeOwl

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise KnowledgeOwl en tant qu’administrateur.

1. Cliquez sur **Paramètres** , puis sélectionnez **Sécurité** .

    ![Capture d’écran montrant l’élément Security sélectionné dans le menu Settings.](./media/knowledgeowl-tutorial/configure1.png)

1. Faites défiler jusqu’à **Intégration de l'authentification unique SAML** et effectuez les étapes suivantes :

    ![Capture d’écran montrant la zone d’intégration de l’authentification unique SAML dans laquelle vous pouvez apporter les modifications décrites ici.](./media/knowledgeowl-tutorial/configure2.png)

    a. Sélectionnez **Enable SAML SSO** (Activer l’authentification unique SAML).

    b. Copiez la valeur **ID d’entité du fournisseur de services** et collez-la dans la zone de texte **Identificateur (ID d’entité)** de la section **Configuration SAML de base** du Portail Azure.

    c. Copiez la valeur de l’ **URL de connexion SP** et collez-la dans la zone de texte **URL de connexion et URL de réponse** de la section **Configuration SAML de base** du Portail Azure.

    d. Dans la zone de texte **ID d’entité** , collez la valeur **Identificateur Azure AD** que vous avez copiée dans le Portail Azure.

    e. Dans la zone de texte **Identity Provider Login URL** , collez la valeur d’ **URL de connexion** que vous avez copiée sur le Portail Azure.

    f. Dans la zone de texte **Identity Provider Login URL** , collez la valeur d’ **URL de connexion** que vous avez copiée sur le Portail Azure

    g. Chargez le certificat téléchargé à partir du portail Azure en cliquant sur **Upload IdP Certificate (Charger un certificat IdP)** .

    h. Cliquez sur **Map SAML Attributes (Mapper les attributs SAML)** pour mapper les attributs, puis procédez comme suit :

    ![Capture d’écran montrant l’élément Map SAML Attributes, dans lequel vous pouvez apporter les modifications décrites ici.](./media/knowledgeowl-tutorial/configure3.png)

    * Dans la zone de texte **SSO ID** (ID d’authentification unique), tapez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/ssoid`.
    * Dans la zone de texte **Username/Email** (Nom d’utilisateur/E-mail), tapez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`.
    * Dans la zone de texte **First Name** (Prénom), tapez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`.
    * Dans la zone de texte **Last Name** (Prénom), tapez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`.
    * Cliquez sur **Enregistrer** .

    i. Cliquez sur **Enregistrer** au bas de la page.

    ![Capture d’écran montrant le bouton Save.](./media/knowledgeowl-tutorial/configure4.png)

### <a name="create-knowledgeowl-test-user"></a>Créer un utilisateur de test KnowledgeOwl

Dans cette section, un utilisateur appelé B.Simon est créé dans KnowledgeOwl. KnowledgeOwl prend en charge l’attribution d’utilisateurs juste-à-temps, une option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. Si l’utilisateur souhaité n’existe pas déjà dans KnowledgeOwl, il est créé après l’authentification.

> [!Note]
> Si vous devez créer un utilisateur manuellement, contactez [l’équipe de support KnowledgeOwl](mailto:support@knowledgeowl.com).

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette KnowledgeOwl dans le volet d’accès, vous devriez être connecté automatiquement à l’application KnowledgeOwl pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer KnowledgeOwl avec Azure AD](https://aad.portal.azure.com/)