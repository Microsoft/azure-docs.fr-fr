---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Zoom | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Zoom.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/11/2020
ms.author: jeedes
ms.openlocfilehash: 2fd4346467b0cacbbb4eeb78072749e416b16de1
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358518"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-zoom"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Zoom

Dans ce tutoriel, vous allez découvrir comment intégrer Zoom dans Azure Active Directory (Azure AD). Quand vous intégrez Zoom à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Zoom.
* Permettre à vos utilisateurs de se connecter automatiquement à Zoom avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Zoom pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Zoom prend en charge l’authentification unique lancée par le **fournisseur de services**. 
* Zoom prend en charge le provisionnement d’utilisateurs [**automatisé**](./zoom-provisioning-tutorial.md).

## <a name="adding-zoom-from-the-gallery"></a>Ajout de Zoom à partir de la galerie

Pour configurer l’intégration de Zoom avec Azure AD, vous devez ajouter Zoom, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Zoom** dans la zone de recherche.
1. Sélectionnez **Zoom** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-zoom"></a>Configurer et tester l’authentification unique (SSO) Azure AD pour Zoom

Configurez et testez l’authentification unique Azure AD avec Zoom à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Zoom associé.

Pour configurer et tester l’authentification unique Azure AD avec Zoom, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
2. **[Configurer l’authentification unique Zoom](#configure-zoom-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Zoom](#create-zoom-test-user)** pour disposer, dans Zoom, d’un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
3. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Zoom**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, effectuez les étapes suivantes :

    a. Dans la zone de texte **URL de connexion**, saisissez une URL au format suivant : `https://<companyname>.zoom.us`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `<companyname>.zoom.us`

    c. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://<companyname>.zoom.us`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique Zoom](https://support.zoom.us/hc/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **Certificat (Base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Zoom**, copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

> [!NOTE]
> Pour découvrir comment configurer un rôle dans Azure AD, consultez [Configurer les revendications de rôle émises dans le jeton SAML pour les applications d’entreprise](../develop/active-directory-enterprise-app-role-management.md).

> [!NOTE]
> Zoom peut s’attendre à une revendication de groupe dans la charge utile SAML. Si vous avez créé des groupes, contactez l’[équipe de support client de Zoom](https://support.zoom.us/hc/) et donnez-lui les informations des groupes afin qu’elle puisse configurer les informations de groupe de son côté. Vous devez également fournir l’ID d’objet à l’[équipe de support client de Zoom](https://support.zoom.us/hc/) afin qu’elle le configure de son côté. Pour obtenir l’ID d’objet, consultez [Configuration de Zoom avec Azure](https://support.zoom.us/hc/articles/115005887566).

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Zoom.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Zoom**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-zoom-sso"></a>Configurer l’authentification unique Zoom

1. Pour automatiser la configuration dans Zoom, vous devez installer l’**extension de navigateur de connexion sécurisée à Mes applications** en cliquant sur **Installer l’extension**.

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Zoom** pour être dirigé vers l’application Zoom. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Zoom. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 6.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer Zoom manuellement, dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise Zoom en tant qu’administrateur.

2. Cliquez sur **Single Sign-On**.

    ![Onglet Single Sign-On](./media/zoom-tutorial/zoom-sso1.png "Authentification unique")

3. Cliquez sur l'onglet **Security Control** et accédez aux paramètres **Single Sign-On**.

4. Dans la section Single Sign-On, procédez comme suit :

    ![Section Single Sign-On](./media/zoom-tutorial/zoom-sso2.png "Authentification unique")

    a. Dans la zone de texte **URL de la page de connexion**, collez la valeur **URL de connexion** que vous avez copiée à partir du portail Azure.

    b. Pour la valeur d’**URL de la page de déconnexion** valeur, vous devez vous rendre sur le portail Azure et cliquer sur **Azure Active Directory** à gauche, puis accéder à **Inscriptions d’applications**.

    ![Bouton Azure Active Directory](./media/zoom-tutorial/appreg.png)

    c. Cliquez sur **Points de terminaison**

    ![Bouton Point de terminaison](./media/zoom-tutorial/endpoint.png)

    d. Copiez **SAML-P SIGN-OUT ENDPOINT** et collez-le dans la zone de texte **URL de la page de déconnexion**.

    ![Bouton Copier le point de terminaison](./media/zoom-tutorial/endpoint1.png)

    e. Ouvrez votre certificat codé en base 64 dans le Bloc-notes, copiez le contenu de celui-ci dans le Presse-papiers, puis collez-le dans la zone de texte **Certificat de fournisseur d'identité**.

    f. Dans la zone de texte **Émetteur**, collez la valeur **Identificateur Azure AD** que vous avez copiée à partir du portail Azure. 

    g. Cliquez sur **Enregistrer les modifications**.

    > [!NOTE]
    > Pour plus d’informations, consultez la documentation de Zoom à l’adresse [https://zoomus.zendesk.com/hc/articles/115005887566](https://zoomus.zendesk.com/hc/articles/115005887566)

### <a name="create-zoom-test-user"></a>Créer un utilisateur de test Zoom

L’objectif de cette section est de créer un utilisateur appelé B.Simon dans Zoom. Zoom prend en charge le provisionnement automatique d’utilisateurs, option activée par défaut. Vous trouverez plus d’informations [ici](./zoom-provisioning-tutorial.md) sur la façon de configurer l’attribution automatique d’utilisateurs.

> [!NOTE]
> Si vous devez créer un utilisateur manuellement, contactez l’[équipe du support technique de Zoom](https://support.zoom.us/hc/)

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes.

* Cliquez sur **Tester cette application** dans le portail Azure. Cette opération redirige vers l’URL de connexion Zoom, où vous pouvez lancer le processus de connexion.

* Accédez directement à l’URL de connexion Zoom pour lancer le processus de connexion.

* Vous pouvez utiliser Mes applications de Microsoft. Le fait de cliquer sur la vignette Zoom dans Mes applications vous redirige vers l’URL de connexion Zoom. Pour plus d’informations sur Mes applications, consultez [Présentation de Mes applications](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Azure AD Zoom, vous pouvez appliquer le contrôle de session qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)
