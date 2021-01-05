---
title: 'Didacticiel : intégration d’Azure Active Directory à la formation de sensibilisation à la sécurité KnowBe4 | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et la formation de sensibilisation à la sécurité KnowBe4.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/22/2020
ms.author: jeedes
ms.openlocfilehash: 77d810fc8327e3777102f2a09f0ca32835aad4b3
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96180759"
---
# <a name="tutorial-azure-active-directory-integration-with-knowbe4-security-awareness-training"></a>Didacticiel : intégration d’Azure Active Directory à la formation de sensibilisation à la sécurité KnowBe4

Dans ce didacticiel, vous allez apprendre à intégrer la formation de sensibilisation à la sécurité KnowBe4 à Azure Active Directory (Azure AD).
L’intégration de la formation de sensibilisation à la sécurité KnowBe4 à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à la formation de sensibilisation à la sécurité KnowBe4.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à la formation de sensibilisation à la sécurité KnowBe4 (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à la formation de sensibilisation à la sécurité KnowBe4, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement à la formation de sensibilisation à la sécurité KnowBe4 pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* La formation de sensibilisation à la sécurité KnowBe4 prend en charge l’authentification unique initiée par le **fournisseur de services**

* La formation de sensibilisation à la sécurité KnowBe4 prend en charge l’attribution d’utilisateurs **Juste-à-temps**

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-knowbe4-from-the-gallery"></a>Ajout de KnowBe4 à partir de la galerie

Pour configurer l’intégration de KnowBe4 à Azure AD, vous devez ajouter KnowBe4 à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **KnowBe4** dans la zone de recherche.
1. Sélectionnez **KnowBe4** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec KnowBe4 sur un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation doit être établie entre l’utilisateur Azure AD et l’utilisateur associé dans KnowBe4.

Pour configurer et tester l’authentification unique Azure AD avec KnowBe4, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique (SSO) Azure AD avec Britta Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique (SSO) Azure AD.
2. **[Configurer l’authentification unique avec la formation de sensibilisation à la sécurité KnowBe4](#configure-knowbe4-security-awareness-training-sso)** pour configurer les paramètres d’authentification unique côté application.
    * **[Créer un utilisateur de test de la formation de sensibilisation à la sécurité KnowBe4](#create-knowbe4-security-awareness-training-test-user)** pour avoir un équivalent de Britta Simon dans la formation de sensibilisation à la sécurité KnowBe4, lié à la représentation de l’utilisateur Azure AD.
3. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **KnowBe4**, recherchez la section **Gérer**, puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    Dans la zone de texte **URL de connexion**, entrez une URL au format suivant :  `https://<companyname>.KnowBe4.com/auth/saml/<instancename>`

    > [!NOTE]
    > La valeur d’URL de connexion n’est pas réelle. Mettez à jour cette valeur avec l’URL de connexion réelle. Contactez l’[équipe de support client de formation de sensibilisation à la sécurité KnowBe4](mailto:support@KnowBe4.com) pour obtenir cette valeur. Vous pouvez également consulter le modèle figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML**, à la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Raw)** en fonction des options définies, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificateraw.png)

6. Dans la section **Configurer la formation de sensibilisation à la sécurité KnowBe4**, copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory**, **Utilisateurs**, puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans le champ **Nom**, entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur**, entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en accordant l’accès à KnowBe4.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **KnowBe4**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-knowbe4-security-awareness-training-sso"></a>Configurer l’authentification unique avec la formation de sensibilisation à la sécurité KnowBe4

Pour configurer l’authentification unique du côté de la **formation de sensibilisation à la sécurité KnowBe4**, vous devez envoyer le **Certificat (brut)** téléchargé et les URL appropriées copiées sur le portail Azure à l’[équipe du support technique de la formation de sensibilisation à la sécurité KnowBe4](mailto:support@KnowBe4.com). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-knowbe4-security-awareness-training-test-user"></a>Créer un utilisateur de test de la formation de sensibilisation à la sécurité KnowBe4

Dans cette section, un utilisateur nommé Britta Simon est créé dans KnowBe4. KnowBe4 prend en charge l’attribution d’utilisateurs juste-à-temps, qui est activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans KnowBe4, il en est créé un après l’authentification.

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

1. Cliquez sur **Tester cette application** dans le portail Azure. Cette opération redirige vers l’URL de connexion KnowBe4, où vous pouvez lancer le flux de connexion. 

2. Accédez directement à l’URL de connexion KnowBe4 et lancez le flux de connexion à partir de là.

3. Vous pouvez utiliser le volet d’accès Microsoft. Le fait de cliquer sur la vignette KnowBe4 dans le volet d’accès vous redirige vers l’URL de connexion KnowBe4. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré KnowBe4, vous pouvez appliquer un contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).