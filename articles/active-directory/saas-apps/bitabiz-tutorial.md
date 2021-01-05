---
title: 'Didacticiel : Intégration de Azure Active Directory à BitaBIZ | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et BitaBIZ.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/06/2019
ms.author: jeedes
ms.openlocfilehash: f564032873be6e4c70426d48c4576371862ea35d
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97673603"
---
# <a name="tutorial-azure-active-directory-integration-with-bitabiz"></a>Didacticiel : Intégration de Azure Active Directory à BitaBIZ

Dans ce tutoriel, vous allez apprendre à intégrer Azure Active Directory (Azure AD) dans BitaBIZ.
L’intégration de BitaBIZ dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à BitaBIZ.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à BitaBIZ (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration de Azure AD avec BitaBIZ, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement BitaBIZ pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* BitaBIZ prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**

## <a name="adding-bitabiz-from-the-gallery"></a>Ajout de BitaBIZ depuis la galerie

Pour configurer l’intégration de BitaBIZ avec Azure AD, vous devez ajouter BitaBIZ, disponible dans la galerie, à votre liste d’applications SaaS gérées.

**Pour ajouter BitaBIZ à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **BitaBIZ**, sélectionnez **BitaBIZ** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![BitaBIZ dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec BitaBIZ, avec un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur BitaBIZ associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec BitaBIZ, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique BitaBIZ](#configure-bitabiz-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test BitaBIZ](#create-bitabiz-test-user)** pour avoir un équivalent de Britta Simon dans BitaBIZ lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec BitaBIZ, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **BitaBIZ**, sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité**, effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL BitaBIZ](common/idp-identifier.png)

    Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `https://www.bitabiz.com/<instanceId>`

    > [!NOTE]
    > La valeur dans l’URL ci-dessus est uniquement à des fins de démonstration. Mettez à jour la valeur avec l’identificateur réel. La procédure est expliquée plus loin dans le didacticiel.

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    ![image](common/both-preintegrated-signon.png)

    Dans la zone de texte **URL de connexion**, tapez l’URL : `https://www.bitabiz.com/dashboard`

6. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

7. Dans la section **Configurer BitaBIZ**, copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-bitabiz-single-sign-on"></a>Configurer l’authentification unique BitaBIZ

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre client BitaBIZ en tant qu’administrateur.

2. Cliquez sur **CONFIGURATION ADMINISTRATEUR**.

    ![Capture d’écran montrant une partie d’une fenêtre de navigateur avec Setup Admin sélectionné.](./media/bitabiz-tutorial/settings1.png)

3. Cliquez sur **Intégrations Microsoft** sous la section **Ajouter une valeur**.

    ![Capture d’écran montrant Add value avec Microsoft integrations sélectionné.](./media/bitabiz-tutorial/settings2.png)

4. Faites défiler jusqu’à la section **Microsoft Azure AD (activer l’authentification unique)** et procédez comme suit :

    ![Capture d’écran montrant la section Microsoft Azure AD, où vous entrez les informations décrites dans cette étape.](./media/bitabiz-tutorial/settings3.png)

    a. Copiez la valeur de la zone de texte **Entity ID** (« Identificateur » dans Azure AD) et collez-la dans la zone de texte **Identificateur** de la section **Configuration SAML de base** dans le portail Azure. 

    b. Dans la zone de texte **Azure AD Single Sign-On Service URL** (URL du service d’authentification unique Azure AD), collez la valeur **URL de connexion** que vous avez copiée à partir du portail Azure.

    c. Dans la zone de texte **Azure AD SAML Entity ID** (ID d’entité SAML Azure AD), collez l’**Identificateur Azure AD** que vous avez copié dans le portail Azure.

    d. Ouvrez dans le Bloc-notes votre fichier **Certificate(Base64)** téléchargé, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **Certificat de signature Azure AD (encodé Base64)** .

    e. Ajoutez le nom de domaine de messagerie de votre entreprise, ici « mycompany.com » dans la zone de texte **Nom de domaine** pour attribuer l’authentification unique aux utilisateurs de votre société avec ce domaine de messagerie (NON OBLIGATOIRE).

    f. Cochez la case **Authentification unique activée** pour le compte BitaBIZ.

    g. Cliquez sur **Enregistrer la configuration Azure AD** pour enregistrer et activer la configuration de l’authentification unique.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom**, entrez **BrittaSimon**.

    b. Dans le champ **Nom d’utilisateur**, tapez **brittasimon\@domainedevotreentreprise.extension**.  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à BitaBIZ.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **BitaBIZ**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **BitaBIZ**.

    ![Lien BitaBIZ dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-bitabiz-test-user"></a>Créer un utilisateur de test BitaBIZ

Pour permettre aux utilisateurs Azure AD de se connecter à BitaBIZ, vous devez les attribuer dans BitaBIZ.  
En l’occurrence, cet approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise BitaBIZ en tant qu’administrateur.

2. Cliquez sur **CONFIGURATION ADMINISTRATEUR**.

    ![Capture d’écran montrant une partie de votre fenêtre de navigateur avec Setup Admin sélectionné.](./media/bitabiz-tutorial/settings1.png)

3. Cliquez sur **Ajouter des utilisateurs** sous la section **Organisation**.

    ![Capture d’écran montrant la section Organization avec Add users sélectionné.](./media/bitabiz-tutorial/user1.png)

4. Cliquez sur **Ajouter un nouvel employé**.

    ![Capture d’écran montrant Add users avec Add new employee sélectionné.](./media/bitabiz-tutorial/user2.png)

5. Dans la boîte de dialogue **Ajouter un nouvel employé**, effectuez les étapes suivantes :

    ![Capture d’écran montrant la page où vous entrez les informations décrites dans cette étape.](./media/bitabiz-tutorial/user3.png)

    a. Dans la zone de texte **Prénom**, tapez le prénom de l’utilisateur, par exemple Britta.

    b. Dans la zone de texte **Nom de famille**, tapez le nom de l’utilisateur, par exemple Simon.

    c. Dans la zone de texte **Email** (E-mail), tapez l’adresse e-mail d’un utilisateur, par exemple, Brittasimon@contoso.com.

    d. Sélectionnez une date dans **Date d’emploi**.

    e. Il existe d’autres attributs non obligatoires pouvant être configurés pour l’utilisateur. Veuillez consulter le [document de configuration d’un nouvel employé](https://help.bitabiz.dk/manage-or-set-up-your-account/on-boarding-employees/new-employee) pour plus d’informations.

    f. Cliquez sur **Enregistrer un employé**.

    > [!NOTE]
    > Le titulaire du compte Azure Active Directory reçoit un e-mail contenant un lien à suivre pour confirmer son compte et l’activer.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette BitaBIZ dans le volet d’accès, vous devez être connecté automatiquement à l’application BitaBIZ pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)