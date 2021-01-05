---
title: 'Tutoriel : Intégration d’Azure Active Directory à Freedcamp | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Freedcamp.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/20/2019
ms.author: jeedes
ms.openlocfilehash: 19378251408d55868ed844a5505ae48ece55dc3b
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92451484"
---
# <a name="tutorial-integrate-freedcamp-with-azure-active-directory"></a>Tutoriel : Intégrer Freedcamp à Azure Active Directory

Dans ce didacticiel, vous allez apprendre à intégrer Freedcamp à Azure Active Directory (Azure AD). Lorsque vous intégrez Freedcamp à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Freedcamp.
* Permettre à vos utilisateurs de se connecter automatiquement à Freedcamp avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Freedcamp pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. Freedcamp prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité** .

## <a name="adding-freedcamp-from-the-gallery"></a>Ajout de Freedcamp depuis la galerie

Pour configurer l’intégration de Freedcamp avec Azure AD, vous devez ajouter Freedcamp disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Freedcamp** dans la zone de recherche.
1. Sélectionnez **Freedcamp** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec Freedcamp à l’aide d’un utilisateur de test appelé **Britta Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Freedcamp associé.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec Freedcamp, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer Freedcamp](#configure-freedcamp)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Freedcamp](#create-freedcamp-test-user)** pour avoir un équivalent de Britta Simon dans Freedcamp lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [Portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Freedcamp** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , effectuez les étapes suivantes :

    1. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<SUBDOMAIN>.freedcamp.com/sso/<UNIQUEID>`

    2. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<SUBDOMAIN>.freedcamp.com/sso/acs/<UNIQUEID>`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<SUBDOMAIN>.freedcamp.com/login`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Les utilisateurs peuvent entrer également les valeurs d’URL de leur propre domaine client qui ne sont pas nécessairement dans le modèle `freedcamp.com` ; il peut s’agir de n’importe quelle valeur de domaine client propre à leur instance d’application. Vous pouvez également contacter [l'équipe du support client Freedcamp ](mailto:devops@freedcamp.com) pour plus d’informations sur les modèles d’URL.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (Base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

   ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Freedcamp** , copiez la ou les URL appropriées en fonction de vos besoins.

   ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="configure-freedcamp"></a>Configurer Freedcamp

1. Pour automatiser la configuration dans Freedcamp, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Install the extension** (Installer l’extension).

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Freedcamp** pour être dirigé vers l’application Freedcamp. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Freedcamp. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 5.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement Freedcamp, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Freedcamp en tant qu’administrateur et procédez comme suit :

4. Dans le coin supérieur droit de la page, cliquez sur **Profil** , puis accédez à **Mon compte** .

    ![Capture d’écran montrant « Profile » et « My Account » sélectionnés.](./media/freedcamp-tutorial/config01.png)

5. Sur le côté gauche de la barre de menus, cliquez sur **SSO** et dans la page **Vos connexions SSO** , effectuez les étapes suivantes :

    ![Capture d’écran montrant « SSO » sélectionné dans la barre de menus de gauche et la page « Your SSO connections » avec des valeurs entrées et le bouton « Submit » sélectionné.](./media/freedcamp-tutorial/config02.png)

    a. Dans la zone de texte **Title** , tapez le titre.

    b. Dans la zone de texte **ID d’entité** , collez la valeur **Identificateur Azure AD** que vous avez copiée dans le Portail Azure.

    c. Dans la zone de texte **URL de connexion** , collez la valeur d’ **URL de connexion** que vous avez copiée sur le Portail Azure.

    d. Ouvrez le certificat encodé en Base64 dans le bloc-notes, copiez son contenu et collez-le dans la zone de texte **Certificat** .

    e. Cliquez sur **Envoyer** .

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé Britta Simon dans le Portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `Britta Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `BrittaSimon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Freedcamp.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Freedcamp** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-freedcamp-test-user"></a>Créer un utilisateur de test Freedcamp

Pour se connecter à Freedcamp, les utilisateurs d’Azure AD doivent être approvisionnés dans Freedcamp. Dans Freedcamp, le provisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Dans une autre fenêtre de navigateur web, connectez-vous à Freedcamp en tant qu’administrateur de la sécurité.

2. Dans l’angle supérieur droit de la page, cliquez sur **Profile** , puis accédez à **Manage System** .

    ![Configuration de Freedcamp](./media/freedcamp-tutorial/config03.png)

3. Dans la partie droite de la page Gérer le système, procédez comme suit :

    ![Capture d’écran montrant le bouton « Add Or Invite Users » sélectionné, le champ « Email » mis en évidence et le bouton « Add User » sélectionné.](./media/freedcamp-tutorial/config04.png)

    a. Cliquez sur **Ajouter ou inviter les utilisateurs** .

    b. Dans la zone de texte **Email** (E-mail), entrez l’adresse e-mail de l’utilisateur, par exemple `Brittasimon@contoso.com`.

    c. Cliquez sur **Add User** .

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Quand vous sélectionnez la vignette Freedcamp dans le volet d’accès, vous devez être connecté automatiquement à l’application Freedcamp pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)