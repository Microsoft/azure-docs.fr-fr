---
title: 'Tutoriel : Intégration d’ExpenseIn à Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et ExpenseIn.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/17/2020
ms.author: jeedes
ms.openlocfilehash: eb7062b745ae6eb97dc2cd4ba2051cc221031630
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96179161"
---
# <a name="tutorial-integrate-expensein-with-azure-active-directory"></a>Tutoriel : Intégrer ExpenseIn à Azure Active Directory

Dans ce tutoriel, vous allez apprendre à intégrer ExpenseIn à Azure Active Directory (Azure AD). Quand vous intégrez ExpenseIn à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à ExpenseIn.
* Autoriser les utilisateurs à se connecter automatiquement à ExpenseIn avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement ExpenseIn pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. 
* ExpenseIn prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité**.
* Après avoir configuré ExpenseIn, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).


## <a name="adding-expensein-from-the-gallery"></a>Ajouter ExpenseIn à partir de la galerie

Pour configurer l’intégration d’ExpenseIn à Azure AD, vous devez ajouter ExpenseIn à votre liste d’applications SaaS managées, à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **ExpenseIn** dans la zone de recherche.
1. Sélectionnez **ExpenseIn** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-expensein"></a>Configurer et tester l’authentification unique Azure AD pour ExpenseIn

Configurez et testez l’authentification unique Azure AD avec ExpenseIn pour un utilisateur de test nommé **B. Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur ExpenseIn associé.

Pour configurer et tester l’authentification unique Azure AD avec ExpenseIn, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique ExpenseIn](#configure-expensein-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test ExpenseIn](#create-expensein-test-user)** pour avoir un équivalent de B. Simon dans ExpenseIn lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **ExpenseIn**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base**, l’utilisateur n’a rien à faire, car l’application est déjà intégrée à Azure.

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion**, tapez une URL : `https://app.expensein.com/saml`

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur l’icône Copier pour copier l’**URL des métadonnées de fédération d’application** et cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** et l’enregistrer sur votre ordinateur.

   ![Lien Téléchargement de certificat](./media/expensein-tutorial/copy-metdataurl-certificate.png)

1. Dans la section **Configurer ExpenseIn**, copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à ExpenseIn.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **ExpenseIn**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.


## <a name="configure-expensein-sso"></a>Configurer l’authentification unique ExpenseIn

1. Pour automatiser la configuration dans ExpenseIn, vous devez installer l’**extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension**.

    ![Extension My apps](common/install-myappssecure-extension.png)

1. Une fois l’extension ajoutée au navigateur, le fait de cliquer sur **Configurer ExpenseIn** vous redirige vers l’application ExpenseIn. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à ExpenseIn. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 5.

    ![Configuration](common/setup-sso.png)

1. Si vous voulez configurer ExpenseIn manuellement, connectez-vous à votre site d’entreprise ExpenseIn en tant qu’administrateur.

1. Cliquez sur **Admin** en haut de la page, puis accédez à **Single Sign-On** (Authentification unique) et cliquez sur **Add provider** (Ajouter un fournisseur).

     ![Capture d’écran montrant l’onglet « Admin » et la page « Single Sign-On - Providers » et « Add Provider » sélectionné.](./media/expenseIn-tutorial/config01.png)

1. Dans la fenêtre contextuelle **New Identity Provider** (Nouveau fournisseur d’identité), effectuez les étapes suivantes :

    ![Capture d’écran montrant la fenêtre contextuelle « Edit Identity Provider » avec des valeurs entrées.](./media/expenseIn-tutorial/config02.png)

    a. Dans la zone de texte **Provider Name** (Nom du fournisseur), tapez le nom, par exemple Azure.

    b. Sélectionnez **Yes** pour **Allow Provider Intitiated Sign-On** (Autoriser l’authentification lancée par le fournisseur).

    c. Dans la zone de texte **Target Url** (URL cible), collez la valeur **URL de connexion** que vous avez copiée à partir du portail Azure.

    d. Dans la zone de texte **Issuer** (Émetteur), collez la valeur **Identificateur Azure AD** que vous avez copiée depuis le portail Azure.

    e. Ouvrez le certificat (Base64) dans le Bloc-notes, copiez son contenu et collez-le dans la zone de texte **Certificate** (Certificat).

    f. Cliquez sur **Créer**.

### <a name="create-expensein-test-user"></a>Créer un utilisateur de test ExpenseIn

Pour permettre aux utilisateurs Azure AD de se connecter à ExpenseIn, vous devez les provisionner dans ExpenseIn. Dans ExpenseIn, le provisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à ExpenseIn en tant qu’administrateur.

2. Cliquez sur **Admin** en haut de la page, puis accédez à **Users** (Utilisateurs) et cliquez sur **New User** (Nouvel utilisateur).

     ![Capture d’écran montrant l’onglet « Admin » et la page « Manage Users » avec « New User » sélectionné.](./media/expenseIn-tutorial/config03.png)

3. Dans la fenêtre contextuelle **Details** (Détails), effectuez les étapes suivantes :

    ![Configuration d’ExpenseIn](./media/expenseIn-tutorial/config04.png)

    a. Dans la zone de texte **First Name** (Prénom), tapez le prénom de l’utilisateur, par exemple **B**.

    b. Dans la zone de texte **Nom**, entrez le nom de l’utilisateur, par exemple **Simon**.

    c. Dans la zone de texte **E-mail**, entrez l’adresse e-mail de l’utilisateur, comme `B.Simon@contoso.com`.

    d. Cliquez sur **Créer**.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Quand vous sélectionnez la vignette ExpenseIn dans le volet d’accès, vous devez être connecté automatiquement à l’application ExpenseIn pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer ExpenseIn avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger ExpenseIn avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)