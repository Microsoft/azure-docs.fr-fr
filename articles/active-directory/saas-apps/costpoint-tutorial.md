---
title: 'Tutoriel : Intégration de Azure Active Directory à Costpoint | Microsoft Docs'
description: Découvrez comment configurer l'authentification unique entre Azure Active Directory et Costpoint.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/06/2019
ms.author: jeedes
ms.openlocfilehash: 16c9d64cc4dd49898245d74108cd6a4f2f0e4660
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92455168"
---
# <a name="tutorial-integrate-costpoint-with-azure-active-directory"></a>Tutoriel : Intégrer Costpoint à Azure Active Directory

Dans ce tutoriel, vous allez apprendre à intégrer Costpoint avec Azure Active Directory (Azure AD). Quand vous intégrez Costpoint à Azure AD, vous pouvez :

* Contrôler qui a accès à Costpoint dans Azure AD.
* Permettre à vos utilisateurs de se connecter automatiquement à Costpoint avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Costpoint pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. Costpoint prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité** .

## <a name="generate-costpoint-metadata"></a>Générer des métadonnées Costpoint

La configuration de l’authentification unique SAML Costpoint est expliquée dans le guide **DeltekCostpoint711Security. pdf** . Téléchargez ce guide à partir du site de support Deltek Costpoint et reportez-vous à la section **SAML Single Sign-on Setup** > **Configure SAML Single Sign-on between Costpoint and Microsoft Azure** (Configuration de l’authentification unique SAML -> Configurer l’authentification unique SAML entre Costpoint et Microsoft Azure). Suivez les instructions et générez un fichier **XML de métadonnées de fédération du fournisseur de services Costpoint** . 

![Capture d’écran montrant « Product Configuration Utility » avec l’onglet « Weblogic - Security » sélectionné.](./media/costpoint-tutorial/config-utility.png)

## <a name="add-costpoint-from-the-gallery"></a>Ajouter Costpoint à partir de la galerie

Pour intégrer Costpoint à Azure AD, commencez par ajouter Costpoint à votre liste d’applications SaaS managées à partir de la galerie dans le portail Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.

1. Dans le volet de navigation gauche, sélectionnez le service **Azure Active Directory** .

   ![Bouton Azure Active Directory](common/select-azuread.png)

1. Sélectionnez **Applications d’entreprise** > **Toutes les applications** .

   ![Panneau Applications d’entreprise](common/enterprise-applications.png)

1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .

   ![Bouton Nouvelle application](common/add-new-app.png)

1. Dans la section **Ajouter à partir de la galerie** , entrez **Costpoint** dans la zone de recherche.

   ![Costpoint dans la liste des résultats](common/search-new-app.png)

1. Dans la liste des résultats, sélectionnez **Costpoint** , puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sgn-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec Costpoint au moyen d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Costpoint associé.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec Costpoint, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
1. **[Configurer Costpoint](#configure-costpoint)** pour configurer les paramètres d’authentification unique SAML côté application.
1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Créer un utilisateur de test Costpoint](#create-a-costpoint-test-user)** pour avoir un équivalent de B. Simon dans Costpoint lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le portail Azure :

1. Dans la page d’intégration de l’application **Costpoint** , sélectionnez **Authentification unique** .

   ![Lien Configurer l’authentification unique](common/select-sso.png)

1. Dans la section **Configuration SAML de base** , si vous disposez du *fichier de métadonnées du fournisseur de services* , suivez les étapes ci-dessous :

   > [!NOTE]
   > Vous recevez le fichier de métadonnées du fournisseur de services dans [Générer des métadonnées Costpoint](#generate-costpoint-metadata). L’utilisation du fichier est expliquée plus loin dans le tutoriel.
 
   1. Sélectionnez le bouton **Charger le fichier de métadonnées** , puis le fichier **XML de métadonnées de fédération du fournisseur de services Costpoint** généré précédemment par Costpoint, puis le bouton **Ajouter** pour charger le fichier.

      ![Charger le fichier de métadonnées](./media/costpoint-tutorial/upload-metadata.png)
    
   1. Une fois le fichier de métadonnées correctement chargé, les valeurs **Identificateur** et **URL de réponse** sont renseignées automatiquement dans la section Costpoint.

      > [!NOTE]
      > Si les valeurs **Identificateur** et **URL de réponse** ne sont pas renseignées automatiquement, entrez-les manuellement en fonction de vos besoins. Vérifiez que les valeurs **Identificateur (ID d’entité)** et **URL de réponse (URL Assertion Consumer Service)** sont correctement définies et que l’ **URL ACS** est une URL Costpoint valide qui se termine par **/LoginServlet.cps** .

   1. Sélectionnez **Définir des URL supplémentaires** . Pour **État de relais** , entrez une valeur en utilisant le modèle suivant : `system=[your system]` (par exemple, **system=DELTEKCP** ).

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , sélectionnez l’icône **Copier** pour copier l’ **URL des métadonnées de fédération d’application** et enregistrez-la dans le Bloc-notes.

   ![Certificat de signature SAML](common/copy-metadataurl.png)

### <a name="configure-costpoint"></a>Configurer Costpoint

1. Revenez à l’utilitaire de configuration de Costpoint. Dans la zone de texte **XML de métadonnées de fédération de fournisseur d’identité** , collez le contenu du fichier *URL des métadonnées de fédération d’application* . 

   ![Utilitaire de configuration de Costpoint](./media/costpoint-tutorial/config-utility-idp.png)

1. Suivez les instructions du guide **DeltekCostpoint711Security.pdf** pour terminer la configuration SAML de Costpoint.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé B.Simon dans le portail Azure.

1. Dans le portail Azure, dans le volet gauche, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs** .

   ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

1. Sélectionnez **Nouvel utilisateur** .

   ![Bouton Nouvel utilisateur](common/new-user.png)

1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :

   ![Boîte de dialogue Utilisateur](common/user-properties.png)

   1. Dans le champ **Nom** , entrez **B.Simon** .
   
   1. Dans le champ **Nom de l’utilisateur** , entrez `b.simon\@yourcompanydomain.extension` (par exemple, B.Simon@contoso.com).
   
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   
   1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Costpoint.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications** .

1. Dans la liste des applications, sélectionnez **Costpoint** .

1. Dans la section **Gérer** de la page Vue d’ensemble de l’ application, sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** . Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Utilisateurs et groupes** .

   ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , dans la liste **Utilisateurs** , sélectionnez **B.Simon** . Ensuite, choisissez **Sélectionner** .

1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis choisissez **Sélectionner** .

1. Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Affecter** .

### <a name="create-a-costpoint-test-user"></a>Créer un utilisateur de test Costpoint

Dans cette section, vous créez un utilisateur dans Costpoint. Partez du principe que l’ID d’utilisateur est **B.SIMON** et que le nom de l’utilisateur est **B.Simon** . Collaborez avec l’[équipe de support technique Costpoint](https://www.deltek.com/about/contact-us) pour ajouter l’utilisateur dans la plateforme Costpoint. L’utilisateur doit être créé et activé pour pouvoir utiliser l’authentification unique.

Une fois l’utilisateur créé, la sélection de la **Méthode d’authentification** de l’utilisateur doit être **Active Directory** , la case **Authentification unique SAML** doit être cochée et le nom d’utilisateur Azure Active Directory doit être **Active Directory ou ID de certificat** (situation présentée dans la capture d’écran suivante).

![Utilisateur Costpoint](./media/costpoint-tutorial/costpoint-user.png)

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Quand vous sélectionnez la vignette Costpoint dans le volet d’accès, vous devez être connecté automatiquement à l’application Costpoint, car vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels pour intégrer des applications SaaS à Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)