---
title: 'Didacticiel : intégration d’Azure Active Directory à SAP Business ByDesign | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SAP Business ByDesign.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/18/2019
ms.author: jeedes
ms.openlocfilehash: 6d04601cb11e57f50c82680077793aef1b816bfa
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895123"
---
# <a name="tutorial-azure-active-directory-integration-with-sap-business-bydesign"></a>Didacticiel : Intégration d’Azure Active Directory à SAP Business ByDesign

Dans ce didacticiel, vous allez apprendre à intégrer SAP Business ByDesign à Azure Active Directory (Azure AD).
L’intégration de SAP Business ByDesign à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez déterminer qui a accès à SAP Business ByDesign.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à SAP Business ByDesign (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD avec SAP Business ByDesign, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/)
* Abonnement SAP Business ByDesign pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* SAP Business ByDesign prend en charge l’authentification unique lancée par le **fournisseur de services**

## <a name="adding-sap-business-bydesign-from-the-gallery"></a>Ajouter SAP Business ByDesign à partir de la galerie

Pour configurer l’intégration de SAP Business ByDesign à Azure AD, vous devez ajouter SAP Business ByDesign depuis la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter SAP Business ByDesign à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, saisissez **SAP Business ByDesign.** , sélectionnez **SAP Business ByDesign.** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![SAP Business ByDesign dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous configurez et testez l’authentification unique Azure AD avec SAP Business ByDesign pour un utilisateur de test appelé **Britta Simon**.
Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans SAP Business ByDesign.

Pour configurer et tester l’authentification unique Azure AD avec SAP Business ByDesign, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique SAP Business ByDesign](#configure-sap-business-bydesign-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test SAP Business ByDesign](#create-sap-business-bydesign-test-user)** pour avoir dans SAP Business ByDesign un équivalent de Britta Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec SAP Business ByDesign, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **SAP Business ByDesign** , sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans la section Domaine et URL SAP Business ByDesign](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<servername>.sapbydesign.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<servername>.sapbydesign.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez [l’équipe de support technique SAP Business ByDesign](https://www.sap.com/products/cloud-analytics.support.html). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. L’application SAP Business ByDesign attend les assertions SAML dans un format spécifique. Configurez les revendications suivantes pour cette application. Vous pouvez gérer les valeurs de ces attributs à partir de la section **Attributs utilisateur** sur la page d’intégration des applications. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur le bouton **Modifier** pour ouvrir la boîte de dialogue **Attributs utilisateur**.

    ![Capture d’écran montrant la boîte de dialogue « User Attributes » avec le bouton « Edit » sélectionné.](common/edit-attribute.png)

6. Cliquez sur l’icône **Modifier** pour changer la **Valeur de nom d’identificateur**.

    ![Capture d’écran montrant la page « Attributs et revendications de l’utilisateur » avec l’icône « Modifier » sélectionnée](media/sapbusinessbydesign-tutorial/mail-prefix1.png)

7. Dans la section **Gérer les revendications des utilisateurs** , effectuez les étapes suivantes :  ![image](media/sapbusinessbydesign-tutorial/mail-prefix2.png)

    a. Sélectionnez **Transformation** dans le champ **Source**.

    b. Dans la liste déroulante **Transformation** , sélectionnez **ExtractMailPrefix()** .

    c. Dans la liste déroulante **Paramètre 1** , sélectionnez l’attribut utilisateur à utiliser pour votre implémentation. Par exemple, si vous souhaitez utiliser EmployeeID comme identificateur d’utilisateur unique et que vous avez stocké la valeur d’attribut dans ExtensionAttribute2, sélectionnez user.extensionattribute2.

    d. Cliquez sur **Enregistrer**.

8. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

9. Dans la section **Configurer SAP Business ByDesign** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-sap-business-bydesign-single-sign-on"></a>Configurer l’authentification unique SAP Business ByDesign

1. Connectez-vous à votre portail SAP Business ByDesign avec des droits d’administrateur.

2. Accédez à **Application and User Management Common Task (Tâche courante d’application et de gestion des utilisateurs)** et cliquez sur l’onglet **Identity Provider (Fournisseur d’identité)** .

3. Cliquez sur **New Identity Provider** (Nouveau fournisseur d’identité) et sélectionnez le fichier XML de métadonnées que vous avez téléchargé à partir du portail Azure. En important les métadonnées, le système charge automatiquement le certificat de signature ainsi que le certificat de chiffrement requis.

    ![Capture d’écran montrant la page « Configure Single Sign-On » avec l’onglet « Identity Provider » sélectionné et l’action « New Identity Provider » sélectionnée](./media/sapbusinessbydesign-tutorial/tutorial_sapbusinessbydesign_54.png)

4. Pour inclure **l’URL Assertion Consumer Service** dans la requête SAML, sélectionnez **Include Assertion Consumer Service URL (Inclure l’URL Assertion Consumer Service)** .

5. Cliquez sur **Activate Single Sign-On** (Activer l’authentification unique).

6. Enregistrez vos modifications.

7. Cliquez sur l’onglet **My System** (Mon système).

    ![Capture d’écran montrant la page « Configure Single Sign-On » avec l’onglet « My System » sélectionné](./media/sapbusinessbydesign-tutorial/tutorial_sapbusinessbydesign_52.png)

8. Dans la zone de texte **Azure AD Sign On URL** (URL de connexion Azure AD), collez l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    ![Configure Single Sign-On](./media/sapbusinessbydesign-tutorial/tutorial_sapbusinessbydesign_53.png)

9. Précisez si l’employé peut choisir manuellement entre l’authentification à l’aide d’un ID d’utilisateur/mot de passe ou l’authentification unique en cliquant sur **Manual Identity Provider Selection** (Sélection manuelle du fournisseur d’identité).

10. Dans la section **SSO URL** (URL SSO), spécifiez l’URL devant être utilisée par l’employé pour se connecter au système.
    Dans la liste déroulante URL Sent to Employee (URL envoyée à l’employé), vous pouvez choisir entre les options suivantes :

    **Non-SSO URL**

    Le système envoie uniquement l’URL du système normale à l’employé. L’employé ne peut pas se connecter à l’aide de l’authentification unique ; il doit utiliser un mot de passe ou un certificat à la place.

    **URL SSO**

    Le système envoie uniquement l’URL SSO à l’employé. L’employé peut se connecter à l’aide de l’authentification unique. La demande d’authentification est redirigée via l’IdP.

    **Sélection automatique**

    Si l’authentification unique n’est pas activée, le système envoie l’URL du système normale à l’employé. Si l’authentification unique est activée, le système vérifie si l’employé possède un mot de passe. Le cas échéant, les URL SSO et les URL non SSO sont envoyées à l’employé. Toutefois, si l’employé ne possède aucun mot de passe, seule l’URL SSO est envoyée à l’employé.

11. Enregistrez vos modifications.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs**.

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon**.
  
    b. Dans le champ **Nom d’utilisateur** , tapez `brittasimon@yourcompanydomain.extension`. Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à SAP Business ByDesign.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **SAP Business ByDesign**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **SAP Business ByDesign**.

    ![Lien correspondant à SAP Business ByDesign dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

### <a name="create-sap-business-bydesign-test-user"></a>Créer un utilisateur de test pour SAP Business ByDesign

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans SAP Business ByDesign. Veuillez contacter [l’équipe de prise en charge des clients de SAP Business ByDesign](https://www.sap.com/products/cloud-analytics.support.html) pour ajouter des utilisateurs sur la plateforme SAP Business ByDesign. 

> [!NOTE]
> Assurez-vous que la valeur NameID correspond au champ de nom d’utilisateur dans la plateforme SAP Business ByDesign.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette SAP Business ByDesign dans le volet d’accès, vous devez être connecté automatiquement à l’application SAP Business ByDesign pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)