---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à EasySSO pour BitBucket | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et EasySSO pour BitBucket.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/18/2020
ms.author: jeedes
ms.openlocfilehash: 0433c0fdc1584ce209eb0409b0e8f5cbfc2719ed
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92454403"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-easysso-for-bitbucket"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à EasySSO pour BitBucket

Dans ce tutoriel, vous allez apprendre à intégrer EasySSO pour BitBucket à Azure Active Directory (Azure AD). Quand vous intégrez EasySSO pour BitBucket à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à EasySSO pour BitBucket.
* Permettre aux utilisateurs de se connecter automatiquement à EasySSO pour BitBucket avec leur compte Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS (software as a service) à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer des éléments suivants :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement EasySSO pour BitBucket pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* EasySSO pour BitBucket prend en charge l’authentification unique lancée par le fournisseur de services et le fournisseur d’identité.
* EasySSO pour BitBucket prend en charge l’attribution d’utilisateurs « juste-à-temps ».
* Après avoir configuré EasySSO pour BitBucket, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="add-easysso-for-bitbucket-from-the-gallery"></a>Ajouter EasySSO pour BitBucket à partir de la galerie

Pour configurer l’intégration d’EasySSO pour BitBucket dans Azure AD, vous devez ajouter EasySSO pour BitBucket, disponible dans la galerie, à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire, ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **EasySSO pour BitBucket** dans la zone de recherche.
1. Sélectionnez **EasySSO pour BitBucket** dans les résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-easysso-for-bitbucket"></a>Configurer et tester l’authentification unique Azure AD pour EasySSO pour BitBucket

Configurez et testez l’authentification unique Azure AD avec EasySSO pour BitBucket avec un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur EasySSO pour BitBucket qui lui est associé.

Pour configurer et tester l’authentification unique Azure AD avec EasySSO pour BitBucket, effectuez les étapes suivantes :

1. [Configurer l’authentification unique Azure AD](#configure-azure-ad-sso) pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. [Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user) pour tester l’authentification unique Azure AD avec B.Simon.
    1. [Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user) pour permettre à B.Simon d’utiliser l’authentification unique Azure AD.
1. [Configurer l’authentification unique EasySSO pour BitBucket](#configure-easysso-for-bitbucket-sso) afin de configurer les paramètres de l’authentification unique côté application.
    1. [Créer un utilisateur de test EasySSO pour BitBucket](#create-an-easysso-for-bitbucket-test-user) afin d’avoir dans EasySSO pour BitBucket un équivalent de B.Simon qui soit lié à sa représentation dans Azure AD.
1. [Tester l’authentification unique](#test-sso) pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **EasySSO pour BitBucket** , et recherchez la section **Gérer** . Sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , sélectionnez l’icône de crayon pour **Configuration SAML de base** afin de modifier les paramètres.

   ![Capture d’écran de la page Configurer l’authentification unique avec SAML avec l’icône de crayon mise en évidence](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous voulez configurer l’application en mode **lancé par le fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , saisissez une URL au format suivant : `https://<server-base-url>/plugins/servlet/easysso/saml`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/easysso/saml`

1. Sélectionnez **Définir des URL supplémentaires** et effectuez l’étape suivante si vous souhaitez configurer l’application en mode lancé par le **fournisseur de services**  :

    - Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<server-base-url>/login.jsp`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. En cas de doute, contactez l’[équipe du support technique EasySSO](mailto:support@techtime.co.nz) pour obtenir ces valeurs. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Votre application EasySSO pour BitBucket attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![Capture d’écran des attributs par défaut](common/default-attributes.png)

1. L’application EasySSO pour BitBucket s’attend aussi à ce que quelques attributs supplémentaires soient repassés dans la réponse SAML. Le tableau suivant contient ces attributs. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.
    
    | Nom | Attribut source|
    | ---------------| --------- |
    | urn:oid:0.9.2342.19200300.100.1.1 | user.userprincipalname |
    | urn:oid:0.9.2342.19200300.100.1.3 | user.mail |
    | urn:oid:2.16.840.1.113730.3.1.241 | user.displayname |
    | urn:oid:2.5.4.4 | user.surname |
    | urn:oid:2.5.4.42 | user.givenname |
    
    Si vos utilisateurs Azure AD ont configuré **sAMAccountName** , vous devez mapper **urn:oid:0.9.2342.19200300.100.1.1** sur l’attribut **sAMAccountName** .
    
1. Dans la page **Configurer l’authentification unique avec SAML** , à la section **Certificat de signature SAML** , sélectionnez les liens de téléchargement pour les options **Certificat (Base64)** ou **XML de métadonnées de fédération** . Enregistrez l’un ou l’autre, ou les deux, sur votre ordinateur. Vous en aurez besoin ultérieurement pour configurer EasySSO pour BitBucket.

    ![Capture d’écran de la section Certificat de signature SAML, avec mise en évidence des liens de téléchargement](./media/easysso-for-bitbucket-tutorial/certificate.png)
    
    Si vous envisagez de configurer EasySSO pour BitBucket manuellement avec un certificat, vous devez également copier l’ **URL de connexion** et l’ **Identificateur Azure AD** , puis les enregistrer sur votre ordinateur.

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test, B.Simon, dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs** .
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur** , effectuez les étapes suivantes :
   1. Pour **Nom** , entrez `B.Simon`.  
   1. Entrez username@companydomain.extension comme **Nom d’utilisateur** . Par exemple : `B.Simon@contoso.com`.
   1. Cochez la case **Afficher le mot de passe** et notez celui-ci.
   1. Sélectionnez **Create** (Créer).

### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à EasySSO pour BitBucket.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** > **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **EasySSO pour BitBucket** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Capture d’écran de la section Gérer, avec Utilisateurs et groupes mis en évidence](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** . Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Utilisateurs et groupes** .

    ![Capture d’écran de la page Utilisateurs et groupes, avec Ajouter un utilisateur mis en évidence](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B.Simon** dans la liste **Utilisateurs** , puis choisissez **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste. Choisissez **Sélectionner** au bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , sélectionnez **Affecter** .

## <a name="configure-easysso-for-bitbucket-sso"></a>Configurer l’authentification unique côté EasySSO pour BitBucket

1. Connectez-vous à votre instance d’Atlassian BitBucket avec des privilèges d’administrateur et accédez à la section **Administration** . 

    ![Capture d’écran de l’instance BitBucket, avec l’icône en forme de roue dentée mise en évidence](./media/easysso-for-bitbucket-tutorial/bitbucket-admin-1.png)
1. Recherchez puis sélectionnez **EasySSO** .

    ![Capture d’écran de l’option Easy SSO](./media/easysso-for-bitbucket-tutorial/bitbucket-admin-2.png)

1. Sélectionnez **SAML** . Vous accédez alors à la section de configuration SAML.

    ![Capture d’écran de la page d’administration EasySSO, avec l’option SAML mise en évidence](./media/easysso-for-bitbucket-tutorial/bitbucket-admin-3.png)

1. Sélectionnez l’onglet **Certificats** pour obtenir l’écran suivant :

    ![Capture d’écran de l’onglet Certificats, avec différentes options mises en évidence](./media/easysso-for-bitbucket-tutorial/bitbucket-admin-4.png)

1. Localisez le **certificat (Base64)** ou le **fichier de métadonnées** que vous avez enregistré dans la section précédente de ce tutoriel. Pour cela, vous pouvez procéder de la manière suivante :

    - Utilisez le **fichier de métadonnées** de fédération d’application que vous avez téléchargé dans un fichier local sur votre ordinateur. Sélectionnez la case d’option **Charger** , puis suivez les instructions concernant votre système d’exploitation.

    - Ouvrez le **fichier de métadonnées** de fédération d’application pour afficher son contenu dans n’importe quel éditeur de texte brut. Copiez-le dans le Presse-papiers. Sélectionnez **Input** (Entrée) et collez le contenu du Presse-papiers dans le champ de texte.
 
    - Procédez à une configuration entièrement manuelle. Ouvrez le **certificat (Base64)** de fédération d’application pour afficher son contenu dans n’importe quel éditeur de texte brut. Copiez-le dans le Presse-papiers, puis collez-le dans le champ de texte **IdP Token Signing Certificates** (Certificats de signature de jetons du fournisseur d’identité). Accédez ensuite à l’onglet **General** et renseignez les champs **POST Binding URL** (URL de liaison POST) et **Entity ID** (ID d’entité) avec les valeurs respectives d’ **URL de connexion** et d’ **Identificateur Azure AD** que vous avez enregistrées plus tôt.
 
1. Sélectionnez **Save** (Enregistrer) en bas de la page. Le contenu des fichiers de métadonnées ou de certificat sera alors analysé dans les champs de configuration. La configuration d’EasySSO for BitBucket est terminée.

1. Pour tester la configuration, accédez à l’onglet **Look & Feel** (Apparence), puis sélectionnez **SAML Login Button** (Bouton de connexion SAML). Un bouton distinct s’affichera alors dans l’écran de connexion BitBucket pour vous permettre de tester votre intégration SAML Azure AD de bout en bout. Vous pouvez également laisser ce bouton activé et configurer son emplacement, sa couleur et sa traduction pour le mode de production.

    ![Capture d’écran de l’onglet Look & Feel (Apparence) de la page SAML, avec le bouton de connexion SAML Login Button (Bouton de connexion SAML) mis en évidence](./media/easysso-for-bitbucket-tutorial/bitbucket-admin-5.png)
    > [!NOTE]
    >Si vous rencontrez des problèmes, contactez l’[équipe du support technique EasySSO](mailto:support@techtime.co.nz).

### <a name="create-an-easysso-for-bitbucket-test-user"></a>Créer un utilisateur de test EasySSO pour BitBucket

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans BitBucket. EasySSO for BitBucket prend en charge l’attribution d’utilisateurs juste-à-temps, qui est désactivée par défaut. Pour l’activer, vous devez activer explicitement l’option **Create user on successful login** (Créer un utilisateur lors d’une connexion réussie) dans la section **General** de la configuration du plug-in EasySSO. S’il n’existe pas encore d’utilisateur dans BitBucket, il en est créé un après l’authentification.

Toutefois, si vous ne souhaitez pas activer l’attribution automatique d’utilisateurs lorsque l’utilisateur se connecte pour la première fois, les annuaires utilisateur que l’instance BitBucket utilise doivent contenir des utilisateurs. Par exemple, l’annuaire en question peut être LDAP ou Atlassian.

![Capture d’écran de la section General de la configuration du plug-in EasySSO, avec l’option Create user on successful login (Créer un utilisateur lors d’une connexion réussie) mise en évidence](./media/easysso-for-bitbucket-tutorial/bitbucket-admin-6.png)

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

### <a name="idp-initiated-workflow"></a>Workflow lancé par le fournisseur d’identité

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous sélectionnez la vignette EasySSO pour BitBucket, vous devez être automatiquement connecté à l’instance de BitBucket pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations, consultez [Se connecter et démarrer des applications à partir du portail Mes applications](../user-help/my-apps-portal-end-user-access.md).

### <a name="sp-initiated-workflow"></a>Workflow lancé par le fournisseur de services

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du bouton **SAML Login** de BitBucket.

![Capture d’écran de l’écran Connexion, avec la connexion SAML mise en évidence](./media/easysso-for-bitbucket-tutorial/bitbucket-admin-7.png)

Ce scénario part du principe que vous avez activé le **bouton de connexion SAML** sous l’onglet **Look & Feel** dans votre page de configuration EasySSO pour BitBucket. Ouvrez votre URL de connexion BitBucket en mode incognito du navigateur pour éviter toute interférence avec vos sessions existantes. Sélectionnez le bouton **SAML Login** (Connexion SAML). Vous êtes alors redirigé vers le flux d’authentification des utilisateurs Azure AD. Une fois l’opération terminée, vous serez redirigé vers votre instance de BitBucket en tant qu’utilisateur authentifié par le biais de SAML.

Il est possible que vous rencontriez l’écran suivant après avoir été redirigé à partir d’Azure AD :

![Capture d’écran de l’échec EasySSO, avec le numéro de référence mis en évidence](./media/easysso-for-bitbucket-tutorial/bitbucket-admin-8.png)

Dans ce cas, vous devez suivre les [instructions fournies dans cette page](https://techtime.co.nz/display/TECHTIME/EasySSO+How+to+get+the+logs#EasySSOHowtogetthelogs-RETRIEVINGTHELOGS) pour accéder au fichier **atlassian-bitbucket.log** . Les détails de l’erreur sont disponibles en fonction de l’ID de référence figurant dans la page d’erreur EasySSO.

Si vous rencontrez des problèmes, contactez l’[équipe du support technique EasySSO](mailto:support@techtime.co.nz).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels pour l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer EasySSO pour BitBucket avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Protéger EasySSO pour BitBucket avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)