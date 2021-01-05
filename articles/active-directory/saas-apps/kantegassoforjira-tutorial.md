---
title: 'Didacticiel : Intégration d’Azure Active Directory avec Kantega SSO pour JIRA | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Kantega SSO pour JIRA.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/16/2019
ms.author: jeedes
ms.openlocfilehash: 9643d0e63e85a9b500021a415e3cdaf3edc756c5
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97608729"
---
# <a name="tutorial-azure-active-directory-integration-with-kantega-sso-for-jira"></a>Didacticiel : Intégration d’Azure Active Directory avec Kantega SSO pour JIRA

Dans ce didacticiel, vous allez apprendre à intégrer Kantega SSO pour JIRA avec Azure Active Directory (Azure AD).
L’intégration de Kantega SSO pour JIRA avec Azure AD offre les avantages suivants :

* Vous pouvez contrôler dans Azure AD qui a accès à Kantega SSO pour JIRA.
* Vous pouvez permettre à vos utilisateurs d’être connectés automatiquement à Kantega SSO pour JIRA (authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec Kantega SSO pour JIRA, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Abonnement Kantega SSO pour JIRA pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Kantega SSO pour JIRA prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité**

## <a name="adding-kantega-sso-for-jira-from-the-gallery"></a>Ajout de Kantega SSO pour JIRA à partir de la galerie

Pour configurer l’intégration de Kantega SSO pour JIRA dans Azure AD, vous devez ajouter Kantega SSO pour JIRA à partir de la galerie à la liste des applications SaaS managées.

**Pour ajouter Kantega SSO pour JIRA à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Kantega SSO pour JIRA**, sélectionnez **Kantega SSO pour JIRA** dans le panneau des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Kantega SSO pour JIRA dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Kantega SSO pour JIRA sur un utilisateur de test nommé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur associé dans Kantega SSO pour JIRA doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Kantega SSO pour JIRA, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Kantega SSO pour JIRA](#configure-kantega-sso-for-jira-single-sign-on)** pour configurer les paramètres d’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Kantega SSO pour JIRA](#create-kantega-sso-for-jira-test-user)** pour avoir un équivalent de Britta Simon dans Kantega SSO pour JIRA qui soit lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Kantega SSO pour JIRA, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Kantega SSO pour JIRA**, cliquez sur **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité**, effectuez les étapes suivantes :

    ![Capture d’écran montrant la « Configuration SAML de base » avec les zones de texte « Identificateur » et « URL de réponse » mises en évidence, et le bouton « Enregistrer » sélectionné.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    ![Informations d’authentification unique pour le domaine et les URL de Kantega SSO pour JIRA](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Ces valeurs sont reçues durant la configuration du plug-in JIRA qui est décrite plus loin dans le didacticiel.

6. Sur la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

7. Dans la section **Configurer Kantega SSO pour JIRA**, copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-kantega-sso-for-jira-single-sign-on"></a>Configurer l’authentification unique Kantega SSO pour JIRA

1. Dans une autre fenêtre du navigateur web, connectez-vous à votre serveur local JIRA en tant qu’administrateur.

1. Pointez sur le roue dentée, puis cliquez sur **Modules complémentaires**.

    ![Capture d’écran montrant l’icône « Roue dentée » sélectionnée, et l’option « Add-ons » sélectionnée dans la liste déroulante.](./media/kantegassoforjira-tutorial/addon1.png)

1. Sous l’onglet Modules complémentaires, cliquez sur **Find new add-ons** (Trouver de nouveaux modules complémentaires). Recherchez **Kantega SSO for JIRA (SAML & Kerberos)** , puis cliquez sur le bouton **Installer** pour installer le nouveau plug-in SAML.

    ![Capture d’écran montrant la section « Find new Add-ons », avec « Kantego SSO for JIRA (SAML & Kerberos) » dans la zone de recherche et le bouton « Install » sélectionné.](./media/kantegassoforjira-tutorial/addon2.png)

1. L’installation du plug-in démarre.

    ![Capture d’écran montrant la boîte de dialogue « Installing » du plug-in.](./media/kantegassoforjira-tutorial/addon3.png)

1. Une fois l’installation terminée. Cliquez sur **Fermer**.

    ![Capture d’écran montrant la boîte de dialogue « Installed and ready to go! » avec l’action « Close » sélectionnée.](./media/kantegassoforjira-tutorial/addon33.png)

1.  Cliquez sur **Gérer**.

    ![Capture d’écran affichant la page de l’application « Kantega SSO » avec le bouton « Manage » sélectionné.](./media/kantegassoforjira-tutorial/addon34.png)
    
1. Le nouveau plug-in est répertorié sous **INTÉGRATIONS**. Cliquez sur **Configurer** pour configurer le nouveau plug-in.

    ![Capture d’écran montrant l’élément « INTEGRATIONS » mis en évidence dans le menu de navigation de gauche, et le bouton « Configure » sélectionné dans la section « Manage add-ons ».](./media/kantegassoforjira-tutorial/addon35.png)

1. Dans la section **SAML**. Dans le menu déroulant **Ajouter le fournisseur d’identité**, sélectionnez **Azure Active Directory (Azure AD)**.

    ![Capture d’écran montrant la liste déroulante « Add identity provider » avec l’option « Azure Active Directory (Azure AD) » sélectionnée.](./media/kantegassoforjira-tutorial/addon4.png)

1. Sélectionnez le niveau d’abonnement **De base**.

    ![Capture d’écran montrant la section « Preparing Azure AD » avec l’option « Basic » sélectionnée.](./media/kantegassoforjira-tutorial/addon5.png)

1. Dans la section **Propriétés de l’application**, procédez comme suit : 

    ![Capture d’écran montrant la section « App properties » avec la zone de texte « App ID URL » et l’icône de copie mises en évidence ; le bouton « Next » est sélectionné.](./media/kantegassoforjira-tutorial/addon6.png)

    1. Copiez la valeur de **URI ID d’application** et utilisez-la en tant que **Identificateur, URL de réponse et URL de connexion** dans la section **Configuration SAML de base** du portail Azure.

    1. Cliquez sur **Suivant**.

1. Dans la section **Metadata import** (Importation des métadonnées), procédez comme suit : 

    ![Capture d’écran montrant la section « Metadata import » avec l’option « Metadata file on my computer » sélectionnée.](./media/kantegassoforjira-tutorial/addon7.png)

    1. Sélectionnez **Metadata file on my computer** (Fichier de métadonnées sur mon ordinateur), puis chargez le fichier de métadonnées que vous avez téléchargé à partir du portail Azure.

    1. Cliquez sur **Suivant**.

1. Dans la section **Name and SSO location** (Nom et emplacement de l’authentification unique), procédez comme suit :

    ![Capture d’écran montrant la section « Name and SSO location », avec la zone de texte « Identity provider name » mise en évidence et le bouton « Next » sélectionné.](./media/kantegassoforjira-tutorial/addon8.png)

    1. Ajoutez le nom du fournisseur d’identité dans la zone de texte **Identity provider name** (Nom du fournisseur d’identité) (par exemple, Azure AD).

    1. Cliquez sur **Suivant**.

1. Vérifiez le certificat de signature, puis cliquez sur **Suivant**.

    ![Capture d’écran montrant la section « Signature verification » avec le bouton « Next » sélectionné.](./media/kantegassoforjira-tutorial/addon9.png)

1. Dans la section **JIRA user accounts** (Comptes d’utilisateur JIRA), procédez comme suit :

    ![Capture d’écran montrant la section « JIRA user accounts », avec l’option « Create users in JIRA’s Internal Directory if needed » mise en évidence et le bouton « Next » sélectionné.](./media/kantegassoforjira-tutorial/addon10.png)

    1. Sélectionnez **Create users in JIRA’s internal Directory if needed** (Créer des utilisateurs dans l’annuaire interne de JIRA si nécessaire) et saisissez le nom de groupe approprié pour les utilisateurs (il peut s’agir de plusieurs groupes séparés par des virgules).

    1. Cliquez sur **Suivant**.

1. Cliquez sur **Terminer**.

    ![Capture d’écran montrant la section « Summary » avec le bouton « Finish » sélectionné.](./media/kantegassoforjira-tutorial/addon11.png)

1. Dans la section **Known domains for Azure AD** (Domaines connus pour Azure AD), procédez comme suit :

    ![Configure Single Sign-On](./media/kantegassoforjira-tutorial/addon12.png)

    1. Sélectionnez **Known domains** (Domaines connus) dans le volet gauche de la page.

    2. Entrez le nom de domaine dans la zone de texte **Known domains** (Domaines connus).

    3. Cliquez sur **Enregistrer**.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    1. Dans le champ **Nom**, entrez **BrittaSimon**.

    1. Dans le champ **Nom d’utilisateur**, tapez `brittasimon@yourcompanydomain.extension`. Par exemple : BrittaSimon@contoso.com

    1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ Mot de passe.

    1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Kantega SSO pour JIRA.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **Kantega SSO pour JIRA**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Kantega SSO pour JIRA**.

    ![Le lien Kantega SSO pour JIRA dans la liste Applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-kantega-sso-for-jira-test-user"></a>Créer un utilisateur de test Kantega SSO pour JIRA

Pour se connecter à JIRA, les utilisateurs Azure AD doivent être provisionnés dans JIRA. Dans Kantega SSO pour JIRA, cet approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre serveur local JIRA en tant qu’administrateur.

1. Pointez sur la roue dentée, puis cliquez sur **Gestion des utilisateurs**.

    ![Capture d’écran montrant l’icône « Roue dentée » mise en évidence et l’option « User management » sélectionnée dans la liste déroulante.](./media/kantegassoforjira-tutorial/user1.png) 

1. Sous l’onglet **User management** (Gestion des utilisateurs), cliquez sur **Create user** (Créer un utilisateur).

    ![Capture d’écran montrant la section « User management » avec le bouton « Create User » sélectionné.](./media/kantegassoforjira-tutorial/user2.png) 

1. Dans la page de boîte de dialogue **Create New User** (Créer un utilisateur), procédez comme suit :

    ![Ajouter un employé](./media/kantegassoforjira-tutorial/user3.png) 

    1. Dans la zone de texte **Email address** (Adresse e-mail), tapez l’adresse e-mail d’un utilisateur, par exemple, Brittasimon@contoso.com.

    2. Dans la zone de texte **Full Name** (Nom complet), tapez le nom complet d’un utilisateur, par exemple, Britta Simon.

    3. Dans la zone de texte **Username** (Nom d’utilisateur), tapez l’e-mail d’un utilisateur, par exemple, Brittasimon@contoso.com.

    4. Dans la zone de texte **Password** (Mot de passe), tapez le mot de passe de l’utilisateur.

    5. Cliquez sur **Create User** (Créer un utilisateur).

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Kantega SSO pour JIRA dans le volet d’accès, vous êtes normalement connecté automatiquement à l’application Kantega SSO pour JIRA pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)