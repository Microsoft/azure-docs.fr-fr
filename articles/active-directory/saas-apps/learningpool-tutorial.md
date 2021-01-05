---
title: 'Didacticiel : Intégration d’Azure Active Directory avec Learningpool Act | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Learningpool Act.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/25/2019
ms.author: jeedes
ms.openlocfilehash: eb8d0c229ed5f41dcb5999243a57ee19c2b67f74
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92458725"
---
# <a name="tutorial-azure-active-directory-integration-with-learningpool-act"></a>Didacticiel : Intégration d’Azure Active Directory à Learningpool Act

Dans ce didacticiel, vous allez apprendre à intégrer Learningpool Act à Azure Active Directory (Azure AD).
L’intégration de Learningpool Act dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Learningpool Act.
* Vous pouvez permettre à vos utilisateurs d’être automatiquement connectés à Learningpool Act (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Learningpool Act, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Learningpool Act pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Learningpool Act prend en charge l’authentification unique initiée par le **fournisseur de services**

## <a name="adding-learningpool-act-from-the-gallery"></a>Ajout de Learningpool Act à partir de la galerie

Pour configurer l’intégration de Learningpool Act à Azure AD, vous devez ajouter Learningpool Act à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Learningpool Act à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Learningpool Act** , sélectionnez **Learningpool Act** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Learningpool Act dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD auprès de Learningpool Act, à l’aide d’un utilisateur de test nommé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Learningpool Act associé doit être établie.

Pour configurer et tester l’authentification unique avec Azure AD avec Learningpool Act, vous devez compléter les blocs de construction suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique de Learningpool Act](#configure-learningpool-act-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer l’utilisateur de test Learningpool Act](#create-learningpool-act-test-user)** pour avoir dans Learningpool Act un équivalent de Britta Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD auprès de Learningpool Act, effectuez les étapes suivantes :

1. Dans la page d’intégration de l’application [Learningpool Act](https://portal.azure.com/) sur le **portail Azure** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique relatives au domaine et aux URL Learningpool Act](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion** , tapez l’URL : `https://parliament.preview.Learningpool.com/auth/shibboleth/index.php`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant :
    
    ```http
    https://<subdomain>.Learningpool.com/shibboleth
    https://<subdomain>.preview.Learningpool.com/shibboleth
    ```

    > [!NOTE]
    > La valeur de l'identificateur n'est pas réelle. Mettez à jour cette valeur avec l’identificateur réel. Pour obtenir cette valeur, contactez l’[équipe du support technique de Learningpool Act](https://www.learningpool.com/support). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Votre application Learningpool Act attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut. Cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue Attributs d’utilisateur.

    ![Capture d’écran montrant User Attributes avec l’icône Edit sélectionnée.](common/edit-attribute.png)

6. Dans la section **Revendications des utilisateurs** de la boîte de dialogue **Attributs utilisateur** , modifiez les revendications en utilisant l’icône **Modifier** ou ajoutez des revendications en utilisant l’option **Ajouter une nouvelle revendication** pour configurer l’attribut de jeton SAML comme sur l’image ci-dessus et procédez comme suit : 

    | Name | Attribut source|
    | ------------------- | -------------------- |
    | urn:oid:1.2.840.113556.1.4.221 | user.userprincipalname |
    | urn:oid:2.5.4.42 | user.givenname |
    | urn:oid:0.9.2342.19200300.100.1.3 | user.mail |
    | urn:oid:2.5.4.4 | user.surname |
    | | |

    a. Cliquez sur le bouton **Ajouter une nouvelle revendication** pour ouvrir la boîte de dialogue **Gérer les revendications des utilisateurs** .

    ![Capture d’écran montrant User claims avec l’option permettant d’ajouter une nouvelle revendication.](common/new-save-attribute.png)

    ![Capture d’écran montrant la boîte de dialogue Manage user claims où vous pouvez entrer les valeurs décrites.](common/new-attribute-details.png)

    b. Dans la zone de texte **Attribut** , indiquez le nom d’attribut pour cette ligne.

    c. Laissez le champ **Espace de noms** vide.

    d. Sélectionnez Source comme **Attribut** .

    e. Dans la liste **Attribut de la source** , tapez la valeur d’attribut indiquée pour cette ligne.

    f. Cliquez sur **OK**

    g. Cliquez sur **Enregistrer** .

7. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

8. Dans la section **Configurer Learningpool Act** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-learningpool-act-single-sign-on"></a>Configurer l’authentification unique de Learningpool Act

Pour configurer l’authentification unique côté **Learningpool Act** , vous devez envoyer le **XML des métadonnées de fédération** téléchargé et les URL appropriées, copiées à partir du portail Azure, à l’ [équipe du support technique de Learningpool Act](https://www.learningpool.com/support). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Learningpool Act.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Learningpool Act** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Learningpool Act** .

    ![Lien Learningpool Act dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-learningpool-act-test-user"></a>Créer l’utilisateur de test Learningpool Act

Pour permettre aux utilisateurs Azure AD de se connecter à Learningpool Act, vous devez les configurer dans Learningpool Act.

Aucun élément d’action ne vous permet de configurer l’approvisionnement des utilisateurs dans Learningpool Act.  
Les utilisateurs doivent être créés par votre [équipe de support technique Learningpool Act](https://www.Learningpool.com/support).

> [!NOTE]
> Vous pouvez utiliser tout autre outil ou API de création de compte d’utilisateur Learningpool Act fourni par Learningpool Act pour provisionner des comptes d’utilisateur Azure AD.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Learningpool Act dans le volet d’accès doit vous connecter automatiquement à l’application Learningpool Act pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)