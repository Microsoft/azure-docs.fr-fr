---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Moxtra | Microsoft Docs'
description: Découvrez comment configurer l'authentification unique entre Azure Active Directory et Moxtra.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/05/2019
ms.author: jeedes
ms.openlocfilehash: be63ea7ebf905b825ffe73dfc2b37dcdc3bb88fd
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92507514"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-moxtra"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Moxtra

Dans ce tutoriel, vous allez apprendre à intégrer Moxtra à Azure Active Directory (Azure AD). Quand vous intégrez Moxtra à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Moxtra.
* Permettre à vos utilisateurs de se connecter automatiquement à Moxtra avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Moxtra pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Moxtra prend en charge l’authentification unique (SSO) initiée par le **fournisseur de services** .

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-moxtra-from-the-gallery"></a>Ajout de Moxtra à partir de la galerie

Pour configurer l'intégration de Moxtra à Azure AD, vous devez ajouter Moxtra à votre liste d'applications SaaS gérées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Moxtra** dans la zone de recherche.
1. Sélectionnez **Moxtra** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-moxtra"></a>Configurer et tester l’authentification unique Azure AD pour Moxtra

Configurez et testez l’authentification unique Azure AD avec Moxtra à l’aide d’un utilisateur de test nommé **B. Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Moxtra associé.

Pour configurer et tester l’authentification unique Azure AD auprès de Moxtra, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Moxtra](#configure-moxtra-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Moxtra](#create-moxtra-test-user)** pour obtenir un équivalent de B.Simon dans Moxtra lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Moxtra** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    Dans la zone de texte **URL de connexion** , tapez une URL : `https://www.moxtra.com/service/#login`

1. L’application Moxtra attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut. Cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue Attributs d’utilisateur.

    ![image](common/edit-attribute.png)

1. En plus de ce qui précède, l’application Moxtra s’attend à ce que quelques attributs supplémentaires soient passés dans la réponse SAML. Dans la section Revendications des utilisateurs de la boîte de dialogue Attributs utilisateur, effectuez les étapes suivantes pour ajouter le jeton SAML, comme indiqué dans le tableau ci-dessous :

    | Nom | Attribut source|
    | ------------------- | -------------------- |    
    | firstname | user.givenname |
    | lastname | user.surname |
    | idpid    | < Identificateur Azure Active Directory >

    > [!Note]
    > La valeur de l’attribut **idpid** n’est pas réelle. Vous pouvez trouver la valeur réelle dans la section **Configurer Moxtra** à partir de l’étape 8. 

    1. Cliquez sur le bouton **Ajouter une nouvelle revendication** pour ouvrir la boîte de dialogue **Gérer les revendications des utilisateurs** .

    1. Dans la zone de texte **Attribut** , indiquez le nom d’attribut pour cette ligne.

    1. Laissez le champ **Espace de noms** vide.

    1. Sélectionnez Source comme **Attribut** .

    1. Dans la liste **Attribut de la source** , tapez la valeur d’attribut indiquée pour cette ligne.

    1. Cliquez sur **OK**

    1. Cliquez sur **Enregistrer** .

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Moxtra** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Moxtra.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Moxtra** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-moxtra-sso"></a>Configurer l’authentification unique Moxtra

1. Dans une autre fenêtre de navigateur, connectez-vous à votre site d'entreprise Moxtra en tant qu'administrateur.

2. Dans la barre d’outils située à gauche, cliquez sur **Console Administrateur > Authentification unique SAML** , puis sur **Nouveau** .
   
    ![Capture d’écran montrant la page d’authentification unique SAML, avec l’option permettant de créer une authentification unique SAML.](./media/moxtra-tutorial/tutorial_moxtra_06.png) 

3. Sur la page **SAML** , procédez comme suit :
   
    ![Capture d’écran montrant la page SAML dans laquelle vous pouvez indiquer les valeurs décrites.](./media/moxtra-tutorial/tutorial_moxtra_08.png)   
 
    a. Dans la zone de texte **Nom** , entrez le nom de votre configuration (par exemple, *SAML* ). 
  
    b. Dans la zone de texte **IdP Entity ID** (Identificateur d’entité du fournisseur d’identité), collez l’ **Identificateur Azure AD** que vous avez copié dans le portail Azure. 
 
    c. Dans la zone de texte **Login URL** (URL de connexion), collez la valeur **URL de connexion** que vous avez copiée dans le Portail Azure. 
 
    d. Dans la zone de texte **AuthnContextClassRef** , tapez **urn:oasis:names:tc:SAML:2.0:ac:classes:Password** . 
 
    e. Dans la zone de texte **Format NameID** , entrez **urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress** . 
 
    f. Dans le bloc-notes, ouvrez le certificat que vous avez téléchargé depuis le portail Azure, copiez son contenu, puis collez-le dans la zone de texte **Certificat** .    
 
    g. Dans la zone de texte du domaine de messagerie SAML, tapez votre domaine de messagerie SAML.    
  
    >[!NOTE]
    >Pour connaître les étapes de vérification du domaine, cliquez sur le « **i** » ci-dessous.

    h. Cliquez sur **Update** .

### <a name="create-moxtra-test-user"></a>Créer un utilisateur de test Moxtra

L’objectif de cette section est de créer un utilisateur appelé B.Simon dans Moxtra.

**Pour créer un utilisateur appelé B.Simon dans Moxtra, effectuez les étapes suivantes :**

1. Connectez-vous à votre site d’entreprise Moxtra en tant qu’administrateur.

1. Dans la barre d’outils située à gauche, cliquez sur **Console Administrateur > Gestion des utilisateurs** , puis sur **Ajouter un utilisateur** .
   
    ![Capture d’écran montrant la page User Management avec l’élément Users sélectionné.](./media/moxtra-tutorial/tutorial_moxtra_10.png) 

1. Dans la boîte de dialogue **Ajouter un utilisateur** , procédez comme suit :
  
    a. Dans la zone de texte **First Name** , tapez **B** .
  
    b. Dans la zone de texte **Last Name** , tapez **Simon** .
  
    c. Dans la zone de texte **Email** , entrez l’adresse e-mail de B.Simon, telle qu’elle est indiquée dans le portail Azure.
  
    d. Dans la zone de texte **Division** , saisissez **Développement** .
  
    e. Dans la zone de texte **Service** , saisissez **Informatique** .
  
    f. Sélectionnez **Administrateur** .
  
    g. Cliquez sur **Add** .

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Moxtra dans le panneau d’accès doit vous connecter automatiquement à l’application Moxtra pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Moxtra avec Azure AD](https://aad.portal.azure.com/)