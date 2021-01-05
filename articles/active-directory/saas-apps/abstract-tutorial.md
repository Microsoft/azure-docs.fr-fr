---
title: 'Tutoriel : Intégration d’Azure Active Directory à Abstract | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Abstract.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/16/2019
ms.author: jeedes
ms.openlocfilehash: e1c3236c4c1957b4d0daee8d30c71f03fb8674dd
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97587806"
---
# <a name="tutorial-integrate-abstract-with-azure-active-directory"></a>Tutoriel : Intégrer Abstract à Azure Active Directory

Dans ce tutoriel, vous allez apprendre à intégrer Abstract à Azure Active Directory (Azure AD). Quand vous intégrez Abstract à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Abstract.
* Autoriser les utilisateurs à se connecter automatiquement à Abstract avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Abstract pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Abstract prend en charge l’authentification unique lancée par le **fournisseur de services et le fournisseur d’identité**

## <a name="adding-abstract-from-the-gallery"></a>Ajout d’Abstract à partir de la galerie

Pour configurer l’intégration d’Abstract avec Azure AD, vous devez ajouter Abstract à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Abstract** dans la zone de recherche.
1. Sélectionnez **Abstract** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on"></a>Configurer et tester l’authentification unique Azure AD

Configurez et testez l’authentification unique Azure AD avec Abstract pour un utilisateur de test nommé **B. Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Abstract associé.

Pour configurer et tester l’authentification unique Azure AD avec Abstract, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Configurer l’authentification unique Abstract](#configure-abstract-sso)** pour configurer les paramètres de l’authentification unique côté application.
3. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
4. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Créer un utilisateur de test Abstract](#create-abstract-test-user)** pour avoir un équivalent de Britta Simon dans Abstract lié à la représentation Azure AD de l’utilisateur.
6. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Abstract**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, l’application est préconfigurée en mode Lancement par le **fournisseur d’identité** et les URL nécessaires sont déjà préremplies avec Azure. L’utilisateur doit enregistrer la configuration en cliquant sur le bouton **Enregistrer**.

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services**, cliquez sur **Définir des URL supplémentaires**, puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion**, tapez une URL : `https://app.abstract.com/signin`

4. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, cliquez sur le bouton Copier pour copier l’**URL des métadonnées de fédération d’application**, puis enregistrez-la sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/copy-metadataurl.png)

### <a name="configure-abstract-sso"></a>Configurer l’authentification unique Abstract

Veillez à récupérer votre `App Federation Metadata Url` et l’`Azure AD Identifier` à partir du portail Azure, car vous en aurez besoin pour configurer l’authentification unique sur Abstract.

Vous trouverez ces informations dans la page **Configurer l’authentification unique avec SAML** :

* L’`App Federation Metadata Url` se trouve dans la section **Certificat de signature SAML**.
* L’`Azure AD Identifier` se trouve dans la section **Configurer Abstract**.


Vous êtes maintenant prêt à configurer l’authentification unique sur Abstract :

>[!Note]
>Vous devrez vous authentifier auprès d’un compte d’administrateur de l’organisation pour accéder aux paramètres d’authentification unique sur Abstract.

1. Ouvrez l’[application web Abstract](https://app.abstract.com/).
2. Accédez à la page **Permissions** (Autorisations) dans la barre latérale gauche.
3. Dans la section **Configure SSO** (Configurer l’authentification unique), entrez votre URL de métadonnées (**Metadata URL**) et votre ID d’entité (**Entity ID**).
4. Entrez les éventuelles exceptions manuelles. Les e-mails figurant dans la section d’exceptions manuelles ignoreront l’authentification unique et pourront se connecter avec un e-mail et un mot de passe. 
5. Cliquez sur **Enregistrer les modifications**.

>[!Note] 
>Vous devez utiliser les adresses e-mail principales dans la liste des exceptions manuelles. L’activation de l’authentification unique échoue si l’e-mail que vous spécifiez est l’e-mail secondaire d’un utilisateur. Si cela se produit, un message d’erreur s’affiche avec l’adresse e-mail principale du compte défaillant. Ajoutez cette adresse e-mail principale à la liste des exceptions manuelles une fois que vous avez vérifié que vous connaissez l’utilisateur.

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

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Abstract.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Abstract**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

### <a name="create-abstract-test-user"></a>Créer un utilisateur de test Abstract

Pour tester l’authentification unique sur Abstract

1. Ouvrez l’[application web Abstract](https://app.abstract.com/).
2. Accédez à la page **Permissions** (Autorisations) dans la barre latérale gauche.
3. Cliquez sur **Test with my Account** (Tester avec mon compte). Si le test échoue, [contactez notre équipe de support](https://help.abstract.com/hc/).

>[!Note]
>Vous devrez vous authentifier auprès d’un compte d’administrateur de l’organisation pour accéder aux paramètres d’authentification unique sur Abstract.
Ce compte d’administrateur de l’organisation doit être affecté à Abstract sur le portail Azure.

### <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Abstract dans le volet d’accès, vous devez être connecté automatiquement à l’application Abstract pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)