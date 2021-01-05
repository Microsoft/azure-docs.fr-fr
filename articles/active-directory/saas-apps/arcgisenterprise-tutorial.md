---
title: 'Tutoriel : Intégration d’Azure Active Directory à ArcGIS Enterprise | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et ArcGIS Enterprise.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/28/2018
ms.author: jeedes
ms.openlocfilehash: f7578972b054747c75cdbbc2371fc0bf35c6039a
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97672557"
---
# <a name="tutorial-azure-active-directory-integration-with-arcgis-enterprise"></a>Tutoriel : Intégration d’Azure Active Directory à ArcGIS Enterprise

Dans ce tutoriel, vous allez apprendre à intégrer ArcGIS Enterprise à Azure Active Directory (Azure AD).
L’intégration du logiciel ArcGIS Enterprise à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous contrôlez qui a accès à ArcGIS Enterprise.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à ArcGIS Enterprise (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à ArcGIS Enterprise, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement ArcGIS Enterprise pour lequel l’authentification unique est activée

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* ArcGIS Enterprise prend en charge l’authentification unique initiée par **le fournisseur de services et le fournisseur d’identité**
* ArcGIS Enterprise prend en charge l’attribution d’utilisateurs **Juste-à-temps**


## <a name="adding-arcgis-enterprise-from-the-gallery"></a>Ajout d’ArcGIS Enterprise à partir de la galerie

Pour configurer l’intégration d’ArcGIS Enterprise à Azure AD, vous devez ajouter ArcGIS Enterprise à votre liste d’applications SaaS gérées, à partir de la galerie.

**Pour ajouter ArcGIS Enterprise à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **ArcGIS Enterprise**, sélectionnez **ArcGIS Enterprise** dans le volet des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![ArcGIS Enterprise dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec [Nom de l’application] sur un utilisateur de test nommé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur [Nom de l’application] associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec [Nom de l’application], vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique ArcGIS Enterprise](#configure-arcgis-enterprise-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test ArcGIS Enterprise](#create-arcgis-enterprise-test-user)** pour obtenir un équivalent de Britta Simon dans ArcGIS Enterprise, qui soit lié à sa représentation dans Azure AD.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec [Nom de l’application], procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **ArcGIS Enterprise**, sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, suivez les étapes ci-dessous si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** :

    ![Capture d’écran montrant Configuration SAML de base, où vous pouvez entrer l’identificateur, l’URL de réponse, et sélectionner Enregistrer.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `<EXTERNAL_DNS_NAME>.portal`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://<EXTERNAL_DNS_NAME>/portal/sharing/rest/oauth2/saml/signin2`

    c. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    ![Capture d’écran montrant Définir des URL supplémentaires, où vous pouvez entrer une URL de connexion.](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://<EXTERNAL_DNS_NAME>/portal/sharing/rest/oauth2/saml/signin`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez [l’équipe du support technique ArcGIS Enterprise](mailto:support@esri.com). La valeur de l’identificateur peut être obtenue dans la section **Set Identity Provider** (Configurer le fournisseur d’identité), ce qui est expliqué plus loin dans ce tutoriel.

5. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur le bouton Copier pour copier l’**URL des métadonnées de fédération d’application**, puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

### <a name="configure-arcgis-enterprise-single-sign-on"></a>Configurer l’authentification unique ArcGIS Enterprise

1. Pour automatiser la configuration dans ArcGIS Enterprise, vous devez installer l’**extension de navigateur de connexion sécurisée à Mes applications** en cliquant sur **Installer l’extension**.

    ![Extension My apps](common/install-myappssecure-extension.png)

1. Une fois l’extension ajoutée au navigateur, le fait de cliquer sur **Configurer ArcGIS Enterprise** vous redirige vers l’application ArcGIS Enterprise. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à ArcGIS Enterprise. Cette extension de navigateur configure automatiquement l’application pour vous et automatise les étapes 3 à 7.

    ![Configuration](common/setup-sso.png)

1. Si vous voulez configurer ArcGIS Enterprise manuellement, connectez-vous à votre site d’entreprise ArcGIS Enterprise en tant qu’administrateur.


1. Sélectionnez **Organization > EDIT SETTINGS** (Organisation > Modifier les paramètres).

    ![Capture d’écran montrant l’onglet ArcGIS Enterprise Organization avec Edit settings sélectionné.](./media/arcgisenterprise-tutorial/configure1.png)

1. Sélectionnez l’onglet **Sécurité**.

    ![Capture d’écran montrant l’onglet Security sélectionné.](./media/arcgisenterprise-tutorial/configure2.png)

1. Faites défiler jusqu’à la section **Enterprise Logins via SAML** (Connexions d’entreprise via SAML), puis sélectionnez **SET ENTERPRISE LOGIN** (Définir les connexions d’entreprise).

    ![Capture d’écran montrant Enterprise Logins via SAML, où vous pouvez sélectionner Set Enterprise Login.](./media/arcgisenterprise-tutorial/configure3.png)

1. Dans la section **Set Identity Provider** (Configurer le fournisseur d’identité), effectuez les étapes suivantes :

    ![Capture d’écran montrant Set Identity Provider, où vous effectuez les étapes décrites ici.](./media/arcgisenterprise-tutorial/configure4.png)

    a. Fournissez un nom tel que **Azure Active Directory Test** dans la zone de texte **Name**.

    b. Dans la zone de texte **URL**, collez la valeur du champ **URL des métadonnées de fédération d’application** du portail Azure.

    c. Cliquez sur **Show advanced settings** (Afficher les paramètres avancés). Copiez la valeur qui se trouve sous **Entity ID** (ID d’entité), puis collez-la dans la zone de texte **Identificateur** de la section **Domaines et URL ArcGIS Enterprise** du portail Azure.

    ![Capture d’écran montrant où se procurer l’ID d’entité et mettre à jour le fournisseur d’identité.](./media/arcgisenterprise-tutorial/configure5.png)

    d. Cliquez sur **UPDATE IDENTITY PROVIDER** (Mettre à jour le fournisseur d’identité).

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom**, entrez **BrittaSimon**.

    b. Dans le champ **Nom d’utilisateur**, tapez **brittasimon\@domainedevotreentreprise.extension**.  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à ArcGIS Enterprise.

1. Dans le Portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **ArcGIS Enterprise**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, tapez et sélectionnez **ArcGIS Enterprise**.

    ![Lien ArcGIS Enterprise dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-arcgis-enterprise-test-user"></a>Créer un utilisateur de test ArcGIS Enterprise

Dans cette section, un utilisateur appelé Britta Simon est créé dans ArcGIS Enterprise. ArcGIS Enterprise prend en charge l’attribution d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans ArcGIS Enterprise, il en est créé un après l’authentification.

> [!Note]
> Si vous devez créer un utilisateur manuellement, contactez [l’équipe du support technique ArcGIS Enterprise](mailto:support@esri.com).

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette ArcGIS Enterprise dans le panneau d’accès doit vous connecter automatiquement à l’application ArcGIS Enterprise pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)