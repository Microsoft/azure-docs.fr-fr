---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Concur Travel and Expense | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Concur Travel and Expense.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/10/2019
ms.author: jeedes
ms.openlocfilehash: 525c70c18354e35998e564680c68a975bdb3ec54
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92455440"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-concur-travel-and-expense"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Concur Travel and Expense

Dans ce tutoriel, vous allez apprendre à intégrer Concur Travel and Expense à Azure Active Directory (Azure AD). Quand vous intégrez Concur Travel and Expense à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Concur Travel and Expense.
* Permettre à vos utilisateurs de se connecter automatiquement à Concur Travel and Expense avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Abonnement Concur Travel and Expense.
* Un rôle « Company Administrator » (Administrateur d’entreprise) sous votre compte d’utilisateur Concur. Vous pouvez vérifier si vous disposez de l’accès adéquat en accédant à [Concur SSO Self-Service Tool](https://www.concursolutions.com/nui/authadmin/ssoadmin) (Outil libre-service d’authentification unique Concur). Si vous ne disposez pas de l’accès, contactez le chef de projet d’implémentation ou de support Concur. 

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD.

* Concur Travel and Expense prend en charge l’authentification unique lancée par le **fournisseur d’identité** et le **fournisseur de services**
* Concur Travel and Expense prend en charge le test de l’authentification unique dans l’environnement de production et dans l’environnement d’implémentation 

> [!NOTE]
> L’identificateur de cette application est une valeur de chaîne fixe pour chacune des trois régions : États-Unis, EMEA et Chine. Ainsi, une seule instance peut être configurée pour chaque région dans un locataire. 

## <a name="adding-concur-travel-and-expense-from-the-gallery"></a>Ajout de Concur Travel and Expense à partir de la galerie

Pour configurer l’intégration de Concur Travel and Expense à Azure AD, vous devez ajouter Concur Travel and Expense, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Concur Travel and Expense** dans la zone de recherche.
1. Sélectionnez **Concur Travel and Expense** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-concur-travel-and-expense"></a>Configurer et tester l’authentification unique Azure AD pour Concur Travel and Expense

Configurez et testez l’authentification unique Azure AD avec Concur Travel and Expense à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Concur Travel and Expense associé.

Pour configurer et tester l’authentification unique Azure AD avec Concur Travel and Expense, effectuez les modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Concur Travel and Expense](#configure-concur-travel-and-expense-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Concur Travel and Expense](#create-concur-travel-and-expense-test-user)** pour avoir un équivalent de B.Simon dans Concur Travel and Expense lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Concur Travel and Expense** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , l’application est préconfigurée en mode Lancement par le **fournisseur d’identité** et les URL nécessaires sont déjà préremplies avec Azure. L’utilisateur doit enregistrer la configuration en cliquant sur le bouton **Enregistrer** .

    > [!NOTE]
    > L’identificateur (ID d’entité) et l’URL de réponse (URL Assertion Consumer Service) sont spécifiques à la région. Veuillez les sélectionner en fonction du centre de données de votre entité Concur. Si vous ne connaissez pas le centre de données de votre entité Concur, contactez le support technique Concur. 

5. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Attribut utilisateur** afin de modifier les paramètres. L’identifiant utilisateur unique doit correspondre à l’identifiant de connexion de l’utilisateur Concur. En règle générale, vous devez remplacer **user.userprincipalname** par **user.mail** .

    ![Modifier un attribut utilisateur](common/edit-attribute.png)

6. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger les métadonnées et les enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en accordant l’accès à Concur Travel and Expense.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Concur Travel and Expense** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-concur-travel-and-expense-sso"></a>Configurer l’authentification unique Concur Travel and Expense

1. Pour configurer l’authentification unique côté **Concur Travel and Expense** , vous devez charger le fichier **XML de métadonnées de fédération** sur [Concur SSO Self-Service Tool](https://www.concursolutions.com/nui/authadmin/ssoadmin) et vous connecter avec un compte ayant le rôle « Company Administrator ». 

1. Cliquez sur **Add** .
1. Entrez un nom personnalisé pour votre fournisseur d’identité, par exemple « Azure AD (US) ». 
1. Cliquez sur **Upload XML File** (Charger le fichier XML) et joignez le fichier **XML de métadonnées de fédération** que vous avez téléchargé.
1. Cliquez sur **Add Metadata** (Ajouter les métadonnées) pour enregistrer la modification.

    ![Capture d’écran de l’outil libre-service d’authentification unique](./media/concur-travel-and-expense-tutorial/add-metadata-concur-self-service-tool.png)

### <a name="create-concur-travel-and-expense-test-user"></a>Créer un utilisateur de test Concur Travel and Expense

Dans cette section, vous créez un utilisateur appelé B.Simon dans Concur Travel and Expense. Collaborez avec l’équipe du support technique Concur pour ajouter les utilisateurs à la plateforme Concur Travel and Expense. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique. 

> [!NOTE]
> L’ID de connexion Concur de B.Simon doit correspondre à l’identificateur unique de B.Simon sur Azure AD. Par exemple, si l’identificateur unique Azure AD de B.Simon est `B.Simon@contoso.com`, l’ID de connexion Concur de B.Simon doit également être `B.Simon@contoso.com`. 

## <a name="configure-concur-mobile-sso"></a>Configurer l’authentification unique mobile Concur
Pour activer l’authentification unique mobile Concur, vous devez fournir à l’équipe de support technique Concur l’ **URL de l’accès utilisateur** . Suivez les étapes ci-dessous pour obtenir l’ **URL de l’accès utilisateur** auprès d’Azure AD :
1. Accédez à **Applications d’entreprise** .
1. Cliquez sur **Concur Travel and Expense** .
1. Cliquez sur **Propriétés** .
1. Copiez l’ **URL de l’accès utilisateur** , puis communiquez-la à l’équipe de support.

> [!NOTE]
> L’option libre-service pour configurer l’authentification unique n’étant pas disponible, collaborez avec l’équipe du support technique Concur pour activer l’authentification unique mobile. 

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Concur Travel and Expense dans le panneau d’accès doit vous connecter automatiquement à l’application Concur Travel and Expense pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Concur Travel and Expense avec Azure AD](https://aad.portal.azure.com/)