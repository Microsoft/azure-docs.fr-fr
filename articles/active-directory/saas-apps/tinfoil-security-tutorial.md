---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec TINFOIL SECURITY | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et TINFOIL SECURITY.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/16/2019
ms.author: jeedes
ms.openlocfilehash: 5c2ac2c7bb1b60c87075a1bc62241edab2fc310e
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92516289"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-tinfoil-security"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory avec TINFOIL SECURITY

Dans ce tutoriel, vous allez apprendre à intégrer TINFOIL SECURITY à Azure Active Directory (Azure AD). Quand vous intégrez TINFOIL SECURITY à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à TINFOIL SECURITY.
* Permettre à vos utilisateurs de se connecter automatiquement à TINFOIL SECURITY avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement TINFOIL SECURITY pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* TINFOIL SECURITY prend en charge l’authentification unique lancée par le **fournisseur d’identité**

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-tinfoil-security-from-the-gallery"></a>Ajout de TINFOIL SECURITY à partir de la galerie

Pour configurer l’intégration de TINFOIL SECURITY à Azure AD, vous devez ajouter TINFOIL SECURITY à votre liste d’applications SaaS gérées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **TINFOIL SECURITY** dans la zone de recherche.
1. Sélectionnez **TINFOIL SECURITY** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-tinfoil-security"></a>Configurer et tester l’authentification unique Azure AD pour TINFOIL SECURITY

Configurez et testez l’authentification unique Azure AD avec TINFOIL SECURITY pour un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur TINFOIL SECURITY associé.

Pour configurer et tester l’authentification unique Azure AD avec TINFOIL SECURITY, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique TINFOIL SECURITY](#configure-tinfoil-security-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test TINFOIL SECURITY](#create-tinfoil-security-test-user)** pour avoir un équivalent de B.Simon dans TINFOIL SECURITY lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Sur le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **TINFOIL SECURITY** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , l’application est préconfigurée et les URL nécessaires sont déjà préremplies avec Azure. L’utilisateur doit enregistrer la configuration en cliquant sur le bouton **Enregistrer** .

1. Votre application Visitly s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application Visitly s’attend à ce que quelques attributs supplémentaires (présentés ci-dessous) soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source |
    | ------------------- | -------------|
    | accountid | UXXXXXXXXXXXXX |

    > [!NOTE]
    > Les valeurs accountid vous seront expliquées plus loin dans le tutoriel.

1. Dans la section **Certificat de signature SAML** , cliquez sur le bouton **Modifier** pour ouvrir la boîte de dialogue **Certificat de signature SAML** .

    ![Modifier le certificat de signature SAML](common/edit-certificate.png)

1. Dans la section **Certificat de signature SAML** , copiez la **valeur de l’empreinte** et enregistrez-la sur votre ordinateur.

    ![Copier la valeur de l’empreinte](common/copy-thumbprint.png)

1. Dans la section **Configurer TINFOIL SECURITY** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez permettre à B.Simon d’utiliser l’authentification unique Azure en lui accordant l’accès à TINFOIL SECURITY.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste d’applications, sélectionnez **TINFOIL SECURITY** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-tinfoil-security-sso"></a>Configurer l’authentification unique TINFOIL SECURITY

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise TINFOIL SECURITY en tant qu’administrateur.

1. Dans la barre d’outils située en haut, cliquez sur **My Account** .

    ![Tableau de bord](./media/tinfoil-security-tutorial/ic798971.png "tableau de bord")

1. Cliquez sur **Sécurité** .

    ![Sécurité](./media/tinfoil-security-tutorial/ic798972.png "Sécurité")

1. Dans la page de configuration **Single Sign on** , procédez comme suit :

    ![Authentification unique](./media/tinfoil-security-tutorial/ic798973.png "Single Sign on")

    a. Sélectionnez **Enable SAML** .

    b. Cliquez sur **Manual Configuration** .

    c. Dans la zone de texte **SAML Post URL** , collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.

    d. Dans la zone de texte **Empreinte numérique du certificat SAML** , collez la valeur de l’ **empreinte** que vous avez copiée à partir de la section **Certificat de signature SAML** .
  
    e. Copiez la valeur de l’ **ID de votre compte** et collez-la dans la zone de texte **Attribut source** située sous la section **Attributs et revendications de l’utilisateur** sur le portail Azure.

    f. Cliquez sur **Enregistrer** .

### <a name="create-tinfoil-security-test-user"></a>Créer un utilisateur de test TINFOIL SECURITY

Pour se connecter à TINFOIL SECURITY, les utilisateurs d’Azure AD doivent être provisionnés dans TINFOIL SECURITY. Dans le cas de TINFOIL SECURITY, l’approvisionnement est une tâche manuelle.

**Pour configurer l’approvisionnement des utilisateurs, procédez comme suit :**

1. Si l’utilisateur fait partie d’un compte d’entreprise, vous devez [contacter l’équipe de support technique TINFOIL SECURITY](https://www.tinfoilsecurity.com/contact) pour obtenir le compte d’utilisateur créé.

1. Si l’utilisateur est un utilisateur normal de SaaS TINFOIL SECURITY, il peut ajouter un collaborateur à n’importe lequel de ses sites. Cela déclenche l’envoi, à l’adresse e-mail spécifiée, d’une invitation à créer un compte d’utilisateur TINFOIL SECURITY.

> [!NOTE]
> Vous pouvez utiliser n’importe quels outils ou API de création de compte d’utilisateur, fournis par TINFOIL SECURITY, pour approvisionner des comptes d’utilisateur Azure AD.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette TINFOIL SECURITY dans le volet d’accès, vous devez être connecté automatiquement à l’application TINFOIL SECURITY pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer TINFOIL SECURITY avec Azure AD](https://aad.portal.azure.com/)