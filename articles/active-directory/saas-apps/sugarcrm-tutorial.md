---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec Sugar CRM | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Sugar CRM.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/17/2019
ms.author: jeedes
ms.openlocfilehash: c875c89d5b565e95fccaceebdccad82da75b238e
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92521915"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-sugar-crm"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec Sugar CRM

Dans ce tutoriel, vous allez découvrir comment intégrer Sugar CRM avec Azure Active Directory (Azure AD). Quand vous intégrez Sugar CRM avec Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Sugar CRM.
* Permettre à vos utilisateurs de se connecter automatiquement à Sugar CRM avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Sugar CRM pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Sugar CRM prend en charge l’authentification unique lancée par le **fournisseur de services**

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-sugar-crm-from-the-gallery"></a>Ajout de Sugar CRM à partir de la galerie

Pour configurer l’intégration de Sugar CRM à Azure AD, vous devez ajouter Sugar CRM à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **Sugar CRM** dans la zone de recherche.
1. Sélectionnez **Sugar CRM** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-sugar-crm"></a>Configurer et tester l’authentification unique Azure AD pour Sugar CRM

Configurez et testez l’authentification unique Azure AD avec Sugar CRM pour un utilisateur de test appelé **B. Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans Sugar CRM.

Pour configurer et tester l’authentification unique Azure AD avec Sugar CRM, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Sugar CRM](#configure-sugar-crm-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Sugar CRM](#create-sugar-crm-test-user)** pour avoir un équivalent de B. Simon dans Sugar CRM lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Sur le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Sugar CRM** , recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL d’authentification** , tapez une URL au format suivant :

    - `https://<companyname>.sugarondemand.com`
    - `https://<companyname>.trial.sugarcrm`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant :

    - `https://<companyname>.sugarondemand.com/<companyname>`
    - `https://<companyname>.trial.sugarcrm.com/<companyname>`
    - `https://<companyname>.trial.sugarcrm.eu/<companyname>`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de réponse et l’URL de connexion réelles. Pour obtenir ces valeurs, contactez l’[équipe de support du client Sugar CRM](https://support.sugarcrm.com/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Sugar CRM** , copiez la ou les URL appropriées, selon vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Sugar CRM.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Sugar CRM**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

## <a name="configure-sugar-crm-sso"></a>Configurer l’authentification unique Sugar CRM

1. Dans une autre fenêtre de navigateur web, connectez-vous au site de votre entreprise Sugar CRM en tant qu’administrateur.

1. Accédez à **Admin**.

    ![Administrateur](./media/sugarcrm-tutorial/ic795888.png "Admin")

1. Dans la section **Administration** , cliquez sur **Password Management (Gestion des mots de passe)** .

    ![Capture d’écran montrant la section Administration dans laquelle vous pouvez sélectionner Password Management](./media/sugarcrm-tutorial/ic795889.png "Administration")

1. Sélectionnez **Enable SAML Authentication**.

    ![Capture d’écran montrant l’option qui permet de sélectionner l’authentification SAML](./media/sugarcrm-tutorial/ic795890.png "Administration")

1. Dans la section **SAML Authentication** , procédez comme suit :

    ![Authentification SAML](./media/sugarcrm-tutorial/ic795891.png "Authentification SAML")  

    a. Dans la zone de texte **Login URL** (URL de connexion), collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.
  
    b. Dans la zone de texte **SLO URL** (URL de déconnexion unique), collez la valeur de l’ **URL de déconnexion** que vous avez copiée à partir du portail Azure.
  
    c. Ouvrez votre certificat codé en base 64 dans le bloc-notes, copiez son contenu dans le Presse-papiers et collez-le dans la zone de texte **X.509 Certificate** .
  
    d. Cliquez sur **Enregistrer**.

### <a name="create-sugar-crm-test-user"></a>Créer un utilisateur de test Sugar CRM

Pour se connecter à Sugar CRM, les utilisateurs Azure AD doivent être provisionnés dans Sugar CRM. Dans le cas de Sugar CRM, l’approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre site d’entreprise **Sugar CRM** en tant qu’administrateur.

1. Accédez à **Admin**.

    ![Administrateur](./media/sugarcrm-tutorial/ic795888.png "Admin")

1. Dans la section **Administration** , cliquez sur **User Management (Gestion des utilisateurs)** .

    ![Capture d’écran montrant la section Administration dans laquelle vous pouvez sélectionner User Management](./media/sugarcrm-tutorial/ic795893.png "Administration")

1. Accédez à **Users (Utilisateurs) \> Create New User (Créer un utilisateur)** .

    ![Créer un utilisateur](./media/sugarcrm-tutorial/ic795894.png "Create new user")

1. Dans l’onglet **User Profile** , procédez comme suit :

    ![Capture d’écran montrant l’onglet User Profile sous lequel vous pouvez entrer les valeurs décrites](./media/sugarcrm-tutorial/ic795895.png "Nouvel utilisateur")

    * Indiquez le **prénom** , le **nom** et l’ **adresse de messagerie** du compte Azure AD valide que vous souhaitez approvisionner, dans les zones de texte correspondantes.
  
1. Pour **Status (Statut)** , sélectionnez **Active (Actif)** .

1. Dans l’onglet Password, procédez comme suit :

    ![Capture d’écran montrant l’onglet Password sous lequel vous pouvez entrer les valeurs décrites](./media/sugarcrm-tutorial/ic795896.png "Nouvel utilisateur")

    a. Tapez le mot de passe dans la zone de texte correspondante.

    b. Cliquez sur **Enregistrer**.

> [!NOTE]
> Vous pouvez utiliser tout autre outil ou n’importe quelle API de création de compte utilisateur fournis par Sugar CRM pour provisionner des comptes d’utilisateurs Azure AD.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Sugar CRM dans le panneau d’accès doit vous connecter automatiquement à l’application Sugar CRM pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Sugar CRM avec Azure AD](https://aad.portal.azure.com/)