---
title: 'Tutoriel : Intégration d’Azure Active Directory à Chargebee | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Chargebee.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/08/2019
ms.author: jeedes
ms.openlocfilehash: 22753b80931956af6ce448cfee974ae746fff6e6
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92456213"
---
# <a name="tutorial-integrate-chargebee-with-azure-active-directory"></a>Tutoriel : Intégrer Chargebee à Azure Active Directory

Dans ce didacticiel, vous allez apprendre à intégrer Chargebee à Azure Active Directory (Azure AD). Quand vous intégrez Chargebee à Azure AD, vous pouvez :

* Contrôler qui a accès à Chargebee dans Azure AD.
* Permettre à vos utilisateurs de se connecter automatiquement à Chargebee avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Chargebee pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Chargebee prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**

## <a name="adding-chargebee-from-the-gallery"></a>Ajout de Chargebee à partir de la Galerie

Pour configurer l'intégration de Chargebee avec Azure AD, vous devez ajouter Chargebee, disponible depuis la galerie, à votre liste d'applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Chargebee** dans la zone de recherche.
1. Sélectionnez **Chargebee** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-chargebee"></a>Configurer et tester l’authentification unique Azure AD pour Chargebee

Configurez et testez l’authentification unique Azure AD avec Chargebee à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans Chargebee.

Pour configurer et tester l’authentification unique Azure AD avec Chargebee, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
2. **[Configurer l’authentification unique Chargebee](#configure-chargebee-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Chargebee](#create-chargebee-test-user)** pour obtenir dans Chargebee un équivalent de B. Simon lié à la représentation Azure AD associée.
3. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Chargebee** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<domainname>.chargebee.com`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://app.chargebee.com/saml/<domainname>/acs`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<domainname>.chargebee.com`

    > [!NOTE]
    > `<domainname>` est le nom du domaine que l’utilisateur crée après avoir revendiqué le compte. En cas d’autres informations, contactez [l’équipede support client Chargebee](mailto:support@chargebee.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

4. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (Base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

6. Dans la section **Configurer Chargebee** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Chargebee.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Chargebee** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-chargebee-sso"></a>Configurer l’authentification unique Chargebee

1. Ouvrez une nouvelle fenêtre de navigateur web et connectez-vous à votre site d’entreprise Chargebee en tant qu’administrateur.

4. Sur le côté gauche du menu, cliquez sur **Paramètres** > **Sécurité** > **Gérer** .

    ![Capture d’écran montrant le site d’entreprise Chargebee avec Settings, Security et Manage sélectionnés.](./media/chargebee-tutorial/config01.png)

5. Sur la fenêtre contextuelle **Authentification unique** , effectuez les étapes suivantes :

    ![Capture d’écran montrant la boîte de dialogue Single Sign-On avec SAML sélectionné et l’option de confirmation.](./media/chargebee-tutorial/config02.png)

    a. Sélectionnez **SAML** .

    b. Dans la zone de texte **URL de connexion** , collez la valeur d’ **URL de connexion** que vous avez copiée sur le Portail Azure.

    c. Ouvrez le certificat encodé en Base64 dans le bloc-notes, copiez son contenu et collez-le dans la zone de texte **Certificat SAML** .

    d. Cliquez sur **Confirmer** .

### <a name="create-chargebee-test-user"></a>Créer un utilisateur de test Chargebee

Pour pouvoir se connecter à Chargebee, les utilisateurs d’Azure AD doivent être provisionnés dans Chargebee. Dans Chargebee, l’approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Ouvrez une autre fenêtre de navigateur web, puis connectez-vous à Chargebee en tant qu’administrateur de la sécurité.

2. Sur le côté gauche du menu, cliquez sur **Clients** , puis accédez à **Créer un client** .

    ![Capture d’écran montrant le site Chargebee avec Customers et Create a New Customer sélectionnés.](./media/chargebee-tutorial/config03.png)

3. Sur la page **Nouveau client** , renseignez les champs respectifs indiqués ci-dessous et cliquez sur **Créer un client** pour créer un utilisateur.

    ![Capture d’écran montrant la page New Customer où vous pouvez entrer des informations sur le client.](./media/chargebee-tutorial/config04.png)

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la mosaïque Chargebee dans le volet d’accès, vous devez être connecté automatiquement à l’application Chargebee pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)