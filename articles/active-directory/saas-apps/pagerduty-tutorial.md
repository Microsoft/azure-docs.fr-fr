---
title: 'Tutoriel : Intégration d’Azure Active Directory à Pagerduty | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et PagerDuty.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/14/2019
ms.author: jeedes
ms.openlocfilehash: ec3bcda7953d3b5cb1cb1dae1feebe1da4a9fb04
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92513362"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-pagerduty"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à PagerDuty

Dans ce tutoriel, vous allez apprendre à intégrer PagerDuty à Azure Active Directory (Azure AD). Quand vous intégrez PagerDuty à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à PagerDuty
* Permettre aux utilisateurs de se connecter automatiquement à PagerDuty avec leur compte Azure AD
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement PagerDuty pour lequel l’authentification unique est activée

> [!NOTE]
> Si vous utilisez une authentification multifacteur ou sans mot de passe avec Azure AD, désactivez la valeur AuthnContext dans la demande SAML. Autrement, Azure AD lève l’erreur en cas d’incompatibilité d’ AuthnContext et ne renvoie pas le jeton à l’application.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* PagerDuty prend en charge l’authentification unique lancée par le **fournisseur de services**
* Après avoir configuré PagerDuty, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-pagerduty-from-the-gallery"></a>Ajout de PagerDuty à partir de la galerie

Pour configurer l’intégration de PagerDuty à Azure AD, vous devez ajouter PagerDuty, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **PagerDuty** dans la zone de recherche.
1. Sélectionnez **PagerDuty** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-pagerduty"></a>Configurer et tester l’authentification unique Azure AD pour PagerDuty

Configurez et testez l’authentification unique Azure AD avec PagerDuty à l’aide d’un utilisateur de test nommé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur PagerDuty associé.

Pour configurer et tester l’authentification unique Azure AD avec PagerDuty, suivez les modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique PagerDuty](#configure-pagerduty-sso)** pour configurer les paramètres de l’authentification unique côté application
    1. **[Créer un utilisateur de test PagerDuty](#create-pagerduty-test-user)** pour avoir un équivalent de B.Simon dans PagerDuty lié à la représentation Azure AD de l’utilisateur
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **PagerDuty** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<tenant-name>.pagerduty.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<tenant-name>.pagerduty.com`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<tenant-name>.pagerduty.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique PagerDuty](https://www.pagerduty.com/support/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer PagerDuty** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à PagerDuty.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **PagerDuty** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-pagerduty-sso"></a>Configurer l’authentification unique PagerDuty

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise PagerDuty en tant qu’administrateur.

2. Dans le menu situé en haut, cliquez sur **Account Settings** .

    ![Paramètres du compte](./media/pagerduty-tutorial/ic778535.png "Account Settings")

3. Cliquez sur **Single Sign-on** .

    ![Authentification unique](./media/pagerduty-tutorial/ic778536.png "Authentification unique")

4. Dans la page **Enable Single Sign-on (SSO)** , effectuez les étapes suivantes :

    ![Activer l’authentification unique](./media/pagerduty-tutorial/ic778537.png "Activer l’authentification unique")

    a. Ouvrez dans le Bloc-notes votre certificat codé en base 64 téléchargé à partir du portail Azure, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **X.509 Certificate** .
  
    b. Dans la zone de texte **URL de connexion** , collez l’ **URL de connexion** que vous avez copiée dans le portail Azure.
  
    c. Dans la zone de texte **URL de déconnexion** , collez l’ **URL de déconnexion** que vous avez copiée dans le portail Azure.

    d. Sélectionnez **Allow username/password login** (Autoriser la connexion par nom d’utilisateur/mot de passe).

    e. Cochez la case **Require EXACT authentication context comparison** (Requérir la comparaison EXACTE des contextes d’authentification).

    f. Cliquez sur **Enregistrer les modifications** .

### <a name="create-pagerduty-test-user"></a>Créer un utilisateur de test PagerDuty

Pour permettre aux utilisateurs Azure AD de se connecter à PagerDuty, vous devez les provisionner dans PagerDuty. Dans le cas de Pagerduty, l’approvisionnement est une tâche manuelle.

> [!NOTE]
> Vous pouvez utiliser n’importe quel outil ou API de création de compte utilisateur, fourni par Pagerduty, pour approvisionner des comptes d’utilisateur Azure Active Directory.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre locataire **PagerDuty** .

2. Dans le menu situé en haut, cliquez sur **Utilisateurs** .

3. Cliquez sur **Add Users** .
   
    ![Ajout d’utilisateurs](./media/pagerduty-tutorial/ic778539.png "Ajouter des utilisateurs")

4.  Dans la boîte de dialogue **Invite your team** , effectuez les étapes suivantes :
   
    ![Inviter votre équipe](./media/pagerduty-tutorial/ic778540.png "Inviter dans votre équipe")

    a. Tapez le **prénom et le nom** d’un utilisateur, par exemple **B.Simon** . 
   
    b. Entrez l’adresse **E-mail** de l’utilisateur, par exemple **b.simon\@contoso.com** .
   
    c. Cliquez sur **Add** , puis sur **Send Invites** .
   
    > [!NOTE]
    > Tous les utilisateurs recevront une invitation pour créer un compte PagerDuty.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette PagerDuty dans le volet d’accès, vous devez être connecté automatiquement à l’application PagerDuty pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer PagerDuty avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger PagerDuty avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)