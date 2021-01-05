---
title: 'Didacticiel : Intégration d’Azure Active Directory à Central Desktop | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Central Desktop.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/12/2019
ms.author: jeedes
ms.openlocfilehash: 36ba61c86082e191831c2c890de4466181f1a4db
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97674183"
---
# <a name="tutorial-azure-active-directory-integration-with-central-desktop"></a>Didacticiel : Intégration d’Azure Active Directory à Central Desktop

Dans ce didacticiel, vous allez apprendre à intégrer Central Desktop à Azure Active Directory (Azure AD).
L’intégration de Central Desktop à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Central Desktop
* Vous pouvez autoriser vos utilisateurs à se connecter automatiquement à Central Desktop (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Central Desktop, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Central Desktop pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Central Desktop prend en charge l’authentification unique initiée par le **fournisseur de services**

## <a name="adding-central-desktop-from-the-gallery"></a>Ajout de Central Desktop à partir de la galerie

Pour configurer l’intégration de Central Desktop à Azure AD, vous devez ajouter Central Desktop à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Central Desktop à partir de la galerie, effectuez les étapes suivantes :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Central Desktop**, sélectionnez **Central Desktop** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Central Desktop dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Central Desktop, à l’aide d’un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur Central Desktop associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Central Desktop, vous devez effectuer les actions essentielles suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Central Desktop](#configure-central-desktop-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Central Desktop](#create-central-desktop-test-user)** pour avoir dans Central Desktop un équivalent de Britta Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Central Desktop, effectuez les étapes suivantes :

1. Dans la page d’intégration de l’application **Central Desktop** sur le [portail Azure](https://portal.azure.com/), sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, effectuez les étapes suivantes :

    ![Informations d’authentification unique relatives au domaine et aux URL Central Desktop](common/sp-identifier-reply.png)

    a. Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://<companyname>.centraldesktop.com`.

    b. Dans la zone de texte **Identificateur**, tapez une URL au format suivant :

    ```http
    https://<companyname>.centraldesktop.com/saml2-metadata.php
    https://<companyname>.imeetcentral.com/saml2-metadata.php
    ```

    c. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://<companyname>.centraldesktop.com/saml2-assertion.php`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez [l’équipe du support client de Central Desktop](https://imeetcentral.com/contact-us). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML**, à la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Raw)** en fonction des options définies, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificateraw.png)

6. Dans la section **Configurer Central Desktop**, copiez la ou les URL appropriées correspondant à vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-central-desktop-single-sign-on"></a>Configurer l’authentification unique Central Desktop

1. Connectez-vous à votre locataire **Central Desktop**.

2. Accédez à **Settings**. Sélectionnez **Advanced**, puis **Single Sign On**.

    ![Paramétrage - Avancé](./media/central-desktop-tutorial/ic769563.png "Paramétrage - Avancé")

3. Dans la page **Single Sign On Settings**, effectuez les étapes suivantes :

    ![Paramètres d’authentification unique](./media/central-desktop-tutorial/ic769564.png "Single Sign On Settings")

    a. Sélectionnez **Activer l’authentification unique SAMLv2**.

    b. Dans la zone **SSO URL**, collez la valeur **Identificateur Azure AD** copiée à partir du portail Azure.

    c. Dans la zone **SSO Login URL** (URL de connexion SSO), collez la valeur de l’**URL de connexion** que vous avez copiée à partir du portail Azure.

    d. Dans la zone **SSO Logout URL** (URL de déconnexion SSO), collez la valeur de l’**URL de déconnexion** que vous avez copiée à partir du portail Azure.

4. Dans la section **Message Signature Verification Method**, effectuez les étapes suivantes :

    ![Méthode de vérification de la signature du message](./media/central-desktop-tutorial/ic769565.png "Message Signature Verification Method")

    a. Sélectionnez **Certificate**.

    b. Dans la liste **SSO Certificate**, sélectionnez **RSH SHA256**.

    c. Ouvrez votre certificat téléchargé dans le Bloc-notes. Ensuite, copiez le contenu du certificat et collez-le dans le champ **SSO Certificate**.

    d. Sélectionnez **Display a link to your SAMLv2 login page**.

    e. Sélectionnez **Update**.

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en accordant l’accès à Central Desktop.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **Central Desktop**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Central Desktop**.

    ![Lien Central Desktop dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-central-desktop-test-user"></a>Créer l’utilisateur de test Central Desktop

Pour que les utilisateurs Azure AD puissent se connecter, ils doivent être provisionnés dans l’application Central Desktop. Cette section explique comment créer des comptes d’utilisateur Azure AD dans Central Desktop.

> [!NOTE]
> Pour provisionner des comptes d’utilisateur Azure AD, vous pouvez utiliser tout autre outil ou n’importe quelle API de création de compte d’utilisateur fournis par Central Desktop.

**Pour approvisionner des comptes d’utilisateur dans Central Desktop :**

1. Connectez-vous à votre locataire Central Desktop.

2. Sélectionnez **People** (Contacts), puis sélectionnez **Add Internal Members** (Ajouter des membres internes).

    ![Personnes](./media/central-desktop-tutorial/ic781051.png "Personnes")

3. Dans la zone de texte **Email Address of New Members**, tapez un compte Azure AD à provisionner, puis sélectionnez **Next**.

    ![Adresses e-mail des nouveaux membres](./media/central-desktop-tutorial/ic781052.png "Adresses e-mail des nouveaux membres")

4. Sélectionnez **Add Internal member(s)** .

    ![Ajouter un membre interne](./media/central-desktop-tutorial/ic781053.png "Ajouter un membre interne")

   > [!NOTE]
   > Les utilisateurs que vous ajoutez reçoivent un e-mail contenant un lien de confirmation pour l’activation de leur compte.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Central Desktop dans le volet d’accès doit vous connecter automatiquement à l’application Central Desktop pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)