---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Veracode | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Veracode.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/10/2019
ms.author: jeedes
ms.openlocfilehash: a62863607798e7f64a74926ebea4f1a1d05cbd6c
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92517716"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-veracode"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Veracode

Dans ce tutoriel, vous allez apprendre à intégrer Veracode à Azure Active Directory (Azure AD). Quand vous intégrez Veracode à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Veracode.
* Permettre à vos utilisateurs de se connecter automatiquement à Veracode avec leur compte Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS (software as a service) à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Veracode pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. Veracode prend en charge l’authentification unique lancée par le fournisseur d’identité et l’attribution d’utilisateurs juste à temps.

## <a name="add-veracode-from-the-gallery"></a>Ajouter Veracode à partir de la galerie

Pour configurer l’intégration de Veracode à Azure AD, ajoutez Veracode à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire, ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez « Veracode » dans la zone de recherche.
1. Sélectionnez **Veracode** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-veracode"></a>Configurer et tester l’authentification unique Azure AD pour Veracode

Configurez et testez l’authentification unique Azure AD avec Veracode à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Veracode associé.

Pour configurer et tester l’authentification unique Azure AD avec Veracode, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Veracode](#configure-veracode-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Veracode](#create-veracode-test-user)** pour avoir un équivalent de B.Simon dans Veracode lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Veracode** , puis recherchez la section **Gérer** . Sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône de crayon pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Capture d’écran de Configurer l’authentification unique avec SAML, avec l’icône de crayon mise en évidence](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , l’application est préconfigurée et les URL nécessaires sont déjà préremplies avec Azure. Sélectionnez **Enregistrer** .

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (Base64)** . Sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Capture d’écran de la section Certificat de signature SAML, avec mise en évidence du lien Télécharger](common/certificatebase64.png)

1. Veracode s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jetons SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![Capture d’écran de la section Attributs et revendications de l’utilisateur](common/default-attributes.png)

1. Veracode s’attend également à ce que quelques attributs supplémentaires soient passés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source|
    | ---------------| --------------- |
    | firstname |User.givenname |
    | lastname |User.Surname |
    | email |User.mail |

1. Dans la section **Configurer Veracode** , copiez la ou les URL nécessaires en fonction de vos besoins.

    ![Capture d’écran de la section Configurer Veracode, avec mise en évidence des URL de configuration](common/copy-configuration-urls.png)

## <a name="configure-veracode-sso"></a>Configurer l’authentification unique Veracode

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise Veracode en tant qu’administrateur.

1. Dans le menu situé en haut, sélectionnez **Settings** (Paramètres) > **Admin** .
   
    ![Capture d’écran de l’administration de Veracode, avec mise en évidence de l’icône Settings et d’Admin](./media/veracode-tutorial/ic802911.png "Administration")

1. Sélectionnez l’onglet **SAML** .

1. Dans la section **Organization SAML Settings** , procédez comme suit :

    ![Capture d’écran de la section Organization SAML Settings (Paramètres SAML de l’organisation)](./media/veracode-tutorial/ic802912.png "Administration")

    a.  Dans la zone **Issuer** (Émetteur), collez la valeur de l’ **Identificateur Azure AD** copiée à partir du portail Azure.

    b. Pour **Assertion Signing Certificate** (Certificat de signature d’assertion), sélectionnez **Choose File** (Choisir un fichier) pour charger votre certificat téléchargé à partir du portail Azure.

    c. Pour **Self Registration** (Inscription automatique), sélectionnez **Enable Self Registration** (Activer l’inscription automatique).

1. Dans la section **Self Registration Settings** (Paramètre de l’inscription automatique), effectuez les étapes suivantes, puis sélectionnez **Save** (Enregistrer) :

    ![Capture d’écran de la section Self Registration Settings, avec mise en évidence de différentes options](./media/veracode-tutorial/ic802913.png "Administration")

    a. Pour **New User Activation** (Activation du nouvel utilisateur), sélectionnez **No Activation Required** (Aucune activation requise).

    b. Dans **User Data Updates** (Mises à jour des données utilisateur), sélectionnez **Preference Veracode User Data** (Données utilisateur Veracode de préférence).

    c. Dans **SAML Attribute Details** , sélectionnez les éléments suivants :
      * **Rôles d’utilisateur**
      * **Administrateur de la stratégie**
      * **Réviseur**
      * **Responsable de la sécurité**
      * **Responsable**
      * **Expéditeur**
      * **Créateur**
      * **Tous les types d’analyse**
      * **Appartenance aux équipes**
      * **Équipe par défaut**

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** >**Utilisateurs** > **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :

   1. Pour **Nom** , entrez `B.Simon`.  
   1. Entrez username@companydomain.extension comme **Nom d’utilisateur** . Par exemple : `B.Simon@contoso.com`.
   1. Sélectionnez **Afficher le mot de passe** et notez la valeur affichée.
   1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Veracode.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Veracode** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Capture d’écran de la section Gérer, avec Utilisateurs et groupes mis en évidence](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** . Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Utilisateurs et groupes** .

    ![Capture d’écran de la page Utilisateurs et groupes, avec Ajouter un utilisateur mis en évidence](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , à partir d’ **Utilisateurs** , sélectionnez **B.Simon** . Choisissez **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste. Choisissez **Sélectionner** au bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Affecter** .

### <a name="create-veracode-test-user"></a>Créer un utilisateur de test Veracode

Pour se connecter à Veracode, les utilisateurs Azure AD doivent y être attribués. Cette tâche étant automatisée, vous n’avez pas besoin de faire quoi que ce soit manuellement. Au besoin, les utilisateurs sont automatiquement créés lors de la première tentative d’authentification unique.

> [!NOTE]
> Vous pouvez utiliser n’importe quel outil ou API de création de compte d’utilisateur, fourni par Veracode, pour approvisionner des comptes d’utilisateur Azure AD.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous sélectionnez **Veracode** dans le volet d’accès, vous devez être connecté automatiquement à l’application Veracode pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [ Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory ](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Veracode avec Azure AD](https://aad.portal.azure.com/)