---
title: 'Didacticiel : Intégration d’Azure Active Directory à Zscaler Private Access Administrator | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Zscaler Private Access Administrator.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/03/2019
ms.author: jeedes
ms.openlocfilehash: 803f5c25429b33e6d1af130196d38e125b410eb4
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92890987"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-private-access-administrator"></a>Didacticiel : Intégration d’Azure Active Directory à Zscaler Private Access Administrator

Dans ce didacticiel, vous allez apprendre à intégrer Zscaler Private Access Administrator à Azure Active Directory (Azure AD).
L’intégration de Zscaler Private Access Administrator dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Zscaler Private Access Administrator.
* Vous pouvez permettre à vos utilisateurs de se connecter automatiquement à Zscaler Private Access Administrator (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Zscaler Private Access Administrator, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Un abonnement Zscaler Private Access Administrator pour lequel l’authentification unique est activée

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Zscaler Private Access Administrator prend en charge l’authentification unique lancée par le **fournisseur de services** et le **fournisseur d’identité**

## <a name="adding-zscaler-private-access-administrator-from-the-gallery"></a>Ajout de Zscaler Private Access Administrator à partir de la galerie

Pour configurer l’intégration de Zscaler Private Access Administrator à Azure AD, vous devez ajouter Zscaler Private Access Administrator à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Zscaler Private Access Administrator à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Zscaler Private Access Administrator** , sélectionnez **Zscaler Private Access Administrator** dans le panneau des résultats puis cliquez sur le bouton **Ajouter** de l’application.

    ![Zscaler Private Access Administrator dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Zscaler Private Access Administrator avec un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Zscaler Private Access Administrator associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Zscaler Private Access Administrator, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Zscaler Private Access Administrator](#configure-zscaler-private-access-administrator-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Zscaler Private Access Administrator](#create-zscaler-private-access-administrator-test-user)** pour avoir dans Zscaler Private Access Administrator un équivalent de Britta Simon lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Zscaler Private Access Administrator, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), sur la page d’intégration de l’application **Zscaler Private Access Administrator** , sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , effectuez les étapes suivantes :

    ![Capture d’écran montrant Basic SAML Configuration où vous pouvez entrer les valeurs décrites.](common/idp-relay.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<subdomain>.private.zscaler.com/auth/metadata`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<subdomain>.private.zscaler.com/auth/sso`

    c. Cliquez sur **Définir des URL supplémentaires**.

    d. Dans la zone de texte **État de relais** , tapez une URL : `idpadminsso`

5.  Si vous souhaitez configurer l’application en mode initié par le **fournisseur de service** , procédez comme suit :

    ![Capture d’écran montrant Définir des URL supplémentaires, où vous pouvez entrer une URL de connexion.](common/both-signonurl.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<subdomain>.private.zscaler.com/auth/sso`.   

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Contactez l’[équipe de support Zscaler Private Access Administrator](https://help.zscaler.com/zpa-submit-ticket) pour obtenir ces valeurs. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

6. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

7. Dans la section **Configurer Zscaler Private Access Administrator** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-zscaler-private-access-administrator-single-sign-on"></a>Configurer l’authentification unique Zscaler Private Access Administrator

1. Dans une autre fenêtre de navigateur web, connectez-vous à Zscaler Private Access Administrator en tant qu’administrateur.

2. En haut, cliquez sur **Administration** et allez à la section **AUTHENTIFICATION** puis cliquez sur **Configuration IdP**.

    ![Administrateur Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_admin.png)

3. Dans le coin supérieur droit, cliquez sur **Ajouter une configuration IdP**. 

    ![Ajouter IdP Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_addpidp.png)

4. Sur la **page Ajouter une configuration IdP** , procédez comme suit :
 
    ![Zscaler Private Access Administrator idpselect](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_idpselect.png)

    a. Cliquez sur **Sélectionner fichier** pour charger le fichier de métadonnées téléchargé depuis Azure AD dans le champ **Charger fichier de métadonnée IdP**.

    b. Les **métadonnées IdP** sont lu à partir d’Azure AD et tous les champs sont renseignés, comme montré ci-dessous.

    ![Zscaler Private Access Administrator idpconfig](./media/zscalerprivateaccessadministrator-tutorial/idpconfig.png)

    c. Sélectionnez **Authentification unique** comme **Administrateur**.

    d. Sélectionnez votre domaine dans le champ **Domaines**.
    
    e. Cliquez sur **Enregistrer**.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon**.
  
    b. Dans le champ **Nom d’utilisateur** , tapez `brittasimon@yourcompanydomain.extension`. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Zscaler Private Access Administrator.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Zscaler Private Access Administrator**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Zscaler Private Access Administrator**.

    ![Lien Zscaler Private Access Administrator dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

### <a name="create-zscaler-private-access-administrator-test-user"></a>Créer un utilisateur de test Zscaler Private Access Administrator

Pour pouvoir se connecter à Zscaler Private Access Administrator, les utilisateurs Azure AD doivent être provisionnés dans Zscaler Private Access Administrator. Dans le cas de Zscaler Private Access Administrator, la configuration est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise Zscaler Private Access Administrator en tant qu’administrateur.

2. En haut, cliquez sur **Administration** et allez à la section **AUTHENTIFICATION** puis cliquez sur **Configuration IdP**.

    ![Administrateur Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_admin.png)

3. Cliquez sur **Administrateurs** à gauche du menu.

    ![Administrateur Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_adminstrator.png)

4. Dans le coin supérieur droit, cliquez sur **Ajouter un administrateur** :

    ![Ajouter un administrateur Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_addadmin.png)

5. Dans la page **Ajouter un administrateur** , procédez comme suit :

    ![Administrateur d’utilisateurs Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_useradmin.png)

    a. Dans la zone de texte **Nom d’utilisateur** , entrez l’adresse e-mail de l’utilisateur, par exemple BrittaSimon@contoso.com.

    b. Dans la zone de texte **Mot de passe** , tapez le mot de passe.

    c. Dans la zone de texte **Confirmer le mot de passe** , entrez le mot de passe.

    d. Sélectionnez **Rôle** pour **Zscaler Private Access Administrator**.

    e. Dans la zone de texte **E-mail** , entrez l’adresse e-mail de l’utilisateur, par exemple BrittaSimon@contoso.com.

    f. Dans la zone de texte **Téléphone** , tapez le numéro de téléphone.

    g. Dans la zone de texte **Fuseau horaire** , sélectionnez le fuseau horaire.

    h. Cliquez sur **Enregistrer**.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Zscaler Private Access Administrator dans le volet d’accès, vous devez être connecté automatiquement à l’application Zscaler Private Access Administrator pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)