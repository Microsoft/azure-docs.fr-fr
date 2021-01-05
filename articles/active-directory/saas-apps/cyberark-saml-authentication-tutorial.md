---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à CyberArk SAML Authentication | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et CyberArk SAML Authentication.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/03/2019
ms.author: jeedes
ms.openlocfilehash: ddc639b38cb25ff7e395327b303577d9003230e5
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92455063"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-cyberark-saml-authentication"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à CyberArk SAML Authentication

Ce didacticiel explique comment intégrer CyberArk SAML Authentication avec Azure Active Directory (Azure AD). Lorsque vous intégrez CyberArk SAML Authentication avec Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à CyberArk SAML Authentication.
* Autoriser vos utilisateurs à se connecter automatiquement à CyberArk SAML Authentication avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement CyberArk SAML Authentication pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* CyberArk SAML Authentication prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité**

## <a name="adding-cyberark-saml-authentication-from-the-gallery"></a>Ajout de CyberArk SAML Authentication à partir de la galerie

Pour configurer l’intégration de CyberArk SAML Authentication dans Azure AD, vous devez ajouter CyberArk SAML Authentication à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **CyberArk SAML Authentication** dans la zone de recherche.
1. Sélectionnez **CyberArk SAML Authentication** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-cyberark-saml-authentication"></a>Configurer et tester l’authentification unique Azure AD pour CyberArk SAML Authentication

Configurez et testez l’authentification unique Azure AD avec CyberArk SAML Authentication à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur CyberArk SAML Authentication associé.

Pour configurer et tester l’authentification unique Azure AD avec CyberArk SAML Authentication, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique CyberArk SAML Authentication](#configure-cyberark-saml-authentication-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test CyberArk SAML Authentication](#create-cyberark-saml-authentication-test-user)** pour avoir un équivalent de B.Simon dans CyberArk SAML Authentication lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **CyberArk SAML Authentication** , recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<PVWA DNS or IP>/passwordvault/api/auth/saml/logon`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<PVWA DNS or IP>/PasswordVault/v10/logon/saml`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de réponse et l’URL de connexion réelles. Pour obtenir ces valeurs, contactez [l’équipe du support client CyberArk SAML Authentication](mailto:bizdevtech@cyberark.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer CyberArk SAML Authentication** , copiez la ou les URL appropriées correspondant à vos besoins.

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

Dans cette section, vous autorisez B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à CyberArk SAML Authentication.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **CyberArk SAML Authentication**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

## <a name="configure-cyberark-saml-authentication-sso"></a>Configurer l’authentification unique CyberArk SAML Authentication

Pour configurer l’authentification unique côté **CyberArk SAML Authentication** , vous devez envoyer le **certificat (en base64)** téléchargé ainsi que les URL copiées appropriées dans le portail Azure à l’ [équipe de support technique CyberArk SAML Authentication](mailto:bizdevtech@cyberark.com). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-cyberark-saml-authentication-test-user"></a>Créer un utilisateur de test CyberArk SAML Authentication

Dans cette section, vous créez un utilisateur appelé B.Simon dans CyberArk SAML Authentication. Collaborez avec l’[équipe de support technique CyberArk SAML Authentication](mailto:bizdevtech@cyberark.com) pour ajouter les utilisateurs dans la plateforme CyberArk SAML Authentication. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette CyberArk SAML Authentication dans le volet d’accès, vous devez être connecté automatiquement à l’application CyberArk SAML Authentication pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer CyberArk SAML Authentication avec Azure AD](https://aad.portal.azure.com/)