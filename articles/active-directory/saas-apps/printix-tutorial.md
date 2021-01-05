---
title: 'Tutoriel : Intégration d’Azure Active Directory à Printix | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Printix.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/29/2017
ms.author: jeedes
ms.openlocfilehash: dfde9bbbeb7f6b349ecbdc4c2da605d39a0708da
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94357876"
---
# <a name="tutorial-azure-active-directory-integration-with-printix"></a>Tutoriel : Intégration d’Azure Active Directory à Printix

Dans ce didacticiel, vous allez apprendre à intégrer Printix à Azure Active Directory (Azure AD).

L’intégration de Printix à Azure AD vous offre les avantages suivants :

- Dans Azure AD, vous pouvez contrôler qui a accès à Printix.
- Vous pouvez autoriser les utilisateurs à se connecter automatiquement à Printix (via l’authentification unique) avec leur compte Azure AD.
- Vous pouvez gérer vos comptes à partir d’un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Printix, vous avez besoin des éléments suivants :

- Un abonnement Azure AD
- Un abonnement Printix pour lequel l’authentification unique est activée

> [!NOTE]
> Pour tester les étapes de ce didacticiel, nous déconseillons l’utilisation d’un environnement de production.

Vous devez en outre suivre les recommandations ci-dessous :

- N’utilisez pas votre environnement de production, sauf si cela est nécessaire.
- Si vous n’avez pas d’environnement d’essai Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).

## <a name="scenario-description"></a>Description du scénario
Dans ce didacticiel, vous testez l’authentification unique Azure AD dans un environnement de test. Le scénario décrit dans ce didacticiel se compose des deux sections principales suivantes :

1. Ajout de Printix à partir de la galerie
1. Configuration et test de l’authentification unique Azure AD

## <a name="adding-printix-from-the-gallery"></a>Ajout de Printix à partir de la galerie
Pour configurer l’intégration de Printix à Azure AD, vous devez ajouter Printix à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Printix à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**. 

    ![Active Directory][1]

1. Accédez à **Applications d’entreprise**. Accédez ensuite à **Toutes les applications**.

    ![Capture d’écran montrant le portail Azure avec l’option Applications d’entreprise sélectionnée sous Gérer, avec Toutes les applications sélectionné.][2]
    
1. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Capture d’écran montrant l’option Nouvelle application sélectionnée.][3]

1. Dans la zone de recherche, tapez **Printix**.

    ![Capture d’écran montrant la recherche de Printix dans la boîte de dialogue Ajouter à partir de la galerie.](./media/printix-tutorial/tutorial_printix_search.png)

1. Dans le panneau des résultats, sélectionnez **Printix** , puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Capture d’écran montre l’option Printix sélectionnée.](./media/printix-tutorial/tutorial_printix_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>Configuration et test de l’authentification unique Azure AD
Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Printix avec un utilisateur de test appelé « Britta Simon ».

Pour que l’authentification unique fonctionne, Azure AD doit savoir qui est l’utilisateur Printix équivalent dans Azure AD. En d’autres termes, une relation entre l’utilisateur Azure AD et l’utilisateur Printix associé doit être établie.

Dans Printix, assignez la valeur de **nom d’utilisateur** dans Azure AD comme valeur de **nom d’utilisateur** pour établir la relation.

Pour configurer et tester l’authentification unique Azure AD avec Printix, vous devez suivre les indications des sections suivantes :

1. **[Configuration de l’authentification unique Azure AD](#configuring-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
1. **[Création d’un utilisateur de test Azure AD](#creating-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
1. **[Création d’un utilisateur de test Printix](#creating-a-printix-test-user)** pour obtenir un équivalent de Britta Simon dans Printix lié à la représentation Azure AD associée.
1. **[Affectation de l’utilisateur de test Azure AD](#assigning-the-azure-ad-test-user)** : permet à Britta Simon d’utiliser l’authentification unique Azure AD.
1. **[Testing Single Sign-On](#testing-single-sign-on)** pour vérifier si la configuration fonctionne.

### <a name="configuring-azure-ad-single-sign-on"></a>Configuration de l’authentification unique Azure AD

Dans cette section, vous allez activer l’authentification unique Azure AD dans le portail Azure et configurer l’authentification unique dans votre application Printix.

**Pour configurer l’authentification unique Azure AD avec Printix, procédez comme suit :**

1. Dans le Portail Azure, sur la page d’intégration de l’application **Printix** , cliquez sur **Authentification unique**.

    ![Capture d’écran montrant Authentification unique sélectionné sous Gérer dans le portail Azure.][4]

1. Dans la boîte de dialogue **Authentification unique** , pour le **Mode** , sélectionnez **Authentification basée sur SAML** pour activer l’authentification unique.
 
    ![Capture d’écran montrant le mode Authentification basée sur SAML sélectionné.](./media/printix-tutorial/tutorial_printix_samlbase.png)

1. Dans la section **Domaine et URL Printix** , procédez comme suit :

    ![Capture d’écran montrant la section Domaine et URL Printix où vous pouvez spécifier une URL pour l’authentification.](./media/printix-tutorial/tutorial_printix_url.png)

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<subdomain>.printix.net`

    > [!NOTE] 
    > Cette valeur n’est pas la valeur réelle. Mettez à jour la valeur avec l’URL de connexion réelle. Pour obtenir cette valeur, contactez l’[équipe du support technique Printix](mailto:support@printix.net). 
 
1. Dans la section **Certificat de signature SAML** , cliquez sur **Métadonnées XML** puis enregistrez le fichier de métadonnées sur votre ordinateur.

    ![Capture d’écran montrant le volet Certificat de signature SAML dans lequel vous pouvez télécharger un certificat.](./media/printix-tutorial/tutorial_printix_certificate.png) 

1. Cliquez sur le bouton **Enregistrer** .

    ![Capture d’écran montrant le bouton Save.](./media/printix-tutorial/tutorial_general_400.png)

1. Connectez-vous à votre client Printix en tant qu’administrateur.

1. Dans le menu situé en haut, cliquez sur l’icône dans le coin supérieur droit et sélectionnez **Authentification** (Authentification).
   
    ![Capture d’écran montre l’authentification sélectionnée dans le menu.](./media/printix-tutorial/tutorial_printix_06.png)

1. Sous l’onglet **Setup** (Configuration), sélectionnez **Enable Azure/Office 365 authentication** (Activer l’authentification Azure/Office 365).
   
    ![Capture d’écran montrant la page Printix.net où vous pouvez sélectionner Activer l’authentification Azure/Office 365.](./media/printix-tutorial/tutorial_printix_07.png)

1. Sous l’onglet **Azure** , entrez l’URL des métadonnées de fédération dans la zone de texte **Federation metadata document** (Document de métadonnées de fédération). 

    Envoyez le fichier XML de métadonnées que vous avez téléchargé sur Azure AD à [l’équipe de support Printix](mailto:support@printix.net). Elle charge alors le fichier XML et vous fournit une URL des métadonnées de fédération.
   
    ![Capture d’écran montrant la page Printix.net où vous pouvez spécifier un document de métadonnées de fédération.](./media/printix-tutorial/tutorial_printix_08.png)
   
1. Cliquez sur le bouton **Test** , puis sur le bouton **OK** si le test a réussi.
   
     La page Azure Active Directory s’affiche après que vous avez cliqué sur le bouton **Test**. Le message « Test réussi » signifie qu’après avoir entré les informations d’identification de votre compte test Azure, le message « Paramètres testés correctement » s’affiche. Cliquez alors sur le bouton **OK**.
   
    ![Capture d’écran affichant les résultats du test.](./media/printix-tutorial/tutorial_printix_09.png)

1. Cliquez sur le bouton **Save** (Enregistrer) dans la page **Authentication** (Authentification).


> [!TIP]
> Vous pouvez maintenant lire une version concise de ces instructions dans le [portail Azure](https://portal.azure.com), pendant que vous configurez l’application.  Après avoir ajouté cette application à partir de la section **Active Directory > Applications d’entreprise** , cliquez simplement sur l’onglet **Authentification unique** et accédez à la documentation incorporée par le biais de la section **Configuration** en bas. Pour en savoir plus sur la fonctionnalité de documentation incorporée, accédez à : [Documentation incorporée Azure AD]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>Création d’un utilisateur de test Azure AD
L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

![Créer un utilisateur Azure AD][100]

**Pour créer un utilisateur de test dans Azure AD, procédez comme suit :**

1. Dans le panneau de navigation gauche du **portail Azure** , cliquez sur l’icône **Azure Active Directory**.

    ![Capture d’écran affichant le nom et le nom d’utilisateur à créer.](./media/printix-tutorial/create_aaduser_01.png) 

1. Pour afficher la liste des utilisateurs, accédez à **Utilisateurs et groupes** , puis cliquez sur **Tous les utilisateurs**.
    
    ![Capture d’écran montrant l’icône Azure AD dans le portail Azure.](./media/printix-tutorial/create_aaduser_02.png) 

1. Pour ouvrir la boîte de dialogue **Utilisateur** , cliquez sur **Ajouter** en haut de la boîte de dialogue.
 
    ![Capture d’écran montrant Utilisateurs et groupes sélectionné dans le menu Gérer, avec Tous les utilisateurs sélectionné.](./media/printix-tutorial/create_aaduser_03.png) 

1. Dans la boîte de dialogue **Utilisateur** , procédez comme suit :
 
    ![Capture d’écran montrant la boîte de dialogue Utilisateur où vous pouvez entrer les valeurs décrites.](./media/printix-tutorial/create_aaduser_04.png) 

    a. Dans la zone de texte **Nom** , entrez **BrittaSimon**.

    b. Dans la zone de texte **Nom d’utilisateur** , tapez **l’adresse e-mail** de Britta Simon.

    c. Sélectionnez **Afficher le mot de passe** et notez la valeur du **mot de passe**.

    d. Cliquez sur **Créer**.
 
### <a name="creating-a-printix-test-user"></a>Création d’un utilisateur de test Printix

L’objectif de cette section est de créer un utilisateur appelé Britta Simon dans Printix. Printix prend en charge l’approvisionnement juste-à-temps, option activée par défaut.

Vous n’avez aucune opération à effectuer dans cette section. Un nouvel utilisateur est créé lors d’une tentative d’accès à Printix, s’il n’existe pas déjà. 

> [!NOTE]
> Si vous devez créer un utilisateur manuellement, contactez [l’équipe de support Printix](mailto:support@printix.net).
> 

### <a name="assigning-the-azure-ad-test-user"></a>Affectation de l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Printix.

![Capture d’écran montrant un utilisateur disposant d’un accès par défaut.][200] 

**Pour affecter Britta Simon à Printix, procédez comme suit :**

1. Dans le portail Azure, ouvrez la vue des applications, accédez à la vue des répertoires, accédez à **Applications d’entreprise** , puis cliquez sur **Toutes les applications**.

    ![Capture d’écran montrant l’option Applications d’entreprise sélectionnée sous Gérer, avec Toutes les applications sélectionné.][201] 

1. Dans la liste des applications, sélectionnez **Printix**.

    ![Capture d’écran affichant la liste des applications où vous pouvez sélectionner Printix.](./media/printix-tutorial/tutorial_printix_app.png) 

1. Dans le menu de gauche, cliquez sur **Utilisateurs et groupes**.

    ![Capture d’écran montrant Utilisateurs et groupes sélectionné dans le menu Gérer.][202] 

1. Cliquez sur le bouton **Ajouter**. Ensuite, sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une affectation**.

    ![Capture d’écran affichant le bouton Ajouter et la page Ajouter une attribution où vous pouvez sélectionner des utilisateurs et des groupes.][203]

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste des utilisateurs.

1. Cliquez sur le bouton **Sélectionner** dans la boîte de dialogue **Utilisateurs et groupes**.

1. Cliquez sur le bouton **Affecter** dans la boîte de dialogue **Ajouter une affectation**.
    
### <a name="testing-single-sign-on"></a>Test de l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la vignette Printix dans le volet d’accès, vous devez être connecté automatiquement à votre application Printix.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](tutorial-list.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/printix-tutorial/tutorial_general_01.png
[2]: ./media/printix-tutorial/tutorial_general_02.png
[3]: ./media/printix-tutorial/tutorial_general_03.png
[4]: ./media/printix-tutorial/tutorial_general_04.png

[100]: ./media/printix-tutorial/tutorial_general_100.png

[200]: ./media/printix-tutorial/tutorial_general_200.png
[201]: ./media/printix-tutorial/tutorial_general_201.png
[202]: ./media/printix-tutorial/tutorial_general_202.png
[203]: ./media/printix-tutorial/tutorial_general_203.png

