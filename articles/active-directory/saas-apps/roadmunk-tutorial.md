---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Roadmunk'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Roadmunk.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/28/2020
ms.author: jeedes
ms.openlocfilehash: a7f4682be2f7fbf308aba32768efa932f27b7a87
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96181699"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-roadmunk"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Roadmunk

Dans ce tutoriel, vous allez apprendre à intégrer Roadmunk à Azure Active Directory (Azure AD). Quand vous intégrez Roadmunk à Azure AD, vous pouvez :

* Contrôler qui dans Azure AD a accès à Roadmunk.
* Permettre à vos utilisateurs de se connecter automatiquement à Roadmunk avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central, le portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Roadmunk pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

Roadmunk prend en charge l’authentification unique lancée par le *fournisseur de services* et par le *fournisseur d’identité*.

## <a name="add-roadmunk-from-the-gallery"></a>Ajouter Roadmunk à partir de la galerie

Pour intégrer Roadmunk à Azure AD, à partir de la galerie, ajoutez Roadmunk à votre liste d’applications SaaS managées :

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire, ou avec un compte personnel Microsoft.
1. Sélectionnez **Azure Active Directory** dans le volet de gauche.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, dans la zone de recherche, tapez **Roadmunk**.
1. Sélectionnez **Roadmunk** dans les résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-roadmunk"></a>Configurer et tester l’authentification unique Azure AD pour Roadmunk

Configurez et testez l’authentification unique Azure AD avec Roadmunk en utilisant un utilisateur de test appelé *B.Simon*. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Roadmunk associé.

Voici une vue d’ensemble de la configuration et du test de l’authentification unique Azure AD avec Roadmunk :

1. [Configurer l’authentification unique Azure AD](#configure-azure-ad-sso) pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. [Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user) pour tester l’authentification unique Azure AD en utilisant B.Simon.
    1. [Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user) pour permettre à B.Simon d’utiliser l’authentification unique Azure AD.
1. [Configurer l’authentification unique Roadmunk](#configure-roadmunk-sso) pour configurer les paramètres de l’authentification unique côté application.
    1. [Créer un utilisateur de test Roadmunk](#create-roadmunk-test-user) pour pouvoir lier l’équivalent de B.Simon dans Roadmunk à la représentation Azure AD de l’utilisateur.
1. [Tester l’authentification unique](#test-sso) pour s’assurer que la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le portail Azure :

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Roadmunk**, recherchez la section **Gérer**, puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez l’icône de stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Capture d’écran montrant l’icône de modification pour Configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, si vous avez un fichier de métadonnée de fournisseur de services et que vous souhaitez effectuer la configuration en mode lancé par le fournisseur d’identité, effectuez ces étapes :

    a. Sélectionnez **Charger le fichier de métadonnées**.

    ![Capture d’écran montrant le lien Charger le fichier de métadonnées](common/upload-metadata.png)

    b. Sélectionnez l’icône de dossier pour choisir le fichier de métadonnées que vous avez téléchargé à l’étape 4 de la procédure « Configurer l’authentification unique Roadmunk ». Sélectionnez **Charger**.

    ![Capture d’écran montrant comment choisir le fichier de métadonnées](common/browse-upload-metadata.png)

    Une fois le fichier de métadonnées chargé, dans la section **Configuration SAML de base**, les valeurs **Identificateur** et **URL de réponse** sont indiquées automatiquement.

    ![Capture d’écran montrant la section Configuration SAML de base. Le champ Identificateur et le champ URL de réponse sont mis en évidence.](common/idp-intiated.png)

    > [!Note]
    > Si les valeurs **Identificateur** et **URL de réponse** ne sont pas renseignées automatiquement, spécifiez-les manuellement.

1. Si vous voulez configurer l’application en mode lancé par le fournisseur de services, sélectionnez **Définir des URL supplémentaires**. Dans le champ **URL de connexion**, tapez `https://login.roadmunk.com`.

    ![Capture d’écran montrant où définir une URL de connexion pour le mode lancé par le fournisseur de services](common/metadata-upload-additional-signon.png)

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération**. Ensuite, sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Capture d’écran montrant le lien de téléchargement du certificat de signature SAML.](common/metadataxml.png)

1. Dans la section **Configurer Roadmunk**, copiez les URL dont vous avez besoin.

    ![Capture d’écran montrant où copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test dans le portail Azure. Vous nommerez l’utilisateur *B.Simon*.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs**.
1. En haut de la fenêtre, sélectionnez **Nouvel utilisateur**.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans le champ **Nom**, entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur**, entrez username@companydomain.extension. Par exemple, entrez `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Roadmunk.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Roadmunk**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer**, puis sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**. Ensuite, dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Utilisateurs et groupes**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, dans la liste **Utilisateurs**, sélectionnez **B.Simon**. Ensuite, au bas de la boîte de dialogue, choisissez **Sélectionner**.
1. Si vous vous attendez à ce qu’un rôle soit attribué aux utilisateurs, choisissez-le dans le menu déroulant **Sélectionner un rôle**. Si aucun rôle n’a été configuré pour cette application, le rôle **Accès par défaut** est sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Affecter**.

## <a name="configure-roadmunk-sso"></a>Configurer l’authentification unique Roadmunk

1. Connectez-vous au site web Roadmunk en tant qu’administrateur.

1. En bas de la page, sélectionnez l’icône utilisateur, puis sélectionnez **Account Settings** (Paramètres de compte).

    ![Capture d’écran montrant où sélectionner les paramètres de compte d’utilisateur](./media/roadmunk-tutorial/account.png)

1. Accédez à **Company** > **Authentication Settings** (Entreprise > Paramètres d’authentification).

1. Dans la page **Authentication Settings**, effectuez les étapes ci-après :

    ![Capture d’écran de la page Authentication Settings](./media/roadmunk-tutorial/saml-sso.png)

    a. Activez **SAML Single Sign On (SSO)** (Authentification unique SAML).

    b. Dans la section **Step 1** (Étape 1), chargez le fichier XML de métadonnées ou fournissez l’URL des métadonnées.

    c. Dans la section **Step 2** (Étape 2), téléchargez le fichier de métadonnées Roadmunk, puis enregistrez-le sur votre ordinateur.

    d. Si vous souhaitez vous connecter à l’aide de l’authentification unique, dans la section **Step 3** (Étape 3), sélectionnez **Enforce SAML Sign-In Only** (Appliquer la connexion SAML uniquement).

    e. Sélectionnez **Enregistrer**.


### <a name="create-roadmunk-test-user"></a>Créer un utilisateur de test Roadmunk

1. Connectez-vous au site web Roadmunk en tant qu’administrateur.

1. Sélectionnez l’icône utilisateur au bas de la page, puis sélectionnez **Account Settings**.

    ![Capture d’écran montrant comment ouvrir les paramètres de compte pour l’utilisateur de test](./media/roadmunk-tutorial/account.png)

1. Ouvrez l’onglet **Users**, puis sélectionnez **Invite User**.

    ![Capture d’écran montrant l’onglet Users. Le bouton Invite User est mis en évidence. Dans la fenêtre ouverte, les champs Email et Role sont mis en évidence.](./media/roadmunk-tutorial/create-user.png)

1. Dans le formulaire qui s’affiche, renseignez les informations requises et sélectionnez **Invite**.


## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration SSO Azure AD à l’aide du volet d’accès.

Dans le portail Mes applications, quand vous sélectionnez la vignette **Roadmunk**, vous devez normalement être connecté automatiquement au compte Roadmunk pour lequel vous avez configuré l’authentification unique. Pour plus d’informations, consultez [Se connecter et démarrer des applications à partir du portail Mes applications](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Roadmunk, vous pouvez appliquer le contrôle de session. Ce contrôle protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. 

Découvrez comment [appliquer un contrôle de session à l’aide de Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).