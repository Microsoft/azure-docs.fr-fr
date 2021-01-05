---
title: 'Didacticiel : Intégration d’Azure Active Directory avec TurboRater | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et TurboRater.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 3/8/2019
ms.author: jeedes
ms.openlocfilehash: 7483e7a9300088d07e85d238ad897522d59c5464
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92519382"
---
# <a name="tutorial-azure-active-directory-integration-with-turborater"></a>Didacticiel : Intégration d’Azure Active Directory à TurboRater

Dans ce didacticiel, vous allez découvrir comment intégrer TigerText à Azure Active Directory (Azure AD).

Cette intégration vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à TurboRater.
* Vous pouvez permettre à vos utilisateurs de se connecter automatiquement à TurboRater (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure.

Pour obtenir des détails sur l’intégration d’applications SaaS (software as a service) avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à TurboRater, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Un abonnement TurboRater pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

TurboRater prend en charge l’authentification unique lancée par le fournisseur d’identité.

## <a name="add-turborater-from-the-azure-marketplace"></a>Ajouter TurboRater à partir de la Place de marché Azure

Pour configurer l’intégration de TurboRater dans Azure AD, vous devez ajouter TurboRater, disponible dans la Place de marché Azure, à votre liste d’applications SaaS managées :

1. Connectez-vous au [portail Azure](https://portal.azure.com?azure-portal=true).
1. Sélectionnez **Azure Active Directory** dans le volet de gauche.

    ![Option Azure Active Directory](common/select-azuread.png)

1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .

    ![Option Applications d’entreprise](common/enterprise-applications.png)

1. Pour ajouter une nouvelle application, sélectionnez **+ Nouvelle application** dans la partie supérieure du volet.

    ![Option Nouvelle application](common/add-new-app.png)

1. Dans la zone Rechercher, entrez **TurboRater** . Dans les résultats de recherche, sélectionnez **TurboRater** , puis **Ajouter** pour ajouter l’application.

    ![TurboRater dans la liste des résultats](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Dans cette section, vous allez configurer et tester l’authentification unique Azure AD auprès de TurboRater, en ayant recours à un utilisateur de test appelé **B Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur TurboRater associé.

Pour configurer et tester l’authentification unique Azure AD avec TurboRater, vous devez suivre les indications des sections suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
1. **[Configurer l’authentification unique TurboRater](#configure-turborater-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Créer un utilisateur de test TurboRater](#create-a-turborater-test-user)** de sorte qu’il existe un utilisateur nommé B. Simon dans TurboRater qui soit lié à l’utilisateur Azure AD nommé B. Simon.
1. **[Tester l’authentification unique](#test-single-sign-on)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous activez l’authentification unique Azure AD dans le portail Azure.

Pour configurer l’authentification unique Azure AD auprès de TurboRater, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **TurboRater** , sélectionnez **Authentification unique** .

    ![Configurer l’option d’authentification unique](common/select-sso.png)

1. Dans le volet **Sélectionner une méthode d’authentification unique** , sélectionnez le mode **SAML/WS-Fed** pour activer l’authentification unique.

    ![Mode de sélection de l’authentification unique](common/select-saml-option.png)

1. Dans le volet **Configurer l’authentification unique avec SAML** , sélectionnez **Modifier** (icône de crayon) pour ouvrir le volet **Configuration SAML de base** .

    ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans le volet **Configuration SAML de base** , procédez comme suit :

    ![Informations d’authentification unique des URL et du domaine TurboRater](common/idp-intiated.png)

    1. Dans la zone **Identificateur (ID d’entité)** , entrez une URL :

       `https://www.itcdataservices.com`

    1. Dans la zone **URL de réponse (URL Assertion Consumer Service)** , tapez une URL à l’aide du modèle suivant :

       | Environnement | URL |
       | ---------------| --------------- |
       | Test  | `https://ratingqa.itcdataservices.com/webservices/imp/saml/login` |
       | En direct  | `https://www.itcratingservices.com/webservices/imp/saml/login` |

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez [l’équipe de support technique TurboRater](https://www.getitc.com/support). Vous pouvez aussi vous référer aux modèles figurant dans le volet **Configuration SAML de base** sur le portail Azure.

1. Dans le volet **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , sélectionnez **Télécharger** pour télécharger le fichier **XML de métadonnées de fédération** à partir des options données et enregistrez-le sur votre ordinateur.

    ![Option de téléchargement du fichier XML de métadonnées de fédération](common/metadataxml.png)

1. Dans la section **Configurer TurboRater** , copiez les URL dont vous avez besoin :

   * **URL de connexion**
   * **Identificateur Azure AD**
   * **URL de déconnexion**

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="configure-turborater-single-sign-on"></a>Configurer l’authentification unique TurboRater

Pour configurer l’authentification unique côté TurboRater, vous devez envoyer le XML des métadonnées de fédération téléchargé et les URL appropriées copiées à partir du portail Azure à l’[équipe du support technique TurboRater](https://www.getitc.com/support). L’équipe TurboRater vérifiera que la connexion avec l’authentification unique SAML est correctement configurée des deux côtés.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, créez un utilisateur de test nommé Britta Simon dans le portail Azure.

1. Dans le portail Azure, dans le volet gauche, sélectionnez **Azure Active Directory**   > **Utilisateurs** > **Tous les utilisateurs** .

    ![Options Utilisateurs et « Tous les utilisateurs »](common/users.png)

1. En haut de l’écran, sélectionnez **+ Nouvel utilisateur** .

    ![Option Nouvel utilisateur](common/new-user.png)

1. Dans le volet **Utilisateur** , effectuez les étapes suivantes :

    ![Volet Utilisateur](common/user-properties.png)

    1. Dans la zone **Nom** , entrez **BSimon** .
  
    1. Dans la zone **Nom d’utilisateur** , entrez **BSimon\@\<yourcompanydomain>.\<extension>** . Par exemple **BSimon\@contoso.com** .

    1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .

    1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à TurboRater.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications** > **TurboRater** .

    ![Volet Applications d’entreprise](common/enterprise-applications.png)

1. Dans la liste des applications, sélectionnez **TurboRater** .

    ![TurboRater dans la liste des applications](common/all-applications.png)

1. Dans le volet gauche, sous **GÉRER** , sélectionnez **Utilisateurs et groupes** .

    ![Option « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **+ Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans le volet **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

1. Dans le volet **Utilisateurs et groupes** , sélectionnez **B Simon** dans la liste **Utilisateurs** , puis choisissez **Sélectionner** au bas du volet.

1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans le volet **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste. Au bas du volet, choisissez **Sélectionner** .

1. Dans le volet **Ajouter une attribution** , sélectionnez **Attribuer** .

### <a name="create-a-turborater-test-user"></a>Créer un utilisateur de test TurboRater

Dans cette section, vous allez créer un utilisateur appelé B Simon dans TurboRater. Contactez l’[équipe du support technique TurboRater](https://www.getitc.com/support) pour ajouter B. Simon dans TurboRater. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

### <a name="test-single-sign-on"></a>Tester l’authentification unique

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à partir du portail Mes applications.

Quand vous sélectionnez **TurboRater** sur le portail Mes applications, vous devez être connecté automatiquement à l’abonnement TurboRater pour lequel vous avez configuré l’authentification unique. Pour plus d’informations sur le portail Mes applications, consultez [Accéder aux applications du portail Mes applications et les utiliser](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Liste de tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

* [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)