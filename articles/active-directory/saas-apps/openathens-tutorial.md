---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à OpenAthens | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et OpenAthens.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/24/2019
ms.author: jeedes
ms.openlocfilehash: fc2d1c5dca4265bc0f0c26dd0a6c62a7e1698a38
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621465"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-openathens"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à OpenAthens

Dans ce tutoriel, vous allez apprendre à intégrer OpenAthens à Azure Active Directory (Azure AD). Quand vous intégrez OpenAthens à Azure AD, vous pouvez :

* Dans Azure AD, contrôler qui a accès à OpenAthens.
* Permettre à vos utilisateurs de se connecter automatiquement à OpenAthens avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement OpenAthens pour lequel l’authentification unique (SSO) est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* OpenAthens prend en charge l’authentification unique initiée par le **fournisseur d’identité**
* OpenAthens prend en charge l’attribution d’utilisateurs **Juste-à-temps**

## <a name="adding-openathens-from-the-gallery"></a>Ajout d’OpenAthens à partir de la galerie

Pour configurer l’intégration d’OpenAthens à Azure AD, vous devez ajouter OpenAthens à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **OpenAthens** dans la zone de recherche.
1. Sélectionnez **OpenAthens** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-openathens"></a>Configurer et tester l’authentification unique Azure AD pour OpenAthens

Configurez et testez l’authentification unique (SSO) Azure AD avec OpenAthens à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur OpenAthens associé.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec OpenAthens, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique OpenAthens](#configure-openathens-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test OpenAthens](#create-openathens-test-user)** pour avoir un équivalent de B.Simon dans OpenAthens lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **OpenAthens**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, chargez le **fichier de métadonnées du fournisseur de services** ; les étapes pour effectuer cette opération sont indiquées plus loin dans ce tutoriel.

    a. Cliquez sur **Charger un fichier de métadonnées**.

    ![charger les métadonnées pour openathens](common/upload-metadata.png)

    b. Cliquez sur le **logo du dossier** pour sélectionner le fichier de métadonnées, puis cliquez sur **Charger**.

    ![rechercher et charger les métadonnées pour openathens](common/browse-upload-metadata.png)

    c. Une fois le fichier de métadonnées chargé, la valeur **Identificateur** est automatiquement renseignée dans la zone de texte de la section **Configuration SAML de base** :

    ![Informations d’authentification unique dans Domaine et URL OpenAthens](common/idp-identifier.png)

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer OpenAthens**, copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à OpenAthens.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **OpenAthens**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-openathens-sso"></a>Configurer l’authentification unique OpenAthens

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise OpenAthens en tant qu’administrateur.

1. Sélectionnez **Connexions** dans la liste sous l’onglet **Gestion**.

    ![Capture d’écran montrant la page de site d’entreprise « OpenAthens » avec l’élément « Connections » sélectionné dans l’onglet « Management ».](./media/openathens-tutorial/tutorial_openathens_application1.png)

1. Sélectionnez **SAML 1.1/2.0**, puis le bouton **Configurer**.

    ![Capture d’écran montrant la boîte de dialogue « Select local authentication system type. » avec l’option « SAML 1.1/2.0 » et le bouton « Configure » sélectionnés.](./media/openathens-tutorial/tutorial_openathens_application2.png)

1. Pour ajouter la configuration, sélectionnez le bouton **Parcourir** pour charger le fichier .xml de métadonnées que vous avez téléchargé à partir du portail Azure, puis sélectionnez **Ajouter**.

    ![Capture d’écran montrant la boîte de dialogue « Add SAML authentication system. » avec l’action « Browse » et le bouton « Add » sélectionnés.](./media/openathens-tutorial/tutorial_openathens_application3.png)

1. Effectuez les étapes suivantes sous l’onglet **Détails**.

    ![Configurer l’authentification unique](./media/openathens-tutorial/tutorial_openathens_application4.png)

    a. Dans **Display name mapping** (Mappage de nom d’affichage), sélectionnez **Use Attribute** (Utiliser l’attribut).

    b. Dans la zone de texte **Display name attribute** (Attribut de nom d’affichage), entrez la valeur `http://schemas.microsoft.com/identity/claims/displayname`.

    c. Dans **Unique user mapping** (Mappage d’utilisateur unique), sélectionnez **Use Attribute** (Utiliser l’attribut).

    d. Dans la zone de texte **Unique user attribute** (Attribut d’utilisateur unique), entrez la valeur `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`.

    e. Dans **État**, cochez les trois cases.

    f. Dans **Create local accounts** (Créer des comptes locaux), sélectionnez **automatically** (Automatiquement).

    g. Sélectionnez **Enregistrer les modifications**.

    h. Sous l’onglet **</> Relying Party** (Partie de confiance), copiez l’**URL des métadonnées** et ouvrez-la dans le navigateur pour télécharger le **fichier XML de métadonnées du fournisseur de services**. Chargez ce fichier de métadonnées du fournisseur de services dans la section **Configuration SAML de base** dans Azure AD.

    ![Capture d’écran montrant l’onglet « Relying party » sélectionné et l’option « Metadata URL » mise en évidence.](./media/openathens-tutorial/tutorial_openathens_application5.png)

### <a name="create-openathens-test-user"></a>Créer un utilisateur de test OpenAthens

Dans cette section, un utilisateur appelé Britta Simon est créé dans OpenAthens. OpenAthens prend en charge l’**attribution d’utilisateurs juste-à-temps**, option activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans OpenAthens, il en est créé un après l’authentification.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette OpenAthens dans le panneau d’accès doit vous connecter automatiquement à l’application OpenAthens pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer OpenAthens avec Azure AD](https://aad.portal.azure.com/)
