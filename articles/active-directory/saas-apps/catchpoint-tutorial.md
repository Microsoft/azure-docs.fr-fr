---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Catchpoint'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Catchpoint.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/27/2020
ms.author: jeedes
ms.openlocfilehash: 8ff7f6468b5556b56c5c2aeaba6107cac48d1ed4
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92456447"
---
# <a name="tutorial-azure-active-directory-single-sign-on-integration-with-catchpoint"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Catchpoint

Dans ce tutoriel, vous découvrez comment intégrer Catchpoint à Azure Active Directory (Azure AD). Quand vous intégrez Catchpoint à Azure AD, vous pouvez :

* Contrôler l’accès utilisateur à Catchpoint depuis Azure AD.
* Permettre à vos utilisateurs de se connecter automatiquement à Catchpoint avec leur compte Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Catchpoint pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Catchpoint prend en charge l’authentification unique lancée par le fournisseur de services et le fournisseur d’identité.
* Catchpoint prend en charge le provisionnement juste-à-temps des utilisateurs.
* Après avoir configuré Catchpoint, vous pouvez appliquer le contrôle de session. Cette précaution protège contre l’exfiltration et l’infiltration de données sensibles de votre organisation en temps réel. Le contrôle de session est une extension de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="add-catchpoint-from-the-gallery"></a>Ajouter Catchpoint à partir de la galerie

Pour configurer l’intégration de Catchpoint à Azure AD, ajoutez Catchpoint à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte Microsoft professionnel, scolaire ou personnel.
1. Dans le volet de gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Catchpoint** dans la zone de recherche.
1. Sélectionnez **Catchpoint** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-catchpoint"></a>Configurer et tester l’authentification unique Azure AD pour Catchpoint

Pour que l’authentification unique fonctionne, vous devez lier un utilisateur Azure AD à un utilisateur dans Catchpoint. Pour ce tutoriel, nous allons configurer un utilisateur de test nommé **B.Simon** . 

Effectuez les sections suivantes :

1. [Configurer l’authentification unique Azure AD](#configure-azure-ad-sso) afin d’activer cette fonctionnalité pour vos utilisateurs.
    * [Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user) pour tester l’authentification unique Azure AD avec B.Simon.
    * [Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user) pour permettre à B.Simon d’utiliser l’authentification unique Azure AD.
1. [Configurer l’authentification unique Catchpoint](#configure-catchpoint-sso) pour configurer les paramètres de l’authentification unique côté application.
    * [Créer un utilisateur de test Catchpoint](#create-a-catchpoint-test-user) pour permettre la liaison du compte de test Azure AD de B.Simon à un compte d’utilisateur similaire dans Catchpoint.
1. [Tester l’authentification unique](#test-sso) pour vérifier que la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes dans le portail Azure pour activer l’authentification unique Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Dans la page d’intégration de l’application **Catchpoint** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône de stylet pour modifier les paramètres de **Configuration SAML de base** .

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Configurez le mode de lancement pour Catchpoint :
   - Pour le mode de lancement par **fournisseur d’identité** , entrez les valeurs pour les champs suivants :
     - Pour **Identificateur**  : `https://portal.catchpoint.com/SAML2`
     - Pour **URL de réponse**  : `https://portal.catchpoint.com/ui/Entry/SingleSignOn.aspx`
   - Pour le mode de lancement par **fournisseur de services** , sélectionnez **Définir des URL supplémentaires** , puis entrez la valeur suivante :
     - Pour **URL de connexion**  : `https://portal.catchpoint.com/ui/Entry/SingleSignOn.aspx`

1. L’application Catchpoint attend les assertions SAML dans un format spécifique. Ajoutez des mappages d’attributs personnalisés à votre configuration des attributs de jeton SAML. Le tableau suivant contient la liste des attributs par défaut :

    | Nom | Attribut source|
    | ------------ | --------- |
    | Givenname | user.givenneame |
    | Surname | user.surname |
    | Emailaddress | user.mail |
    | Nom | user.userprincipalname |
    | Identificateur d’utilisateur unique | user.userprincipalname |

    ![Capture d’écran de la liste Attributs et revendications de l’utilisateur](common/default-attributes.png)

1. En outre, l’application Catchpoint attend qu’un autre attribut soit passé dans une réponse SAML. Consultez le tableau suivant. Cet attribut est également pré-rempli, mais vous pouvez le consulter et le mettre à jour en fonction de vos besoins.

    | Nom | Attribut source|
    | ------------ | --------- |
    | espace de noms | user.assignedrole |

    > [!NOTE]
    > La revendication `namespace` doit être mappée au nom du compte. Ce nom de compte doit être configuré avec un rôle dans Azure AD à passer en retour dans la réponse SAML. Pour plus d’informations sur les rôles dans Azure AD, consultez [Configurer les revendications de rôle émises dans le jeton SAML pour les applications d’entreprise](../develop/active-directory-enterprise-app-role-management.md).

1. Accédez la page **Configurer l’authentification unique avec SAML** . Dans la section **Certificat de signature SAML** , recherchez **Certificat (Base64)** . Sélectionnez **Télécharger** pour enregistrer le certificat sur votre ordinateur.

    ![Lien de téléchargement du certificat](common/certificatebase64.png)

1. Dans la section **Configurer Catchpoint** , copiez les URL dont vous avez besoin dans une étape ultérieure.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous utilisez le portail Azure pour créer un utilisateur de test Azure AD appelé B.Simon.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple, entrez `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** . Notez la valeur du mot de passe affiché.
   1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Catchpoint.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Catchpoint** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien « Ajouter un utilisateur »](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B.Simon** dans la liste des utilisateurs. Cliquez sur **Sélectionner** dans le bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, regardez dans la boîte de dialogue **Sélectionner un rôle** et choisissez le rôle de l’utilisateur dans la liste. Cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Affecter** .

## <a name="configure-catchpoint-sso"></a>Configurer l’authentification unique Catchpoint

1. Dans une autre fenêtre du navigateur web, connectez-vous à l’application Catchpoint en tant qu’administrateur.

1. Sélectionnez l’icône **Settings** (Paramètres), puis **SSO Identity Provider** (Fournisseur d’identité SSO).

    ![Capture d’écran des paramètres de Catchpoint avec le fournisseur d’identité SSO sélectionné](./media/catchpoint-tutorial/configuration1.png)

1. Dans la page **Single Sign-On** (Authentification unique), entrez les champs suivants :

   ![Capture d’écran de la page d’authentification unique de Catchpoint](./media/catchpoint-tutorial/configuration2.png)

   Champ | Valeur
   ----- | ----- 
   **Espace de noms** | Une valeur d’espace de noms valide.
   **Identity Provider Issuer** (Émetteur du fournisseur d’identité) | La valeur `Azure AD Identifier` du portail Azure.
   **Single Sign On Url** (URL d’authentification unique) | La valeur `Login URL` du portail Azure.
   **Certificate** | Contenu du fichier `Certificate (Base64)` téléchargé depuis le portail Azure. Utilisez le Bloc-notes pour visualiser et copier.

   Vous pouvez aussi charger le fichier **XML des métadonnées de fédération** en sélectionnant l’option **Upload Metadata** (Charger les métadonnées).

1. Sélectionnez **Enregistrer** .

### <a name="create-a-catchpoint-test-user"></a>Créer un utilisateur de test Catchpoint

Catchpoint prend en charge le provisionnement d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucun élément d’action dans cette section. Si B.Simon n’existe pas encore en tant qu’utilisateur dans Catchpoint, il est créé après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à partir du portail Mes applications.

Quand vous sélectionnez la vignette Catchpoint dans le portail Mes applications, vous devez être automatiquement connecté à l’application Catchpoint pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le portail Mes applications, consultez [Se connecter et démarrer des applications à partir du portail Mes applications](../user-help/my-apps-portal-end-user-access.md).

> [!NOTE]
> Quand vous vous connectez à l’application Catchpoint via la page de connexion, après avoir fourni les **informations d’identification Catchpoint** , entrez la valeur de l’espace de noms ( **Namespace** ) valide dans le champ **Company Credentials(SSO)** (Informations d’identification de la société (SSO)), puis sélectionnez **Login** (Se connecter).
> 
> ![Configuration de Catchpoint](./media/catchpoint-tutorial/loginimage.png)

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Catchpoint avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)