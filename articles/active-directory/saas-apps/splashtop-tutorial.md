---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Splashtop | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Splashtop.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/04/2020
ms.author: jeedes
ms.openlocfilehash: b6dda20487caf6fe3ba49578cfdc0b65434a8dfa
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92520555"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-splashtop"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Splashtop

Dans ce tutoriel, vous allez apprendre à intégrer Splashtop à Azure Active Directory (Azure AD). Quand vous intégrez Splashtop à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Splashtop.
* Permettre à vos utilisateurs de se connecter automatiquement à Splashtop avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Splashtop pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Splashtop prend en charge l’authentification unique lancée par le **fournisseur de services** .

* Après avoir configuré Splashtop, vous pouvez appliquer le contrôle de session qui protège l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Les contrôles de session sont étendus à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-splashtop-from-the-gallery"></a>Ajout de Splashtop depuis la galerie

Pour configurer l’intégration de Splashtop à Azure AD, vous devez ajouter Splashtop à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Splashtop** dans la zone de recherche.
1. Sélectionnez **Splashtop** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-splashtop"></a>Configurer et tester l’authentification unique Azure AD pour Splashtop

Configurez et testez l’authentification unique Azure AD avec Splashtop pour un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans Splashtop.

Pour configurer et tester l’authentification unique Azure AD avec Splashtop, suivez les indications des modules ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Splashtop](#configure-splashtop-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Splashtop](#create-splashtop-test-user)** pour avoir un équivalent de B.Simon dans Splashtop, associé à sa représentation dans Azure AD.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Sur le [Portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Splashtop** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    Dans la zone de texte **URL de connexion** , tapez l’URL : `https://my.splashtop.com/login/sso`

1. Votre application Splashtop s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut, où **nameidentifier** est mappé avec **user.userprincipalname** . L’application TicketManager s’attend à ce que **nameidentifier** soit mappé sur **user.mail** . Vous devez donc modifier le mappage d’attribut en cliquant sur l’icône **Modifier** .

    ![Capture d’écran montrant User Attributes avec l’icône Edit sélectionnée.](common/edit-attribute.png)

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (Base64)** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Splashtop** , copiez la ou les URL appropriées, selon vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Splashtop.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Splashtop** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-splashtop-sso"></a>Configurer l’authentification unique Splashtop

Dans cette section, vous devez demander une nouvelle méthode d’authentification unique à partir du [portail web Splashtop](https://my.splashtop.com/login).
1. Sur le portail web Splashtop, accédez à l’onglet **Account info** / **Team** (Informations sur le compte/Équipe), et faites défiler l’affichage jusqu’à la section **Single Sign On** (Authentification unique). Ensuite, cliquez sur **Apply for new SSO method** (Demander la nouvelle méthode d’authentification unique).

    ![Capture d’écran montrant la page Single Sign-On, où vous pouvez sélectionner Apply for new SSO method.](media/splashtop-tutorial/apply-for-new-SSO-method.png)

1. Dans la fenêtre de demande, attribuez un **nom d’authentification unique** . Par exemple, New Azure, puis sélectionnez **Azure** comme type IDP et insérez l’ **URL de connexion** et l’ **identificateur Azure AD** copiés à partir de l’application Splashtop sur le portail Azure.

    ![Capture d’écran montrant la page Apply for SSO method, dans laquelle vous pouvez entrer un nom et d’autres informations.](media/splashtop-tutorial/azure-sso-1.png)

1. Pour les informations de certificat, cliquez avec le bouton droit sur le fichier de certificat téléchargé à partir de l’application Splashtop sur le portail Azure, modifiez-le avec le Bloc-notes, puis copiez le contenu et collez-le dans le champ **Télécharger le certificat (Base64)** .

    ![Capture d’écran montrant la sélection d’un fichier de certificat et son ouverture avec le Bloc-notes.](media/splashtop-tutorial/cert-1.png)
    ![Capture d’écran affichant le contenu du fichier de certificat.](media/splashtop-tutorial/cert-2.png)
    ![Capture d’écran affichant la zone de texte Download Certificate.](media/splashtop-tutorial/azure-sso-2.png)

1. Et voilà ! Cliquez sur **Enregistrer** et l’équipe de validation de l’authentification unique Splashtop vous contactera pour obtenir les informations de vérification, puis activez la méthode d’authentification unique.

### <a name="create-splashtop-test-user"></a>Créer un utilisateur de test Splashtop

1. Une fois la méthode d’authentification unique activée, vérifiez la méthode d’authentification unique nouvellement créée pour l’activer dans la section **Authentification unique** .

    ![Capture d’écran montrant la page Single Sign on, dans laquelle vous pouvez activer la nouvelle méthode.](media/splashtop-tutorial/enable.png)

1. Invitez l’utilisateur de test, par exemple, `B.Simon@contoso.com` dans votre équipe Splashtop avec la méthode d’authentification unique nouvellement créée.

    ![Capture d’écran montrant la page Invite Users, dans laquelle vous pouvez sélectionner votre nouvelle méthode.](media/splashtop-tutorial/invite.png)

1. Vous pouvez également remplacer un compte Splashtop existant par un compte d’authentification unique. Consultez les [instructions](https://support-splashtopbusiness.splashtop.com/hc/en-us/articles/360038685691-How-to-associate-SSO-method-to-existing-team-admin-member-).

1. Et voilà ! Vous pouvez utiliser le compte d’authentification unique pour vous connecter au portail web Splashtop ou à l’application Splashtop Business.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette Splashtop dans le volet d’accès doit vous connecter automatiquement à l’application Splashtop pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Splashtop avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Splashtop avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)