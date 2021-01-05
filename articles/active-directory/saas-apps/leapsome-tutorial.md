---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec Leapsome | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Leapsome.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/17/2019
ms.author: jeedes
ms.openlocfilehash: ddc8cce7b56e0d9d4b3dc33dc1ad2d8c16582841
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92458705"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-leapsome"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec Leapsome

Ce tutoriel explique comment intégrer Leapsome avec Azure Active Directory (Azure AD). Quand vous intégrez Leapsome avec Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Leapsome.
* Permettre à vos utilisateurs de se connecter automatiquement à Leapsome avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Leapsome pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Leapsome prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**

## <a name="adding-leapsome-from-the-gallery"></a>Ajout de Leapsome à partir de la galerie

Pour configurer l’intégration de Leapsome à Azure AD, vous devez ajouter Leapsome à votre liste d’applications SaaS gérées, à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Leapsome** dans la zone de recherche.
1. Sélectionnez **Leapsome** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-leapsome"></a>Configurer et tester l’authentification unique Azure AD pour Leapsome

Configurez et testez l’authentification unique Azure AD avec Leapsome à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Leapsome associé.

Pour configurer et tester l’authentification unique Azure AD avec Leapsome, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Leapsome](#configure-leapsome-sso)** – pour configurer les paramètres d’authentification unique côté application.
    * **[Créer un utilisateur de test Leapsome](#create-leapsome-test-user)** – pour avoir un équivalent de B.Simon dans Leapsome qui soit lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [Portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Leapsome** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL : `https://www.leapsome.com`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://www.leapsome.com/api/users/auth/saml/<CLIENTID>/assert`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://www.leapsome.com/api/users/auth/saml/<CLIENTID>/login`

    > [!NOTE]
    > Les valeurs URL de réponse et URL de connexion ci-dessus ne sont pas de vraies valeurs. Vous devrez les remplacer par les valeurs réelles. La procédure est expliquée plus loin dans le tutoriel.

1. L’application Leapsome attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration des attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application Leapsome s’attend à ce que quelques attributs supplémentaires soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source | Espace de noms |
    | ---------------| --------------- | --------- |  
    | firstname | user.givenname | http://schemas.xmlsoap.org/ws/2005/05/identity/claims |
    | lastname | user.surname | http://schemas.xmlsoap.org/ws/2005/05/identity/claims |
    | title | user.jobtitle | http://schemas.xmlsoap.org/ws/2005/05/identity/claims |
    | picture | URL de la photo de l’employé | http://schemas.xmlsoap.org/ws/2005/05/identity/claims |
    | | |

    > [!Note]
    > La valeur de l’attribut picture n’est pas réelle. Pour cette valeur, fournissez la vraie URL de photo. Pour obtenir cette valeur, contactez [l’équipe du support technique Leapsome](mailto:support@leapsome.com).

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Leapsome** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Leapsome.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Leapsome** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-leapsome-sso"></a>Configurer l’authentification unique Leapsome

1. Dans une autre fenêtre de navigateur web, connectez-vous à Leapsome en tant qu’administrateur de la sécurité.

1. Dans le coin supérieur droit, cliquez sur le logo Paramètres, puis cliquez sur **Admin Settings** (Paramètres d’administration).

    ![Leapsome configuré](./media/leapsome-tutorial/tutorial_leapsome_admin.png)

1. Dans la barre de menu de gauche, cliquez sur **Single Sign On (SSO)** (Authentification unique), puis, dans la page **SAML-based single sign-on (SSO)** (Authentification unique SAML), procédez aux étapes suivantes :

    ![SAML Leapsome](./media/leapsome-tutorial/tutorial_leapsome_samlsettings.png)

    a. Sélectionnez **Enable SAML-based single sign-on** (Activer l’authentification unique SAML).

    b. Copiez la valeur de **Login URL (point your users here to start login)** (URL de connexion (à indiquer aux utilisateurs pour démarrer la connexion)), puis collez-la dans la zone de texte **Sign-on URL** (URL de connexion) de la section **Basic SAML Configuration** (Configuration SAML de base) du portail Azure.

    c. Copiez la valeur de **Reply URL (receives response from your identity provider)** (URL de réponse [réponse reçue du fournisseur d’identité]), puis collez-la dans la zone de texte **Reply URL** (URL de réponse) de la section **Basic SAML Configuration** (Configuration SAML de base) du portail Azure.

    d. Dans la zone de texte **SSO Login URL (provided by identity provider)** (URL de connexion SSO - fournie par le fournisseur d'identité), collez la valeur de **Login URL** (URL de connexion) que vous avez copiée à partir du portail Azure.

    e. Copiez le certificat que vous avez téléchargé à partir du portail Azure sans les commentaires `--BEGIN CERTIFICATE and END CERTIFICATE--`, puis collez-le dans la zone de texte **Certificate (provided by identity provider)** [(Certificat (fourni par le fournisseur d'identité))].

    f. Cliquez sur **Update SSO Settings** (Mettre à jour les paramètres SSO).

### <a name="create-leapsome-test-user"></a>Créer un utilisateur de test Leapsome

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans Leapsome. Rapprochez-vous de l’[équipe du support technique Leapsome](mailto:support@leapsome.com) pour ajouter les utilisateurs ou le domaine à mettre sur liste verte sur la plateforme Leapsome. Si le domaine est ajouté par l’équipe, les utilisateurs sont automatiquement provisionnés dans la plateforme Leapsome. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la vignette Leapsome dans le volet d’accès, vous devez être connecté automatiquement à l’application Leapsome pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Leapsome avec Azure AD](https://aad.portal.azure.com/)