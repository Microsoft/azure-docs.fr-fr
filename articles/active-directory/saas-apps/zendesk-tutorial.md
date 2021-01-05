---
title: 'Tutoriel : Intégration d’Azure Active Directory à Zendesk | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Zendesk.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/20/2020
ms.author: jeedes
ms.openlocfilehash: 4973a5f1311de23cf54e401a52fb083f497687a3
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92894472"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-zendesk"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Zendesk

Dans ce tutoriel, vous allez découvrir comment intégrer Zendesk à Azure Active Directory (Azure AD). Quand vous intégrez Zendesk à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Zendesk.
* Permettre à vos utilisateurs de se connecter automatiquement à Zendesk avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Zendesk pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Zendesk prend en charge l’authentification unique initiée par le **fournisseur de services**
* Zendesk prend en charge [l’attribution d’utilisateurs **automatique**](zendesk-provisioning-tutorial.md)
* Une fois que vous avez configuré Zendesk, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-zendesk-from-the-gallery"></a>Ajout de Zendesk depuis la galerie

Pour configurer l’intégration de Zendesk à Azure AD, vous devez ajouter Zendesk, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **Zendesk** dans la zone de recherche.
1. Sélectionnez **Zendesk** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-zendesk"></a>Configurer et tester l’authentification unique Azure AD pour Zendesk

Configurez et testez l’authentification unique Azure AD avec Zendesk pour un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Zendesk associé.

Pour configurer et tester l’authentification unique Azure AD avec Zendesk, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Zendesk](#configure-zendesk-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Zendesk](#create-zendesk-test-user)**  : pour avoir un équivalent de B.Simon dans Zendesk lié à la représentation de l’utilisateur Azure AD.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Zendesk** , recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<subdomain>.zendesk.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<subdomain>.zendesk.com`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<subdomain>.zendesk.com/access/saml`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique Zendesk](https://support.zendesk.com/hc/en-us/articles/203663676-Using-SAML-for-single-sign-on-Professional-and-Enterprise). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. L’application Zendesk attend les assertions SAML dans un format spécifique. Il n’y a aucun attribut SAML obligatoire, mais si vous le souhaitez, vous pouvez les gérer à partir de la section **Attributs utilisateur** dans la page d’intégration de l’application. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur le bouton **Modifier** pour ouvrir la boîte de dialogue **Attributs utilisateur**.

    ![Capture d’écran montrant User Attributes avec l’icône Edit sélectionnée.](common/edit-attribute.png)

    > [!NOTE]
    > Utilisez des attributs d’extension pour ajouter des attributs qui ne sont pas dans Azure AD par défaut. Cliquez sur [les attributs d’utilisateur qui peuvent être définis dans SAML](https://support.zendesk.com/hc/articles/203663676-Using-SAML-for-single-sign-on-Professional-and-Enterprise-) pour obtenir la liste complète des attributs SAML que **Zendesk** accepte.

1. Dans la section **Certificat de signature SAML** , cliquez sur le bouton **Modifier** pour ouvrir la boîte de dialogue **Certificat de signature SAML**.

    ![Modifier le certificat de signature SAML](common/edit-certificate.png)

1. Dans la section **Certificat de signature SAML** , copiez la **valeur de l’empreinte** et enregistrez-la sur votre ordinateur.

    ![Copier la valeur de l’empreinte](common/copy-thumbprint.png)

1. Dans la section **Configurer Zendesk** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Zendesk.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Zendesk**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

## <a name="configure-zendesk-sso"></a>Configurer l’authentification unique Zendesk

1. Pour automatiser la configuration dans **Zendesk** , vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Install the extension** (Installer l’extension).

    ![Capture d’écran montrant le bouton « Install the extension »](./media/target-process-tutorial/install_extension.png)

1. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Zendesk** , vous êtes alors orienté vers l’application Zendesk. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Zendesk. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 6.

    ![Configuration](common/setup-sso.png)

1. Si vous voulez configurer manuellement Zendesk, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Zendesk en tant qu’administrateur et procédez comme suit :

1. Dans le **centre d’administration Zendesk** , cliquez sur **Paramètres de sécurité** sous l’onglet **Sécurité** .

    ![Capture d’écran montrant le centre d’administration Zendesk avec « Paramètres de sécurités » sélectionnés](./media/zendesk-tutorial/settings.png "Sécurité")

1. Accédez à la page **Authentification unique** , puis cliquez sur **Modifier** dans **SAML**.

    ![Capture d’écran montrant la page Authentification unique avec l’option Modifier sélectionnée](./media/zendesk-tutorial/saml-sso.png "Sécurité")

1. Procédez comme suit dans la page **SSO**.

    ![Authentification unique](./media/zendesk-tutorial/saml-configuration.png "Authentification unique")

    a. Dans la zone de texte **SAML SSO URL** (URL SSO SAML), collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.

    b. Dans la zone de texte **Empreinte du certificat** , collez la valeur du certificat **Empreinte** que vous avez copiée à partir du portail Azure.

    c. Dans la zone de texte **Remote Logout URL** (URL de déconnexion distante), collez la valeur de l’ **URL de déconnexion** que vous avez copiée à partir du portail Azure.

    d. Cliquez sur **Enregistrer**.

### <a name="create-zendesk-test-user"></a>Créer un utilisateur de test Zendesk

L’objectif de cette section est de créer un utilisateur appelé Britta Simon dans Zendesk. Zendesk prend en charge l’approvisionnement automatique d’utilisateurs, qui est activé par défaut. Vous trouverez plus d’informations [ici](Zendesk-provisioning-tutorial.md) sur la façon de configurer l’attribution automatique d’utilisateurs.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Zendesk dans le panneau d’accès doit vous connecter automatiquement à l’application Zendesk pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Zendesk avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Zendesk avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)

- [Configurer l’approvisionnement de l’utilisateur](zendesk-provisioning-tutorial.md)