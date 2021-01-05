---
title: 'Tutoriel : Intégration d’Azure Active Directory à HubSpot | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et HubSpot.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/14/2019
ms.author: jeedes
ms.openlocfilehash: c53b7fed17ba4ef2000b905b202286338cd461ed
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92442917"
---
# <a name="tutorial-azure-active-directory-integration-with-hubspot"></a>Tutoriel : Intégration d’Azure AD à HubSpot

Dans ce didacticiel, vous allez apprendre à intégrer HubSpot dans Azure Active Directory (Azure AD).

L’intégration de HubSpot à Azure AD vous offre les avantages suivants :

* Vous pouvez utiliser Azure AD pour contrôler qui a accès à HubSpot.
* Les utilisateurs peuvent être automatiquement connectés à HubSpot avec leur compte Azure AD (par le biais de l’authentification unique).
* Vous pouvez centraliser la gestion de vos comptes à un seul emplacement : le Portail Azure.

Pour plus d’informations sur l’intégration d’applications software as a service (SaaS) à Azure AD, consultez l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD avec HubSpot, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’abonnement Azure AD, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Un abonnement HubSpot pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test et intégrer HubSpot à Azure AD.

HubSpot prend en charge les fonctionnalités suivantes :

* **Authentification unique lancée par le fournisseur de services**
* **Authentification unique démarrée par le fournisseur d’identité**

## <a name="add-hubspot-in-the-azure-portal"></a>Ajouter HubSpot dans le portail Azure

Pour intégrer HubSpot à Azure AD, vous devez ajouter HubSpot à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans le menu de gauche, sélectionnez **Azure Active Directory** .

    ![Option Azure Active Directory](common/select-azuread.png)

1. Sélectionnez **Applications d’entreprise** > **Toutes les applications** .

    ![Volet Applications d’entreprise](common/enterprise-applications.png)

1. Pour ajouter une application, sélectionnez **Nouvelle application** .

    ![Option Nouvelle application](common/add-new-app.png)

1. Dans la zone de recherche, entrez **HubSpot** . Dans les résultats de la recherche, sélectionnez **HubSpot** , puis **Ajouter** .

    ![HubSpot dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec HubSpot sur un utilisateur de test appelé **Britta Simon** . Pour que l’authentification unique fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur HubSpot associé.

Pour configurer et tester l’authentification unique Azure AD avec HubSpot, vous devez suivre les indications des sections suivantes :

| Tâche | Description |
| --- | --- |
| **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** | Autorise les utilisateurs à utiliser cette fonctionnalité. |
| **[Configurer l’authentification unique HubSpot](#configure-hubspot-single-sign-on)** | Configure les paramètres d’authentification unique dans l’application. |
| **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** | Teste l’authentification unique Azure AD pour un utilisateur appelé Britta Simon. |
| **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** | Active Britta Simon pour lui permettre d'utiliser l’authentification unique Azure AD. |
| **[Créer un utilisateur de test HubSpot](#create-a-hubspot-test-user)** | Crée un équivalent de Britta Simon dans HubSpot lié à la représentation Azure AD associée. |
| **[Tester l’authentification unique](#test-single-sign-on)** | Vérifie que la configuration fonctionne. |

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous allez configurer l’authentification unique Azure AD avec HubSpot dans le portail Azure.

1. Sur le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **HubSpot** , sélectionnez **Authentification unique** .

    ![Configurer l’option d’authentification unique](common/select-sso.png)

1. Dans le volet **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML** ou **SAML/WS-Fed** pour activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

1. Dans le volet **Configurer l’authentification unique avec SAML** , sélectionnez **Modifier** (icône de crayon) pour ouvrir le volet **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans le volet **Configuration SAML de base** , pour configurer le *mode initié par le fournisseur d’identité* , effectuez les étapes suivantes :

    1. Dans le champ **Identificateur** , entrez une URL au format suivant : https:\//api.hubspot.com/login-api/v1/saml/login?portalId=\<CUSTOMER ID\>.

    1. Dans le champ **URL de réponse** , entrez une URL au format suivant : https:\//api.hubspot.com/login-api/v1/saml/acs?portalId=\<CUSTOMER ID\>.

    ![Informations d’authentification unique dans Domaine et URL HubSpot](common/idp-intiated.png)

    > [!NOTE]
    > Pour le format des URL, vous pouvez aussi vous référer aux modèles figurant dans le volet **Configuration SAML de base** sur le portail Azure.

1. Pour configurer l’application en mode *initié par le fournisseur de services*  :

    1. Sélectionnez **Définir des URL supplémentaires** .

    1. Dans le champ **URL de connexion** , entrez **https:\//app.hubspot.com/login** .

    ![Option Définir des URL supplémentaires](common/metadata-upload-additional-signon.png)

1. Dans le volet **Configurer l’authentification unique avec SAML** , sous la section **Certificat de signature SAML** , sélectionnez **Télécharger** en regard de **Certificat (Base64)** . Sélectionnez une option de téléchargement en fonction de vos exigences. Enregistrez le certificat sur votre ordinateur.

    ![Option de téléchargement du certificat (Base64)](common/certificatebase64.png)

1. Dans la section **Configurer HubSpot** , copiez les URL suivantes en fonction de vos besoins :

    * URL de connexion
    * Identificateur Azure AD
    * URL de déconnexion

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="configure-hubspot-single-sign-on"></a>Configurer l’authentification unique HubSpot

1. Ouvrez un nouvel onglet dans votre navigateur et connectez-vous à votre compte d’administrateur HubSpot.

1. Sélectionnez l’icône des **paramètres** dans le coin supérieur droit de la page.

    ![Icône des paramètres dans HubSpot](./media/hubspot-tutorial/config1.png)

1. Sélectionnez **Account Defaults** (Valeurs par défaut du compte).

    ![Option Account Defaults (Valeurs par défaut du compte) dans HubSpot](./media/hubspot-tutorial/config2.png)

1. Faites défiler jusqu’à la section **Security** (Sécurité), puis sélectionnez **Set up** (Configurer).

    ![Option Set up (Configurer) dans HubSpot](./media/hubspot-tutorial/config3.png)

1. Dans la section **Set up single sign-on** (Configurer l’authentification unique), effectuez les étapes suivantes :

    1. Dans le champ **Audience URl (Service Provider Entity ID)** (URI de l’audience (ID de l’entité de fournisseur de services)), sélectionnez **Copy** (Copier) pour copier la valeur. Sur le portail Azure, dans le volet **Configuration SAML de base** , collez la valeur dans la zone **Identificateur** .

    1. Dans le champ **URl, ACS, Recipient, or Redirect** (URL de connexion, ACS, destinataire ou redirection), sélectionnez **Copy** (Copier) pour copier la valeur. Sur le portail Azure, dans le volet **Configuration SAML de base** , collez la valeur dans la zone **URL de réponse** .

    1. Dans HubSpot, dans le champ **Identity Provider Identifier or Issuer URL** (URL de l’identificateur ou l’émetteur du fournisseur d’identité), collez l’ **identificateur Azure AD** que vous avez copié sur le portail Azure.

    1. Dans HubSpot, dans le champ **Identity Provider Single Sign-On URL** (URL d’authentification unique du fournisseur d’identité), collez l’ **URL de connexion** que vous avez copiée dans le portail Azure.

    1. Dans le Bloc-notes Windows, ouvrez le fichier Certificate(Base64) que vous avez téléchargé. Sélectionnez puis copiez le contenu du fichier. Ensuite, dans HubSpot, collez-le dans le champ **X.509 Certificate** .

    1. Sélectionnez **Vérifier** .

        ![Section Set up single sign-on (Configurer l’authentification unique) dans HubSpot](./media/hubspot-tutorial/config4.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, créez un utilisateur de test nommé Britta Simon dans le portail Azure.

1. Dans le portail Azure, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs** .

    ![Options Utilisateurs et Tous les utilisateurs](common/users.png)

1. Sélectionnez **Nouvel utilisateur** .

    ![Option Nouvel utilisateur](common/new-user.png)

1. Dans le volet **Utilisateur** , effectuez les étapes suivantes :

    1. Dans la zone **Nom** , entrez **BrittaSimon** .
  
    1. Dans la zone **Nom d’utilisateur** , entrez **brittasimon\@\<your-company-domain>.\<extension\>** . Par exemple, **brittasimon\@contoso.com** .

    1. Cochez la case **Afficher le mot de passe** . Notez la valeur affichée dans la zone **Mot de passe** .

    1. Sélectionnez **Create** (Créer).

    ![Volet Utilisateur](common/user-properties.png)

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez Britta Simon à accéder à HubSpot pour lui permettre d’utiliser l’authentification unique Azure.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications** > **HubSpot** .

    ![Volet Applications d’entreprise](common/enterprise-applications.png)

1. Dans la liste des applications, sélectionnez **HubSpot** .

    ![HubSpot dans la liste des applications](common/all-applications.png)

1. Dans le menu, sélectionnez **Utilisateurs et groupes** .

    ![Option Utilisateurs et groupes](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** . Ensuite, dans le volet **Ajouter une attribution** , sélectionnez **Utilisateurs et groupes** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

1. Dans le volet **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste des utilisateurs. Choisissez **Select** .

1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans le volet **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste. Choisissez **Select** .

1. Dans le volet **Ajouter une attribution** , sélectionnez **Attribuer** .

### <a name="create-a-hubspot-test-user"></a>Créer un utilisateur de test HubSpot

Pour se connecter à HubSpot, les utilisateurs d’Azure AD doivent être attribués dans HubSpot. Dans HubSpot, l’attribution des utilisateurs doit s’effectuer manuellement.

Pour provisionner un compte d’utilisateur dans HubSpot :

1. Connectez-vous à votre site d’entreprise HubSpot en tant qu’administrateur.

1. Sélectionnez l’icône des **paramètres** dans le coin supérieur droit de la page.

    ![Icône des paramètres dans HubSpot](./media/hubspot-tutorial/config1.png)

1. Sélectionnez **Users & Teams** (Utilisateurs et équipes).

    ![Option Users & Teams (Utilisateurs et équipes) dans HubSpot](./media/hubspot-tutorial/user1.png)

1. Sélectionnez **Create user** (Créer un utilisateur).

    ![Option Create user (Créer un utilisateur) dans HubSpot](./media/hubspot-tutorial/user2.png)

1. Dans le champ **Add email addess(es)** (Ajouter des adresses e-mail), entrez l’adresse e-mail de l’utilisateur au format brittasimon\@contoso.com, puis sélectionnez **Next** (Suivant).

    ![Champ Add email addess(es) (Ajouter des adresses e-mail) dans HubSpot](./media/hubspot-tutorial/user3.png)

1. Dans la section **Create users** (Créer des utilisateurs), sélectionnez chaque onglet. Sous chaque onglet, définissez les options et les autorisations nécessaires pour l’utilisateur. Ensuite, sélectionnez **Suivant** .

    ![Onglets de la section Create users (Créer des utilisateurs) dans HubSpot](./media/hubspot-tutorial/user4.png)

1. Pour envoyer l’invitation à l’utilisateur, sélectionnez **Send** (Envoyer).

    ![Option Send (Envoyer) dans HubSpot](./media/hubspot-tutorial/user5.png)

    > [!NOTE]
    > L’utilisateur est activé dès qu’il accepte l’invitation.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à partir du portail Mes applications.

Après avoir configuré l’authentification unique, lorsque vous sélectionnez **HubSpot** dans le portail Mes applications, vous êtes automatiquement connecté à HubSpot. Pour plus d’informations sur le portail Mes applications, consultez [Accéder aux applications du portail Mes applications et les utiliser](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus, consultez les articles suivants :

- [Listes des tutoriels pour intégrer des applications SaaS à Azure Active Directory](./tutorial-list.md)
- [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md)
- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)