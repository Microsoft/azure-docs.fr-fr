---
title: 'Didacticiel : Intégration d’Azure Active Directory à Adobe Experience Manager | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Adobe Experience Manager.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/17/2019
ms.author: jeedes
ms.openlocfilehash: 9a98a77b9cc89b7a1a05e676048775aa38c83733
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97671964"
---
# <a name="tutorial-azure-active-directory-integration-with-adobe-experience-manager"></a>Didacticiel : Intégration d’Azure Active Directory à Adobe Experience Manager

L’objectif de ce didacticiel est de vous apprendre à intégrer Adobe Experience Manager à Azure Active Directory (Azure AD).
L’intégration d’Adobe Experience Manager à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Adobe Experience Manager.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Adobe Experience Manager (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Adobe Experience Manager, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement à Adobe Experience Manager pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Adobe Experience Manager prend en charge l’authentification unique initiée par **le fournisseur de services et le fournisseur d’identité**

* Adobe Experience Manager prend en charge l’attribution d’utilisateurs **juste-à-temps**.

## <a name="adding-adobe-experience-manager-from-the-gallery"></a>Ajout d’Adobe Experience Manager à partir de la galerie

Pour configurer l’intégration d’Adobe Experience Manager à Azure AD, vous devez ajouter Adobe Experience Manager à votre liste d’applications SaaS gérées, à partir de la galerie.

**Pour ajouter Adobe Experience Manager à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Adobe Experience Manager**, sélectionnez **Adobe Experience Manager** dans le volet des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Adobe Experience Manager dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec [Nom de l’application] sur un utilisateur de test nommé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur [Nom de l’application] associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec [Nom de l’application], vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Adobe Experience Manager](#configure-adobe-experience-manager-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Adobe Experience Manager](#create-adobe-experience-manager-test-user)** pour avoir un équivalent de Britta Simon dans Adobe Experience Manager, lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec [Nom de l’application], procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Adobe Experience Manager**, sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. À la section **Configuration SAML de base**, si vous souhaitez configurer l’application en mode initié par **IDP**, suivez les étapes ci-dessous :

    ![Capture d’écran montrant la section Basic SAML Configuration et mettant en évidence les zones de texte Identifier et Reply URL.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur**, tapez une valeur unique que vous définissez sur votre serveur AEM également.

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://<AEM Server Url>/saml_login`

    > [!NOTE]
    > La valeur de l’URL de réponse n’est pas réelle. Mettez à jour la valeur avec l’URL de réponse réelle. Pour obtenir cette valeur, contactez l’[équipe du support technique Adobe Experience Manager](https://helpx.adobe.com/support/experience-manager.html). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Adobe Experience Manager Domain and URLs (Domaines et URL Adobe Experience Manager)](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion**, tapez l’URL de votre serveur Adobe Experience Manager.

6. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

7. Dans la section **Configurer Adobe Experience Manager**, copiez la ou les URL appropriées correspondant à vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-adobe-experience-manager-single-sign-on"></a>Configurer l’authentification unique Adobe Experience Manager

1. Ouvrez le portail d’administration **Adobe Experience Manager** dans une autre fenêtre de navigateur.

2. Sélectionnez **Paramètres** > **Sécurité** > **Utilisateurs**.

    ![Capture d’écran montrant la vignette Users dans Adobe Experience Manager.](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_user.png)

3. Sélectionnez **Administrateur** ou tout autre utilisateur approprié.

    ![Capture d’écran mettant en évidence l’utilisateur Administrator.](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin6.png)

4. Sélectionnez **Paramètres du compte** > **Gérer TrustStore**.

    ![Capture d’écran montrant Manage TrustStore sous Account settings.](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_managetrust.png)

5. Sous **Ajouter un certificat à partir d’un fichier CER**, cliquez sur **Sélectionner un fichier de certificat**. Accédez au fichier de certificat que vous avez téléchargé à partir du portail Azure et sélectionnez-le.

    ![Capture d’écran mettant en évidence le bouton Select Certificate File.](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_user2.png)

6. Le certificat est ajouté au TrustStore. Notez l’alias du certificat.

    ![Capture d’écran montrant que le certificat est ajouté au TrustStore.](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin7.png)

7. Sur la page **Utilisateurs**, sélectionnez **authentication-service**.

    ![Capture d’écran mettant en évidence authentication-service sur l’écran.](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin8.png)

8. Sélectionnez **Paramètres du compte** > **Create/Manage KeyStore** (Créer/gérer KeyStore). Créez KeyStore en fournissant un mot de passe.

    ![Capture d’écran mettant en évidence Manage KeyStore.](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin9.png)

9. Revenez à l’écran d’administrateur. Puis sélectionnez **Paramètres** > **Opérations** > **Console Web**.

    ![Capture d’écran mettant Web Console sous Operations dans la section Settings.](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin1.png)

    La page de configuration s’ouvre.

    ![Bouton Enregistrer de la page Configurer l’authentification unique](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin2.png)

10. Rechercher **Adobe Granite SAML 2.0 Authentication Handler**. Puis cliquez sur l’icône **Ajouter**.

    ![Capture d’écran mettant en évidence Adobe Granite SAML 2.0 Authentication Handler.](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin3.png)

11. Sur cette page, effectuez les actions suivantes.

    ![Bouton Enregistrer de la page Configurer l’authentification unique](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin4.png)

    a. Dans la zone **Chemin d’accès**, entrez **/** .

    b. Dans la zone **IDP URL** (URL IDP), entrez la valeur **URL de connexion** copiée à partir du portail Azure.

    c. Dans la zone **Alias du certificat IDP**, entrez la valeur **Alias du certificat** que vous avez ajoutée dans TrustStore.

    d. Dans la zone **Security Provided Entity ID** (ID d’entité fourni par la sécurité), entrez la valeur d’**Identificateur Azure AD** unique que vous avez configurée dans le portail Azure.

    e. Dans la zone **URL Assertion Consumer Service**, entrez la valeur **URL de réponse** que vous avez configurée dans le portail Azure.

    f. Dans la zone **Mot de passe de Key Store**, entrez le **mot de passe** que vous avez défini dans KeyStore.

    g. Dans la zone **ID d’attribut utilisateur**, entrez **l’ID de nom** ou un autre ID d’utilisateur pertinent.

    h. Sélectionnez **Créer automatiquement des utilisateurs CRX**.

    i. Dans la zone **Logout URL** (URL de déconnexion), entre la valeur unique **URL de déconnexion** obtenue à partir du portail Azure.

    j. Sélectionnez **Enregistrer**.

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Adobe Experience Manager.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis sélectionnez **Adobe Experience Manager**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Adobe Experience Manager**.

    ![Lien Adobe Experience Manager dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-adobe-experience-manager-test-user"></a>Créer un utilisateur de test Adobe Experience Manager

Dans cette section, vous créez un utilisateur appelé Britta Simon dans Adobe Experience Manager. Si vous avez sélectionné l’option **Autocreate CRX Users** (Créer automatiquement des utilisateurs CRX), les utilisateurs sont créés automatiquement après une authentification réussie.

Si vous souhaitez créer manuellement des utilisateurs, collaborez avec [l’équipe de support technique Adobe Experience Manager](https://helpx.adobe.com/support/experience-manager.html) pour ajouter les utilisateurs dans la plateforme Adobe Experience Manager.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Adobe Experience Manager dans le volet d’accès, vous devez être connecté automatiquement à l’application Adobe Experience Manager pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)