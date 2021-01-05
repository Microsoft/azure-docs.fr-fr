---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Firmex VDR | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Firmex VDR.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/21/2020
ms.author: jeedes
ms.openlocfilehash: 6dbd39b5c56192ad2ca957c5500338b50e8c8963
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92453380"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-firmex-vdr"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Firmex VDR

Dans ce tutoriel, vous allez apprendre à intégrer Firmex VDR à Azure Active Directory (Azure AD). Quand vous intégrez Firmex VDR à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Firmex VDR.
* Permettre à vos utilisateurs de se connecter automatiquement à Firmex VDR avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Firmex VDR pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Firmex VDR prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**

* Après avoir configuré Firmex, vous pouvez appliquer des contrôles de session qui protègent l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Les contrôles de session sont étendus à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-firmex-vdr-from-the-gallery"></a>Ajout de Firmex VDR depuis la galerie

Pour configurer l’intégration de Firmex VDR à Azure AD, vous devez ajouter Firmex VDR depuis la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Firmex VDR** dans la zone de recherche.
1. Sélectionnez **Firmex VDR** dans le panneau de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-firmex-vdr"></a>Configurer et tester l’authentification unique Azure AD pour Firmex VDR

Configurez et testez l’authentification unique Azure AD avec Firmex VDR à l’aide d’un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Firmex VDR associé.

Pour configurer et tester l’authentification unique Azure AD avec Firmex VDR, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Firmex VDR](#configure-firmex-vdr-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test Firmex VDR](#create-firmex-vdr-test-user)** pour avoir un équivalent de B.Simon dans Firmex VDR lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans la page d’intégration de l’application **Firmex VDR** du [portail Azure](https://portal.azure.com/), recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , l’utilisateur n’a rien à faire, car l’application est déjà intégrée à Azure.

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez l’URL : `https://login.firmex.com`

1. Cliquez sur **Enregistrer** .

1. L’application Firmex VDR s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application Firmex VDR s’attend à ce que quelques attributs supplémentaires (présentés ci-dessous) soient retransmis dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.

    | Nom | Attribut source|
    | ------------ | --------- |
    | email | user.mail |

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)

1. Dans la section **Configurer Firmex VDR** , copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Firmex VDR.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Firmex VDR** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-firmex-vdr-sso"></a>Configurer l’authentification unique Firmex VDR

### <a name="before-you-get-started"></a>Avant de commencer

#### <a name="what-youll-need"></a>Ce dont vous avez besoin

-   Un abonnement Firmex actif
-   Azure AD en tant que service d’authentification unique
-   Votre administrateur informatique pour configurer l’authentification unique
-   Une fois l’authentification unique activée, tous les utilisateurs de votre entreprise doivent se connecter à Firmex à l’aide de celle-ci et non avec une connexion/un mot de passe.

#### <a name="how-long-will-this-take"></a>Combien de temps cela prend-il ?

L’implémentation de l’authentification unique prend quelques minutes. Il n’y a quasiment aucun temps d’arrêt entre l’activation de l’authentification unique pour votre site par le support technique Firmex et l’authentification des utilisateurs de votre entreprise à l’aide de celle-ci. Suivez simplement les étapes ci-dessous.

### <a name="step-1---identify-your-companys-domains"></a>Étape 1 : Identifier les domaines de votre entreprise

Identifiez les domaines auxquels les utilisateurs de votre entreprise se connectent.

Par exemple :

- @firmex.com
- @firmex.ca

### <a name="step-2---contact-firmex-support-with-your-domains"></a>Étape 2 : Contacter le support technique Firmex avec vos domaines

Envoyez un e-mail à l’[équipe du support technique Firmex](mailto:support@firmex.com) ou appelez le 1888 688 4042 x.11 pour lui parler. Transmettez les informations sur votre domaine. Le support technique Firmex ajoutera les domaines à votre VDR en tant que **domaines revendiqués** . Votre administrateur doit maintenant configurer l’authentification unique.

Avertissement : Tant que l’administrateur de votre site ne configure pas les domaines revendiqués, les utilisateurs de votre entreprise ne pourront pas se connecter à la VDR. Les utilisateurs n’appartenant pas à l’entreprise (c’est-à-dire des utilisateurs invités) peuvent toujours se connecter à l’aide de leur adresse e-mail/mot de passe. La configuration doit prendre quelques minutes.

### <a name="step-3---configure-the-claimed-domains"></a>Étape 3 : Configurer les domaines revendiqués

1. Connectez-vous à Firmex en tant qu’administrateur de site.
1. Dans l’angle supérieur gauche, cliquez sur le logo de votre entreprise.
1. Sélectionnez l’onglet **SSO** (Authentification unique). Sélectionnez ensuite **SSO Configuration** (Configuration de l’authentification unique). Cliquez sur le domaine que vous souhaitez configurer.

    ![Domaines revendiqués](./media/firmex-vdr-tutorial/edit-sso.png)  

1. Demandez à votre administrateur informatique de renseigner les champs suivants. Les champs doivent être extraits de votre fournisseur d’identité :  

    ![SSO Configuration](./media/firmex-vdr-tutorial/SSO-config.png)

    a. Dans la zone de texte **ID d’entité** , collez l’ **Identificateur Azure AD** que vous avez copié sur le portail Azure.

    b. Dans la zone de texte **URL du fournisseur d’identité** , collez la valeur de l’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    c. **Certificat de clé publique**  : à des fins d’authentification, un message SAML peut être signé numériquement par l’émetteur. Pour vérifier la signature du message, le destinataire utilise une clé publique connue pour appartenir à l’émetteur. De même, pour chiffrer un message, une clé de chiffrement publique appartenant au dernier destinataire doit être connue de l’émetteur. Dans les deux cas, la signature et le chiffrement, des clés publiques approuvées doivent être partagées à l’avance.  Il s’agit du **certificat X509** provenant du **XML de métadonnées de fédération**

    d. Cliquez sur **Enregistrer** pour terminer la configuration de l’authentification unique. Les modifications prennent effet immédiatement.

1. À ce stade, l’authentification unique est activée pour votre site.

### <a name="create-firmex-vdr-test-user"></a>Créer un utilisateur de test Firmex VDR

Dans cette section, vous allez créer un utilisateur appelé B.Simon dans Firmex. Collaborez avec l’[équipe du support technique Firmex](mailto:support@firmex.com) pour ajouter les utilisateurs dans la plateforme Firmex. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Firmex VDR dans le volet d’accès, vous devez être connecté automatiquement à l’application Firmex VDR pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Firmex VDR avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Firmex avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)