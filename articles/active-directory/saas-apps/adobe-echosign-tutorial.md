---
title: 'Didacticiel : Intégration d’Azure Active Directory à Adobe Sign | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Adobe Sign.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/19/2018
ms.author: jeedes
ms.openlocfilehash: bd7d05225ef0583c8d56804fb2164cfc73ad4a6a
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97673288"
---
# <a name="tutorial-azure-active-directory-integration-with-adobe-sign"></a>Didacticiel : Intégration d’Azure Active Directory à Adobe Sign

Dans ce didacticiel, vous allez apprendre à intégrer Adobe Sign à Azure Active Directory (Azure AD).
L’intégration d’Adobe Sign dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Adobe Sign.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Adobe Sign (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Adobe Sign, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Adobe Sign pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Adobe Sign prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-adobe-sign-from-the-gallery"></a>Ajout d’Adobe Sign à partir de la galerie

Pour configurer l’intégration d’Adobe Sign à Azure AD, vous devez ajouter Adobe Sign depuis la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Adobe Sign à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Adobe Sign**, sélectionnez **Adobe Sign** dans le volet des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Adobe Sign dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous configurez et testez l’authentification unique Azure AD avec Adobe Sign au moyen d’un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Adobe Sign associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Adobe Sign, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Adobe Sign](#configure-adobe-sign-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Adobe Sign](#create-adobe-sign-test-user)** pour avoir dans Adobe Sign un équivalent de Britta Simon lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Adobe Sign, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Adobe Sign**, sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, effectuez les étapes suivantes :

    ![Informations relatives à l’authentification unique des URL et du domaine Adobe Sign](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion**, saisissez une URL au format suivant : `https://<companyname>.echosign.com/`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<companyname>.echosign.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe du support technique d’Adobe Sign](https://helpx.adobe.com/in/contact/support.html). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

4. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer Adobe Sign**, copiez la ou les URL appropriées, selon vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-adobe-sign-single-sign-on"></a>Configurer l’authentification unique Adobe Sign

1. Avant la configuration, contactez l’[équipe du support technique Adobe Sign](https://helpx.adobe.com/in/contact/support.html) pour ajouter votre domaine à la liste verte Adobe Sign. Voici comment ajouter le domaine :

    a. L’[équipe de support client d’Adobe Sign](https://helpx.adobe.com/in/contact/support.html) vous enverra un jeton généré aléatoirement. Pour votre domaine, le jeton sera dans ce format : **adobe-sign-verification= xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx**

    b. Publiez le jeton de vérification dans un enregistrement DNS texte et informez [l’équipe de support client d’Adobe Sign](https://helpx.adobe.com/in/contact/support.html).

    > [!NOTE]
    > Cela peut prendre plusieurs jours, voire plus. Notez que les retards de propagation DNS signifient qu’une valeur publiée dans le DNS peut ne pas être visible pendant une heure ou plus. Votre administrateur informatique doit savoir comment publier ce jeton dans un enregistrement DNS texte.

    c. Après que vous aurez averti [l’équipe de support client d’Adobe Sign](https://helpx.adobe.com/in/contact/support.html) via le ticket de support, une fois le jeton publié, elle validera le domaine et l’ajoutera à votre compte.

    d. Il faut généralement procéder comme suit pour publier le jeton dans un enregistrement DNS :

    * Se connecter à son compte de domaine
    * Rechercher la page de mise à jour de l’enregistrement DNS. Cette page peut s’appeler Gestion DNS, Gestion des noms de serveur ou Paramètres avancés.
    * Trouver les enregistrements TXT pour votre domaine.
    * Ajouter un enregistrement TXT avec la valeur complète du jeton fournie par Adobe.
    * Enregistrez vos modifications.

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise Adobe Sign en tant qu’administrateur.

1. Dans le menu SAML, sélectionnez **Paramètres de compte** > **Paramètres SAML**.

    ![Capture d’écran de la page Paramètres SAML d’Adobe Sign](./media/adobe-echosign-tutorial/ic789520.png "Compte")

1. Dans la section **SAML Settings** (Paramètres SAML), procédez comme suit :

    ![Capture d’écran mettant en évidence les paramètres SAML, notamment SAML Mandatory.](./media/adobe-echosign-tutorial/ic789521.png "SAML Settings")

   ![Capture d’écran des paramètres SAML](./media/adobe-echosign-tutorial/ic789522.png "SAML Settings")

   a. Sous **Mode SAML**, sélectionnez **SAML obligatoire**.

   b. Sélectionnez **Autoriser les administrateurs de compte EchoSign à se connecter avec leurs informations d'identification EchoSign** .

   c. Sous **Création d’utilisateurs**, sélectionnez **Ajouter automatiquement les utilisateurs authentifiés via SAML**.

   d. Collez la valeur **Identificateur Azure AD**, que vous avez copiée dans le portail Azure, dans la zone de texte **Idp Entity ID** (ID d’entité du fournisseur d’identité).

   e. Collez l’**URL de connexion**, que vous avez copiée dans le portail Azure, dans la zone de texte **Idp Login URL** (URL de connexion du fournisseur d’identité).

   f. Collez l’**URL de déconnexion**, que vous avez copiée dans le portail Azure, dans la zone de texte **Idp Logout URL** (URL de déconnexion du fournisseur d’identité).

   g. Ouvrez votre **Certificat (Base64)** téléchargé dans le Bloc-notes. Copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **Certificat IdP**.

   h. Sélectionnez **Enregistrer les modifications**.

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

Dans cette section, vous autorisez Britta Simon à utiliser l’authentification unique Azure en accordant l’accès à Adobe Sign.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **Adobe Sign**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, tapez et sélectionnez **Adobe Sign**.

    ![Lien Adobe Sign dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-adobe-sign-test-user"></a>Créer un utilisateur de test Adobe Sign

Pour pouvoir se connecter à Adobe Sign, les utilisateurs d’Azure AD doivent être attribués dans Adobe Sign. Il s’agit d’une tâche manuelle.

>[!NOTE]
>Vous pouvez utiliser tout autre outil ou API de création de compte d’utilisateur fourni par Adobe Sign pour approvisionner des comptes d’utilisateurs Azure AD. 

1. Connectez-vous à votre site d’entreprise **Adobe Sign** en tant qu’administrateur.

2. Dans le menu du haut, sélectionnez **Compte**. Puis, dans le volet gauche, sélectionnez **Utilisateurs et groupes** > **Créer un utilisateur**.

    ![Capture d’écran du site d’entreprise Adobe Sign, avec Compte, Utilisateurs et groupes et Créer un utilisateur mis en surbrillance](./media/adobe-echosign-tutorial/ic789524.png "Compte")

3. Dans la section **Create New User** (Créer un utilisateur), procédez comme suit :

    ![Capture d’écran de la section Créer un utilisateur](./media/adobe-echosign-tutorial/ic789525.png "Create User")

    a. Tapez l’**Adresse e-mail**, le **Prénom** et le **Nom** d’un compte Azure AD valide que vous souhaitez provisionner dans les zones de texte correspondantes.

    b. Sélectionnez **Créer un utilisateur**.

>[!NOTE]
>Le titulaire du compte Azure Active Directory reçoit un e-mail contenant un lien pour confirmer le compte avant qu’il ne soit activé. 

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Adobe Sign dans le panneau d’accès doit vous connecter automatiquement à l’application Adobe Sign pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)