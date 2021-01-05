---
title: "Tutoriel : Intégration d'Azure Active Directory à iLMS | Microsoft Docs"
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et iLMS.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/14/2019
ms.author: jeedes
ms.openlocfilehash: 7289fe2ec1f39679dcec95f1f48a6efa9ed0cdfc
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92460334"
---
# <a name="tutorial-integrate-ilms-with-azure-active-directory"></a>Tutoriel : Intégrer iLMS à Azure Active Directory

Dans ce tutoriel, vous allez apprendre à intégrer iLMS à Azure Active Directory (Azure AD). Quand vous intégrez iLMS à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à iLMS.
* Permettre à vos utilisateurs de se connecter automatiquement à iLMS avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’en avez pas, vous pouvez obtenir un essai gratuit d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement iLMS pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. iLMS prend en charge l’authentification unique initiée par le **fournisseur de services et le fournisseur d’identité** .

## <a name="adding-ilms-from-the-gallery"></a>Ajout d’iLMS à partir de la galerie

Pour configurer l’intégration d’iLMS avec Azure AD, vous devez ajouter iLMS à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **iLMS** dans la zone de recherche.
1. Sélectionnez **iLMS** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec iLMS à l’aide d’un utilisateur de test appelé **Britta Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur iLMS associé.

Pour configurer et tester l’authentification unique Azure AD avec iLMS, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique iLMS](#configure-ilms-sso)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer l’utilisateur de test iLMS](#create-ilms-test-user)** pour avoir dans iLMS un équivalent de Britta Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **iLMS** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la page **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , entrez les valeurs pour le champ suivant :

    a. Dans la zone de texte **Identificateur** , collez la valeur de **Identifier** (Identificateur) que vous copiez à partir de la section **Service Provider** (Fournisseur de services) des paramètres SAML dans le portail d’administration iLMS.

    b. Dans la zone de texte **URL de réponse** , collez la valeur de **Endpoint (URL)** (Point de terminaison (URL)) que vous copiez à partir de la section **Service Provider** (Fournisseur de services) des paramètres SAML dans le portail d’administration iLMS, avec le modèle suivant `https://www.inspiredlms.com/Login/<instanceName>/consumer.aspx`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , collez la valeur de **Endpoint (URL)** (Point de terminaison (URL)) que vous copiez à partir de la section **Service Provider** (Fournisseur de services) des paramètres SAML dans le portail d’administration iLMS en tant que `https://www.inspiredlms.com/Login/<instanceName>/consumer.aspx`

1. Pour permettre le provisionnement JIT, votre application iLMS attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut. Cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue Attributs d’utilisateur.

    > [!NOTE]
    > Vous devez activer **Créer un compte utilisateur non reconnu** dans iLMS pour mapper ces attributs. Suivez les instructions [ici](https://support.inspiredelearning.com/help/adding-updating-and-managing-users#just-in-time-provisioning-with-saml-single-signon) pour avoir une idée sur la configuration des attributs.

1. En plus de ce qui précède, l’application iLMS s’attend à ce que quelques attributs supplémentaires soient repassés dans la réponse SAML. Dans la section **Revendications des utilisateurs** de la boîte de dialogue **Attributs utilisateur** , effectuez les étapes suivantes pour ajouter le jeton SAML comme indiqué dans le tableau ci-dessous :

    | Nom | Attribut source|
    | --------|------------- |
    | division | user.department |
    | region | user.state |
    | department | user.jobtitle |

    a. Cliquez sur le bouton **Ajouter une nouvelle revendication** pour ouvrir la boîte de dialogue **Gérer les revendications des utilisateurs** .

    b. Dans la zone de texte **Attribut** , indiquez le nom d’attribut pour cette ligne.

    c. Laissez le champ **Espace de noms** vide.

    d. Sélectionnez Source comme **Attribut** .

    e. Dans la liste **Attribut de la source** , tapez la valeur d’attribut indiquée pour cette ligne.

    f. Cliquez sur **OK**

    g. Cliquez sur **Enregistrer** .

1. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer iLMS** , copiez l’URL ou les URL appropriées, en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-ilms-sso"></a>Configurer l’authentification unique iLMS

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre **portail d’administration iLMS** en tant qu’administrateur.

2. Cliquez sur **SSO:SAML** sous l’onglet **Paramètres** pour ouvrir les paramètres SAML et effectuer les opérations suivantes :

    ![Capture d’écran montrant l’onglet de paramètres ILMS où vous pouvez sélectionner SSO: SAML.](./media/ilms-tutorial/1.png)

3. Développez la section **Fournisseur de services** et copiez les valeurs **Identificateur** et **URL du point de terminaison** .

    ![Capture d’écran montrant les paramètres SAML où vous pouvez obtenir les valeurs.](./media/ilms-tutorial/2.png) 

4. Sous la section **Fournisseur d’identité** , cliquez sur **importer les métadonnées** .

5. Sélectionnez le fichier **Métadonnées de fédération** téléchargé à partir du portail Azure, à partir de la section **Certificat de signature SAML** .

    ![Capture d’écran montrant les paramètres SAML où vous pouvez sélectionner le fichier de métadonnées.](./media/ilms-tutorial/tutorial_ilms_ssoconfig1.png)

6. Si vous souhaitez activer JIT pour approvisionner des comptes iLMS pour l’annulation de reconnaissance d’utilisateurs, procédez comme suit :

    a. Cochez **Créer un compte utilisateur non reconnu** .

    ![Capture d’écran montrant l’option Create Un-recognized User Account.](./media/ilms-tutorial/tutorial_ilms_ssoconfig2.png)

    b. Mappez les attributs dans Azure AD avec les attributs dans iLMS. Dans la colonne d’attribut, spécifiez le nom des attributs ou la valeur par défaut.

    c. Accédez à l’onglet **Règles d’entreprise** et effectuez les opérations suivantes :

    ![Capture d’écran montrant les paramètres Business Rules, où vous pouvez entrer les informations à cette étape.](./media/ilms-tutorial/5.png)

    d. Cochez **Créer des régions, divisions et départements non reconnus** pour créer des régions, divisions et départements qui n’existent pas encore au moment de l’authentification unique.

    e. Cochez **Mettre à jour le profil utilisateur lors de la connexion** pour spécifier si le profil utilisateur est mis à jour à chaque ouverture de session unique.

    f. Si l’option **Mettre à jour les valeurs vides pour les champs non obligatoires dans le profil utilisateur** est cochée, les champs facultatifs qui sont vides lors de la connexion provoquent également la présence de valeurs vides pour ces champs dans le profil iLMS de l’utilisateur.

    g. Cochez **Envoyer un e-mail de notification d’erreur** et entrez l’adresse e-mail de l’utilisateur pour lequel vous souhaitez recevoir l’e-mail de notification d’erreur.

7. Cliquez sur **Enregistrer** pour enregistrer les paramètres.

    ![Capture d’écran montrant le bouton Save.](./media/ilms-tutorial/save.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé Britta Simon dans le Portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `Britta Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `BrittaSimon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en accordant l’accès à iLMS.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **iLMS** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-ilms-test-user"></a>Créer l’utilisateur de test iLMS

L’application prend en charge la configuration d’utilisateur juste-à-temps, et après authentification, les utilisateurs sont créés automatiquement dans l’application. JIT fonctionnera si vous avez coché la case **Créer un compte utilisateur non reconnu** lors de la configuration de SAML sur le portail d’administration iLMS.

Si vous avez besoin de créer un utilisateur manuellement, suivez les étapes ci-dessous :

1. Connectez-vous à votre site d’entreprise iLMS en tant qu’administrateur.

2. Cliquez sur **Register User** (Inscrire un utilisateur) sous l’onglet **Users** (Utilisateurs) pour ouvrir la page **Register User** (Inscrire un utilisateur).

   ![Capture d’écran montrant l’onglet ILMS settings, où vous pouvez sélectionner Register User.](./media/ilms-tutorial/3.png)

3. Dans la page **Register User** (Inscrire un utilisateur), effectuez les étapes suivantes.

    ![Capture d’écran montrant la page Register User, où vous entrez les informations spécifiées.](./media/ilms-tutorial/create_testuser_add.png)

    a. Dans la zone de texte **First Name** (Prénom), tapez le prénom, par exemple Britta.

    b. Dans la zone de texte **Last Name** (Nom), tapez le nom, par exemple Simon.

    c. Dans la zone de texte **Email ID** (Identificateur e-mail), entrez l’adresse e-mail de l’utilisateur, par exemple BrittaSimon@contoso.com.

    d. Dans la liste déroulante **Région** , sélectionnez la valeur pour la région.

    e. Dans la liste déroulante **Division** , sélectionnez la valeur pour la division.

    f. Dans la liste déroulante **Département** , sélectionnez une valeur pour le département.

    g. Cliquez sur **Enregistrer** .

    > [!NOTE]
    > Vous pouvez envoyer un courrier d’inscription à l’utilisateur en cochant la case **Send Registration Mail** (Envoyer un e-mail d’inscription).

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Quand vous sélectionnez la vignette iLMS dans le panneau d’accès, vous devez être connecté automatiquement à l’application iLMS pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)