---
title: 'Didacticiel : Intégration d’Azure Active Directory à SpaceIQ | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SpaceIQ.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: jeedes
ms.openlocfilehash: 74c3c4d1a9192256c36d41db7a4f9406b6c966f3
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92514722"
---
# <a name="tutorial-azure-active-directory-integration-with-spaceiq"></a>Didacticiel : Intégration d’Azure AD avec SpaceIQ

Dans ce didacticiel, vous allez apprendre à intégrer SpaceIQ à Azure Active Directory (Azure AD).
L’intégration de SpaceIQ dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à SpaceIQ.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à SpaceIQ (par le biais de l’authentification unique) avec leur compte Azure Active Directory.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à SpaceIQ, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Abonnement à SpaceIQ pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* SpaceIQ prend en charge l’authentification unique lancée par le **point de distribution d’émission**

## <a name="adding-spaceiq-from-the-gallery"></a>Ajout de SpaceIQ à partir de la galerie

Pour configurer l’intégration de SpaceIQ avec Azure AD, vous devez ajouter SpaceIQ, disponible dans la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter SpaceIQ à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **SpaceIQ** , sélectionnez **SpaceIQ** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![SpaceIQ dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure Active Directory avec SpaceIQ avec un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure Active Directory et l’utilisateur SpaceIQ associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec SpaceIQ, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique SpaceIQ](#configure-spaceiq-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test SpaceIQ](#create-spaceiq-test-user)** pour avoir un équivalent de Britta Simon dans SpaceIQ lié à la représentation Azure Active Directory associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure Active Directory avec SpaceIQ, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **SpaceIQ** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Sur la page **Configurer l’authentification unique avec SAML** , effectuez les étapes suivantes :

    ![Informations d’authentification unique Domaine et URL SpaceIQ](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une l’URL : `https://api.spaceiq.com`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://api.spaceiq.com/saml/<instanceid>/callback`

    > [!NOTE]
    > Mettez à jour ces valeur avec l’URL de réponse et l’identificateur réels. La procédure est expliquée plus loin dans le didacticiel.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer SpaceIQ** , copiez la ou les URL appropriées en fonction de vos exigences.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-spaceiq-single-sign-on"></a>Configurer l’authentification unique SpaceIQ

1. Ouvrez une nouvelle fenêtre de navigateur, puis connectez-vous au site de votre entreprise SpaceIQ en tant qu’administrateur.

1. Une fois que vous êtes connecté, cliquez sur le signe de puzzle en haut à droite, puis cliquez sur **Intégration**

    ![Account Settings](./media/spaceiq-tutorial/setting1.png) 

1. Sous **All PROVISIONING & SSO** , cliquez sur la vignette **Azure** pour ajouter une instance d’Azure comme fournisseur d’identité.

    ![Icône SAML](./media/spaceiq-tutorial/setting2.png)

1. Dans la boîte de dialogue **SSO** , effectuez les étapes suivantes :

    ![Paramètres d’authentification SAML](./media/spaceiq-tutorial/setting3.png)

    a. Dans la zone **URL de l’émetteur SAML** , collez la valeur **Identificateur Azure Active Directory** copiée à partir de la fenêtre Configuration de l’application Azure Active Directory.

    b. Copiez la valeur **URL du point de terminaison de rappel SAML (en lecture seule)** et collez-la dans la zone **URL de réponse** dans la section **Configuration SML de base** du portail Azure.

    c. Copiez la valeur **URI de l’audience SAML (en lecture seule)** et collez-la dans la zone **Identificateur** dans la section **Configuration SML de base** du portail Azure.

    d. Ouvrez le fichier de certificat que vous avez téléchargé dans le Bloc-notes, copiez son contenu, puis collez-le dans la zone **Certificat X.509** .

    e. Cliquez sur **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez `brittasimon@yourcompanydomain.extension`  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à SpaceIQ.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **SpaceIQ** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **SpaceIQ** .

    ![Lien SpaceIQ dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-spaceiq-test-user"></a>Créer un utilisateur de test SpaceIQ

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans SpaceIQ. Collaborez avec l’[équipe de support technique de SpaceIQ](mailto:eng@spaceiq.com) pour ajouter les utilisateurs dans la plateforme SpaceIQ. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la vignette SpaceIQ dans le volet d’accès, vous devez être connecté automatiquement à l’application SpaceIQ pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)