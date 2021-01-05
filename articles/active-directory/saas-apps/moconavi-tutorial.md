---
title: 'Tutoriel : Intégration d’Azure Active Directory à moconavi | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et moconavi.
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
ms.openlocfilehash: f4ddb41528357d2f1b5c9e671ec2dd1308f9bf11
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92516826"
---
# <a name="tutorial-azure-active-directory-integration-with-moconavi"></a>Tutoriel : Intégration d’Azure Active Directory à moconavi

Dans ce didacticiel, vous allez apprendre à intégrer moconavi à Azure Active Directory (Azure AD).
L’intégration de moconavi à Azure AD offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à moconavi.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à moconavi (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à moconavi, vous aurez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement moconavi pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* monocavi prend en charge l’authentification unique initiée par le **fournisseur de services**

## <a name="adding-moconavi-from-the-gallery"></a>Ajouter moconavi à partir de la galerie

Pour configurer l’intégration de moconavi à Azure AD, vous devez ajouter moconavi à votre liste d’applications SaaS gérées à partir de la galerie.

**Pour ajouter moconavi à partir de la galerie, suivez les étapes ci-dessous :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **moconavi** , sélectionnez **moconavi** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![moconavi dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec moconavi pour un utilisateur de test nommé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur monocavi associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec moconavi, suivez les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique moconavi](#configure-moconavi-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test moconavi](#create-moconavi-test-user)** pour avoir un équivalent de Britta Simon dans moconavi, lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec moconavi, suivez les étapes ci-dessous :

1. Sur la page d’intégration de l’application [moconavi](https://portal.azure.com/) du **portail Azure** , cliquez sur **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL moconavi](common/sp-identifier-reply.png)

    a. Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<yourserverurl>/moconavi-saml2/saml/login`.

    b. Dans la zone de texte **Identificateur** , tapez une URL en utilisant le format suivant : `https://<yourserverurl>/moconavi-saml2`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<yourserverurl>/moconavi-saml2/saml/SSO`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez [l’équipe de support client de moconavi](mailto:support@recomot.co.jp). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configurer moconavi** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-moconavi-single-sign-on"></a>Configurer l'authentification unique moconavi

Pour configurer l’authentification unique côté **moconavi** , vous devez envoyer le **XML des métadonnées de fédération** téléchargé et les URL copiées dans le portail Azure à l’ [équipe du support technique moconavi](mailto:support@recomot.co.jp). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à moconavi.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **moconavi** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **moconavi** .

    ![Lien moconavi dans la liste Applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-moconavi-test-user"></a>Créer un utilisateur de test moconavi

Dans cette section, vous allez créer un utilisateur nommé Britta Simon dans moconavi. Rapprochez-vous de [l’équipe de support de moconavi](mailto:support@recomot.co.jp) pour ajouter les utilisateurs sur la plateforme moconavi. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

1. Installez moconavi à partir de Microsoft Store.

2. Lancez moconavi.

3. Cliquez sur le bouton **Paramètre de connexion** .

    ![Capture d’écran montrant moconavi avec le bouton Connection setting.](./media/moconavi-tutorial/testing1.png)

4. Entrez `https://mcs-admin.moconavi.biz/gateway` dans la zone de texte **Se connecter à l’URL** , puis cliquez sur le bouton **Terminé** .

    ![Capture d’écran montrant la zone Connect to URL et le bouton Done.](./media/moconavi-tutorial/testing2.png)

5. Dans la capture d’écran suivante, suivez les étapes ci-dessous :

    ![Capture d’écran montrant la page moconavi dans laquelle vous pouvez indiquer les valeurs décrites.](./media/moconavi-tutorial/testing3.png)

    a. Entrez la **clé d’authentification**`azureAD` dans la zone de texte **Entrer la clé d’authentification** .

    b. Entrez **l’ID d’utilisateur**`your ad account` dans la zone de texte **Entrer l’ID d’utilisateur** .

    c. Cliquez sur **CONNEXION** .

6. Entrez votre mot de passe Azure AD dans la zone de texte **Mot de passe** , puis cliquez sur le bouton **Connexion** .

    ![Capture d’écran indiquant l’emplacement dans lequel entrer votre mot de passe Azure AD.](./media/moconavi-tutorial/testing4.png)

7. L’authentification Azure AD est réussie lorsque le menu s’affiche.

    ![Capture d’écran montrant l’icône Telephone dans moconavi.](./media/moconavi-tutorial/testing5.png)

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)