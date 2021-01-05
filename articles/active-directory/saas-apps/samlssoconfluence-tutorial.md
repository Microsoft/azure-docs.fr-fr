---
title: 'Tutoriel : Intégration d’Azure Active Directory à SSO SAML pour Confluence de resolution GmbH | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SSO SAML pour Confluence de resolution GmbH.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/24/2018
ms.author: jeedes
ms.openlocfilehash: c8f85c6dd42f1f4505474e03e378c0fe48d70005
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896478"
---
# <a name="tutorial-azure-active-directory-integration-with-saml-sso-for-confluence-by-resolution-gmbh"></a>Tutoriel : Intégration d’Azure Active Directory avec SSO SAML pour Confluence de resolution GmbH

Dans ce didacticiel, vous allez apprendre à intégrer SSO SAML pour Confluence de resolution GmbH avec Azure Active Directory (Azure AD).
L’intégration de SSO SAML pour Confluence de resolution GmbH avec Azure AD offre les avantages suivants :

* Vous pouvez contrôler dans Azure AD qui a accès à SSO SAML pour Confluence de resolution GmbH.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à SSO SAML pour Confluence de resolution GmbH (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD avec SSO SAML pour Confluence de resolution GmbH, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement SSO SAML pour Confluence de resolution GmbH pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* SSO SAML pour Confluence de resolution GmbH prend en charge l’authentification unique initiée par le **fournisseur de services** et le **fournisseur d’identité**

## <a name="adding-saml-sso-for-confluence-by-resolution-gmbh-from-the-gallery"></a>Ajout de SSO SAML pour Confluence de resolution GmbH à partir de la galerie

Pour configurer l’intégration de SSO SAML pour Confluence de resolution GmbH dans Azure AD, vous devez ajouter SSO SAML pour Confluence de resolution GmbH à partir de la galerie dans votre liste d’applications SaaS gérées.

**Pour ajouter SSO SAML pour Confluence de resolution GmbH à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **SSO SAML pour Confluence de resolution GmbH** , sélectionnez **SSO SAML pour Confluence de resolution GmbH** dans le volet de résultats, puis cliquez sur **Ajouter** pour ajouter l’application.

     ![SSO SAML pour Confluence de resolution GmbH dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec SSO SAML pour Confluence de resolution GmbH, en tirant parti d’un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, un lien doit être établi entre un utilisateur Azure AD et l’utilisateur de SSO SAML pour Confluence de resolution GmbH associé.

Pour configurer et tester l’authentification unique Azure AD avec SSO SAML pour Confluence de resolution GmbH, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique SSO SAML pour Confluence de resolution GmbH](#configure-saml-sso-for-confluence-by-resolution-gmbh-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test de SSO SAML pour Confluence de resolution GmbH](#create-saml-sso-for-confluence-by-resolution-gmbh-test-user)** : pour avoir un équivalent de Britta Simon dans SSO SAML pour Confluence de resolution GmbH qui soit lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec SSO SAML pour Confluence de resolution GmbH, effectuez les étapes suivantes :

1. Sur le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **SSO SAML pour Confluence de resolution GmbH** , sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , suivez les étapes ci-dessous si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité**  :

    ![Capture d’écran montrant la « Configuration SAML de base » avec les zones de texte « Identificateur » et « URL de réponse » mises en évidence, et l’action « Enregistrer » sélectionnée.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/samlsso`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/samlsso`

    c. Cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes si vous souhaitez configurer l’application en mode Initié par le fournisseur de services :

    ![informations d’authentification unique pour le domaine et les URL de SSO SAML pour Confluence de resolution GmbH](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/samlsso`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez l’[équipe de support de SSO SAML pour Confluence de resolution GmbH](https://www.resolution.de/go/support). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

4. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

### <a name="configure-saml-sso-for-confluence-by-resolution-gmbh-single-sign-on"></a>Configurer l’authentification unique SSO SAML pour Confluence de resolution GmbH

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre **portail d’administration de SSO SAML pour Confluence de resolution GmbH** en tant qu’administrateur.

2. Pointez sur le roue dentée, puis cliquez sur **Modules complémentaires**.
    
    ![Capture d’écran montrant l’icône « Roue dentée » sélectionnée, et l’option « Add-ons » sélectionnée dans la liste déroulante.](./media/samlssoconfluence-tutorial/addon1.png)

3. Vous êtes redirigé vers la page d’accès administrateur. Entrez le mot de passe, puis cliquez sur le bouton **Confirmer**.

    ![Capture d’écran montrant la page « Administrator Access » avec le bouton « Confirm » sélectionné](./media/samlssoconfluence-tutorial/addon2.png)

4. Sous **ATLASSIAN MARKETPLACE** , cliquez sur **Find new add-ons** (Trouver de nouveaux modules complémentaires). 

    ![Capture d’écran montrant l’onglet « ATTLASSIAN MARKETPLACE » avec l’option « Find new add-ons » sélectionnée](./media/samlssoconfluence-tutorial/addon.png)

5. Recherchez **SAML Single Sign On (SSO) for Confluence** , puis cliquez sur le bouton **Install** pour installer le nouveau plug-in SAML.

    ![Capture d’écran montrant la page « Find new add-ons », avec « SAML Single Sign On (SSO) for Confluence » dans la zone de recherche et le bouton « Install » sélectionné](./media/samlssoconfluence-tutorial/addon7.png)

6. L’installation du plug-in démarre. Cliquez sur **Fermer**.

    ![Capture d’écran montrant la boîte de dialogue « Installing »](./media/samlssoconfluence-tutorial/addon8.png)

    ![Capture d’écran montrant la boîte de dialogue « Installed and ready to go! » avec l’action « Close » sélectionnée.](./media/samlssoconfluence-tutorial/addon9.png)

7.  Cliquez sur **Gérer**.

    ![Capture d’écran montrant la page de l’application « SAML Single Sign On (SSO) for Confluence » avec le bouton « Manage » sélectionné](./media/samlssoconfluence-tutorial/addon10.png)
    
8. Cliquez sur **Configurer** pour configurer le nouveau plug-in.

    ![Capture d’écran montrant la page « Manage » avec le bouton « Configure » sélectionné](./media/samlssoconfluence-tutorial/addon11.png)

9. Ce nouveau plug-in figure également sous l’onglet **UTILISATEURS ET SÉCURITÉ**.

    ![Capture d’écran montrant l’onglet « USERS & SECURITY » avec « SAML SingleSignOn » sélectionné](./media/samlssoconfluence-tutorial/addon3.png)
    
10. Dans la page **SAML SingleSignOn Plugin Configuration** (Configuration du plug-in d’authentification unique SAML), cliquez sur le bouton **Add new IdP** (Ajouter un nouveau fournisseur d’identité) pour configurer les paramètres du fournisseur d’identité.

    ![Capture d’écran montrant la page « SAML SingleSignOn Plugin Configuration », avec le bouton « Add new IdP » sélectionné](./media/samlssoconfluence-tutorial/addon4.png)

11. Dans la page **Choose your SAML Identity Provider** (Choisissez votre fournisseur d’identité SAML), effectuez les actions suivantes :

    ![Capture d’écran montrant la page « Choose your SAML Identity Provider » avec les zones de texte « IdP Type », « Name » et « Description » mises en évidence](./media/samlssoconfluence-tutorial/addon5a.png)
 
    a. Définissez **Azure AD** comme type de fournisseurs d’identité.
    
    b. Ajoutez le **Nom** du fournisseur d’identité (p. ex. Azure AD).
    
    c. Ajoutez la **Description** du fournisseur d’identité (p. ex. Azure AD).
    
    d. Cliquez sur **Suivant**.
    
12. Dans la page **Identity provider configuration** (Configuration du fournisseur d’identité), cliquez sur le bouton **Next** (Suivant).

    ![Capture d’écran montrant la page « Identity provider configuration » avec le bouton « Next » sélectionné](./media/samlssoconfluence-tutorial/addon5b.png)

13. Dans la page **Import SAML IdP Metadata** (Importer les métadonnées du fournisseur d’identité SAML), effectuez les actions suivantes :

    ![Capture d’écran montrant la page « Import SAML IdP Metadata » avec les boutons « Import », « Load File » et « Next » sélectionnés](./media/samlssoconfluence-tutorial/addon5c.png)

    a. Cliquez sur le bouton **Load File** (Charger le fichier) et choisissez le fichier XML de métadonnées que vous avez téléchargé à l’étape 5.

    b. Cliquez sur le bouton **Import** (Importer).
    
    c. Patientez quelques instants jusqu’à la fin de l’importation.
    
    d. Cliquez sur le bouton **Next** (Suivant).
    
14. Dans la page **User ID attribute and transformation** (Attribut d’ID d’utilisateur et transformation), cliquez sur le bouton **Next** (Suivant).

    ![Capture d’écran montrant la page « User ID attribute and transformation » avec le bouton « Next » sélectionné](./media/samlssoconfluence-tutorial/addon5d.png)
    
15. Dans la page **User creation and update** (Création et mise à jour de l’utilisateur), cliquez sur **Save & Next** (Enregistrer et suivant) pour enregistrer les paramètres.   
    
    ![Capture d’écran montrant la page « User creation and update » avec le bouton « Save & Next » sélectionné](./media/samlssoconfluence-tutorial/addon6a.png)
    
16. Dans la page **Test your settings** (Tester vos paramètres), cliquez sur **Skip test & configure manually** (ignorer le test et configurer manuellement) afin d’ignorer le test de l’utilisateur pour l’instant. Cette opération, qui requiert certains paramètres dans le portail Azure, sera effectuée à la section suivante. 
    
    ![Capture d’écran montrant la page « Test your settings » avec le bouton « Skip test & configure manually » sélectionné](./media/samlssoconfluence-tutorial/addon6b.png)
    
17. Dans la boîte de dialogue **Skipping the test means...** (Ignorer le test signifie...), cliquez sur **OK**.
    
    ![Configure Single Sign-On](./media/samlssoconfluence-tutorial/addon6c.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon**.
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon\@domainedevotreentreprise.extension**.  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous activez Britta Simon pour utiliser l’authentification unique Azure en accordant l’accès à SSO SAML pour Confluence de resolution GmbH.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **SSO SAML pour Confluence de resolution GmbH**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, tapez et sélectionnez **SSO SAML pour Confluence de resolution GmbH**.

    ![Lien SSO SAML pour Confluence de resolution GmbH dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

### <a name="create-saml-sso-for-confluence-by-resolution-gmbh-test-user"></a>Créer un utilisateur de test pour SSO SAML pour Confluence de resolution GmbH

Pour permettre à des utilisateurs d’Azure AD de se connecter à SSO SAML pour Confluence de resolution GmbH, vous devez configurer ceux-ci dans SSO SAML pour Confluence de resolution GmbH.  
Dans SSO SAML pour Confluence de resolution GmbH, cette configuration est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise SSO SAML pour Confluence de resolution GmbH en tant qu’administrateur.

2. Pointez sur la roue dentée, puis cliquez sur **Gestion des utilisateurs**.

    ![Capture d’écran montrant l’icône « Roue dentée » mise en évidence et l’option « User management » sélectionnée dans le menu](./media/samlssoconfluence-tutorial/user1.png) 

3. Dans la section Utilisateurs, cliquez sur l’onglet **Add users** (Ajouter des utilisateurs). Dans la page de boîte de dialogue **Add a User** (Ajouter un utilisateur), procédez comme suit :

    ![Ajouter un employé](./media/samlssoconfluence-tutorial/user2.png) 

    a. Dans la zone de texte **Username** (Nom d’utilisateur), tapez le nom d’un utilisateur, par exemple, Britta Simon.

    b. Dans la zone de texte **Full Name** (Nom complet), tapez le nom complet d’un utilisateur, par exemple, Britta Simon.

    c. Dans la zone de texte **Email** (E-mail), tapez l’adresse e-mail d’un utilisateur, par exemple, Brittasimon@contoso.com.

    d. Dans la zone de texte **Password** (Mot de passe), tapez le mot de passe de Britta Simon.

    e. Cliquez sur **Confirm Password** (Confirmer le mot de passe), puis entrez de nouveau le mot de passe.
    
    f. Cliquez sur le bouton **Ajouter**.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette SSO SAML pour Confluence de resolution GmbH dans le panneau d’accès, vous devez être automatiquement connecté à l’application SSO SAML pour Confluence de resolution GmbH pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)