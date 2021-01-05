---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Coda | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Coda.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/23/2020
ms.author: jeedes
ms.openlocfilehash: c78744538991e337b226e49a053807ef8174d507
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92455729"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-coda"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Coda

Dans ce tutoriel, vous allez apprendre à intégrer Coda à Azure Active Directory (Azure AD). Quand vous intégrez Coda à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Coda.
* Permettre à vos utilisateurs de se connecter automatiquement à Coda avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement CODA pour lequel l’authentification unique est activée et l’intégration GDrive est désactivée. Contactez l’[équipe du support technique de Coda](mailto:support@coda.io) afin de désactiver l’intégration de GDrive pour votre organisation si elle est actuellement activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Coda prend en charge l’authentification unique lancée par le **fournisseur d’identité**

* Coda prend en charge le provisionnement d’utilisateurs **juste-à-temps**

* Après avoir configuré Coda, vous pouvez appliquer des contrôles de session qui protègent l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Les contrôles de session sont étendus à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-coda-from-the-gallery"></a>Ajout de Coda depuis la galerie

Pour configurer l’intégration de Coda à Azure AD, vous devez ajouter Coda depuis la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Coda** dans la zone de recherche.
1. Sélectionnez **Coda** dans le panneau de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-coda"></a>Configurer et tester l’authentification unique Azure AD pour Coda

Configurez et testez l’authentification unique Azure AD avec Coda à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Coda associé.

Pour configurer et tester l’authentification unique Azure AD avec Coda, suivez les indications des sections ci-après :

1. **[Commencer la configuration de l’authentification unique Coda](#begin-configuration-of-coda-sso)** pour commencer à configurer l’authentification unique dans Coda.
1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
   * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
   * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Coda](#configure-coda-sso)** pour effectuer la configuration des paramètres d’authentification unique dans Coda.
   * **[Créer un utilisateur de test Coda](#create-coda-test-user)** pour avoir un équivalent de B.Simon dans Coda lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="begin-configuration-of-coda-sso"></a>Commencer la configuration de l’authentification unique Coda

Suivez ces étapes dans Coda pour commencer.

1. Dans Coda, ouvrez le panneau **Organization settings** (Paramètres de l’organisation).

   ![Ouvrez les paramètres d’organisation](media/coda-tutorial/org-settings.png)

1. Vérifiez que l’intégration GDrive est désactivée dans votre organisation. Si elle est activée, contactez l’[équipe du support technique de Coda](mailto:support@coda.io) pour vous aider à migrer hors de GDrive.

   ![GDrive désactivé](media/coda-tutorial/gdrive-off.png)

1. Sous **Authenticate with SSO (SAML)** , (S’authentifier avec l’authentification unique (SAML)), sélectionnez l’option **Configure SAML** (Configurer SAML).

   ![Paramètres SAML](media/coda-tutorial/saml-settings-link.png)

1. Notez les valeurs **Entity ID** (ID d’entité) et **SAML Response URL** (URL de réponse SAML), dont vous aurez besoin lors des étapes suivantes.

   ![ID d’entité et URL de réponse SAML à utiliser dans Azure](media/coda-tutorial/azure-settings.png)

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans la page d’intégration de l’application **Coda** du [portail Azure](https://portal.azure.com/), recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la page **Configurer l’authentification unique avec SAML** , entrez les valeurs pour les champs suivants :

   a. Dans la zone de texte **Identificateur** , entrez la valeur « Entity ID » obtenue ci-dessus. Elle doit suivre ce modèle : `https://coda.io/samlId/<CUSTOMID>`

   b. Dans la zone de texte **URL de réponse** , entrez la valeur « SAML Response URL » obtenue ci-dessus. Elle doit suivre ce modèle : `https://coda.io/login/sso/saml/<CUSTOMID>/consume`

   > [!NOTE]
   > Vos valeurs seront différentes de celles ci-dessus ; vous les trouverez dans la console « Configure SAML » de Coda. Mettez à jour ces valeurs avec l’identificateur et l’URL de réponse réels.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (Base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

   ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Coda** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Coda.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Coda** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

   ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-coda-sso"></a>Configurer l’authentification unique Coda

Pour terminer la configuration, vous allez entrer des valeurs obtenues à partir d’Azure Active Directory dans le panneau **Configure SAML** de Coda.

1. Dans Coda, ouvrez le panneau **Organization settings** .
1. Sous **Authenticate with SSO (SAML)** , sélectionnez l’option **Configure SAML** .
1. Affectez la valeur **Azure Active Directory** à **SAML Provider** (Fournisseur SAML).
1. Dans **Identity Provider Login URL** (URL de connexion au fournisseur d’identité), collez l’ **URL de connexion** obtenue à partir de la console Azure.
1. Dans **Identity Provider Issuer** (Émetteur du fournisseur d’identité), collez l’ **Identificateur Azure AD** obtenu à partir de la console Azure.
1. Dans **Identity Provider Public Certificate** (Certificat public du fournisseur d’identité), sélectionnez l’option **Upload Certificate** (Charger le certificat) et sélectionnez le fichier de certificat que vous avez téléchargé.
1. Sélectionnez **Enregistrer** .

Cela termine le travail nécessaire pour la configuration de la connexion d’authentification unique SAML.

### <a name="create-coda-test-user"></a>Créer un utilisateur de test Coda

Dans cette section, un utilisateur appelé Britta Simon est créé dans Coda. Coda prend en charge le provisionnement d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Coda, il en est créé un après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Coda dans le volet d’accès, vous devez être connecté automatiquement à l’application Coda pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Coda avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Coda avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)