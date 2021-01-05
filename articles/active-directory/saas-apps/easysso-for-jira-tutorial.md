---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à EasySSO for Jira | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et EasySSO for Jira.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/28/2020
ms.author: jeedes
ms.openlocfilehash: b813f1f77b7bda2627f5275e772a3ee7c9d12834
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92454352"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-easysso-for-jira"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à EasySSO for Jira

Dans ce tutoriel, vous allez apprendre à intégrer EasySSO for Jira à Azure Active Directory (Azure AD). Quand vous intégrez EasySSO for Jira à Azure AD, vous pouvez :

* Contrôler dans Azure AD qui a accès à Jira.
* Permettez à vos utilisateurs de se connecter automatiquement à Jira avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

Pour en savoir plus sur l’intégration des applications SaaS à Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement EasySSO for Jira pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* EasySSO for Jira prend en charge l’authentification unique lancée par **le fournisseur de services et le fournisseur d’identité**
* EasySSO for Jira prend en charge le provisionnement d’utilisateurs **juste-à-temps**
* Après avoir configuré EasySSO for Jira, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-easysso-for-jira-from-the-gallery"></a>Ajout de EasySSO for Jira à partir de la galerie

Pour configurer l’intégration de EasySSO for Jira dans Azure AD, vous devez ajouter EasySSO for Jira, disponible dans la galerie, à votre liste d’applications SaaS managées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **EasySSO for Jira** dans la zone de recherche.
1. Sélectionnez **EasySSO for Jira** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-single-sign-on-for-easysso-for-jira"></a>Configurer et tester l’authentification unique Azure AD pour EasySSO for Jira

Configurez et testez l’authentification unique Azure AD avec EasySSO for Jira pour un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur EasySSO for Jira associé.

Pour configurer et tester l’authentification unique Azure AD avec EasySSO for Jira, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique EasySSO for Jira](#configure-easysso-for-jira-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test EasySSO for Jira](#create-easysso-for-jira-test-user)** pour avoir dans EasySSO for Jira un équivalent de B.Simon lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [Portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **EasySSO for Jira** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , si vous souhaitez configurer l’application en mode Initié par le **fournisseur d’identité** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **Identificateur** , tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/easysso/saml`

    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<server-base-url>/plugins/servlet/easysso/saml`

1. Si vous souhaitez configurer l’application en **mode démarré par le fournisseur de services** , cliquez sur **Définir des URL supplémentaires** , puis effectuez les étapes suivantes :

    Dans la zone de texte **URL de connexion** , tapez une URL au format suivant : `https://<server-base-url>/login.jsp`

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’identificateur, l’URL de réponse et l’URL de connexion réels. En cas de doute, contactez l’[équipe du support technique EasySSO](mailto:support@techtime.co.nz) pour obtenir ces valeurs. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Votre application EasySSO for Jira attend les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à votre configuration d’attributs de jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut.

    ![image](common/default-attributes.png)

1. En plus de ce qui précède, l’application EasySSO for Jira s’attend à ce que quelques attributs supplémentaires (présentés ci-dessous) soient repassés dans la réponse SAML. Ces attributs sont également préremplis, mais vous pouvez les examiner pour voir s’ils répondent à vos besoins.
    
    | Nom | Attribut source|
    | ---------------| --------- |
    | urn:oid:0.9.2342.19200300.100.1.1 | user.userprincipalname |
    | urn:oid:0.9.2342.19200300.100.1.3 | user.mail |
    | urn:oid:2.16.840.1.113730.3.1.241 | user.displayname |
    | urn:oid:2.5.4.4 | user.surname |
    | urn:oid:2.5.4.42 | user.givenname |
    
    Si vos utilisateurs Azure AD ont **sAMAccountName** configuré, vous devez mapper **urn:oid:0.9.2342.19200300.100.1.1** sur l’attribut **sAMAccountName** .
    
1. Dans la page **Configurer l’authentification unique avec SAML** , à la section **Certificat de signature SAML** , cliquez sur les liens **Télécharger** pour les options **Certificat (Base64)** ou **XML de métadonnées de fédération** , puis enregistrez l’un des deux, ou les deux, sur votre ordinateur. Vous en aurez besoin ultérieurement pour configurer EasySSO pour Jira.

    ![Lien Téléchargement de certificat](media/easysso-for-jira-tutorial/azure-ad-SAML-certificate.png)
    
    Si vous envisagez d’effectuer une configuration d’EasySSO pour JIRA manuellement avec un certificat, vous devez également copier l’ **URL de connexion** et l’ **Identificateur Azure AD** dans la section ci-dessous, puis les enregistrer sur votre ordinateur.

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

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à EasySSO for Jira.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , puis **Toutes les applications** .
1. Dans la liste des applications, sélectionnez **EasySSO for Jira** .
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes** .

   ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur** , puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Lien Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-easysso-for-jira-sso"></a>Configurer l’authentification unique (SSO) côté EasySSO for Jira

1. Connectez-vous à votre instance d’Atlassian Jira avec des privilèges d’administrateur et accédez à la section **Manage Apps** (Gérer les applications). 

    ![Gérer les applications](media/easysso-for-jira-tutorial/jira-admin-1.png)

2. Sur le côté gauche, recherchez **EasySSO** , puis cliquez dessus.

    ![Easy SSO](media/easysso-for-jira-tutorial/jira-admin-2.png)

3. Sélectionnez l’option **SAML** . Vous accédez alors à la section de configuration SAML.

    ![SAML](media/easysso-for-jira-tutorial/jira-admin-3.png)

4. Sélectionnez l’onglet **Certificats** en haut. L’écran suivant s’affiche alors :

    ![URL de métadonnées](media/easysso-for-jira-tutorial/jira-admin-4.png)

5. À présent, recherchez le **Certificat (Base64)** ou le **Fichier de métadonnées** que vous avez enregistré aux étapes précédentes de la configuration de l’ **authentification unique Azure AD** . Vous avez le choix entre les options suivantes :

    a. Utilisez le **fichier de métadonnées** de fédération d’application que vous avez téléchargé dans un fichier local sur votre ordinateur. Sélectionnez la case d’option **Charger** , puis suivez les instructions de la boîte de dialogue de chargement de fichier spécifique à votre système d’exploitation.

    **OR**

    b. Ouvrez le **fichier de métadonnées** de fédération d’application pour afficher le contenu (dans n’importe quel éditeur de texte brut) du fichier, et copiez-le dans le Presse-papiers. Sélectionnez l’option **Input** (Entrée) et collez le contenu du Presse-papiers dans le champ de texte.

     **OR**

    c. Configuration entièrement manuelle Ouvrez le **certificat (Base64)** de fédération d’application pour afficher le contenu (dans n’importe quel éditeur de texte brut) du fichier, et copiez-le dans le Presse-papiers. Collez-le dans le champ de texte **IdP Token Signing Certificates** (Certificats de signature de jetons IdP). Accédez ensuite à l’onglet **General** et renseignez les champs **POST Binding URL** (URL de liaison POST) et **Entity ID** (ID d’entité) avec les valeurs respectives d’ **URL de connexion** et d’ **Identificateur Azure AD** que vous avez enregistrées plus tôt.
 
6. Cliquez sur le bouton **Save** en bas de la page. Le contenu des fichiers de métadonnées ou de certificat sera alors analysé dans les champs de configuration. La configuration d’EasySSO pour Jira est terminée.

7. Pour une expérience de test optimale, accédez à l’onglet **Look & Feel** (Apparence) et cochez l’option **SAML Login Button** (Bouton de connexion SAML). Un bouton distinct sera alors présent sur l’écran de connexion JIRA pour tester votre intégration SAML Azure AD de bout en bout. Vous pouvez également laisser ce bouton activé et configurer son emplacement, sa couleur et sa traduction pour le mode de production.

    ![Apparence](media/easysso-for-jira-tutorial/jira-admin-5.png)

    > [!NOTE]
    > En cas de problème, contactez l’[équipe de support EasySSO](mailto:support@techtime.co.nz).

### <a name="create-easysso-for-jira-test-user"></a>Créer un utilisateur de test EasySSO for Jira

Dans cette section, un utilisateur appelé Britta Simon est créé dans Jira. EasySSO pour Jira prend en charge le provisionnement d’utilisateurs juste-à-temps, option **désactivée** par défaut. Pour activer le provisionnement d’utilisateurs, vous devez activer explicitement l’option **Create user on successful login** (Créer un utilisateur lors d’une connexion réussie) dans la section General de la configuration du plug-in EasySSO. S’il n’existe pas encore d’utilisateur dans Jira, il en est créé un après l’authentification.

Toutefois, si vous ne souhaitez pas activer le provisionnement automatique d’utilisateurs lors de la première connexion, les utilisateurs doivent exister dans les répertoires utilisateurs back-end utilisés par l’instance de Jira, comme LDAP ou Atlassian Crowd.

![Approvisionnement d'utilisateurs](media/easysso-for-jira-tutorial/jira-admin-6.png)

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

### <a name="idp-initiated-workflow"></a>Workflow lancé par le fournisseur d’identité

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette EasySSO pour Jira dans le volet d’accès, vous devez être automatiquement connecté à l’instance Jira pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

### <a name="sp-initiated-workflow"></a>Workflow lancé par le fournisseur de services

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du bouton **Connexion SAML** de Jira.

![Connexion SAML des utilisateurs](media/easysso-for-jira-tutorial/jira-admin-7.png)

Ce scénario suppose que vous avez activé le **bouton de connexion SAML** sous l’onglet **Apparence** dans votre page de configuration EasySSO pour Jira (voir ci-dessus). Ouvrez votre URL de connexion Jira en mode incognito du navigateur pour éviter toute interférence avec vos sessions existantes. Cliquez sur le bouton **Connexion SAML** . Vous êtes alors redirigé vers le flux d’authentification des utilisateurs Azure AD. Une fois l’opération terminée, vous serez redirigé vers votre instance Jira en tant qu’utilisateur authentifié par le biais de SAML.

Il est possible que vous rencontriez l’écran suivant après avoir été redirigé à partir d’Azure AD

![Écran d’échec d’EasySSO](media/easysso-for-jira-tutorial/jira-admin-8.png)

Dans ce cas, vous devez suivre les [instructions fournies dans cette page]( https://techtime.co.nz/display/TECHTIME/EasySSO+How+to+get+the+logs#EasySSOHowtogetthelogs-RETRIEVINGTHELOGS) pour accéder au fichier **atlassian-jira.log** . Les détails de l’erreur sont disponibles en fonction de l’ID de référence figurant dans la page d’erreur EasySSO.

En cas de problème pour synthétiser les messages du journal, contactez l’[équipe de support technique d’EasySSO](mailto:support@techtime.co.nz).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer EasySSO for Jira avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger EasySSO for Jira avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)