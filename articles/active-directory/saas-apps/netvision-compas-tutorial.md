---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Netvision Compas | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Netvision Compas.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/16/2020
ms.author: jeedes
ms.openlocfilehash: 423ac882c81582f2843eeba37d11c660662ad6e2
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92519399"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-netvision-compas"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Netvision Compas

Dans ce tutoriel, vous allez apprendre à intégrer Netvision Compas à Azure Active Directory (Azure AD). Quand vous intégrez Netvision Compas à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Netvision Compas.
* Permettre à vos utilisateurs de se connecter automatiquement à Netvision Compas avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Netvision Compas pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Netvision Compas prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité** .
* Après avoir configuré Netvision Compas, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)


## <a name="adding-netvision-compas-from-the-gallery"></a>Ajout de Netvision Compas depuis la galerie

Pour configurer l’intégration de Netvision Compas dans Azure AD, vous devez ajouter Netvision Compas à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Netvision Compas** dans la zone de recherche.
1. Sélectionnez **Netvision Compas** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-netvision-compas"></a>Configurer et tester l’authentification unique Azure AD pour Netvision Compas

Configurez et testez l’authentification unique Azure AD avec Netvision Compas à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Netvision Compas associé.

Pour configurer et tester l’authentification unique Azure AD avec Netvision Compas, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Netvision Compas](#configure-netvision-compas-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Configurer un utilisateur de test Netvision Compas](#configure-netvision-compas-test-user)** pour avoir un équivalent de B.Simon dans Netvision Compas lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Netvision Compas** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<TENANT>.compas.cloud/Identity/Saml20`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<TENANT>.compas.cloud/Identity/Auth/AssertionConsumerService`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<TENANT>.compas.cloud/Identity/Auth/AssertionConsumerService`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez l’[équipe de support client de Netvision Compas](mailto:contact@net.vision). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le fichier de métadonnées et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)



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

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Netvision Compas.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Netvision Compas** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-netvision-compas-sso"></a>Configurer l’authentification unique Netvision Compas

Dans cette section, vous allez activer l’authentification unique SAML dans **Netvision Compas** .
1. Connectez-vous à **Netvision Compas** avec un compte d’administrateur et accédez à la zone administration.

    ![Zone d’administration](media/netvision-compas-tutorial/admin.png)

1. Localisez la zone **Système** et sélectionnez **Fournisseurs d’identité** .

    ![Fournisseurs d’identité d’administrateur](media/netvision-compas-tutorial/admin-idps.png)

1. Sélectionnez l’action **Ajouter** pour inscrire Azure AD en tant que nouveau fournisseur d’identité.

    ![Ajouter un fournisseur d’identité](media/netvision-compas-tutorial/idps-add.png)

1. Pour **SAML** pour le **Type de fournisseur** .
1. Entrez des valeurs significatives pour les champs **Nom d’affichage** et **Description** .
1. Affectez des utilisateurs **Netvision Compas** au fournisseur d’identité en les sélectionnant dans la liste **Utilisateurs disponibles** , puis en sélectionnant le bouton **Ajouter la sélection** . Les utilisateurs peuvent également être affectés au fournisseur d’identité tout en suivant la procédure de provisionnement.
1. Pour l’option **Métadonnées** de SAML, cliquez sur le bouton **Choisir un fichier** et sélectionnez le fichier de métadonnées précédemment enregistré sur votre ordinateur.
1. Cliquez sur **Enregistrer** .

    ![Modifier le fournisseur d’identité](media/netvision-compas-tutorial/idp-edit.png)


### <a name="configure-netvision-compas-test-user"></a>Configurer un utilisateur de test Netvision Compas

Dans cette section, vous configurez un utilisateur existant dans **Netvision Compas** pour l’utilisation d’Azure AD pour l’authentification unique.
1. Suivez la procédure de provisionnement d’utilisateurs de **Netvision Compas** , telle qu’elle est définie par votre entreprise, ou bien modifiez un compte d’utilisateur existant.
1. Lors de la définition du profil de l’utilisateur, vérifiez que l’adresse **E-mail (personnel)** correspond au nom d’utilisateur d’Azure AD : username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.

    ![Edit User](media/netvision-compas-tutorial/user-config.png)

Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD.

### <a name="using-the-access-panel-idp-initiated"></a>Utilisation du volet d’accès (lancé par le fournisseur d’identité).

Quand vous cliquez sur la vignette Netvision Compas dans le volet d’accès, vous devez vous connecter automatiquement à l’application Netvision Compas pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

### <a name="directly-accessing-netvision-compas-sp-initiated"></a>Accès direct à Netvision Compas (lancé par le fournisseur de service).

1. Accédez à l’URL de **Netvision Compas** . Par exemple : `https://tenant.compas.cloud`.
1. Entrez le nom d’utilisateur de **Netvision Compas** , puis sélectionnez **Suivant** .

    ![Utilisateur de la connexion](media/netvision-compas-tutorial/login-user.png)

1. **(facultatif)** Si plusieurs fournisseurs sont affectés à l’utilisateur dans **Netvision Compas** , une liste des fournisseurs d’identité disponibles s’affiche. Sélectionnez le fournisseur d’identité Azure AD configuré précédemment dans **Netvision Compas** .

    ![Choix de la connexion](media/netvision-compas-tutorial/login-choose.png)

1. Vous êtes redirigé vers Azure AD pour effectuer l’authentification. Une fois que vous êtes authentifié, vous devez être connecté automatiquement à **Netvision Compas** pour lequel vous configurez l’authentification unique.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Netvision Compas avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)