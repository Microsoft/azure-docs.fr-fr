---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à iSAMS | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et iSAMS.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/04/2020
ms.author: jeedes
ms.openlocfilehash: e994e39cc916c51e3ad6b00015d710bb422cccc9
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92459694"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-isams"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à iSAMS

Dans ce tutoriel, vous allez apprendre à intégrer iSAMS à Azure Active Directory (Azure AD). Quand vous intégrez iSAMS à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à iSAMS.
* Permettre à vos utilisateurs de se connecter automatiquement à iSAMS avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement iSAMS pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.


* iSAMS prend en charge l’authentification unique (SSO) lancée par **le fournisseur de services et le fournisseur d’identité**
* Une fois que vous avez configuré iSAMS, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-isams-from-the-gallery"></a>Ajout d’iSAMS à partir de la galerie

Pour configurer l’intégration d’iSAMS à Azure AD, vous devez ajouter iSAMS, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **iSAMS** dans la zone de recherche.
1. Sélectionnez **iSAMS** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-isams"></a>Configurer et tester l’authentification unique Azure AD pour iSAMS

Configurez et testez l’authentification unique Azure AD avec iSAMS pour un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur associé dans iSAMS.

Pour configurer et tester l’authentification unique Azure AD avec iSAMS, effectuez les modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique iSAMS](#configure-isams-sso)** pour configurer les paramètres d’authentification unique côté application.
    1. **[Créer un utilisateur de test iSAMS](#create-isams-test-user)** pour avoir, dans iSAMS, un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Sur le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **iSAMS** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<SUBDOMAIN>.isams.cloud/main/sso/saml2`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<SUBDOMAIN>.isams.cloud/main/sso/saml2/acs`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<SUBDOMAIN>.isams.cloud/`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez l’[équipe du support technique iSAMS](mailto:support@isams.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à iSAMS.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **iSAMS** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-isams-sso"></a>Configurer l’authentification unique iSAMS

1. Connectez-vous à iSAMS en tant qu’administrateur.

1. Accédez au Panneau de configuration, puis ouvrez le module **Authentification** .
1. Dans le menu de droite, sélectionnez **Fournisseurs d’identité**

    ![Capture d’écran montrant l’élément Active Directory Configuration avec l’option Identity Providers sélectionnée.](./media/isams-tutorial/click-identity-provider.png)

1. Sélectionner **Ajouter un fournisseur**

    ![Capture d’écran montrant la zone Identity Providers avec l’élément Add Providers sélectionné.](./media/isams-tutorial/add-identity-provider.png)


1. Effectuez les étapes ci-après dans la page suivante :

    ![Capture d’écran montrant l’élément Identity Providers Wizard dans lequel vous pouvez effectuer les étapes décrites.](./media/isams-tutorial/configure-isams.png)

    a. Dans la zone de texte **Nom** , spécifiez un nom valide, comme `Saml2 Azure`. Il s’agit du nom qui s’affichera dans la page de connexion.

    b. Dans la zone URL de métadonnées, entrez la valeur **URL des métadonnées de fédération d’application** que vous avez copiée à partir du portail Azure.
    
    c. Appuyez sur **Importer** .
    
    d. Dans la zone de liste **Applications** de la section **Applications clientes activées** , sélectionnez toutes les applications iSAMS pour lesquelles votre fournisseur doit s’afficher dans la page de connexion.

    e. Cliquez sur **Enregistrer et fermer** .

### <a name="create-isams-test-user"></a>Créer un utilisateur de test iSAMS

1. Connectez-vous à iSAMS en tant qu’administrateur.

2.  Accédez à la **page d’accueil du Panneau de configuration** -> **Sécurité et autorisations** -> **Comptes d’utilisateurs** -> **Options et tâches de l’utilisateur** -> **Modifier les propriétés de l’utilisateur**

    ![Capture d’écran montrant la page User Accounts avec l’option Modify User Properties sélectionnée.](./media/isams-tutorial/modify-user-properties.png)


3. Dans la fenêtre contextuelle qui s’affiche, sélectionnez l’onglet **Détails du compte** , puis remplacez l’ **autorisation** par celle du fournisseur d’identité que vous venez de créer.

    ![Capture d’écran montrant la zone des détails du compte, avec une valeur indiquée pour Authorization.](./media/isams-tutorial/account-details.png)

4. Cliquez sur **Enregistrer et fermer** .

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette iSAMS dans le volet d’accès doit vous connecter automatiquement à l’application iSAMS pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer iSAMS avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)