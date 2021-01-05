---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Jamf Pro | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Jamf Pro.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/11/2020
ms.author: jeedes
ms.openlocfilehash: 48818c300f1ac9ff41f78f39f08e41f95b379b31
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92459566"
---
# <a name="tutorial-azure-active-directory-sso-integration-with-jamf-pro"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec Jamf Pro

Dans ce tutoriel, vous allez apprendre à intégrer Jamf Pro dans Azure Active Directory (Azure AD). Quand vous intégrez Jamf Pro à Azure AD, vous pouvez :

* Utiliser Azure AD pour contrôler qui a accès à Jamf Pro.
* Connecter automatiquement vos utilisateurs à Jamf Pro avec leur compte Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS avec Azure AD, consultez [Authentification unique avec Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Jamf Pro pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. 

* Jamf Pro prend en charge l’authentification unique initiée par le **fournisseur de services** et le **fournisseur d’identité** .
* Après avoir configuré Jamf Pro, vous pouvez appliquer le contrôle de session, qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)

## <a name="add-jamf-pro-from-the-gallery"></a>Ajouter Jamf Pro à partir de la galerie

Pour configurer l’intégration de Jamf Pro à Azure AD, vous devez ajouter Jamf Pro à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec votre compte personnel Microsoft.
1. Dans le volet de gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , entrez *Jamf Pro* dans la zone de recherche.
1. Sélectionnez **Jamf Pro** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-sso-in-azure-ad-for-jamf-pro"></a>Configurer et tester l’authentification unique dans Azure AD pour Jamf Pro

Configurez et testez l’authentification unique Azure AD avec Jamf Pro pour un utilisateur de test appelé B. Simon. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Jamf Pro associé.

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Jamf Pro.

1. [Configurez l’authentification unique dans Azure AD](#configure-sso-in-azure-ad) pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. [Créez un utilisateur de test Azure AD](#create-an-azure-ad-test-user) pour tester l’authentification unique Azure AD avec le compte B. Simon.
    1. [Affectez l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user) pour permettre à B. Simon d’utiliser l’authentification unique dans Azure AD.
1. [Configurez l’authentification unique dans Jamf Pro](#configure-sso-in-jamf-pro) pour configurer les paramètres d’authentification unique côté application.
    1. [Créez un utilisateur de test Jamf Pro](#create-a-jamf-pro-test-user) pour avoir dans Jamf Pro un équivalent de B. Simon lié à la représentation Azure AD de l’utilisateur.
1. [Testez la configuration de l’authentification unique](#test-the-sso-configuration) pour vérifier que la configuration fonctionne.

## <a name="configure-sso-in-azure-ad"></a>Configurer l’authentification unique dans Azure AD

Dans cette section, vous allez activer l’authentification unique Azure AD sur le portail Azure.

1. Sur le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Jamf Pro** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône de stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifiez la page Configuration SAML de base.](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous voulez configurer l’application en mode **initié par le fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , entrez une URL qui utilise la formule suivante : `https://<subdomain>.jamfcloud.com/saml/metadata`

    b. Dans la zone de texte **URL de réponse** , entrez une URL qui utilise la formule suivante : `https://<subdomain>.jamfcloud.com/saml/SSO`

1. Sélectionnez **Définir des URL supplémentaires** . Si vous voulez configurer l’application en mode **initié par le fournisseur de services** , dans la zone de texte **URL de connexion** , entrez une URL qui utilise la formule suivante : `https://<subdomain>.jamfcloud.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Vous obtenez la valeur réelle de l’identificateur dans la section **Single Sign-On** (Authentification unique) du portail Jamf Pro. La procédure est expliquée plus loin dans le tutoriel. Vous pouvez extraire la valeur réelle du sous-domaine à partir de la valeur d’identificateur et utiliser ces informations de sous-domaine comme URL de connexion et URL de réponse. Vous pouvez aussi vous reporter aux formules présentées dans la section **Configuration SAML de base** du portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , accédez à la section **Certificat de signature SAML** , cliquez sur le bouton **Copier** pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Lien de téléchargement du certificat de signature SAML](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon sur le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.
   1. Dans le champ **Nom d’utilisateur** , entrez [nom]@[domaine_entreprise].[extension]. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous accordez à B. Simon l’accès à Jamf Pro.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Jamf Pro** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Sélectionner Utilisateurs et groupes](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Cliquer sur le bouton Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste des utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur. Cliquez ensuite sur le bouton **Sélectionner** au bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-sso-in-jamf-pro"></a>Configurer l’authentification unique dans Jamf Pro

1. Pour automatiser la configuration dans Jamf Pro, installez l’ **extension de navigateur de connexion sécurisée à Mes applications** en sélectionnant **Installer l’extension** .

    ![Page de l’extension de navigateur de connexion sécurisée à Mes applications](common/install-myappssecure-extension.png)

2. Après avoir ajouté l’extension au navigateur, sélectionnez **Configurer Jamf Pro.** Quand l’application Jamf Pro s’ouvre, fournissez les informations d’identification de l’administrateur pour vous connecter. L’extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 7.

    ![Page de configuration dans Jamf Pro](common/setup-sso.png)

3. Pour configurer manuellement Jamf Pro, ouvrez une nouvelle fenêtre de navigateur web et connectez-vous à votre site d’entreprise Jamf Pro en tant qu’administrateur. Effectuez ensuite les étapes suivantes.

4. Sélectionnez l’ **icône des paramètres** dans le coin supérieur droit de la page.

    ![Sélectionner l’icône des paramètres dans Jamf Pro](./media/jamfprosamlconnector-tutorial/configure1.png)

5. Sélectionnez **Single Sign-On** (Authentification unique).

    ![Sélectionner Single Sign-On (Authentification unique) dans Jamf Pro](./media/jamfprosamlconnector-tutorial/configure2.png)

6. Dans la page **Single Sign-On** , effectuez les étapes suivantes.

    ![Page Single Sign-On dans Jamf Pro](./media/jamfprosamlconnector-tutorial/configure3.png)

    a. Sélectionnez **Modifier** .

    b. Cochez la case **Enable Single Sign-On Authentication** (Activer l’authentification unique).

  c. Sélectionnez l’option **Azure** dans le menu déroulant **Fournisseur d’identité** .

  d. Copiez la valeur **ENTITY ID** et collez-la dans le champ **Identificateur (ID d’entité)** dans la section **Configuration SAML de base** du portail Azure.

> [!NOTE]
> Utilisez la valeur du champ `<SUBDOMAIN>` pour renseigner l’URL de connexion et l’URL de réponse dans la section **Configuration SAML de base** du portail Azure.

  e. Sélectionnez **URL des métadonnées** dans le menu déroulant **Source des métadonnées du fournisseur d’identité** . Dans le champ qui apparaît, collez la valeur d’ **URL des métadonnées de fédération d’application** que vous avez copiée sur le portail Azure.

  f. (Facultatif) Modifiez la valeur d’expiration du jeton ou sélectionnez « Désactiver l’expiration du jeton SAML ».

7. Dans la même page, faites défiler jusqu’à la section **User Mapping** (Mappage utilisateur). Effectuez ensuite les étapes suivantes.

    ![Section User Mapping de la page Single Sign-on de JAMF Pro.](./media/jamfprosamlconnector-tutorial/tutorial-jamfprosamlconnector-single.png)

    a. Sélectionnez l’option **NameID** pour **Mappage d’utilisateur de fournisseur d’identité** . Par défaut, cette option est définie sur **NameID** , mais vous pouvez définir un attribut personnalisé.

    b. Sélectionnez **E-mail** pour **Mappage d’utilisateur Jamf Pro** . Jamf Pro mappe les attributs SAML envoyés par le fournisseur d’identité en commençant par les utilisateurs puis par les groupes. Quand un utilisateur tente d’accéder à Jamf Pro, Jamf Pro obtient les informations sur l’utilisateur auprès du fournisseur d’identité et les compare à tous les comptes d’utilisateurs Jamf Pro. Si le compte d’utilisateur entrant est introuvable, Jamf Pro tente de le faire correspondre à un nom de groupe.

    c. Collez la valeur `http://schemas.microsoft.com/ws/2008/06/identity/claims/groups` dans le champ **IDENTITY PROVIDER GROUP ATTRIBUTE NAME** (Nom d’attribut de groupe de fournisseur d’identité).

    d. Dans la même page, faites défiler l’écran jusqu’à la section **Sécurité** , puis sélectionnez **Autoriser les utilisateurs à ignorer l’authentification unique** . Les utilisateurs ne sont alors pas redirigés vers la page de connexion du fournisseur d’identité pour l’authentification et peuvent se connecter directement à Jamf Pro. Lorsqu’un utilisateur tente d’accéder à Jamf Pro via le fournisseur d’identité, l’autorisation et l’authentification SSO lancée par le fournisseur d’identité se produisent.

    e. Sélectionnez **Enregistrer** .

### <a name="create-a-jamf-pro-test-user"></a>Créer un utilisateur de test Jamf Pro

Pour pouvoir se connecter à Jamf Pro, les utilisateurs Azure AD doivent être provisionnés dans Jamf Pro. Dans Jamf Pro, le provisionnement est manuel.

Pour provisionner un compte d’utilisateur, effectuez les étapes suivantes :

1. Connectez-vous à votre site d’entreprise Jamf Pro en tant qu’administrateur.

2. Sélectionnez l’icône des **paramètres** dans le coin supérieur droit de la page.

    ![Icône de paramètres dans Jamf Pro](./media/jamfprosamlconnector-tutorial/configure1.png)

3. Sélectionnez **Jamf Pro User Accounts & Groups** (Groupes et comptes d’utilisateurs Jamf Pro).

    ![Icône Jamf Pro User Accounts & Groups dans les paramètres Jamf Pro](./media/jamfprosamlconnector-tutorial/user1.png)

4. Sélectionnez **Nouveau** .

    ![Page de paramètres système Jamf Pro User Accounts & Groups](./media/jamfprosamlconnector-tutorial/user2.png)

5. Sélectionnez **Create Standard Account (Créer un compte standard)** .

    ![Option Create Standard Account (Créer un compte standard) dans la page Jamf Pro User Accounts & Groups](./media/jamfprosamlconnector-tutorial/user3.png)

6. Dans la boîte de dialogue **New Account** (Nouveau compte), effectuez les étapes suivantes :

    ![Options de configuration de nouveau compte dans les paramètres système Jamf Pro](./media/jamfprosamlconnector-tutorial/user4.png)

    a. Dans le champ **USERNAME** , entrez `Britta Simon` (nom complet de l’utilisateur de test).

    b. Sélectionnez des options pour **ACCESS LEVEL** (Niveau d’accès), **PRIVILEGE SET** (Ensemble de privilèges) et **ACCESS STATUS** (État d’accès) en fonction des règles de votre organisation.

    c. Dans le champ **FULL NAME** (Nom complet), entrez `Britta Simon`.

    d. Dans le champ **EMAIL ADDRESS** (Adresse e-mail), tapez l’adresse e-mail du compte de Britta Simon.

    e. Dans le champ **PASSWORD** (Mot de passe), entrez le mot de passe de l’utilisateur.

    f. Dans le champ **VERIFY PASSWORD** (Vérifier le mot de passe), entrez à nouveau le mot de passe de l’utilisateur.

    g. Sélectionnez **Enregistrer** .

## <a name="test-the-sso-configuration"></a>Tester la configuration SSO

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous sélectionnez la vignette Jamf Pro dans le panneau d’accès, vous devez être connecté automatiquement au compte Jamf Pro pour lequel vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)
- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)
- [Essayer Jamf Pro avec Azure AD](https://aad.portal.azure.com/)