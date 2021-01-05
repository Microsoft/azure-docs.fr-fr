---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Workday | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Workday.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: jeedes
ms.openlocfilehash: af001ddd8dc468d06706e63eaf092d1179fe3fdc
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96181371"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-workday"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à Workday

Dans ce tutoriel, vous allez apprendre à intégrer Workday à Azure Active Directory (Azure AD). Quand vous intégrez Workday à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Workday.
* Permettre à vos utilisateurs de se connecter automatiquement à Workday avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement Workday pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* Workday prend en charge l’authentification unique lancée par le **fournisseur de services**.

* L’application Workday Mobile peut désormais être configurée avec Azure AD pour activer l’authentification unique. Pour plus d’informations sur la configuration, suivez [ce](workday-mobile-tutorial.md) lien.

> [!NOTE]
> L’identificateur de cette application étant une valeur de chaîne fixe, une seule instance peut être configurée dans un locataire.

## <a name="adding-workday-from-the-gallery"></a>Ajout de Workday à partir de la galerie

Pour configurer l’intégration de Workday à Azure AD, vous devez ajouter Workday à partir de la galerie à votre liste d’applications SaaS gérées.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **Workday** dans la zone de recherche.
1. Sélectionnez **Workday** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-workday"></a>Configurer et tester l’authentification unique Azure AD pour Workday

Configurez et testez l’authentification unique (SSO) Azure AD avec Workday à l’aide d’un utilisateur de test nommé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur Workday associé.

Pour configurer et tester l’authentification unique Azure AD avec Workday, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
2. **[Configurer Workday](#configure-workday)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test Workday](#create-workday-test-user)** pour disposer, dans Workday, d’un équivalent de B.Simon lié à la représentation Azure AD de l’utilisateur.
3. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Sur le portail Azure, accédez à la page d’intégration de l’application **Workday**, recherchez la section **Gérer**, puis sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la boîte de dialogue **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion**, tapez une URL au format suivant : `https://impl.workday.com/<tenant>/login-saml2.flex`.

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://impl.workday.com/<tenant>/login-saml.htmld`

    c. Dans la zone de texte **URL de déconnexion**, tapez une URL au format suivant : `https://impl.workday.com/<tenant>/login-saml.htmld`

    > [!NOTE]
    > Il ne s’agit pas des valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’URL de réponse et l’URL de déconnexion réels. Votre URL de réponse doit avoir un sous-domaine (par exemple, www, wd2, wd3, wd3-impl, wd5, wd5-impl).
    > `http://www.myworkday.com` fonctionne à la différence de `http://myworkday.com`. Pour obtenir ces valeurs, contactez l’[équipe de support technique de Workday](https://www.workday.com/en-us/partners-services/services/support.html). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Votre application Workday attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration Attributs du jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut, où **nameidentifier** est mappé avec **user.userprincipalname**. L’application Workday s’attend à ce que **nameidentifier** soit mappé avec **user.mail**, **UPN**, etc. Vous devez donc modifier le mappage d’attribut en cliquant sur l’icône **Modifier**.

    ![Capture d’écran montrant User Attributes avec l’icône Edit sélectionnée.](common/edit-attribute.png)

    > [!NOTE]
    > Ici, nous avons mappé l’ID de nom sur le nom d’utilisateur principal (user.userprincipalname) par défaut. Pour que l’authentification unique fonctionne correctement, vous devez mapper l’ID de nom sur l’identificateur d’utilisateur réel de votre compte Workday (votre adresse e-mail, votre nom d’utilisateur principal, etc.).

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **Certificat (Base64)** , puis sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

   ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Pour modifier les options **Signature** selon vos besoins, cliquez sur le bouton **Modifier** afin d’ouvrir la boîte de dialogue **Certificat de signature SAML**.

    ![Certificat](common/edit-certificate.png) 

    ![Certificat de signature SAML](./media/workday-tutorial/signing-option.png)

    a. Sélectionnez **Signer la réponse et l’assertion SAML** comme **Option de signature**.

    b. Cliquez sur **Enregistrer**.

1. Dans la section **Configurer Workday**, copiez l’URL ou les URL appropriées en fonction de vos besoins.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à Workday.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Workday**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-workday"></a>Configurer Workday

1. Dans une autre fenêtre de navigateur web, connectez-vous à votre site d’entreprise Workday en tant qu’administrateur.

1. Dans la **zone de recherche**, recherchez le nom **Edit Tenant Setup – Security** en haut à gauche de la page d’accueil.

    ![Modifier la sécurité du locataire](./media/workday-tutorial/IC782925.png "Modifier la sécurité du locataire")


1. Dans la section **Configuration de SAML**, procédez comme suit :

    ![SAML Setup](./media/workday-tutorial/IC782926.png "SAML Setup")

    a.  Sélectionnez **Enable SAML Authentication**.

    b.  Cliquez sur le **signe plus** pour ajouter une ligne.

1. Dans la section **SAML Identity Providers** (Fournisseurs d’identité SAML), effectuez les actions suivantes pour la ligne qui vient d’être créée.

    a. Effectuez les actions suivantes pour les champs en évidence ci-dessous.

    ![Fournisseurs d’identité SAML 1](./media/workday-tutorial/IC7829271.png "Fournisseurs d’identité SAML")

    * Dans la zone de texte **Identity Provider Name** (Nom du fournisseur d’identité), entrez le nom d’un fournisseur (par exemple : *SPInitiatedSSO*).

    * Dans le portail Azure, dans la fenêtre **Set up Workday** (Configurer Workday), copiez la valeur de **Identificateur Azure AD**, puis collez-la dans la zone de texte **Émetteur**.

    * Ouvrez le certificat (**Certificate**) téléchargé à partir du portail Azure dans le Bloc-notes, puis collez le contenu dans la zone de texte **x.509 Certificate** (Certificat x.509).

    b. Effectuez les actions suivantes pour les champs en évidence ci-dessous.

    ![Fournisseurs d’identité SAML 2](./media/workday-tutorial/saml-identity-provider-2.png "Fournisseurs d’identité SAML")

    * Cliquez sur la case à cocher **Enable IDP Initiated Logout** (Activer la déconnexion initiée par IDP).

    * Dans la zone de texte **Logout Response URL** (URL de réponse de déconnexion), tapez **http://www.workday.com** .

    * Dans la zone de texte **Logout Request URL** (URL de demande de déconnexion), collez la valeur de **Logout URL** (URL de déconnexion) que vous avez copiée à partir du portail Azure.

    * Cliquez sur la case à cocher **SP Initiated** (Initié par IDP).

    * Dans la zone de texte **ID fournisseur de services**, tapez **http://www.workday.com**.


    * Sélectionnez **Ne pas compresser une demande d'authentification initialisée par SP**.

    c. Effectuez les actions suivantes pour les champs en évidence ci-dessous.

    ![Fournisseurs d’identité SAML 3](./media/workday-tutorial/saml-identity-provider-3.png "Fournisseurs d’identité SAML")

    * Dans le portail Azure, dans la fenêtre **Set up Workday** (Configurer Workday), copiez l’**URL de connexion**, puis collez-la dans la zone de texte **IdP SSO Service URL** (URL du service d’authentification unique du fournisseur d'identité).

    * Dans la zone de texte **Used for Environments** (Utilisé pour les environnements), sélectionnez les noms d’environnement appropriés dans la liste déroulante.

1. Effectuez les étapes suivantes dans l’image ci-dessous.

    ![Workday](./media/workday-tutorial/service-provider.png "Fournisseurs d’identité SAML")

    a. Dans la zone de texte **Service Provider ID (Will be Deprecated)** [ID du fournisseur de services (prochainement déconseillé)], tapez **http://www.workday.com** .

    b. Dans la zone de texte **IDP SSO Service URL (Will be Deprecated)** [URL du service d’authentification unique IDP (prochainement déconseillé)], tapez la valeur de **Login URL** (URL de connexion).

    c. Sélectionnez **Do Not Deflate SP-initiated Authentication Request (Will be Deprecated)** [(Ne pas compresser la demande d’authentification initiée par le fournisseur de services (prochainement déconseillé)].

    d. Pour **Authentication Request Signature Method** (Méthode de signature de la demande d’authentification), sélectionnez **SHA256**.

    e. Cliquez sur **OK**.

    > [!NOTE]
    > Vérifiez que vous avez correctement configuré l’authentification unique. Si vous activez l’authentification unique avec une configuration incorrecte, vous risquez de ne pas pouvoir accéder à l’application avec vos informations d’identification et de vous retrouver verrouillé. Dans ce type de situation, Workday fournit une URL de connexion de secours permettant aux utilisateurs de se connecter en utilisant leurs nom d’utilisateur et mot de passe standard au format suivant : [votre URL Workday]/login.flex?redirect=n

### <a name="create-workday-test-user"></a>Créer un utilisateur de test Workday

1. Connectez-vous à votre site d’entreprise Workday en tant qu’administrateur.

1. Cliquez sur **Profile** en haut à droite, sélectionnez **Home** (Accueil), puis cliquez sur **Directory** (Répertoire) sous l’onglet **Applications**. 

1. Dans la page **Directory** (Répertoire), sélectionnez **Find Workers** (Rechercher des collaborateurs) sous l’onglet View (Affichage).

    ![Rechercher des collaborateurs](./media/workday-tutorial/user-directory.png)

1.  Dans la page **Find Workers** (Rechercher des collaborateurs), sélectionnez l’utilisateur dans les résultats.

1. Dans la page suivante, sélectionnez **Job > Worker Security (Sécurité du collaborateur)**  ; le compte Workday (**Workday account**) doit correspondre à l’instance Azure Active Directory comme ID de nom (**Name ID**).

    ![Sécurité du collaborateur](./media/workday-tutorial/worker-security.png)

> [!NOTE]
> Pour plus d’informations sur la création d’un utilisateur de test Workday, contactez l’[équipe de support client Workday](https://www.workday.com/en-us/partners-services/services/support.html).

## <a name="test-sso"></a>Tester l’authentification unique (SSO)

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

1. Cliquez sur **Tester cette application** dans le portail Azure. Vous êtes alors redirigé vers l’URL de connexion Workday où vous pouvez initier le flux de connexion. 

2. Accédez directement à l’URL de connexion Workday pour initier le flux de connexion.

3. Vous pouvez utiliser le volet d’accès Microsoft. Le fait de cliquer sur la vignette Workday dans le panneau d’accès doit vous connecter automatiquement à Workday pour lequel vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré Workday, vous pouvez appliquer le contrôle de session, qui protège en temps réel contre l’exfiltration et l’infiltration des données sensibles de votre organisation. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)