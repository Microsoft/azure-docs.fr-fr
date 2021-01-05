---
title: 'Tutoriel : Intégration d’Azure Active Directory à TOPdesk - Public | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et TOPdesk - Public.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/02/2019
ms.author: jeedes
ms.openlocfilehash: 3688dc2db67c0c1343a74be4652cb52466ef762e
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92520436"
---
# <a name="tutorial-azure-active-directory-integration-with-topdesk---public"></a>Tutoriel : Intégration d’Azure Active Directory à TOPdesk - Public

Dans ce didacticiel, vous allez apprendre à intégrer TOPdesk - Public à Azure Active Directory (Azure AD).
L’intégration de TOPdesk - Public dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à TOPdesk - Public.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à TOPdesk - Public (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à TOPdesk - Public, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement TOPdesk - Public pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* TOPdesk - Public prend en charge l’authentification unique initiée par le **fournisseur de services**

## <a name="adding-topdesk---public-from-the-gallery"></a>Ajout de TOPdesk - Public à partir de la galerie

Pour configurer l’intégration de TOPdesk - Public à Azure AD, vous devez ajouter TOPdesk - Public à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter TOPdesk - Public à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **TOPdesk - Public** , sélectionnez **TOPdesk - Public** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![TOPdesk - Public dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec TOPdesk - Public avec un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur TOPdesk - Public associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec TOPdesk - Public, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique TOPdesk - Public](#configure-topdesk---public-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test TOPdesk - Public](#create-topdesk---public-test-user)** pour avoir un équivalent de Britta Simon dans TOPdesk - Public lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec TOPdesk - Public, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **TOPdesk - Public** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4.  Dans la section **Configuration SAML de base** , si vous disposez d’un **fichier de métadonnées du fournisseur de services** , suivez les étapes ci-dessous :

    >[!NOTE]
    >Vous obtiendrez le **fichier de métadonnées de fournisseur de services** dans la section **Configurer l’authentification unique TOPdesk - Public** du tutoriel.

    a. Cliquez sur **Charger un fichier de métadonnées** .
    
    ![Charger le fichier de métadonnées](common/upload-metadata.png)

    b. Cliquez sur le **logo du dossier** pour sélectionner le fichier de métadonnées, puis cliquez sur **Charger** .

    ![choisir le fichier de métadonnées](common/browse-upload-metadata.png)

    c. Une fois le fichier de métadonnées chargé, les valeurs **Identificateur** et **URL de réponse** sont automatiquement renseignées dans la section Configuration SAML de base.

    ![Informations d’authentification unique dans Domaine et URL TOPdesk - Public](common/sp-identifier-reply.png)

    d. Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<companyname>.topdesk.net`.

    e. Dans la zone de texte **URL d’identificateur** , renseignez l’URL des métadonnées TOPdesk que vous pouvez récupérer à partir de la configuration TOPdesk. Elle doit utiliser le modèle suivant : `https://<companyname>.topdesk.net/saml-metadata/<identifier>`
    
    f. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<companyname>.topdesk.net/tas/public/login/verify`
    
    > [!NOTE] 
    > Si les valeurs **Identificateur** et **URL de réponse** ne sont pas renseignées automatiquement, vous devez les entrer manuellement. Pour Identificateur, suivez le modèle comme indiqué ci-dessus, et vous obtenez la valeur URL de réponse à partir de la section **Configurer l’authentification unique TOPdesk - Public** plus loin dans le tutoriel. La valeur **URL de connexion** n’est pas réelle. Vous devez donc la mettre à jour avec l’URL de connexion réelle. Contactez l’[équipe de support technique TOPdesk - Public](https://help.topdesk.com/saas/enterprise/user/) pour obtenir la valeur. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configurer TOPdesk - Public** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-topdesk---public-single-sign-on"></a>Configurer l’authentification unique TOPdesk - Public

1. Connectez-vous à votre site d’entreprise **TOPdesk - Public** en tant qu’administrateur.

2. Dans le menu **TOPdesk** , cliquez sur **Settings** .
   
    ![Paramètres](./media/topdesk-public-tutorial/ic790598.png "Paramètres")

3. Cliquez sur **Login Settings** .
   
    ![Paramètres de connexion](./media/topdesk-public-tutorial/ic790599.png "Login Settings")

4. Développez le menu **Login Settings** , puis cliquez sur **General** .
   
    ![Généralités](./media/topdesk-public-tutorial/ic790600.png "Général")

5. Dans la section **Public** de la section de configuration **SAML login** , procédez comme suit :
   
    ![Paramètres techniques](./media/topdesk-public-tutorial/ic790601.png "Technical Settings")
   
    a. Cliquez sur **Download** pour télécharger le fichier de métadonnées public et enregistrez-le en local sur votre ordinateur.
   
    b. Ouvrez le fichier de métadonnées téléchargé et recherchez le nœud **AssertionConsumerService** .

    ![AssertionConsumerService](./media/topdesk-public-tutorial/ic790619.png "AssertionConsumerService")
   
    c. Copiez la valeur **AssertionConsumerService** et collez-la dans la zone de texte **URL de réponse** dans la section **Configuration SAML de base** .      
   
6. Pour créer un fichier de certificat, procédez comme suit :
    
    ![Certificate](./media/topdesk-public-tutorial/ic790606.png "Certificat")
    
    a. Ouvrez le fichier de métadonnées téléchargé à partir du portail Azure.
    
    b. Développez le nœud **RoleDescriptor** dont le **xsi:type** est **fed:ApplicationServiceType** .
    
    c. Copiez la valeur du nœud **X509Certificate** .
    
    d. Enregistrez la valeur de **X509Certificate** copiée, dans un fichier local sur votre ordinateur.

7. Dans la section **Public** , cliquez sur **Add** .
    
    ![Connexion SAML](./media/topdesk-public-tutorial/ic790625.png "SAML login")

8. Dans la boîte de dialogue **SAML configuration assistant** , procédez comme suit :
    
    ![Assistant de configuration SAML](./media/topdesk-public-tutorial/ic790608.png "SAML configuration assistant")
    
    a. Pour charger votre fichier de métadonnées téléchargé à partir du portail Azure, dans **Métadonnées de fédération** , cliquez sur **Parcourir** .

    b. Pour charger votre fichier de certificat, sous **Certificate (RSA)** , cliquez sur **Browse** .

    c. Pour charger le fichier de logo que vous avez obtenu de l’équipe de support TOPdesk, sous **Logo icon** , cliquez sur **Browse** .

    d. Dans la zone de texte **User name attribute** (Attribut de nom d’utilisateur), entrez `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`.

    e. Dans la zone de texte **Display name** , indiquez le nom de votre configuration.

    f. Cliquez sur **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez brittasimon@yourcompanydomain.extension. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à TOPdesk - Public.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **TOPdesk - Public** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **TOPdesk - Public** .

    ![Lien TOPdesk - Public dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-topdesk---public-test-user"></a>Créer un utilisateur de test TOPdesk - Public

Pour pouvoir se connecter à TOPdesk - Public, les utilisateurs d’Azure AD doivent être provisionnés dans TOPdesk - Public. Dans le cas de TOPdesk - Public, l’approvisionnement est une tâche manuelle.

### <a name="to-configure-user-provisioning-perform-the-following-steps"></a>Pour configurer l'approvisionnement des utilisateurs, procédez comme suit :

1. Connectez-vous à votre site d’entreprise **TOPdesk - Public** en tant qu’administrateur.

2. Dans le menu en haut, cliquez sur **TOPdesk \> New \> Support Files \> Person** .
   
    ![Person](./media/topdesk-public-tutorial/ic790628.png "Personne")

3. Dans la boîte de dialogue New User, procédez comme suit :
   
    ![Nouvelle personne](./media/topdesk-public-tutorial/ic790629.png "New Person")
   
    a. Cliquez sur l’onglet General.

    b. Dans la zone de texte **Surname** , tapez le nom de l’utilisateur, par exemple Simon.
 
    c. Sélectionnez un **Site** pour le compte.
 
    d. Cliquez sur **Enregistrer** .

> [!NOTE]
> Vous pouvez utiliser n’importe quel outil ou API de création de compte d’utilisateur, fourni par TOPdesk - Public, pour approvisionner des comptes d’utilisateur Azure AD.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette TOPdesk - Public dans le volet d’accès, vous devez être connecté automatiquement à l’application TOPdesk - Public pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)