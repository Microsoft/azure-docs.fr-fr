---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à CloudPassage | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et CloudPassage.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/22/2019
ms.author: jeedes
ms.openlocfilehash: b720f7e49fc0679de5c3f430122bab05d5b706f8
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92455752"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cloudpassage"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à CloudPassage

Dans ce tutoriel, vous allez découvrir comment intégrer CloudPassage à Azure Active Directory (Azure AD). Quand vous intégrez CloudPassage à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à CloudPassage.
* Permettre à vos utilisateurs de se connecter automatiquement à CloudPassage avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement CloudPassage pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* CloudPassage prend en charge l’authentification unique lancée par le **fournisseur de services**

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-cloudpassage-from-the-gallery"></a>Ajout de CloudPassage à partir de la galerie

Pour configurer l’intégration de CloudPassage avec Azure AD, vous devez ajouter CloudPassage, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **CloudPassage** dans la zone de recherche.
1. Sélectionnez **CloudPassage** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-cloudpassage"></a>Configurer et tester l’authentification unique Azure AD pour CloudPassage

Configurez et testez l’authentification unique Azure AD avec CloudPassage pour un utilisateur de test nommé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans CloudPassage.

Pour configurer et tester l’authentification unique Azure AD avec CloudPassage, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique CloudPassage](#configure-cloudpassage-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test CloudPassage](#create-cloudpassage-test-user)** pour avoir un équivalent de B.Simon dans CloudPassage qui soit lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **CloudPassage** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

     a. Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://portal.cloudpassage.com/saml/init/accountid`.

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://portal.cloudpassage.com/saml/consume/accountid`. Vous pouvez obtenir la valeur de cet attribut en cliquant sur **SSO Setup documentation** dans la section **Single Sign-on Settings** de votre portail CloudPassage.

    ![Capture d’écran montrant le portail CloudPassage avec le lien SSO Setup documentation activé.](./media/cloudpassage-tutorial/tutorial_cloudpassage_05.png)

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de réponse et l’URL de connexion réelles. Pour obtenir ces valeurs, contactez l’[équipe de support technique de CloudPassage](https://www.cloudpassage.com/company/contact/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. L’application CloudPassage attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jetons SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/edit-attribute.png)

1. En plus de ce qui précède, l’application CloudPassage s’attend à ce que quelques attributs supplémentaires soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source|
    | ---------------| --------------- |
    | firstname |user.givenname |
    | lastname |user.surname |
    | email |user.mail |

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer CloudPassage** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à CloudPassage.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **CloudPassage** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-cloudpassage-sso"></a>Configurer l’authentification unique CloudPassage

1. Dans une autre fenêtre de navigateur Web, connectez-vous au site de votre entreprise CloudPassage en tant qu’administrateur.

1. Dans le menu situé en haut, cliquez sur **Settings** puis sur **Site Administration** . 
   
    ![Capture d’écran montrant le site CloudPassage avec Site Administration sélectionné.][12]

1. Cliquez sur l’onglet **Authentication Settings** . 
   
    ![Capture d’écran montrant le site CloudPassage avec l’onglet Authentication Settings sélectionné.][13]

1. Dans la section **Single Sign-On Settings** , procédez comme suit : 
   
    ![Capture d’écran montrant la section Single Sign-on Settings, où vous pouvez entrer les informations à cette étape.][14]

    a. Cochez la case **Activer l’authentification unique (SSO) (Documentation de configuration de l’authentification unique)** .
    
    b. Collez la valeur **Identificateur Azure AD** dans la zone de texte **SAML issuer URL** (URL de l’émetteur SAML).
  
    c. Collez la valeur **URL de connexion** dans la zone de texte **SAML endpoint URL** (URL du point de terminaison SAML).
  
    d. Collez la valeur **URL de déconnexion** dans la zone de texte **Logout landing page** (Page de déconnexion).
  
    e. Ouvrez votre certificat téléchargé dans le Bloc-notes, copiez son contenu dans le Presse-papiers et collez-le dans la zone de texte **Certificat X 509** .
  
    f. Cliquez sur **Enregistrer** .

### <a name="create-cloudpassage-test-user"></a>Créer un utilisateur de test CloudPassage

L’objectif de cette section est de créer un utilisateur appelé B.Simon dans CloudPassage.

**Pour créer un utilisateur appelé B.Simon dans CloudPassage, effectuez les étapes suivantes :**

1. Connectez-vous à votre site d’entreprise **CloudPassage** en tant qu’administrateur. 

1. Dans la barre d’outils située en haut, cliquez sur **Paramètres** , puis sur **Administration du site** . 
   
    ![Capture d’écran montrant CloudPassage avec Site Administration sélectionné.][22] 

1. Cliquez sur l’onglet **Users** , puis sur **Add a user** . 
   
    ![Capture d’écran montrant l’administration de site CloudPassage avec l’onglet Users sélectionné et l’option permettant d’ajouter un nouvel utilisateur.][23]

1. Dans la section **Add New User** , procédez comme suit : 
   
    ![Capture d’écran montrant la section Add New User, où vous pouvez spécifier des informations sur l’utilisateur.][24]
    
    a. Dans la zone de texte **First Name** , tapez Britta. 
  
    b. Dans la zone de texte **Last Name** , tapez Simon.
  
    c. Dans les zones de texte **Username** , **Email** et **Retype Email** , entrez le nom d’utilisateur de Britta dans Azure AD.
  
    d. En tant que **Type d’accès** , sélectionnez **Activer l’accès portail Halo** .
  
    e. Cliquez sur **Add** .

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette CloudPassage dans le volet d’accès, vous devez être connecté automatiquement à l’application CloudPassage pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer CloudPassage avec Azure AD](https://aad.portal.azure.com/)

<!--Image references-->

[12]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_07.png
[13]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_08.png
[14]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_09.png
[15]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_10.png
[22]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_15.png
[23]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_16.png
[24]: ./media/cloudpassage-tutorial/tutorial_cloudpassage_17.png