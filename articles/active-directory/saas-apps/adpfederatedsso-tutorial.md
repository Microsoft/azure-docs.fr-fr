---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à ADP | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et ADP.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/26/2019
ms.author: jeedes
ms.openlocfilehash: 0d78f2c8da338a92ef88734371647a48ddb190c3
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97591206"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-adp"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à ADP

Dans ce tutoriel, vous allez apprendre à intégrer ADP dans Azure Active Directory (Azure AD). Quand vous intégrez ADP dans Azure AD, vous pouvez :

* contrôler dans Azure AD qui a accès à ADP ;
* permettre à vos utilisateurs de se connecter automatiquement à ADP avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement ADP pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* ADP prend en charge l’authentification unique initiée par le **fournisseur d’identité**

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-adp-from-the-gallery"></a>Ajout d’ADP depuis la galerie

Pour configurer l’intégration d’ADP à Azure AD, vous devez ajouter ADP disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **ADP** dans la zone de recherche.
1. Sélectionnez **ADP** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-adp"></a>Configurer et tester l’authentification unique Azure AD pour ADP

Configurez et testez l’authentification unique Azure AD avec ADP à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur ADP associé.

Pour configurer et tester l’authentification unique Azure AD avec ADP, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
2. **[Configurer l’authentification unique ADP](#configure-adp-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test ADP](#create-adp-test-user)** pour avoir un équivalent de B.Simon dans ADP, associé à sa représentation dans Azure AD.
3. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, dans la page d’intégration de l’application **ADP**, cliquez sur **l’onglet Propriétés** et effectuez les étapes suivantes : 

    ![Propriétés de l’authentification unique](./media/adpfederatedsso-tutorial/tutorial_adp_prop.png)

    a. Définissez le champ **Activé pour que les utilisateurs se connectent** sur **Oui**.

    b. Copiez **l’URL d’accès utilisateur** et collez-la dans la **section Configurer l’URL d’authentification**, comme expliqué plus loin dans le didacticiel.

    c. Définissez le champs **Affectation de l’utilisateur obligatoire** sur **Oui**.

    d. Définissez le champ **Visible par les utilisateurs** sur **Non**.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **ADP**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, effectuez les étapes suivantes :

    Dans la zone de texte **Identificateur (ID d’entité)** , tapez une URL : `https://fed.adp.com`

4. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

6. Dans la section **Configurer ADP**, copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à ADP.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **ADP**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-adp-sso"></a>Configurer l’authentification unique ADP

Pour configurer l’authentification unique côté **ADP**, vous devez charger le **XML de métadonnées** téléchargé sur le [site web ADP](https://adpfedsso.adp.com/public/login/index.fcc).

> [!NOTE]  
> Ce processus peut prendre quelques jours.

### <a name="configure-your-adp-services-for-federated-access"></a>Configurer vos services ADP pour un accès fédéré

>[!Important]
> Vous devez assigner à l’application de service ADP les employés qui ont besoin d’un accès fédéré à vos services ADP, puis réassigner les utilisateurs à un service ADP spécifique.
Une fois la confirmation reçue de votre représentant ADP, configurez vos services ADP et assignez/gérez les utilisateurs pour contrôler l’accès utilisateur à un service ADP spécifique.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **ADP** dans la zone de recherche.
1. Sélectionnez **ADP** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.
1. Dans le portail Azure, dans la page d’intégration de votre application **ADP**, cliquez sur **l’onglet Propriétés** et effectuez les étapes suivantes :  

    ![Propriétés de l’authentification unique liée](./media/adpfederatedsso-tutorial/tutorial_adp_linkedproperties.png)

    1. Définissez le champ **Activé pour que les utilisateurs se connectent** sur **Oui**.

    1. Définissez le champs **Affectation de l’utilisateur obligatoire** sur **Oui**.

    1. Définissez le champ **Visible par les utilisateurs** sur **Oui**.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **ADP**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.

1. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le **Mode****Lié**. pour lier votre application à **ADP**.

    ![Authentification unique liée](./media/adpfederatedsso-tutorial/tutorial_adp_linked.png)

1. Accédez à la section **Configurer l’URL d’authentification**, puis effectuez les étapes suivantes :

    ![Propriétés de l’authentification unique](./media/adpfederatedsso-tutorial/tutorial_adp_linkedsignon.png)

    1. Collez l’**URL d’accès utilisateur**, que vous avez copiée à partir de l’**onglet Propriétés** plus haut (à partir de l’application ADP principale).

    1. Voici les 5 applications qui prennent en charge différentes **URL d’état de relais**. Vous devez ajouter la valeur **d’URL d’état de relais** appropriée pour une application particulière manuellement à **l’URL d’accès utilisateur**.

        * **ADP Workforce Now**

            `<User access URL>&relaystate=https://fed.adp.com/saml/fedlanding.html?WFN`

        * **ADP Workforce Now Enhanced Time**

            `<User access URL>&relaystate=https://fed.adp.com/saml/fedlanding.html?EETDC2`

        * **ADP Vantage HCM**

            `<User access URL>&relaystate=https://fed.adp.com/saml/fedlanding.html?ADPVANTAGE`

        * **ADP Enterprise HR**

            `<User access URL>&relaystate=https://fed.adp.com/saml/fedlanding.html?PORTAL`

        * **MyADP**

            `<User access URL>&relaystate=https://fed.adp.com/saml/fedlanding.html?REDBOX`

1. **Enregistrez** les changements apportés.

1. Une fois la confirmation reçue de votre représentant ADP, commencez à effectuer un test avec un ou deux utilisateurs.

    1. Assignez quelques utilisateurs à l’application de service ADP pour tester l’accès fédéré.

    1. Le test est réussi quand les utilisateurs accèdent à l’application de service ADP dans la galerie et qu’ils peuvent accéder à leur service ADP.

1. Une fois confirmée la réussite d’un test, assignez le service ADP fédéré à des utilisateurs ou à des groupes d’utilisateurs, comme expliqué plus loin dans le didacticiel, et déployez-le pour vos employés.

### <a name="create-adp-test-user"></a>Créer un utilisateur de test ADP

L’objectif de cette section est de créer un utilisateur appelé B. Simon dans ADP. Pour ajouter les utilisateurs au compte ADP, contactez l’[équipe du support ADP](https://www.adp.com/contact-us/overview.aspx). 

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette ADP dans le panneau d’accès doit vous connecter automatiquement à l’application ADP pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)