---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec Bizagi for Digital Process Automation | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Bizagi for Digital Process Automation.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/27/2020
ms.author: jeedes
ms.openlocfilehash: efbb8a9ca0d475939d7713fa6a6a4a8245aead90
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92457059"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-bizagi-for-digital-process-automation"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory avec Bizagi for Digital Process Automation

Dans ce tutoriel, vous allez apprendre à intégrer Bizagi for Digital Process Automation Services ou Server avec Azure AD (Azure Active Directory). Quand vous intégrez Bizagi for Digital Process Automation avec Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Bizagi for Digital Process Automation Services ou Server.
* Permettre à vos utilisateurs de se connecter automatiquement à un projet de Bizagi for Digital Process Automation Services ou Server avec leurs comptes Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/)
* Un projet Bizagi utilisant Automation Services ou Server 
* Disposer de vos propres certificats pour les signatures d’assertion SAML. Ces certificats doivent être générés au format p12 ou pfx
* Avoir un fichier de métadonnées au format XML généré à partir du projet Bizagi 

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un projet Bizagi à l’aide des services ou du serveur Automation.

* Bizagi for Digital Process Automation prend en charge l’authentification unique lancée par le **fournisseur de services**
* Une fois que vous avez configuré Bizagi for Digital Process Automation, vous pouvez appliquer des contrôles de session, qui protègent l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-bizagi-for-digital-process-automation-from-the-gallery"></a>Ajout de Bizagi for Digital Process Automation à partir de la galerie

Pour configurer l’intégration de Bizagi for Digital Process Automation avec Azure AD, vous devez ajouter Bizagi for Digital Process Automation à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , dans la zone de recherche, tapez **Bizagi for Digital Process Automation** .
1. Sélectionnez **Bizagi for Digital Process Automation** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-bizagi-for-digital-process-automation"></a>Configurer et tester l’authentification unique Azure AD pour Bizagi for Digital Process Automation

Configurez et testez l’authentification unique Azure AD avec Bizagi for Digital Process Automation à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans le projet Bizagi.

Pour configurer et tester l’authentification unique Azure AD avec Bizagi for Digital Process Automation, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Bizagi for Digital Process Automation](#configure-bizagi-for-digital-process-automation-sso)** pour configurer les paramètres d’authentification unique côté application.
    1. **[Créer un utilisateur de test Bizagi for Digital Process Automation](#create-bizagi-for-digital-process-automation-test-user)** pour avoir un équivalent de B.Simon dans Bizagi for Digital Process Automation qui soit lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Bizagi for Digital Process Automation** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Chargez le fichier de métadonnées Bizagi dans l’option **Charger le fichier de métadonnées** .
1. Examinez la configuration. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , tapez l’URL de votre projet Bizagi : `https://<COMPANYNAME>.bizagi.com/<PROJECTNAME>`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , tapez l’URL de votre projet Bizagi : `https://<COMPANYNAME>.bizagi.com/<PROJECTNAME>`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe du support technique de Bizagi for Digital Process Automation](mailto:jarvein.rivera@bizagi.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)
    
    Cette URL de métadonnées doit être inscrite dans les options d’authentification de votre projet Bizagi.
    
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Attributs utilisateur et revendications** afin de modifier l’identifiant utilisateur unique.
    
    Définissez l’identifiant utilisateur unique en tant que user.mail.

### <a name="create-an-azure-ad-test"></a>Créer un utilisateur de test Azure AD 

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez permettre à B.Simon d’utiliser l’authentification unique Azure en lui octroyant l’accès à Bizagi for Digital Process Automation.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Bizagi for Digital Process Automation** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-bizagi-for-digital-process-automation-sso"></a>Configurer l’authentification unique pour Bizagi for Digital Process Automation

Pour configurer l’authentification unique côté **Bizagi for Digital Process Automation** , vous devez envoyer l’ **URL des métadonnées de fédération d’application** à l’ [équipe du support technique de Bizagi for Digital Process Automation](mailto:jarvein.rivera@bizagi.com). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-bizagi-for-digital-process-automation-test-user"></a>Créer un utilisateur de test Bizagi for Digital Process Automation

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans Bizagi for Digital Process Automation. Collaborez avec l’[équipe du support technique de Bizagi for Digital Process Automation](mailto:jarvein.rivera@bizagi.com) pour ajouter les utilisateurs à la plateforme Bizagi for Digital Process Automation. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Bizagi for Digital Process Automation dans le volet d’accès, vous devez être connecté automatiquement au portail de Bizagi for Digital Process Automation pour lequel vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Bizagi for Digital Process Automation avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)