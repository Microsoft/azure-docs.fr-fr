---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à NS1 SSO for Azure | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et NS1 SSO for Azure.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/12/2020
ms.author: jeedes
ms.openlocfilehash: a7fad0af26d15bc77a52d55309c594937ba3be13
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92518617"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-ns1-sso-for-azure"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à NS1 SSO for Azure

Dans ce tutoriel, vous allez apprendre à intégrer NS1 SSO for Azure à Azure Active Directory (Azure AD). Quand vous intégrez NS1 SSO for Azure avec Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à NS1 SSO for Azure.
* Permettre à vos utilisateurs de se connecter automatiquement à NS1 SSO for Azure avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central, le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS (software as a service) à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement NS1 SSO for Azure pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* NS1 SSO for Azure prend en charge l’authentification unique lancée par le fournisseur de services et le fournisseur d’identité.
* Après avoir configuré NS1 SSO for Azure, vous pouvez appliquer le contrôle de session. Cela protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).


## <a name="add-ns1-sso-for-azure-from-the-gallery"></a>Ajout de NS1 SSO for Azure à partir de la galerie

Pour configurer l’intégration de NS1 SSO for Azure dans Azure AD, vous devez ajouter NS1 SSO for Azure à partir de la galerie à la liste des applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire, ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **NS1 SSO for Azure** dans la zone de recherche.
1. Sélectionnez **NS1 SSO for Azure** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-ns1-sso-for-azure"></a>Configurer et tester l’authentification unique NS1 SSO for Azure

Configurez et testez l’authentification unique Azure AD avec NS1 SSO for Azure pour un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, établissez un lien entre un utilisateur Azure AD et l’utilisateur NS1 SSO for Azure associé.

Voici la procédure générale pour configurer et tester l’authentification unique Azure AD pour NS1 SSO for Azure :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.

    a. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.

    b. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique NS1 SSO for Azure](#configure-ns1-sso-for-azure-sso)** pour configurer les paramètres de l’authentification unique côté application.

    a. **[Créer un utilisateur de test NS1 SSO for Azure](#create-an-ns1-sso-for-azure-test-user)** pour avoir un équivalent de B. Simon dans NS1 SSO for Azure. Cet équivalent est lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Sur le [Portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **NS1 SSO for Azure** , recherchez la section **Gérer** . Sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône de crayon pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Capture d’écran de la page Configurer l’authentification unique avec SAML, avec l’icône de crayon mise en évidence](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous voulez configurer l’application en mode lancé par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez l’URL suivante : `https://api.nsone.net/saml/metadata`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://api.nsone.net/saml/sso/<ssoid>`

1. Sélectionnez **Définir des URL supplémentaires** , puis effectuez l’étape suivante si vous voulez configurer l’application en mode initié par le **fournisseur de services**  :

    Dans la zone de texte **URL de connexion** , tapez l’URL suivante : `https://my.nsone.net/#/login/sso`

    > [!NOTE]
    > La valeur de l’URL de réponse n’est pas réelle. Mettez à jour la valeur avec l’URL de réponse réelle. Pour obtenir cette valeur, contactez l’[équipe de support client NS1 SSO for Azure](mailto:techops@nsone.net). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. L’application NS1 SSO for Azure attend les assertions SAML dans un format spécifique. Configurez les revendications suivantes pour cette application. Vous pouvez gérer les valeurs de ces attributs à partir de la section **Attributs utilisateur et revendications** dans la page d’intégration d’application. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône de crayon pour ouvrir la boîte de dialogue **Attributs utilisateur** .

    ![Capture d’écran de la section Attributs utilisateur et revendications, avec l’icône de crayon mise en évidence](./media/ns1-sso-for-azure-tutorial/attribute-edit-option.png)

1. Sélectionnez le nom de l’attribut pour modifier la revendication.

    ![Capture d’écran de la section Attributs utilisateur et revendications, avec le nom de l’attribut mis en évidence](./media/ns1-sso-for-azure-tutorial/attribute-claim-edit.png)

1. Sélectionnez **Transformation** .

    ![Capture d’écran de la section Gérer la revendication, avec l’option Transformation mise en évidence](./media/ns1-sso-for-azure-tutorial/prefix-edit.png)

1. Dans la section **Gérer la transformation** , effectuez les étapes suivantes :

    ![Capture d’écran de la section Gérer la transformation avec différents champs mis en évidence](./media/ns1-sso-for-azure-tutorial/prefix-added.png)

    1. Sélectionnez **ExactMailPrefix()** comme **Transformation** .

    1. Sélectionnez **user.userprincipalname** comme **Paramètre 1** .

    1. Sélectionnez **Ajouter** .

    1. Sélectionnez **Enregistrer** .

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , sélectionnez le bouton Copier. Cela copie l’ **URL des métadonnées de fédération d’application** et l’enregistre sur votre ordinateur.

    ![Capture d’écran du certificat de signature SAML, avec le bouton Copier mis en évidence](common/copy-metadataurl.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon sur le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :

   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur figurant dans le champ **Mot de passe** .
   1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à NS1 SSO for Azure.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **NS1 SSO for Azure** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Capture d’écran de la section Gérer, avec Utilisateurs et groupes mis en évidence](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** . Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Utilisateurs et groupes** .

    ![Capture d’écran de la page Utilisateurs et groupes, avec Ajouter un utilisateur mis en évidence](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B.Simon** dans la liste des utilisateurs. Ensuite, choisissez le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste. Ensuite, choisissez le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Affecter** .

## <a name="configure-ns1-sso-for-azure-sso"></a>Configurer l’authentification unique NS1 SSO for Azure

Pour configurer l’authentification unique côté NS1 SSO for Azure, vous devez envoyer l’URL des métadonnées de fédération d’application à l’[équipe du support technique NS1 SSO for Azure](mailto:techops@nsone.net). Celle-ci configure ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-an-ns1-sso-for-azure-test-user"></a>Créer un utilisateur de test NS1 SSO for Azure

Dans cette section, vous allez créer un utilisateur appelé B.Simon dans NS1 SSO for Azure. Collaborez avec l’équipe du support technique NS1 SSO for Azure pour ajouter les utilisateurs à la plateforme NS1 SSO for Azure. Vous ne pouvez pas utiliser l’authentification unique avant d’avoir créé et activé les utilisateurs.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous sélectionnez la vignette NS1 SSO for Azure dans le volet d’accès, vous devez être connecté automatiquement à l’application NS1 SSO for Azure pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations, consultez [Présentation du volet d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer NS1 SSO for Azure avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)