---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Samsara | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Samsara.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/15/2020
ms.author: jeedes
ms.openlocfilehash: 9512287f656c6c64399cb8749b7451a5a780bba8
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92677641"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-samsara"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Samsara

Dans ce tutoriel, vous allez apprendre à intégrer Samsara à Azure Active Directory (Azure AD). Quand vous intégrez Samsara à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Samsara.
* Permettre à vos utilisateurs de se connecter automatiquement à Samsara avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Samsara pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Samsara prend en charge l’authentification unique lancée par le **fournisseur de services** et le **fournisseur d’identité** .
* Samsara prend en charge le provisionnement d’utilisateurs **juste-à-temps** .

## <a name="adding-samsara-from-the-gallery"></a>Ajout de Samsara à partir de la galerie

Pour configurer l’intégration de Samsara à Azure AD, vous devez ajouter Samsara à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)
    
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .

    ![Bouton Nouvelle application](common/add-new-app.png)

1. Dans la section **Ajouter à partir de la galerie** , tapez **Samsara** dans la zone de recherche.

     ![OneTrust Privacy Management Software dans la liste des résultats](common/search-new-app.png)

1. Sélectionnez **Samsara** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-samsara"></a>Configurer et tester l’authentification unique Azure AD pour Samsara

Configurez et testez l’authentification unique Azure AD avec Samsara pour un utilisateur de test nommé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Samsara associé.

Pour configurer et tester l’authentification unique Azure AD avec Samsara, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Samsara](#configure-samsara-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Samsara](#create-samsara-test-user)** pour avoir, dans Samsara, un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Samsara** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL d’authentification** , tapez une URL au format suivant : `https://cloud.samsara.com/signin/<ORGID>` pour les clients cloud aux États-Unis `https://cloud.eu.samsara.com/signin/<ORGID>` pour les clients cloud en Europe

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `urn:auth0:samsara-dev:samlp-orgid-<ORGID>`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://samsara-dev.auth0.com/login/callback?connection=samlp-orgid-<ORGID>`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’URL de réponse et l’identificateur réels. Contactez l’ [équipe du support technique Samsara](mailto:support@samsara.com) pour obtenir ces valeurs, ou dans Samsara, accédez à **Settings** (Paramètres) > **Single-Sign-On** (Authentification unique) > **New SAML Connection** (Nouvelle connexion SAML) pour obtenir le \<ORGID\>. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Samsara** , copiez l’ **URL de connexion** .

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Samsara.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Samsara** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .
1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .
1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-samsara-sso"></a>Configurer l’authentification unique Samsara

Pour configurer l’authentification unique côté **Samsara** , vous devez envoyer le **certificat (Base64)** téléchargé et l’ **URL de connexion** du portail Azure à l’ [équipe du support technique Samsara](mailto:support@samsara.com). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-samsara-test-user"></a>Créer un utilisateur de test Samsara

Dans cette section, un utilisateur appelé B.Simon est créé dans Samsara. Samsara prend en charge le provisionnement d’utilisateurs juste-à-temps, lequel est activé par défaut. Vous n’avez aucune opération à effectuer dans cette section. Si l’utilisateur n’existe pas déjà dans Samsara, il est créé après l’authentification avec le rôle par défaut Standard Admin (No Dash Cam Access) pour l’organisation. L’accès de l’utilisateur peut ensuite être revu à la hausse ou à la baisse en fonction des besoins dans Samsara.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

1. Cliquez sur **Tester cette application** dans le portail Azure. Cette opération redirige vers l’URL de connexion Samsara où vous pouvez lancer le processus de connexion. 

2. Accédez directement à l’URL de connexion Samsara pour lancer le processus de connexion.

3. Vous pouvez utiliser le volet d’accès Microsoft. Le fait de cliquer sur la vignette Samsara dans le volet d’accès vous redirige vers l’URL de connexion Samsara. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Samsara, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).