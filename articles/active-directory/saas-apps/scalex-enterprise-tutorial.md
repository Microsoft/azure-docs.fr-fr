---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à ScaleX Enterprise | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et ScaleX Enterprise.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/16/2019
ms.author: jeedes
ms.openlocfilehash: a2065286581b6e7de61e4d1a29382b98563cdbb8
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895054"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-scalex-enterprise"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à ScaleX Enterprise

Dans ce tutoriel, vous allez apprendre à intégrer ScaleX Enterprise à Azure Active Directory (Azure AD). Quand vous intégrez ScaleX Enterprise à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à ScaleX Enterprise.
* Permettre à vos utilisateurs d’être automatiquement connectés à ScaleX Enterprise avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement ScaleX Enterprise pour lequel l’authentification unique (SSO) est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* ScaleX Enterprise prend en charge l’authentification unique (SSO) lancée par **le fournisseur de services et le fournisseur d’identité**

## <a name="adding-scalex-enterprise-from-the-gallery"></a>Ajout de ScaleX Enterprise à partir de la galerie

Pour configurer l’intégration de ScaleX Enterprise à Azure AD, vous devez ajouter ScaleX Enterprise de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie** , tapez **ScaleX Enterprise** dans la zone de recherche.
1. Sélectionnez **ScaleX Enterprise** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-scalex-enterprise"></a>Configurer et tester l’authentification unique (SSO) Azure AD pour ScaleX Enterprise

Configurez et testez l’authentification unique (SSO) Azure AD avec ScaleX Enterprise à l’aide d’un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique (SSO) fonctionne, vous devez établir une relation entre un utilisateur Azure AD et l’utilisateur ScaleX Enterprise associé.

Pour configurer et tester l’authentification unique (SSO) Azure AD avec ScaleX Enterprise, suivez les modules suivants :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    * **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    * **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique (SSO) ScaleX Enterprise](#configure-scalex-enterprise-sso)** pour configurer les paramètres de l’authentification unique côté application.
    * **[Créer un utilisateur de test ScaleX Enterprise](#create-scalex-enterprise-test-user)** pour disposer, dans ScaleX Enterprise, d’un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans la page d’intégration de l’application **ScaleX Enterprise** du [portail Azure](https://portal.azure.com/), recherchez la section **Gérer** , puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://platform.rescale.com/saml2/<company id>/`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://platform.rescale.com/saml2/<company id>/acs/`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://platform.rescale.com/saml2/<company id>/sso/`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. Pour obtenir ces valeurs, contactez [l’équipe de support technique ScaleX Enterprise](https://info.rescale.com/contact_sales). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Votre application ScaleX Enterprise s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration Attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut, où **emailaddress** est mappé à **user.mail**. L’application ScaleX Enterprise s’attend à ce que **emailaddress** soit mappé à **user.userprincipalname**. Vous devez donc modifier le mappage d’attribut en cliquant sur l’icône **Modifier**.

    ![image](common/edit-attribute.png)

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , recherchez **Certificat (en base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer ScaleX Enterprise** , copiez la ou les URL appropriées, selon vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à ScaleX Enterprise.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **ScaleX Enterprise**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer**.

## <a name="configure-scalex-enterprise-sso"></a>Configurer l’authentification unique (SSO) ScaleX Enterprise

1. Pour automatiser la configuration dans ScaleX Enterprise, vous devez installer l’ **extension de navigateur My Apps Secure Sign-in** en cliquant sur **Installer l’extension**.

    ![Extension My apps](common/install-myappssecure-extension.png)

1. Après l’ajout de l’extension au navigateur, cliquez sur **Configurer ScaleX Enterprise** pour être dirigé vers l’application ScaleX Enterprise. À partir de là, indiquez les informations d’identification de l’administrateur pour vous connecter à ScaleX Enterprise. Cette extension de navigateur configure automatiquement l’application et automatise les étapes 3 à 6.

    ![Configuration](common/setup-sso.png)

1. Si vous souhaitez configurer manuellement ScaleX Enterprise, ouvrez une nouvelle fenêtre de navigateur web, connectez-vous à votre site d’entreprise ScaleX Enterprise en tant qu’administrateur et effectuez les étapes suivantes :

1. Cliquez sur le menu dans le coin supérieur droit et sélectionnez **Contoso Administration**.

    > [!NOTE]
    > Contoso est simplement un exemple. Ce doit être le nom réel de votre société.

    ![Capture d’écran montrant un exemple de nom de société sélectionné à partir du menu en haut à droite.](./media/scalex-enterprise-tutorial/Test_Admin.png)

1. Sélectionnez **Intégrations** dans le menu du haut, puis **Authentification unique**.

    ![Capture d’écran montrant l’élément « Intégrations » sélectionné ainsi que l’option « Authentification unique » sélectionnée dans le menu déroulant.](./media/scalex-enterprise-tutorial/admin_sso.png) 

1. Remplissez le formulaire comme suit :

    ![Configurer l’authentification unique](./media/scalex-enterprise-tutorial/scalex_admin_save.png)

    a. Sélectionnez **Créer tout utilisateur pouvant s’authentifier avec l’authentification unique**

    b. **Service Provider saml** (SAML du fournisseur de services) : Collez la valeur * *_urn:oasis:names:tc:SAML:2.0:nameid-format:persistent_* _

    c. _*Name of Identity Provider email field in ACS response** (Nom du champ d’adresse e-mail du fournisseur d’identité dans la réponse ACS) : Collez la valeur `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`

    d. **Identity Provider EntityDescriptor Entity ID** (ID d’entité EntityDescriptor du fournisseur d’identité) : Collez la valeur **Identificateur Azure AD** copiée à partir du portail Azure.

    e. **URL SingleSignOnService du fournisseur d’identité :** Collez l’ **URL de connexion** à partir du portail Azure.

    f. **Identity Provider public X509 certificate** (Certificat X509 public du fournisseur d’identité) : Ouvrez le certificat X509 téléchargé à partir d’Azure dans le bloc-notes et collez le contenu dans cette zone. Assurez-vous qu’aucun saut de ligne ne se trouve au milieu du contenu du certificat.

    g. Cochez les cases suivantes : **Enabled (Activé), Encrypt NameID (Chiffrer NameID) et Sign AuthnRequests (Signer AuthnRequests).**

    h. Cliquez sur **Mettre à jour les paramètres de l’authentification unique** pour enregistrer les paramètres.

### <a name="create-scalex-enterprise-test-user"></a>Créer un utilisateur de test ScaleX Enterprise

Pour permettre aux utilisateurs Azure AD de se connecter à ScaleX Enterprise, vous devez les attribuer dans ScaleX Enterprise. Dans le cas de ScaleX Enterprise, cette configuration est une tâche automatique et aucune opération manuelle n’est nécessaire. Tout utilisateur authentifié avec des informations d’authentification unique est automatiquement configuré du côté ScaleX.

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Le fait de cliquer sur la vignette ScaleX Enterprise dans le volet d’accès doit vous connecter automatiquement à l’application ScaleX Enterprise pour laquelle vous avez configuré l’authentification unique (SSO). Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer ScaleX Enterprise avec Azure AD](https://aad.portal.azure.com/)