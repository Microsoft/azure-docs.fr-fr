---
title: 'Tutoriel : Intégration d’Azure Active Directory à Evidence.com | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Evidence.com.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/24/2020
ms.author: jeedes
ms.openlocfilehash: ff77ace78dd3cdd4ee942308432795421f6da997
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92453790"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-evidencecom"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Evidence.com

Dans ce tutoriel, vous allez découvrir comment intégrer Evidence.com à Azure Active Directory (Azure AD). Quand vous intégrez Evidence.com à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Evidence.com.
* Permettre à vos utilisateurs de se connecter automatiquement à Evidence.com avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Evidence.com pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Evidence.com prend en charge l’authentification unique lancée par le **fournisseur de services**
* Après avoir configuré Evidence.com, vous pouvez appliquer le contrôle de session qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-evidencecom-from-the-gallery"></a>Ajout d’Evidence.com à partir de la galerie

Pour configurer l’intégration d’Evidence.com à Azure AD, vous devez ajouter Evidence.com à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Evidence.com** dans la zone de recherche.
1. Sélectionnez **Evidence.com** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-evidencecom"></a>Configurer et tester l’authentification unique Azure AD pour Evidence.com

Configurez et testez l’authentification unique Azure AD avec Evidence.com pour un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Evidence.com associé.

Pour configurer et tester l’authentification unique Azure AD avec Evidence.com, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Evidence.com](#configure-evidencecom-sso)** – pour configurer les paramètres d’authentification unique côté application.
    1. **[Créer un utilisateur de test Evidence.com](#create-evidencecom-test-user)** pour avoir un équivalent de B.Simon dans Evidence.com lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Evidence.com** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la page **Configurer l’authentification unique avec SAML** , effectuez les étapes suivantes :

    a. Dans la zone de texte **URL de connexion** , saisissez une URL au format suivant : `https://<yourtenant>.evidence.com`

    b. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://<yourtenant>.evidence.com`

    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<your tenant>.evidence.com/?class=UIX&proc=Login`
    
    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Pour obtenir ces valeurs, contactez l’[équipe de support technique Evidence.com](https://communities.taser.com/support/SupportContactUs?typ=LE). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

5. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Evidence.com** , copiez la ou les URL appropriées, en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Evidence.com.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Evidence.com** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-evidencecom-sso"></a>Configurer l’authentification unique Evidence.com

1. Dans une fenêtre de navigateur web distincte, connectez-vous à votre client Evidence.com en tant qu’administrateur et accédez à l’onglet **Admin** .

2. Cliquez sur **Agency Single Sign On**

3. Sélectionnez **SAML Based Single Sign On**

4. Copiez les valeurs **Identificateur Azure AD** , **URL de connexion** et **URL de déconnexion** dans le portail Azure et collez-les dans les champs correspondants dans Evidence.com.

5. Ouvrez dans le Bloc-notes votre fichier Certificate(Base64) téléchargé, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **Certificat de sécurité** . 

6. Enregistrez la configuration dans Evidence.com.

### <a name="create-evidencecom-test-user"></a>Créer un utilisateur de test Evidence.com

Pour que les utilisateurs Azure AD puissent se connecter, leur accès doit être approvisionné dans l’application Evidence.com. Cette section décrit comment créer des comptes d’utilisateur Azure AD dans Evidence.com.

**Pour approvisionner un compte d’utilisateur dans Evidence.com :**

1. Dans une fenêtre de navigateur web, connectez-vous à votre site d’entreprise Evidence.com en tant qu’administrateur.

2. Accédez à l’onglet **Admin** .

3. Cliquez sur **Add User** .

4. Cliquez sur le bouton **Add** .

5. La valeur **Email Address** (Adresse de messagerie) de l’utilisateur ajouté doit correspondre au nom d’utilisateur de l’utilisateur auquel vous souhaitez accorder l’accès dans Azure AD. Si le nom d’utilisateur et l’adresse de messagerie n’ont pas la même valeur dans votre organisation, vous pouvez modifier la section **Evidence.com > Attributs > Authentification unique** du portail Azure de sorte que l’identificateur du nom envoyé à Evidence.com soit l’adresse de messagerie.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Evidence.com dans le volet d’accès, vous devez être connecté automatiquement à l’application Evidence.com pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Evidence.com avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Evidence.com avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)