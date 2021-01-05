---
title: 'Tutoriel : Intégration d’Azure Active Directory à dmarcian | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et dmarcian.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/01/2019
ms.author: jeedes
ms.openlocfilehash: 4a5ea47352bfbe86fe194821da0b6bf10818caf9
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92454726"
---
# <a name="tutorial-integrate-dmarcian-with-azure-active-directory"></a>Tutoriel : Intégrer dmarcian à Azure Active Directory

Dans ce tutoriel, vous allez apprendre à intégrer dmarcian à Azure Active Directory (Azure AD). Quand vous intégrez dmarcian à Azure AD, vous pouvez :

* Dans Azure AD, contrôler qui a accès à dmarcian
* Permettre à vos utilisateurs de se connecter automatiquement à dmarcian avec leur compte Azure AD
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement dmarcian pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* dmarcian prend en charge l’authentification unique initiée par le **fournisseur de services et le fournisseur d’identité**

## <a name="adding-dmarcian-from-the-gallery"></a>Ajout de dmarcian à partir de la galerie

Pour configurer l’intégration de dmarcian à Azure AD, vous devez ajouter dmarcian à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **dmarcian** dans la zone de recherche.
1. Sélectionnez **dmarcian** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec dmarcian au moyen d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur dmarcian associé.

Pour configurer et tester l’authentification unique Azure AD avec dmarcian, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer dmarcian](#configure-dmarcian-sso)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test dmarcian](#create-dmarcian-test-user)** pour avoir dans dmarcian un équivalent de B. Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **dmarcian** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

4. À la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode initié par **IDP** , suivez les étapes ci-dessous :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant :

    ```http
    https://us.dmarcian.com/sso/saml/<ACCOUNT_ID>/sp.xml
    https://dmarcian-eu.com/sso/saml/<ACCOUNT_ID>/sp.xml
    https://dmarcian-ap.com/sso/saml/<ACCOUNT_ID>/sp.xml
    ```

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant :

    ```http
    https://us.dmarcian.com/login/<ACCOUNT_ID>/handle/
    https://dmarcian-eu.com/login/<ACCOUNT_ID>/handle/
    https://dmarcian-ap.com/login/<ACCOUNT_ID>/handle/
    ```

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL d’authentification** , tapez une URL au format suivant :
    
    ```http
    https://us.dmarcian.com/login/<ACCOUNT_ID>
    https://dmarcian-eu.com/login/<ACCOUNT_ID>
    https://dmarciam-ap.com/login/<ACCOUNT_ID>
    ```
     
    > [!NOTE] 
    > Il ne s’agit pas de valeurs réelles. Vous mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. La procédure est expliquée plus loin dans le didacticiel.

4. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

### <a name="configure-dmarcian-sso"></a>Configurer l’authentification unique dmarcian

1. Afin d’automatiser la configuration dans dmarcian, vous devez installer l’ **extension de navigateur permettant la connexion sécurisée à Mes applications** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer dmarcian** pour être dirigé vers l’application dmarcian. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à dmarcian. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 6.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement dmarcian, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise dmarcian en tant qu’administrateur et effectuez les étapes suivantes :

4. Cliquez sur **Profile** en haut à droite et accédez à **Preferences** .

    ![Les préférences](./media/dmarcian-tutorial/tutorial_dmarcian_pref.png)

5. Faites défiler, puis cliquez sur la section **Authentification unique** et sur **Configurer** .

    ![Authentification unique](./media/dmarcian-tutorial/tutorial_dmarcian_sso.png)

6. Sur la page **Authentification unique SAML** , définissez **État** sur **Activé** et suivez les étapes ci-après :

    ![L’authentification](./media/dmarcian-tutorial/tutorial_dmarcian_auth.png)

    * Sous la section **Add dmarcian to your Identity Provider** (Ajouter dmarcian à votre fournisseur d’identité), cliquez sur **COPIER** pour copier l’ **URL du service consommateur d’assertion** de votre instance et collez-la dans la zone de texte **URL de réponse** dans la section **Configuration SAML de base** du portail Azure.

    * Sous la section **Add dmarcian to your Identity Provider** (Ajouter dmarcian à votre fournisseur d’identité), cliquez sur **COPIER** pour copier l’ **ID d’entité** de votre instance et collez-la dans la zone de texte **Identificateur** dans la section **Configuration SAML de base** du portail Azure.

    * Sous la section **Set up Authentication** (Configurer l’authentification), dans la zone de texte **Identity Provider Metadata** (Métadonnées du fournisseur d’identité), collez **l’URL des métadonnées de fédération d’application** , que vous avez copiée à partir du Portail Azure.

    * Sous la section **Set up Authentication** (Configurer l’authentification), dans la zone de texte **Attribute Statements** (Instructions d’attribut), collez l’URL `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`.

    * Sous la section **Set up Login URL** (Configurer l’URL de connexion), copiez l’ **URL de connexion** de votre instance et collez-la dans la zone de texte **URL de connexion** de la section **Configuration SAML de base** du portail Azure.

        > [!Note]
        > Vous pouvez modifier l’ **URL de connexion** en fonction de votre organisation.

    * Cliquez sur **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en accordant l’accès à dmarcian.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **dmarcian** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-dmarcian-test-user"></a>Créer un utilisateur de test dmarcian

Pour se connecter à dmarcian, les utilisateurs Azure AD doivent être approvisionnés dans dmarcian. Dans dmarcian, l’approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à dmarcian en tant qu’administrateur de la sécurité.

2. Cliquez sur **Profile** en haut à droite et accédez à **Manage Users** (Gérer les utilisateurs).

    ![L’utilisateur](./media/dmarcian-tutorial/tutorial_dmarcian_user.png)

3. Sur le côté droit de la section **SSO Users** (Utilisateurs SSO), cliquez sur **Ajouter un nouvel utilisateur** .

    ![L’utilisateur ajouté](./media/dmarcian-tutorial/tutorial_dmarcian_addnewuser.png)

4. Dans la fenêtre contextuelle **Ajouter un nouvel utilisateur** , procédez comme suit :

    ![Le nouvel utilisateur](./media/dmarcian-tutorial/tutorial_dmarcian_save.png)

    a. Dans la zone de texte **Nouvel e-mail d’utilisateur** , entrez l’e-mail de l’utilisateur, par exemple **brittasimon\@contoso.com** .

    b. Si vous souhaitez accorder des droits d’administrateur à l’utilisateur, sélectionnez **Make User an Admin** (Rendre l’utilisateur administrateur).

    c. Cliquez sur **Add User** .

### <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette dmarcian dans le volet d’accès, vous devez être connecté automatiquement à l’application dmarcian pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)