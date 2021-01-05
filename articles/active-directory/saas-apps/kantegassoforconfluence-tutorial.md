---
title: 'Tutoriel : Intégration d’Azure Active Directory avec Kantega SSO pour Confluence | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Kantega SSO pour Confluence.
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
ms.openlocfilehash: ff2d7f45626055cdbc98ca944eecaa01f9bd052b
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92459201"
---
# <a name="tutorial-azure-active-directory-integration-with-kantega-sso-for-confluence"></a>Tutoriel : Intégration d’Azure Active Directory avec Kantega SSO pour Confluence

Dans ce didacticiel, vous allez apprendre à intégrer Kantega SSO pour Confluence avec Azure Active Directory (Azure AD).
L’intégration de Kantega SSO pour Confluence avec Azure AD offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Kantega SSO pour Confluence.
* Vous pouvez permettre à vos utilisateurs d’être connectés automatiquement à Kantega SSO pour Confluence (authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD avec Kantega SSO pour Confluence, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Abonnement Kantega SSO pour Confluence pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Kantega SSO pour Confluence prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité**

## <a name="adding-kantega-sso-for-confluence-from-the-gallery"></a>Ajout de Kantega SSO pour Confluence à partir de la galerie

Pour configurer l’intégration de Kantega SSO pour Confluence dans Azure AD, vous devez ajouter Kantega SSO pour Confluence à partir de la galerie à la liste des applications SaaS managées.

**Pour ajouter Kantega SSO pour Confluence à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Kantega SSO pour Confluence** , sélectionnez **Kantega SSO pour Confluence** dans le panneau des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Kantega SSO pour Confluence dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Kantega SSO pour Confluence sur un utilisateur de test nommé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur associé dans Kantega SSO pour Confluence doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Kantega SSO pour Confluence, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Kantega SSO pour Confluence](#configure-kantega-sso-for-confluence-single-sign-on)** pour configurer les paramètres d’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Création d’un utilisateur de test Kantega SSO pour Confluence](#create-kantega-sso-for-confluence-test-user)** pour avoir un équivalent de Britta Simon dans Kantega SSO pour Confluence qui soit lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Kantega SSO pour Confluence, procédez comme suit :

1. Sur le [Portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Kantega SSO pour Confluence** , cliquez sur **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , effectuez les étapes suivantes :

    ![Capture d’écran montrant la section « Configuration SAML de base » avec les champs « Identificateur » et « URL de réponse » mis en évidence, et le bouton « Enregistrer » sélectionné.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    ![Informations d’authentification unique pour le domaine et les URL de Kantega SSO pour Confluence](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/no.kantega.saml/sp/<uniqueid>/login`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Ces valeurs sont reçues durant la configuration du plug-in Confluence qui est décrite plus loin dans le didacticiel.

6. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

7. Dans la section **Configurer Kantega SSO pour Confluence** , copiez la ou les URL appropriée(s) en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-kantega-sso-for-confluence-single-sign-on"></a>Configurer l’authentification unique Kantega SSO pour Confluence

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre **portail d’administration Confluence** en tant qu’administrateur.

1. Pointez sur le roue dentée, puis cliquez sur **Modules complémentaires** .

    ![Capture d’écran montrant l’icône de menu « Roue dentée » et l’option « Add-ons » sélectionnée.](./media/kantegassoforconfluence-tutorial/addon1.png)

1. Sous **ATLASSIAN MARKETPLACE** , cliquez sur **Find new add-ons** (Trouver de nouveaux modules complémentaires).

    ![Capture d’écran montrant l’onglet « ATTLASSIAN MARKETPLACE » avec l’option « Find new add-ons » sélectionnée.](./media/kantegassoforconfluence-tutorial/addon.png)

1. Recherchez **Kerberos SAML Kantega SSO pour Confluence** , puis cliquez sur le bouton **Installer** pour installer le nouveau plug-in SAML.

    ![Capture d’écran montrant la page « Find new add-ons », avec « Kantega SSO for Confluence SAML Kerberos » dans la zone de recherche et le bouton « Install » sélectionné.](./media/kantegassoforconfluence-tutorial/addon2.png)

1. L’installation du plug-in démarre.

    ![Capture d’écran montrant la fenêtre « Installing » du plug-in.](./media/kantegassoforconfluence-tutorial/addon3.png)

1. Une fois l’installation terminée. Cliquez sur **Fermer** .

    ![Capture d’écran montrant l’écran « Installed and ready to go » avec l’action « Close » sélectionnée.](./media/kantegassoforconfluence-tutorial/addon33.png)

1. Cliquez sur **Gérer** .

    ![Capture d’écran montrant le plug-in « Kantega Single Sign-on with Kerberos and SAML » avec le bouton « Manage » sélectionné.](./media/kantegassoforconfluence-tutorial/addon34.png)

1. Cliquez sur **Configurer** pour configurer le nouveau plug-in.

    ![Capture d’écran montrant la page « Kantega Single Sign-on with Kerberos and SAML » avec le bouton « Configure » sélectionné.](./media/kantegassoforconfluence-tutorial/addon35.png)

1. Ce nouveau plug-in figure également sous l’onglet **UTILISATEURS ET SÉCURITÉ** .

    ![Capture d’écran montrant l’onglet « USERS & SECURITY » avec l’action « Kantega Single Sign-on » sélectionnée.](./media/kantegassoforconfluence-tutorial/addon36.png)

1. Dans la section **SAML** . Dans le menu déroulant **Ajouter le fournisseur d’identité** , sélectionnez **Azure Active Directory (Azure AD)** .

    ![Capture d’écran montrant la section « SAML » avec les éléments « Add Identity provider » et « Azure Active Directory (Azure AD) » sélectionnés.](./media/kantegassoforconfluence-tutorial/addon4.png)

1. Sélectionnez le niveau d’abonnement **De base** .

    ![Capture d’écran montrant la page « Preparing Azure AD » avec l’option « Basic » sélectionnée.](./media/kantegassoforconfluence-tutorial/addon5.png)

1. Dans la section **Propriétés de l’application** , procédez comme suit :

    ![Capture d’écran montrant la section « App properties » avec le champ « App ID URL » et le bouton « Copy » mis en évidence, et le bouton « Next » sélectionné.](./media/kantegassoforconfluence-tutorial/addon6.png)

    a. Copiez la valeur de **URI ID d’application** et utilisez-la en tant que **Identificateur, URL de réponse et URL de connexion** dans la section **Configuration SAML de base** du portail Azure.

    b. Cliquez sur **Suivant** .

1. Dans la section **Metadata import** (Importation des métadonnées), procédez comme suit : 

    ![Capture d’écran montrant la section « Metadata import » avec l’option « Metadata file on my computer » sélectionnée.](./media/kantegassoforconfluence-tutorial/addon7.png)

    a. Sélectionnez **Metadata file on my computer** (Fichier de métadonnées sur mon ordinateur), puis chargez le fichier de métadonnées que vous avez téléchargé à partir du portail Azure.

    b. Cliquez sur **Suivant** .

1. Dans la section **Name and SSO location** (Nom et emplacement de l’authentification unique), procédez comme suit :

    ![Capture d’écran montrant la section « Name and SSO location », avec la zone de texte « Identity provider name » mise en évidence et le bouton « Next » sélectionné.](./media/kantegassoforconfluence-tutorial/addon8.png)

    a. Ajoutez le nom du fournisseur d’identité dans la zone de texte **Identity provider name** (Nom du fournisseur d’identité) (par exemple, Azure AD).

    b. Cliquez sur **Suivant** .

1. Vérifiez le certificat de signature, puis cliquez sur **Suivant** .

    ![Capture d’écran montrant la section « Signature verification » avec le bouton « Next » sélectionné.](./media/kantegassoforconfluence-tutorial/addon9.png)

1. Dans la section **Confluence user accounts** (Comptes d’utilisateur Confluence), procédez comme suit :

    ![Capture d’écran montrant la section « Confluence user accounts » avec l’option « Create users in Confluence’s Internal Directory if needed » et le bouton « Next » sélectionnés.](./media/kantegassoforconfluence-tutorial/addon10.png)

    a. Sélectionnez **Create users in Confluence’s internal Directory if needed** (Créer des utilisateurs dans l’annuaire interne de Confluence si nécessaire) et entrez le nom de groupe approprié pour les utilisateurs (il peut s’agir de plusieurs groupes séparés par des virgules).

    b. Cliquez sur **Suivant** .

1. Cliquez sur **Terminer** .

    ![Capture d’écran de la page « Summary » avec le bouton « Finish » sélectionné.](./media/kantegassoforconfluence-tutorial/addon11.png)

1. Dans la section **Known domains for Azure AD** (Domaines connus pour Azure AD), procédez comme suit : 

    ![Capture d’écran montrant la page « Known domains for Azure AD » avec la zone de texte « Known domains » mise en évidence et le bouton « Save » sélectionné.](./media/kantegassoforconfluence-tutorial/addon12.png)

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

Dans cette section, vous permettez à Britta Simon d’utiliser l’authentification unique Azure en lui accordant l’accès à Kantega SSO pour Confluence.

1. Dans le Portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Kantega SSO pour Confluence** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Kantega SSO pour Confluence** .

    ![Le lien Kantega SSO pour Confluence dans la liste des applications.](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-kantega-sso-for-confluence-test-user"></a>Création d’un utilisateur de test Kantega SSO pour Confluence

Pour permettre aux utilisateurs Azure AD de se connecter à Confluence, vous devez les configurer dans Confluence. Dans le cas de Kantega SSO pour Confluence, cet approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise Kantega SSO pour Confluence en tant qu’administrateur.

1. Pointez sur la roue dentée, puis cliquez sur **Gestion des utilisateurs** .

    ![Capture d’écran montrant l’icône « Roue dentée » et l’option « User management » sélectionnées.](./media/kantegassoforconfluence-tutorial/user1.png)

1. Dans la section Utilisateurs, cliquez sur l’onglet **Add users** (Ajouter des utilisateurs). Dans la page de boîte de dialogue **Add a User** (Ajouter un utilisateur), procédez comme suit :

    ![Ajouter un employé](./media/kantegassoforconfluence-tutorial/user2.png)

    a. Dans la zone de texte **Username** (Nom d’utilisateur), tapez l’e-mail d’un utilisateur, par exemple, Brittasimon@contoso.com.

    b. Dans la zone de texte **Full Name** (Nom complet), tapez le nom complet d’un utilisateur, par exemple, Britta Simon.

    c. Dans la zone de texte **Email** (E-mail), tapez l’adresse e-mail d’un utilisateur, par exemple, Brittasimon@contoso.com.

    d. Dans la zone de texte **Password** (Mot de passe), tapez le mot de passe de l’utilisateur.

    e. Cliquez sur **Confirm Password** (Confirmer le mot de passe), puis entrez de nouveau le mot de passe.

    f. Cliquez sur le bouton **Ajouter** .

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Kantega SSO pour Confluence dans le volet d’accès, vous devriez normalement être connecté automatiquement à l’application Kantega SSO pour Confluence pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)