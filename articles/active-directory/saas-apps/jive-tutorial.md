---
title: 'Tutoriel : Intégration d’Azure Active Directory à Jive | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Jive.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/16/2020
ms.author: jeedes
ms.openlocfilehash: 3d15e5e13b2b0defe45fd45450aee7c262052b4f
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92459415"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-jive"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Jive

Dans ce tutoriel, vous allez apprendre à intégrer Jive à Azure Active Directory (Azure AD). Quand vous intégrez Jive à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Jive.
* Permettre à vos utilisateurs de se connecter automatiquement à Jive avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Jive pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Jive prend en charge l’authentification unique lancée par le **fournisseur de services** .
* Jive prend en charge l’ [attribution d’utilisateurs **automatisé**](jive-provisioning-tutorial.md).
* Après avoir configuré Jive, vous pouvez appliquer des contrôles de session qui protègent l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Les contrôles de session sont étendus à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)

## <a name="adding-jive-from-the-gallery"></a>Ajout de Jive depuis la galerie

Pour configurer l’intégration de Jive avec Azure AD, vous devez ajouter Jive disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Jive** dans la zone de recherche.
1. Sélectionnez **Jive** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-jive"></a>Configurer et tester l’authentification unique Azure AD pour Jive

Configurez et testez l’authentification unique Azure AD avec Jive à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Jive associé.

Pour configurer et tester l’authentification unique Azure AD avec Jive, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Jive](#configure-jive-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Jive](#create-jive-test-user)** pour avoir un équivalent de B.Simon dans Jive lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Jive** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

   a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<instance name>.jivecustom.com`

   b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant :
   ```http
   https://<instance name>.jiveon.com
   ```

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique Jive](https://www.jivesoftware.com/services-support/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configurer Jive** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Jive.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Jive** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-jive-sso"></a>Configurer l’authentification unique Jive

1. Pour configurer l’authentification unique côté **Jive** , connectez-vous au locataire Jive en tant qu’administrateur.

1. Dans le menu du haut, cliquez sur **SAML** .

    ![Capture d’écran montrant l’onglet SAML avec l’élément Enabled sélectionné.](./media/jive-tutorial/tutorial_jive_002.png)

    a. Sélectionnez **Activé** sous l’onglet **Général** .

    b. Cliquez sur le bouton **SAVE ALL SAML SETTINGS** (Enregistrer tous les paramètres SAML).

1. Accédez à l’onglet **IDP METADATA** (Métadonnées du fournisseur d’identité).

    ![Capture d’écran montrant l’onglet SAML avec l’élément IDP METADATA sélectionné.](./media/jive-tutorial/tutorial_jive_003.png)

    a. Copiez le contenu du fichier XML de métadonnées téléchargé et collez-le dans la zone de texte **Métadonnées du fournisseur d'identité (IDP)** .

    b. Cliquez sur le bouton **SAVE ALL SAML SETTINGS** (Enregistrer tous les paramètres SAML).

1. Sélectionnez l’onglet **USER ATTRIBUTE MAPPING** (Mappage d’attributs utilisateur).

    ![Capture d’écran montrant l’onglet SAML avec l’élément USER ATTRIBUTE MAPPING sélectionné.](./media/jive-tutorial/tutorial_jive_004.png)

    a. Dans la zone de texte **E-mail** , copiez et collez le nom de la valeur **mail** .

    b. Dans la zone de texte **Prénom** , copiez et collez le nom de la valeur **givenname** .

    c. Dans la zone de texte **Nom** , copiez et collez le nom de la valeur **surname** .

### <a name="create-jive-test-user"></a>Créer un utilisateur de test Jive

L’objectif de cette section est de créer un utilisateur appelé Britta Simon dans Jive. Jive prend en charge l’approvisionnement automatique d’utilisateurs, option activée par défaut. Vous trouverez plus d’informations [ici](jive-provisioning-tutorial.md) sur la façon de configurer l’attribution automatique d’utilisateurs.

Si vous avez besoin de créer un utilisateur manuellement, collaborez avec [l’équipe du support du client Jive](https://www.jivesoftware.com/services-support/) pour ajouter des utilisateurs dans la plateforme Jive.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Jive dans le volet d’accès, vous devez être connecté automatiquement à l’application Jive pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Jive avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Configurer l’approvisionnement de l’utilisateur](jive-provisioning-tutorial.md)

- [Guide pratique pour protéger Jive avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)