---
title: 'Tutoriel : Intégration d’Azure Active Directory à Saba TalentSpace | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Saba TalentSpace.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/20/2020
ms.author: jeedes
ms.openlocfilehash: f928df7b2e2da59b716950d4e7757cc508cb8936
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92446347"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-saba-talentspace"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Saba TalentSpace

Dans ce tutoriel, vous allez découvrir comment intégrer Saba TalentSpace à Azure Active Directory (Azure AD). Quand vous intégrez Saba TalentSpace à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Saba TalentSpace.
* Permettre à vos utilisateurs de se connecter automatiquement à Saba TalentSpace avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Saba TalentSpace pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Saba TalentSpace prend en charge l’authentification unique lancée par le **fournisseur de services**
* Après avoir configuré Saba TalentSpace, vous pouvez appliquer le contrôle de session qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-saba-talentspace-from-the-gallery"></a>Ajout de Saba TalentSpace à partir de la galerie

Pour configurer l’intégration de Saba TalentSpace à Azure AD, vous devez ajouter Saba TalentSpace à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Saba TalentSpace** dans la zone de recherche.
1. Sélectionnez **Saba TalentSpace** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-saba-talentspace"></a>Configurer et tester l’authentification unique Azure AD pour Saba TalentSpace

Configurez et testez l’authentification unique Azure AD avec Saba TalentSpace pour un utilisateur de test nommé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Saba TalentSpace associé.

Pour configurer et tester l’authentification unique Azure AD avec Saba TalentSpace, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Saba TalentSpace](#configure-saba-talentspace-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Saba TalentSpace](#create-saba-talentspace-test-user)** pour disposer dans Saba TalentSpace d’un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Saba TalentSpace** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://global.hgncloud.com/[companyname]/saml/login`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://global.hgncloud.com/[companyname]/saml/metadata`

    c. Dans la zone de texte **URL de réponse (URL Assertion Consumer Service)** , tapez une URL à l’aide du modèle suivant : `https://global.hgncloud.com/[companyname]/saml/SSO`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe du support technique de Saba TalentSpace](https://support.saba.com/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer Saba TalentSpace** , copiez la ou les URL appropriées, en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Saba TalentSpace.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Saba TalentSpace** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-saba-talentspace-sso"></a>Configurer l’authentification unique Saba TalentSpace

1. Dans une autre fenêtre de navigateur, connectez-vous à votre application **Saba TalentSpace** en tant qu’administrateur.

2. Cliquez sur l’onglet **Options** .
  
    ![Capture d’écran montrant la page d’accueil « Saba TalentSpace » avec l’onglet « Options » sélectionné.](./media/halogen-software-tutorial/tutorial-halogen-12.png)

3. Dans le volet de navigation de gauche, cliquez sur **SAML Configuration** (Configuration SAML).
  
    ![Capture d’écran montrant le volet de navigation de gauche « User Interface » avec l’option « SAML Configuration » sélectionnée.](./media/halogen-software-tutorial/tutorial-halogen-13.png)

4. Dans la page **SAML Configuration** , procédez comme suit :

    ![Capture d’écran montrant la page « SAML Configuration » avec les options de la zone « Settings » mises en évidence.](./media/halogen-software-tutorial/tutorial-halogen-14.png)

    a. Dans **Unique Identifier** (Identificateur unique), sélectionnez **NameID** .

    b. Dans **Unique Identifier Maps To** (l’identificateur unique mappe vers), sélectionnez **Username** (Nom d’utilisateur).
  
    c. Pour charger votre fichier de métadonnées téléchargé, cliquez sur **Browse** (Parcourir) pour sélectionner le fichier, puis sur **Upload File** (Charger le fichier).

    d. Pour tester la configuration, cliquez sur **Run Test** (Exécuter le test).

    > [!NOTE]
    > Vous devez attendre que le message «  *The SAML test is complete. Please close this window*  » s’affiche. Ensuite, fermez la fenêtre du navigateur. La case à cocher **Enable SAML** (Activer SAML) est sélectionnée uniquement si le test a été effectué.

    e. Sélectionnez **Enable SAML** .

    f. Cliquez sur **Enregistrer les modifications** .

### <a name="create-saba-talentspace-test-user"></a>Créer un utilisateur de test Saba TalentSpace

L’objectif de cette section est de créer un utilisateur appelé Britta Simon dans Saba TalentSpace.

**Pour créer un utilisateur appelé Britta Simon dans Saba TalentSpace, effectuez les étapes suivantes :**

1. Connectez-vous à votre application **Saba TalentSpace** en tant qu’administrateur.

2. Cliquez sur l’onglet **User Center** (Centre utilisateur), puis sur **Create User** (Créer un utilisateur).

    ![Capture d’écran montrant l’onglet « User Center » et l’élément « Create User » sélectionnés.](./media/halogen-software-tutorial/tutorial-halogen-300.png)  

3. Dans la boîte de dialogue **New User** , procédez comme suit :

    ![Qu’est-ce qu’Azure AD Connect ?](./media/halogen-software-tutorial/tutorial-halogen-301.png)

    a. Dans la zone de texte **First Name** (Prénom), tapez le prénom de l’utilisateur, par exemple **B** .

    b. Dans la zone de texte **Nom** , entrez le nom de l’utilisateur, par exemple **Simon** .

    c. Dans la zone de texte **Username** (Nom d’utilisateur), entrez **B.Simon** , le nom d’utilisateur du portail Azure.

    d. Dans la zone **Password** (Mot de passe), entrez un mot de passe pour B.Simon.

    e. Cliquez sur **Enregistrer** .

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Saba TalentSpace dans le volet d’accès, vous devez être connecté automatiquement à l’application Saba TalentSpace pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Saba TalentSpace avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)