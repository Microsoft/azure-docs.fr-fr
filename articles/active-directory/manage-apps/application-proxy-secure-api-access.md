---
title: Accéder aux API locales avec le proxy d’application Azure AD
description: Le proxy d’application Azure Active Directory permet aux applications natives d’accéder de façon sécurisée aux API et à la logique métier que vous hébergez localement ou sur des machines virtuelles cloud.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 02/12/2020
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: eaa4a8f92809dcf44969958573231ad37d908daa
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94835695"
---
# <a name="secure-access-to-on-premises-apis-with-azure-ad-application-proxy"></a>Sécuriser l’accès aux API locales avec le proxy d’application Azure AD

Vos API de logique métier peuvent s’exécuter localement ou être hébergées sur des machines virtuelles dans le cloud. Vos applications natives Android, iOS, Mac ou Windows doivent interagir avec les points de terminaison d’API afin d’utiliser des données ou de permettre une interaction utilisateur. Le proxy d'application Azure AD et la [bibliothèque d'authentification Microsoft (MSAL)](../azuread-dev/active-directory-authentication-libraries.md) permettent à vos applications natives d'accéder de façon sécurisée à vos API locales. L’utilisation du proxy d’application Azure Active Directory est plus rapide et plus sécurisée que l’ouverture des ports du pare-feu et le contrôle de l’authentification et des autorisations au niveau de l’application.

Cet article explique comment configurer une solution de proxy d’application Azure AD pour l’hébergement d’un service API web auquel peuvent accéder les applications natives.

## <a name="overview"></a>Vue d’ensemble

Le schéma suivant montre une méthode traditionnelle utilisée pour publier des API locales. Cette approche nécessite l’ouverture des ports entrants 80 et 443.

![Accès classique aux API](./media/application-proxy-secure-api-access/overview-publish-api-open-ports.png)

Le diagramme suivant montre comment utiliser le proxy d’application Azure AD pour publier des API de façon sécurisée, sans ouvrir les ports entrants :

![Proxy d’application Azure AD - Accès aux API](./media/application-proxy-secure-api-access/overview-publish-api-app-proxy.png)

Le proxy d’application Azure AD constitue le cœur de la solution. En effet, il sert de point de terminaison public pour l’accès aux API et fournit des fonctionnalités d’authentification et d’autorisation. Vous pouvez également accéder à vos API à partir d’un large éventail de plateformes à l’aide des bibliothèques [MSAL (Microsoft Authentication Library)](../azuread-dev/active-directory-authentication-libraries.md).

Étant donné que les fonctionnalités d’autorisation et d’authentification du proxy d’application Azure AD sont intégrées à Azure AD, vous pouvez utiliser l’accès conditionnel Azure AD pour garantir que seuls les appareils approuvés puissent accéder aux API publiées via le proxy d’application. Utilisez les options Jonction Azure AD ou Joint à une version hybride d’Azure AD pour les postes de travail, et l’option Géré par Intune pour les appareils. Vous pouvez également tirer parti des fonctionnalités d'Azure Active Directory Premium, comme Azure AD Multi-Factor Authentication, et de la sécurité reposant sur le Machine Learning d'[Azure Identity Protection](../identity-protection/overview-identity-protection.md).

## <a name="prerequisites"></a>Prérequis

Pour suivre cette procédure pas à pas, vous avez besoin des éléments suivants :

- Un accès administrateur à un annuaire Azure, avec un compte qui peut créer et inscrire des applications
- L’exemple d’API web et les applications clientes natives issues de [https://github.com/jeevanbisht/API-NativeApp-ADAL-SampleApp](https://github.com/jeevanbisht/API-NativeApp-ADAL-SampleApp)

## <a name="publish-the-api-through-application-proxy"></a>Publier l’API via le proxy d’application

Pour publier une API en dehors de votre intranet via le proxy d’application, vous devez suivre la même procédure que pour la publication des applications web. Pour plus d’informations, consultez [Didacticiel : Ajouter une application locale pour un accès à distance via le service Proxy d’application d’Azure Active Directory](application-proxy-add-on-premises-application.md).

Pour publier l’API web SecretAPI via le proxy d’application :

1. Générez puis publiez l’exemple de projet SecretAPI comme une application web ASP.NET sur votre ordinateur local ou dans l’intranet. Vérifiez que vous pouvez accéder à l’application web localement.

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory**. Supprimez ensuite toutes les **applications d’entreprise**.

1. En haut de la page **Applications d’entreprise - Toutes les applications**, sélectionnez **Nouvelle application**.

1. Dans la page **Ajouter une application**, sélectionnez **Applications locales**. La page **Ajouter votre propre application locale** s’affiche.

1. Si aucun connecteur Proxy d’application n’est installé, vous allez être invité à l’installer. Sélectionnez **Télécharger le connecteur Proxy d’application** pour télécharger et installer le connecteur.

1. Une fois que vous avez installé le connecteur Proxy d’application, dans la page **Ajouter votre propre application locale** :

   1. À côté de *Nom*, entrez **SecretAPI**.

   1. À côté de **URL interne**, entrez l’URL à utiliser pour accéder à l’API à partir de votre intranet.

   1. Vérifiez que l’option **Pré-authentification** est configurée sur **Azure Active Directory**.

   1. En haut de la page, sélectionnez **Ajouter**, puis attendez que l’application soit créée.

   ![Ajouter une API](./media/application-proxy-secure-api-access/3-add-api-app.png)

1. Dans la page **Applications d’entreprise - Toutes les applications**, sélectionnez l’application **SecretAPI**.

1. Dans la page **SecretAPI - Vue d’ensemble**, sélectionnez **Propriétés** dans le volet de navigation gauche.

1. Ces API ne doivent pas être disponibles pour les utilisateurs finaux dans le panneau **MyApps**. Vous devez donc configurer **Visible par les utilisateurs ?** sur **Non** au bas de la page **Propriétés**, puis sélectionner **Enregistrer**.

   ![Non visible par les utilisateurs](./media/application-proxy-secure-api-access/5-not-visible-to-users.png)

Vous avez publié votre API web via le proxy d’application Azure AD. Maintenant, ajoutez les utilisateurs qui peuvent accéder à l’application.

1. Dans la page **SecretAPI - Vue d’ensemble**, sélectionnez **Utilisateurs et groupes** dans le volet de navigation gauche.

1. Dans la page **Utilisateurs et groupes**, sélectionnez **Ajouter un utilisateur**.

1. Dans la page **Ajouter une affectation**, sélectionnez **Utilisateurs et groupes**.

1. Dans la page **Utilisateurs et groupes**, recherchez puis sélectionnez les utilisateurs pouvant accéder à l’application, y compris vous-même. Après avoir sélectionné tous les utilisateurs, sélectionnez **Sélectionner**.

   ![Sélectionner et affecter les utilisateurs](./media/application-proxy-secure-api-access/7-select-admin-user.png)

1. De retour dans la page **Ajouter une affectation**, sélectionnez **Affecter**.

> [!NOTE]
> Les API qui utilisent l’authentification Windows intégrée peuvent nécessiter des [étapes supplémentaires](./application-proxy-configure-single-sign-on-with-kcd.md).

## <a name="register-the-native-app-and-grant-access-to-the-api"></a>Inscrire l’application native et accorder l’accès à l’API

Les applications natives sont des programmes conçus pour être utilisés sur une plateforme ou un appareil spécifique. Pour que votre application native puisse se connecter et accéder à une API, vous devez l’inscrire dans Azure AD. Les étapes suivantes montrent comment inscrire une application native et lui donner accès à l’API web que vous avez publiée via le proxy d’application.

Pour inscrire l’application native AppProxyNativeAppSample :

1. Dans la page **Vue d’ensemble** d’Azure Active Directory, sélectionnez **Inscriptions des applications**, puis, en haut du volet **Inscriptions des applications**, sélectionnez **Nouvelle inscription**.

1. Dans la page **Inscrire une application** :

   1. Sous **Nom**, entrez *AppProxyNativeAppSample*.

   1. Sous **Types de comptes pris en charge**, sélectionnez **Comptes dans un annuaire organisationnel et comptes personnels Microsoft**.

   1. Sous **URI de redirection**, dans la liste déroulante, sélectionnez **Client public (mobile et poste de travail)** , puis entrez *https://login.microsoftonline.com/common/oauth2/nativeclient* .

   1. Sélectionnez **Inscrire**, puis attendez que l’application soit correctement inscrite.

      ![Nouvelle inscription d’application](./media/application-proxy-secure-api-access/8-create-reg-ga.png)

Vous venez d’inscrire l’application AppProxyNativeAppSample dans Azure Active Directory. Pour permettre à votre application native d’accéder à l’API web SecretAPI :

1. Dans la page **Vue d’ensemble** > **Inscriptions** d’Azure Active Directory, sélectionnez l’application **AppProxyNativeAppSample**.

1. Dans la page **AppProxyNativeAppSample**, sélectionnez **API autorisées** dans le volet de navigation gauche.

1. Dans la page **API autorisées**, sélectionnez **Ajouter une autorisation**.

1. Dans la première page **Demander des autorisations d’API**, sélectionnez l’onglet **API utilisées par mon organisation**, puis recherchez et sélectionnez **SecretAPI**.

1. Dans la page **Demander des autorisations d’API** suivante, cochez la case située à côté de **user_impersonation**, puis sélectionnez **Ajouter des autorisations**.

    ![Sélectionnez une API](./media/application-proxy-secure-api-access/10-secretapi-added.png)

1. Dans la page **API autorisées**, vous pouvez sélectionner **Accorder un consentement d’administrateur pour Contoso** afin d’éviter aux utilisateurs de donner chacun leur consentement à l’application.

## <a name="configure-the-native-app-code"></a>Configurer le code de l’application native

La dernière étape consiste à configurer l’application native. Dans l’exemple d’application NativeClient, l’extrait de code suivant tiré du fichier *Form1.cs* entraîne l’acquisition du jeton par la bibliothèque MSAL pour demander l’appel d’API, et l’ajout du jeton en tant que porteur à l’en-tête de l’application.

   ```
   // Acquire Access Token from AAD for Proxy Application
 IPublicClientApplication clientApp = PublicClientApplicationBuilder
.Create(<App ID of the Native app>)
.WithDefaultRedirectUri() // will automatically use the default Uri for native app
.WithAuthority("https://login.microsoftonline.com/{<Tenant ID>}")
.Build();

AuthenticationResult authResult = null;
var accounts = await clientApp.GetAccountsAsync();
IAccount account = accounts.FirstOrDefault();

IEnumerable<string> scopes = new string[] {"<Scope>"};

try
 {
    authResult = await clientApp.AcquireTokenSilent(scopes, account).ExecuteAsync();
 }
    catch (MsalUiRequiredException ex)
 {
     authResult = await clientApp.AcquireTokenInteractive(scopes).ExecuteAsync();                
 }
 
if (authResult != null)
 {
  //Use the Access Token to access the Proxy Application
  
  HttpClient httpClient = new HttpClient();
  HttpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", authResult.AccessToken);
  HttpResponseMessage response = await httpClient.GetAsync("<Proxy App Url>");
 }
```

Pour configurer l’application native dans le but de vous connecter à Azure Active Directory et d’appeler le proxy d’application de l’API, remplacez les valeurs des espaces réservés du fichier *App.config* de l’exemple d’application NativeClient par les valeurs issues d’Azure AD :

- Collez l’**ID de l’annuaire (locataire)** dans le champ `<add key="ida:Tenant" value="" />`. Vous pouvez trouver et copier cette valeur (un GUID) dans la page **Vue d’ensemble** de l’une de vos applications.

- Collez l’**ID d’application (client)** d’AppProxyNativeAppSample dans le champ `<add key="ida:ClientId" value="" />`. Vous pouvez trouver et copier cette valeur (un GUID) dans la page **Vue d’ensemble** d’AppProxyNativeAppSample.

- Collez l’**URI de redirection** AppProxyNativeAppSample dans le champ `<add key="ida:RedirectUri" value="" />`. Vous pouvez trouver et copier cette valeur (un URI) dans la page **Authentification** d’AppProxyNativeAppSample.

- Collez l’**URI d’ID d’application** de SecretAPI dans le champ `<add key="todo:TodoListResourceId" value="" />`. Vous pouvez trouver et copier cette valeur (un URI) dans la page **Exposer une API** de SecretAPI.

- Collez l’**URL de la page d’accueil** de SecretAPI dans le champ `<add key="todo:TodoListBaseAddress" value="" />`. Vous pouvez trouver et copier cette valeur (une URL) dans la page **Personnalisation** de SecretAPI.

Après avoir configuré les paramètres, générez puis exécutez l’application native. Lorsque vous sélectionnez le bouton **Connexion**, l’application vous permet de vous connecter, puis affiche un écran de confirmation vous informant que vous êtes bien connecté à SecretAPI.

![Capture d'écran affichant un message qui indique que la connexion à SecretAPI a abouti ainsi qu'un bouton OK.](./media/application-proxy-secure-api-access/success.png)

## <a name="next-steps"></a>Étapes suivantes

- [Tutoriel : Ajouter une application locale pour un accès à distance via le proxy d’application d’Azure Active Directory](application-proxy-add-on-premises-application.md)
- [Démarrage rapide : configurer une application cliente pour accéder aux API web](../develop/quickstart-configure-app-access-web-apis.md).
- [Comment autoriser les applications clientes natives à interagir avec des applications de proxy](application-proxy-configure-native-client-application.md)