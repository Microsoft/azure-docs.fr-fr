---
title: 'Didacticiel : Intégration d’Azure Active Directory à TOPdesk - Secure | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et TOPdesk - Secure.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/27/2018
ms.author: jeedes
ms.openlocfilehash: 1dd57dd31c0dcf42b7847dc8e3f3945961c3d37f
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92520389"
---
# <a name="tutorial-azure-active-directory-integration-with-topdesk---secure"></a>Didacticiel : Intégration d’Azure Active Directory à TOPdesk - Secure

Dans ce didacticiel, vous allez apprendre à intégrer TOPdesk - Secure à Azure Active Directory (Azure AD).
L’intégration de TOPdesk - Secure à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à TOPdesk - Secure.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à TOPdesk - Secure (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à TOPdesk - Secure, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement TOPdesk - Secure pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* TOPdesk - Secure prend en charge l’authentification unique initiée par le **fournisseur de services**

## <a name="adding-topdesk---secure-from-the-gallery"></a>Ajout de TOPdesk - Secure à partir de la galerie

Pour configurer l’intégration de TOPdesk - Secure à Azure AD, vous devez ajouter TOPdesk - Secure à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter TOPdesk - Secure à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **TOPdesk - Secure** , sélectionnez **TOPdesk - Secure** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![TOPdesk - Secure dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec TOPdesk - Secure avec un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur TOPdesk - Secure associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec TOPdesk - Secure, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique TOPdesk - Secure](#configure-topdesk---secure-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test TOPdesk - Secure](#create-topdesk---secure-test-user)** pour avoir un équivalent de Britta Simon dans TOPdesk - Secure lié à la représentation de l’utilisateur dans Azure AD.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec TOPdesk - Secure, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **TOPdesk - Secure** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL TOPdesk - Secure](common/sp-identifier-reply.png)

    a. Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<companyname>.topdesk.net`.

    b. Dans la zone **URL d’identificateur** , renseignez l’URL des métadonnées TOPdesk que vous pouvez récupérer à partir de la configuration TOPdesk. Elle doit utiliser le modèle suivant : `https://<companyname>.topdesk.net/saml-metadata/<identifier>`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<companyname>.topdesk.net/tas/secure/login/verify`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Contactez l’[équipe de support technique TOPdesk - Secure](https://www.topdesk.com/us/support/) pour obtenir ces valeurs. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configurer TOPdesk - Secure** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-topdesk---secure-single-sign-on"></a>Configurer l’authentification unique TOPdesk - Secure

1. Connectez-vous à votre site d’entreprise **TOPdesk - Secure** en tant qu’administrateur.

2. Dans le menu **TOPdesk** , cliquez sur **Settings** .

    ![Paramètres](./media/topdesk-secure-tutorial/ic790598.png "Paramètres")

3. Cliquez sur **Login Settings** .

    ![Paramètres de connexion](./media/topdesk-secure-tutorial/ic790599.png "Login Settings")

4. Développez le menu **Login Settings** , puis cliquez sur **General** .

    ![Généralités](./media/topdesk-secure-tutorial/ic790600.png "Général")

5. Dans la section **Secure** de la section de configuration **SAML login** , procédez comme suit :

    ![Paramètres techniques](./media/topdesk-secure-tutorial/ic790855.png "Technical Settings")

    a. Cliquez sur **Download** pour télécharger le fichier de métadonnées public et enregistrez-le en local sur votre ordinateur.

    b. Ouvrez le fichier de métadonnées et recherchez le nœud **AssertionConsumerService** .

    ![Assertion Consumer Service](./media/topdesk-secure-tutorial/ic790856.png "Assertion Consumer Service")

    c. Copiez la valeur **AssertionConsumerService** , collez-la dans la zone de texte URL de réponse dans la section **Domaine et URL TOPdesk - Secure** .

6. Pour créer un fichier de certificat, procédez comme suit :

    ![Certificate](./media/topdesk-secure-tutorial/ic790606.png "Certificat")

    a. Ouvrez le fichier de métadonnées téléchargé à partir du portail Azure.

    b. Développez le nœud **RoleDescriptor** dont le **xsi:type** est **fed:ApplicationServiceType** .

    c. Copiez la valeur du nœud **X509Certificate** .

    d. Enregistrez la valeur de **X509Certificate** copiée, dans un fichier local sur votre ordinateur.

7. Dans la section **Public** , cliquez sur **Add** .

    ![Ajouter](./media/topdesk-secure-tutorial/ic790607.png "Ajouter")

8. Dans la boîte de dialogue **SAML configuration assistant** , procédez comme suit :

    ![Assistant de configuration SAML](./media/topdesk-secure-tutorial/ic790608.png "SAML configuration assistant")

    a. Pour charger votre fichier de métadonnées téléchargé à partir du portail Azure, dans **Métadonnées de fédération** , cliquez sur **Parcourir** .

    b. Pour charger votre fichier de certificat, sous **Certificate (RSA)** , cliquez sur **Browse** .

    c. Pour **Private key(RSA, PKCS8, DER)** , vous pouvez charger votre propre clé privée ou contacter [l’équipe de support technique TOPdesk](https://www.topdesk.com/us/support) pour obtenir la clé privée.

    d. Pour charger le fichier de logo que vous avez obtenu de l’équipe de support TOPdesk, sous **Logo icon** , cliquez sur **Browse** .

    e. Dans la zone de texte **User name attribute** (Attribut de nom d’utilisateur), entrez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`.

    f. Dans la zone de texte **Display name** , indiquez le nom de votre configuration.

    g. Cliquez sur **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon\@domainedevotreentreprise.extension** .  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à TOPdesk - Secure.

1. Sur le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **TOPdesk - Secure** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, tapez et sélectionnez **TOPdesk - Secure** .

    ![Lien TOPdesk - Secure dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-topdesk---secure-test-user"></a>Créer un utilisateur de test TOPdesk - Secure

Pour se connecter à TOPdesk - Secure, les utilisateurs d’Azure AD doivent être approvisionnés dans TOPdesk - Secure.  
Dans le cas de TOPdesk - Secure, l’approvisionnement est une tâche manuelle.

### <a name="to-configure-user-provisioning-perform-the-following-steps"></a>Pour configurer l'approvisionnement des utilisateurs, procédez comme suit :

1. Connectez-vous à votre site d’entreprise **TOPdesk - Secure** en tant qu’administrateur.

2. Dans le menu en haut, cliquez sur **TOPdesk \> New \> Support Files \> Operator** .

    ![Opérateur](./media/topdesk-secure-tutorial/ic790610.png "Opérateur")

3. Dans la boîte de dialogue **New Operator** , procédez comme suit :

    ![Nouvel opérateur](./media/topdesk-secure-tutorial/ic790611.png "New Operator")

    a. Cliquez sur l’onglet **General** (Général).

    b. Dans la zone de texte **Surname** (Nom), tapez le nom de l’utilisateur, par exemple **Simon** .

    c. Sélectionnez un **Site** pour le compte dans la section **Emplacement** .

    d. Dans la zone de texte **Login Name** de la section **TOPdesk Login** , indiquez le nom de connexion de votre utilisateur.

    e. Cliquez sur **Enregistrer** .

> [!NOTE]
> Vous pouvez utiliser tout autre outil ou API de création de compte d’utilisateur TOPdesk - Secure fourni par TOPdesk - Secure pour provisionner des comptes d’utilisateur Azure AD.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette TOPdesk - Secure dans le panneau d’accès doit vous connecter automatiquement à l’application TOPdesk - Secure pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)