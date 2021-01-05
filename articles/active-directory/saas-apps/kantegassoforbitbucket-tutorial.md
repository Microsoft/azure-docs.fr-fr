---
title: 'Didacticiel : Intégration d’Azure Active Directory avec Kantega SSO pour Bitbucket | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Kantega SSO pour Bitbucket.
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
ms.openlocfilehash: 52879eb7cb7a9d90113971aa66c590b99b2d5e88
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92459281"
---
# <a name="tutorial-azure-active-directory-integration-with-kantega-sso-for-bitbucket"></a>Didacticiel : Intégration d’Azure Active Directory avec Kantega SSO pour Bitbucket

Dans ce didacticiel, vous allez apprendre à intégrer Kantega SSO pour Bitbucket avec Azure Active Directory (Azure AD).
L’intégration de Kantega SSO pour Bitbucket avec Azure AD offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Kantega SSO pour Bitbucket.
* Vous pouvez permettre à vos utilisateurs d’être connectés automatiquement à Kantega SSO pour Bitbucket (authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec Kantega SSO pour Bitbucket, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Abonnement Kantega SSO pour Bitbucket avec l’authentification unique activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Kantega SSO pour Bitbucket prend en charge l’authentification unique initiée par le **SP et l’IDP**

## <a name="adding-kantega-sso-for-bitbucket-from-the-gallery"></a>Ajout de Kantega SSO pour Bitbucket à partir de la galerie

Pour configurer l’intégration de Kantega SSO pour Bitbucket dans Azure AD, vous devez ajouter Kantega SSO pour Bitbucket à partir de la galerie à la liste des applications SaaS managées.

**Pour ajouter Kantega SSO pour Bitbucket à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Kantega SSO pour Bitbucket** , sélectionnez **Kantega SSO pour Bitbucket** dans le panneau des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Kantega SSO pour JIRA dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Kantega SSO pour Bitbucket grâce à un utilisateur de test nommé **Britta Simon** .
Pour que l’authentification unique fonctionne, il faut établir une relation entre un utilisateur Azure AD et l’utilisateur associé dans Kantega SSO pour Bitbucket.

Pour configurer et tester l’authentification unique Azure AD avec Kantega SSO pour Bitbucket, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Kantega SSO pour Bitbucket](#configure-kantega-sso-for-bitbucket-single-sign-on)** pour configurer les paramètres d’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Kantega SSO pour Bitbucket](#create-kantega-sso-for-bitbucket-test-user)** pour avoir un équivalent de Britta Simon dans Kantega SSO pour Bitbucket lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Kantega SSO pour Bitbucket, procédez comme suit :

1. Sur le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Kantega SSO pour Bitbucket** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. À la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode initié par **IDP** , suivez les étapes ci-dessous :

    ![Capture d’écran montrant Configuration SAML de base, où vous pouvez entrer l’identificateur, l’URL de réponse, et sélectionner Enregistrer.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    ![Capture d’écran montrant Définir des URL supplémentaires, où vous pouvez entrer une URL de connexion.](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Ces valeurs sont reçues durant la configuration du plug-in Bitbucket qui est décrite plus loin dans le tutoriel.

6. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

7. Dans la section **Configurer Kantega SSO pour Bitbucket** , copiez la ou les URL appropriée(s) selon vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-kantega-sso-for-bitbucket-single-sign-on"></a>Configurer l’authentification unique Kantega SSO pour Bitbucket

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre portail d’administration Bitbucket en tant qu’administrateur.

1. Cliquez sur le roue dentée, puis cliquez sur **Find new add-ons (Trouver de nouveaux modules complémentaires)** .

    ![Capture d’écran montrant l’administration de BitBucket avec l’option Find new add-ons sélectionnée.](./media/kantegassoforbitbucket-tutorial/addon1.png)

1. Recherchez **Kerberos SAML Kantega SSO pour Bitbucket** , puis cliquez sur le bouton **Installer** pour installer le nouveau plug-in SAML.

    ![Capture d’écran montrant l’option Kantega SSO for Bitbucket SAML & Kerberos avec la possibilité d’installation.](./media/kantegassoforbitbucket-tutorial/addon2.png)

1. L’installation du plug-in démarre.

    ![Capture d’écran montrant la progression de l’installation.](./media/kantegassoforbitbucket-tutorial/addon31.png)

1. Une fois l’installation terminée. Cliquez sur **Fermer** .

    ![Capture d’écran montrant le bouton Close.](./media/kantegassoforbitbucket-tutorial/addon33.png)

1. Cliquez sur **Gérer** .

    ![Capture d’écran montrant le bouton Manage.](./media/kantegassoforbitbucket-tutorial/addon34.png)

1. Cliquez sur **Configurer** pour configurer le nouveau plug-in.

    ![Capture d’écran affichant la section User-installed add-ons avec l’action Configure sélectionnée.](./media/kantegassoforbitbucket-tutorial/addon35.png)

1. Dans la section **SAML** . Dans le menu déroulant **Ajouter le fournisseur d’identité** , sélectionnez **Azure Active Directory (Azure AD)** .

    ![Capture d’écran montrant l’authentification unique Kantega avec la sélection d’Azure AD comme fournisseur d’identité.](./media/kantegassoforbitbucket-tutorial/addon4.png)

1. Sélectionnez le niveau d’abonnement **De base** .

    ![Capture d’écran montrant la préparation d’Azure AD avec l’option Basic sélectionnée.](./media/kantegassoforbitbucket-tutorial/addon5.png)

1. Dans la section **Propriétés de l’application** , procédez comme suit :

    ![Capture d’écran montrant la section App properties, dans laquelle vous pouvez fournir les informations de cette étape.](./media/kantegassoforbitbucket-tutorial/addon6.png)

    a. Copiez la valeur de **URI ID d’application** et utilisez-la en tant que **Identificateur, URL de réponse et URL de connexion** dans la section **Configuration SAML de base** du portail Azure.

    b. Cliquez sur **Suivant** .

1. Dans la section **Metadata import** (Importation des métadonnées), procédez comme suit :

    ![Capture d’écran montrant la section Metadata import dans laquelle vous pouvez accéder à un fichier de métadonnées.](./media/kantegassoforbitbucket-tutorial/addon7.png)

    a. Sélectionnez **Metadata file on my computer** (Fichier de métadonnées sur mon ordinateur), puis chargez le fichier de métadonnées que vous avez téléchargé à partir du portail Azure.

    b. Cliquez sur **Suivant** .

1. Dans la section **Name and SSO location** (Nom et emplacement de l’authentification unique), procédez comme suit :

    ![Capture d’écran montrant la section Name and SSO location dans laquelle Azure AD est le nom du fournisseur d’identité.](./media/kantegassoforbitbucket-tutorial/addon8.png)

    a. Ajoutez le nom du fournisseur d’identité dans la zone de texte **Identity provider name** (Nom du fournisseur d’identité) (par exemple, Azure AD).

    b. Cliquez sur **Suivant** .

1. Vérifiez le certificat de signature, puis cliquez sur **Suivant** .

    ![Capture d’écran montrant la section Signature verification.](./media/kantegassoforbitbucket-tutorial/addon9.png)

1. Dans la section **Bitbucket user accounts** (Comptes d’utilisateur Bitbucket), procédez comme suit :

    ![Capture d’écran montrant la section BitBucket user accounts dans laquelle vous avez la possibilité de créer des utilisateurs.](./media/kantegassoforbitbucket-tutorial/addon10.png)

    a. Sélectionnez **Create users in Bitbucket's internal Directory if needed** (Créer des utilisateurs dans l’annuaire interne de JIRA si nécessaire) et entrez le nom de groupe approprié pour les utilisateurs (peut être plusieurs groupes séparés par des virgules).

    b. Cliquez sur **Suivant** .

1. Cliquez sur **Terminer** .

    ![Capture d’écran montrant la page Summary.](./media/kantegassoforbitbucket-tutorial/addon11.png)

1. Dans la section **Known domains for Azure AD** (Domaines connus pour Azure AD), procédez comme suit :

    ![Capture d’écran montrant la section Known domains for Azure AD dans laquelle vous pouvez effectuer ces étapes.](./media/kantegassoforbitbucket-tutorial/addon12.png)

    a. Sélectionnez **Known domains** (Domaines connus) dans le volet gauche de la page.

    b. Entrez le nom de domaine dans la zone de texte **Known domains** (Domaines connus).

    c. Cliquez sur **Enregistrer** .

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

Dans cette section, vous permettez à Britta Simon d’utiliser l’authentification unique Azure en lui accordant l’accès à Kantega SSO pour Bitbucket.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Kantega SSO pour Bitbucket** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Kantega SSO pour Bitbucket** .

    ![Lien Kantega SSO pour Bitbucket dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-kantega-sso-for-bitbucket-test-user"></a>Créer un utilisateur de test Kantega SSO pour Bitbucket

Pour permettre aux utilisateurs Azure AD de se connecter à Bitbucket, vous devez les attribuer dans Bitbucket. Pour Kantega SSO pour Bitbucket, cet approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise Bitbucket en tant qu’administrateur.

1. Cliquez sur l’icône Paramètres.

    ![Capture d’écran montrant l’icône Settings.](./media/kantegassoforbitbucket-tutorial/user1.png) 

1. Sous la section **Administration** , cliquez sur **Users (Utilisateurs)** .

    ![Capture d’écran montrant l’administration de BitBucket avec l’élément Users sélectionné. ](./media/kantegassoforbitbucket-tutorial/user2.png)

1. Cliquez sur **Create User** (Créer un utilisateur).

    ![Capture d’écran montrant l’administration de BitBucket avec l’action Create user sélectionnée.](./media/kantegassoforbitbucket-tutorial/user3.png)   

1. Dans la page de boîte de dialogue **Create User** (Créer un utilisateur), procédez comme suit :

    ![Capture d’écran montrant la boîte de dialogue Create user dans laquelle vous pouvez effectuer ces étapes.](./media/kantegassoforbitbucket-tutorial/user4.png) 

    a. Dans la zone de texte **Username** (Nom d’utilisateur), tapez l’e-mail d’un utilisateur, par exemple, Brittasimon@contoso.com.

    b. Dans la zone de texte **Full Name** (Nom complet), tapez le nom complet d’un utilisateur, par exemple, Britta Simon.

    c. Dans la zone de texte **Email address** (Adresse e-mail), tapez l’adresse e-mail d’un utilisateur, par exemple, Brittasimon@contoso.com.

    d. Dans la zone de texte **Password** (Mot de passe), tapez le mot de passe de l’utilisateur.

    e. Dans la zone de texte **Confirm Password** (Confirmer le mot de passe), entrez à nouveau le mot de passe de l’utilisateur.

    f. Cliquez sur **Create User** (Créer un utilisateur).

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Kantega SSO pour Bitbucket dans le volet d’accès, vous êtes normalement connecté automatiquement à l’application Kantega SSO pour Bitbucket pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)