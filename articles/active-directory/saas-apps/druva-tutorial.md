---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory avec Druva | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Druva.
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
ms.openlocfilehash: a7868d702ff3f1190d7f51e4ad7316508d453015
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92454522"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-druva"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory avec Druva

Dans ce tutoriel, vous allez apprendre à intégrer Druva dans Azure Active Directory (Azure AD). Quand vous intégrez Druva à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Druva.
* Permettre à vos utilisateurs de se connecter automatiquement à Druva avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Druva pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Druva prend en charge l’authentification unique lancée par le **fournisseur d’identité**
* Une fois que vous avez configuré l’authentification unique Druva, vous pouvez appliquer le contrôle de session, qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-druva-from-the-gallery"></a>Ajout de Druva depuis la galerie

Pour configurer l’intégration de Druva avec Azure AD, vous devez ajouter Druva disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Druva** dans la zone de recherche.
1. Sélectionnez **Druva** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-druva"></a>Configurer et tester l’authentification unique Azure AD pour Druva

Configurez et testez l’authentification unique Azure AD avec Druva à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Druva associé.

Pour configurer et tester l’authentification unique Azure AD auprès de Druva, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Druva](#configure-druva-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Druva](#create-druva-test-user)** pour avoir un équivalent de B.Simon dans Druva lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Druva** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    a. Dans la zone de texte **Identificateur (ID d’entité)** , tapez la valeur de chaîne suivante : `DCP-login`.
    
    b. Dans la zone de texte **URL de réponse (URL Assertion Consumer Service)** , tapez l’URL `https://cloud.druva.com/wrsaml/consume`.

1. Cliquez sur **Enregistrer** .

1. L’application Druva s’attend à recevoir les assertions SAML dans un format spécifique, ce qui requiert que vous ajoutiez des mappages d’attributs personnalisés à votre configuration d’attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application Druva s’attend à ce que quelques attributs supplémentaires présentés ci-dessous soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source|
    | ------------------- | -------------------- |
    | emailAddress | user.email |
    | druva_auth_token | Jeton SSO généré à partir de la console d’administration DCP, sans guillemets.  Par exemple : X-XXXXX-XXXX-S-A-M-P-L-E + TXOXKXEXNX =. Azure ajoute automatiquement des guillemets autour du jeton d’authentification. |

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Druva** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Druva.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Druva** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-druva-sso"></a>Configurer l’authentification unique Druva

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise Druva en tant qu’administrateur.

1. Cliquez sur le logo Druva dans l’angle supérieur gauche, puis sur **Druva Cloud Settings** (Paramètres cloud de Druva).

    ![Paramètres](./media/druva-tutorial/ic795091.png "Paramètres")

1. Sous l’onglet **Authentification unique** , cliquez sur **Modifier** .

    ![Capture d’écran montrant l’onglet « Access Settings - Single Sign-On » avec le bouton « Edit » sélectionné.](./media/druva-tutorial/ic795092.png "Paramètres d’authentification unique")

1. Dans la page **Edit Single Sign-On Settings** (Modifier les paramètres d’authentification unique), procédez comme suit :

    ![Paramètres d’authentification unique](./media/druva-tutorial/ic795095.png "Paramètres d’authentification unique")

    1. Dans la zone de texte **ID Provider Login URL** (URL de connexion du fournisseur d’identité), collez la valeur de l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    1. Ouvrez votre certificat codé en base 64 dans le bloc-notes, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **ID Provider Certificate** .

       > [!NOTE]
       > Pour activer l’authentification unique pour les administrateurs, sélectionnez les cases à cocher **Administrators log into Druva Cloud through SSO provider** (Les administrateurs se connectent au cloud Druva via le fournisseur SSO) et **Allow failsafe access to Druva Cloud administrators(recommended)** (Autoriser l’accès fiable des administrateurs au cloud Druva (recommandé)). Druva recommande d’activer l’option **Failsafe for Administrators** (Accès fiable pour les administrateurs) afin que les administrateur aient accès à la console DCP en cas de défaillance du fournisseur d’identité. Il permet également aux administrateurs d’utiliser des mots de passe SSO et DCP pour accéder à la console DCP.

    1. Cliquez sur **Enregistrer** . Cela permet d’accéder à la plateforme cloud Druva à l’aide de l’authentification unique.

### <a name="create-druva-test-user"></a>Créer un utilisateur de test Druva

Dans cette section, un utilisateur appelé B.Simon est créé dans Druva. Druva prend en charge l’approvisionnement d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Druva, il en est créé un après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Druva dans le volet d’accès, vous devez être connecté automatiquement à l’application Druva pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Druva avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)