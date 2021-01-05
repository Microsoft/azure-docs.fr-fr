---
title: 'Didacticiel : Intégration d’Azure Active Directory à Hosted Graphite | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Hosted Graphite.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/15/2019
ms.author: jeedes
ms.openlocfilehash: 786c6309f685af6b8e42108c490ce86c7844bc81
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92443135"
---
# <a name="tutorial-azure-active-directory-integration-with-hosted-graphite"></a>Didacticiel : Intégration d’Azure Active Directory à Hosted Graphite

Dans ce didacticiel, vous allez apprendre à intégrer Hosted Graphite à Azure Active Directory (Azure AD).
L’intégration de Hosted Graphite à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Hosted Graphite.
* Vous pouvez autoriser les utilisateurs à se connecter automatiquement à Hosted Graphite (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Hosted Graphite, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement Hosted Graphite pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Hosted Graphite prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**
* Hosted Graphite prend en charge l’attribution d’utilisateurs **juste-à-temps**

## <a name="adding-hosted-graphite-from-the-gallery"></a>Ajout de Hosted Graphite à partir de la galerie

Pour configurer l’intégration de Hosted Graphite à Azure AD, vous devez ajouter Hosted Graphite à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Hosted Graphite à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Hosted Graphite** , sélectionnez **Hosted Graphite** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Hosted Graphite dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Hosted Graphite pour un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur Hosted Graphite associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Hosted Graphite, vous avez besoin de suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Hosted Graphite](#configure-hosted-graphite-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Hosted Graphite](#create-hosted-graphite-test-user)** pour avoir un équivalent de Britta Simon dans Hosted Graphite lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Hosted Graphite, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Hosted Graphite** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. À la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode initié par **IDP** , suivez les étapes ci-dessous :

    ![Capture d’écran montrant Basic SAML Configuration, où vous pouvez entrer Identifier, et Reply URL, et sélectionner Save.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://www.hostedgraphite.com/metadata/<user id>`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://www.hostedgraphite.com/complete/saml/<user id>`

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    ![Capture d’écran montrant Set additional URLs, où vous pouvez entrer une URL de connexion.](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://www.hostedgraphite.com/login/saml/<user id>/`

    > [!NOTE]
    > Notez qu’il ne s’agit pas des valeurs réelles. Vous devez mettre à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, vous pouvez accéder à Accéder->Configuration de SAML ou contactez [l’équipe de support d’Hosted Graphite](mailto:help@hostedgraphite.com).

6. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

7. Dans la section **Configurer Hosted Graphite** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-hosted-graphite-single-sign-on"></a>Configurer l’authentification unique Hosted Graphite

1. Connectez-vous à votre client Hosted Graphite en tant qu’administrateur.

2. Accédez à la **page de configuration de SAML** dans la barre latérale ( **Accéder -> Configuration SAML** ).

    ![Capture d’écran montrant le menu Access avec SAML Setup sélectionné.](./media/hostedgraphite-tutorial/tutorial_hostedgraphite_000.png)

3. Confirmez que ces URL correspondent à votre configuration dans la section **Configuration SAML de base** du portail Azure.

    ![Capture d’écran montrant Basic SAML Configuration.](./media/hostedgraphite-tutorial/tutorial_hostedgraphite_001.png)

4. Dans les zones de texte **ID d’entité ou d’émetteur** et **URL de connexion à authentification unique** , collez les valeurs **Identificateur Azure AD** et **URL de connexion**  que vous avez copiées à partir du portail Azure.

    ![Capture d’écran montrant des entrées pour Identity Provider.](./media/hostedgraphite-tutorial/tutorial_hostedgraphite_002.png)

5. Sélectionnez **Lecture seule** comme **Rôle d’utilisateur par défaut** .

    ![Capture d’écran montrant Default User Role, qui est Read-only.](./media/hostedgraphite-tutorial/tutorial_hostedgraphite_004.png)

6. Ouvrez dans le Bloc-notes votre certificat codé en base 64 téléchargé à partir du portail Azure, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **Certificat X.509** .

    ![Capture d’écran montrant un certificat x.509.](./media/hostedgraphite-tutorial/tutorial_hostedgraphite_005.png)

7. Cliquez sur le bouton **Enregistrer** .

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Hosted Graphite.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Hosted Graphite** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Hosted Graphite** .

    ![Lien Hosted Graphite dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-hosted-graphite-test-user"></a>Créer un utilisateur de test Hosted Graphite

Dans cette section, un utilisateur appelé Britta Simon est créé dans Hosted Graphite. Hosted Graphite prend en charge l’attribution d’utilisateurs juste-à-temps, qui est une option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Hosted Graphite, il en est créé un après l’authentification.

> [!NOTE]
> Si vous devez créer un utilisateur manuellement, contactez l’équipe de support d’Hosted Graphite via <mailto:help@hostedgraphite.com>.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Hosted Graphite dans le panneau d’accès doit vous connecter automatiquement à l’application Hosted Graphite pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)