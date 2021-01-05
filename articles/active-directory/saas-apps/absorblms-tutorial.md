---
title: 'Didacticiel : Intégration d’Azure Active Directory à Absorb LMS | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Absorb LMS.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/02/2019
ms.author: jeedes
ms.openlocfilehash: 3d90d35e113b5f9757faf59681bb2532b66f2b09
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97673866"
---
# <a name="tutorial-azure-active-directory-integration-with-absorb-lms"></a>Didacticiel : Intégration d’Azure Active Directory avec Absorb LMS

Dans ce didacticiel, vous allez apprendre à intégrer Absorb LMS avec Azure Active Directory (Azure AD).
L’intégration d’Absorb LMS avec Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Absorb LMS.
* Vous pouvez permettre à vos utilisateurs d’être connectés automatiquement à Absorb LMS (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec Absorb LMS, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Abonnement Absorb LMS pour lequel l’authentification unique est activée

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Absorb LMS prend en charge l’authentification unique initiée par **IDP**

## <a name="adding-absorb-lms-from-the-gallery"></a>Ajout d’Absorb LMS à partir de la galerie

Pour configurer l’intégration d’Absorb LMS avec Azure AD, vous devez ajouter Absorb LMS à partir de la galerie dans votre liste d’applications SaaS gérées.

**Pour ajouter Absorb LMS à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Absorb LMS**, sélectionnez **Absorb LMS** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Absorb LMS dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Absorb LMS, avec un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Absorb LMS associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Absorb LMS, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Absorb LMS](#configure-absorb-lms-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur test Absorb LMS](#create-absorb-lms-test-user)** pour avoir un équivalent de Britta Simon dans Absorb LMS lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Absorb LMS, procédez comme suit :

1. Dans le [Portail Azure](https://portal.azure.com/), sur la page d’intégration de l’application **Absorb LMS**, sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Sur la page **Configurer l’authentification unique avec SAML**, cliquez sur le bouton **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Informations d’authentification unique dans Domaine et URL Absorb LMS](common/idp-intiated.png)

    Avec **Absorb 5 - UI**, utilisez la configuration suivante :

    a. Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `https://company.myabsorb.com/account/saml`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://company.myabsorb.com/account/saml`

    Avec **Absorb 5 - New Learner Experience**, utilisez la configuration suivante :

    a. Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `https://company.myabsorb.com/api/rest/v2/authentication/saml`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://company.myabsorb.com/api/rest/v2/authentication/saml`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique Absorb LMS](https://support.absorblms.com/hc/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. La capture d’écran suivante montre la liste des attributs par défaut, où **nameidentifier** est mappé avec **user.userprincipalname**.

    ![image](common/edit-attribute.png)

6. Sur la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

7. Dans la section **Configurer Absorb LMS**, copiez la ou les URL appropriées, selon vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-absorb-lms-single-sign-on"></a>Configurer l’authentification unique Absorb LMS

1. Dans une nouvelle fenêtre de navigateur web, connectez-vous au site de votre entreprise Absorb LMS en tant qu’administrateur.

2. Sélectionnez le bouton **Compte** en haut à droite.

    ![Bouton Compte](./media/absorblms-tutorial/1.png)

3. Dans le volet Compte, sélectionnez **Paramètres du portail**.

    ![Lien Paramètres du portail](./media/absorblms-tutorial/2.png)

4. Sélectionnez l’onglet **Manage SSO Settings** (Gérer les paramètres d’authentification unique).

    ![L’onglet Utilisateurs](./media/absorblms-tutorial/managesso.png)

5. Dans la page **Manage Single Sign-On Settings** (Gérer les paramètres d’authentification unique), procédez comme suit :

    ![Page Configuration d’authentification unique](./media/absorblms-tutorial/settings.png)

    a. Dans la zone de texte **Nom**, entrez un nom, par exemple Azure AD Marketplace SSO.

    b. Sélectionnez **SAML** comme **Méthode**.

    c. Dans le bloc-notes, ouvrez le certificat que vous avez téléchargé à partir du portail Azure. Supprimez les balises **-----BEGIN CERTIFICATE-----** et **-----END CERTIFICATE-----**. Puis, dans la zone **Clé**, collez le reste du contenu.

    d. Dans la zone **Mode**, sélectionnez **Identity Provider Initiated** (Initiée par le fournisseur d’identité).

    e. Dans la zone **Propriété ID** , sélectionnez l’attribut que vous avez configuré comme identificateur d’utilisateur dans Azure AD. Par exemple, si *nameidentifier* est sélectionné dans Azure AD, sélectionnez **Nom d’utilisateur**.

    f. Sélectionnez **Sha256** comme **Type de signature**.

    g. Dans la zone **Login URL** (URL de connexion), collez l’**URL d’accès utilisateur** indiquée dans la page **Propriétés** du portail Azure.

    h. Dans **Logout URL** (URL de déconnexion), collez la valeur de l’**URL de déconnexion** que vous avez copiée depuis la fenêtre **Configurer l’authentification** du portail Azure.

    i. Faites basculer **Automatically Redirect** (Redirection automatique) sur **Activé**.

6. Sélectionnez **Enregistrer**.

    ![Activer/désactiver Autoriser uniquement la connexion SSO](./media/absorblms-tutorial/save.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom**, entrez **BrittaSimon**.

    b. Dans le champ **Nom d’utilisateur**, tapez `brittasimon\@yourcompanydomain.extension`  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Absorb LMS.

1. Dans le Portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **Absorb LMS**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, entrez et sélectionnez **Absorb LMS**.

    ![Lien Absorb LMS dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-absorb-lms-test-user"></a>Créer un utilisateur de test Absorb LMS

Les utilisateurs Azure AD désirant se connecter à Absorb LMS doivent être configurés dans Absorb LMS. Dans le cas d’Absorb LMS, l’approvisionnement est une tâche manuelle.

**Pour configurer l'approvisionnement des utilisateurs, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise Absorb LMS en tant qu’administrateur.

2. Dans le volet **Utilisateurs**, sélectionnez **Utilisateurs**.

    ![Lien Utilisateurs](./media/absorblms-tutorial/absorblms_userssub.png)

3. Sélectionnez l’onglet **Utilisateur**.

    ![Liste déroulante Ajouter un nouveau](./media/absorblms-tutorial/absorblms_createuser.png)

4. Dans la page **Ajouter un utilisateur**, procédez comme suit :

    ![Page Ajouter un utilisateur](./media/absorblms-tutorial/user.png)

    a. Dans la zone **Prénom**, tapez le prénom, par exemple **Britta**.

    b. Dans la zone **Nom**, tapez le nom de famille, par exemple **Simon**.

    c. Dans la zone **Nom d’utilisateur**, tapez le nom complet, par exemple **Britta Simon**.

    d. Dans la zone **Mot de passe**, entrez le mot de passe utilisateur.

    e. Dans la zone **Confirmer le mot de passe**, saisissez de nouveau le mot de passe.

    f. Configurez le réglage **Est actif** sur **Actif**.

5. Sélectionnez **Enregistrer**.

    ![Activer/désactiver Autoriser uniquement la connexion SSO](./media/absorblms-tutorial/save.png)

    > [!NOTE]
    > Par défaut, l’approvisionnement d’utilisateurs n’est pas activé dans l’authentification unique. Si le client souhaite activer cette fonctionnalité, il doit la définir comme indiqué dans [cette](https://support.absorblms.com/hc/en-us/articles/360014083294-Incoming-SAML-2-0-SSO-Account-Provisioning) documentation. Notez également que l’approvisionnement d’utilisateurs est uniquement disponible avec **Absorb 5 - New Learner Experience** avec l’URL ACS-`https://company.myabsorb.com/api/rest/v2/authentication/saml`

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Absorb LMS dans le panneau d’accès doit vous connecte automatiquement à l’application Absorb LMS pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)