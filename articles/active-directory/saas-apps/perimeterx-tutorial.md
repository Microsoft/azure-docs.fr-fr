---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à PerimeterX | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et PerimeterX.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/27/2020
ms.author: jeedes
ms.openlocfilehash: 29ffaaa1b1b6efbcd5523a76018c92645e13d187
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96181794"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-perimeterx"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à PerimeterX

Dans ce tutoriel, vous allez apprendre à intégrer PerimeterX à Azure Active Directory (Azure AD). Quand vous intégrez PerimeterX à Azure AD, vous pouvez :

* Contrôler qui dans Azure AD a accès à PerimeterX.
* Permettre aux utilisateurs de se connecter automatiquement à PerimeterX avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement PerimeterX pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.


* PerimeterX prend en charge l’authentification unique lancée par le **fournisseur d’identité**

## <a name="adding-perimeterx-from-the-gallery"></a>Ajout de PerimeterX à partir de la galerie

Pour configurer l’intégration de PerimeterX à Azure AD, vous devez ajouter PerimeterX à partir de la galerie à votre liste d’applications SaaS managées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, entrez **PerimeterX** dans la zone de recherche.
1. Sélectionnez **PerimeterX** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-perimeterx"></a>Configurer et tester l’authentification unique Azure AD pour PerimeterX

Configurez et testez l’authentification unique Azure AD avec PerimeterX pour un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans PerimeterX.

Pour configurer et tester l’authentification unique Azure AD avec PerimeterX, procédez comme suit :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique PerimeterX](#configure-perimeterx-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test PerimeterX](#create-perimeterx-test-user)** pour avoir un équivalent de B.Simon dans PerimeterX lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **PerimeterX**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, l’application est préconfigurée et les URL nécessaires sont déjà préremplies avec Azure. L’utilisateur doit enregistrer la configuration en cliquant sur le bouton **Enregistrer**.


1. L’application PerimeterX s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jetons SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application PerimeterX s’attend à ce que quelques attributs supplémentaires (présentés ci-dessous) soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.
    
    | Nom | Attribut source|
    | ------------ | --------- |
    | firstName | user.givenname |
    | lastName  | user.surname |

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer PerimeterX**, copiez la ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à PerimeterX.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **PerimeterX**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-perimeterx-sso"></a>Configurer l’authentification unique PerimeterX

1. Connectez-vous à votre console PerimeterX avec les autorisations d’administrateur.

1. Accédez à **Admin > ACCOUNTS** (Admin > COMPTES). 

    ![Authentification unique PerimeterX](./media/perimeterx-tutorial/accounts.png)

1. Cliquez sur **Modifier**

4.  Dans la page Edit Account (Modifier le compte), effectuez les étapes suivantes.

    ![Authentification unique PerimeterX : modifier le compte](./media/perimeterx-tutorial/saml-sso.png)

    a.  Cochez **Enable Single Sign-On (SSO)** (Activer l’authentification unique (SSO))

    b.  Sélectionnez **Azure SAML**.

    c.  Dans la zone de texte **SAML Endpoint** (Point de terminaison SAML), collez l’**URL de connexion** que vous avez copiée à partir du portail Azure.

    d. Dans la zone **Issuer** (Émetteur), collez la valeur Identificateur Azure AD copiée à partir du portail Azure.

    e. Ouvrez le **Certificat (en base64)** téléchargé à partir du portail Azure dans le Bloc-notes, puis collez le contenu dans la zone de texte **X.509 Certificate**.

    f. Cliquez sur **Save Changes** (Enregistrer les changements).

### <a name="create-perimeterx-test-user"></a>Créer un utilisateur de test PerimeterX

Pour obtenir des instructions sur la création de l’utilisateur de test PerimeterX, reportez-vous au document [PerimeterX Managing Users Guide](https://docs.perimeterx.com/pxconsole/docs/managing-users).

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes.

1. Sur le portail Azure, cliquez sur Tester cette application. Vous êtes alors automatiquement connecté à l’instance de PerimeterX pour laquelle vous avez configuré l’authentification unique.

1. Vous pouvez utiliser le volet d’accès Microsoft. Le fait de cliquer sur la vignette PerimeterX dans le panneau d’accès doit vous connecter automatiquement à l’application PerimeterX pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).


## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré PerimeterX, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).