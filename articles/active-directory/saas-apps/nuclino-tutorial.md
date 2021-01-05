---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Nuclino | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Nuclino.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/17/2019
ms.author: jeedes
ms.openlocfilehash: 4307838df56fb2dc1cdf6ee4044b8ce8de164227
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92518600"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-nuclino"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Nuclino

Dans ce tutoriel, vous allez apprendre à intégrer Nuclino à Azure Active Directory (Azure AD). Quand vous intégrez Nuclino à Azure AD, vous pouvez :

* Contrôler qui a accès à Nuclino dans Azure AD.
* Permettre à vos utilisateurs de se connecter automatiquement à Nuclino avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Nuclino pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Nuclino prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité** .
* Nuclino prend en charge l’approvisionnement d’utilisateurs **Juste-à-temps** .

## <a name="adding-nuclino-from-the-gallery"></a>Ajout de Nuclino à partir de la galerie

Pour configurer l’intégration de Nuclino dans Azure AD, vous devez ajouter Nuclino depuis la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Nuclino** dans la zone de recherche.
1. Sélectionnez **Nuclino** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-nuclino"></a>Configurer et tester l’authentification unique Azure AD pour Nuclino

Configurez et testez l’authentification unique Azure AD avec Nuclino pour un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Nuclino associé.

Pour configurer et tester l’authentification unique Azure AD avec Nuclino, effectuez les modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Nuclino](#configure-nuclino-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Nuclino](#create-nuclino-test-user)** pour avoir dans Nuclino un équivalent de B.Simon lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Nuclino** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://api.nuclino.com/api/sso/<UNIQUE-ID>/metadata`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://api.nuclino.com/api/sso/<UNIQUE-ID>/acs`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur et l’URL de réponse réels à partir de la section **Authentification** , expliquée plus loin dans ce didacticiel.

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://app.nuclino.com/<UNIQUE-ID>/login`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeur, contactez l’[équipe du support Nuclino](mailto:contact@nuclino.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

6. Votre application Nuclino s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration Attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/edit-attribute.png)

7. En plus de ce qui précède, l’application Nuclino s’attend à ce que quelques attributs supplémentaires soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom |  Attribut source|
    | ---------------| --------- |
    | first_name | user.givenname |
    | last_name | user.surname |

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Nuclino** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Nuclino.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Nuclino** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-nuclino-sso"></a>Configurer l’authentification unique Nuclino

1. Pour automatiser la configuration dans Nuclino, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Nuclino** pour être dirigé vers l’application Nuclino. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Nuclino. Cette extension de navigateur configure automatiquement l’application pour vous et automatise les étapes 3 à 7.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement Nuclino, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Nuclino en tant qu’administrateur et effectuez les étapes suivantes :

4. Cliquez sur l’ **icône** .

    ![Capture d’écran montrant l’icône « Menu » sélectionnée en regard de « Azure AD SSO ».](./media/nuclino-tutorial/configure1.png)

5. Cliquez sur l’ **authentification unique Azure AD** et sélectionnez **Paramètres d’équipe** dans la liste déroulante.

    ![Capture d’écran montrant la liste déroulante « Azure AD SSO » avec l’élément « Team settings » sélectionné.](./media/nuclino-tutorial/configure2.png)

6. Sélectionnez **Authentification** à partir du volet de navigation gauche.

    ![Capture d’écran montrant l’option « Authentication » sélectionnée.](./media/nuclino-tutorial/configure3.png)

7. Dans la section **Authentication** , procédez comme suit :

    ![Configuration de Nuclino](./media/nuclino-tutorial/configure4.png)

    a. Sélectionnez **Authentification unique SAML (SSO)** .

    b. Copiez la valeur **URL ACS (vous devez la copier et la coller dans votre fournisseur d’authentification unique)** et collez-la dans la zone de texte **URL de réponse** de la section **Configuration SAML de base** dans le portail Azure.

    c. Copiez la valeur **ID d’entité (vous devez la copier et la coller dans votre fournisseur d’authentification unique)** et collez-la dans la zone de texte **Identificateur** de la section **Configuration SAML de base** dans le portail Azure.

    d. Dans la zone de texte **URL SSO** , collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.

    e. Dans la zone de texte **ID d’entité** , collez la valeur **Identificateur Azure AD** que vous avez copiée dans le portail Azure.

    f. Ouvrez votre **Certificat (Base64)** téléchargé dans le Bloc-notes. Copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **Certificat public** .

    g. Cliquez sur **ENREGISTRER LES MODIFICATIONS** .

### <a name="create-nuclino-test-user"></a>Créer un utilisateur de test Nuclino

Dans cette section, un utilisateur appelé B.Simon est créé dans Nuclino. Nuclino prend en charge l’approvisionnement d’utilisateurs juste-à-temps, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Nuclino, il en est créé un après l’authentification.

> [!Note]
> Si vous devez créer un utilisateur manuellement, contactez [l’équipe de support Nuclino](mailto:contact@nuclino.com).

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Nuclino dans le panneau d’accès doit vous connecter automatiquement à l’application Nuclino pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Nuclino avec Azure AD](https://aad.portal.azure.com/)