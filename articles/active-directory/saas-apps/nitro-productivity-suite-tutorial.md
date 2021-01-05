---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Nitro Productivity Suite | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Nitro Productivity Suite.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/28/2020
ms.author: jeedes
ms.openlocfilehash: 3280c97e735e68aa36f018a8de59964ade9567b7
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96181904"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-nitro-productivity-suite"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Nitro Productivity Suite

Dans ce tutoriel, vous allez découvrir comment intégrer Nitro Productivity Suite à Azure Active Directory (Azure AD). Quand vous intégrez Nitro Productivity Suite à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Nitro Productivity Suite.
* Permettre à vos utilisateurs de se connecter automatiquement à Nitro Productivity Suite avec leur compte Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un [abonnement Enterprise](https://www.gonitro.com/pricing) à Nitro Productivity Suite.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Nitro Productivity Suite prend en charge l’authentification unique initiée par le **fournisseur de services** et le **fournisseur d’identité**.
* Nitro Productivity Suite prend en charge le provisionnement des utilisateurs **juste-à-temps**.

## <a name="add-nitro-productivity-suite-from-the-gallery"></a>Ajouter Nitro Productivity Suite à partir de la galerie

Pour configurer l’intégration de Nitro Productivity Suite dans Azure AD, vous devez ajouter Nitro Productivity Suite, disponible dans la galerie, à votre liste d’applications SaaS managées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire, ou avec un compte personnel Microsoft.
1. Sélectionnez **Azure Active Directory** dans le volet de gauche.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Nitro Productivity Suite** dans la zone de recherche.
1. Sélectionnez **Nitro Productivity Suite** dans les résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-nitro-productivity-suite"></a>Configurer et tester l’authentification unique Azure AD pour Nitro Productivity Suite

Configurez et testez l’authentification unique Azure AD avec Nitro Productivity Suite à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Nitro Productivity Suite associé.

Pour configurer et tester l’authentification unique Azure AD avec Nitro Productivity Suite, suivez les indications des sections ci-après :

1. [Configurer l’authentification unique Azure AD](#configure-azure-ad-sso) pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.

    a. [Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user) pour tester l’authentification unique Azure AD avec B.Simon.
    
    b. [Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user) pour permettre à B.Simon d’utiliser l’authentification unique Azure AD.
    
2. [Créer un utilisateur de test Nitro Productivity Suite](#create-a-nitro-productivity-suite-test-user) pour avoir un équivalent de B.Simon dans Nitro Productivity Suite lié à la représentation Azure AD de l’utilisateur
1. [Tester l’authentification unique](#test-sso) pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Nitro Productivity Suite**, puis recherchez la section **Gérer**. Sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la section **Certificat de signature SAML**, recherchez **Certificat (en base64)** . Sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Capture d’écran de la section Certificat de signature SAML, avec mise en évidence du lien Télécharger](common/certificatebase64.png)
    
1. Dans la section **Configurer Nitro Productivity Suite**, sélectionnez l’icône de copie en regard du champ **URL de connexion**.
    
    ![Capture d’écran de la section Configurer Nitro Productivity Suite avec les URL et les icônes de copie mises en évidence](common/copy-configuration-urls.png)
    
1. Sur le [portail Nitro Admin](https://admin.gonitro.com/), dans la page **Enterprise Settings**, recherchez la section **Single Sign-On**. Sélectionnez **Setup SAML SSO**.

    a. Collez l’**URL de connexion** de l’étape précédente dans le champ **Sign In URL**.
    
    b. Chargez le **certificat (en base64)** de l’étape précédente dans le champ **X509 Signing Certificate**.
    
    c. Sélectionnez **Envoyer**.
    
    d. Sélectionnez **Activer l'authentification unique**.


1. Revenez au [portail Azure](https://portal.azure.com/). Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez l’icône de crayon pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Capture d’écran de la page Configurer l’authentification unique avec SAML avec l’icône de crayon mise en évidence](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, si vous voulez configurer l’application en mode lancé par le **fournisseur d’identité**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur**, copiez et collez la valeur du champ **SAML Entity ID** (ID d’entité SAML) à partir du [portail Nitro Admin](https://admin.gonitro.com/). Son modèle doit être le suivant : `urn:auth0:gonitro-prod:<ENVIRONMENT>`

    b. Dans la zone de texte **URL de réponse**, copiez et collez la valeur du champ **ASC URL** (URL ASC) à partir du [portail Nitro Admin](https://admin.gonitro.com/). Son modèle doit être le suivant : `https://gonitro-prod.eu.auth0.com/login/callback?connection=<ENVIRONMENT>`

1. Sélectionnez **Définir des URL supplémentaires**, puis effectuez l’étape suivante si vous voulez configurer l’application en mode initié par le **fournisseur de services** :

    Dans la zone de texte **URL de connexion**, tapez l’URL : `https://sso.gonitro.com/login`

1. Sélectionnez **Enregistrer**.

1. L’application Nitro Productivity Suite attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![Capture d’écran des attributs par défaut](common/default-attributes.png)

1. L’application Nitro Productivity Suite s’attend à ce que certains attributs (en plus des attributs précédents) soient repassés dans la réponse SAML. Ces attributs sont préremplis, mais vous pouvez les rectifier en fonction de vos besoins.
    
    | Nom  |  Attribut source|
    | ---------------| --------------- |
    | employeeNumber |  user.objectid |


### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon sur le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans le champ **Nom**, entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur**, entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** et notez le mot de passe.
   1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Nitro Productivity Suite.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Nitro Productivity Suite**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**. Ensuite, dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Utilisateurs et groupes**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B.Simon** dans la liste des utilisateurs. Choisissez **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Affecter**.

### <a name="create-a-nitro-productivity-suite-test-user"></a>Créer un utilisateur de test Nitro Productivity Suite

Nitro Productivity Suite prend en charge le provisionnement d’utilisateurs juste-à-temps, qui est activé par défaut. Vous n’avez aucune opération supplémentaire à effectuer. S’il n’existe pas encore d’utilisateur dans Nitro Productivity Suite, il en est créé un après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

#### <a name="sp-initiated"></a>Lancée par le fournisseur de services :

1. Cliquez sur **Tester cette application** dans le portail Azure. Cette opération redirige vers l’URL de connexion Nitro Productivity Suite où vous pouvez lancer le processus de connexion.  

2. Accédez directement à l’URL d’authentification unique de Nitro Productivity Suite pour lancer le flux de connexion.

#### <a name="idp-initiated"></a>Lancée par le fournisseur d’identité :

* Sur le portail Azure, cliquez sur **Tester cette application**. Vous êtes alors automatiquement connecté à l’instance de Nitro Productivity Suite pour laquelle vous avez configuré l’authentification unique. 

Vous pouvez aussi utiliser le panneau d’accès Microsoft pour tester l’application dans n’importe quel mode. Si, quand vous cliquez sur la vignette Nitro Productivity Suite dans le volet d’accès, le mode Fournisseur de services est configuré, vous êtes redirigé vers la page de connexion de l’application pour lancer le processus de connexion ; s’il s’agit du mode Fournisseur d’identité, vous êtes automatiquement connecté à l’instance de Nitro Productivity Suite pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez configuré l’instance de Nitro Productivity Suite, vous pouvez appliquer les contrôles de session qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Les contrôles de session sont étendus à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).