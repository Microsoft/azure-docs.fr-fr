---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à LinkedIn Talent Solutions | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et LinkedIn Talent Solutions.
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
ms.openlocfilehash: ccad668476ecf3311d51a476138a2b2190ac7966
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97364389"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-linkedin-talent-solutions"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à LinkedIn Talent Solutions

Dans ce tutoriel, vous allez apprendre à intégrer LinkedIn Talent Solutions à Azure Active Directory (Azure AD). Quand vous intégrez LinkedIn Talent Solutions à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à LinkedIn Talent Solutions.
* Permettre à vos utilisateurs de se connecter automatiquement à LinkedIn Talent Solutions avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Accès au centre des comptes dans le tableau de bord LinkedIn Talent Solutions

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* LinkedIn Talent Solutions prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**.
* LinkedIn Talent Solutions prend en charge l’attribution d’utilisateurs **juste-à-temps**


## <a name="adding-linkedin-talent-solutions-from-the-gallery"></a>Ajout de LinkedIn Talent Solutions à partir de la galerie

Pour configurer l’intégration de LinkedIn Talent Solutions à Azure AD, vous devez ajouter LinkedIn Talent Solutions à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **LinkedIn Talent Solutions** dans la zone de recherche.
1. Sélectionnez **LinkedIn Talent Solutions** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-linkedin-talent-solutions"></a>Configurer et tester l’authentification unique Azure AD pour LinkedIn Talent Solutions

Configurez et testez l’authentification unique Azure AD avec LinkedIn Talent Solutions pour un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur LinkedIn Talent Solutions associé.

Pour configurer et tester l’authentification unique Azure AD avec LinkedIn Talent Solutions, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique LinkedIn Talent Solutions](#configure-linkedin-talent-solutions-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test LinkedIn Talent Solutions](#create-linkedin-talent-solutions-test-user)** pour avoir un équivalent de B.Simon dans LinkedIn Talent Solutions lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, dans la page d’intégration de l’application **LinkedIn Talent Solutions**, recherchez la section **Gérer**, puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, si vous disposez d’un **fichier de métadonnées du fournisseur de services**, suivez les étapes ci-dessous :

    a. Cliquez sur **Charger un fichier de métadonnées**.

    ![image1](common/upload-metadata.png)

    b. Cliquez sur le **logo du dossier** pour sélectionner le fichier de métadonnées, puis cliquez sur **Charger**.

    ![image2](common/browse-upload-metadata.png)

    c. Une fois le fichier de métadonnées chargé, les valeurs **Identificateur** et **URL de réponse** sont automatiquement renseignées dans la section Configuration SAML de base :

    ![image3](common/idp-intiated.png)

    > [!Note]
    > Si les valeurs **Identificateur** et **URL de réponse** ne sont pas automatiquement renseignées, renseignez-les manuellement en fonction de vos besoins.

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion**, tapez l’URL : `https://www.linkedin.com/talent/`

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer LinkedIn Talent Solutions**, copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à LinkedIn Talent Solutions.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **LinkedIn Talent Solutions**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-linkedin-talent-solutions-sso"></a>Configurer l’authentification unique LinkedIn Talent Solutions

1. Connectez-vous au site web LinkedIn Talent Solutions en tant qu’administrateur.
1. Accédez à **Account Center** (Centre des comptes). 
1. Dans la barre de navigation, sélectionnez l’onglet **Settings** (Paramètres).

    ![Page de paramètres](./media/linkedin-talent-solutions-tutorial/settings.png)

1. Développez la section **Single Sign-On (SSO)** (Authentification unique (SSO)). 

1. Cliquez sur le bouton **Download** (Télécharger) pour télécharger le **fichier de métadonnées** ou cliquez sur le lien **or click here to load and copy individual fields from the form** (ou cliquez ici pour charger et copier des champs individuels à partir du formulaire) pour afficher les données de configuration.

    ![ Données de configuration](./media/linkedin-talent-solutions-tutorial/sso-settings.png)

1. Procédez comme suit pour copier les champs individuels à partir du formulaire.

    ![ Configuration avec données d’entrée](./media/linkedin-talent-solutions-tutorial/configuration.png)

    a. Copiez la valeur du champ **Entity ID** (ID d’entité), puis collez-la dans la zone de texte **Identificateur Azure AD** de la section **Configuration SAML de base** du portail Azure.

    b. Copiez la valeur du champ **ACS URL** (URL ACS), puis collez-la dans la zone de texte **URL de réponse** de la section **Configuration SAML de base** du portail Azure.

    c. Copiez le contenu de la zone de texte **SP X.509 Certificate(signing)** (Certificat SP X.509 (signature)) dans le bloc-notes et enregistrez-le sur votre ordinateur.

1. Cliquez sur **Upload XML file** (Charger le fichier XML) pour charger le fichier **XML de métadonnées de fédération** que vous avez copié à partir du Portail Azure.

    ![ Charger le fichier XML](./media/linkedin-talent-solutions-tutorial/xml-file.png)

### <a name="create-linkedin-talent-solutions-test-user"></a>Créer un utilisateur de test LinkedIn Talent Solutions

Dans cette section, un utilisateur appelé Britta Simon est créé dans LinkedIn Talent Solutions. LinkedIn Talent Solutions prend en charge l’attribution d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans LinkedIn Talent Solutions, il en est créé un après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

#### <a name="sp-initiated"></a>Lancée par le fournisseur de services :

* Cliquez sur **Tester cette application** dans le portail Azure. Cette opération redirige vers l’URL de connexion LinkedIn Talent Solutions, où vous pouvez lancer le processus de connexion.  

* Accédez directement à l’URL de connexion LinkedIn Talent Solutions et lancez le processus de connexion à partir de là.

#### <a name="idp-initiated"></a>Lancée par le fournisseur d’identité :

* Cliquez sur **Tester cette application** dans le portail Azure : vous devez être connecté automatiquement à l’instance de LinkedIn Talent Solutions pour laquelle vous avez configuré l’authentification unique. 

Vous pouvez aussi utiliser Mes applications de Microsoft pour tester l’application dans n’importe quel mode. Si, quand vous cliquez sur la vignette LinkedIn Talent Solutions dans Mes applications, le mode Fournisseur de services est configuré, vous êtes redirigé vers la page de connexion de l’application pour lancer le flux de connexion ; s’il s’agit du mode Fournisseur d’identité, vous êtes automatiquement connecté à l’instance LinkedIn Talent Solutions pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur Mes applications, consultez [Présentation de Mes applications](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré LinkedIn Talent Solutions, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).
