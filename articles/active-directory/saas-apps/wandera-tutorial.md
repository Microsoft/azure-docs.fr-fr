---
title: 'Tutoriel : Intégration d’Azure Active Directory à Wandera RADAR Admin | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Wandera RADAR Admin.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/27/2020
ms.author: jeedes
ms.openlocfilehash: d13619b818e18c64d9882f9e3181824173403859
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96181388"
---
# <a name="tutorial-integrate-wandera-radar-admin-with-azure-active-directory"></a>Tutoriel : Intégrer Wandera RADAR Admin à Azure Active Directory

Dans ce tutoriel, vous allez apprendre à intégrer Wandera RADAR Admin à Azure Active Directory (Azure AD). Quand vous intégrez Wandera RADAR Admin à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Wandera RADAR Admin.
* Permettre à vos utilisateurs de se connecter automatiquement à Wandera RADAR Admin avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Wandera RADAR Admin pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Wandera RADAR Admin prend en charge l’authentification unique lancée par le **fournisseur d’identité**.
* Après avoir configuré Wandera RADAR Admin, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-wandera-radar-admin-from-the-gallery"></a>Ajout de Wandera RADAR Admin à partir de la galerie

Pour configurer l’intégration de Wandera RADAR Admin à Azure AD, vous devez ajouter Wandera RADAR Admin à votre liste d’applications SaaS gérées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Wandera RADAR Admin** dans la zone de recherche.
1. Sélectionnez **Wandera RADAR Admin** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec Wandera RADAR Admin à l’aide d’un utilisateur de test nommé **B. Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Wandera RADAR Admin associé.

Pour configurer et tester l’authentification unique Azure AD avec Wandera RADAR Admin, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
   * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
   * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Wandera RADAR Admin](#configure-wandera-radar-admin-sso)** pour configurer les paramètres de l’authentification unique côté application.
   * **[Créer un utilisateur de test Wandera RADAR Admin](#create-wandera-radar-admin-test-user)** pour avoir dans Wandera RADAR Admin un équivalent de B. Simon lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Wandera RADAR Admin**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://radar.wandera.com/saml/acs/<tenant id>`

    > [!NOTE]
    > Cette valeur n’est pas la valeur réelle. Mettez à jour la valeur avec l’URL de réponse réelle. Contactez [l’équipe de support client de Wandera RADAR Admin](https://www.wandera.com/about-wandera/contact/#supportsection) pour obtenir la valeur. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet pour **Certificat de signature SAML** afin de modifier les paramètres.

    ![Option de signature](common/signing-option.png)

    1. Sélectionnez **Option de signature** pour **Signer la réponse et l’assertion SAML**.

    1. Dans **Algorithme de signature**, sélectionnez **SHA-256**.

1. Dans la section **Configurer Wandera RADAR Admin**, copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Wandera RADAR Admin.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Wandera RADAR Admin**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-wandera-radar-admin-sso"></a>Configurer l’authentification unique Wandera RADAR Admin

1. Afin d’automatiser la configuration dans Wandera RADAR Admin, vous devez installer **l’extension de navigateur permettant la connexion sécurisée à Mes applications** en cliquant sur **Installer l’extension**.

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Wandera RADAR Admin**, vous êtes alors orienté vers l’application Wandera RADAR Admin. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Wandera RADAR Admin. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 et 4.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement Wandera RADAR Admin, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Wandera RADAR Admin en tant qu’administrateur et effectuez les étapes suivantes :

4. En haut à droite de la page, cliquez sur **Settings** > **Administration** > **Single Sign-On** (Authentification unique), puis cochez l’option **Enable SAML 2.0** (Activer SAML 2.0) pour effectuer les étapes suivantes.

    ![Configuration de Wandera RADAR Admin](./media/wandera-tutorial/config01.png)

    a. Cliquez sur **Or manually enter the required fields** (Ou entrer manuellement les champs obligatoires).

    b. Dans la zone de texte **IdP EntityId**, collez la valeur **Azure AD Identifier** que vous avez copiée dans le portail Azure.

    c. Ouvrez le XML de métadonnées de fédération dans le Bloc-notes, copiez son contenu et collez-le dans la zone de texte **IdP Public X.509 Certificate**.

    d. Cliquez sur **Enregistrer**.

### <a name="create-wandera-radar-admin-test-user"></a>Créer un utilisateur de test Wandera RADAR Admin

Dans cette section, vous créez un utilisateur appelé B.Simon dans Wandera RADAR Admin. Demandez l’aide de l’[équipe du support technique Wandera RADAR Admin](https://www.wandera.com/about-wandera/contact/#supportsection) pour ajouter des utilisateurs dans la plateforme Wandera RADAR Admin. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Wandera RADAR Admin dans le panneau d’accès doit vous connecter automatiquement à l’application Wandera RADAR Admin pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)