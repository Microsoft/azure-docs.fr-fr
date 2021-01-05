---
title: "Tutoriel : Intégration d'Azure Active Directory à Zscaler Private Access (ZPA) | Microsoft Docs"
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Zscaler Private Access (ZPA).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/23/2019
ms.author: jeedes
ms.openlocfilehash: aaef5c01e554ac226828f601dc6144d13846198a
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92517410"
---
# <a name="tutorial-integrate-zscaler-private-access-zpa-with-azure-active-directory"></a>Tutoriel : Intégrer Zscaler Private Access (ZPA) à Azure Active Directory

Dans ce tutoriel, vous allez apprendre à intégrer Zscaler Private Access (ZPA) à Azure Active Directory (Azure AD). Quand vous intégrez Zscaler Private Access (ZPA) à Azure AD, vous pouvez :

* contrôler dans Azure AD qui a accès à Zscaler Private Access (ZPA).
* Permettre à vos utilisateurs de se connecter automatiquement à Zscaler Private Access (ZPA) avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Zscaler Private Access (ZPA) pour lequel l’authentification unique est activée

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. Zscaler Private Access (ZPA) prend en charge l’authentification unique lancée par le **fournisseur de services** .

## <a name="adding-zscaler-private-access-zpa-from-the-gallery"></a>Ajout de Zscaler Private Access (ZPA) à partir de la galerie

Pour configurer l’intégration de Zscaler Private Access (ZPA) à Azure AD, vous devez ajouter Zscaler Private Access (ZPA) à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Zscaler Private Access (ZPA)** dans la zone de recherche.
1. Sélectionnez **Zscaler Private Access (ZPA)** dans le panneau de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec Zscaler Private Access (ZPA) à l’aide d’un utilisateur de test appelé **Britta Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Zscaler Private Access (ZPA) associé.

Pour configurer et tester l’authentification unique Azure AD avec Zscaler Private Access (ZPA), suivez les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer Zscaler Private Access (ZPA)](#configure-zscaler-private-access-zpa)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Zscaler Private Access (ZPA)](#create-zscaler-private-access-zpa-test-user)** pour avoir un équivalent de Britta Simon dans Zscaler Private Access (ZPA) lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Zscaler Private Access (ZPA)** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la boîte de dialogue **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    1. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://samlsp.private.zscaler.com/auth/login?domain=<your-domain-name>`

    1. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL : `https://samlsp.private.zscaler.com/auth/metadata`

    > [!NOTE]
    > La valeur du champ **URL de connexion** n’est pas réelle. Remplacez cette valeur par l’URL de connexion réelle. Contactez l’[équipe de support client de Zscaler Private Access (ZPA)](https://help.zscaler.com/zpa-submit-ticket) pour obtenir cette valeur. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

   ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer Zscaler Private Access (ZPA)** , copiez la ou les URL appropriées en fonction de vos besoins.

   ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="configure-zscaler-private-access-zpa"></a>Configurer Zscaler Private Access (ZPA)

1. Pour automatiser la configuration dans Zscaler Private Access (ZPA), vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Zscaler Private Access (ZPA)** pour être dirigé vers l’application Zscaler Private Access (ZPA). À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Zscaler Private Access (ZPA). Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 6.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement Zscaler Private Access (ZPA), ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Zscaler Private Access (ZPA) en tant qu’administrateur, puis effectuez les étapes suivantes :

4. En haut, cliquez sur **Administration** et accédez à la section **AUTHENTICATION** puis cliquez sur **IdP Configuration** .

    ![Administrateur Zscaler Private Access - Administration](./media/zscalerprivateaccess-tutorial/tutorial-zscaler-private-access-administration.png)

5. Dans le coin supérieur droit, cliquez sur **Ajouter une configuration IdP** . 

    ![Administrateur Zscaler Private Access - idp](./media/zscalerprivateaccess-tutorial/tutorial-zscaler-private-access-idp.png)

6. Sur la **page Ajouter une configuration IdP** , procédez comme suit :
 
    ![Administrateur Zscaler Private Access - Sélection](./media/zscalerprivateaccess-tutorial/tutorial-zscaler-private-access-select.png)

    a. Cliquez sur **Sélectionner fichier** pour charger le fichier de métadonnées téléchargé depuis Azure AD dans le champ **Charger fichier de métadonnée IdP** .

    b. Les **métadonnées IdP** sont lu à partir d’Azure AD et tous les champs sont renseignés, comme montré ci-dessous.

    ![Administrateur Zscaler Private Access - Configuration](./media/zscalerprivateaccess-tutorial/config.png)

    c. Sélectionnez votre domaine dans le champ **Domaines** .
    
    d. Cliquez sur **Enregistrer** .

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Zscaler Private Access (ZPA).

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Zscaler Private Access (ZPA)** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-zscaler-private-access-zpa-test-user"></a>Créer un utilisateur de test Zscaler Private Access (ZPA)

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans Zscaler Private Access (ZPA). Collaborez avec [l’équipe de support technique Zscaler Private Access (ZPA)](https://help.zscaler.com/zpa-submit-ticket) pour ajouter les utilisateurs dans la plate-forme Zscaler Private Access (ZPA).

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Quand vous sélectionnez la vignette Zscaler Private Access (ZPA) dans le volet d’accès, vous devez être connecté automatiquement à l’application Zscaler Private Access (ZPA) pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)