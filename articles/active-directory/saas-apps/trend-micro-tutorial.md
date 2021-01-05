---
title: 'Tutoriel : Intégration de l’authentification unique Azure Active Directory à Trend Micro Web Security (TMWS)'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Trend Micro Web Security (TMWS).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 04/21/2020
ms.author: jeedes
ms.openlocfilehash: b76c41787d7a35fb3024fa18c0122bc966243bbc
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008393"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-trend-micro-web-security-tmws"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à Trend Micro Web Security (TMWS)

Dans ce tutoriel, vous allez découvrir comment intégrer Trend Micro Web Security (TMWS) à Azure Active Directory (Azure AD). Quand vous intégrez TMWS à Azure AD, vous pouvez :

* Contrôler, dans Azure AD, qui a accès à TMWS.
* Permettre aux utilisateurs de se connecter automatiquement à TMWS avec leur compte Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

Pour en savoir plus sur l’intégration d’applications SaaS à Azure AD, consultez l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un abonnement TMWS activé pour l’authentification unique.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification SSO Azure AD dans un environnement de test.

* TMWS prend en charge l’authentification unique lancée par le fournisseur de services.
* Après avoir configuré TMWS, vous pouvez appliquer le contrôle de session qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. Pour savoir comment appliquer un contrôle de session avec Microsoft Cloud App Security, consultez [Déployer le contrôle d’application par accès conditionnel pour les applications](/cloud-app-security/proxy-deployment-any-app).

## <a name="add-tmws-from-the-gallery"></a>Ajouter TMWS à partir de la galerie

Pour configurer l’intégration de TMWS à Azure AD, vous devez ajouter TMWS, disponible dans la galerie, à votre liste d’applications SaaS managées.

1. Connectez-vous au [Portail Azure](https://portal.azure.com) avec un compte professionnel ou scolaire, ou avec un compte Microsoft personnel.
1. Dans le volet de gauche, sélectionnez le service **Azure Active Directory**.
1. Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application**.
1. Dans la section **Ajouter à partir de la galerie**, entrez **Trend Micro Web Security (TMWS)** dans la zone de recherche.
1. Sélectionnez **Trend Micro Web Security (TMWS)** dans les résultats de la recherche, puis ajoutez l’application. Patientez quelques secondes pendant que l’application est ajoutée à votre locataire.

## <a name="configure-and-test-azure-ad-sso-for-tmws"></a>Configurer et tester l’authentification unique Azure AD pour TMWS

Vous allez configurer et tester l’authentification unique Azure AD avec TMWS pour un utilisateur de test appelé B.Simon. Pour que l’authentification unique fonctionne, vous devez établir un lien entre un utilisateur Azure AD et l’utilisateur TMWS associé.

Pour configurer et tester l’authentification unique Azure AD avec TMWS, effectuez ces étapes :

1. [Configurer l’authentification unique Azure AD](#configure-azure-ad-sso) afin d’activer la fonctionnalité pour vos utilisateurs.
    1. [Créer un utilisateur de test Azure AD](#create-an-azure-ad-test-user) pour tester l’authentification unique Azure AD.
    1. [Accorder à l’utilisateur de test Azure AD](#grant-the-azure-ad-test-user-access-to-tmws) l’accès à TMWS.
    1. [Configurer les paramètres de synchronisation des utilisateurs et des groupes dans Azure AD](#configure-user-and-group-synchronization-settings-in-azure-ad).
1. [Configurer l’authentification unique pour TMWS](#configure-tmws-sso) côté application.
1. [Tester l’authentification unique](#test-sso) pour vérifier la configuration.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Effectuez les étapes suivantes pour activer l’authentification unique Azure AD dans le Portail Azure.

1. Dans le [Portail Azure](https://portal.azure.com/), dans la page d’intégration de l’application **Trend Micro Web Security(TMWS)** , dans la section **Gérer**, sélectionnez **Authentification unique**.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**.
1. Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez le bouton représentant un crayon et correspondant à **Configuration SAML de base** pour modifier les paramètres :

   ![Modifier les paramètres Configuration SAML de base](common/edit-urls.png)

1. Dans la section **Configuration SAML de base**, entrez des valeurs dans les zones suivantes :

    a. Dans la zone de texte **Identificateur (ID d’entité)** , entrez une URL au format suivant :

    `https://auth.iws-hybrid.trendmicro.com/([0-9a-f]{16})`

    b. Dans la zone de texte **URL de réponse**, entrez cette URL :

    `https://auth.iws-hybrid.trendmicro.com/simplesaml/module.php/saml/sp/saml2-acs.php/ics-sp`

    > [!NOTE]
    > La valeur de l’identificateur de l’étape précédente n’est pas celle que vous devez entrer. Vous devez utiliser l’identificateur réel. Cette valeur est indiquée dans la section **Paramètres du fournisseur de services du portail d’administration Azure**, dans la page **Méthode d’authentification** pour Azure AD en accédant à **Administration > Services d’annuaire**.

1. Comme TMWS s’attend à recevoir les assertions SAML dans un format spécifique, vous devez ajouter des mappages d’attributs personnalisés à votre configuration des attributs de jeton SAML. Cette capture d’écran montre les attributs par défaut :

    ![Attributs par défaut](common/default-attributes.png)

1. Outre les attributs illustrés dans la capture d’écran précédente, TMWS s’attend à ce que deux attributs supplémentaires soient repassés dans la réponse SAML. Ces attributs sont présentés dans le tableau suivant. Les attributs sont préremplis, mais vous pouvez les changer en fonction de vos besoins.
    
    | Nom | Attribut source|
    | --------------- | --------- |
    | sAMAccountName | user.onpremisessamaccountname |
    | uPN | user.userprincipalname |

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, recherchez **Certificat (Base64)** . Sélectionnez le lien **Télécharger** à côté de ce nom de certificat pour télécharger le certificat et l’enregistrer sur votre ordinateur :

    ![Lien de téléchargement du certificat](common/certificatebase64.png)

1. Dans la section **Configurer Trend Micro Web Security (TMWS)** , copiez l’URL ou les URL appropriées en fonction de vos besoins :

    ![Copier les URL de configuration](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Créer un utilisateur de test Azure AD

Dans cette section, vous allez créer un utilisateur de test appelé B.Simon dans le portail Azure.

1. Dans le volet gauche du portail Azure, sélectionnez **Azure Active Directory**. Sélectionnez **Utilisateurs**, puis **Tous les utilisateurs**.
1. Sélectionnez **Nouvel utilisateur** dans la partie supérieure de l’écran.
1. Dans les propriétés **Utilisateur**, effectuez les étapes suivantes :
   1. Dans la zone **Nom**, entrez `B.Simon`.  
   1. Dans la zone **Nom d’utilisateur**, entrez **_nom_utilisateur_@* domaine_entreprise *.* extension***. Par exemple : `B.Simon@contoso.com`.
   1. Sélectionnez **Afficher le mot de passe**, puis notez la valeur affichée dans la zone **Mot de passe**.
   1. Sélectionnez **Create** (Créer).

### <a name="grant-the-azure-ad-test-user-access-to-tmws"></a>Accorder à l’utilisateur de test Azure AD l’accès à TMWS

Dans cette section, vous allez autoriser B.Simon à utiliser l’authentification unique Azure en lui accordant l’accès à TMWS.

1. Dans le portail Azure, sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.
1. Dans la liste des applications, sélectionnez **Trend Micro Web Security (TMWS)** .
1. Dans la page de vue d’ensemble de l’application, dans la section **Gérer**, sélectionnez **Utilisateurs et groupes** :

   ![Sélectionner Utilisateurs et groupes](common/users-groups-blade.png)

1. Sélectionnez **Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans la boîte de dialogue **Ajouter une attribution**.

    ![Sélectionnez Ajouter un utilisateur](common/add-assign-user.png)

1. Dans la boîte de dialogue **Utilisateurs et groupes**, sélectionnez **B.Simon** dans la liste **Utilisateurs**, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Si vous attendez une valeur de rôle dans l’assertion SAML, dans la boîte de dialogue **Sélectionner un rôle**, sélectionnez le rôle approprié pour l’utilisateur dans la liste, puis cliquez sur le bouton **Sélectionner** en bas de l’écran.
1. Dans la boîte de dialogue **Ajouter une attribution**, sélectionnez **Affecter**.

### <a name="configure-user-and-group-synchronization-settings-in-azure-ad"></a>Configurer les paramètres de synchronisation des utilisateurs et des groupes dans Azure AD

1. Sélectionnez **Azure Active Directory** dans le volet de gauche.

1. Sous **Gérer**, sélectionnez **Inscriptions d’applications**, puis sélectionnez la nouvelle application d’entreprise sous **Toutes les applications**.

1. Sous **Gérer**, sélectionnez **Certificats et secrets**.

1. Dans la zone **Secrets client**, sélectionnez **Nouveau secret client**.

1. Dans la fenêtre **Ajouter un secret client**, ajoutez éventuellement une description, sélectionnez une période d’expiration du secret client, puis sélectionnez **Ajouter**. Le nouveau secret client apparaît dans la zone **Secrets client**.

1. Notez la valeur du secret client, car vous devrez l’entrer plus tard dans TMWS.

1. Sous **Gérer**, sélectionnez **Autorisations de l’API**. 

1. Dans la fenêtre **Autorisations de l’API**, sélectionnez **Ajouter une autorisation**.

1. Sous l’onglet **API Microsoft** de la fenêtre **Demander des autorisations d’API**, sélectionnez **Microsoft Graph**, puis sélectionnez **Autorisations pour les applications**.

1. Recherchez et ajoutez ces autorisations : 

    * Group.Read.All
    * User.Read.All

1. Sélectionnez **Ajouter des autorisations**. Un message s’affiche pour confirmer que vos paramètres ont été enregistrés. Les nouvelles autorisations s’affichent dans la fenêtre **Autorisations de l’API**.

1. Dans la zone **Donner son consentement**, sélectionnez **Accorder un consentement d’administrateur pour *votre compte d’administrateur* (Répertoire par défaut)** , puis sélectionnez **Oui**. Un message s’affiche pour confirmer que le consentement d’administrateur a été accordé pour les autorisations demandées.

1. Sélectionnez **Vue d’ensemble**. 

1. Notez les valeurs **ID d’application (client)** et **ID de l’annuaire (locataire)** qui sont indiquées dans le panneau à droite. Plus tard, vous entrerez ces valeurs dans TMWS. Vous pouvez aussi sélectionner **Noms de domaine personnalisés** sous **Azure Active Directory > Gérer** et noter le nom de domaine affiché dans le volet de droite.

## <a name="configure-tmws-sso"></a>Configurer l’authentification unique côté TMWS

Effectuez ces étapes pour configurer l’authentification unique pour TMWS côté application.

1. Connectez-vous à la console de gestion TMWS et accédez à **Administration** > **USERS & AUTHENTICATION** > **Directory Services**.

1. Sélectionnez **here** (ici) dans la partie supérieure de l’écran.

1. Dans la page **Authentication Method** (Méthode d’authentification), sélectionnez **Azure AD**.

1. Sélectionnez **On** ou **Off** pour définir si les utilisateurs Azure AD dans votre organisation sont autorisés à accéder aux sites web par le biais de TMWS quand leurs données ne sont pas synchronisées avec TMWS.

    > [!NOTE]
    > Les utilisateurs qui ne sont pas synchronisés à partir d’Azure AD peuvent être authentifiés uniquement par le biais de passerelles TMWS connues ou du port dédié à votre organisation.

1. Dans la section **Identity Provider Settings** (Paramètres du fournisseur d’identité), effectuez ces étapes :

    a. Dans la zone **Service URL** (URL du service), entrez la valeur **URL de connexion** que vous avez copiée du Portail Azure.

    b. Dans la zone **Logon name attribute** (Attribut du nom de connexion), entrez le **Nom de la revendication utilisateur** avec l’attribut source **user.onpremisessamaccountname** copié à partir du Portail Azure.

    c. Dans la zone **Public SSL certificate** (Certificat SSL public), utilisez le **Certificat (Base64)** téléchargé à partir du Portail Azure.

1. Dans la section **Synchronization Settings** (Paramètres de synchronisation), effectuez ces étapes :

    a. Dans la zone **Tenant** (Locataire), entrez l’**ID d’annuaire (locataire)** ou le **Nom de domaine personnalisé** copié du Portail Azure.

    b. Dans la zone **Application ID** (ID application), entrez l’**ID d’application (client)** copié du Portail Azure.

    c. Dans la zone **Client secret** (Secret client), entrez le **Secret client** copié du Portail Azure.

    d. Sélectionnez **Synchronization schedule** (Planification de la synchronisation) pour effectuer une synchronisation avec Azure AD manuelle ou selon une planification. Si vous sélectionnez **Manually** (Manuelle), chaque fois que des modifications sont apportées aux informations des utilisateurs Active Directory, n’oubliez pas de revenir dans la page **Directory Services** et d’effectuer une synchronisation manuelle pour que les informations restent à jour dans TMWS.

    e. Sélectionnez **Test Connection** (Tester la connexion) pour vérifier que la connexion au service Azure AD fonctionne.
    
    f. Sélectionnez **Enregistrer**.
 
 > [!NOTE]
 > Pour plus d’informations sur la configuration de TMWS avec Azure AD, consultez [Configuration des paramètres Azure AD sur TMWS](https://docs.trendmicro.com/en-us/enterprise/trend-micro-web-security-online-help/administration_001/directory-services/azure-active-directo/configuring-azure-ad.aspx).

## <a name="test-sso"></a>Tester l’authentification unique (SSO) 

Après avoir configuré le service Azure AD et spécifié Azure AD comme méthode d’authentification utilisateur, vous pouvez vous connecter au serveur proxy TMWS pour vérifier votre configuration. Dès lors que votre compte a été vérifié par l’authentification Azure AD, vous pouvez naviguer sur Internet.

> [!NOTE]
> TMWS ne prend pas en charge le test de l’authentification unique à partir du Portail Azure AD, sous **Vue d’ensemble** > **Authentification unique** > **Configurer l’authentification unique avec SAML** > **Tester** votre nouvelle application d’entreprise.

1. Supprimez tous les cookies du navigateur, puis redémarrez ce dernier. 

1. Faites pointer votre navigateur vers le serveur proxy TMWS. Pour plus d’informations, consultez [Traffic Forwarding Using PAC Files](https://docs.trendmicro.com/en-us/enterprise/trend-micro-web-security-online-help/administration_001/pac-files/traffic-forwarding-u.aspx#GUID-A4A83827-7A29-4596-B866-01ACCEDCC36B).

1. Accédez à un site web sur Internet. TMWS vous dirige vers le portail captif TMWS.

1. Spécifiez un compte Active Directory (format : *domain*\\*sAMAccountName* ou *sAMAccountName*@*domain*), une adresse e-mail ou un nom d’utilisateur principal (UPN), puis sélectionnez **Log On** (Ouvrir une session). TMWS vous dirige vers la fenêtre de connexion à Azure AD.

1. Dans la fenêtre de connexion à Azure AD, entrez les informations d’identification de votre compte Azure AD. Vous devez maintenant être connecté à TMWS.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Tutoriels sur l’intégration d’applications SaaS avec Azure Active Directory](./tutorial-list.md)

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)

- [Essayer Trend Micro Web Security avec Azure AD](https://aad.portal.azure.com/)

- [Qu’est-ce que le contrôle de session dans Microsoft Cloud App Security ?](/cloud-app-security/proxy-intro-aad)

- [Guide pratique pour protéger Trend Micro Web Security avec une visibilité et des contrôles avancés](/cloud-app-security/proxy-intro-aad)