---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Adobe Creative Cloud | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Adobe Creative Cloud.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/14/2020
ms.author: jeedes
ms.openlocfilehash: d526a3cef25ad88b1565a71d1f488ab70f28b8c4
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92308980"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-adobe-creative-cloud"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Adobe Creative Cloud

> [!NOTE]
> Cet article décrit la configuration basée sur SAML personnalisée d’Adobe Admin Console pour Azure Active Directory (Azure AD). Pour les nouvelles configurations, nous vous recommandons d’utiliser [Azure AD Connector](https://helpx.adobe.com/enterprise/using/sso-setup-azure.html). Configurable en quelques minutes, Azure AD Connector permet d’écourter les processus de revendication de domaine, de configuration de l’authentification unique et de synchronisation des utilisateurs.

Dans ce tutoriel, vous allez apprendre à intégrer Adobe Creative Cloud à Azure Active Directory (Azure AD). En intégrant Adobe Creative Cloud à Azure AD, vous pouvez :

* Contrôler, dans Azure AD, qui a accès à Adobe Creative Cloud
* Permettre aux utilisateurs de se connecter automatiquement à Adobe Creative Cloud avec leur compte Azure AD
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Adobe Creative Cloud pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Adobe Creative Cloud prend en charge l’authentification unique lancée par le **fournisseur de services**
* Après avoir configuré Adobe Creative Cloud, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-adobe-creative-cloud-from-the-gallery"></a>Ajout de Adobe Creative Cloud à partir de la galerie

Pour configurer l’intégration de Adobe Creative Cloud à Azure AD, vous devez ajouter Adobe Creative Cloud de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Adobe Creative Cloud** dans la zone de recherche.
1. Sélectionnez **Adobe Creative Cloud** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-adobe-creative-cloud"></a>Configurer et tester l’authentification unique Azure AD pour Adobe Creative Cloud

Configurez et testez l’authentification unique Azure AD auprès d’Adobe Creative Cloud avec un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Adobe Creative Cloud associé.

Pour configurer et tester l’authentification unique (SSO) Azure AD auprès d’Adobe Creative Cloud, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Adobe Creative Cloud](#configure-adobe-creative-cloud-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Adobe Creative Cloud](#create-adobe-creative-cloud-test-user)** pour avoir un équivalent de B.Simon dans Adobe Creative Cloud lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Adobe Creative Cloud**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion**, tapez une URL : `https://adobe.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://www.okta.com/saml2/service-provider/<token>`

    > [!NOTE]
    > La valeur de l'identificateur n'est pas réelle. Suivez les instructions de l’étape 4 de la section **Configurer l’authentification unique Adobe Cloud**. Vous pouvez ouvrir le **fichier XML de métadonnées de fédération** et obtenir la valeur d’ID d’entité, et la placer en tant que valeur d’identificateur dans la configuration Azure AD. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. L’application Adobe Creative Cloud attend les assertions SAML dans un certain format, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration des attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/edit-attribute.png)

1. En plus de ce qui précède, l’application Adobe Creative Cloud s’attend à ce que quelques attributs supplémentaires (présentés ci-dessous) soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source|
    |----- | --------- |
    | FirstName | user.givenname |
    | LastName | user.surname |
    | E-mail | user.mail |

    > [!NOTE]
    > Les utilisateurs doivent avoir une licence Microsoft 365 ExO pour que la valeur de la revendication d’e-mail soit renseignée dans la réponse SAML.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération**, puis sélectionnez **Télécharger** pour télécharger le fichier de métadonnées XML et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Adobe Creative Cloud**, copiez l’URL ou les URL appropriées selon vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Adobe Creative Cloud.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Adobe Creative Cloud**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-adobe-creative-cloud-sso"></a>Configurer l’authentification unique Adobe Creative Cloud

1. Dans une autre fenêtre de navigateur web, connectez-vous à [Adobe Admin Console](https://adminconsole.adobe.com) en tant qu’administrateur système.

1. Accédez à **Settings** (Paramètres) dans la barre de navigation supérieure, puis choisissez **Identity** (Identité). La liste des répertoires s’ouvre. Sélectionnez le répertoire fédéré de votre choix.

1. Dans la page **Directory Details** (Détails du répertoire), sélectionnez **Configure** (Configurer).

1. Copiez l’ID d’entité et l’URL ACS (URL Assertion Consumer Service ou URL de réponse). Entrez les URL dans les champs appropriés du portail Azure.

    ![Configurer l’authentification unique côté application](./media/adobe-creative-cloud-tutorial/tutorial_adobe-creative-cloud_003.png)

    a. Utilisez la valeur de l’ID d’entité que vous a fournie Adobe comme **Identificateur** dans la boîte de dialogue **Configurer les paramètres d’application**.

    b. Utilisez la valeur de l’URL ACS (URL Assertion Consumer Service) que vous a fournie Adobe comme **URL de réponse** dans la boîte de dialogue **Configurer les paramètres d’application**.

1. Vers le bas de la page, chargez le fichier **XML de métadonnées de fédération** que vous avez téléchargé à partir du portail Azure. 

    ![Fichier XML de métadonnées de fédération](https://helpx.adobe.com/content/dam/help/en/enterprise/kb/configure-microsoft-azure-with-adobe-sso/jcr_content/main-pars/procedure/proc_par/step_228106403/step_par/image_copy/saml_signinig_certificate.png "XML de métadonnées IdP")

1. Sélectionnez **Enregistrer**.


### <a name="create-adobe-creative-cloud-test-user"></a>Créer un utilisateur de test Adobe Creative Cloud

Pour permettre aux utilisateurs Azure AD de se connecter à Adobe Creative Cloud, vous devez les provisionner dans Adobe Creative Cloud. Dans le cas d’Adobe Creative Cloud, cet approvisionnement est une tâche manuelle.

### <a name="to-provision-a-user-accounts-perform-the-following-steps"></a>Pour approvisionner un compte d’utilisateur, procédez comme suit :

1. Connectez-vous au site [Adobe Admin Console](https://adminconsole.adobe.com) en tant qu’administrateur.

2. Ajoutez l’utilisateur au sein de la console Adobe en tant qu’ID fédéré et attribuez-le à un profil de produits. Pour plus d’informations sur l’ajout des utilisateurs, consultez la section [Ajouter des utilisateurs dans Adobe Admin Console](https://helpx.adobe.com/enterprise/using/users.html#Addusers). 

3. À ce stade, tapez votre adresse de messagerie/nom d’utilisateur principal dans le formulaire de connexion Adobe, appuyez sur la touche de tabulation, et vous devez être fédéré à Azure AD :
   * Accès Web : www\.adobe.com > connexion
   * Dans l’utilitaire de l’application de bureau > connexion
   * Dans l’application > aide > connexion

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la vignette Adobe Creative Cloud dans le volet d’accès, vous devez être connecté automatiquement à l’application Adobe Creative Cloud pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Adobe Creative Cloud avec Azure AD](https://aad.portal.azure.com/)

- [Configurer une identité (adobe.com)](https://helpx.adobe.com/enterprise/using/set-up-identity.html)
  
- [Configurer Azure pour une utilisation avec l’authentification unique Adobe (adobe.com)](https://helpx.adobe.com/enterprise/kb/configure-microsoft-azure-with-adobe-sso.html)