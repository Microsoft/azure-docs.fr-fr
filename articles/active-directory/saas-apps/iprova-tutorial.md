---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à iProva | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et iProva.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/19/2020
ms.author: jeedes
ms.openlocfilehash: 717696053a742abae6756655a15416ac81221144
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92459745"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-iprova"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à iProva

Dans ce tutoriel, vous allez apprendre à intégrer iProva à Azure Active Directory (Azure AD). Quand vous intégrez iProva à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à iProva.
* Permettre à vos utilisateurs de se connecter automatiquement à iProva avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement iProva pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* iProva prend en charge l’authentification unique (SSO) initiée par le **fournisseur de services**

* Après avoir configuré iProva, vous pouvez appliquer le contrôle de session, qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-iprova-from-the-gallery"></a>Ajout d’iProva à partir de la galerie

Pour configurer l’intégration d’iProva à Azure AD, vous devez ajouter iProva, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **iProva** dans la zone de recherche.
1. Sélectionnez **iProva** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-iprova"></a>Configurer et tester l’authentification unique Azure AD pour iProva

Configurez et testez l’authentification unique Azure AD avec iProva à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur iProva associé.

Pour configurer et tester l’authentification unique Azure AD avec iProva, suivez les indications des sections ci-après :

1. **[Récupérer des informations de configuration à partir d’iProva](#retrieve-configuration-information-from-iprova)** pour préparer les étapes suivantes.
1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Créer un utilisateur de test iProva](#create-iprova-test-user)** pour avoir un équivalent de B.Simon dans iProva lié à la représentation Azure AD associée.
1. **[Configurer l’authentification unique iProva](#configure-iprova-sso)** pour configurer les paramètres de l’authentification unique côté application.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="retrieve-configuration-information-from-iprova"></a>Récupérer des informations de configuration à partir d’iProva

Dans cette section, vous allez récupérer des informations à partir d’iProva pour configurer l’authentification unique Azure AD.

1. Ouvrez un navigateur web et accédez à la page **SAML2 info** dans iProva, en utilisant le modèle d’URL suivant :

    ```https
    https://SUBDOMAIN.iprova.nl/saml2info
    https://SUBDOMAIN.iprova.be/saml2info
    ```

    ![Afficher la page d’informations SAML2 iProva](media/iprova-tutorial/iprova-saml2-info.png)

1. Laissez l’onglet du navigateur ouvert pendant que vous effectuez les étapes qui suivent dans un autre onglet de navigateur.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **iProva** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , effectuez les étapes suivantes :

    a. Renseignez le champ **URL de connexion** avec la valeur affichée derrière l’étiquette **Sign-on URL** dans la page **iProva SAML2 info** . Cette page est toujours ouverte dans votre autre onglet de navigateur.

    b. Renseignez le champ **Identificateur** avec la valeur affichée derrière l’étiquette **EntityID** dans la page **iProva SAML2 info** . Cette page est toujours ouverte dans votre autre onglet de navigateur.

    c. Renseignez le champ **URL de réponse** avec la valeur affichée derrière l’étiquette **Reply URL** dans la page **iProva SAML2 info** . Cette page est toujours ouverte dans votre autre onglet de navigateur.

1. L’application iProva s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration de vos attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application iProva s’attend à ce que quelques attributs supplémentaires, indiqués ci-après, soient passés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source| Espace de noms  |
    | ---------------| -------- | -----|
    | `samaccountname` | `user.onpremisessamaccountname`| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims`|

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur le bouton Copier pour copier l’ **URL des métadonnées de fédération d’application** , puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

## <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe** .
   1. Cliquez sur **Créer** .

## <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à iProva.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **iProva** (iProva).
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="create-iprova-test-user"></a>Créer un utilisateur de test iProva

1. Connectez-vous à iProva avec le compte **Administrateur** .

2. Ouvrez le menu **Atteindre** .

3. Sélectionnez **Application management** (Gestion des applications).

4. Sélectionnez **Users** (Utilisateurs) dans le panneau **Users and user groups** (Utilisateurs et groupes d’utilisateurs).

5. Sélectionnez **Ajouter** .

6. Dans la zone de texte **Username** (Nom d’utilisateur), entrez le nom d’un utilisateur, par exemple `B.Simon@contoso.com`.

7. Dans la zone **Full name** (Nom complet), entrez un nom complet d’utilisateur, comme **B.Simon** .

8. Sélectionnez l’option **No password (use single sign-on)** (Aucun mot de passe (utiliser l’authentification unique)).

9. Dans la zone de texte **Email address** (Adresse e-mail), entrez l’adresse e-mail de l’utilisateur, par exemple `B.Simon@contoso.com`.

10. Faites défiler jusqu’à la fin de la page, puis sélectionnez **Finish** (Terminer).

## <a name="configure-iprova-sso"></a>Configurer l’authentification unique iProva

1. Connectez-vous à iProva avec le compte **Administrateur** .

2. Ouvrez le menu **Atteindre** .

3. Sélectionnez **Application management** (Gestion des applications).

4. Sélectionnez **General** (Général) dans le panneau **System settings** (Paramètres système).

5. Sélectionnez **Modifier** .

6. Faites défiler jusqu’à **Contrôle d’accès** .

    ![Paramètres de contrôle d’accès iProva](media/iprova-tutorial/iprova-accesscontrol.png)

7. Recherchez le paramètre **Users are automatically logged on with their network accounts** (Les utilisateurs sont automatiquement connectés avec leur compte réseau) et remplacez-le par **Yes, authentication via SAML** (Oui, authentification via SAML). Des options supplémentaires s’affichent maintenant.

8. Sélectionnez **Set up** (Configurer).

9. Sélectionnez **Suivant** .

10. iProva vous demande si vous souhaitez télécharger des données de fédération à partir d’une URL ou les charger à partir d’un fichier. Sélectionnez l’option **From URL** (À partir d’une URL).

    ![Télécharger des métadonnées Azure AD](media/iprova-tutorial/iprova-download-metadata.png)

11. Collez l’URL de métadonnées que vous avez enregistrée lors de la dernière étape de la section « Configurer l’authentification unique Azure AD ».

12. Sélectionnez le bouton en forme de flèche pour télécharger les métadonnées à partir d’Azure AD.

13. Une fois le téléchargement terminé, le message de confirmation **Valid Federation Data file downloaded** (Fichier de données de fédération valide téléchargé) s’affiche.

14. Sélectionnez **Suivant** .

15. Ignorez l’option **Test login** (Tester la connexion) pour l’instant et sélectionnez **Suivant** .

16. Dans la zone de liste déroulante **Claim to use** (Revendication à utiliser), sélectionnez **windowsaccountname** .

17. Sélectionnez **Terminer** .

18. Vous revenez à présent à l’écran **Edit general settings** (Modifier les paramètres généraux). Faites défiler jusqu’en bas de la page et sélectionnez **OK** pour enregistrer votre configuration.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette iProva dans le panneau d’accès doit vous connecter automatiquement à l’application iProva pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer iProva avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger iProva avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)