---
title: 'Tutoriel : Intégration d’Azure Active Directory à TigerConnect Secure Messenger | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et TigerConnect Secure Messenger.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/21/2020
ms.author: jeedes
ms.openlocfilehash: 6831e96ebd8fd2db51d640ff3d93955b1a9f1477
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96182153"
---
# <a name="tutorial-azure-active-directory-integration-with-tigerconnect-secure-messenger"></a>Tutoriel : Intégration d’Azure Active Directory à TigerConnect Secure Messenger

Dans ce didacticiel, vous allez apprendre à intégrer TigerConnect Secure Messenger à Azure Active Directory (Azure AD).

L’intégration de TigerConnect Secure Messenger dans Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à TigerConnect Secure Messenger.
* Vous pouvez autoriser vos utilisateurs à se connecter automatiquement à TigerConnect Secure Messenger (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure.

Pour obtenir des détails sur l’intégration d’applications SaaS (software as a service) avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à TigerConnect Secure Messenger, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Un abonnement TigerConnect Secure Messenger pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test et intégrer TigerConnect Secure Messenger à Azure AD.

* TigerConnect Secure Messenger prend en charge l’authentification unique lancée par le **fournisseur de services**
* Après avoir configuré TigerConnect Secure Messenger, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-tigerconnect-secure-messenger-from-the-gallery"></a>Ajout de TigerConnect Secure Messenger à partir de la galerie

Pour configurer l’intégration de TigerConnect Secure Messenger à Azure AD, vous devez ajouter TigerConnect Secure Messenger, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **TigerConnect Secure Messenger** dans la zone de recherche.
1. Sélectionnez **TigerConnect Secure Messenger** dans le panneau de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD auprès de TigerConnect Secure Messenger pour un utilisateur de test appelé **Britta Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur TigerConnect Secure Messenger associé.

Pour configurer et tester l’authentification unique Azure AD avec TigerConnect Secure Messenger, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique TigerConnect Secure Messenger](#configure-tigerconnect-secure-messenger-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test TigerConnect Secure Messenger](#create-a-tigerconnect-secure-messenger-test-user)** pour avoir un équivalent de Britta Simon dans TigerConnect Secure Messenger, lié à l’utilisateur Azure AD nommé Britta Simon.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD auprès de TigerConnect Secure Messenger, effectuez les étapes suivantes :

1. Sur le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **TigerConnect Secure Messenger**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    1. Dans la zone **URL de connexion**, entrez une URL :

       `https://home.tigertext.com`

    1. Dans la zone de texte **Identificateur (ID d’entité)**, saisissez une URL au format suivant :

       `https://saml-lb.tigertext.me/v1/organization/<instance ID>`

    > [!NOTE]
    > La valeur de **Identificateur (ID d’entité)** n’est pas réelle. Mettez-la à jour avec l’identificateur réel. Contactez l’[équipe du support technique de TigerConnect Secure Messenger](mailto:prosupport@tigertext.com) pour obtenir cette valeur. Vous pouvez aussi vous référer aux modèles figurant dans le volet **Configuration SAML de base** sur le portail Azure.

1. Dans le volet **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, sélectionnez **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** à partir des options données et enregistrez-le sur votre ordinateur.

    ![Option de téléchargement du fichier XML de métadonnées de fédération](common/metadataxml.png)

1. Dans la section **Configurer TigerConnect Secure Messenger**, copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à TigerConnect Secure Messenger.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **TigerConnect Secure Messenger**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-tigerconnect-secure-messenger-sso"></a>Configurer l’authentification unique TigerConnect Secure Messenger

Pour configurer l’authentification unique côté TigerConnect Secure Messenger, vous devez envoyer le XML des métadonnées de fédération téléchargé et les URL copiées appropriées du Portail Azure à l’[équipe du support technique TigerConnect Secure Messenger](mailto:prosupport@tigertext.com). L’équipe TigerConnect Secure Messenger vérifiera que la connexion avec l’authentification unique SAML est correctement configurée des deux côtés.

## <a name="create-a-tigerconnect-secure-messenger-test-user"></a>Créer un utilisateur de test TigerConnect Secure Messenger

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans TigerConnect Secure Messenger. Contactez l’[équipe du support technique TigerConnect Secure Messenger](mailto:prosupport@tigertext.com) pour ajouter Britta Simon à la plateforme TigerConnect Secure Messenger. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à partir du portail Mes applications.

Quand vous sélectionnez **TigerConnect Secure Messenger** sur le portail Mes applications, vous devez être connecté automatiquement à l’abonnement TigerConnect Secure Messenger pour lequel vous avez configuré l’authentification unique. Pour plus d’informations sur le portail Mes applications, consultez [Accéder aux applications du portail Mes applications et les utiliser](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Listes des tutoriels pour intégrer des applications SaaS à Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayez TigerConnect Secure Messenger avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)