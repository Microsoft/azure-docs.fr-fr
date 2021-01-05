---
title: 'Tutoriel : Intégration d’Azure Active Directory à SharePoint (local) | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et SharePoint (local).
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/10/2020
ms.author: jeedes
ms.openlocfilehash: a693b22c609829f3bf6e76637eac5793d73703e6
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96862307"
---
# <a name="tutorial-azure-active-directory-single-sign-on-integration-with-sharepoint-on-premises"></a>Tutoriel : Intégration de l’authentification unique Azure Active Directory à SharePoint (local)

Dans ce tutoriel, vous allez apprendre à intégrer SharePoint (local) avec Azure Active Directory (Azure AD). Quand vous intégrez SharePoint (local) à Azure AD, vous pouvez :

* Contrôler qui a accès à SharePoint (local) dans Azure AD.
* Permettre à vos utilisateurs de se connecter automatiquement à SharePoint (local) avec leur compte Azure AD.
* Gérer vos comptes dans le portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour configurer l’intégration d’Azure AD avec SharePoint (local), vous avez besoin de ces éléments :

* Un abonnement Azure AD Si vous n’avez pas d’environnement Azure AD, vous pouvez obtenir un [compte gratuit](https://azure.microsoft.com/free/).
* Une batterie de serveurs SharePoint 2013 ou version ultérieure.

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester l’authentification unique Azure AD dans un environnement de test. Les utilisateurs Azure AD peuvent accéder à votre site SharePoint local.

## <a name="create-enterprise-applications-in-the-azure-portal"></a>Créer des applications d’entreprise dans le portail Azure

Pour configurer l’intégration de SharePoint (local) avec Azure AD, vous devez l’ajouter à votre liste d’applications SaaS gérées à partir de la galerie.

Pour ajouter SharePoint (local) à partir de la galerie

1. Dans le portail Azure, dans le volet le plus à gauche, sélectionnez **Azure Active Directory**.

   > [!NOTE]
   > Si l’élément n’est pas disponible, vous pouvez aussi l’ouvrir par le biais du lien **Tous les services** en haut du volet le plus à gauche. Dans la présentation suivante, le lien **Azure Active Directory** se trouve dans la section **Identité**. Vous pouvez également le rechercher à l’aide de la zone de filtre.

1. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

1. Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la boîte de dialogue.

1. Dans la zone de recherche, entrez le filtre **SharePoint (local)** . Dans le volet résultats, sélectionnez **SharePoint (local)** .

    <kbd>![SharePoint (local) dans la liste des résultats](./media/sharepoint-on-premises-tutorial/search-new-app.png)</kbd>

1. Spécifiez un nom pour votre instance SharePoint locale, puis sélectionnez **Ajouter** pour ajouter l’application.

1. Dans la nouvelle application d’entreprise, sélectionnez **Propriétés** et vérifiez la valeur de l’option **Affectation de l’utilisateur obligatoire**.

   <kbd>![Bouton bascule Affectation de l’utilisateur obligatoire](./media/sharepoint-on-premises-tutorial/user-assignment-required.png)</kbd>

   Dans ce scénario, la valeur est définie sur **Non**.

## <a name="configure-and-test-azure-ad"></a>Configurer et tester Azure AD

Dans cette section, vous allez configurer l’authentification unique Azure AD avec SharePoint (local). Pour que l’authentification unique fonctionne, vous établissez un lien entre un utilisateur Azure AD et l’utilisateur SharePoint (local) associé.

Pour configurer et tester l’authentification unique Azure AD avec SharePoint (local), suivez les indications de ces modules :

- [Configurer l’authentification unique Azure AD](#configure-azure-ad-sso) pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
- [Configurer SharePoint (local)](#configure-sharepoint-on-premises) pour configurer les paramètres de l’authentification unique côté application.
- [Créer un utilisateur de test Azure AD dans le portail Azure](#create-an-azure-ad-test-user-in-the-azure-portal) pour créer un utilisateur dans Azure AD pour l’authentification unique.
- [Créer un groupe de sécurité AD dans le portail Azure](#create-an-azure-ad-security-group-in-the-azure-portal) pour créer un groupe de sécurité dans Azure AD pour l’authentification unique.
- [Accorder à un compte Azure AD des autorisations d’accès à SharePoint (local)](#grant-permissions-to-an-azure-ad-account-in-sharepoint-on-premises) pour accorder des autorisations à un utilisateur Azure AD.
- [Accorder à un groupe Azure AD des autorisations d’accès à SharePoint (local)](#grant-permissions-to-an-azure-ad-group-in-sharepoint-on-premises) pour accorder des autorisations à un groupe Azure AD.
- [Accorder à un compte invité l’accès à SharePoint (local) dans le portail Azure](#grant-access-to-a-guest-account-to-sharepoint-on-premises-in-the-azure-portal) pour accorder des autorisations à un compte invité dans Azure AD pour SharePoint (local).
- [Configurer le fournisseur d’identité approuvé pour plusieurs applications web](#configure-the-trusted-identity-provider-for-multiple-web-applications) afin d’utiliser le même fournisseur d’identité approuvé pour plusieurs applications web.

### <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

Dans cette section, vous allez activer l’authentification unique Azure AD sur le portail Azure.

Pour configurer l’authentification unique Azure AD avec SharePoint (local)

1. Sur le portail Azure, sélectionnez **Azure Active Directory** > **Applications d’entreprise**. Sélectionnez le nom de l’application d’entreprise créée précédemment, puis sélectionnez **Authentification unique**.

1. Dans la boîte de dialogue **Sélectionner une méthode d’authentification unique**, sélectionnez le mode **SAML** pour activer l’authentification unique.
 
1. Dans la page **Configurer l’authentification unique avec SAML**, sélectionnez l’icône **Modifier** pour ouvrir la boîte de dialogue **Configuration SAML de base**.

1. Dans la section **Configuration SAML de base**, effectuez les étapes suivantes :

    ![Informations d’authentification unique dans Domaine et URL de SharePoint (local)](./media/sharepoint-on-premises-tutorial/sp-identifier-reply.png)

    1. Dans la zone **Identificateur**, entrez une URL au format suivant : `urn:<sharepointFarmName>:<federationName>`.

    1. Dans la zone **URL de réponse**, entrez une URL au format suivant : `https://<YourSharePointSiteURL>/_trust/`.

    1. Dans la zone **URL de connexion**, entrez une URL au format suivant : `https://<YourSharePointSiteURL>/`.
    1. Sélectionnez **Enregistrer**.

    > [!NOTE]
    > Il ne s’agit pas de valeurs réelles. Mettez à jour ces valeurs avec l’URL de connexion, l’identificateur et l’URL de réponse réels.

1. Dans la page **Configurer l’authentification unique avec SAML**, dans la section **Certificat de signature SAML**, sélectionnez **Télécharger** pour télécharger le **Certificat (Base64)** en fonction des options définies par rapport à vos besoins, puis enregistrez-le sur votre ordinateur.

    ![Lien de téléchargement du certificat](./media/sharepoint-on-premises-tutorial/certificatebase64.png)

1. Dans la section **Configurer SharePoint (local)** , copiez la ou les URL appropriées en fonction de vos besoins.
    
    - **URL de connexion**
    
        Copiez l’URL de connexion et remplacez **/saml2** à la fin par **/wsfed** afin qu’elle ressemble à https://login.microsoftonline.com/2c4f1a9f-be5f-10ee-327d-a95dac567e4f/wsfed. (Cette URL n’est pas exacte.)

    - **Identificateur Azure AD**
    - **URL de déconnexion**

    > [!NOTE]
    > Cette URL ne peut pas être utilisée telle quelle dans SharePoint. Vous devez remplacer **/saml2** par **/wsfed**. L’application SharePoint (local) utilise un jeton SAML 1.1. Azure AD attend donc une requête WS Fed en provenance du serveur SharePoint. Après l’authentification, il émet le jeton SAML 1.1.

### <a name="configure-sharepoint-on-premises"></a>Configurer SharePoint (local)

1. Créez un fournisseur d’identité approuvé dans SharePoint Server 2016.

    Connectez-vous au serveur SharePoint, puis ouvrez SharePoint Management Shell. Spécifiez les valeurs suivantes :
    - **$realm** est la valeur d’identificateur de la section Domaine et URL pour SharePoint (local) dans le portail Azure.
    - **$wsfedurl** est l’URL du service d’authentification unique.
    - **$filepath** est le chemin où vous avez téléchargé le fichier de certificat à partir du portail Azure.

    Exécutez les commandes suivantes pour configurer un nouveau fournisseur d’identité approuvé.

    > [!TIP]
    > Si vous ne connaissez pas PowerShell ou que vous souhaitez en savoir plus sur son fonctionnement, consultez [SharePoint PowerShell](/powershell/sharepoint/overview).


    ```
    $realm = "urn:sharepoint:sps201x"
    $wsfedurl="https://login.microsoftonline.com/2c4f1a9f-be5f-10ee-327d-a95dac567e4f/wsfed"
    $filepath="C:\temp\SharePoint 2019 OnPrem.cer"
    $cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($filepath)
    New-SPTrustedRootAuthority -Name "AzureAD" -Certificate $cert
    $map1 = New-SPClaimTypeMapping -IncomingClaimType "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name" -IncomingClaimTypeDisplayName "name" -LocalClaimType "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn"
    $map2 = New-SPClaimTypeMapping -IncomingClaimType "http://schemas.microsoft.com/ws/2008/06/identity/claims/role" -IncomingClaimTypeDisplayName "Role" -SameAsIncoming
    $ap = New-SPTrustedIdentityTokenIssuer -Name "AzureAD" -Description "Azure AD SharePoint server 201x" -realm $realm -ImportTrustCertificate $cert -ClaimsMappings $map1,$map2 -SignInUrl $wsfedurl -IdentifierClaim $map1.InputClaimType
    ```
1. Activez le fournisseur d’identité approuvé pour votre application.

    1. Dans l’**Administration centrale**, accédez à **Gérer les applications web** et sélectionnez l’application web que vous souhaitez sécuriser avec Azure AD.

    1. Dans le ruban, sélectionnez **Fournisseurs d’authentification** et choisissez la zone que vous souhaitez utiliser.

    1. Sélectionnez **Fournisseur d’identité approuvé** et sélectionnez le fournisseur d’identité que vous venez d’inscrire, nommé *AzureAD*.

    1. Sélectionnez **OK**.

    ![Configuration de votre fournisseur d’authentification](./media/sharepoint-on-premises-tutorial/config-auth-provider.png)

### <a name="create-an-azure-ad-test-user-in-the-azure-portal"></a>Créer un utilisateur de test Azure AD dans le portail Azure

L’objectif de cette section est de créer un utilisateur de test dans le portail Azure.

1. Dans le portail Azure, dans le volet de gauche, sélectionnez **Azure Active Directory**. Dans le volet **Gérer**, sélectionnez **Utilisateurs**.

1. Sélectionnez **Tous les utilisateurs** > **Nouvel utilisateur** dans la partie supérieure de l’écran.

1. Sélectionnez **Créer un utilisateur** puis, dans les propriétés de l’utilisateur, effectuez les étapes suivantes. Vous pourrez peut-être créer des utilisateurs dans votre annuaire Azure AD à l’aide de votre suffixe de locataire ou de n’importe quel domaine vérifié. 

    1. Dans la zone **Nom**, entrez le nom d’utilisateur. Nous avons utilisé **TestUser**.
  
    1. Dans la zone **Nom d’utilisateur**, entrez `TestUser@yourcompanydomain.extension`. Cet exemple affiche `TestUser@contoso.com`.

       ![Boîte de dialogue Utilisateur](./media/sharepoint-on-premises-tutorial/user-properties.png)

    1. Cochez la case **Afficher le mot de passe**, puis notez la valeur affichée dans la zone **Mot de passe**.

    1. Sélectionnez **Create** (Créer).

    1. Vous pouvez maintenant partager le site avec TestUser@contoso.com et autoriser cet utilisateur à y accéder.

### <a name="create-an-azure-ad-security-group-in-the-azure-portal"></a>Créer un groupe de sécurité Azure AD dans le portail Azure

1. Sélectionnez **Azure Active Directory** > **Groupes**.

1. Sélectionnez **Nouveau groupe**.

1. Renseignez les zones **Type de groupe**, **Nom du groupe**, **Description du groupe** et **Type d’appartenance**. Sélectionnez les flèches pour sélectionner des membres, puis recherchez ou sélectionnez les membres que vous souhaitez ajouter au groupe. Choisissez **Sélectionner** pour ajouter les membres sélectionnés, puis **Créer**.

![Créer un groupe de sécurité Azure AD](./media/sharepoint-on-premises-tutorial/new-group.png)

### <a name="grant-permissions-to-an-azure-ad-account-in-sharepoint-on-premises"></a>Accorder à un compte Azure AD des autorisations d’accès à SharePoint (local)

Pour accorder l’accès à un utilisateur Azure AD dans SharePoint (local), partagez la collection de sites ou ajoutez l’utilisateur Azure AD à l’un des groupes de la collection de sites. Les utilisateurs peuvent maintenant se connecter à SharePoint 201x à l’aide d’identités Azure AD, mais il est encore possible d’améliorer l’expérience utilisateur. Par exemple, la recherche d’un utilisateur présente plusieurs résultats de recherche dans le sélecteur de personnes. Il existe un résultat de recherche pour chacun des types de revendications créés dans le mappage de revendications. Pour choisir un utilisateur à l’aide du sélecteur de personnes, vous devez entrer son nom d’utilisateur exactement et choisir le résultat de revendication **nom**.

![Résultats de recherche de revendications](./media/sharepoint-on-premises-tutorial/claims-search-results.png)

Il n’existe aucune validation des valeurs que vous recherchez, ce qui peut entraîner des fautes d’orthographe ou faire en sorte que des utilisateurs choisissent accidentellement le type de revendication incorrect. Cette situation risque d’empêcher les utilisateurs d’accéder aux ressources.

Pour corriger ce scénario lié au sélecteur de personnes, une solution open source appelée [AzureCP](https://yvand.github.io/AzureCP/) offre un fournisseur de revendications personnalisé pour SharePoint 2013, 2016 et 2019. Elle utilise l’API Microsoft Graph pour résoudre les valeurs entrées par les utilisateurs et effectuer la validation. Pour plus d’informations, consultez [AzureCP](https://yvand.github.io/AzureCP/).

  > [!NOTE]
  > Si vous n’utilisez pas AzureCP, vous pouvez ajouter des groupes en ajoutant l’ID du groupe Azure AD. Cependant, cette méthode n’est ni conviviale, ni fiable. Voici à quoi cela ressemble :
  > 
  >![Ajouter un groupe Azure AD à un groupe SharePoint par ID](./media/sharepoint-on-premises-tutorial/adding-group-by-id.png)
  
### <a name="grant-permissions-to-an-azure-ad-group-in-sharepoint-on-premises"></a>Accorder à un groupe Azure AD des autorisations d’accès à SharePoint (local)

Pour affecter des groupes de sécurité Azure AD à SharePoint (local), il est nécessaire d’utiliser un fournisseur de revendications personnalisé pour SharePoint Server. Cet exemple utilise AzureCP.

 > [!NOTE]
 > AzureCP n’est pas un produit Microsoft et n’est pas pris en charge par Support Microsoft. Pour télécharger, installer et configurer le fournisseur de revendications Azure sur la batterie de serveurs SharePoint (local), consultez le site web d’[AzureCP](https://yvand.github.io/AzureCP/). 

1. Configurez AzureCP sur la batterie de serveurs SharePoint (local) ou une autre solution de fournisseur de revendications personnalisé. Pour configurer AzureCP, consultez ce site web [AzureCP](https://yvand.github.io/AzureCP/Register-App-In-AAD.html).

1. Sur le portail Azure, sélectionnez **Azure Active Directory** > **Applications d’entreprise**. Sélectionnez le nom de l’application d’entreprise créée précédemment, puis sélectionnez **Authentification unique**.

1. Dans la page **Configurer l’authentification unique avec SAML**, modifiez la section **Attributs utilisateur et revendications**.

1. Sélectionnez **Ajouter une revendication de groupe**.

1. Sélectionnez les groupes associés à l’utilisateur qui doivent être retournés dans la revendication. Dans le cas présent, sélectionnez **Tous les groupes**. Dans la section **Attribut source**, sélectionnez **ID de groupe**, puis **Enregistrer**.

Pour accorder l’accès au groupe de sécurité Azure AD dans SharePoint (local), partagez la collection de sites ou ajoutez le groupe de sécurité Azure AD à l’un des groupes de la collection de sites.

1. Naviguez jusqu’à **Collection de sites SharePoint**. Sous **Paramètres du site** pour la collection de sites, sélectionnez **Personnes et groupes**. 

1. Sélectionnez le groupe SharePoint, puis sélectionnez **Nouveau** > **Ajouter des utilisateurs à ce groupe**. À mesure que vous tapez le nom de votre groupe, le sélecteur de personnes affiche le groupe de sécurité Azure AD.

    ![Ajouter un groupe Azure AD à un groupe SharePoint](./media/sharepoint-on-premises-tutorial/permission-azure-ad-group.png)

### <a name="grant-access-to-a-guest-account-to-sharepoint-on-premises-in-the-azure-portal"></a>Accorder à un compte invité l’accès à SharePoint (local) dans le portail Azure

Vous pouvez accorder l’accès à votre site SharePoint à un compte invité de manière cohérente, car l’UPN est désormais modifié. Par exemple, l’utilisateur `jdoe@outlook.com` est représenté sous la forme `jdoe_outlook.com#ext#@TENANT.onmicrosoft.com`. Pour partager votre site avec des utilisateurs externes, vous devez apporter quelques modifications dans la section **Attributs utilisateur et revendications** dans le portail Azure.

1. Sur le portail Azure, sélectionnez **Azure Active Directory** > **Applications d’entreprise**. Sélectionnez le nom de l’application d’entreprise créée précédemment, puis sélectionnez **Authentification unique**.

1. Dans la page **Configurer l’authentification unique avec SAML**, modifiez la section **Attributs utilisateur et revendications**.

1. Dans la zone **Revendication requise**, sélectionnez **Identificateur unique de l’utilisateur (ID de nom)** .

1. Remplacez la propriété **Attribut source** en spécifiant la valeur **user.localuserprincipalname**, puis sélectionnez **Enregistrer**.

    ![Attributs utilisateur et revendications - présentation initiale avec l’attribut source](./media/sharepoint-on-premises-tutorial/manage-claim.png)

1. À l’aide du ruban, revenez à **Authentification basée sur SAML**. À présent, la section **Attributs utilisateur et revendications** ressemble à ce qui suit : 

    ![Attributs utilisateur et revendications - présentation finale](./media/sharepoint-on-premises-tutorial/user-attributes-claims-final.png)

    > [!NOTE]
    > Avec cette configuration, le nom et le prénom ne sont pas obligatoires.

1. Dans le portail Azure, dans le volet de gauche, sélectionnez **Azure Active Directory**, puis **Utilisateurs**.

1. Sélectionnez **Nouvel utilisateur invité**.

1. Sélectionnez l’option **Inviter un utilisateur**. Renseignez les propriétés de l’utilisateur, puis sélectionnez **Inviter**.

1. Vous pouvez maintenant partager le site avec MyGuestAccount@outlook.com et autoriser cet utilisateur à y accéder.

    ![Partage d’un site avec un compte invité](./media/sharepoint-on-premises-tutorial/sharing-guest-account.png)

### <a name="configure-the-trusted-identity-provider-for-multiple-web-applications"></a>Configurer le fournisseur d’identité approuvé pour plusieurs applications web

La configuration peut s’appliquer à une application web. Toutefois, une configuration supplémentaire est nécessaire si vous prévoyez d’utiliser le même fournisseur d’identité approuvé pour plusieurs applications web. Par exemple, supposez que vous avez étendu une application web pour utiliser l’URL `https://sales.contoso.com`, et que vous souhaitez maintenant authentifier les utilisateurs auprès de `https://marketing.contoso.com`. Pour ce faire, mettez à jour le fournisseur d’identité de manière à honorer le paramètre WReply, et mettez à jour l’inscription de l’application dans Azure AD pour ajouter une URL de réponse.

1. Sur le portail Azure, sélectionnez **Azure Active Directory** > **Applications d’entreprise**. Sélectionnez le nom de l’application d’entreprise créée précédemment, puis sélectionnez **Authentification unique**.

1. Dans la page **Configurer l’authentification unique avec SAML**, modifiez la section **Configuration SAML de base**.

    ![Configuration SAML de base](./media/sharepoint-on-premises-tutorial/add-reply-url.png)

1. Pour **URL de réponse (URL Assertion Consumer Service)** , ajoutez l’URL pour les applications web supplémentaires et sélectionnez **Enregistrer**.

    ![Modifier la configuration SAML de base](./media/sharepoint-on-premises-tutorial/reply-url-for-web-application.png)

1. Sur le serveur SharePoint, ouvrez SharePoint 201x Management Shell et exécutez les commandes suivantes. Utilisez le nom de l’émetteur de jeton d’identité approuvé que vous avez utilisé plus tôt.
    ```
    $t = Get-SPTrustedIdentityTokenIssuer "AzureAD"
    $t.UseWReplyParameter=$true
    $t.Update()
    ```
1. Dans **Administration centrale**, accédez à l’application web et activez le fournisseur d’identité approuvé existant.

Vous pouvez avoir d’autres scénarios où vous souhaitez autoriser l’accès à votre instance de SharePoint (local) pour vos utilisateurs internes. Pour ce scénario, vous devez déployer Microsoft Azure Active Directory Connect pour autoriser la synchronisation de vos utilisateurs locaux avec Azure AD. Cette configuration est abordée dans un autre article.

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré SharePoint (local), vous pouvez appliquer le contrôle de session, qui protège contre l’exfiltration et l’infiltration des données sensibles de votre organisation en temps réel. Le contrôle de session est étendu à partir de l’accès conditionnel. [Découvrir comment appliquer un contrôle de session avec Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad)
