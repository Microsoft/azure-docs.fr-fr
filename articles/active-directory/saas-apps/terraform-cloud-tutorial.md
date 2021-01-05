---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Terraform Cloud | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Terraform Cloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/29/2020
ms.author: jeedes
ms.openlocfilehash: a960bb3590138ca5efa5146988240db1707a3cf0
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96182119"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-terraform-cloud"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Terraform Cloud

Dans ce tutoriel, vous allez découvrir comment intégrer Terraform Cloud à Azure Active Directory (Azure AD). Quand vous intégrez Terraform Cloud à Azure AD, vous pouvez :

* Contrôler, dans Azure AD, qui a accès à Terraform Cloud.
* Permettre à vos utilisateurs de se connecter automatiquement à Terraform Cloud avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Terraform Cloud pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Terraform Cloud prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**
* Terraform Cloud prend en charge l’attribution d’utilisateurs **juste-à-temps**


## <a name="adding-terraform-cloud-from-the-gallery"></a>Ajout de Terraform Cloud à partir de la galerie

Pour configurer l’intégration de Terraform Cloud à Azure AD, vous devez ajouter Terraform Cloud, disponible dans la galerie, à votre liste d’applications SaaS managées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Terraform Cloud** dans la zone de recherche.
1. Sélectionnez **Terraform Cloud** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-terraform-cloud"></a>Configurer et tester l’authentification unique Azure AD pour Terraform Cloud

Configurez et testez l’authentification unique Azure AD avec Terraform Cloud pour un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur associé dans Terraform Cloud.

Pour configurer et tester l’authentification unique Azure AD avec Terraform Cloud, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Terraform Cloud](#configure-terraform-cloud-sso)** pour configurer les paramètres d’authentification unique côté application.
    1. **[Créer un utilisateur de test Terraform Cloud](#create-terraform-cloud-test-user)** pour disposer, dans Terraform Cloud, d’un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Terraform Cloud**, recherchez la section **Gérer**, puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité**, entrez les valeurs pour les champs suivants :

    Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `https://app.terraform.io/sso/saml/samlconf-<ID>/metadata`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion**, tapez l’URL : `https://app.terraform.io/session`

    > [!NOTE]
    > La valeur de l'identificateur n'est pas réelle. Mettez à jour cette valeur avec l’identificateur réel. Pour obtenir ces valeurs, contactez l’[équipe de support technique Terraform Cloud](mailto:tf-cloud@hashicorp.support). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur le bouton Copier pour copier l’**URL des métadonnées de fédération d’application**, puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)
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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Terraform Cloud.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Terraform Cloud**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-terraform-cloud-sso"></a>Configurer l’authentification unique Terraform Cloud

1. Dans une autre fenêtre de navigateur web, connectez-vous au site web Terraform Cloud en tant qu’administrateur.

2. Accédez à **Settings > SSO > Edit Settings** (Paramètres > SSO > Modifier les paramètres)

    ![Paramètres Terraform Cloud](./media/terraform-cloud-tutorial/sso-settings.png)

3. Dans la page **Edit SSO** (Modifier l’authentification unique), effectuez les étapes suivantes.

    ![Modifier l’authentification unique dans Terraform Cloud](./media/terraform-cloud-tutorial/edit-sso.png)

    a. Dans la zone de texte **Sign-On URL** (URL de connexion), collez la valeur **URL de connexion** que vous avez copiée à partir du portail Azure.

    b. Dans la zone de texte **Entity ID or Identifier** (ID d’entité ou identificateur), collez la valeur **Identificateur Azure AD** que vous avez copiée dans le portail Azure.

    c. Ouvrez le certificat (**Certificate**) téléchargé à partir du portail Azure dans le Bloc-notes, puis collez le contenu dans la zone de texte **Public Certificate** (Certificat public).

    d. Cliquez sur **Save settings** (Enregistrer les paramètres).

### <a name="create-terraform-cloud-test-user"></a>Créer un utilisateur de test Terraform Cloud

Dans cette section, un utilisateur appelé Britta Simon est créé dans Terraform Cloud. Terraform Cloud prend en charge l’attribution d’utilisateurs juste-à-temps, qui est activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Terraform Cloud, il en est créé un après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

#### <a name="sp-initiated"></a>Lancée par le fournisseur de services :

* Cliquez sur **Tester cette application** dans le portail Azure. Cette opération redirige vers l’URL de connexion Terraform Cloud, où vous pouvez lancer le flux de connexion.  

* Accédez directement à l’URL de connexion Terraform Cloud et lancez le flux de connexion à partir de là.

#### <a name="idp-initiated"></a>Lancée par le fournisseur d’identité :

* Cliquez sur **Tester cette application** dans le portail Azure. Vous êtes alors automatiquement connecté à l’instance de Terraform Cloud pour laquelle vous avez configuré l’authentification unique. 

Vous pouvez aussi utiliser le panneau d’accès Microsoft pour tester l’application dans n’importe quel mode. Si, quand vous cliquez sur la vignette Terraform Cloud dans le volet d’accès, le mode Fournisseur de services est configuré, vous êtes redirigé vers la page de connexion de l’application pour lancer le flux de connexion ; s’il s’agit du mode Fournisseur d’identité, vous êtes automatiquement connecté à l’instance de Terraform Cloud pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Terraform Cloud, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).