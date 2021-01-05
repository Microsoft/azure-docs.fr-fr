---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à IntelligenceBank | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et IntelligenceBank.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/15/2020
ms.author: jeedes
ms.openlocfilehash: 9887c435c2aa8dc7ba8cab9481cf30195c4b334e
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92459932"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-intelligencebank"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à IntelligenceBank

Dans ce tutoriel, vous allez apprendre à intégrer IntelligenceBank à Azure Active Directory (Azure AD). Quand vous intégrez IntelligenceBank à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à IntelligenceBank.
* Permettre à vos utilisateurs de se connecter automatiquement à IntelligenceBank avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement IntelligenceBank pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* IntelligenceBank prend en charge l’authentification unique initiée par le **fournisseur de services** .

* Après avoir configuré IntelligenceBank, vous pouvez appliquer le contrôle de session, qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-intelligencebank-from-the-gallery"></a>Ajout d’IntelligenceBank à partir de la galerie

Pour configurer l’intégration d’IntelligenceBank à Azure AD, vous devez ajouter IntelligenceBank à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **IntelligenceBank** dans la zone de recherche.
1. Sélectionnez **IntelligenceBank** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-intelligencebank"></a>Configurer et tester l’authentification unique Azure AD pour IntelligenceBank

Configurez et testez l’authentification unique Azure AD avec IntelligenceBank à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur IntelligenceBank associé.

Pour configurer et tester l’authentification unique Azure AD avec IntelligenceBank, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique IntelligenceBank](#configure-intelligencebank-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test IntelligenceBank](#create-intelligencebank-test-user)** pour avoir un équivalent de B.Simon dans IntelligenceBank lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **IntelligenceBank** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<SUBDOMAIN>.intelligencebank.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , utilisez l’un quelconque des éléments suivants :

    - `IB`
    - `IntelligenceBank`
    - `https://<SUBDOMAIN>.intelligencebank.com`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<SUBDOMAIN>.intelligencebank.com/auth`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez l’[équipe de support client IntelligenceBank](mailto:helpdesk@intelligencebank.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer IntelligenceBank** , copiez la ou les URL appropriées selon vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à IntelligenceBank.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **IntelligenceBank** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-intelligencebank-sso"></a>Configurer l’authentification unique IntelligenceBank

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise IntelligenceBank en tant qu’administrateur.

1. Cliquez sur **Authentificateur** et cliquez sur **Ajouter nouveau** .

    ![Capture d’écran montrant l’onglet Administrator sélectionné ainsi que l’icône Add New.](./media/intelligencebank-tutorial/authenticator.PNG)

1. Procédez comme suit :

    ![Capture d’écran montrant les champs dans lesquels vous entrez les informations à cette étape.](./media/intelligencebank-tutorial/urls.PNG)

    a. Dans la zone de texte **Nom** , entrez le nom, par exemple `azureadsso`.

    b. Dans la zone de texte **Description** , entrez une description valide.

    c. Sélectionnez **SAML** dans la liste déroulante comme **Type** .

    d. Dans la zone de texte **URL distante** , collez la valeur d’ **URL de connexion** que vous avez copiée sur le portail Azure.

    e. Dans la zone de texte **Hôte** , collez la valeur d’ **ID d’entité** que vous avez copiée sur le portail Azure.

    f. Ouvrez le **certificat (base64)** téléchargé à partir du portail Azure dans le Bloc-notes et collez le contenu dans la zone de texte **CertData** .

    g. Dans la zone de texte **SingleLogoutService** (URL de déconnexion unique), collez la valeur d’ **URL de déconnexion** que vous avez copiée sur le portail Azure.

    h. Cliquez sur le bouton **Enregistrer** .

### <a name="create-intelligencebank-test-user"></a>Créer un utilisateur de test IntelligenceBank

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise IntelligenceBank en tant qu’administrateur.

1. Accédez à **Administrateur** -> **Utilisateurs** et sélectionnez l’icône **Ajouter un nouvel utilisateur** pour ajouter l’ **utilisateur** .

    ![Capture d’écran montrant l’icône Users sélectionnée sous l’onglet Users.](./media/intelligencebank-tutorial/creating-user.PNG)

1. Remplissez les champs nécessaires conformément aux exigences de votre organisation, puis cliquez sur **Enregistrer** .

    ![Capture d’écran montrant la page Add New User dans laquelle vous pouvez entrer des informations sur l’utilisateur.](./media/intelligencebank-tutorial/creating-user-1.PNG)

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette IntelligenceBank dans le volet d’accès, vous devez être connecté automatiquement à l’application IntelligenceBank pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer IntelligenceBank avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger IntelligenceBank avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)