---
title: 'Tutoriel : Intégration d’Azure Active Directory à Knowledge Anywhere LMS | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Knowledge Anywhere LMS.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/22/2019
ms.author: jeedes
ms.openlocfilehash: eab1304408f896f5c9d736488997c1246ae167f2
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92458980"
---
# <a name="tutorial-integrate-knowledge-anywhere-lms-with-azure-active-directory"></a>Tutoriel : Intégrer Knowledge Anywhere LMS à Azure Active Directory

Dans ce tutoriel, vous allez apprendre à intégrer Knowledge Anywhere LMS à Azure Active Directory (Azure AD). Quand vous intégrez Knowledge Anywhere LMS à Azure AD, vous pouvez :

* Contrôler dans Azure AD l’accès des utilisateurs à Knowledge Anywhere LMS.
* Permettre aux utilisateurs de se connecter automatiquement à Knowledge Anywhere LMS avec leurs comptes Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Knowledge Anywhere LMS pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. Knowledge Anywhere LMS prend en charge l’authentification unique lancée par **le fournisseur d’identité** et le provisionnement d’utilisateurs **juste à temps** .

## <a name="adding-knowledge-anywhere-lms-from-the-gallery"></a>Ajout de Knowledge Anywhere LMS à partir de la galerie

Pour configurer l’intégration de Knowledge Anywhere LMS à Azure AD, vous devez ajouter Knowledge Anywhere LMS à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Knowledge Anywhere LMS** dans la zone de recherche.
1. Sélectionnez **Knowledge Anywhere LMS** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec Knowledge Anywhere LMS à l’aide d’un utilisateur de test appelé **B. Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Knowledge Anywhere LMS associé.

Pour configurer et tester l’authentification unique Azure AD avec Knowledge Anywhere LMS, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer Knowledge Anywhere LMS](#configure-knowledge-anywhere-lms)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Knowledge Anywhere LMS](#create-knowledge-anywhere-lms-test-user)** pour avoir un équivalent de B. Simon dans Knowledge Anywhere LMS lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Knowledge Anywhere LMS** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , effectuez les étapes suivantes :

    1. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<CLIENTNAME>.knowledgeanywhere.com/`

    1. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<CLIENTNAME>.knowledgeanywhere.com/SSO/SAML/Response.aspx?<IDPNAME>`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de réponse réels. La procédure est expliquée plus loin dans le tutoriel.

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<CLIENTNAME>.knowledgeanywhere.com/`

    > [!NOTE]
    > La valeur de l’URL de connexion n’est pas réelle. Mettez à jour la valeur avec l’URL de connexion réelle. Contactez l’[équipe du support client de Knowledge Anywhere LMS](https://knowany.zendesk.com/hc/en-us/articles/360000469034-SAML-2-0-Single-Sign-On-SSO-Set-Up-Guide) pour obtenir cette valeur. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (Base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

   ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Knowledge Anywhere LMS** , copiez les URL appropriées, selon vos besoins.

   ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="configure-knowledge-anywhere-lms"></a>Configurer Knowledge Anywhere LMS

1. Pour automatiser la configuration dans Knowledge Anywhere LMS, vous devez installer l’ **extension de connexion sécurisée à Mes applications** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Knowledge Anywhere LMS** pour être orienté vers l’application Knowledge Anywhere LMS. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Knowledge Anywhere LMS. Cette extension de navigateur configure automatiquement l’application pour vous et automatise les étapes 3 à 7.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement Knowledge Anywhere LMS, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Knowledge Anywhere LMS en tant qu’administrateur, puis effectuez les étapes suivantes :

4. Sélectionnez l’onglet **Site** .

    ![Capture d’écran montrant l’onglet Site.](./media/knowledge-anywhere-lms-tutorial/configure1.png)

5. Sélectionnez l’onglet **Paramètres SAML** .

    ![Capture d’écran montrant la page Knowledge Anywhere avec les paramètres SAML sélectionnés.](./media/knowledge-anywhere-lms-tutorial/configure2.png)

6. Cliquez sur **Ajouter un nouveau** .

    ![Capture d’écran montrant le bouton Add New dans les paramètres du fournisseur de services.](./media/knowledge-anywhere-lms-tutorial/configure3.png)

7. Dans la page **Add/Update SAML Settings** , effectuez les étapes suivantes :

    ![Capture d’écran montrant la page Add/Update SAML Settings dans laquelle vous pouvez apporter les modifications décrites ici.](./media/knowledge-anywhere-lms-tutorial/configure4.png)

    a. Entrez le nom IDP en fonction de votre organisation. Exemple : - `Azure`.

    b. Dans la zone de texte **IDP Entity ID** , collez la valeur de l’ **identificateur Azure AD** que vous avez copiée à partir du portail Azure.

    c. Dans la zone de texte **IDP URL** , collez la valeur de l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    d. Dans le Bloc-notes, ouvrez le fichier de certificat téléchargé à partir du portail Azure, copiez le contenu du certificat, puis collez-le dans la zone de texte **Certificat** .

    e. Dans la zone de texte **URL de déconnexion** , collez la valeur de l’ **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    f. Dans la liste déroulante, sélectionnez **Site principal** pour le **domaine** .

    g. Copiez la valeur de l’ **ID d’entité du fournisseur de services** , puis collez-la dans la zone de texte **Identificateur** de la section **Configuration SAML de base** du portail Azure.

    h. Copiez la valeur de l’ **URL de réponse (ACS) du fournisseur de services** , puis collez-la dans la zone de texte **URL de réponse** de la section **Configuration SAML de base** du portail Azure.

    i. Cliquez sur **Enregistrer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B. Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `BrittaSimon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Knowledge Anywhere LMS.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Knowledge Anywhere LMS** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-knowledge-anywhere-lms-test-user"></a>Créer un utilisateur de test Knowledge Anywhere LMS

Dans cette section, un utilisateur appelé B. Simon est créé dans Knowledge Anywhere LMS. Knowledge Anywhere LMS prend en charge l’attribution d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. Si l’utilisateur souhaité n’existe pas déjà dans Knowledge Anywhere LMS, il est créé après l’authentification.

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Le fait de sélectionner la vignette Knowledge Anywhere LMS dans le volet d’accès doit automatiquement vous connecter à l’application Knowledge Anywhere LMS pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)