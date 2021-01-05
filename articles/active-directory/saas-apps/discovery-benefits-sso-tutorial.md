---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Discovery Benefits SSO | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Discovery Benefits SSO.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/03/2019
ms.author: jeedes
ms.openlocfilehash: 0e5d67e00ee56b5c4006a8422c713e3cabb32bfc
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92454760"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-discovery-benefits-sso"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Discovery Benefits SSO

Dans ce tutoriel, vous allez apprendre à intégrer Discovery Benefits SSO à Azure Active Directory (Azure AD). Quand vous intégrez Discovery Benefits SSO à Azure AD, vous pouvez :

* Dans Azure AD, contrôler qui a accès à Discovery Benefits SSO.
* Permettre à vos utilisateurs de se connecter automatiquement à Discovery Benefits SSO avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Discovery Benefits SSO pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Discovery Benefits SSO prend en charge l’authentification unique lancée par le **fournisseur d’identité**

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-discovery-benefits-sso-from-the-gallery"></a>Ajout de Discovery Benefits SSO à partir de la galerie

Pour configurer l’intégration de Discovery Benefits SSO à Azure AD, vous devez ajouter Discovery Benefits SSO, disponible dans la galerie, à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **Discovery Benefits SSO** dans la zone de recherche.
1. Sélectionnez **Discovery Benefits SSO** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-discovery-benefits-sso"></a>Configurer et tester l’authentification unique Azure AD pour Discovery Benefits SSO

Configurez et testez l’authentification unique Azure AD avec Discovery Benefits SSO à l’aide d’un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Discovery Benefits SSO associé.

Pour configurer et tester l’authentification unique Azure AD avec Discovery Benefits SSO, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique Discovery Benefits SSO](#configure-discovery-benefits-sso-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Discovery Benefits SSO](#create-discovery-benefits-sso-test-user)** pour avoir un équivalent de B.Simon dans Discovery Benefits SSO lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Discovery Benefits SSO** , recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , l’application est préconfigurée en mode Lancement par le **fournisseur d’identité** et les URL nécessaires sont déjà préremplies avec Azure. L’utilisateur doit enregistrer la configuration en cliquant sur le bouton **Enregistrer**.

1. L’application Discovery Benefits SSO s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration Attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut. Cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue Attributs d’utilisateur.

    ![image](common/edit-attribute.png)

    a. Cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Identificateur d’utilisateur unique (ID nom)** .

    ![Capture d’écran montrant la section « Attributs utilisateur et revendications » avec les ellipses « Revendication requise » sur le côté droit sélectionnées.](./media/discovery-benefits-sso-tutorial/attribute01.png)

    ![Configuration de Discovery Benefits SSO](./media/discovery-benefits-sso-tutorial/attribute02.png)

    b. Cliquez sur l’icône **Modifier** pour ouvrir la boîte de dialogue **Gérer la transformation**.

    c. Dans la zone de texte **Transformation** , tapez la chaîne **ToUppercase()** indiquée pour cette ligne.

    d. Dans la zone de texte **Paramètre 1** , tapez le paramètre tel que `<Name Identifier value>`.

    e. Cliquez sur **Add**.

    > [!NOTE]
    > Discovery Benefits SSO requiert la transmission d’une valeur de chaîne fixe dans **Identificateur d’utilisateur unique (ID nom)** pour que cette intégration fonctionne. Azure AD ne prend pas en charge cette fonctionnalité ; en guise de solution de contournement, vous pouvez donc utiliser des transformations **ToUpper** ou **ToLower** de NameID pour définir une valeur de chaîne fixe comme indiqué ci-dessus dans la capture d’écran.

    f. Nous avons rempli automatiquement les revendications supplémentaires requises pour la configuration de l’authentification unique (`SSOInstance` et `SSOID`). Utilisez l’icône **Modifier** pour mapper les valeurs en fonction de votre organisation.

    ![Capture d’écran montrant « Attributs utilisateur et revendications » avec les valeurs « SSO Instance » et « SSOID » mises en évidence.](./media/discovery-benefits-sso-tutorial/attribute03.png)

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Discovery Benefits SSO** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B. Simon dans le portail Azure.

1. Dans le volet gauche du Portail Azure, sélectionnez **Azure Active Directory** , **Utilisateurs** , puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Dans le champ **Nom** , entrez `B.Simon`.  
   1. Dans le champ **Nom de l’utilisateur** , entrez username@companydomain.extension. Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ **Mot de passe**.
   1. Cliquez sur **Créer**.

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Discovery Benefits SSO.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Discovery Benefits SSO**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

## <a name="configure-discovery-benefits-sso-sso"></a>Configurer l’authentification unique Discovery Benefits SSO

Pour configurer l’authentification unique côté **Discovery Benefits SSO** , vous devez envoyer le **certificat (Base64)** téléchargé et les URL appropriées copiées à partir du portail Azure à l’ [équipe du support technique Discovery Benefits SSO](mailto:Jsimpson@DiscoveryBenefits.com). Celles-ci configurent ensuite ce paramètre pour que la connexion SSO SAML soit définie correctement des deux côtés.

### <a name="create-discovery-benefits-sso-test-user"></a>Créer un utilisateur de test Discovery Benefits SSO

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans Discovery Benefits SSO. Collaborez avec l’[équipe de support technique Discovery Benefits SSO](mailto:Jsimpson@DiscoveryBenefits.com) pour ajouter les utilisateurs à la plateforme Discovery Benefits SSO. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Discovery Benefits SSO dans le volet d’accès, vous devez être connecté automatiquement à l’application Discovery Benefits SSO pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Discovery Benefits SSO avec Azure AD](https://aad.portal.azure.com/)