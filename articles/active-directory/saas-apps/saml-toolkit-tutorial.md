---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Azure AD SAML Toolkit | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Azure AD SAML Toolkit.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/10/2020
ms.author: jeedes
ms.openlocfilehash: 26618382223a87f779f95452000a39126f37efbb
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92675421"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-azure-ad-saml-toolkit"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Azure AD SAML Toolkit

Dans ce didacticiel, vous allez apprendre à intégrer Azure AD SAML Toolkit avec Azure Active Directory (Azure AD). Quand vous intégrez Azure AD SAML Toolkit avec Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Azure AD SAML Toolkit.
* Autoriser les utilisateurs à se connecter automatiquement à Azure AD SAML Toolkit avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Azure AD SAML Toolkit pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Azure AD SAML Toolkit prend en charge l’authentification unique lancée par le **fournisseur de services**

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-azure-ad-saml-toolkit-from-the-gallery"></a>Ajout d’Azure AD SAML Toolkit à partir de la galerie

Pour configurer l’intégration d’Azure AD SAML Toolkit dans Azure AD, vous devez ajouter Azure AD SAML Toolkit à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Azure AD SAML Toolkit** dans la zone de recherche.
1. Sélectionnez **Azure AD SAML Toolkit** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-azure-ad-saml-toolkit"></a>Configurer et tester l’authentification unique Azure AD pour Azure AD SAML Toolkit

Configurez et testez l’authentification unique Azure AD avec Azure AD SAML Toolkit pour un utilisateur de test nommé **B. Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Azure AD SAML Toolkit associé.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec Azure AD SAML Toolkit, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Azure AD SAML Toolkit](#configure-azure-ad-saml-toolkit-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Azure AD SAML Toolkit](#create-azure-ad-saml-toolkit-test-user)** pour avoir un équivalent de B.Simon dans Azure AD SAML Toolkit lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **Azure AD SAML Toolkit** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la boîte de dialogue **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , tapez l’URL : `https://samltoolkit.azurewebsites.net/`

    b. Dans la zone **URL de réponse** , tapez l’URL : `https://samltoolkit.azurewebsites.net/SAML/Consume`

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (brut)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificateraw.png)

1. Dans la section **Configurer Azure AD SAML Toolkit** , copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B. Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Azure AD SAML Toolkit.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Azure AD SAML Toolkit** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .
1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .
1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-azure-ad-saml-toolkit-sso"></a>Configurer l’authentification unique Azure AD SAML Toolkit

1. Ouvrez une nouvelle fenêtre de navigateur web et, si vous n’êtes pas inscrit dans le site web Azure AD SAML Toolkit, commencez par vous inscrire en cliquant sur **Inscrire** . Si vous êtes déjà inscrit, connectez-vous à votre site d’entreprise Azure AD SAML Toolkit à l’aide des informations d’identification de connexion inscrites.

    ![Inscription d’Azure AD SAML Toolkit](./media/saml-toolkit-tutorial/register.png)

1. Cliquez sur **Configuration SAML** .

    ![Configuration SAML d’Azure AD SAML Toolkit](./media/saml-toolkit-tutorial/saml-configure.png)

1. Cliquez sur **Créer** .

    ![Azure AD SAML Toolkit](./media/saml-toolkit-tutorial/createsso.png)

1. Dans la page **Configuration de l’authentification unique SAML** , procédez comme suit :

    ![Azure AD SAML Toolkit - Créer une configuration de l’authentification unique](./media/saml-toolkit-tutorial/fill-details.png)

    1. Dans la zone de texte **URL de connexion** , collez la valeur d’ **URL de connexion** que vous avez copiée sur le Portail Azure.

    1. Dans la zone de texte **Identificateur Azure AD** , collez la valeur **Identificateur Azure AD** que vous avez copiée dans le Portail Azure.

    1. Dans la zone de texte **URL de déconnexion** , collez l’ **URL de déconnexion** que vous avez copiée sur le portail Azure.

    1. Cliquez sur **Sélectionner un fichier** et chargez le fichier **Certificat (brut)** que vous avez téléchargé à partir du Portail Azure.

    1. Cliquez sur **Créer** .

    1. Copiez les valeurs de l’URL de connexion, de l’identificateur et de l’URL ACS figurant dans la page de configuration SSO de SAML Toolkit, puis collez-les dans les zones de texte prévues de la **section Configuration SAML** dans le Portail Azure.

### <a name="create-azure-ad-saml-toolkit-test-user"></a>Créer un utilisateur de test Azure AD SAML Toolkit

Dans cette section, un utilisateur appelé B. Simon est créé dans Azure AD SAML Toolkit. Créez un utilisateur de test dans l’outil en inscrivant un nouvel utilisateur, puis fournissez tous les détails de l’utilisateur. 

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

1. Cliquez sur **Tester cette application** dans le portail Azure. Cette opération redirige vers l’URL de connexion SAML Toolkit où vous pouvez lancer le flux de connexion. 

2. Accédez directement à l’URL de connexion SAML Toolkit pour lancer le flux de connexion à partir de là.

3. Vous pouvez utiliser le volet d’accès Microsoft. Le fait de cliquer sur la vignette SAML Toolkit dans le volet d’accès doit vous connecter automatiquement à l’application SAML Toolkit pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Azure AD SAML Toolkit, vous pouvez appliquer le contrôle de session qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)