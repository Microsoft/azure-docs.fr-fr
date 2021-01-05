---
title: 'Tutoriel : Intégration d’Azure Active Directory à Meta Networks Connector| Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Meta Networks Connector.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/21/2019
ms.author: jeedes
ms.openlocfilehash: b14a75dba2860c9dee58e40673d3299fdde277e7
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92516863"
---
# <a name="tutorial-azure-active-directory-integration-with-meta-networks-connector"></a>Tutoriel : Intégration d’Azure Active Directory à Meta Networks Connector

Dans ce tutoriel, vous apprenez à intégrer Meta Networks Connector à Azure Active Directory (Azure AD).
L’intégration de Meta Networks Connector à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Meta Networks Connector.
* Vous pouvez permettre à vos utilisateurs d’être automatiquement connectés à Meta Networks Connector (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Meta Networks Connector, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement à Meta Networks Connector pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Meta Networks Connector prend en charge l’authentification unique initiée par le **fournisseur de services** et le **fournisseur d’identité**
 
* Meta Networks Connector prend en charge l’attribution d’utilisateurs **Juste-à-temps**

## <a name="adding-meta-networks-connector-from-the-gallery"></a>Ajout de Meta Networks Connector à partir de la galerie

Pour configurer l’intégration de Meta Networks Connector à Azure AD, vous devez ajouter Meta Networks Connector à votre liste d’applications SaaS gérées, à partir de la galerie.

**Pour ajouter Meta Networks Connector à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Meta Networks Connector** , sélectionnez **Meta Networks Connector** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Meta Networks Connector dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous configurez et testez l’authentification unique Azure AD auprès de Meta Networks Connector, au moyen d’un utilisateur de test nommé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation doit être établie entre un utilisateur Azure AD et l’utilisateur Meta Networks Connector qui lui est associé.

Pour configurer et tester l’authentification unique Azure AD avec Meta Networks Connector, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique de Meta Networks Connector](#configure-meta-networks-connector-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer l’utilisateur de test Meta Networks Connector](#create-meta-networks-connector-test-user)** pour avoir dans Meta Networks Connector un équivalent de Britta Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD auprès de Meta Networks Connector, effectuez les étapes suivantes :

1. Dans la page d’intégration de l’application [Meta Networks Connector](https://portal.azure.com/) sur le **portail Azure** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. À la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode initié par **IDP** , suivez les étapes ci-dessous :

    ![Capture d’écran montrant Configuration SAML de base, où vous pouvez entrer l’identificateur, l’URL de réponse, et sélectionner Enregistrer.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://login.nsof.io/v1/<ORGANIZATION-SHORT-NAME>/saml/metadata`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://login.nsof.io/v1/<ORGANIZATION-SHORT-NAME>/sso/saml`

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    ![Capture d’écran montrant Définir des URL supplémentaires, où vous pouvez entrer une URL de connexion.](common/both-advanced-urls.png)

    a. Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<ORGANIZATION-SHORT-NAME>.metanetworks.com/login`.

    b. Dans la zone de texte **État de relais** , entrez une URL en utilisant le modèle suivant : `https://<ORGANIZATION-SHORT-NAME>.metanetworks.com/#/`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. La procédure est expliquée plus loin dans le tutoriel.

6. L’application Meta Networks Connector attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut. Cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Attributs d’utilisateur** .

    ![Capture d’écran montrant User Attributes avec l’icône Edit sélectionnée.](common/edit-attribute.png)
    
7. En plus de ce qui précède, l’application Meta Networks Connector s’attend à ce que quelques attributs supplémentaires soient repassés dans la réponse SAML. Dans la section **Revendications des utilisateurs** de la boîte de dialogue **Attributs utilisateur** , effectuez les étapes suivantes pour ajouter le jeton SAML comme indiqué dans le tableau ci-dessous :
    
    | Name | Attribut source | Espace de noms|
    | ---------------| --------------- | -------- |
    | firstname | user.givenname | |
    | lastname | user.surname | |
    | emailaddress| user.mail| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims` |
    | name | user.userprincipalname| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims` |
    | phone | user.telephonenumber | |

    a. Cliquez sur le bouton **Ajouter une nouvelle revendication** pour ouvrir la boîte de dialogue **Gérer les revendications des utilisateurs** .

    ![Capture d’écran montrant User claims avec l’option permettant d’ajouter une nouvelle revendication.](common/new-save-attribute.png)

    ![Capture d’écran montrant la boîte de dialogue Manage user claims où vous pouvez entrer les valeurs décrites.](common/new-attribute-details.png)

    b. Dans la zone de texte **Attribut** , indiquez le nom d’attribut pour cette ligne.

    c. Laissez le champ **Espace de noms** vide.

    d. Sélectionnez Source comme **Attribut** .

    e. Dans la liste **Attribut de la source** , tapez la valeur d’attribut indiquée pour cette ligne.

    f. Cliquez sur **OK**

    g. Cliquez sur **Enregistrer** .

8. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

9. Dans la section **Configurer Meta Networks Connector** , copiez l’URL ou les URL appropriées, en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-meta-networks-connector-single-sign-on"></a>Configurer l’authentification unique de Meta Networks Connector

1. Ouvrez un nouvel onglet dans votre navigateur et connectez-vous à votre compte d’administrateur Meta Networks Connector.
    
    > [!NOTE]
    > Meta Networks Connector est un système sécurisé. Par conséquent, avant d’accéder à son portail, vous devez faire approuver votre adresse IP publique pour qu’elle figure dans la liste verte de ce système. Pour obtenir votre adresse IP publique, suivez le lien spécifié [ici](https://whatismyipaddress.com/). Envoyez votre adresse IP à l’[équipe du support technique de Meta Networks Connector](mailto:support@metanetworks.com) pour qu’elle soit ajoutée à la liste verte.
    
2. Accédez à **Administrator** (Administrateur) et sélectionnez **Settings** (Paramètres).
    
    ![Capture d’écran montrant l’élément Settings sélectionné dans le menu Administration.](./media/metanetworksconnector-tutorial/configure3.png)
    
3. Assurez-vous que les options **Log Internet Traffic** (Journaliser le trafic Internet) et **Force VPN MFA** (Forcer l’authentification multifacteur VPN) sont désactivées.
    
    ![Capture d’écran montrant la désactivation de ces paramètres.](./media/metanetworksconnector-tutorial/configure1.png)
    
4. Accédez à **Administrator** (Administrateur) et sélectionnez **SAML** (SAML).
    
    ![Capture d’écran montrant l’élément SAML sélectionné dans le menu Administration.](./media/metanetworksconnector-tutorial/configure4.png)
    
5. Effectuez les étapes suivantes dans la page **DETAILS** (DÉTAILS) :
    
    ![Capture d’écran montrant la page DETAILS dans laquelle vous pouvez indiquer les valeurs décrites.](./media/metanetworksconnector-tutorial/configure2.png)
    
    a. Copiez la valeur de **SSO URL** (URL SSO) et collez-la dans la zone de texte **URL de connexion** de la section **Domaine et URL Meta Networks Connector** .
    
    b. Copiez la valeur de **Recipient URL** (URL du destinataire) et collez-la dans la zone de texte **URL de réponse** de la section **Domaine et URL Meta Networks Connector** .
    
    c. Copiez la valeur de **Audience URI (SP Entity ID)** (URI d’audience - ID d’entité SP) et collez-la dans la zone de texte **Identificateur (ID d’entité)** de la section **Domaine et URL Meta Networks Connector** .
    
    d. Activer SAML
    
6. Sous l’onglet **GENERAL** , effectuez les étapes suivantes :

    ![Capture d’écran montrant la page GENERAL dans laquelle vous pouvez indiquer les valeurs décrites.](./media/metanetworksconnector-tutorial/configure5.png)

    a. Dans la zone **Identity Provider Single Sign-On URL** (URL d’authentification unique du fournisseur d’identité), collez l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    b. Dans la zone **Identity Provider Issuer** (Émetteur du fournisseur d’identité), collez l’ **identificateur Azure AD** que vous avez copié à partir du portail Azure.

    c. Dans le bloc-notes, ouvrez le certificat téléchargé à partir du portail Azure et collez-le dans la zone de texte **X.509 Certificate** (Certificat X.509).

    d. Activez l’option **Just-in-Time Provisioning** (Provisionnement juste-à-temps).

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon\@domainedevotresociété.extension** .  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez Britta Simon à utiliser l’authentification unique Azure en accordant l’accès à Meta Networks Connector.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Meta Networks Connector** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Meta Networks Connector** .

    ![Lien Meta Networks Connector dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-meta-networks-connector-test-user"></a>Créer l’utilisateur de test Meta Networks Connector

Dans cette section, un utilisateur nommé Britta Simon est créé dans Meta Networks Connector. Meta Networks Connector prend en charge le provisionnement juste-à-temps, une option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. Si l’utilisateur souhaité n’existe pas déjà dans Meta Networks Connector, il est créé quand vous tentez d’accéder à Meta Networks Connector.

>[!Note]
>Si vous devez créer un utilisateur manuellement, contactez [l’équipe du support technique du client Meta Networks Connector](mailto:support@metanetworks.com).

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Meta Networks Connector dans le volet d’accès doit vous connecter automatiquement à l’application Meta Networks Connector pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)