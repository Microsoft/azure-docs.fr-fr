---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à InVision | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et InVision.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/09/2020
ms.author: jeedes
ms.openlocfilehash: 1be5a46afbaa248232ff113cbfb45a8798af77c1
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92459864"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-invision"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à InVision

Dans ce tutoriel, vous allez apprendre à intégrer InVision à Azure Active Directory (Azure AD). Quand vous intégrez InVision à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à InVision.
* Autoriser les utilisateurs à se connecter automatiquement à InVision avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement InVision pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* InVision prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité**
* Après avoir configuré InVision, vous pouvez appliquer le contrôle de session qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-invision-from-the-gallery"></a>Ajout d’InVision à partir de la galerie

Pour configurer l’intégration d’InVision à Azure AD, vous devez ajouter InVision à votre liste d'applications SaaS managées, à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **InVision** dans la zone de recherche.
1. Sélectionnez **InVision** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-invision"></a>Configurer et tester l’authentification unique Azure AD pour InVision

Configurez et testez l’authentification unique Azure AD avec InVision pour un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur InVision associé.

Pour configurer et tester l’authentification unique Azure AD avec InVision, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique InVision](#configure-invision-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test InVision](#create-invision-test-user)** pour avoir un équivalent de B.Simon dans InVision lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **InVision** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<SUBDOMAIN>.invisionapp.com`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<SUBDOMAIN>.invisionapp.com//sso/auth`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<SUBDOMAIN>.invisionapp.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez l’[équipe du support technique InVision](mailto:support@invisionapp.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer InVision** , copiez la ou les URL appropriées selon vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à InVision.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **InVision** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-invision-sso"></a>Configurer l’authentification unique InVision

1. Dans une autre fenêtre du navigateur web, connectez-vous au site InVision en tant qu’administrateur.

1. Cliquez sur **Team** (Équipe), puis sélectionnez **Settings** (Paramètres).

    ![Capture d’écran montrant l’onglet Team avec l’élément Settings, tous deux sélectionnés.](./media/invision-tutorial/config1.png)

1. Faites défiler la page jusqu’à **Single sign-on** (Authentification unique) et cliquez sur **Change** (Changer).

    ![Capture d’écran montrant le bouton Change pour Single Sign-On.](./media/invision-tutorial/config3.png)

1. Dans la page **Single sign-on** (Authentification unique), effectuez les étapes suivantes :

    ![Capture d’écran montrant la page Single Sign-On dans laquelle vous pouvez entrer les valeurs de cette étape.](./media/invision-tutorial/config4.png)

    a. Changez **Require SSO for every member of < account name >** (Exiger l’authentification unique pour chaque membre de < nom du compte >) sur **On** (Activé).

    b. Dans la zone de texte **Name** , entrez le nom, par exemple `azureadsso`.

    c. Entrez la valeur de l’URL de connexion dans la zone de texte **Sign-in URL** .

    d. Dans la zone de texte **Sign-Out URL** (URL de déconnexion), collez la valeur du champ **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    e. Dans la zone de texte **SAML Certificate** (Certificat SAML), ouvrez le fichier **Certificat (en base64)** téléchargé du portail Azure dans le Bloc-notes, copiez son contenu et collez-le dans la zone de texte SAML Certificate.

    f. Dans la zone de texte **Name** **ID Format** , utilisez `Unspecified`.

    g. Sélectionnez **SHA-256** dans la liste déroulante **HASH Algorithm** (Algorithme de hachage).

    h. Dans **SSO Button Label** , entrez un nom comme étiquette du bouton SSO.

    i. Activez **Allow Just-in-Time provisioning** (Autoriser le provisionnement juste-à-temps).

    j. Cliquez sur **Update** .

### <a name="create-invision-test-user"></a>Créer un utilisateur de test InVision

1. Dans une autre fenêtre du navigateur web, connectez-vous au site InVision en tant qu’administrateur.

1. Cliquez sur **Team** (Équipe), puis sélectionnez **People** (Personnes).

    ![Capture d’écran montrant l’onglet Team avec l’élément People, tous deux sélectionnés.](./media/invision-tutorial/config2.png)

1. Cliquez sur l’ **ICÔNE +** pour ajouter un nouvel utilisateur.

    ![Capture d’écran montrant l’icône + pour ajouter un utilisateur.](./media/invision-tutorial/user1.png)

1. Entrez l’adresse e-mail de l’utilisateur, puis cliquez sur **Next** (Suivant).

    ![Capture d’écran montrant la boîte de dialogue Invite to dans laquelle vous pouvez entrer des adresses.](./media/invision-tutorial/user2.png)

1. Une fois l’adresse e-mail vérifiée, cliquez sur **Invite** (Inviter).

    ![Capture d’écran montrant la boîte de dialogue Invite, dans laquelle vous pouvez sélectionner Invite pour continuer.](./media/invision-tutorial/user3.png)

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la vignette InVision dans le volet d’accès, vous êtes automatiquement connecté à l’application InVision pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayez InVision avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger InVision avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)