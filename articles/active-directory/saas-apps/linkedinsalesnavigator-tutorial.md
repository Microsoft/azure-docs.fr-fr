---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec LinkedIn Sales Navigator | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et LinkedIn Sales Navigator.
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
ms.openlocfilehash: 3f4d4d1e0fc44b9fb031ce5e3e45219fa8169562
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92458504"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-linkedin-sales-navigator"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec LinkedIn Sales Navigator

Ce tutoriel explique comment intégrer LinkedIn Sales Navigator avec Azure Active Directory (Azure AD). Quand vous intégrez LinkedIn Sales Navigator avec Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à LinkedIn Sales Navigator.
* Permettre à vos utilisateurs de se connecter automatiquement à LinkedIn Sales Navigator avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement LinkedIn Sales Navigator pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* LinkedIn Sales Navigator prend en charge l’authentification unique initiée par le **fournisseur de services et le fournisseur d’identité**
* LinkedIn Sales Navigator prend en charge l’attribution d’utilisateurs **juste-à-temps**
* LinkedIn Sales Navigator prend en charge l’attribution d’utilisateurs [juste-à-temps **automatisée**](linkedinsalesnavigator-provisioning-tutorial.md)

## <a name="adding-linkedin-sales-navigator-from-the-gallery"></a>Ajout de LinkedIn Sales Navigator à partir de la galerie

Pour configurer l’intégration de LinkedIn Sales Navigator avec Azure AD, vous devez ajouter LinkedIn Sales Navigator à votre liste d’applications SaaS gérées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **LinkedIn Sales Navigator** dans la zone de recherche.
1. Sélectionnez **LinkedIn Sales Navigator** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-linkedin-sales-navigator"></a>Configurer et tester l’authentification unique Azure AD pour LinkedIn Sales Navigator

Configurez et testez l’authentification unique Azure AD avec LinkedIn Sales Navigator à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur LinkedIn Sales Navigator associé.

Pour configurer et tester l’authentification unique Azure AD avec LinkedIn Sales Navigator, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique LinkedIn Sales Navigator](#configure-linkedin-sales-navigator-sso)** – pour configurer les paramètres d’authentification unique côté application.
    * **[Créer un utilisateur de test LinkedIn Sales Navigator](#create-linkedin-sales-navigator-test-user)** – pour avoir un équivalent de B.Simon dans LinkedIn Sales Navigator qui soit lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [Portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **LinkedIn Sales Navigator** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , entrez la valeur **Entity ID** (la procédure à suivre pour copier cette valeur à partir du portail Linkedin est expliquée plus loin dans ce tutoriel).

    b. Dans la zone de texte **URL de réponse** , entrez la valeur **Assertion Consumer Access (ACS) URL** (la procédure à suivre pour copier cette valeur à partir du portail Linkedin est expliquée plus loin dans ce tutoriel).

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://www.linkedin.com/checkpoint/enterprise/login/<account id>?application=salesNavigator`

1. L’application LinkedIn Sales Navigator attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration Attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application LinkedIn Sales Navigator s’attend à ce que quelques attributs supplémentaires, indiqués ci-dessous, soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source|
    | --- | --- |
    | email| user.mail |
    | department| user.department |
    | firstname| user.givenname |
    | lastname| user.surname |
    | Identificateur d’utilisateur unique | user.mail |

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer LinkedIn Sales Navigator** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à LinkedIn Sales Navigator.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **LinkedIn Sales Navigator** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-linkedin-sales-navigator-sso"></a>Configurer l’authentification unique LinkedIn Sales Navigator

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site web **LinkedIn Sales Navigator** en tant qu’administrateur.

1. Dans le **Centre des comptes** , cliquez sur **Paramètres globaux** sous **Paramètres** . En outre, sélectionnez **Sales Navigator** dans la liste déroulante.

    ![Capture d’écran montrant la section Authentication Settings dans laquelle vous pouvez sélectionner Sales Navigator.](./media/linkedinsalesnavigator-tutorial/tutorial_linkedin_admin_01.png)

1. Cliquez sur **OR Click Here to load and copy individual fields from the form** (OU cliquez ici pour charger et copier des champs individuels à partir du formulaire) et effectuez les étapes suivantes :

    ![Capture d’écran montrant l’authentification unique pour laquelle vous pouvez entrer les valeurs décrites.](./media/linkedinsalesnavigator-tutorial/tutorial_linkedin_admin_031.png)

    a. Copiez la valeur **Entity ID** et collez-la dans la zone de texte **Identificateur** de la section **Configuration SAML de base** du portail Azure.

    b. Copiez la valeur **Assertion Consumer Access (ACS) URL** et collez-la dans la zone de texte **URL de réponse** de la section **Configuration SAML de base** du portail Azure.

1. Accédez à la section **Paramètres de l’administrateur LinkedIn** . Chargez le fichier XML que vous avez téléchargé à partir du portail Azure en cliquant sur l’option **Upload XML file** (Charger un fichier XML).

    ![Capture d’écran montrant la section Configure the LinkedIn service provider SSO settings, dans laquelle vous pouvez charger un fichier XML.](./media/linkedinsalesnavigator-tutorial/tutorial_linkedin_metadata_03.png)

1. Cliquez sur **Activer** pour activer l’authentification unique. L’état de l’authentification unique passe de **Non connecté** à **Connecté**

    ![Capture d’écran montrant l’authentification unique pour laquelle vous pouvez activer l’option Authenticate users with SSO.](./media/linkedinsalesnavigator-tutorial/tutorial_linkedin_admin_05.png)

### <a name="create-linkedin-sales-navigator-test-user"></a>Créer un utilisateur de test LinkedIn Sales Navigator

L’application LinkedIn Sales Navigator prend en charge la configuration d’utilisateur juste à temps (JIT), et après authentification, les utilisateurs sont créés automatiquement dans l’application. Activez **Affecter automatiquement les licences** pour affecter une licence à l’utilisateur.

   ![Création d’un utilisateur de test Azure AD](./media/linkedinsalesnavigator-tutorial/LinkedinUserprovswitch.png)

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la vignette LinkedIn Sales Navigator dans le volet d’accès, vous devez être connecté automatiquement à l’application LinkedIn Sales Navigator pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer LinkedIn Sales Navigator avec Azure AD](https://aad.portal.azure.com/)