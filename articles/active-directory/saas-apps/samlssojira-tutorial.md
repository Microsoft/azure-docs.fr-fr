---
title: 'Tutoriel : Intégration d’Azure Active Directory à SSO SAML pour Jira de Resolution GmbH | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SSO SAML pour Jira de resolution GmbH.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/03/2018
ms.author: jeedes
ms.openlocfilehash: fe241a3fd74e1421f1bd3d39087fe776ee7b61d9
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92891514"
---
# <a name="tutorial-azure-active-directory-integration-with-saml-sso-for-jira-by-resolution-gmbh"></a>Tutoriel : Intégration d’Azure Active Directory à SSO SAML pour Jira de resolution GmbH

Dans ce tutoriel, vous allez découvrir comment configurer SSO SAML pour Jira de resolution GmbH avec Azure Active Directory (Azure AD).
L’intégration de SSO SAML pour Jira de resolution GmbH avec Azure AD offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui est autorisé à se connecter à Jira avec le plug-in SSO SAML de resolution GmbH.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Jira avec leurs comptes Azure AD et le plug-in SSO SAML pour Jira de resolution GmbH (par le biais de l’authentification unique).
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD et SSO SAML pour Jira de resolution GmbH, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement SSO SAML pour Jira de resolution GmbH pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* SSO SAML pour Jira de resolution GmbH prend en charge l’authentification unique lancée par le **fournisseur de services** et le **fournisseur d’identité**

## <a name="adding-an-enterprise-application-for-single-sign-on"></a>Ajout d’une application d’entreprise pour l’authentification unique

Pour configurer l’authentification unique dans Azure AD, vous devez ajouter une application d’entreprise. Pour cela, vous pouvez utiliser l’application préconfigurée **SSO SAML pour Jira de resolution GmbH** qui est disponible dans la galerie.

**Pour ajouter SSO SAML pour Jira de resolution GmbH à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis cliquez sur **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **SSO SAML pour Jira de resolution GmbH** , sélectionnez **SSO SAML pour Jira de resolution GmbH** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application. Vous pouvez renommer l’application d’entreprise si vous le souhaitez.

     ![SSO SAML pour Jira de resolution GmbH dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-single-sign-on-with-the-saml-sso-plugin-and-azure-ad"></a>Configurer et tester l’authentification unique avec le plug-in SSO SAML et Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique à Jira pour un utilisateur Azure AD. Cet utilisateur de test est nommé **Britta Simon**.
Pour que l’authentification unique fonctionne, un lien doit être établi entre un utilisateur Azure AD et l’utilisateur de SSO SAML pour Jira de resolution GmbH associé.

Pour configurer et tester l’authentification unique, effectuez les étapes suivantes :

1. **[Configurer l’application d’entreprise Azure AD pour l’authentification unique](#configure-the-azure-ad-enterprise-application-for-single-sign-on)** .
2. **[Configurer le plug-in SSO SAML de votre instance Jira](#configure-the-saml-sso-plugin-of-your-jira-instance)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour créer un utilisateur de test dans Azure AD.
1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour lui permettre d’utiliser l’authentification unique côté Azure.
1. **[Créer l’utilisateur de test dans Jira](#create-the-test-user-also-in-jira)** pour créer un utilisateur de test dans Jira équivalent à l’utilisateur de test Azure AD.
1. **[Tester l’authentification unique](#test-single-sign-on)** pour vérifier si la configuration fonctionne.

### <a name="configure-the-azure-ad-enterprise-application-for-single-sign-on"></a>Configurer l’application d’entreprise Azure AD pour l’authentification unique

Dans cette section, vous configurez l’authentification unique dans le portail Azure.

Pour configurer l’authentification unique avec SSO SAML pour Jira de resolution GmbH, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans l’application d'entreprise **SSO SAML pour Jira de resolution GmbH** que vous venez de créer, sélectionnez **Authentification unique** dans le panneau gauche.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Pour **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Ensuite, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , effectuez ensuite les étapes suivantes :

    ![Capture d’écran montrant la section « Configuration SAML de base » avec les zones de texte « Identificateur » et « URL de réponse » mises en évidence, et le bouton « Enregistrer » sélectionné.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/samlsso`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/samlsso`

    c. Cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes si vous souhaitez configurer l’application en mode lancé par le **fournisseur de services**  :

    ![Informations d’authentification unique pour le domaine et les URL de SSO SAML pour Jira de resolution GmbH](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/samlsso`

    > [!NOTE]
    > Dans l’Identificateur, l’URL de réponse et l’URL de connexion, remplacez **\<server-base-url>** par l’URL de base de votre instance Jira. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure. Si vous rencontrez un problème, contactez le [support technique de SSO SAML pour Jira de resolution GmbH](https://www.resolution.de/go/support).

4. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , téléchargez le fichier **XML de métadonnées de fédération** et enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

### <a name="configure-the-saml-sso-plugin-of-your-jira-instance"></a>Configurer le plug-in SSO SAML de votre instance Jira 

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre instance Jira comme administrateur.

2. Pointez sur la roue dentée à droite et cliquez sur **Gérer les applications**.
    
    ![Capture d’écran montrant une flèche pointant sur l’icône en forme de roue dentée et l’option « Gérer les applications » sélectionnée dans la liste déroulante](./media/samlssojira-tutorial/addon1.png)

3. Si vous êtes redirigé vers la page d’accès administrateur, entrez le **mot de passe** , puis cliquez sur le bouton **Confirmer**.

    ![Capture d’écran montrant la page d’accès administrateur](./media/samlssojira-tutorial/addon2.png)

4. En principe, Jira vous redirige vers la Place de marché Atlassian. Si ce n’est pas le cas, cliquez sur **Find new apps** (Rechercher les nouvelles applications) dans le panneau de gauche. Recherchez **SAML Single Sign On (SSO) for JIRA** (SSO SAML pour Jira), puis cliquez sur le bouton **Install** (Installer) pour installer le plug-in SAML.

    ![Capture d’écran montrant la page « Atlassian Marketplace for JIRA » avec une flèche pointant sur le bouton « Install » pour l’application « SAML Single Sign On (SSO) JIRA, SAML/SSO »](./media/samlssojira-tutorial/store.png)

5. L’installation du plug-in démarre. À la fin de l’installation, cliquez sur le bouton **Fermer**.

    ![Capture d’écran montrant la boîte de dialogue « Installing »](./media/samlssojira-tutorial/store-2.png)

    ![Capture d’écran montrant la boîte de dialogue « Installed and ready to go! » avec le bouton « Close » sélectionné](./media/samlssojira-tutorial/store-3.png)

6. Cliquez ensuite sur **Gérer**.

    ![Capture d’écran montrant l’application « SAML Single Sign On (SSO) Jira, SAML/SSO » avec le bouton « Manage » sélectionné](./media/samlssojira-tutorial/store-4.png)
    
8. Cliquez maintenant sur **Configurer** pour configurer le nouveau plug-in installé.

    ![Capture d’écran montrant la page « Manage apps », avec le bouton « Configure » sélectionné pour l’application « SAML SingleSignOn for Jira »](./media/samlssojira-tutorial/store-5.png)

9. Dans l’Assistant **SAML SingleSignOn Plugin Configuration** (Configuration du plug-in d’authentification unique SAML), cliquez sur **Add new IdP** (Ajouter un nouveau fournisseur d’identité) pour configurer Azure AD comme nouveau fournisseur d’identité.

    ![Capture d’écran montrant la page « Welcome », avec le bouton « Add new IdP » sélectionné](./media/samlssojira-tutorial/addon4.png) 

10. Dans la page **Choose your SAML Identity Provider** (Choisissez votre fournisseur d’identité SAML), effectuez les étapes suivantes :

    ![Capture d’écran montrant la page « Choose your SAML Identity Provider » avec les zones de texte « IdP Type » et « Name » mises en évidence et le bouton « Next » sélectionné](./media/samlssojira-tutorial/addon5a.png)
 
    a. Définissez **Azure AD** comme type de fournisseurs d’identité.
    
    b. Entrez le **Nom** du fournisseur d’identité (par exemple, Azure AD).
    
    c. Ajoutez une **Description** facultative du fournisseur d’identité (par exemple, Azure AD).
    
    d. Cliquez sur **Suivant**.
    
11. Sur la page **Configuration du fournisseur d’identité** , cliquez sur **Suivant**.
 
    ![Capture d’écran montrant la page « Identity provider configuration »](./media/samlssojira-tutorial/addon5b.png)

12. Dans la page **Import SAML IdP Metadata** (Importer les métadonnées du fournisseur d’identité SAML), effectuez les actions suivantes :

    ![Capture d’écran montrant la page « Import SAML IdP Metadata » avec l’action « Select Metadata XML File » sélectionnée](./media/samlssojira-tutorial/addon5c.png)

    a. Cliquez sur le bouton **Select Metadata XML File** (Sélectionner le fichier XML de métadonnées), puis sélectionnez le fichier **XML de métadonnées de fédération** que vous avez téléchargé précédemment.

    b. Cliquez sur le bouton **Import**.
     
    c. Patientez quelques instants jusqu’à la fin de l’importation.  
     
    d. Cliquez sur le bouton **Suivant**.
    
13. Dans la page **User ID attribute and transformation** (Attribut d’ID d’utilisateur et transformation), cliquez sur le bouton **Next** (Suivant).

    ![Capture d’écran montrant la page « User ID attribute and transformation » avec le bouton « Next » sélectionné](./media/samlssojira-tutorial/addon5d.png)
    
14. Dans la page **User creation and update** (Création et mise à jour de l’utilisateur), cliquez sur **Save & Next** (Enregistrer et suivant) pour enregistrer les paramètres.
    
    ![Capture d’écran montrant la page « User creation and update » avec le bouton « Save & Next » sélectionné](./media/samlssojira-tutorial/addon6a.png)
    
15. Dans la page **Test your settings** (Tester vos paramètres), cliquez sur **Skip test & configure manually** (ignorer le test et configurer manuellement) afin d’ignorer le test de l’utilisateur pour l’instant. Cette opération, qui requiert certains paramètres dans le portail Azure, sera effectuée à la section suivante.
    
    ![Capture d’écran montrant la page « Test your settings » avec le bouton « Skip test & configure manually » sélectionné](./media/samlssojira-tutorial/addon6b.png)
    
16. Cliquez sur **OK** pour ignorer l’avertissement.
    
    ![Capture d’écran montrant la boîte de dialogue d’avertissement avec le bouton « OK » sélectionné](./media/samlssojira-tutorial/addon6c.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure. Avec l’utilisateur, vous allez tester l’authentification unique.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Choisissez **Nouvel utilisateur** en haut de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les **Propriétés de l’utilisateur** , effectuez les étapes suivantes :

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **Britta Simon**.
  
    b. Dans le champ **Nom de l’utilisateur** , entrez <b>BrittaSimon@contoso.com</b>.

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous ajoutez Britta Simon à l’application d’entreprise afin qu’elle puisse utiliser l’authentification unique.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**. 

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, recherchez l’application d’entreprise que vous avez créée au début de ce tutoriel. Si vous avez suivi les étapes de ce tutoriel, l’application s’appelle **SSO SAML pour Jira de resolution GmbH**. Si vous lui avez donné un autre nom, recherchez l’application avec ce nom.

    ![Lien SSO SAML pour Jira de resolution GmbH dans la liste des applications](common/all-applications.png)

3. Dans le panneau de gauche, cliquez sur **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

### <a name="create-the-test-user-also-in-jira"></a>Créer l’utilisateur de test dans Jira également

Pour permettre à des utilisateurs Azure AD de se connecter à SSO SAML pour Jira de resolution GmbH, vous devez les provisionner dans SSO SAML pour Jira de resolution GmbH. Dans le cadre de ce tutoriel, vous devez effectuer le provisionnement manuellement. Toutefois, il existe d’autres modèles de provisionnement disponibles pour le plug-in SSO SAML de resolution, comme le provisionnement **Just In Time** (Juste-à-temps). Consultez la documentation de resolution sur [SAML SSO by resolution GmbH](https://wiki.resolution.de/doc/saml-sso/latest/all). Si vous avez une question à ce sujet, contactez le [support technique de resolution](https://www.resolution.de/go/support).

**Pour provisionner manuellement un compte d’utilisateur, effectuez les étapes suivantes :**

1. Connectez-vous à l’instance Jira comme administrateur.

2. Pointez sur la roue dentée et sélectionnez **User management** (Gestion des utilisateurs).

   ![Capture d’écran montrant une flèche pointant sur l’icône en forme de roue dentée avec l’option « User management » sélectionnée dans la liste déroulante](./media/samlssojira-tutorial/user1.png)

3. Si vous êtes redirigé vers la page d’accès administrateur, entrez le **mot de passe** , puis cliquez sur le bouton **Confirmer**.

    ![Capture d’écran montrant la page « accès administrateur » avec la zone de texte « mot de passe » mise en surbrillance.](./media/samlssojira-tutorial/user2.png) 

4. Sous l’onglet **User management** (Gestion des utilisateurs), cliquez sur **Create user** (Créer un utilisateur).

    ![Capture d’écran montrant l’onglet « User management » avec le bouton « Create user » sélectionné.](./media/samlssojira-tutorial/user3-new.png) 

5. Dans la boîte de dialogue **Create New User** (Créer un utilisateur), effectuez les étapes suivantes. L’utilisateur que vous créez doit être strictement identique à son équivalent dans Azure AD :

    ![Ajouter un employé](./media/samlssojira-tutorial/user4-new.png) 

    a. Dans la zone de texte **Email Address** , entrez l’adresse e-mail de l’utilisateur : <b>BrittaSimon@contoso.com</b>.

    b. Dans la zone de texte **Full Name** (Nom complet), tapez le nom complet de l’utilisateur : **Britta Simon**.

    c. Dans la zone de texte **Username** (Nom d’utilisateur), tapez l’adresse e-mail de l’utilisateur : <b>BrittaSimon@contoso.com</b>. 

    d. Dans la zone de texte **Password** (Mot de passe), tapez le mot de passe de l’utilisateur.

    e. Cliquez sur **Create user** (Créer l’utilisateur) pour terminer la création de l’utilisateur.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette SSO SAML pour Jira de resolution GmbH dans le panneau d’accès, vous devez être automatiquement connecté à l’application SSO SAML pour Jira de resolution GmbH pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

Vous pouvez également tester l’authentification unique à partir de `https://<server-base-url>/plugins/servlet/samlsso`. Remplacez **\<server-base-url>** par l’URL de base de votre instance Jira.


## <a name="enable-single-sign-on-redirection-for-jira"></a>Activer la redirection d’authentification unique pour Jira

Comme indiqué dans la section précédente, il existe actuellement deux façons de déclencher l’authentification unique : soit à partir du **portail Azure** , soit par le biais d’un **lien spécifique vers votre instance Jira**. Le plug-in SSO SAML de resolution GmbH vous permet aussi de déclencher l’authentification unique simplement **en accédant à une URL pointant vers votre instance Jira**.

En fait, tous les utilisateurs qui accèdent à Jira sont redirigés vers l’authentification unique après l’activation d’une option dans le plug-in.

Pour activer la redirection vers l’authentification unique, effectuez les étapes suivantes dans **votre instance Jira**  :

1. Accédez à la page de configuration du plug-in SSO SAML dans Jira.
1. Cliquez sur **Redirection** dans le panneau de gauche.

   ![Capture d’écran partielle de la page de configuration du plug-in Jira SAML SingleSignOn mettant en évidence le lien de redirection dans le volet de navigation gauche.](./media/samlssojira-tutorial/ssore1.png)

1. Cochez **Enable SSO Redirect** (Activer la redirection vers l’authentification unique).

   ![Capture d’écran partielle de la page de configuration du plug-in JIRA SAML SingleSignOn mettant en évidence la case « Enable SSO Redirect » (Activer la redirection de l’authentification unique) cochée.](./media/samlssojira-tutorial/ssore2.png) 

1. Choisissez le bouton **Save Settings** (Enregistrer les paramètres) dans l’angle supérieur droit.

Après avoir activé l’option, vous pouvez toujours accéder à l’invite du nom d’utilisateur/mot de passe, si l’option **Enable nosso** (Activer nosso) est cochée, à partir de `https://\<server-base-url>/login.jsp?nosso`. Comme toujours, remplacez **\<server-base-url>** par votre URL de base.


## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)