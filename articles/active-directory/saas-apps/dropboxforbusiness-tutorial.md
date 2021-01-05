---
title: 'Tutoriel : Intégration d’Azure Active Directory à Dropbox Business | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Dropbox Business.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/23/2020
ms.author: jeedes
ms.openlocfilehash: bf86656ff670df19162867d597a869d762a012e8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019342"
---
# <a name="tutorial-integrate-dropbox-business-with-azure-active-directory"></a>Tutoriel : Intégrer Dropbox Business à Azure Active Directory

Dans ce tutoriel, vous allez apprendre à intégrer Dropbox Business à Azure Active Directory (Azure AD). Quand vous intégrez Dropbox Business à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Dropbox Business.
* Permettre à vos utilisateurs de se connecter automatiquement à Dropbox Business avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’en avez pas, vous pouvez obtenir un essai gratuit d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Dropbox Business pour lequel l’authentification unique est activée.

> [!NOTE]
> Cette intégration peut également être utilisée à partir de l’environnement cloud US Government Azure AD. Cette application est disponible dans la Galerie d’applications cloud US Government Azure AD et peut être configurée de la même façon que dans le cloud public.

## <a name="scenario-description"></a>Description du scénario

* Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. Dropbox Business prend en charge l’authentification unique lancée par le **fournisseur de services**

* Dropbox Business prend en charge l’[attribution et l’annulation d’attribution d’utilisateurs](dropboxforbusiness-tutorial.md)
* Après avoir configuré Dropbox, vous pouvez appliquer le contrôle de session, qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)

## <a name="adding-dropbox-business-from-the-gallery"></a>Ajout de Dropbox Business à partir de la galerie

Pour configurer l’intégration de Dropbox Business à Azure AD, vous devez ajouter Dropbox Business, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Dropbox Business** dans la zone de recherche.
1. Sélectionnez **Dropbox Business** dans le volet des résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec Dropbox Business en utilisant un utilisateur de test appelé **Britta Simon**. Pour que l’authentification unique fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur Dropbox Business associé.

Pour configurer et tester l’authentification unique Azure AD avec Dropbox Business, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.    
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Dropbox Business](#configure-dropbox-business-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Dropbox Business](#create-dropbox-business-test-user)** pour disposer, dans Dropbox Business, d’un équivalent de Britta Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Dropbox Business**, recherchez la section **Gérer**, puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la boîte de dialogue **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion**, saisissez une URL au format suivant : `https://www.dropbox.com/sso/<id>`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , tapez une valeur : `Dropbox`

    > [!NOTE]
    > La valeur URL de connexion ci-dessus n’est pas une valeur réelle. Vous mettrez à jour la valeur avec l’URL de connexion réelle. La procédure est expliquée plus loin dans le didacticiel.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Dropbox Business**, copiez la ou les URL appropriées, en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion


### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé Britta Simon dans le Portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory**, **Utilisateurs**, puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans le champ **Nom**, entrez `Britta Simon`.  
   1. Dans le champ **Nom de l’utilisateur**, entrez username@companydomain.extension. Par exemple : `BrittaSimon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Dropbox Business.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Dropbox Business**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-dropbox-business-sso"></a>Configurer l’authentification unique Dropbox Business

1. Pour automatiser la configuration dans Dropbox Business, vous devez installer l’**extension de navigateur de connexion sécurisée à Mes applications** en cliquant sur **Installer l’extension**.

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après avoir ajouté l’extension au navigateur, cliquez sur **Configurer Dropbox Business** pour être dirigé vers l’application Dropbox Business. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Dropbox Business. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 8.

    ![Configuration](common/setup-sso.png)

3. Si vous voulez configurer Dropbox Business manuellement, ouvrez une nouvelle fenêtre de navigateur web, puis accédez et connectez-vous à votre locataire Dropbox Business. Effectuez les étapes suivantes :

    ![Capture d’écran montrant la page « Dropbox Business Sign in ».](./media/dropboxforbusiness-tutorial/ic769509.png "Configurer l’authentification unique")

4. Cliquez sur l’**Icône Utilisateur** et sélectionnez l’onglet **Paramètres**.

    ![Capture d’écran montrant l’action « USER ICON » et « Settings » sélectionnés.](./media/dropboxforbusiness-tutorial/configure1.png "Configurer l’authentification unique")

5. Dans le volet de navigation gauche, cliquez sur **Console d’administration**.

    ![Capture d’écran montrant « Admin console » sélectionné.](./media/dropboxforbusiness-tutorial/configure2.png "Configurer l’authentification unique")

6. Dans la **Console d’administration**, cliquez sur **Paramètres** dans le volet de navigation gauche.

    ![Capture d’écran montrant « Paramètres » sélectionné.](./media/dropboxforbusiness-tutorial/configure3.png "Configurer l’authentification unique")

7. Sélectionnez **Authentification unique** sous la section **Authentification**.

    ![Capture d’écran montrant la section « Authentication » avec l’option « Single sign-on » sélectionnée.](./media/dropboxforbusiness-tutorial/configure4.png "Configurer l’authentification unique")

8. Dans la section **Authentification unique**, procédez comme suit :  

    ![Capture d’écran montrant les paramètres de configuration de « Single sign-on »](./media/dropboxforbusiness-tutorial/configure5.png "Configurer l’authentification unique").

    a. Sélectionnez **Requis** en tant qu’option dans la liste déroulante de l’**Authentification unique**.

    b. Cliquez sur **Ajouter une URL de connexion**, puis dans la zone de texte **URL de connexion du fournisseur d’identité**, collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure, et sélectionnez **Terminé**.

    ![Configurer l’authentification unique](./media/dropboxforbusiness-tutorial/configure6.png "Configurer l’authentification unique")

    c. Cliquez sur **Charger un certificat**, puis cherchez votre **Fichier de certificat codé en Base64** que vous avez téléchargé à partir du portail Azure.

    d. Cliquez sur **Copier le lien** et collez la valeur copiée dans la zone de texte **URL de connexion** de la section **Domaine et URL Dropbox Business** du portail Azure.

    e. Cliquez sur **Enregistrer**.

### <a name="create-dropbox-business-test-user"></a>Créer un utilisateur de test Dropbox Business

Dans cette section, un utilisateur appelé B.Simon est créé dans Dropbox Business. Dropbox Business prend en charge l’attribution d’utilisateurs juste-à-temps, qui est activée par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Dropbox Business, il en est créé un après l’authentification.

>[!Note]
>Si vous devez créer un utilisateur manuellement, contactez l’[équipe du support technique Dropbox Business](https://www.dropbox.com/business/contact)

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Quand vous sélectionnez la vignette Dropbox Business dans le volet d’accès, vous devez être connecté automatiquement à l’application Dropbox Business pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](/cloud-app-security/proxy-intro-aad)

- [Essayer Dropbox Business avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)