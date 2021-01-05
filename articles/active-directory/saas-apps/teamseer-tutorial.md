---
title: 'Tutoriel : Intégration d’Azure Active Directory à TeamSeer | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et TeamSeer.
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
ms.openlocfilehash: 6085ba5091b2b9973354280175aeb01f93ad7e28
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92521167"
---
# <a name="tutorial-azure-active-directory-integration-with-teamseer"></a>Tutoriel : Intégration d’Azure Active Directory à TeamSeer

Dans ce didacticiel, vous allez apprendre à intégrer TeamSeer à Azure Active Directory (Azure AD).
L’intégration de TeamSeer à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à TeamSeer.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à TeamSeer (par le biais de l’authentification unique) avec leurs comptes Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à TeamSeer, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Un abonnement TeamSeer pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* TeamSeer prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-teamseer-from-the-gallery"></a>Ajout de TeamSeer à partir de la galerie

Pour configurer l’intégration de TeamSeer à Azure AD, vous devez ajouter TeamSeer à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter TeamSeer à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **TeamSeer** , sélectionnez **TeamSeer** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![TeamSeer dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec TeamSeer à l’aide d’un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur TeamSeer associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec TeamSeer, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique TeamSeer](#configure-teamseer-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test TeamSeer](#create-teamseer-test-user)** pour avoir un équivalent de Britta Simon dans TeamSeer lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec TeamSeer, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **TeamSeer** , sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL TeamSeer](common/sp-signonurl.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://www.teamseer.com/<companyid>`

    > [!NOTE]
    > Cette valeur n’est pas la valeur réelle. Mettez à jour la valeur avec l’URL de connexion réelle. Pour obtenir cette valeur, contactez [l’équipe de support technique TeamSeer](https://pages.theaccessgroup.com/solutions_business-suite_absence-management_contact.html). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer TeamSeer** , copiez les URL appropriées, en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-teamseer-single-sign-on"></a>Configurer l’authentification unique TeamSeer

1. Dans une autre fenêtre de navigateur web, connectez-vous au site de votre entreprise TeamSeer en tant qu’administrateur.

1. Accédez à **HR Admin**.

    ![Capture d’écran montrant HR Admin sélectionné dans la fenêtre TeamSeer.](./media/teamseer-tutorial/ic789634.png "HR Admin")

1. Cliquez sur **Setup**.

    ![Paramétrage](./media/teamseer-tutorial/ic789635.png "Programme d’installation")

1. Cliquez sur **Set up SAML provider details**.

    ![Capture d’écran montrant Set up SAML provider details sélectionné.](./media/teamseer-tutorial/ic789636.png "SAML Settings")

1. Dans la section des détails sur le fournisseur SAML, procédez comme suit :

    ![Capture d’écran montrant les détails sur le fournisseur SAML où vous pouvez entrer les valeurs décrites.](./media/teamseer-tutorial/ic789637.png "SAML Settings")

    a. Dans la zone de texte **URL** , collez l’ **URL de connexion** que vous avez copiée dans le portail Azure.

    b. Ouvrez votre certificat codé en base 64 dans le Bloc-notes, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **IdP Public Certificate** (Certificat public de fournisseur d’identité).

1. Pour configurer le fournisseur SAML, procédez comme suit :

    ![Capture d’écran montrant la configuration du fournisseur SAML où vous pouvez entrer les valeurs décrites.](./media/teamseer-tutorial/ic789638.png "SAML Settings")

    a. Dans la zone de test **Tester l’adresse de messagerie** , entrez l’adresse de messagerie de l’utilisateur de test.
  
    b. Dans la zone de texte **Émetteur** , entrez l’URL de l’émetteur du fournisseur du service.
  
    c. Cliquez sur **Enregistrer**.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon**.
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon@yourcompanydomain.extension**  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à TeamSeer.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis sélectionnez **TeamSeer**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **TeamSeer**.

    ![Lien TeamSeer dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

### <a name="create-teamseer-test-user"></a>Créer un utilisateur de test TeamSeer

Pour se connecter à TeamSeer, les utilisateurs d’Azure AD doivent être provisionnés dans TeamSeer. Dans le cas de TeamSeer, l’approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous au site d’entreprise **TeamSeer** en tant qu’administrateur.

1. Accédez à **HR Admin \> Users** (Administrateur RH > Utilisateurs), puis cliquez sur **Run the New User wizard** (Exécuter l’Assistant Nouvel utilisateur).

    ![Capture d’écran montrant l’onglet HR Admin où vous pouvez sélectionner un Assistant à exécuter.](./media/teamseer-tutorial/ic789640.png "HR Admin")

1. Dans la section **User Details** , procédez comme suit :

    ![Détails de l’utilisateur](./media/teamseer-tutorial/ic789641.png "User Details")

    a. Dans les zones de texte correspondantes, indiquez le **prénom** , le **nom** et le **nom d’utilisateur (adresse e-mail)** d’un compte Azure AD valide que vous souhaitez provisionner.
  
    b. Cliquez sur **Suivant**.

1. Suivez les instructions à l’écran pour ajouter un nouvel utilisateur, puis cliquez sur **Terminer**.

> [!NOTE]
> Vous pouvez utiliser n’importe quel outil ou API de création de compte d’utilisateur, fourni par TeamSeer, pour approvisionner des comptes utilisateur AAD.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette TeamSeer dans le volet d’accès, vous devez être automatiquement connecté à l’application TeamSeer pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)