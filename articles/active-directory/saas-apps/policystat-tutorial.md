---
title: 'Tutoriel : Intégration d’Azure Active Directory avec PolicyStat | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et PolicyStat.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/28/2019
ms.author: jeedes
ms.openlocfilehash: 4d6274f3e1356a8ff8a997e830f0ebee43f50cff
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92893014"
---
# <a name="tutorial-azure-active-directory-integration-with-policystat"></a>Tutoriel : Intégration d’Azure Active Directory avec PolicyStat

Dans ce didacticiel, vous allez apprendre à intégrer PolicyStat avec Azure Active Directory (Azure AD).
L’intégration de PolicyStat avec Azure AD vous offre les avantages suivants :

* Vous pouvez contrôler dans Azure AD qui a accès à PolicyStat.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à PolicyStat (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD avec PolicyStat, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Un abonnement PolicyStat pour lequel l’authentification unique est activée

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* PolicyStat prend en charge l’authentification unique lancée par le **fournisseur de services**

* PolicyStat prend en charge le provisionnement d’utilisateurs **juste-à-temps**

## <a name="adding-policystat-from-the-gallery"></a>Ajout de PolicyStat à partir de la galerie

Pour configurer l’intégration de PolicyStat à Azure AD, vous devez ajouter PolicyStat à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter PolicyStat à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **PolicyStat** , sélectionnez **PolicyStat** dans le panneau de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![PolicyStat dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec PolicyStat sur un utilisateur de test nommé **Britta Simon**.
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et l’utilisateur PolicyStat associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD avec PolicyStat, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique PolicyStat](#configure-policystat-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
4. **[Créer un utilisateur de test PolicyStat](#create-policystat-test-user)** pour obtenir un équivalent de Britta Simon dans PolicyStat lié à la représentation Azure AD associée.
5. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec PolicyStat, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **PolicyStat** , sélectionnez **Authentification unique**.

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL PolicyStat](common/sp-identifier.png)

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<companyname>.policystat.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<companyname>.policystat.com/saml2/metadata/`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique PolicyStat](http://www.policystat.com/support/). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

4. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

5. Votre application PolicyStat s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jetons SAML. La capture d’écran suivante montre la liste des attributs par défaut. Cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Attributs d’utilisateur**.

    ![Capture d’écran montrant la boîte de dialogue « Attributs utilisateur » avec l’icône « Modifier » sélectionnée.](common/edit-attribute.png)

6. En plus de ce qui précède, l’application PolicyStat s’attend à ce que quelques attributs supplémentaires soient repassés dans la réponse SAML. Dans la section **Revendications des utilisateurs** de la boîte de dialogue **Attributs utilisateur** , effectuez les étapes suivantes pour ajouter le jeton SAML comme indiqué dans le tableau ci-dessous :

    | Nom | Attribut source |
    |------------------- | -------------------- |
    | uid | ExtractMailPrefix([mail]) |

    a. Cliquez sur le bouton **Ajouter une nouvelle revendication** pour ouvrir la boîte de dialogue **Gérer les revendications des utilisateurs**.
    
    ![Capture d’écran montrant la section « User claims » avec les actions « Add new claim » et « Save » mis en évidence.](common/new-save-attribute.png)

    ![Capture d’écran montrant la boîte de dialogue « Gérer les revendications des utilisateurs » avec les zones de texte « Nom », « Transformation » et « Paramètre » mises en évidence, et le bouton « Enregistrer » sélectionné.](./media/policystat-tutorial/attribute01.png)

    b. Dans la zone de texte **Attribut** , indiquez le nom d’attribut pour cette ligne.

    c. Laissez le champ **Espace de noms** vide.

    d. Sélectionner Source comme **Transformation**.

    e. Dans la liste **Transformation** , tapez la valeur d’attribut affichée pour cette ligne.
    
    f. Dans la liste **Paramètre 1** , tapez la valeur d’attribut affichée pour cette ligne.

    g. Cliquez sur **Enregistrer**.

7. Dans la section **Configurer PolicyStat** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="configure-policystat-single-sign-on"></a>Configurer l’authentification unique PolicyStat

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise PolicyStat en tant qu’administrateur.

2. Cliquez sur l’onglet **Admin** , puis sur **Single Sign-On Configuration** dans le volet de navigation de gauche.
   
    ![Menu Administrateur](./media/policystat-tutorial/ic808633.png "Menu Administrateur")

3. Cliquez sur **Your IDP Metadata** , puis, dans la section **Your IDP Metadata** , procédez comme suit :
   
    ![Capture d’écran montrant l’action « Your IDP Metadata » sélectionnée](./media/policystat-tutorial/ic808636.png "Single Sign-On Configuration")
   
    a. Ouvrez votre fichier de métadonnées téléchargé, copiez son contenu, puis collez-le dans la zone de texte **Your Identity Provider Metadata**.

    b. Cliquez sur **Enregistrer les modifications**.

4. Cliquez sur **Configure Attributes** , puis, dans la section **Configure Attributes** , procédez comme suit :
   
    a. Dans la zone de texte **Username Attribute** , entrez **uid**.

    b. Dans la zone de texte **First Name Attribute** , tapez votre revendication d’attribut de prénom obtenue à partir d’Azure **`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname`** .

    c. Dans la zone de texte **Last Name Attribute** , tapez votre revendication d’attribut de nom obtenue à partir d’Azure **`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`** .

    d. Dans la zone de texte **Email Attribute** , tapez votre revendication d’attribut d’e-mail obtenue à partir d’Azure **`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`** .

    e. Cliquez sur **Enregistrer les modifications**.

5. Dans la section **Setup** , sélectionnez **Enable Single Sign-on Integration**.
   
    ![Configuration de l’authentification unique](./media/policystat-tutorial/ic808634.png "Single Sign-On Configuration")


### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser votre propre compte à utiliser l’authentification unique Azure en lui accordant l’accès à PolicyStat.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis **PolicyStat**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **PolicyStat**.

    ![Lien PolicyStat dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez votre compte dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

### <a name="create-policystat-test-user"></a>Créer un utilisateur de test PolicyStat

Dans cette section, un utilisateur nommé Britta Simon est créé dans PolicyStat. PolicyStat prend en charge le provisionnement d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans PolicyStat, il en est créé un après l’authentification.

>[!NOTE]
>Vous pouvez utiliser n’importe quel outil ou API de création de compte utilisateur, fourni par PolicyStat, pour approvisionner des comptes utilisateur Azure AD.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette PolicyStat dans le volet d’accès, vous devez être connecté automatiquement à l’application PolicyStat pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)