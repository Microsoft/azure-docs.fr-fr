---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Pipedrive | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Pipedrive.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/06/2020
ms.author: jeedes
ms.openlocfilehash: f85cb97406e8b6cbb4811268696fc36f47ec3adb
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896546"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-pipedrive"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Pipedrive

Dans ce tutoriel, vous allez apprendre à intégrer Pipedrive à Azure Active Directory (Azure AD). Quand vous intégrez Pipedrive à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Pipedrive.
* Autoriser les utilisateurs à se connecter automatiquement à Pipedrive avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Pipedrive pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Pipedrive prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**
* Une fois que vous avez configuré l’authentification unique Pipedrive, vous pouvez appliquer le contrôle de session, qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).


## <a name="adding-pipedrive-from-the-gallery"></a>Ajout de Pipedrive à partir de la galerie

Pour configurer l’intégration de Pipedrive à Azure AD, vous devez ajouter Pipedrive à votre liste d’applications SaaS managées, à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **Pipedrive** dans la zone de recherche.
1. Sélectionnez **Pipedrive** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-pipedrive"></a>Configurer et tester l’authentification unique Azure AD pour Pipedrive

Configurez et testez l’authentification unique Azure AD avec Pipedrive pour un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans Pipedrive.

Pour configurer et tester l’authentification unique Azure AD avec Pipedrive, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Pipedrive](#configure-pipedrive-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Pipedrive](#create-pipedrive-test-user)** pour avoir un équivalent de B.Simon dans Pipedrive qui soit lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [Portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Pipedrive** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<COMPANY-NAME>.pipedrive.com/sso/auth/samlp/metadata.xml`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<COMPANY-NAME>.pipedrive.com/sso/auth/samlp`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<COMPANY-NAME>.pipedrive.com/`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique Pipedrive](mailto:support@pipedrive.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Votre application Pipedrive s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application Pipedrive s’attend à ce que quelques attributs supplémentaires soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source|
    | ------------ | --------- |
    | email | user.mail |

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur. De plus, copiez l’ **URL des métadonnées de fédération de l’application** et enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](./media/pipedrive-tutorial/certificate-data.png)

1. Dans la section **Configurer Pipedrive** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Pipedrive.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Pipedrive**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

## <a name="configure-pipedrive-sso"></a>Configurer l’authentification unique pour Pipedrive

1. Dans une autre fenêtre de navigateur, connectez-vous au site web Pipedrive en tant qu’administrateur.

1. Cliquez sur **User profile** (Profil utilisateur) et sélectionnez **Settings** (Paramètres).

    ![Capture d’écran montrant « Settings » sélectionné dans le menu « User profile »](./media/pipedrive-tutorial/configure1.png)

1. Faites défiler la page jusqu’à Security Center et sélectionnez **Single sign-on** (Authentification unique).

    ![Capture d’écran montrant « Single sign-on » sélectionné dans « Security Center »](./media/pipedrive-tutorial/configure2.png)

1. Dans la section **SAML configuration for pipedrive** (Configuration de SAML pour Pipedrive), effectuez les étapes suivantes :

    ![Capture d’écran montrant la section « SAML configuration for Pipedrive » avec toutes les zones de texte mises en évidence](./media/pipedrive-tutorial/configure3.png)

    a. Dans la zone de texte **Issuer** (Émetteur), collez la valeur du champ **URL des métadonnées de fédération d’application** que vous avez copiée du Portail Azure.

    b. Dans la zone de texte **Single Sign On (SSO) URL** (URL d’authentification unique), collez l’ **URL de connexion** que vous avez copiée du Portail Azure.

    c. Dans la zone de texte **Single Log Out (SLO) URL** (URL de déconnexion unique), collez l’ **URL de déconnexion** que vous avez copiée du Portail Azure.

    d. Dans la zone de texte **x.509 certificate** (Certificat x.509), ouvrez le fichier **Certificat (en base64)** téléchargé du portail Azure dans le Bloc-notes, copiez son contenu et collez-le dans la zone de texte **x.509 certificate** , puis enregistrez les changements apportés.

### <a name="create-pipedrive-test-user"></a>Créer un utilisateur de test Pipedrive

1. Dans une autre fenêtre de navigateur, connectez-vous au site web Pipedrive en tant qu’administrateur.

1. Faites défiler la page jusqu’à la section de l’entreprise et sélectionnez **Manage users** (Gérer les utilisateurs).

    ![Capture d’écran montrant « Manage users » sélectionné dans le menu « Company »](./media/pipedrive-tutorial/user1.png)

1. Cliquez sur **Add Users** (Ajouter des utilisateurs).
    
    ![Capture d’écran montrant la page « Manage users » avec le bouton « Add users » sélectionné sur le côté droit](./media/pipedrive-tutorial/user2.png)

1. Dans la section **Manage users** , effectuez les étapes suivantes :

    ![Configuration de Pipedrive](./media/pipedrive-tutorial/user3.png)

    a. Dans la zone de texte **Email** (E-mail), entrez l’adresse e-mail de l’utilisateur, par exemple `B.Simon@contoso.com`.

    b. Dans la zone de texte **First Name** , entrez le prénom de l’utilisateur.

    c. Dans la zone de texte **Nom** , entrez le nom de l’utilisateur.

    d. Cliquez sur **Confirm and invite users** (Confirmer et inviter les utilisateurs).

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Pipedrive dans le volet d’accès, vous êtes automatiquement connecté à l’application Pipedrive pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayez Pipedrive avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)