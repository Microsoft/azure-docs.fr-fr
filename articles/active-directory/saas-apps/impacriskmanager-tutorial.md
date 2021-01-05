---
title: 'Didacticiel : Intégration d’Azure Active Directory à IMPAC Risk Manager | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et IMPAC Risk Manager.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/14/2019
ms.author: jeedes
ms.openlocfilehash: 1462e815b5f045ab9d52bdea59b78b12ff77c19b
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92460289"
---
# <a name="tutorial-azure-active-directory-integration-with-impac-risk-manager"></a>Didacticiel : Intégration d’Azure Active Directory à IMPAC Risk Manager

L’objectif de ce didacticiel est de vous apprendre à intégrer IMPAC Risk Manager à Azure Active Directory (Azure AD).
L’intégration d’IMPAC Risk Manager à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à IMPAC Risk Manager.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à IMPAC Risk Manager (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à IMPAC Risk Manager, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement IMPAC Risk Manager pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* IMPAC Risk Manager prend en charge l’authentification unique démarrée par **le fournisseur de services et le fournisseur d’identité**

## <a name="adding-impac-risk-manager-from-the-gallery"></a>Ajout d’IMPAC Risk Manager à partir de la galerie

Pour configurer l’intégration d’IMPAC Risk Manager à Azure AD, vous devez ajouter IMPAC Risk Manager à votre liste d’applications SaaS gérées, à partir de la galerie.

**Pour ajouter IMPAC Risk Manager à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **IMPAC Risk Manager** , sélectionnez **IMPAC Risk Manager** dans le volet des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![IMPAC Risk Manager dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous configurez et vous testez l’authentification unique Azure AD avec IMPAC Risk Manager, via un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, l’utilisateur Azure AD et l’utilisateur IMPAC Risk Manager, doivent être associés.

Pour configurer et tester l’authentification unique d’Azure AD avec IMPAC Risk Manager, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique IMPAC Risk Manager](#configure-impac-risk-manager-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test IMPAC Risk Manager](#create-impac-risk-manager-test-user)** pour avoir un équivalent de Britta Simon dans IMPAC Risk Manager, lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec IMPAC Risk Manager, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **IMPAC Risk Manager** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur d’identité** , effectuez les étapes suivantes :

    ![Capture d’écran montrant Configuration SAML de base, où vous pouvez entrer l’identificateur, l’URL de réponse, et sélectionner Enregistrer.](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une valeur fournie par IMPAC

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant :

    | Environnement | Modèle d’URL |
    | ---------------|--------------- |
    | Pour la production |`https://www.riskmanager.co.nz/DotNet/SSOv2/AssertionConsumerService.aspx?client=<ClientSuffix>`|
    | Pour la préproduction et la formation  |`https://staging.riskmanager.co.nz/DotNet/SSOv2/AssertionConsumerService.aspx?client=<ClientSuffix>`|
    | Pour le développement  |`https://dev.riskmanager.co.nz/DotNet/SSOv2/AssertionConsumerService.aspx?client=<ClientSuffix>`|
    | Pour l’assurance qualité |`https://QA.riskmanager.co.nz/DotNet/SSOv2/AssertionConsumerService.aspx?client=<ClientSuffix>`|
    | Pour les tests |`https://test.riskmanager.co.nz/DotNet/SSOv2/AssertionConsumerService.aspx?client=<ClientSuffix>`|
    | | |

5. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    ![Capture d’écran montrant Définir des URL supplémentaires, où vous pouvez entrer une URL de connexion.](common/metadata-upload-additional-signon.png)

    Dans la zone de texte **URL d’authentification** , tapez une URL au format suivant :

    | Environnement | Modèle d’URL |
    | ---------------|--------------- |
    | Pour la production |`https://www.riskmanager.co.nz/SSOv2/<ClientSuffix>`|
    | Pour la préproduction et la formation  |`https://staging.riskmanager.co.nz/SSOv2/<ClientSuffix>`|
    | Pour le développement  |`https://dev.riskmanager.co.nz/SSOv2/<ClientSuffix>`|
    | Pour l’assurance qualité |`https://QA.riskmanager.co.nz/SSOv2/<ClientSuffix>`|
    | Pour les tests |`https://test.riskmanager.co.nz/SSOv2/<ClientSuffix>`|
    | | |

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique d’IMPAC Risk Manager](mailto:rmsupport@Impac.co.nz). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

6. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

7. Dans la section **Configurer IMPAC Risk Manager** , copiez la ou les URL appropriées correspondant à vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-impac-risk-manager-single-sign-on"></a>Configurer l’authentification unique IMPAC Risk Manager

Pour configurer l’authentification unique côté **IMPAC Risk Manager** , vous devez envoyer le **certificat (Base64)** téléchargé et les URL appropriées, copiées à partir du portail Azure, à l’ [équipe du support technique d’IMPAC Risk Manager](mailto:rmsupport@Impac.co.nz). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon@yourcompanydomain.extension**  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez Britta Simon à utiliser l’authentification unique d’Azure en lui accordant l’accès à IMPAC Risk Manager.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis sélectionnez **IMPAC Risk Manager** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **IMPAC Risk Manager** .

    ![Lien IMPAC Risk Manager dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-impac-risk-manager-test-user"></a>Créer un utilisateur de test IMPAC Risk Manager

Dans cette section, vous créez un utilisateur appelé Britta Simon dans IMPAC Risk Manager. Collaborez avec [l’équipe de support technique IMPAC Risk Manager](mailto:rmsupport@Impac.co.nz) pour ajouter les utilisateurs dans la plateforme IMPAC Risk Manager. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette IMPAC Risk Manager dans le panneau d’accès doit vous connecter automatiquement à l’application IMPAC Risk Manager pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)