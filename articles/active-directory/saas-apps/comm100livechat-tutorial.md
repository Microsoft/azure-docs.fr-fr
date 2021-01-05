---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Comm100 Live Chat | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Comm100 Live Chat.
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
ms.openlocfilehash: 9636c8cbb517c7aece450f53cfc37e4ddd9803b7
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92455491"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-comm100-live-chat"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Comm100 Live Chat

Dans ce tutoriel, vous allez découvrir comment intégrer Comm100 Live Chat à Azure Active Directory (Azure AD). Quand vous intégrez Comm100 Live Chat avec Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Comm100 Live Chat.
* Permettre à vos utilisateurs de se connecter automatiquement à Comm100 Live Chat avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Comm100 Live Chat pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Comm100 Live Chat prend en charge l’authentification unique lancée par le **fournisseur de services**

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-comm100-live-chat-from-the-gallery"></a>Ajout de Comm100 Live Chat à partir de la galerie

Pour configurer l’intégration de Comm100 Live Chat avec Azure AD, vous devez ajouter Comm100 Live Chat à votre liste d’applications SaaS gérées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Comm100 Live Chat** dans la zone de recherche.
1. Sélectionnez **Comm100 Live Chat** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-comm100-live-chat"></a>Configurer et tester l’authentification unique Azure AD pour Comm100 live chat

Configurez et testez l’authentification unique Azure AD avec Comm100 Live Chat à l’aide d’un utilisateur de test appelé **B. Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Comm100 Live Chat associé.

Pour configurer et tester l’authentification unique Azure AD avec Comm100 Live Chat, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Comm100 Live Chat](#configure-comm100-live-chat-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Comm100 Live Chat](#create-comm100-live-chat-test-user)** pour avoir un équivalent de B.Simon dans Comm100 Live Chat lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Comm100 Live Chat** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<SUBDOMAIN>.comm100.com/AdminManage/LoginSSO.aspx?siteId=<SITEID>`

    > [!NOTE] 
    > La valeur de l’URL de connexion n’est pas réelle. Vous mettrez à jour cette valeur avec l’URL de connexion réelle. La procédure est expliquée plus loin dans le didacticiel.

1. L’application Comm100 Live Chat s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jetons SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/edit-attribute.png)

1. En plus de ce qui précède, l’application Comm100 Live Chat s’attend à ce que quelques attributs supplémentaires soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom |  Attribut source|
    | ---------------| --------------- |
    |   email    | user.mail |

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Comm100 Live Chat** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Comm100 Live Chat.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Comm100 Live Chat** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-comm100-live-chat-sso"></a>Configurer l’authentification unique Comm100 Live Chat

1. Ouvrez une autre fenêtre de navigateur web et connectez-vous à Comm100 Live Chat en tant qu’administrateur de la sécurité.

1. Dans l’angle supérieur droit de la page, cliquez sur **My Account** (Mon compte).

   ![Comm100 Live Chat : myaccount](./media/comm100livechat-tutorial/tutorial_comm100livechat_account.png)

1. Dans le menu, à gauche, cliquez sur **Security** (Sécurité), puis sur **Agent Single Sign-On** (Authentification unique des agents).

   ![Capture d’écran montrant le menu de compte de gauche avec « Security » et « Agent Single Sign-On » mis en évidence.](./media/comm100livechat-tutorial/tutorial_comm100livechat_security.png)

1. Dans la page **Agent Single Sign-On** (Authentification unique des agents), effectuez les opérations suivantes :

   ![Comm100 Live Chat : sécurité](./media/comm100livechat-tutorial/tutorial_comm100livechat_singlesignon.png)

   a. Copiez le premier lien mis en surbrillance et collez-le dans la zone de texte **URL d’authentification** de la section **Configuration SAML de base** du portail Azure.

   b. Dans la zone de texte **SAML SSO URL** (URL d’authentification unique SAML), collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.

   c. Dans la zone de texte **Remote Logout URL** (URL de déconnexion à distance), collez la valeur de l’ **URL de déconnexion** que vous avez copiée à partir du Portail Azure.

   d. Sous **Certificat** , cliquez sur **Choose a File** (Choisir un fichier) pour charger le certificat codé en base 64 que vous avez téléchargé à partir du Portail Azure.

   e. Cliquez sur **Enregistrer les modifications** .

### <a name="create-comm100-live-chat-test-user"></a>Créer un utilisateur de test Comm100 Live Chat

Pour permettre aux utilisateurs d’Azure AD de se connecter à Comm100 Live Chat, vous devez les provisionner dans Comm100 Live Chat. Dans Comm100 Live Chat, l’approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à Comm100 Live Chat en tant qu’administrateur de la sécurité.

2. Dans l’angle supérieur droit de la page, cliquez sur **My Account** (Mon compte).

    ![Comm100 Live Chat : myaccount](./media/comm100livechat-tutorial/tutorial_comm100livechat_account.png)

3. Dans le menu, à gauche, cliquez sur **Agents** , puis sur **New Agent** (Nouvel agent).

    ![Comm100 Live Chat : agent](./media/comm100livechat-tutorial/tutorial_comm100livechat_agent.png)

4. Dans la page **New Agent** (Nouvel agent), effectuez les opérations suivantes :

    ![Comm100 Live Chat : nouvel agent](./media/comm100livechat-tutorial/tutorial_comm100livechat_newagent.png)

    a. a. Dans la zone de texte **Email** , tapez l’adresse e-mail de l’utilisateur, par exemple **B.simon\@contoso.com** .

    b. Dans la zone de texte **First Name** (Prénom), tapez le prénom de l’utilisateur, par exemple **B** .

    c. Dans la zone de texte **Last Name** (Nom), saisissez le nom de famille de l’utilisateur, par exemple **Simon** .

    d. Dans la zone de texte **Display Name** (Nom d’affichage), tapez le nom d’affichage de l’utilisateur, par exemple **B.Simon** .

    e. Dans la zone de texte **Password** (Mot de passe), tapez votre mot de passe.

    f. Cliquez sur **Enregistrer** .

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Comm100 Live Chat dans le volet d’accès, vous devez être connecté automatiquement à l’application Comm100 Live Chat pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Comm100 Live Chat avec Azure AD](https://aad.portal.azure.com/)