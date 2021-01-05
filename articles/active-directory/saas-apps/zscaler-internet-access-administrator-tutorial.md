---
title: "Tutoriel : Intégration d'Azure Active Directory à Zscaler Internet Access Administrator | Microsoft Docs"
description: Apprenez à configurer l'authentification unique entre Azure Active Directory et Zscaler Internet Access Administrator.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/17/2019
ms.author: jeedes
ms.openlocfilehash: d74057e32b6f16bdb6dae3d96ac46c5cc93571aa
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97609103"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-internet-access-administrator"></a>Tutoriel : Intégration d'Azure Active Directory à Zscaler Internet Access Administrator

Dans ce tutoriel, vous allez apprendre à intégrer Zscaler Internet Access Administrator à Azure Active Directory (Azure AD).
L'intégration de Zscaler Internet Access Administrator à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Zscaler Internet Access Administrator.
* Vous pouvez autoriser vos utilisateurs à se connecter automatiquement à Zscaler Internet Access Administrator (via l'authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l'intégration d'Azure AD à Zscaler Internet Access Administrator, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Zscaler Internet Access Administrator

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Zscaler Internet Access Administrator prend en charge l'authentification unique initiée par **IDP**

## <a name="adding-zscaler-internet-access-administrator-from-the-gallery"></a>Ajout de Zscaler Internet Access Administrator à partir de la galerie

Pour configurer l'intégration de Zscaler Internet Access Administrator à Azure AD, vous devez ajouter Zscaler Internet Access Administrator à votre liste d'applications SaaS gérées à partir de la galerie.

**Pour ajouter Zscaler Internet Access Administrator à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Zscaler Internet Access Administrator**, sélectionnez **Zscaler Internet Access Administrator** dans le panneau des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l'application.

     ![Zscaler Internet Access Administrator dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l'authentification unique d'Azure AD auprès de Zscaler Internet Access Administrator avec un utilisateur de test appelé **Britta Simon**.
Pour que l'authentification unique fonctionne, une relation doit être établie entre un utilisateur Azure AD et un utilisateur Zscaler Internet Access Administrator associé.

Pour configurer et tester l'authentification unique d'Azure AD auprès de Zscaler Internet Access Administrator, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l'authentification unique Zscaler Internet Access Administrator](#configure-zscaler-internet-access-administrator-single-sign-on)** pour configurer les paramètres de l'authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Zscaler Internet Access Administrator](#create-zscaler-internet-access-administrator-test-user)** pour avoir dans Zscaler Internet Access Administrator un équivalent de Britta Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l'authentification unique d'Azure AD auprès de Zscaler Internet Access Administrator, procédez comme suit :

1. Sur le [portail Azure](https://portal.azure.com/), accédez à la page d'intégration de l'application **Zscaler Internet Access Administrator** et sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Sur la page **Configurer l’authentification unique avec SAML**, cliquez sur le bouton **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Informations d'authentification unique dans Domaine et URL Zscaler Internet Access Administrator](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur**, entrez une URL répondant à vos besoins :

    | Identificateur |
    |------------|
    | `https://admin.zscaler.net` |
    | `https://admin.zscalerone.net` |
    | `https://admin.zscalertwo.net` |
    | `https://admin.zscalerthree.net` |
    | `https://admin.zscloud.net` |
    | `https://admin.zscalerbeta.net` |

    b. Dans la zone de texte **URL de réponse**, entrez une URL répondant à vos besoins :

    | URL de réponse |
    |-----------|
    | `https://admin.zscaler.net/adminsso.do` |
    | `https://admin.zscalerone.net/adminsso.do` |
    | `https://admin.zscalertwo.net/adminsso.do` |
    | `https://admin.zscalerthree.net/adminsso.do` |
    | `https://admin.zscloud.net/adminsso.do` |
    | `https://admin.zscalerbeta.net/adminsso.do` |

5. L'application Zscaler Internet Access Administrator attend les assertions SAML dans un format spécifique. Configurez les revendications suivantes pour cette application. Vous pouvez gérer les valeurs de ces attributs à partir de la section **Attributs utilisateur et revendications** dans la page d’intégration des applications. Sur la page **Configurer l'authentification unique avec SAML**, cliquez sur le bouton **Modifier** pour ouvrir la boîte de dialogue **Attributs utilisateur et revendications**.

    ![Le lien Attribut](./media/zscaler-internet-access-administrator-tutorial/tutorial_zscaler-internet_attribute.png)

6. Dans la section **Revendications des utilisateurs** de la boîte de dialogue **Attributs utilisateur**, configurez le jeton SAML comme sur l’image ci-dessus et procédez comme suit :

    | Nom  | Attribut source  |
    | ---------| ------------ |
    | Role | user.assignedroles |

    a. Cliquez sur le bouton **Ajouter une nouvelle revendication** pour ouvrir la boîte de dialogue **Gérer les revendications des utilisateurs**.

    ![Capture d’écran montrant User claims avec l’option permettant d’ajouter une nouvelle revendication.](./common/new-save-attribute.png)
    
    ![Capture d’écran montrant la boîte de dialogue Manage user claims où vous pouvez entrer les valeurs décrites.](./common/new-attribute-details.png)

    b. Dans la liste **Attribut de la source**, sélectionnez la valeur de l’attribut.

    c. Cliquez sur **OK**.

    d. Cliquez sur **Enregistrer**.

    > [!NOTE]
    > Cliquez [ici](../develop/active-directory-enterprise-app-role-management.md) pour savoir comment configurer un rôle dans Azure AD.

7. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

8. Dans la section **Configurer Zscaler Internet Access Administrator**, copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-zscaler-internet-access-administrator-single-sign-on"></a>Configurer l'authentification unique de Zscaler Internet Access Administrator

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre UI d’administrateur Zscaler Internet Access.

2. Accédez à **Administration > Administrator Management**, effectuez les étapes suivantes, puis cliquez sur Save :

    ![Capture d’écran montrant Administrator Management avec les options permettant d’activer l’authentification SAML, de charger un certificat SSL et de spécifier un émetteur.](./media/zscaler-internet-access-administrator-tutorial/AdminSSO.png "Administration")

    a. Cliquez sur **Enable SAML Authentication**.

    b. Cliquez sur **Charger** pour charger le certificat de signature SAML Azure que vous avez téléchargé depuis le portail Azure dans le **Certificat SSL public**.

    c. Si vous le souhaitez, pour renforcer la sécurité, ajoutez les détails de l’émetteur (dans le champ **Issuer**) afin de vérifier l’émetteur de la réponse SAML.

3. Dans l’UI de l’administrateur, effectuez les étapes suivantes :

    ![Capture d’écran montrant l’IU d’administration où vous pouvez effectuer les étapes.](./media/zscaler-internet-access-administrator-tutorial/ic800207.png)

    a. Passez la souris sur le menu **Activation** en bas à gauche.

    b. Cliquez sur **Activer**.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**, sélectionnez **Utilisateurs**, puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom**, entrez **BrittaSimon**.
  
    b. Dans le champ **Nom d’utilisateur**, tapez **brittasimon\@domainedevotreentreprise.extension**.  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l'authentification unique Azure en lui accordant l'accès à Zscaler Internet Access Administrator.

1. Sur le portail Azure, sélectionnez **Applications d'entreprise**, **Toutes les applications**, puis **Zscaler Internet Access Administrator**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, entrez et sélectionnez **Zscaler Internet Access Administrator**.

    ![Lien Zscaler Internet Access Administrator dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur**, puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-zscaler-internet-access-administrator-test-user"></a>Créer un utilisateur de test Zscaler Internet Access Administrator

L'objectif de cette section est de créer un utilisateur appelé Britta Simon dans Zscaler Internet Access Administrator. Zscaler Internet Access ne prend pas en charge le provisionnement juste-à-temps pour l’authentification unique des administrateurs. Vous devez créer manuellement un compte d’administrateur.
Pour savoir comment créer un compte d’administrateur, reportez-vous à la documentation de Zscaler :

https://help.zscaler.com/zia/adding-admins

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Zscaler Internet Access Administrator du panneau d’accès, vous devez être automatiquement connecté à l’UI d’administrateur Zscaler Internet Access pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)
