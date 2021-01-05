---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à FortiWeb Web Application Firewall | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et FortiWeb Web Application Firewall.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/24/2020
ms.author: jeedes
ms.openlocfilehash: b2da377ac6498940ed2fc260ad89494f13803ba5
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96523217"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-fortiweb-web-application-firewall"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à FortiWeb Web Application Firewall

Dans ce tutoriel, vous allez découvrir comment intégrer FortiWeb Web Application Firewall à Azure Active Directory (Azure AD). Quand vous intégrez FortiWeb Web Application Firewall à Azure AD, vous pouvez :

* Contrôler qui dans Azure AD a accès à FortiWeb Web Application Firewall.
* Permettre à vos utilisateurs de se connecter automatiquement à FortiWeb Web Application Firewall avec leur compte Azure AD.
* Gérer vos comptes à un emplacement central : le Portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez disposer de ce qui suit :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement FortiWeb Web Application Firewall pour lequel l’authentification unique est activée.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test.

* FortiWeb Web Application Firewall prend en charge l’authentification unique initiée par le **fournisseur de services**.

## <a name="adding-fortiweb-web-application-firewall-from-the-gallery"></a>Ajout de FortiWeb Web Application Firewall à partir de la galerie

Pour configurer l’intégration de FortiWeb Web Application Firewall à Azure AD, vous devez ajouter FortiWeb Web Application Firewall à votre liste d’applications SaaS managées à partir de la galerie.

1. Connectez-vous au portail Azure avec un compte professionnel ou scolaire ou avec un compte personnel Microsoft.
1. Dans le panneau de navigation gauche, sélectionnez le service **Azure Active Directory**.
1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, tapez **FortiWeb Web Application Firewall** dans la zone de recherche.
1. Sélectionnez **FortiWeb Web Application Firewall** dans le volet de résultats, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.


## <a name="configure-and-test-azure-ad-sso-for-fortiweb-web-application-firewall"></a>Configurer et tester l’authentification unique Azure AD pour FortiWeb Web Application Firewall

Configurez et testez l’authentification unique Azure AD avec FortiWeb Web Application Firewall pour un utilisateur de test appelé **B.Simon**. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur associé dans FortiWeb Web Application Firewall.

Pour configurer et tester l’authentification unique Azure AD avec FortiWeb Web Application Firewall, effectuez les étapes suivantes :

1. **[Configurer l’authentification unique Azure AD](#configure-azure-ad-sso)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
    1. **[Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec B. Simon.
    1. **[Affecter l’utilisateur de test Azure AD](#assign-the-azure-ad-test-user)** pour permettre à B. Simon d’utiliser l’authentification unique Azure AD.
1. **[Configurer l’authentification unique FortiWeb Web Application Firewall](#configure-fortiweb-web-application-firewall-sso)** pour configurer les paramètres de l’authentification unique côté application.
    1. **[Créer un utilisateur de test FortiWeb Web Application Firewall](#create-fortiweb-web-application-firewall-test-user)** pour avoir un équivalent de B.Simon dans FortiWeb Web Application Firewall lié à la représentation Azure AD de l’utilisateur.
1. **[Tester l’authentification unique](#test-sso)** pour vérifier si la configuration fonctionne.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le portail Azure, accédez à la page d’intégration de l’application **FortiWeb Web Application Firewall**, recherchez la section **Gérer** et sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, cliquez sur l’icône de modification/stylet de **Configuration SAML de base** pour modifier les paramètres.

   ![Modifier la configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez les valeurs pour les champs suivants :

   a. Dans la zone de texte **Identificateur (ID d’entité)** , saisissez une URL au format suivant : `https://www.<CUSTOMER_DOMAIN>.com`

    b. Dans la zone de texte **URL de réponse**, tapez une URL au format suivant : `https://www.<CUSTOMER_DOMAIN>.com/<FORTIWEB_NAME>/saml.sso/SAML2/POST`

    c. Dans la zone de texte **URL de connexion**, saisissez une URL au format suivant : `https://www.<CUSTOMER_DOMAIN>.com`

    d. Dans la zone de texte **URL de déconnexion**, tapez une URL au format suivant : `https://www.<CUSTOMER_DOMAIN>.info/<FORTIWEB_NAME>/saml.sso/SLO/POST`
 
    > [!NOTE]
    > `<FORTIWEB_NAME>` est l’identificateur de nom qui sera utilisé par la suite quand la configuration sera communiquée à FortiWeb.
    > Pour obtenir les valeurs d’URL réelles, contactez l’[équipe de support technique FortiWeb Web Application Firewall](mailto:support@fortinet.com). Vous pouvez également consulter les modèles figurant à la section **Configuration SAML de base** dans le portail Azure.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **XML de métadonnées de fédération** et sélectionnez **Télécharger** pour télécharger le certificat et l’enregistrer sur votre ordinateur.

    ![Lien Téléchargement de certificat](common/metadataxml.png)


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

Dans cette section, vous allez permettre à B.Simon d’utiliser l’authentification unique Azure en lui accordant l’accès à FortiWeb Web Application Firewall.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **FortiWeb Web Application Firewall**.
1. Dans la page de vue d’ensemble de l’application, recherchez la section **Gérer** et sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.
1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B. Simon** dans la liste Utilisateurs, puis cliquez sur le bouton **Sélectionner** au bas de l’écran.
1. Si vous attendez qu’un rôle soit attribué aux utilisateurs, vous pouvez le sélectionner dans la liste déroulante **Sélectionner un rôle** . Si aucun rôle n’a été configuré pour cette application, vous voyez le rôle « Accès par défaut » sélectionné.
1. Dans la boîte de dialogue **Ajouter une attribution**, cliquez sur le bouton **Attribuer**.

## <a name="configure-fortiweb-web-application-firewall-sso"></a>Configurer l’authentification unique pour FortiWeb Web Application Firewall

1.  Accédez à `https://<address>:8443` où `<address>` est le nom de domaine complet ou l’adresse IP publique affecté(e) à la machine virtuelle FortiWeb.

2.  Connectez-vous à l’aide des informations d’identification d’administrateur fournies pendant le déploiement de la machine virtuelle FortiWeb.

1. Effectuez les étapes ci-après dans la page suivante.

    ![Page SAML Server (Serveur SAML)](./media/fortiweb-web-application-firewall-tutorial/configure-sso.png)

    a.  Dans le menu de gauche, cliquez sur **User** (Utilisateur).

    b.  En dessous de User, cliquez sur **Remote Server** (Serveur distant).

    c.  Cliquez sur **SAML Server** (Serveur SAML).

    d.  Cliquez sur **Créer nouveau**.

    e.  Dans le champ **Name** (Name), indiquez la valeur de `<fwName>` utilisée dans la section Configurer Azure AD.

    f.  Dans la zone de texte **ID d’entité**, collez la valeur **Identificateur Azure AD** que vous avez copiée dans le portail Azure.

    g. À côté de **Metadata** (Métadonnées), cliquez sur **Choose File** (Choisir un fichier), puis sélectionnez le fichier **XML de métadonnées de fédération** que vous avez téléchargé sur le portail Azure.

    h.  Cliquez sur **OK**.

### <a name="create-a-site-publishing-rule"></a>Créer une règle de publication de site

1.  Accédez à `https://<address>:8443` où `<address>` est le nom de domaine complet ou l’adresse IP publique affecté(e) à la machine virtuelle FortiWeb.

1.  Connectez-vous à l’aide des informations d’identification d’administrateur fournies pendant le déploiement de la machine virtuelle FortiWeb.
1. Effectuez les étapes ci-après dans la page suivante.

    ![Règle de publication de site](./media/fortiweb-web-application-firewall-tutorial/site-publish-rule.png)

    a.  Dans le menu de gauche, cliquez sur **Application Delivery** (Livraison d’application).
    
    b.  En dessous de **Application Delivery**, cliquez sur **Site Publish** (Publication de site).
    
    c.  En dessous de **Site Publish**, cliquez sur **Site Publish**.
    
    d.  Cliquez sur **Site Publish Rule** (Règle de publication de site).
    
    e.  Cliquez sur **Créer nouveau**.
    
    f.  Attribuez un nom à la règle de publication de site.
    
    g.  En regard de **Published Site Type** (Type de site publié), cliquez sur **Regular Expression** (Expression régulière).
    
    i.  À côté de **Published Site** (Site publié), indiquez une chaîne qui correspond à l’en-tête d’hôte du site web que vous publiez.
    
    j.  À côté de **Path** (Chemin), ajoutez un caractère /.
    
    k.  À côté de **Client Authentication Method** (Méthode d’authentification du client), sélectionnez **SAML Authentication** (Authentification SAML).
    
    l.  Dans la liste déroulante **SAML Server** (Serveur SAML), sélectionnez le serveur SAML que vous avez créé précédemment.
    
    m.  Cliquez sur **OK**.


### <a name="create-a-site-publishing-policy"></a>Créer une stratégie de publication de site

1.  Accédez à `https://<address>:8443` où `<address>` est le nom de domaine complet ou l’adresse IP publique affecté(e) à la machine virtuelle FortiWeb.

2.  Connectez-vous à l’aide des informations d’identification d’administrateur fournies pendant le déploiement de la machine virtuelle FortiWeb.

1. Effectuez les étapes ci-après dans la page suivante.

    ![Stratégie de publication de site](./media/fortiweb-web-application-firewall-tutorial/site-publish-policy.png)

    a.  Dans le menu de gauche, cliquez sur **Application Delivery** (Livraison d’application).

    b.  En dessous de **Application Delivery**, cliquez sur **Site Publish** (Publication de site).

    c.  En dessous de **Site Publish**, cliquez sur **Site Publish**.

    d.  Cliquez sur **Site Publish Policy** (Stratégie de publication de site).

    e.  Cliquez sur **Créer nouveau**.

    f.  Attribuez un nom à la stratégie de publication de site.

    g.  Cliquez sur **OK**.

    h.  Cliquez sur **Créer nouveau**.

    i.  Dans la liste déroulante **Rule** (Règle), sélectionnez la règle de publication de site que vous avez créée précédemment.

    j.  Cliquez sur **OK**.

### <a name="create-and-assign-a-web-protection-profile"></a>Créer et affecter un profil de protection web

1.  Accédez à `https://<address>:8443` où `<address>` est le nom de domaine complet ou l’adresse IP publique affecté(e) à la machine virtuelle FortiWeb.

2.  Connectez-vous à l’aide des informations d’identification d’administrateur fournies pendant le déploiement de la machine virtuelle FortiWeb.
3.  Dans le menu de gauche, cliquez sur **Policy** (Stratégie).
4.  En dessous de **Policy**, cliquez sur **Web Protection Profile** (Profil de protection web).
5.  Cliquez sur **Inline Standard Protection** (Protection standard en ligne), puis cliquez sur **Cloner** (Cloner).
6.  Attribuez un nom au nouveau profil de protection web, puis cliquez sur **OK**.
7.  Sélectionnez le nouveau profil de protection web, puis cliquez sur **Edit** (Modifier).
8.  À côté de **Site Publish** (Publication de site), sélectionnez la stratégie de publication de site que vous avez créée précédemment.
9.  Cliquez sur **OK**.
 
    ![publication de site](./media/fortiweb-web-application-firewall-tutorial/web-protection.png)

10. Dans le menu de gauche, cliquez sur **Policy** (Stratégie).
11. En dessous de **Policy**, cliquez sur **Server Policy** (Stratégie serveur).
12. Sélectionnez la stratégie serveur utilisée pour publier le site web pour lequel vous souhaitez utiliser Azure Active Directory pour l’authentification.
13. Cliquez sur **Modifier**.
14. Dans la liste déroulante **Web Protection Profile** (Profil de protection web), sélectionnez le profil de protection web que vous venez de créer.
15. Cliquez sur **OK**.
16. Essayez d’accéder à l’URL externe à laquelle FortiWeb publie le site web. Vous êtes alors redirigé vers Azure Active Directory pour l’authentification.


### <a name="create-fortiweb-web-application-firewall-test-user"></a>Créer un utilisateur de test pour FortiWeb Web Application Firewall

Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans FortiWeb Web Application Firewall. Rapprochez-vous de l’[équipe de support technique FortiWeb Web Application Firewall](mailto:support@fortinet.com) pour ajouter des utilisateurs dans la plateforme FortiWeb Web Application Firewall. Les utilisateurs doivent être créés et activés avant que vous utilisiez l’authentification unique.

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Dans cette section, vous allez tester votre configuration de l’authentification unique Azure AD avec les options suivantes. 

* Cliquez sur **Tester cette application** dans le portail Azure. Vous êtes alors redirigé vers l’URL de connexion FortiWeb Web Application Firewall où vous pouvez initier le flux de connexion. 

* Accédez directement à l’URL de connexion FortiWeb Web Application Firewall pour y initier le flux de connexion.

* Vous pouvez utiliser Mes applications de Microsoft. Quand vous cliquez sur la vignette FortiWeb Web Application Firewall dans Mes applications, vous êtes redirigé vers l’URL de connexion FortiWeb Web Application Firewall. Pour plus d’informations sur Mes applications, consultez [Présentation de Mes applications](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).


## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez configuré FortiWeb Web Application Firewall, vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrez comment appliquer un contrôle de session avec Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).


