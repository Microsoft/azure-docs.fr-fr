---
title: 'Tutoriel : Intégration d’Azure Active Directory à Fidelity NetBenefits | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Fidelity NetBenefits.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/12/2019
ms.author: jeedes
ms.openlocfilehash: ac41bf89a3c2997fe0c179ef1a64787b5fb8d617
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92453584"
---
# <a name="tutorial-azure-active-directory-integration-with-fidelity-netbenefits"></a>Tutoriel : Intégration d’Azure Active Directory à Fidelity NetBenefits

Dans ce didacticiel, vous allez apprendre à intégrer Fidelity NetBenefits à Azure Active Directory (Azure AD).
L’intégration de Fidelity NetBenefits à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à Fidelity NetBenefits.
* Vous pouvez autoriser vos utilisateurs à se connecter automatiquement à Fidelity NetBenefits (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD à Fidelity NetBenefits, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Fidelity NetBenefits pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Fidelity NetBenefits prend en charge l’authentification unique initiée par le **fournisseur d’identité**

* Fidelity NetBenefits prend en charge l’attribution d’utilisateurs **Juste-à-temps**

## <a name="adding-fidelity-netbenefits-from-the-gallery"></a>Ajout de Fidelity NetBenefits à partir de la galerie

Pour configurer l’intégration de Fidelity NetBenefits à Azure AD, vous devez ajouter Fidelity NetBenefits à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Fidelity NetBenefits depuis la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Fidelity NetBenefits** , sélectionnez **Fidelity NetBenefits** dans le volet des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Fidelity NetBenefits dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec Fidelity NetBenefits, à l’aide d’un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur Fidelity NetBenefits associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec Fidelity NetBenefits, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Fidelity NetBenefits](#configure-fidelity-netbenefits-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer l’utilisateur de test Fidelity NetBenefits](#create-fidelity-netbenefits-test-user)** pour disposer dans Fidelity NetBenefits d’un équivalent à Britta Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Fidelity NetBenefits, effectuez les étapes suivantes :

1. Dans la page d’intégration de l’application **Fidelity NetBenefits** sur le [portail Azure](https://portal.azure.com/), sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Sur la page **Configurer l’authentification unique avec SAML** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL Fidelity NetBenefits](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant :

    Pour l’environnement de test : `urn:sp:fidelity:geninbndnbparts20:uat:xq1`

    Pour l’environnement de production : `urn:sp:fidelity:geninbndnbparts20`

    b. Dans la zone de texte **URL de réponse** , entrez une URL fournie par Fidelity lors de l’implémentation ou contactez votre responsable de service client Fidelity assigné.

5. Votre application Fidelity NetBenefits attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut, où **nameidentifier** est mappé avec **user.userprincipalname** . Comme l’application Fidelity NetBenefits attend le mappage de **nameidentifier** avec **employeeid** , ou de toute autre revendication applicable à votre organisation en tant que **nameidentifier** , vous devez modifier le mappage d’attributs en cliquant sur l’icône **Modifier** , ce qui vous permet de changer le mappage d’attributs.

    ![image](common/edit-attribute.png)

    >[!Note]
    >Fidelity NetBenefits prend en charge la fédération statique et dynamique. Par statique, nous entendons que l’application n’utilisera pas l’attribution d’utilisateurs juste-à-temps d’après SAML. Par dynamique, nous entendons qu’elle prendra en charge l’attribution d’utilisateurs juste-à-temps. Pour l’utiliser, les clients doivent ajouter d’autres revendications dans Azure AD comme la date de naissance de l’utilisateur, etc. Ces détails sont fournis par votre **responsable de service client Fidelity** assigné et doivent activer la fédération dynamique pour votre instance.

6. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

7. Dans la section **Configurer Fidelity NetBenefits** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-fidelity-netbenefits-single-sign-on"></a>Configurer l’authentification unique Fidelity NetBenefits

Pour configurer l’authentification unique côté **Fidelity NetBenefits** , vous devez envoyer le **XML des métadonnées de fédération** téléchargé et les URL appropriées copiées à partir du portail Azure à l’ [équipe du support technique de Fidelity NetBenefits](mailto:SSOMaintenance@fmr.com). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon\@domainedevotreentreprise.extension** .  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Fidelity NetBenefits.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **Fidelity NetBenefits** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Fidelity NetBenefits** .

    ![Lien Fidelity NetBenefits dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-fidelity-netbenefits-test-user"></a>Créer l’utilisateur de test Fidelity NetBenefits

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans Fidelity NetBenefits. Dans le cadre d’une fédération statique, veuillez collaborer avec votre **responsable de service client Fidelity** assigné pour créer des utilisateurs dans la plateforme Fidelity NetBenefits. Ces utilisateurs doivent être créés et activés avant d’utiliser l’authentification unique.

Dans le cadre d’une fédération dynamique, les utilisateurs sont créés à l’aide de l’attribution juste-à-temps. Pour l’utiliser, les clients doivent ajouter d’autres revendications dans Azure AD comme la date de naissance de l’utilisateur, etc. Ces détails sont fournis par votre **responsable de service client Fidelity** assigné et doivent activer la fédération dynamique pour votre instance.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Fidelity NetBenefits dans le volet d’accès doit vous connecter automatiquement à l’application Fidelity NetBenefits pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)