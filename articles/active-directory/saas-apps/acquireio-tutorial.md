---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à AcquireIO | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et AcquireIO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/14/2019
ms.author: jeedes
ms.openlocfilehash: 5fe070bc1abe0592b3082c597c1812781335448a
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97673181"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-acquireio"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à AcquireIO

Dans ce tutoriel, vous allez apprendre à intégrer AcquireIO à Azure Active Directory (Azure AD). Quand vous intégrez AcquireIO à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à AcquireIO.
* Permettre à vos utilisateurs de se connecter automatiquement à AcquireIO avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement AcquireIO pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* AcquireIO prend en charge l’authentification unique lancée par le **fournisseur d’identité**

## <a name="adding-acquireio-from-the-gallery"></a>Ajout de AcquireIO depuis la galerie

Pour configurer l’intégration de AcquireIO avec Azure AD, vous devez ajouter AcquireIO à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **AcquireIO** dans la zone de recherche.
1. Sélectionnez **AcquireIO** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-acquireio"></a>Configurer et tester l’authentification unique Azure AD pour AcquireIO

Configurez et testez l’authentification unique Azure AD avec AcquireIO à l’aide d’un utilisateur de test appelé **B. Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur AcquireIO associé.

Pour configurer et tester l’authentification unique Azure AD avec AcquireIO, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique AcquireIO](#configure-acquireio-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test AcquireIO](#create-acquireio-test-user)** pour avoir dans AcquireIO un équivalent de B.Simon lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **AcquireIO**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://app.acquire.io/ad/<acquire_account_uid>`

    > [!NOTE]
    > Cette valeur n’est pas la valeur réelle. Vous obtiendrez l’URL de réponse avec la valeur réelle. La procédure est expliquée plus loin dans le didacticiel **Configurer AcquireIO**. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer AcquireIO**, copiez l’URL ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory**, **Utilisateurs**, puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans le champ **Nom**, entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur**, entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à AcquireIO.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **AcquireIO**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-acquireio-sso"></a>Configurer l’authentification unique AcquireIO

1. Pour automatiser la configuration dans AcquireIO, vous devez installer l’**extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension**.

    ![Extension My apps](common/install-myappssecure-extension.png)

1. Après avoir ajouté l’extension au navigateur, cliquez sur **Configurer AcquireIO**, ce qui vous dirige vers l’application AcquireIO. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à AcquireIO. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 6.

    ![Configuration](common/setup-sso.png)

1. Si vous souhaitez configurer AcquireIO manuellement, dans une autre fenêtre de navigateur web, connectez-vous à AcquireIO en tant qu’administrateur.

1. Sur le côté gauche du menu, cliquez sur **App Store**.

    ![Capture d’écran mettant en évidence l’App Store.](./media/acquireio-tutorial/config01.png)

1. Faites défiler l’écran vers le bas jusqu’à **Active Directory**, puis cliquez sur **Installer**.

    ![Capture d’écran mettant en évidence la section Active Directory et le bouton Install.](./media/acquireio-tutorial/config02.png)

1. Sur la fenêtre contextuelle d’Active Directory, procédez comme suit :

    ![Capture d’écran montrant l’écran Active Directory.](./media/acquireio-tutorial/config03.png)

    a. Cliquez sur **COPY** pour copier l’URL de réponse pour votre instance et collez-la dans la zone de texte **URL de réponse** de la section **Configuration SAML de base** du portail Azure.

    b. Dans la zone de texte **Login URL** (URL de connexion), collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.

    c. Ouvrez le certificat encodé en Base64 dans le bloc-notes, copiez son contenu et collez-le dans la zone de texte **Certificat X.509**.

    d. Cliquez sur **Se connecter maintenant**.

### <a name="create-acquireio-test-user"></a>Créer un utilisateur test d’AcquireIO

Pour permettre aux utilisateurs Azure AD de se connecter à AcquireIO, vous devez les approvisionner dans AcquireIO. Dans AcquireIO, le provisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Dans une autre fenêtre de navigateur web, connectez-vous à AcquireIO en tant qu’administrateur.

1. Sur le côté gauche du menu, cliquez sur **Profils** et accédez à **Ajouter un profil**.

    ![Capture d’écran mettant en évidence Profiles dans le menu sur le côté gauche de l’écran ainsi que l’option Add Profile.](./media/acquireio-tutorial/config04.png)

1. Dans la fenêtre contextuelle **Add customer** (Ajouter un client), effectuez les étapes suivantes :

    ![Configuration d’AcquireIO](./media/acquireio-tutorial/config05.png)

    a. Dans la zone de texte **Nom**, entrez le nom d’un utilisateur, par exemple **B. Simon**.

    b. Dans la zone de texte **Email** (E-mail), entrez l’adresse e-mail de l’utilisateur, par exemple **B.simon@contoso.com** .

    c. Cliquez sur **Envoyer**.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette AcquireIO dans le volet d’accès, vous devez être connecté automatiquement à l’application AcquireIO pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer AcquireIO avec Azure AD](https://aad.portal.azure.com/)