---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Check Point CloudGuard Dome9 Arc | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Check Point CloudGuard Dome9 Arc.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/17/2019
ms.author: jeedes
ms.openlocfilehash: 0dee0b84b249446755d5124a8477e993e13e52b0
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92454709"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-check-point-cloudguard-dome9-arc"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Check Point CloudGuard Dome9 Arc

Dans ce tutoriel, vous allez apprendre à intégrer Check Point CloudGuard Dome9 Arc à Azure Active Directory (Azure AD). Quand vous intégrez Check Point CloudGuard Dome9 Arc à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Check Point CloudGuard Dome9 Arc
* Permettre aux utilisateurs de se connecter automatiquement à Check Point CloudGuard Dome9 Arc à l’aide de leurs comptes Azure AD
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Check Point CloudGuard Dome9 Arc pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Check Point CloudGuard Dome9 Arc prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-check-point-cloudguard-dome9-arc-from-the-gallery"></a>Ajout de Check Point CloudGuard Dome9 Arc à partir de la galerie

Pour configurer l’intégration de Check Point CloudGuard Dome9 Arc à Azure AD, vous devez ajouter Check Point CloudGuard Dome9 Arc à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **Check Point CloudGuard Dome9 Arc** dans la zone de recherche.
1. Sélectionnez **Check Point CloudGuard Dome9 Arc** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-single-sign-on-for-check-point-cloudguard-dome9-arc"></a>Configurer et tester l’authentification unique Azure AD pour Check Point CloudGuard Dome9 Arc

Configurez et testez l’authentification unique Azure AD avec Check Point CloudGuard Dome9 Arc pour un utilisateur de test nommé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Check Point CloudGuard Dome9 Arc associé.

Pour configurer et tester l’authentification unique Azure AD avec Check Point CloudGuard Dome9 Arc, suivez les indications des modules ci-dessous :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurez l’authentification unique Check Point CloudGuard Dome9 Arc](#configure-check-point-cloudguard-dome9-arc-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créez l’utilisateur de test Check Point CloudGuard Dome9 Arc](#create-check-point-cloudguard-dome9-arc-test-user)** pour avoir un équivalent de B.Simon dans Check Point CloudGuard Dome9 Arc lié à la représentation Azure AD de cet utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **Check Point CloudGuard Dome9 Arc** , recherchez la section **Gérer** , puis sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL : `https://secure.dome9.com/`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://secure.dome9.com/sso/saml/<yourcompanyname>`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://secure.dome9.com/sso/saml/<yourcompanyname>`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de réponse et l’URL de connexion réelles. Vous obtiendrez la valeur `<company name>` en suivant la procédure décrite dans la section **Configurer l’authentification Check Point CloudGuard Dome9 Arc** , plus loin dans ce tutoriel. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. L’application Check Point CloudGuard Dome9 Arc s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration de vos attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/edit-attribute.png)

1. En plus de ce qui précède, l’application Check Point CloudGuard Dome9 Arc s’attend à ce que quelques attributs supplémentaires (indiqués ci-dessous) soient passés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.
    
    | Nom |  Attribut source|
    | ---------------| --------------- |
    | memberof | user.assignedroles |

    >[!NOTE]
    >Cliquez [ici](./apptio-tutorial.md) pour découvrir comment créer des rôles dans Azure AD.

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer Check Point CloudGuard Dome9 Arc** , copiez les URL appropriées en fonction des exigences.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Check Point CloudGuard Dome9 Arc.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **Check Point CloudGuard Dome9 Arc** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-check-point-cloudguard-dome9-arc-sso"></a>Configurer l’authentification unique Check Point CloudGuard Dome9 Arc

1. Pour automatiser la configuration dans Check Point CloudGuard Dome9 Arc, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension** .

    ![Extension My apps](common/install-myappssecure-extension.png)

2. Après avoir ajouté l’extension au navigateur, cliquez sur **Configurer Check Point CloudGuard Dome9 Arc** afin d’accéder à l’application Check Point CloudGuard Dome9 Arc. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à Check Point CloudGuard Dome9 Arc. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 6.

    ![Configuration](common/setup-sso.png)

3. Si vous souhaitez configurer manuellement Check Point CloudGuard Dome9 Arc, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise Check Point CloudGuard Dome9 Arc en tant qu’administrateur et effectuez les étapes suivantes :

2. Cliquez sur les **paramètres du profil** dans l’angle supérieur droit puis cliquez sur les **paramètres du compte** . 

    ![Capture d’écran montrant le menu « Profile Settings » avec l’option « Account Settings » sélectionnée.](./media/dome9arc-tutorial/configure1.png)

3. Accédez à **SSO** , puis cliquez sur **ACTIVER** .

    ![Capture d’écran montrant l’onglet « SSO » et l’option « Enable » sélectionnée.](./media/dome9arc-tutorial/configure2.png)

4. Dans la section de la configuration de l’authentification unique, effectuez les étapes suivantes :

    ![Configuration de Check Point CloudGuard Dome9 Arc](./media/dome9arc-tutorial/configure3.png)

    a. Entrez le nom de la société dans la zone de texte **Account ID** (ID de compte). Cette valeur doit être utilisée dans l’URL de **réponse** et de **connexion** mentionnée dans la section **Configuration SAML de base** du portail Azure.

    b. Dans la zone de texte **Issuer (Émetteur)** , collez la valeur de l’ **identificateur Azure AD** que vous avez copiée à partir du portail Azure.

    c. Dans la zone de texte **Idp endpoint url** (URL du point de terminaison IDP), collez la valeur **URL de connexion** que vous avez copiée dans le portail Azure.

    d. Ouvrez le certificat codé en base64 que vous avez téléchargé dans le Bloc-notes, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **X.509 certificate** (Certificat X.509).

    e. Cliquez sur **Enregistrer** .

### <a name="create-check-point-cloudguard-dome9-arc-test-user"></a>Créer un utilisateur de test Check Point CloudGuard Dome9 Arc

Pour permettre aux utilisateurs Azure AD de se connecter à Check Point CloudGuard Dome9 Arc, vous devez les provisionner dans l’application. Check Point CloudGuard Dome9 Arc prend en charge le provisionnement juste-à-temps. Mais pour que cela fonctionne correctement, l’utilisateur doit sélectionner un **rôle** particulier et se l’attribuer.

   >[!Note]
   >Pour la création de **rôle** et pour d’autres informations, contactez [l’équipe du support technique Check Point CloudGuard Dome9 Arc](mailto:Dome9@checkpoint.com).

**Pour provisionner un compte d’utilisateur manuellement, effectuez les étapes suivantes :**

1. Connectez-vous au site d’entreprise Check Point CloudGuard Dome9 Arc en tant qu’administrateur.

2. Cliquez sur **Users & Roles** (Utilisateurs et rôles), puis cliquez sur **Users** (utilisateurs).

    ![Capture d’écran montrant « Users & Roles » avec l’action « Users » sélectionnée.](./media/dome9arc-tutorial/user1.png)

3. Cliquez sur **ADD USER** (Ajouter un utilisateur).

    ![Capture d’écran montrant « Users & Roles » avec le bouton « ADD USER » sélectionnée.](./media/dome9arc-tutorial/user2.png)

4. Dans la section **Create User** , procédez comme suit :

    ![Ajouter un employé](./media/dome9arc-tutorial/user3.png)

    a. Dans la zone de texte **Email** , entrez l’adresse e-mail de l’utilisateur, par exemple, B.Simon@contoso.com.

    b. Dans la zone de texte **First Name** (Prénom), tapez le prénom de l’utilisateur, par exemple Britta.

    c. Dans la zone de texte **Last Name** , entrez le nom de l’utilisateur, par exemple Simon.

    d. Définissez **SSO User** (SSO utilisateur) sur **On** (Activé).

    e. Cliquez sur **CREATE** (Créer).

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette Check Point CloudGuard Dome9 Arc dans le volet d’accès, vous devez être automatiquement connecté à l’application Check Point CloudGuard Dome9 Arc pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Check Point CloudGuard Dome9 Arc avec Azure AD](https://aad.portal.azure.com/)