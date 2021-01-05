---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Software AG Cloud | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Software AG Cloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/09/2020
ms.author: jeedes
ms.openlocfilehash: f6f56958fb63e73e254a45e7cf8ad457be8269ef
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97607641"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-software-ag-cloud"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Software AG Cloud

Dans ce tutoriel, vous allez découvrir comment intégrer Software AG Cloud à Azure Active Directory (Azure AD). Quand vous intégrez Software AG Cloud à Azure AD, vous pouvez :

* Contrôler, dans Azure AD, qui a accès à Software AG Cloud.
* Permettre à vos utilisateurs de se connecter automatiquement à Software AG Cloud avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Software AG Cloud pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Software AG Cloud prend en charge l’authentification unique lancée par le **fournisseur de services**
* Software AG Cloud prend en charge l’attribution d’utilisateurs **juste-à-temps**

## <a name="adding-software-ag-cloud-from-the-gallery"></a>Ajout de Software AG Cloud à partir de la galerie

Pour configurer l’intégration de Software AG Cloud à Azure AD, vous devez ajouter Software AG Cloud, disponible dans la galerie, à votre liste d’applications SaaS managées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Software AG Cloud** dans la zone de recherche.
1. Sélectionnez **Software AG Cloud** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-software-ag-cloud"></a>Configurer et tester l’authentification unique Azure AD pour Software AG Cloud

Configurez et testez l’authentification unique Azure AD avec Software AG Cloud pour un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur associé dans Software AG Cloud.

Pour configurer et tester l’authentification unique Azure AD avec Software AG Cloud, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Software AG Cloud](#configure-software-ag-cloud-sso)** pour configurer les paramètres d’authentification unique côté application.
    1. **[Créer un utilisateur de test Software AG Cloud](#create-software-ag-cloud-test-user)** pour disposer, dans Software AG Cloud, d’un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Software AG Cloud**, recherchez la section **Gérer**, puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    1. Dans la zone de texte **URL de connexion**, saisissez une URL au format suivant : 

        `https://*.softwareag.cloud/auth/realms/TENANT-NAME/broker/IDENTITY-PROVIDER-NAME/endpoint`

    1. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant :

        `https://*.softwareag.cloud/auth/realms/TENANT-NAME`

        > [!NOTE]
        > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique Software AG Cloud](mailto:support@softwareag.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer Software AG Cloud**, copiez la ou les URL appropriées, en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)
### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory**, **Utilisateurs**, puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans le champ **Nom**, entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur**, entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Software AG Cloud.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Software AG Cloud**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-software-ag-cloud-sso"></a>Configurer l’authentification unique Software AG Cloud

1. Dans une autre fenêtre de navigateur web, connectez-vous au site web Software AG Cloud en tant qu’administrateur.

1.  Cliquez sur **Administration**.

    ![Configuration de l’administration Software AG Cloud](./media/software-ag-cloud-tutorial/admin.png)

1. Accédez à **Single-sign on > Add identity provider** (Authentification unique > Ajouter un fournisseur d’identité)

    ![Configuration du fournisseur d’identité Software AG Cloud](./media/software-ag-cloud-tutorial/add-identity-provider.png)

1. Effectuez les étapes ci-après dans la page suivante.

    ![Étapes suivantes de la configuration de Software AG Cloud](./media/software-ag-cloud-tutorial/saml-1.png)

    a. Dans la zone de texte **Identity Provider Name** (Nom complet du fournisseur d’identité), donnez un nom comme `azure ad`.

    b. Dans la zone de texte **Identity provider unique identifier for use in Software AG Cloud redirect URI** (Identificateur unique du fournisseur d’identité à utiliser dans l’URI de redirection de Software AG Cloud), entrez un nom unique pour le fournisseur d’identité. Le champ **Software AG Cloud redirect URI** (URI de redirection de Software AG Cloud) est actualisé et rempli avec l’URI. Copiez cet URI et utilisez-le pour configurer l’**ID d’entité** et d’autres informations dans le portail Azure selon les modèles définis.

    c. Importez le fichier **XML de métadonnées de fédération** dans **Identity provider configuration** (Configuration du fournisseur d’identité), puis cliquez sur **Next** (Suivant).

    d. Accédez à la page **Configuration**, puis renseignez les champs en fonction des besoins.

### <a name="create-software-ag-cloud-test-user"></a>Créer un utilisateur de test Software AG Cloud

Dans cette section, vous créez un utilisateur appelé Britta Simon dans Software AG Cloud. Software AG Cloud prend en charge l’attribution d’utilisateurs juste-à-temps, qui est activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Software AG Cloud, il en est créé un après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec l’option suivante. 

En supposant que Microsoft Azure est configuré en tant que fournisseur dans Software AG Cloud, accédez à `www.softwareag.cloud` et cliquez sur le bouton de connexion, puis entrez le nom de l’environnement. Dans l’écran suivant, cliquez sur le lien « Log in with <IDP NAME> » (Se connecter avec <IDP NAME>), puis entrez les informations d’identification. Une fois authentifié, vous êtes connecté à la page d’accueil de Software AG Cloud.

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Software AG Cloud, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).


