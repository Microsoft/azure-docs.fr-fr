---
title: 'Tutoriel : Intégration d’Azure Active Directory à OpsGenie | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et OpsGenie.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/19/2020
ms.author: jeedes
ms.openlocfilehash: a71c29032730e0724213f9f5ef17f7336c188741
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92516679"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-opsgenie"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à OpsGenie

Dans ce tutoriel, vous allez découvrir comment intégrer OpsGenie à Azure Active Directory (Azure AD). Quand vous intégrez OpsGenie à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à OpsGenie.
* Permettre aux utilisateurs de se connecter automatiquement à OpsGenie avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement OpsGenie pour lequel l’authentification unique (SSO) est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* OpsGenie prend en charge l’authentification unique lancée par le **fournisseur d’identité**
* Une fois que vous avez configuré OpsGenie, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-opsgenie-from-the-gallery"></a>Ajout de OpsGenie à partir de la galerie

Pour configurer l’intégration de OpsGenie à Azure AD, vous devez ajouter OpsGenie à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **OpsGenie** dans la zone de recherche.
1. Sélectionnez **OpsGenie** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-opsgenie"></a>Configurer et tester l’authentification unique Azure AD pour OpsGenie

Configurez et testez l’authentification unique Azure AD avec OpsGenie pour un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur OpsGenie associé.

Pour configurer et tester l’authentification unique Azure AD avec OpsGenie, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique OpsGenie](#configure-opsgenie-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test OpsGenie](#create-opsgenie-test-user)** pour obtenir un équivalent de B.Simon dans OpsGenie lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **OpsGenie** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://app.opsginie.com/auth/saml/<UNIQUEID>`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://app.opsginie.com/auth/saml?id=<UNIQUEID>`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeur avec l’identificateur et l’URL de réponse réels : ceci est expliqué plus loin dans ce tutoriel.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

1. Dans la section **Configurer OpsGenie** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à OpsGenie.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **OpsGenie** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-opsgenie-sso"></a>Configurer l’authentification unique OpsGenie

1. Ouvrez une autre instance de navigateur, puis connectez-vous à OpsGenie en tant qu’administrateur.

2. Cliquez sur **Paramètres** , puis cliquez sur l’onglet **Authentification unique** .
   
    ![Authentification unique OpsGenie](./media/opsgenie-tutorial/tutorial-opsgenie-06.png)

3. Pour activer l’authentification unique, sélectionnez **Activé** .
   
    ![Capture d’écran montrant la case à cocher « Enabled » sélectionnée.](./media/opsgenie-tutorial/tutorial-opsgenie-07.png) 

4. Dans la section **Fournisseur** cliquez sur l’onglet **Azure Active Directory** .
   
    ![Capture d’écran montrant la section « Provider » avec l’onglet « Azure Active Directory » sélectionné.](./media/opsgenie-tutorial/tutorial-opsgenie-08.png) 

5. Sur la page de boîte de dialogue Azure Active Directory, procédez comme suit :
   
    ![Capture d’écran montrant la section « Single sign-on » avec le bouton bascule « Enable single sign-on » ainsi que les options « SAML 2.0 Endpoint » et « Metadata URL » sélectionnés.](./media/opsgenie-tutorial/tutorial-opsgenie-09.png)
    
    a. Copiez la valeur de **URI ID d’application** , puis collez-la dans la zone de texte **Identificateur (ID d’entité)** de la section **Configuration SAML de base** du portail Azure.

    a. Copiez la valeur de **URL de réponse** , puis collez-la dans la zone de texte **URL de réponse** de la section **Configuration SAML de base** du portail Azure.

    a. Dans la zone de texte **Point de terminaison SAML 2.0** , collez la valeur de l’ **URL de connexion** que vous avez copiée à partir du Portail Azure.
    
    b. Dans la zone de texte **URL des métadonnées** , collez la valeur **URL des métadonnées de fédération de l’application** que vous avez copiée sur le portail Azure.
    
    c. Pour activer l’authentification unique, sélectionnez **Activer l’authentification unique** .

    d. Cliquez sur **Appliquer les paramètres de l’authentification unique** .

### <a name="create-opsgenie-test-user"></a>Créer un utilisateur de test OpsGenie

L’objectif de cette section est de créer un utilisateur appelé B.Simon dans OpsGenie. 

1. Dans une fenêtre de navigateur web, connectez-vous à votre client OpsGenie en tant qu’administrateur.

2. Accédez à la liste Utilisateurs en cliquant sur **Utilisateurs** dans le volet gauche.
   
    ![Paramètres OpsGenie](./media/opsgenie-tutorial/tutorial-opsgenie-10.png) 

3. Cliquez sur **Add User** .

4. Dans la boîte de dialogue **Ajouter un utilisateur** , procédez comme suit :
   
    ![Capture d’écran montrant la boîte de dialogue « Add User » avec les zones de texte « Email » et « Full name » mises en évidence, et le bouton « Save » sélectionné.](./media/opsgenie-tutorial/tutorial-opsgenie-11.png)
   
    a. Dans la zone de texte **E-mail** , entrez l’adresse e-mail de B.Simon utilisée comme adresse dans Azure Active Directory.
   
    b. Dans la zone de texte **Nom complet** , tapez **B.Simon** .
   
    c. Cliquez sur **Enregistrer** . 

> [!NOTE]
> B.Simon reçoit un e-mail contenant des instructions pour configurer son profil.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette OpsGenie dans le volet d’accès, vous devez être connecté automatiquement à l’application OpsGenie pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer OpsGenie avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)