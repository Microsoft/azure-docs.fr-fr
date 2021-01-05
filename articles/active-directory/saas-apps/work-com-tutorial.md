---
title: "Tutoriel : Intégration d'Azure Active Directory à Work.com | Microsoft Docs"
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Work.com.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/03/2019
ms.author: jeedes
ms.openlocfilehash: a77b6472ea91a6ba4b6e31d1b7739aae236a1783
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92636151"
---
# <a name="tutorial-azure-active-directory-integration-with-workcom"></a>Tutoriel : Intégration d'Azure Active Directory à Work.com

Dans ce didacticiel, vous allez apprendre à intégrer Work.com dans Azure Active Directory (Azure AD).
L’intégration de Work.com dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Work.com.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à Work.com (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Work.com, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Un abonnement Work.com pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Work.com prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-workcom-from-the-gallery"></a>Ajout de Work.com à partir de la galerie

Pour configurer l’intégration de Work.com à Azure AD, vous devez ajouter Work.com à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Work.com à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Work.com** , sélectionnez **Work.com** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Work.com dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD auprès de Work.com avec un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre l’utilisateur Azure AD et l’utilisateur Work.com associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Work.com, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Work.com](#configure-workcom-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Work.com](#create-workcom-test-user)** pour avoir un équivalent de Britta Simon dans Work.com lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

>[!NOTE]
>Pour configurer l’authentification unique, vous devez encore configurer un nom de domaine personnalisé Work.com. Vous devez définir au moins un nom de domaine, le tester, puis le déployer dans l’ensemble de votre entreprise.

Pour configurer l’authentification unique Azure AD auprès de Work.com, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Work.com** , sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Work.com](common/sp-signonurl.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `http://<companyname>.my.salesforce.com`

    > [!NOTE]
    > Cette valeur n’est pas la valeur réelle. Mettez à jour la valeur avec l’URL de connexion réelle. Pour obtenir cette valeur, contactez l’[équipe du support technique Work.com](https://help.salesforce.com/articleView?id=000159855&type=3). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer Work.com** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-workcom-single-sign-on"></a>Configurer l’authentification unique Work.com

1. Connectez-vous à votre locataire Work.com en tant qu’administrateur.

2. Accédez à **Setup**.
   
    ![Capture d’écran montrant l’élément Setup sélectionné à partir du menu utilisateur.](./media/work-com-tutorial/ic794108.png "Programme d’installation")

3. Dans le volet de navigation gauche, dans la section **Administer** , cliquez sur **Domain Management** pour développer la section associée, puis cliquez sur **My Domain** pour ouvrir la page **My Domain**. 
   
    ![Capture d’écran montrant My Domain sélectionné sous Domain Management dans le volet Administer.](./media/work-com-tutorial/ic767825.png "My Domain")

4. Pour vérifier que votre domaine a été configuré correctement, assurez-vous qu’il figure dans **Step 4 Deployed to Users** , puis passez en revue **My Domain Settings**.
   
    ![Domaine déployé pour l’utilisateur](./media/work-com-tutorial/ic784377.png "Domaine déployé pour l’utilisateur")

5. Connectez-vous à votre locataire Work.com.

6. Accédez à **Setup**.
    
    ![Capture d’écran montrant l’élément Setup sélectionné à partir du menu utilisateur.](./media/work-com-tutorial/ic794108.png "Programme d’installation")

7. Développez le menu **Security Controls** , puis cliquez sur **Single Sign-On Settings**.
    
    ![Paramètres d’authentification unique](./media/work-com-tutorial/ic794113.png "Paramètres d’authentification unique")

8. Dans la page **Single Sign on Settings** , procédez comme suit :
    
    ![SAML activé](./media/work-com-tutorial/ic781026.png "SAML Enabled")
    
    a. Sélectionnez **SAML Enabled**.
    
    b. Cliquez sur **Nouveau**.

9. Dans la section **SAML Single Sign-On Settings** , procédez comme suit :
    
    ![Paramètre d’authentification unique SAML](./media/work-com-tutorial/ic794114.png "Paramètre d’authentification unique SAML")
    
    a. Dans la zone de texte **Name** , tapez le nom de votre configuration.  
       
    > [!NOTE]
    > Le fait d’entrer une valeur pour **Name** renseigne automatiquement la zone de texte **API Name**.
    
    b. Dans la zone de texte **Émetteur** , collez la valeur de l’ **identificateur Azure AD** que vous avez copiée à partir du portail Azure.
    
    c. Pour charger le certificat téléchargé à partir du portail Azure, cliquez sur **Parcourir**.
    
    d. Dans la zone de texte **ID d’entité** , tapez `https://salesforce-work.com`.
    
    e. Pour **SAML Identity Type (Type d’identité SAML)** , sélectionnez **Assertion contains the Federation ID from the User object (L’assertion contient l’ID de fédération de l’objet utilisateur)** .
    
    f. Pour **SAML Identity Location** , sélectionnez **Identity is in the NameIdentfier element of the Subject statement**.
    
    g. Dans la zone de texte **URL de connexion du fournisseur d’identité** , collez la valeur de l’ **URL de connexion** que vous avez copiée dans le portail Azure.

    h. Dans la zone de texte **URL de déconnexion du fournisseur d’identité** , collez la valeur de l’ **URL de déconnexion** que vous avez copiée dans le portail Azure.
    
    i. Pour **Service Provider Initiated Request Binding** , sélectionnez **HTTP POST**.
    
    j. Cliquez sur **Enregistrer**.

10. Dans le volet de navigation gauche du portail classique Work.com, cliquez sur **Domain Management** pour développer la section associée, puis sur **My Domain** pour ouvrir la page **My Domain**. 
    
    ![Capture d’écran montrant My Domain sélectionné à partir de Domain Management.](./media/work-com-tutorial/ic794115.png "My Domain")

11. Dans la section **Login Page Branding** de la page **My Domain** , cliquez sur **Edit**.
    
    ![Capture d’écran montrant la section Login Page Branding où vous pouvez sélectionner Edit.](./media/work-com-tutorial/ic767826.png "Personnalisation de la page de connexion")

12. Le nom des  **SAML SSO Settings** s’affiche dans la section **Authentication Service** de la page **Login Page Branding**. Sélectionnez-le, puis cliquez sur **Save**.
    
    ![Capture d’écran montrant la page Login Page Branding dans laquelle vous pouvez sélectionner le nom de votre paramètre, en l’occurrence P P E.](./media/work-com-tutorial/ic784366.png "Personnalisation de la page de connexion")

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon**.
  
    b. Dans le champ **Nom d’utilisateur** , tapez `brittasimon@yourcompanydomain.extension`. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Work.com.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Work.com**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Work.com**.

    ![Lien Work.com dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

### <a name="create-workcom-test-user"></a>Créer un utilisateur de test Work.com

Pour que les utilisateurs d’Azure AD puissent se connecter, leur accès doit être approvisionné dans Work.com. Dans le cas de Work.com, l’approvisionnement est une tâche manuelle.

### <a name="to-configure-user-provisioning-perform-the-following-steps"></a>Pour configurer l'approvisionnement des utilisateurs, procédez comme suit :

1. Connectez-vous à votre site d’entreprise Work.com en tant qu’administrateur.

2. Accédez à **Setup**.
   
    ![Capture d’écran montrant l’élément Setup sélectionné à partir du menu utilisateur.](./media/work-com-tutorial/ic794108.png "Programme d’installation")

3. Accédez à **Manage Users \> Users**.
   
    ![Gérer les utilisateurs](./media/work-com-tutorial/IC784369.png "Manage Users")

4. Cliquez sur **New User**.
   
    ![Tous les utilisateurs](./media/work-com-tutorial/IC794117.png "Tous les utilisateurs")

5. Dans la section Modification de l’utilisateur, procédez comme suit pour les attributs d’un compte Azure AD valide que vous souhaitez configurer dans les zones de texte correspondantes :
   
    ![Modification de l’utilisateur](./media/work-com-tutorial/ic794118.png "Modification de l’utilisateur")
   
    a. Dans la zone de texte **Prénom** , entrez le **prénom** de l’utilisateur **Britta**.
    
    b. Dans la zone de texte **Nom** , entrez le **nom** de l’utilisateur **Simon**.
    
    c. Dans la zone de texte **Alias** , entrez l’ **alias** de l’utilisateur **BrittaS**.
    
    d. Dans la zone de texte **E-mail** , tapez l’ **adresse e-mail** de l’utilisateur, Brittasimon@contoso.com.
    
    e. Dans la zone de texte **Nom d’utilisateur** , entrez le nom de l’utilisateur, par exemple Brittasimon@contoso.com.
    
    f. Dans la zone de texte **Surnom** , entrez un **surnom** , par exemple **Simon**.
    
    g. Sélectionnez **Rôle** , **Licence utilisateur** et **Profil**.
    
    h. Cliquez sur **Enregistrer**.  
      
    > [!NOTE]
    > Le titulaire du compte Azure AD reçoit alors un message électronique contenant un lien pour confirmer le compte avant qu’il ne soit activé.
    > 

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Work.com dans le volet d’accès doit vous connecter automatiquement à l’application Work.com pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)