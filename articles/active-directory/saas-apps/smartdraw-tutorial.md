---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à SmartDraw | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SmartDraw.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/02/2020
ms.author: jeedes
ms.openlocfilehash: b4b9aa16dfc474575bd522238f06fe487508e7e0
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92515846"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-smartdraw"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à SmartDraw

Dans ce tutoriel, vous allez apprendre à intégrer SmartDraw avec Azure Active Directory (Azure AD). Quand vous intégrez SmartDraw à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à SmartDraw.
* Permettre à vos utilisateurs de se connecter automatiquement à SmartDraw avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement pour lequel l’authentification unique SmartDraw est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* SmartDraw prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité** .
* SmartDraw prend en charge l’approvisionnement d’utilisateurs **juste-à-temps** .

## <a name="adding-smartdraw-from-the-gallery"></a>Ajout de SmartDraw à partir de la galerie

Pour configurer l’intégration de SmartDraw à Azure AD, vous devez ajouter SmartDraw à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **SmartDraw** dans la zone de recherche.
1. Sélectionnez **SmartDraw** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-smartdraw"></a>Configurer et tester l’authentification unique Azure AD pour SmartDraw

Configurez et testez l’authentification unique Azure AD avec SmartDraw pour un utilisateur de test nommé **B. Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur SmartDraw associé.

Pour configurer et tester l’authentification unique Azure AD avec SmartDraw, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique SmartDraw](#configure-smartdraw-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test SmartDraw](#create-smartdraw-test-user)** pour avoir un équivalent de B.Simon dans SmartDraw lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **SmartDraw** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , l’utilisateur n’a rien à faire, car l’application est déjà intégrée à Azure.

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://cloud.smartdraw.com/sso/saml/login/<domain>`

    > [!NOTE]
    > La valeur de l’URL de connexion n’est pas réelle. Vous mettrez à jour cette valeur avec l’URL de connexion réelle. La procédure est expliquée plus loin dans le didacticiel. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Cliquez sur **Enregistrer** .

1. L’application SmartDraw s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration Attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application SmartDraw s’attend à ce que quelques attributs supplémentaires soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source|
    | ---------------| --------------- |
    | FirstName | user.givenname |
    | LastName | user.surname |
    | E-mail | user.mail |
    | Groupes | user.groups |

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer SmartDraw** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à SmartDraw.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **SmartDraw** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-smartdraw-sso"></a>Configurer l’authentification unique SmartDraw

1. Pour automatiser la configuration dans SmartDraw, vous devez installer l’ **extension de navigateur de connexion sécurisée à Mes applications** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

1. Après avoir ajouté l’extension au navigateur, cliquez sur **Configurer SmartDraw** pour être dirigé vers l’application SmartDraw. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à SmartDraw. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 5.

    ![Configuration](common/setup-sso.png)

1. Si vous souhaitez configurer SmartDraw manuellement, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise SmartDraw en tant qu’administrateur et effectuez les étapes suivantes :

1. Cliquez sur **Authentification unique** sous Gérer votre licence SmartDraw.

    ![Capture d’écran affichant la boîte de dialogue Manage your SmartDraw License, dans laquelle vous pouvez sélectionner Single Sign-On.](./media/smartdraw-tutorial/configure01.png)

1. Dans la page Configuration, procédez comme suit :

    ![Capture d’écran montrant la page Configuration dans laquelle vous pouvez indiquer les valeurs décrites.](./media/smartdraw-tutorial/configure02.png)

    a. Dans la zone de texte **Votre domaine (par exemple, acme.com)** , tapez votre domaine.

    b. Copiez la valeur de **Your SP Initiated Login Url will be** (Votre URL de connexion initiée par le fournisseur de services sera) et collez-la dans la zone de texte URL de connexion de la section **Configuration SAML de base** sur le portail Azure.

    c. Dans la zone de texte **Security Groups to Allow SmartDraw Access** (Groupes de sécurité pour autoriser l’accès SmartDraw), tapez **Tout le monde** .

    d. Dans la zone de texte **Your SAML Issuer URL** (URL de votre émetteur SAML), collez la valeur d’ **Identifiant Azure AD** que vous avez copiée dans le portail Azure.

    e. Dans le Bloc-notes, ouvrez le fichier XML de métadonnées que vous avez téléchargé à partir du portail Azure, copiez son contenu, puis collez-le dans la zone **Your SAML MetaData** (Vos métadonnées SAML).

    f. Cliquez sur **Enregistrer la configuration** . 

### <a name="create-smartdraw-test-user"></a>Créer un utilisateur de test SmartDraw

Dans cette section, un utilisateur appelé B.Simon est créé dans SmartDraw. SmartDraw prend en charge l’approvisionnement d’utilisateurs juste-à-temps, une option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans SmartDraw, un nouvel utilisateur est créé après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette SmartDraw dans le volet d’accès, vous devez être connecté automatiquement à l’application SmartDraw pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer SmartDraw avec Azure AD](https://aad.portal.azure.com/)