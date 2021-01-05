---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Euromonitor Passport | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Euromonitor Passport.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/31/2019
ms.author: jeedes
ms.openlocfilehash: 422b22c265b6df353b86ff19db7c42f86bce8835
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92453886"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-euromonitor-passport"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Euromonitor Passport

Dans ce tutoriel, vous allez apprendre à intégrer Euromonitor Passport à Azure Active Directory (Azure AD). En intégrant Euromonitor Passport à Azure AD, vous pouvez :

* Contrôler, dans Azure AD, qui a accès à Euromonitor Passport
* Permettre aux utilisateurs de se connecter automatiquement à Euromonitor Passport avec leur compte Azure AD
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Euromonitor Passport pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Euromonitor Passport prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité**

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-euromonitor-passport-from-the-gallery"></a>Ajout d’Euromonitor Passport à partir de la galerie

Pour configurer l’intégration d’Euromonitor Passport dans Azure AD, vous devez ajouter Euromonitor Passport à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **Euromonitor Passport** dans la zone de recherche.
1. Sélectionnez **Euromonitor Passport** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-euromonitor-passport"></a>Configurer et tester l’authentification unique Azure AD pour Euromonitor Passport

Configurez et testez l’authentification unique Azure AD auprès d’Euromonitor Passport avec un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Euromonitor Passport associé.

Pour configurer et tester l’authentification unique Azure AD auprès d’Euromonitor Passport, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Euromonitor Passport](#configure-euromonitor-passport-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Euromonitor Passport](#create-euromonitor-passport-test-user)** pour avoir un équivalent de B.Simon dans Euromonitor Passport, associé à sa représentation dans Azure AD.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Euromonitor Passport** , recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , l’utilisateur n’a rien à faire, car l’application est déjà intégrée à Azure.

1. Si vous souhaitez configurer l’application en mode lancée par le **fournisseur de services** , vous devez contacter l’ [équipe du support technique Euromonitor Passport](mailto:passport.support@euromonitor.com) afin d’obtenir l’URL de connexion. Une fois que vous aurez obtenu l’URL de connexion, cliquez sur **Définir des URL supplémentaires**  et effectuez les étapes suivantes :

    Collez la valeur d’URL de connexion fournie par l’équipe de support Euromonitor Passport dans la zone de texte URL de connexion.

1. Cliquez sur **Enregistrer**.

1. L’application Euromonitor Passport s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application Euromonitor Passport s’attend à ce que quelques attributs supplémentaires (présentés ci-dessous) soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source|
    | ---------------| --------- |
    | Valeur de nom d’identificateur | user.userprincipalname |

    > [!NOTE]
    > Les administrateurs clients peuvent ajouter/modifier des attributs conformément à leurs besoins.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Euromonitor Passport.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Euromonitor Passport**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

## <a name="configure-euromonitor-passport-sso"></a>Configurer l’authentification unique Euromonitor Passport

Pour configurer l’authentification unique côté **Euromonitor Passport** , vous devez envoyer l’ **URL des métadonnées de fédération de l’application** à l’ [équipe du support technique Euromonitor Passport](mailto:passport.support@euromonitor.com). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-euromonitor-passport-test-user"></a>Créer un utilisateur de test Euromonitor Passport

Dans cette section, vous allez créer un utilisateur appelé B.Simon dans Euromonitor Passport. Contactez l’[équipe du support technique Euromonitor Passport](mailto:passport.support@euromonitor.com) pour ajouter des utilisateurs à la plateforme Euromonitor Passport. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Euromonitor Passport dans le volet d’accès, vous devez vous connecter automatiquement à l’application Euromonitor Passport pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Euromonitor Passport avec Azure AD](https://aad.portal.azure.com/)