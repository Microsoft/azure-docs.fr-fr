---
title: 'Tutoriel : Intégration d’Azure Active Directory à RStudio Connect SAML Authentication | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et RStudio Connect SAML Authentication.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/21/2020
ms.author: jeedes
ms.openlocfilehash: 84e8c7fc1d2655ea0685ac79841a9c467bf766cf
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96182389"
---
# <a name="tutorial-azure-active-directory-integration-with-rstudio-connect-saml-authentication"></a>Tutoriel : Intégration d’Azure Active Directory à RStudio Connect SAML Authentication

Ce tutoriel explique comment intégrer RStudio Connect SAML Authentication à Azure Active Directory (Azure AD).
L’intégration de RStudio Connect SAML Authentication à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à RStudio Connect SAML Authentication.
* Vous pouvez permettre à vos utilisateurs de se connecter automatiquement à RStudio Connect SAML Authentication (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration de RStudio Connect SAML Authentication à Azure AD, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* RStudio Connect SAML Authentication. Il existe une [évaluation gratuite de 45 jours.](https://www.rstudio.com/products/connect/)

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* RStudio Connect SAML Authentication prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité**

* RStudio Connect SAML Authentication prend en charge l’attribution d’utilisateurs **juste-à-temps**

## <a name="adding-rstudio-connect-saml-authentication-from-the-gallery"></a>Ajout de RStudio Connect SAML Authentication à partir de la galerie

Pour configurer l’intégration de RStudio Connect SAML Authentication à Azure AD, vous devez ajouter RStudio Connect SAML Authentication à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **RStudio Connect SAML Authentication** dans la zone de recherche.
1. Sélectionnez **RStudio Connect SAML Authentication** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-rstudio-connect-saml-authentication"></a>Configurer et tester l’authentification unique Azure AD pour RStudio Connect SAML Authentication

Configurez et testez l’authentification unique Azure AD avec RStudio Connect SAML Authentication à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur RStudio Connect SAML Authentication associé.

Pour configurer et tester l’authentification unique Azure AD avec RStudio Connect SAML Authentication, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
2. **[Configurer l’authentification unique RStudio Connect SAML Authentication](#configure-rstudio-connect-saml-authentication-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test RStudio Connect SAML Authentication](#create-rstudio-connect-saml-authentication-test-user)** pour avoir un équivalent de Britta Simon dans RStudio Connect SAML Authentication lié à la représentation Azure AD de l’utilisateur.
3. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **RStudio Connect SAML Authentication**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité**, effectuez les étapes suivantes, en remplaçant `<example.com>` par l’adresse et le port de votre serveur RStudio Connect SAML Authentication :

    ![Informations d’authentification unique dans Domaine et URL RStudio Connect SAML Authentication](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `https://<example.com>/__login__/saml`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://<example.com>/__login__/saml/acs`

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    ![Chargement des métadonnée RStudio Connect SAML Authentication](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://<example.com>/`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Elles sont déterminées à partir de l’adresse du serveur RStudio Connect SAML Authentication (`https://example.com` dans les exemples ci-dessus). Contactez l’[équipe du support technique RStudio Connect SAML Authentication](mailto:support@rstudio.com) si vous rencontrez des problèmes. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

6. Votre application RStudio Connect SAML Authentication s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut, où **nameidentifier** est mappé avec **user.userprincipalname**. L’application RStudio Connect SAML Authentication s’attend à ce que **nameidentifier** soit mappé sur **user.mail**. Vous devez donc changer le mappage d’attribut en cliquant sur l’icône **Modifier**.

    ![image](common/edit-attribute.png)

7. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur le bouton Copier pour copier l’**URL des métadonnées de fédération d’application**, puis enregistrez-la sur votre ordinateur.

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

Dans cette section, vous autorisez B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à RStudio Connect SAML Authentication.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **RStudio Connect SAML Authentication**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-rstudio-connect-saml-authentication-sso"></a>Configurer l’authentification unique RStudio Connect SAML Authentication

Pour configurer l’authentification unique pour **RStudio Connect SAML Authentication**, vous devez utiliser l’**URL des métadonnées de fédération d’application** et l’**adresse du serveur** utilisées ci-dessus. Cette opération est effectuée dans le fichier de configuration RStudio Connect SAML Authentication dans `/etc/rstudio-connect.rstudio-connect.gcfg`.

Voici un exemple de fichier de configuration :

```
[Server]
SenderEmail =

; Important! The user-facing URL of your RStudio Connect SAML Authentication server.
Address = 

[Http]
Listen = :3939

[Authentication]
Provider = saml

[SAML]
Logging = true

; Important! The URL where your IdP hosts the SAML metadata or the path to a local copy of it placed in the RStudio Connect SAML Authentication server.
IdPMetaData = 

IdPAttributeProfile = azure
SSOInitiated = IdPAndSP
```

Stockez l’**adresse du serveur** dans la valeur `Server.Address`, et l’**URL des métadonnées de fédération d'application** dans la valeur `SAML.IdPMetaData`. Notez que cet exemple de configuration utilise une connexion HTTP non chiffrée, alors qu’Azure AD exige l’utilisation d’une connexion HTTPS chiffrée. Vous pouvez soit utiliser un [proxy inverse](https://docs.rstudio.com/connect/admin/proxy/) devant RStudio Connect SAML Authentication, soit configurer RStudio Connect SAML Authentication pour une [utilisation directe de HTTPS](https://docs.rstudio.com/connect/admin/appendix/configuration/#HTTPS). 

Si vous rencontrez des problèmes de configuration, vous pouvez lire le [Guide de l’administrateur RStudio Connect SAML Authentication](https://docs.rstudio.com/connect/admin/authentication/saml/) ou envoyer un e-mail à l’[équipe du support technique de RStudio](mailto:support@rstudio.com) pour obtenir de l’aide.

### <a name="create-rstudio-connect-saml-authentication-test-user"></a>Créer un utilisateur de test RStudio Connect SAML Authentication

Dans cette section, un utilisateur appelé Britta Simon est créé dans RStudio Connect SAML Authentication. RStudio Connect SAML Authentication prend en charge le provisionnement juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’y a pas déjà un utilisateur dans RStudio Connect SAML Authentication, un utilisateur est créé quand vous tentez d’y accéder.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

#### <a name="sp-initiated"></a>Lancée par le fournisseur de services :

* Cliquez sur **Tester cette application** dans le portail Azure. Vous êtes alors redirigé vers l’URL de connexion RStudio Connect SAML Authentication, à partir de laquelle vous pouvez lancer le flux de connexion.  

* Accédez directement à l’URL de connexion RStudio Connect SAML Authentication pour lancer le flux de connexion.

#### <a name="idp-initiated"></a>Lancée par le fournisseur d’identité :

* Sur le portail Azure, cliquez sur **Tester cette application**. Vous êtes alors automatiquement connecté à l’application RStudio Connect SAML Authentication pour laquelle vous avez configuré l’authentification unique 

Vous pouvez aussi utiliser le panneau d’accès Microsoft pour tester l’application dans n’importe quel mode. Si, au moment de cliquer sur la vignette RStudio Connect SAML Authentication dans le volet d’accès, le mode Fournisseur de services est configuré, vous êtes redirigé vers la page de connexion de l’application pour lancer le flux de connexion ; s’il s’agit du mode Fournisseur d’identité, vous êtes automatiquement connecté à l’application RStudio Connect SAML Authentication pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré RStudio Connect SAML Authentication, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).