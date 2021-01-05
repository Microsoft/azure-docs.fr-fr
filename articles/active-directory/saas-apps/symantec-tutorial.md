---
title: 'Didacticiel : Intégration d’Azure Active Directory à Symantec Web Security Service (WSS) | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Symantec Web Security Service (WSS).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/25/2018
ms.author: jeedes
ms.openlocfilehash: 230f7fd9c62f657ce8ab893db2256808dce9a7ba
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92518362"
---
# <a name="tutorial-azure-active-directory-integration-with-symantec-web-security-service-wss"></a>Didacticiel : Intégration d’Azure Active Directory à Symantec Web Security Service (WSS)

Dans ce didacticiel, vous allez apprendre à intégrer votre compte Symantec Web Security Service (WSS) à votre compte Azure Active Directory (Azure AD) afin que WSS puisse authentifier un utilisateur final approvisionné dans Azure AD à l’aide de l’authentification SAML et appliquer les règles de stratégie au niveau de l’utilisateur ou du groupe.

L’intégration de Symantec Web Security Service (WSS) à Azure AD vous fait bénéficier des avantages suivants :

- Gérer tous les groupes et utilisateurs finaux utilisés par votre compte WSS à partir de votre portail Azure AD.

- Autoriser les utilisateurs finaux à s’authentifier sur WSS à l’aide de leurs informations d’identification Azure AD.

- Activer l’application des règles de stratégie au niveau de l’utilisateur et du groupe définies dans votre compte WSS.

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à Symantec Web Security Service (WSS), vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Symantec Web Security Service (WSS) pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* Symantec Web Security Service (WSS) prend en charge l’authentification unique initiée par le **fournisseur d’identité**

## <a name="adding-symantec-web-security-service-wss-from-the-gallery"></a>Ajout de Symantec Web Security Service (WSS) à partir de la galerie

Pour configurer l’intégration de Symantec Web Security Service (WSS) à Azure AD, vous devez ajouter Symantec Web Security Service (WSS) à partir de la galerie à votre liste d’applications SaaS gérées.

**Pour ajouter Symantec Web Security Service (WSS) à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory** .

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez l’option **Toutes les applications** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Symantec Web Security Service (WSS)** , sélectionnez **Symantec Web Security Service (WSS)** dans le volet des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

     ![Symantec Web Security Service (WSS) dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD auprès de Symantec Web Security Service (WSS) avec un utilisateur de test appelé **Britta Simon** .
Pour que l’authentification unique fonctionne, une relation entre un utilisateur Azure AD et un utilisateur Symantec Web Security Service (WSS) associé doit être établie.

Pour configurer et tester l’authentification unique Azure AD auprès de Symantec Web Security Service (WSS), vous avez besoin de suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **Configurer l’authentification unique Symantec Web Security Service (WSS)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Symantec Web Security Service (WSS)](#create-symantec-web-security-service-wss-test-user)** pour avoir dans Symantec Web Security Service (WSS) un équivalent de Britta Simon lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD avec Symantec Web Security Service (WSS), effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Symantec Web Security Service (WSS)** , sélectionnez **Authentification unique** .

    ![Lien Configurer l’authentification unique](common/select-sso.png)

2. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** afin d’activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

3. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

4. Dans la boîte de dialogue **Configuration SAML de base** , effectuez les étapes suivantes :

    ![Informations d’authentification unique Domaine et URL Symantec Web Security Service (WSS)](common/idp-intiated.png)

    a. Dans la zone de texte **Identificateur** , tapez une URL : `https://saml.threatpulse.net:8443/saml/saml_realm`

    b. Dans la zone de texte **URL de réponse** , tapez l’URL : `https://saml.threatpulse.net:8443/saml/saml_realm/bcsamlpost`

    > [!NOTE]
    > Contactez l’ [équipe de support technique de Symantec Web Security Service (WSS)](https://www.symantec.com/contact-us) si les valeurs **Identificateur** et **URL de réponse** ne fonctionnent pas pour une raison quelconque. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Sur la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** en fonction des options définies selon vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

### <a name="configure-symantec-web-security-service-wss-single-sign-on"></a>Configurer l’authentification unique Symantec Web Security Service (WSS)

Pour configurer l’authentification unique côté Symantec Web Security Service (WSS), consultez la documentation en ligne de WSS. Le **XML de métadonnées de fédération** devra être importé dans le portail WSS. Contactez l’[équipe de support Symantec Web Security Service (WSS)](https://www.symantec.com/contact-us) si vous avez besoin d’aide pour configurer le portail WSS.

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

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Symantec Web Security Service (WSS).

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis sélectionnez **Symantec Web Security Service (WSS)** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, tapez et sélectionnez **Symantec Web Security Service (WSS)** .

    ![Lien Symantec Web Security Service (WSS) dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-symantec-web-security-service-wss-test-user"></a>Créer un utilisateur de test Symantec Web Security Service (WSS)

Dans cette section, vous créez un utilisateur appelé Britta Simon dans Symantec Web Security Service (WSS). Le nom d’utilisateur final correspondante peut être créé manuellement dans le portail WSS ou vous pouvez attendre que les utilisateurs/groupes approvisionnés dans Azure AD se synchronisent dans le portail WSS après quelques minutes (environ 15 minutes). Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique. L’adresse IP publique de l’ordinateur de l’utilisateur final, qui sera utilisée pour parcourir les sites web, doit également être configurée dans le portail Symantec Web Security Service (WSS).

> [!NOTE]
> [Cliquez ici](https://www.bing.com/search?q=my+ip+address&qs=AS&pq=my+ip+a&sc=8-7&cvid=29A720C95C78488CA3F9A6BA0B3F98C5&FORM=QBLH&sp=1) pour obtenir l’adresse IP publique de votre ordinateur.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la fonctionnalité d’authentification unique maintenant que vous avez configuré votre compte WSS pour utiliser votre système Azure AD pour l’authentification SAML.

Une fois que vous avez configuré votre navigateur web pour le trafic de proxy WSS, lorsque vous ouvrez votre navigateur web et que vous essayez d’accéder à un site, vous serez redirigé vers la page d’ouverture de session Azure. Entrez les informations d’identification de l’utilisateur final de test qui a été approvisionné dans Azure AD (autrement dit, BrittaSimon) et le mot de passe correspondant. Une fois authentifié, vous pourrez accéder au site web que vous avez choisi. Si vous devez créer une règle de stratégie côté WSS pour empêcher BrittaSimon d’accéder à un site spécifique, vous devriez voir la page de blocage WSS lorsque vous tentez d’accéder à ce site en tant que BrittaSimon.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)