---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à New Relic | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et New Relic.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/04/2020
ms.author: jeedes
ms.openlocfilehash: 29e19eea51b5ee55831bf1d694a9a6473a62d471
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97504047"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-new-relic"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à New Relic

Dans ce tutoriel, vous allez découvrir comment intégrer New Relic à Azure Active Directory (Azure AD). Quand vous intégrez New Relic à Azure AD, vous pouvez :

* Contrôler qui dans Azure AD a accès à New Relic.
* Permettre à vos utilisateurs de se connecter automatiquement à New Relic avec leur compte Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS (software as a service) à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement New Relic pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* New Relic prend en charge l’authentification unique lancée par le fournisseur de services ou le fournisseur d’identité.
* Une fois que vous avez configuré New Relic, vous pouvez appliquer des contrôles de session, qui protègent contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="add-new-relic-from-the-gallery"></a>Ajouter New Relic depuis la galerie

Pour configurer l’intégration de New Relic à Azure AD, vous devez ajouter **New Relic (By Organization)** à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire, ou avec un compte personnel Microsoft.
1. Sélectionnez le service **Azure Active Directory**.
1. Sélectionnez **Applications d’entreprise** > **Nouvelle application**.
1. Dans la page **Parcourir la galerie Azure AD**, tapez **New Relic (By Organization)** dans la zone de recherche.
1. Sélectionnez **New Relic (By Organization)** dans les résultats, puis sélectionnez **Créer**. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-new-relic"></a>Configurer et tester l’authentification unique Azure AD pour New Relic

Configurez et testez l’authentification unique Azure AD avec New Relic en utilisant un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans New Relic.

Pour configurer et tester l’authentification unique Azure AD avec New Relic :

1. [Configurer l’authentification unique Azure AD](#configure-azure-ad-sso) pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
   1. [Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user) pour tester l’authentification unique Azure AD avec B.Simon.
   1. [Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user) pour permettre à B.Simon d’utiliser l’authentification unique Azure AD.
1. [Configurer l’authentification unique New Relic](#configure-new-relic-sso) pour configurer les paramètres d’authentification unique côté New Relic.
   1. [Créer un utilisateur de test New Relic](#create-a-new-relic-test-user) pour obtenir un équivalent de B.Simon dans New Relic lié à l’utilisateur Azure AD.
1. [Tester l’authentification unique](#test-sso) pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **New Relic by Organization**, puis recherchez la section **Gérer**. Ensuite, sélectionnez **Authentification unique**.

1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.

1. Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez l’icône de crayon pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Capture d’écran de Configurer l’authentification unique avec SAML, où l’icône de crayon est mise en évidence.](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez des valeurs pour **Identificateur** et **URL de réponse**.

   * Récupérez ces valeurs à l’aide de l’application New Relic **My Organization**. Pour utiliser cette application :
      1. [Connectez-vous](https://login.newrelic.com/) à New Relic.
      1. Dans le menu du haut, sélectionnez **Apps** (Applications).
      1. Dans la section **Your apps** (Vos applications), sélectionnez **My Organization** (Mon organisation) > **Authentication domains** (Domaines d’authentification).
      1. Choisissez le domaine d’authentification auquel l’authentification unique Azure AD doit se connecter (si vous avez plusieurs domaines d’authentification). La plupart des entreprises n’ont qu’un seul domaine d’authentification appelé **Default** (Par défaut). S’il n’existe qu’un seul domaine d’authentification, vous n’avez rien à sélectionner.
      1. Dans la section **Authentication** (Authentification), **Assertion consumer URL** (URL du consommateur d’assertion) contient la valeur à utiliser pour **Reply URL** (URL de réponse).
      1. Dans la section **Authentication** (Authentification), **Our entity ID** (Notre ID d’entité) contient la valeur à utiliser pour **Identifier** (Identificateur).

1. Dans la section **User Attributes & Claims** (Attributs et revendications utilisateur), vérifiez que **Unique User Identifier** (Identificateur utilisateur unique) est mappé à un champ qui contient l’adresse e-mail utilisée dans New Relic.

   * Le champ par défaut **user.userprincipalname** fonctionne pour vous si ses valeurs correspondent aux adresses e-mail New Relic.
   * Il se peut que le champ **user.mail** fonctionne mieux si **user.userprincipalname** n’est pas l’adresse e-mail New Relic.

1. Dans la section **Certificat de signature SAML**, copiez la valeur de **URL des métadonnées de fédération d’application**, puis enregistrez-la pour une utilisation ultérieure.

1. Dans la section **Configurer New Relic by Organization**, copiez la valeur de **URL de connexion**, puis enregistrez-la pour une utilisation ultérieure.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Voici comment créer un utilisateur de test appelé B.Simon sur le portail Azure.

1. Dans le portail Azure, sélectionnez **Azure Active Directory**.
1. Sélectionnez **Utilisateurs** > **Nouvel utilisateur**.
1. Dans la page **Nouvel utilisateur** :
   1. Dans le champ **Nom de l’utilisateur**, entrez `username@companydomain.extension`. Par exemple : `b.simon@contoso.com`. Cela doit correspondre à l’adresse e-mail que vous utiliserez côté New Relic.
   1. Dans le champ **Nom**, entrez `B.Simon`.  
   1. Sélectionnez **Afficher le mot de passe**, puis enregistrez la valeur affichée.
   1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Voici comment autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à l’application New Relic (By Organization).

1. Dans le portail Azure, sélectionnez **Azure Active Directory**.
1. Sélectionnez **Applications d’entreprise** > **New Relic by Organization**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Capture d’écran de la section Gérer, avec Utilisateurs et groupes mis en évidence.](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**. Dans **Ajouter une attribution**, sélectionnez **Utilisateurs et groupes** (ou **Utilisateurs**, selon le niveau de votre plan).

   ![Capture d’écran de l’option Ajouter un utilisateur](common/add-assign-user.png)

1. Dans **Utilisateurs et groupes** (ou **Utilisateurs**), sélectionnez **B.Simon** dans la liste **Utilisateurs**, puis choisissez **Sélectionner** en bas de l’écran.
1. Sous **Ajouter une attribution**, sélectionnez **Attribuer**.

## <a name="configure-new-relic-sso"></a>Configurer l’authentification unique pour New Relic

Pour configurer l’authentification unique dans New Relic, suivez ces étapes.

1. [Connectez-vous](https://login.newrelic.com/) à New Relic.

1. Dans le menu du haut, sélectionnez **Apps** (Applications).

1. Dans la section **Your apps** (Vos applications), sélectionnez **My Organization** (Mon organisation) > **Authentication domains** (Domaines d’authentification).

1. Choisissez le domaine d’authentification auquel l’authentification unique Azure AD doit se connecter (si vous avez plusieurs domaines d’authentification). La plupart des entreprises n’ont qu’un seul domaine d’authentification appelé **Default** (Par défaut). S’il n’existe qu’un seul domaine d’authentification, vous n’avez rien à sélectionner.

1. Dans la section **Authentication** (Authentification), sélectionnez **Configure** (Configurer).

   1. Pour **Source of SAML metadata** (Source des métadonnées SAML), entrez la valeur que vous avez enregistrée à partir du champ **URL des métadonnées de fédération d’application** dans Azure AD.

   1. Pour **SSO target URL** (URL cible SSO), entrez la valeur que vous avez enregistrée à partir du champ **URL de connexion** dans Azure AD.

   1. Après avoir vérifié que les paramètres sont bien configurés du côté d’Azure AD et de New Relic, sélectionnez **Save** (Enregistrer). Si la configuration est incorrecte des deux côtés, vos utilisateurs ne pourront pas se connecter à New Relic.

### <a name="create-a-new-relic-test-user"></a>Créer un utilisateur de test New Relic

Dans cette section, vous allez créer un utilisateur appelé B.Simon dans New Relic.

1. [Connectez-vous](https://login.newrelic.com/) à New Relic.

1. Dans le menu du haut, sélectionnez **Apps** (Applications).

1. Dans la section **Your apps** (Vos applications), sélectionnez **User Management** (Gestion des utilisateurs).

1. Sélectionnez **Ajouter un utilisateur**.

   1. Pour **Name** (Nom), entrez **B.Simon**.
   
   1. Pour **Email** (E-mail), entrez la valeur qui sera envoyée par l’authentification unique Azure AD.
   
   1. Choisissez un type d’utilisateur dans **Type** et un groupe d’utilisateurs dans **Group** pour l’utilisateur. Pour un utilisateur de test, **Basic User** (Utilisateur de base) pour Type et **User** (Utilisateur) pour Group sont des choix raisonnables.
   
   1. Pour enregistrer l’utilisateur, sélectionnez **Add User** (Ajouter un utilisateur).

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Voici comment tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de sélectionner **New Relic by Organization** dans le panneau d’accès doit vous connecter automatiquement à l’application New Relic. Pour plus d’informations sur le volet d’accès, consultez [Se connecter et démarrer des applications à partir du portail Mes applications](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer New Relic avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)
