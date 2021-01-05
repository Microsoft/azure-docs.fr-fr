---
title: 'Tutoriel : Intégration d’Azure Active Directory à Way We Do | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Way We Do.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/20/2019
ms.author: jeedes
ms.openlocfilehash: 0205617198ec8e57cb891d2415d2b5413a945f1b
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97608933"
---
# <a name="tutorial-integrate-way-we-do-with-azure-active-directory"></a>Tutoriel : Intégrer Way We Do à Azure Active Directory

Dans ce tutoriel, vous allez apprendre à intégrer Way We Do à Azure Active Directory (Azure AD). Quand vous intégrez Way We Do à Azure AD, vous pouvez :

* Dans Azure AD, contrôlez qui a accès à Way We Do.
* Permettre à vos utilisateurs de se connecter automatiquement à Way We Do avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous n’en avez pas, vous pouvez obtenir un essai gratuit d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement Way We Do avec l’authentification unique activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Way We Do prend en charge l’authentification unique (SSO) lancée par le **fournisseur de services**
* Way We Do prend en charge le provisionnement d’utilisateurs **Juste-à-temps**

## <a name="adding-way-we-do-from-the-gallery"></a>Ajout de Way We Do à partir de la galerie

Pour configurer l’intégration de Way We Do à Azure AD, vous devez ajouter Way We Do à partir de la galerie, dans votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Way We Do** dans la zone de recherche.
1. Sélectionnez **Way We Do** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec Way We Do pour un utilisateur de test nommé **B. Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Way We Do associé.

Pour configurer et tester l’authentification unique Azure AD avec Way We Do, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l'authentification unique Way We Do](#configure-way-we-do-sso)** pour configurer les paramètres de l'authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Way We Do](#create-way-we-do-test-user)**  : pour avoir un équivalent de Britta Simon dans Way We Do lié à la représentation Azure AD associée.
6. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Way We Do**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la boîte de dialogue **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion**, saisissez une URL au format suivant : `https://<SUBDOMAIN>.waywedo.com/Authentication/ExternalSignIn`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<SUBDOMAIN>.waywedo.com`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion et l’identificateur réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique de Way We Do](mailto:support@waywedo.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **Certificat (Raw)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

   ![Lien Téléchargement de certificat](common/certificateraw.png)

1. Dans la section **Configurer Way We Do**, copiez la ou les URL appropriées en fonction de vos besoins.

   ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="configure-way-we-do-sso"></a>Configurer l’authentification unique Way We Do

1. Pour automatiser la configuration dans Way We Do, vous devez installer l’**extension de navigateur My Apps Secure Sign-in** en cliquant sur **Install the extension** (Installer l’extension).

    ![Extension My apps](common/install-myappssecure-extension.png)

1. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer Way We Do**, vous êtes alors orienté vers l’application Way We Do. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Way We Do. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 6.

    ![Configuration](common/setup-sso.png)

1. Si vous souhaitez configurer manuellement Way We Do, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Way We Do en tant qu’administrateur et effectuez les étapes suivantes :

1. Cliquez sur l’**icône représentant une personne** dans le coin supérieur droit d’une page de Way We Do, puis cliquez sur **Account** (Compte) dans le menu déroulant.

    ![Compte Way We Do](./media/waywedo-tutorial/tutorial_waywedo_account.png)

1. Cliquez sur l’**icône de menu** pour ouvrir le menu de navigation de commande, puis cliquez sur **Single Sign On** (Authentification unique).

    ![Authentification unique Way We Do](./media/waywedo-tutorial/tutorial_waywedo_single.png)

1. Dans la page **Single sign-on setup** (Configuration de l’authentification unique), procédez comme suit :

    ![Enregistrer Way We Do](./media/waywedo-tutorial/tutorial_waywedo_save.png)

    1. Cliquez sur le bouton bascule de **Turn on single sign-on** (Activer l’authentification unique) pour choisir **Yes** (Oui) et ainsi activer l’authentification unique.

    1. Dans la zone de texte **Single sign-on name** (Nom de l’authentification unique), entrez votre nom.

    1. Dans la zone de texte **Entity ID**, collez l’**Identificateur Azure AD** que vous avez copié dans le portail Azure.

    1. Dans la zone de texte **SAML SSO URL** (URL d’authentification unique SAML), collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.

    1. Chargez le certificat en cliquant sur le bouton **Select** (sélectionner) situé en regard de **Certificate** (Certificat).

    1. **Paramètres facultatifs** -

        * Enable Passwords (Activer les mots de passe) - Lorsque cette option est désactivée, le mot de passe habituel fonctionne pour Way We Do de telle sorte que les utilisateurs n’ont pas d’autre choix que celui d’utiliser l’authentification unique.

        * Enable Auto-provisioning (Activer le provisionnement automatique) - Lorsque cette option est activée, l’adresse e-mail utilisée pour l’authentification est automatiquement comparée à la liste des utilisateurs dans Way We Do. Si l’adresse e-mail ne correspond pas à un utilisateur actif dans Way We Do, un nouveau compte d’utilisateur pour la personne se connectant est ajouté, dans lequel il lui est demandé de renseigner les informations manquantes.

          > [!NOTE]
          > Les utilisateurs ajoutés via l’authentification unique sont ajoutés en tant qu’utilisateurs généraux et aucun rôle ne leur est attribué dans le système. Un administrateur est en mesure d’accéder et de modifier son rôle de sécurité en tant qu’éditeur ou administrateur, il peut également affecter un ou plusieurs rôles d’organigramme.

    1. Cliquez sur **Save** pour enregistrer vos paramètres.

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

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Way We Do.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Way We Do**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-way-we-do-test-user"></a>Créer un utilisateur de test Way We Do

Dans cette section, l’utilisateur Britta Simon est créé dans Way We Do. Way We Do prend en charge le provisionnement d’utilisateurs juste-à-temps, activé par défaut. Vous n’avez aucune opération à effectuer dans cette section. S’il n’existe pas encore d’utilisateur dans Way We Do, un nouveau est créé après l’authentification.

> [!Note]
> Si vous devez créer un utilisateur manuellement, contactez [l’équipe de support technique du client Way We Do](mailto:support@waywedo.com).

### <a name="test-sso"></a>Tester l’authentification unique (SSO)

Quand vous sélectionnez la vignette Way We Do dans le panneau d’accès, vous devez être automatiquement connecté à l’application Way We Do pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)