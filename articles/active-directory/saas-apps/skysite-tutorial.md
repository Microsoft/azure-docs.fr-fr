---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à SKYSITE | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SKYSITE.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/27/2019
ms.author: jeedes
ms.openlocfilehash: 9d4413c47e40a611fb1559b32aac1e32a71d1998
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92515982"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-skysite"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à SKYSITE

Dans ce tutoriel, vous allez découvrir comment intégrer SKYSITE à Azure Active Directory (Azure AD). Quand vous intégrez SKYSITE à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à SKYSITE.
* Permettre à vos utilisateurs de se connecter automatiquement à SKYSITE avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement SKYSITE pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* SKYSITE prend en charge l’authentification unique lancée par le **fournisseur d’identité**

* SKYSITE prend en charge le provisionnement d’utilisateurs **juste-à-temps**

## <a name="adding-skysite-from-the-gallery"></a>Ajout de SKYSITE à partir de la galerie

Pour configurer l’intégration de SKYSITE à Azure AD, vous devez ajouter SKYSITE à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **SKYSITE** dans la zone de recherche.
1. Sélectionnez **SKYSITE** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-skysite"></a>Configurer et tester l’authentification unique Azure AD pour SKYSITE

Configurez et testez l’authentification unique Azure AD avec SKYSITE en utilisant un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur SKYSITE associé.

Pour configurer et tester l’authentification unique Azure AD avec SKYSITE, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique SKYSITE](#configure-skysite-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test SKYSITE](#create-skysite-test-user)** pour avoir un équivalent de B.Simon dans SKYSITE lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **SKYSITE** , cliquez sur l’ **onglet Propriétés** et effectuez l’étape suivante : 

    ![Propriétés de l’authentification unique](./media/skysite-tutorial/config05.png)

    * Copiez l’ **URL de l’accès utilisateur** et collez-la dans la **section Configurer l’authentification unique SKYSITE** , comme expliqué plus loin dans le tutoriel.

1. Dans la page d’intégration de l’application **SKYSITE** , accédez à **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , l’application est préconfigurée en mode Lancement par le **fournisseur d’identité** et les URL nécessaires sont déjà préremplies avec Azure. L’utilisateur doit enregistrer la configuration en cliquant sur le bouton **Enregistrer** .

1. L’application SKYSITE s’attend à recevoir les assertions SAML dans un certain format, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration des attributs de jetons SAML. La capture d’écran suivante montre la liste des attributs par défaut. Cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue Attributs d’utilisateur.

    ![Capture d’écran montrant User Attributes avec l’icône Edit sélectionnée.](common/edit-attribute.png)

1. En plus de ce qui précède, l’application SKYSITE s’attend à ce que quelques attributs supplémentaires soient repassés dans la réponse SAML. Dans la section **Attributs et revendications de l’utilisateur** de la boîte de dialogue **Revendications de groupe (préversion)** , effectuez les étapes suivantes :

    a. Cliquez sur le **stylo** en regard de **Groupes renvoyés dans la revendication** .

    ![Capture d’écran montrant User claims avec l’option permettant d’ajouter une nouvelle revendication.](./media/skysite-tutorial/config01.png)

    ![Capture d’écran montrant la boîte de dialogue Manage user claims où vous pouvez entrer les valeurs décrites.](./media/skysite-tutorial/config02.png)

    b. Sélectionnez **Tous les groupes** dans la liste d’options.

    c. Sélectionnez **Attribut source** de **ID de groupe** .

    d. Cliquez sur **Enregistrer** .

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer SKYSITE** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à SKYSITE.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **SKYSITE** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="configure-skysite-sso"></a>Configurer l’authentification unique SKYSITE

1. Ouvrez une nouvelle fenêtre de navigateur web et connectez-vous à votre site d’entreprise SKYSITE en tant qu’administrateur, puis effectuez les étapes suivantes :

4. Cliquez sur **Settings** (Paramètres) en haut à droite de la page, puis accédez à **Account setting** (Paramètres du compte).

    ![Capture d’écran montrant l’élément Account setting sélectionné à partir de Settings.](./media/skysite-tutorial/config03.png)

5. Basculez vers l’onglet **Single sign on (SSO)** (Authentification unique) et effectuez les étapes suivantes :

    ![Capture d’écran montrant l’onglet Authentification unique dans lequel vous pouvez entrer les valeurs décrites.](./media/skysite-tutorial/config04.png)

    a. Dans la zone de texte **Identity Provider sign in URL** (URL de connexion du fournisseur d’identité), collez la valeur **URL de l’accès utilisateur** que vous avez copiée à partir de l’onglet **Propriétés** du portail Azure.

    b. Cliquez sur **Upload certificate** (Charger un certificat) pour charger le certificat codé en Base64 que vous avez téléchargé à partir du portail Azure.

    c. Cliquez sur **Enregistrer** .

### <a name="create-skysite-test-user"></a>Créer un utilisateur de test SKYSITE

Dans cette section, un utilisateur appelé Britta Simon est créé dans SKYSITE. SKYSITE prend en charge le provisionnement d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans SKYSITE, il en est créé un après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette SKYSITE dans le volet d’accès, vous devez être connecté automatiquement à l’application SKYSITE pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer SKYSITE avec Azure AD](https://aad.portal.azure.com/)