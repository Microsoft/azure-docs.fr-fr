---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à WhosOnLocation | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et WhosOnLocation.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/30/2020
ms.author: jeedes
ms.openlocfilehash: d0e5134da9083e97b3977b05d601c2cfba25f5d4
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92636678"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-whosonlocation"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à WhosOnLocation

Dans ce tutoriel, vous allez apprendre à intégrer WhosOnLocation à Azure Active Directory (Azure AD). Quand vous intégrez WhosOnLocation à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à WhosOnLocation.
* Permettre à vos utilisateurs de se connecter automatiquement à WhosOnLocation avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement WhosOnLocation pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* WhosOnLocation prend en charge l’authentification unique lancée par le **fournisseur de services**

* Après avoir configuré WhosOnLocation, vous pouvez appliquer le contrôle de session, qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-whosonlocation-from-the-gallery"></a>Ajout de WhosOnLocation à partir de la galerie

Pour configurer l’intégration de WhosOnLocation avec Azure AD, vous devez ajouter WhosOnLocation, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , entrez **WhosOnLocation**  dans la zone de recherche.
1. Sélectionnez **WhosOnLocation** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-whosonlocation"></a>Configurer et tester l’authentification unique (SSO) Azure AD pour WhosOnLocation

Configurez et testez l’authentification unique (SSO) Azure AD avec WhosOnLocation à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur associé dans WhosOnLocation.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec WhosOnLocation, effectuez les modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique WhosOnLocation](#configure-whosonlocation-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test WhosOnLocation](#create-whosonlocation-test-user)** pour avoir, dans WhosOnLocation, un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Sur le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **WhosOnLocation** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://login.whosonlocation.com/saml/login/<CUSTOM_ID>`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://login.whosonlocation.com/saml/metadata/<CUSTOM_ID>`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://login.whosonlocation.com/saml/acs/<CUSTOM_ID>`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’URL de réponse et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe du support technique WhosOnLocation](mailto:support@whosonlocation.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer WhosOnLocation** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à WhosOnLocation.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **WhosOnLocation**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

## <a name="configure-whosonlocation-sso"></a>Configurer l’authentification unique WhosOnLocation

1. Dans une autre fenêtre de navigateur, connectez-vous à votre site d’entreprise WhosOnLocation en tant qu’administrateur.

2. Cliquez sur **Outils** -> **Compte**.

    ![Capture d’écran montrant l’élément Compte sélectionné dans le menu Outils du site WhosOnLocation](./media/WhosOnLocation-tutorial/config1.png)

3. Dans le navigateur de gauche, sélectionnez **Accès des employés**.

    ![Capture d’écran montrant l’élément Accès des employés sélectionné dans le profil de compte](./media/WhosOnLocation-tutorial/config2.png)

4. Effectuez les étapes ci-après dans la page suivante.

    ![Capture d’écran montrant l’onglet Accès des employés où vous pouvez entrer des données utilisateur](./media/WhosOnLocation-tutorial/config3.png)

    a. Définissez **Configurer l’authentification unique avec SAML** sur **Oui**.

    b. Dans la zone de texte **URL de l’émetteur** , collez la valeur **ID d’entité** que vous avez copiée à partir du portail Azure.

    c. Dans la zone de texte **Point de terminaison SSO** , collez la valeur **URL de connexion** que vous avez copiée à partir du portail Azure.

    d. Ouvrez le **Certificat (en base64)** téléchargé à partir du portail Azure dans le Bloc-notes, puis collez le contenu dans la zone de texte **Certificat**.

    e. Cliquez sur **Enregistrer la configuration SAML**.

### <a name="create-whosonlocation-test-user"></a>Créer un utilisateur de test WhosOnLocation

Dans cette section, vous allez créer un utilisateur appelé B.Simon dans WhosOnLocation. Travaillez en collaboration avec l’[équipe de support technique WhosOnLocation](mailto:support@whosonlocation.com) pour ajouter des utilisateurs dans la plateforme WhosOnLocation. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette WhosOnLocation dans le volet d’accès, vous êtes automatiquement connecté à l’application WhosOnLocation pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer WhosOnLocation avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger WhosOnLocation avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)