---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Helpshift | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Helpshift.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/20/2019
ms.author: jeedes
ms.openlocfilehash: b20d47962bf5467b9a5d69327e7fc152215edaa7
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92445028"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-helpshift"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Helpshift

Dans ce tutoriel, vous allez découvrir comment intégrer Helpshift à Azure AD (Azure Active Directory). Quand vous intégrez Helpshift à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Helpshift.
* Permettre à vos utilisateurs de se connecter automatiquement à Helpshift avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Helpshift pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Helpshift prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**

## <a name="adding-helpshift-from-the-gallery"></a>Ajout de Helpshift à partir de la galerie

Pour configurer l’intégration de Helpshift à Azure AD, vous devez ajouter Helpshift à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **Helpshift** dans la zone de recherche.
1. Sélectionnez **Helpshift** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-helpshift"></a>Configurer et tester l’authentification unique Azure AD pour Helpshift

Configurez et testez l’authentification unique Azure AD avec Helpshift à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Helpshift associé.

Pour configurer et tester l’authentification unique Azure AD avec Helpshift, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Helpshift](#configure-helpshift-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Helpshift](#create-helpshift-test-user)** pour avoir un équivalent de B.Simon dans Helpshift lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Helpshift** , recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<YourDOMAIN>.helpshift.com/`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<YourDOMAIN>.helpshift.com/login/saml/acs/`

1. Cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes si vous souhaitez configurer l’application en mode lancé par le **fournisseur de services**  :

    d. Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<YourDOMAIN>.helpshift.com/login/saml/idp-login/`.

    e. Dans la zone de texte **État de relais** , entrez une URL en utilisant le modèle suivant : `https://<YourDOMAIN>.helpshift.com/`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse, l’URL de connexion et l’État de relais exacts. Pour obtenir ces valeurs, contactez l’[équipe de support client de Helpshift](mailto:support@helpshift.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Helpshift** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Helpshift.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Helpshift**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

## <a name="configure-helpshift-sso"></a>Configurer l’authentification unique Helpshift

1. Dans un autre navigateur web, connectez-vous à votre application Helpshift en tant qu’administrateur.

1. Ouvrez le tableau de bord **Helpshift** , puis cliquez sur l’icône **Paramètres**.

    ![Capture d’écran montrant l’icône des paramètres Helpshift.](./media/helpshift-tutorial/configuration01.png)

1. Cliquez sur l’onglet **Integrations** et effectuez les étapes suivantes :

    ![Capture d’écran montrant l’onglet Integration où vous pouvez effectuer les étapes décrites.](./media/helpshift-tutorial/configuration02.png)

    a. Activez **Single Sign-On(SAML – SSO)** .

    b. Sélectionnez **Azure Active Directory** comme **Identity Provider(IDP)** .

    c. Dans la zone de texte **SAML 2.0 Endpoint URL** (URL de point de terminaison SAML 2.0), collez l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    d. Ouvrez le fichier de **Certificat (base64)** téléchargé dans le Bloc-notes, copiez le contenu du fichier (sans les lignes « —–BEGIN CERTIFICATE—– » et « —–END CERTIFICATE—– »), puis collez-le dans la zone de texte **X.509 Certificate**.

    e. Dans la zone de texte **Issuer URL** (URL de l’émetteur), collez l’ **Identificateur Azure AD** que vous avez copié à partir du portail Azure.

    f. Cliquez sur **APPLY CHANGES** (Appliquer les modifications).

### <a name="create-helpshift-test-user"></a>Créer un utilisateur de test Helpshift

Dans cette section, vous allez créer un utilisateur appelé B.Simon dans Helpshift. Collaborez avec l’[équipe de support technique Helpshift](mailto:support@helpshift.com) pour ajouter des utilisateurs à la plateforme Helpshift. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Helpshift dans le volet d’accès, vous devez être connecté automatiquement à l’application Helpshift pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Helpshift avec Azure AD](https://aad.portal.azure.com/)