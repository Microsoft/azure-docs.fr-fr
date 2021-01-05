---
title: 'Didacticiel : intégration d’Azure Active Directory à SkyDesk Email | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SkyDesk Email.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/07/2019
ms.author: jeedes
ms.openlocfilehash: 30e94c9737241ff49c29898adcc5e50c6b73a9b1
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92516022"
---
# <a name="tutorial-azure-active-directory-integration-with-skydesk-email"></a>Didacticiel : Intégration d’Azure Active Directory à SkyDesk Email

Dans ce didacticiel, vous allez apprendre à intégrer SkyDesk Email à Azure Active Directory (Azure AD).
L’intégration de SkyDesk Email à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à SkyDesk Email.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à SkyDesk Email (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec SkyDesk Email, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement SkyDesk Email pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* SkyDesk Email prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-skydesk-email-from-the-gallery"></a>Ajout de SkyDesk Email à partir de la galerie

Pour configurer l’intégration de SkyDesk Email à Azure AD, vous devez ajouter SkyDesk Email disponible dans la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter SkyDesk Email à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **SkyDesk Email** , sélectionnez **SkyDesk Email** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![SkyDesk Email dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD auprès de SkyDesk Email, avec un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur SkyDesk Email associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec SkyDesk Email, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique SkyDesk Email](#configure-skydesk-email-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test SkyDesk Email](#create-skydesk-email-test-user)** pour avoir un équivalent de Britta Simon dans SkyDesk Email lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Skydesk Email, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **SkyDesk Email** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL SkyDesk Email](common/sp-signonurl.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://mail.skydesk.jp/portal/<companyname>`

    > [!NOTE]
    > Cette valeur n’est pas la valeur réelle. Mettez à jour la valeur avec l’URL de connexion réelle. Contactez l’[équipe de support client SkyDesk Email](https://www.skydesk.jp/apps/support/) pour obtenir la valeur. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer SkyDesk Email** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-skydesk-email-single-sign-on"></a>Configurer l’authentification unique SkyDesk Email

1. Dans un autre navigateur web, connectez-vous à votre compte SkyDesk Email en tant qu’administrateur.

1. Dans le menu situé en haut, cliquez sur **Setup** , puis sélectionnez **Org** .

    ![Capture d’écran affichant l’élément Org sélectionné dans le menu Setup.](./media/skydeskemail-tutorial/tutorial_skydeskemail_51.png)
  
1. Cliquez sur **Domains** dans le volet de gauche.

    ![Capture d’écran affichant l’élément Domains sélectionné dans Control Panel.](./media/skydeskemail-tutorial/tutorial_skydeskemail_53.png)

1. Cliquez sur **Add Domain** .

    ![Capture d’écran montrant l’option d’ajout d’un domaine.](./media/skydeskemail-tutorial/tutorial_skydeskemail_54.png)

1. Entrez votre nom de domaine et vérifiez le domaine.

    ![Capture d’écran affichant l’onglet Add Domain, dans lequel vous pouvez entrer votre domaine.](./media/skydeskemail-tutorial/tutorial_skydeskemail_55.png)

1. Cliquez sur **Authentification SAML** dans le panneau de gauche.

    ![Capture d’écran affichant l’élément SAML Authentication sélectionné dans Control Panel.](./media/skydeskemail-tutorial/tutorial_skydeskemail_52.png)

1. Dans la page de boîte de dialogue **SAML Authentication** , procédez comme suit :

    ![Capture d’écran montrant la boîte de dialogue SAML Authentication Details, dans laquelle vous pouvez entrer les valeurs décrites.](./media/skydeskemail-tutorial/tutorial_skydeskemail_56.png)

    > [!NOTE]
    > Pour utiliser l’authentification SAML, un **domaine vérifié** ou **l’URL du portail** doit être configuré. Vous pouvez définir l’URL du portail avec un nom unique.

    ![Capture d’écran affichant la section Portal URL, dans laquelle vous entrez le nom.](./media/skydeskemail-tutorial/tutorial_skydeskemail_57.png)

    a. Dans la zone de texte **Login URL** (URL de connexion), collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.

    b. Dans la zone de texte **Logout URL** (URL de déconnexion), collez la valeur de l’ **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    c. **Modifier l’URL de mot de passe** est facultative, vous pouvez donc laisser cette valeur vide.

    d. Cliquez sur **Get Key From File (Obtenir la clé à partir d’un fichier)** pour sélectionner votre certificat téléchargé à partir du portail Azure, puis sur **Ouvrir** pour charger le certificat.

    e. Comme **Algorithme** , sélectionnez  **RSA** .

    f. Cliquez sur **OK** pour enregistrer les modifications.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon@yourcompanydomain.extension**  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à SkyDesk Email.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis sélectionnez **SkyDesk Email** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **SkyDesk Email** .

    ![Lien SkyDesk Email dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-skydesk-email-test-user"></a>Créer un utilisateur de test SkyDesk Email

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans SkyDesk Email.

Cliquez sur **User Access** (Accès utilisateur) dans le panneau de gauche de SkyDesk Email, puis entrez votre nom d’utilisateur.

![Capture d’écran montrant l’élément User Access sélectionné dans Control Panel.](./media/skydeskemail-tutorial/tutorial_skydeskemail_58.png)

> [!NOTE]
> Si vous avez besoin de créer des utilisateurs en bloc, contactez l’[équipe de support client SkyDesk Email](https://www.skydesk.jp/apps/support/).

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette SkyDesk Email dans le panneau d’accès doit vous connecter automatiquement à l’application SkyDesk Email pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)