---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Claromentis | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Claromentis.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/04/2019
ms.author: jeedes
ms.openlocfilehash: 92b068ee9b8aaf4c462002354bbb6490f4888a80
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92455899"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-claromentis"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Claromentis

Dans ce tutoriel, vous allez apprendre à intégrer Claromentis à Azure Active Directory (Azure AD). Quand vous intégrez Claromentis à Azure AD, vous pouvez :

* Contrôler qui dans Azure AD a accès à Claromentis.
* Permettre à vos utilisateurs de se connecter automatiquement à Claromentis avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Claromentis pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Claromentis prend en charge l’authentification unique initiée par le **fournisseur de services et le fournisseur d’identité**
* Claromentis prend en charge le provisionnement d’utilisateurs **juste-à-temps**

## <a name="adding-claromentis-from-the-gallery"></a>Ajout de Claromentis à partir de la galerie

Pour configurer l’intégration de Claromentis à Azure AD, vous devez ajouter Claromentis à votre liste d’applications SaaS gérées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Claromentis** dans la zone de recherche.
1. Sélectionnez **Claromentis** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-claromentis"></a>Configurer et tester l’authentification unique Azure AD pour Claromentis

Configurez et testez l’authentification unique Azure AD auprès de Claromentis pour un utilisateur de test nommé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Claromentis associé.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec Claromentis, suivez les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Claromentis](#configure-claromentis-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Claromentis](#create-claromentis-test-user)** pour avoir un équivalent de B.Simon dans Claromentis lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Sur le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Claromentis** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , entrez la valeur d’identificateur telle que l’exige votre organisation.

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<customer_site_url>/custom/loginhandler/simplesaml/www/module.php/saml/sp/saml2-acs.php/claromentis`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL d’authentification** , tapez une URL au format suivant :

    ```https
    https://<customer_site_url>/login
    https://<customer_site_url>/login?no_auto=0
    ```

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. La procédure est expliquée plus loin dans le tutoriel.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer Claromentis** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez permettre à B.Simon d’utiliser l’authentification unique Azure en lui accordant l’accès à Claromentis.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Claromentis** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-claromentis-sso"></a>Configurer l’authentification unique Claromentis

1. Dans une autre fenêtre de navigateur, connectez-vous au site web Claromentis en tant qu’administrateur.

1. Cliquez sur l’ **icône d’applications** et sélectionnez **Admin** .

    ![Capture d’écran montrant le site web Claromentis avec Admin sélectionné.](./media/claromentis-tutorial/config1.png)

1. Sélectionnez l’onglet **Custom Login Handler** (Gestionnaire de connexion personnalisée).

    ![Capture d’écran montrant la page Administration avec Custom Login Handler sélectionné.](./media/claromentis-tutorial/config2.png)

1. Sélectionnez **SAML Config** .

    ![Capture d’écran montrant la page de configuration pour SAML.](./media/claromentis-tutorial/config3.png)

1. Sous l’onglet **SAML Config** , faites défiler l’écran jusqu’à la section **Config** et effectuez les étapes suivantes :

    ![Capture d’écran montrant la section Config de la page où vous pouvez entrer les informations décrites dans cette étape.](./media/claromentis-tutorial/config4.png)

    a. Dans la zone de texte **Technical Contact Name** (Nom du contact technique), entrez le nom de la personne correspondant au contact technique.

    b. Dans la zone de texte **Technical Contact Email** (Adresse e-mail du contact technique), entrez l’adresse e-mail de la personne correspondant au contact technique.

    c. Indiquez le mot de passe dans la zone de texte **Auth Admin Password** (Mot de passe d’administrateur de l’authentification).

1. Faites défiler l’écran jusqu’à **Auth Sources** (Sources d’authentification) et effectuez les étapes suivantes :

    ![Capture d’écran montrant la section Auth Sources où vous pouvez entrer les informations décrites dans cette étape.](./media/claromentis-tutorial/config5.png)

    a. Dans la zone de texte **IDP** (Fournisseur d’identité), collez la valeur du champ **Identificateur Azure AD** que vous avez copiée à partir du portail Azure.

    b. Dans la zone de texte **Entity ID** (ID d’entité), entrez la valeur d’ID d’entité.

    c. Chargez le fichier **XML de métadonnées de fédération** que vous avez téléchargé à partir du portail Azure.

    d. Cliquez sur **Enregistrer** .

1. Vous remarquerez alors que toutes les URL ont été renseignées dans la section **Identity Provider** de la section **SAML Config** .

    ![Capture d’écran montrant la page Identity Provider contenant des URL.](./media/claromentis-tutorial/config6.png)

    a. Copiez la valeur du champ **Identifier (Entity ID)** , puis collez-la dans la zone de texte **Identificateur** de la section **Configuration SAML de base** du portail Azure.

    b. Copiez la valeur du champ **Reply URL** (URL de réponse), puis collez-la dans la zone de texte **URL de réponse** de la section **Configuration SAML de base** du portail Azure.

    c. Copiez la valeur du champ **Sign On URL** (URL de connexion), puis collez-la dans la zone de texte **URL de connexion** de la section **Configuration SAML de base** du portail Azure.

### <a name="create-claromentis-test-user"></a>Créer un utilisateur de test Claromentis

Dans cette section, un utilisateur appelé B.Simon est créé dans Claromentis. Claromentis prend en charge le provisionnement d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Claromentis, il en est créé un après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Claromentis dans le volet d’accès doit vous connecter automatiquement à l’application Claromentis pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Claromentis avec Azure AD](https://aad.portal.azure.com/)