---
title: 'Didacticiel : Intégration d’Azure Active Directory avec FreshDesk | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et FreshDesk.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/24/2020
ms.author: jeedes
ms.openlocfilehash: f120554574f8dc7d0b97744af6dad447b404005b
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92451527"
---
# <a name="tutorial-azure-active-directory-integration-with-freshdesk"></a>Didacticiel : Intégration d’Azure Active Directory à FreshDesk

Dans ce didacticiel, vous allez apprendre à intégrer FreshDesk à Azure Active Directory (Azure AD).
L’intégration de FreshDesk dans Azure AD vous offre les avantages suivants :

* Azure AD vous permet de contrôler qui a accès à FreshDesk.
* Vous pouvez permettre aux utilisateurs de se connecter automatiquement à FreshDesk (par le biais de l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes dans un emplacement central : le portail Azure

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md).
Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer l’intégration d’Azure AD à FreshDesk, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).
* Abonnement FreshDesk pour lequel l’authentification unique est activée

## <a name="scenario-description"></a>Description du scénario

Dans ce didacticiel, vous configurez et testez l’authentification unique Azure AD dans un environnement de test.

* FreshDesk prend en charge l’authentification unique initiée par le **fournisseur de services**
* Une fois que vous avez configuré FreshDesk, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-freshdesk-from-the-gallery"></a>Ajout de FreshDesk depuis la galerie

Pour configurer l’intégration de FreshDesk dans Azure AD, vous devez ajouter FreshDesk depuis la galerie dans votre liste d’applications SaaS gérées.

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory** .
1. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications** .
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** .
1. Dans la section **Ajouter à partir de la galerie** , tapez **FreshDesk** dans la zone de recherche.
1. Sélectionnez **FreshDesk** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-freshdesk"></a>Configurer et tester l’authentification unique Azure AD pour FreshDesk

Configurez et testez l’authentification unique Azure AD avec FreshDesk pour un utilisateur de test appelé **B.Simon** . Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur FreshDesk associé.

Pour configurer et tester l’authentification unique Azure AD avec FreshDesk, suivez les indications des sections ci-après :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique FreshDesk](#configure-freshdesk-single-sign-on)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test FreshDesk](#create-freshdesk-test-user)** pour avoir un équivalent de Britta Simon dans FreshDesk, lié à la représentation Azure AD associée.
1. **[Tester l’authentification unique](#test-single-sign-on)** : pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-single-sign-on"></a>Configurer l’authentification unique Azure AD

1. Sur le [portail Azure](https://portal.azure.com/), accédez à la page d’intégration de l’application **FreshDesk** , recherchez la section **Gérer** et sélectionnez **Authentification unique** .
1. Dans la page **Sélectionner une méthode d’authentification unique** , sélectionnez **SAML** .
1. Dans la page **Configurer l’authentification unique avec SAML** , cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base** , entrez les valeurs pour les champs suivants :

    a. Dans la zone de texte **URL de connexion** , tapez une URL en utilisant le modèle `https://<tenant-name>.freshdesk.com` ou toute autre valeur suggérée par FreshDesk.

    b. Dans la zone de texte **Identificateur (ID d’entité)** , tapez une URL en utilisant le modèle `https://<tenant-name>.freshdesk.com`, ou toute autre valeur suggérée par FreshDesk.
     
    c. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://<tenant-name>.freshdesk.com/login/saml`
    
    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels. Contactez [l’équipe de support FreshDesk](https://freshdesk.com/helpdesk-software?utm_source=Google-AdWords&utm_medium=Search-IND-Brand&utm_campaign=Search-IND-Brand&utm_term=freshdesk&device=c&gclid=COSH2_LH7NICFVUDvAodBPgBZg) pour obtenir ces valeurs. Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. L’application FreshDesk s’attend à recevoir les assertions SAML dans un format spécifique, ce qui vous oblige à ajouter des mappages d’attributs personnalisés à la configuration des attributs de votre jeton SAML. La capture d’écran suivante montre la liste des attributs par défaut. **Unique User Identifier**  (Identificateur d’utilisateur unique) est mappé à **user.userPrincipalName** , mais FreshDesk s’attend à ce que cette revendication soit mappée à **user. mail** . Vous devez donc modifier le mappage d’attributs en cliquant sur l’icône de modification et changer le mappage d’attributs.

    ![image](common/edit-attribute.png)

1. Dans la page **Configurer l’authentification unique avec SAML** , dans la section **Certificat de signature SAML** , cliquez sur **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/certificatebase64.png)

1. Dans la section **Configurer FreshDesk** , copiez la ou les URL appropriées en fonction de vos besoins.

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

    a. URL de connexion

    b. Identificateur Azure AD

    c. URL de déconnexion

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD 

L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory** , sélectionnez **Utilisateurs** , puis sélectionnez **Tous les utilisateurs** .

    ![Liens « Utilisateurs et groupes » et « Tous les utilisateurs »](common/users.png)

2. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.

    ![Bouton Nouvel utilisateur](common/new-user.png)

3. Dans les propriétés de l’utilisateur, effectuez les étapes suivantes.

    ![Boîte de dialogue Utilisateur](common/user-properties.png)

    a. Dans le champ **Nom** , entrez **BrittaSimon** .
  
    b. Dans le champ **Nom d’utilisateur** , tapez **brittasimon\@domainedevotreentreprise.extension** .  
    Par exemple : BrittaSimon@contoso.com

    c. Cochez la case **Afficher le mot de passe** , puis notez la valeur affichée dans le champ Mot de passe.

    d. Cliquez sur **Créer** .


### <a name="assign-the-azure-ad-test-user"></a>Affecter l’utilisateur de test Azure AD

Dans cette section, vous autorisez Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à FreshDesk.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise** , **Toutes les applications** , puis sélectionnez **FreshDesk** .

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, tapez et sélectionnez **FreshDesk** .

    ![Lien FreshDesk dans la liste des applications](common/all-applications.png)

3. Dans le menu de gauche, sélectionnez **Utilisateurs et groupes** .

    ![Lien « Utilisateurs et groupes »](common/users-groups-blade.png)

4. Cliquez sur le bouton **Ajouter un utilisateur** , puis sélectionnez **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution** .

    ![Volet Ajouter une attribution](common/add-assign-user.png)

5. Dans la boîte de dialogue **Utilisateurs et groupes** , sélectionnez **Britta Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

6. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle** , sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.

7. Dans la boîte de dialogue **Ajouter une attribution** , cliquez sur le bouton **Attribuer** .

## <a name="configure-freshdesk-single-sign-on"></a>Configurer l’authentification unique FreshDesk

1. Dans une autre fenêtre de navigateur web, connectez-vous au site de votre entreprise Freshdesk en tant qu’administrateur.

2. Sélectionnez l’icône **Settings** (Paramètres) puis, dans la section **Security** (Sécurité), effectuez les étapes suivantes :

    ![Authentification unique](./media/freshdesk-tutorial/configure-1.png "Authentification unique")
  
    a. Pour **Single Sign On** (Authentification unique), sélectionnez **On** (Activé).

    b. Dans **Login Method** (Méthode de connexion), sélectionnez **SAML SSO** .

    c. Dans la zone de texte **Entity ID provided by the IdP** (ID d’entité fourni par le fournisseur d’identité), collez la valeur **ID d’entité** que vous avez copiée sur le portail Azure.

    d. Dans la zone de texte **URL d’authentification unique SAML** , collez la valeur d’ **URL de connexion** que vous avez copiée à partir du portail Azure.

    e. Dans **Signing Options** (Options de signature), sélectionnez **Only Signed Assertions** (Uniquement les assertions signées) dans la liste déroulante.

    f. Dans la zone de texte **Logout URL** (URL de déconnexion), collez l’ **URL de déconnexion** que vous avez copiée sur le portail Azure.

    g. Dans la zone de texte **Security Certificate** (Certificat de sécurité), collez la valeur de **Certificat (en base64)** que vous avez obtenue précédemment.
  
    h. Cliquez sur **Enregistrer** .

## <a name="create-freshdesk-test-user"></a>Créer un utilisateur de test FreshDesk

Pour permettre aux utilisateurs Azure AD de se connecter à FreshDesk, vous devez les approvisionner dans FreshDesk.  
Dans le cas de FreshDesk, l’approvisionnement est une tâche manuelle.

**Pour approvisionner un compte d’utilisateur, procédez comme suit :**

1. Connectez-vous à votre locataire **Freshdesk** .

1. Dans le menu de gauche, cliquez sur **Admin** puis, sous l’onglet **General Settings** (Paramètres généraux), cliquez sur **Agents** .
  
    ![Agents](./media/freshdesk-tutorial/create-user-1.png "Agents")

1. Cliquez sur **New Agent** .

    ![Nouvel agent](./media/freshdesk-tutorial/create-user-2.png "New Agent")

1. Dans la boîte de dialogue Agent Information (Informations sur l’agent), complétez les champs obligatoires, puis cliquez sur **Create agent** (Créer un agent).

    ![Informations sur l’agent](./media/freshdesk-tutorial/create-user-3.png "Informations sur l’agent")

    >[!NOTE]
    >Le titulaire du compte Azure AD reçoit alors un message électronique qui contient un lien pour confirmer le compte avant qu’il soit activé.
    >
    >[!NOTE]
    >Vous pouvez utiliser tout autre outil ou n’importe quelle API de création de compte d’utilisateur fournis par Freshdesk pour provisionner des comptes d’utilisateurs Azure AD à Freshdesk.

### <a name="test-single-sign-on"></a>Tester l’authentification unique 

Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Quand vous cliquez sur la vignette FreshDesk dans le volet d’accès, vous devez être connecté automatiquement à l’application FreshDesk pour laquelle vous avez configuré l’authentification unique. Pour plus d’informations sur le panneau d’accès, consultez [Présentation du panneau d’accès](../user-help/my-apps-portal-end-user-access.md).

## <a name="additional-resources"></a>Ressources supplémentaires

- [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)