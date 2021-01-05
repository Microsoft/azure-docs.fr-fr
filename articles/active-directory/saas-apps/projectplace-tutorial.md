---
title: 'Tutoriel : Intégration d’Azure Active Directory à Projectplace | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Projectplace.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/29/2019
ms.author: jeedes
ms.openlocfilehash: 508a3ca02ec6c0475e78d16691f6597e1a389f97
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92511849"
---
# <a name="tutorial-integrate-projectplace-with-azure-active-directory"></a>Tutoriel : Intégrer Projectplace à Azure Active Directory

Dans ce didacticiel, vous allez apprendre à intégrer Projectplace à Azure Active Directory (Azure AD). Lorsque vous intégrez Projectplace à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Projectplace.
* Autoriser les utilisateurs à se connecter automatiquement à Projectplace avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.
* Les utilisateurs peuvent être provisionnés automatiquement dans Projectplace.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Projectplace pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. Projectplace prend en charge l’authentification unique lancée par **le fournisseur d’identité et le fournisseur de service** ainsi que le provisionnement d’utilisateurs **juste-à-temps** .

## <a name="adding-projectplace-from-the-gallery"></a>Ajout de Projectplace à partir de la galerie

Pour configurer l’intégration de Projectplace avec Azure AD, vous devez ajouter Projectplace à votre liste d’applications SaaS gérées, à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Projectplace** dans la zone de recherche.
1. Sélectionnez **Projectplace** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec Projectplace pour un utilisateur de test nommé **B. Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Projectplace associé.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec Projectplace, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer Projectplace](#configure-projectplace)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Projectplace](#create-projectplace-test-user)** pour avoir dans Projectplace un équivalent de B. Simon lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Projectplace** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode lancé par le **fournisseur de services** , l’application est préconfigurée et les URL nécessaires sont préremplies avec Azure. L’utilisateur doit enregistrer la configuration en cliquant sur le bouton **Enregistrer** .

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL : `https://service.projectplace.com`

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur l’icône **Copier** pour copier l’ **URL des métadonnées de fédération d’application** appropriée et enregistrez-la dans le Bloc-notes.

   ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

1. Dans la section **Configurer Projectplace** , copiez la ou les URL appropriées, selon vos besoins.

   ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="configure-projectplace"></a>Configurer Projectplace

Pour configurer l’authentification unique côté **Projectplace** , vous devez envoyer l’ **URL des métadonnées de fédération d’application** copiée depuis le portail Azure à l’ [équipe du support technique Projectplace](https://success.planview.com/Projectplace/Support). Celle-ci vérifiera que l’authentification unique SAML est configurée correctement des deux côtés.

>[!NOTE]
>La configuration de l’authentification unique doit être effectuée par [l’équipe du support technique Projectplace](https://success.planview.com/Projectplace/Support). Vous recevez une notification dès qu’elle est terminée. 

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B. Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `BrittaSimon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Projectplace.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Projectplace** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

### <a name="create-projectplace-test-user"></a>Créer un utilisateur de test Projectplace

>[!NOTE]
>Vous pouvez ignorer cette étape si le provisionnement d’utilisateurs est activé dans Projectplace. Vous pouvez demander à l’[équipe du support technique Projectplace](https://success.planview.com/Projectplace/Support) d’activer le provisionnement. Après cela, les utilisateurs seront créés dans Projectplace au moment de la première connexion.

Pour autoriser les utilisateurs Azure AD à se connecter à Projectplace, vous devez les ajouter dans Projectplace. Vous devez les ajouter manuellement.

**Pour créer un compte d’utilisateur, effectuez les étapes suivantes :**

1. Connectez-vous au site d’entreprise **Projectplace** en tant qu’administrateur.

2. Accédez à **Contacts** , puis cliquez sur **Membres**  :
   
    ![Accéder à People, puis sélectionner Members](./media/projectplace-tutorial/ic790228.png "Personnes")

3. Sélectionnez **Ajouter un membre**  :
   
    ![Sélectionner Add Member](./media/projectplace-tutorial/ic790232.png "Ajouter des membres")

4. Dans la section **Ajouter un membre** , procédez comme suit.
   
    ![Section Add Member](./media/projectplace-tutorial/ic790233.png "New Members")
   
    1. Dans la zone **Nouveaux membres** , entrez l'adresse e-mail d'un compte Azure AD valide que vous souhaitez ajouter.
   
    1. Sélectionnez **Envoyer** .

   Un message électronique contenant un lien pour confirmer le compte avant qu’il ne soit activé est envoyé au titulaire du compte Azure AD.

>[!NOTE]
>Pour ajouter des comptes d’utilisateur Azure AD, vous pouvez utiliser n’importe quel autre outil ou API de création de compte d’utilisateur fourni par Projectplace.


### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Quand vous sélectionnez la vignette Projectplace dans le volet d’accès, vous devez être connecté automatiquement à l’application Projectplace pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)