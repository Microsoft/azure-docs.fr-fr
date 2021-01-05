---
title: Publier des applications clientes natives - Azure AD | Microsoft Docs
description: Explique comment activer des applications clientes natives de manière à communiquer avec le connecteur de proxy d'application Azure AD pour fournir un accès à distance sécurisé à vos applications locales.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/12/2020
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 09e813e0dff09885f104e43099b10d6f6f8a7f9b
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94657429"
---
# <a name="how-to-enable-native-client-applications-to-interact-with-proxy-applications"></a>Comment autoriser les applications clientes natives à interagir avec des applications de proxy

Vous pouvez utiliser le proxy d’application Azure Active Directory (Azure AD) pour publier des applications web, mais aussi pour publier des applications clientes natives configurées avec la bibliothèque d’authentification Microsoft (MSAL). Les applications clientes natives sont différentes des applications web parce qu’elles sont installées sur un appareil alors que les applications web sont accessibles via un navigateur.

Pour prendre en charge les applications clientes natives, le proxy d’application accepte les jetons émis par Azure AD qui sont envoyés dans l’en-tête. Le service de proxy d’application effectue l’authentification pour les utilisateurs. Cette solution n’utilise aucun jeton d’application pour l’authentification.

![Relation entre les utilisateurs finaux, Azure Active Directory et les applications publiées](./media/application-proxy-configure-native-client-application/richclientflow.png)

Pour publier des applications natives, utilisez la bibliothèque d’authentification Microsoft, qui prend en charge l’authentification et de nombreux environnements clients. Le proxy d’application s’intègre au scénario [Application de bureau qui appelle une API web pour le compte d’un utilisateur connecté](../develop/authentication-flows-app-scenarios.md#desktop-app-that-calls-a-web-api-on-behalf-of-a-signed-in-user).

Cet article vous guide dans quatre étapes pour publier une application native avec le proxy d’application et la bibliothèque Azure AD Authentication.

## <a name="step-1-publish-your-proxy-application"></a>Étape 1 : Publiez votre application proxy

Publiez votre application proxy comme vous le feriez pour toute autre application et affectez des utilisateurs pour accéder à votre application. Pour plus d'informations, consultez [Publier des applications avec le proxy d'application](application-proxy-add-on-premises-application.md).

## <a name="step-2-register-your-native-application"></a>Étape 2 : Inscrivez votre application native

Vous devez désormais enregistrer votre application dans Azure AD, en procédant comme suit :

1. Connectez-vous au [portail Azure Active Directory](https://aad.portal.azure.com/). Le **tableau de bord** du **Centre d’administration Azure Active Directory** s’affiche.
1. Dans la barre latérale, sélectionnez **Azure Active Directory**. La vue d’ensemble de **Microsoft Azure Active Directory** s’affiche.
1. Dans la barre latérale de la vue d’ensemble Azure AD, sélectionnez **Inscriptions d’applications**. La liste de toutes les inscriptions d’applications s’affiche.
1. Sélectionnez **Nouvelle inscription**. La page **Inscrire une application** s’affiche.

   ![Créer une inscription d’application sur le portail Azure](./media/application-proxy-configure-native-client-application/create.png)

1. Dans l’en-tête **Nom**, indiquez un nom d’affichage côté utilisateur pour l’application.
1. Sous l’en-tête **Types de comptes pris en charge**, sélectionnez un niveau d’accès en tenant compte des instructions suivantes :

   - Pour cibler uniquement les comptes internes de votre organisation, sélectionnez **Comptes dans cet annuaire organisationnel uniquement**.
   - Pour cibler uniquement les clients professionnels et dans l’éducation, sélectionnez **Comptes dans un annuaire organisationnel**.
   - Pour cibler le plus grand jeu d’identités Microsoft, sélectionnez **Comptes dans un annuaire organisationnel et comptes personnels Microsoft**.
1. Sous **URI de redirection**, sélectionnez **Client public (mobile et bureau)** , puis saisissez l’URI de redirection `https://login.microsoftonline.com/common/oauth2/nativeclient` pour votre application.
1. Sélectionnez et lisez les **stratégies de la plateforme Microsoft**, puis sélectionnez **Inscrire**. Une page présentant la nouvelle inscription d’application est créée et affichée.

Pour en savoir plus sur la création d’une inscription d’application, voir [Integrating applications with Azure Active Directory](../develop/quickstart-register-app.md) (Intégration d’applications à Azure Active Directory).

## <a name="step-3-grant-access-to-your-proxy-application"></a>Étape 3 : Accordez l’accès à votre application proxy

Maintenant que vous avez inscrit votre application native, vous pouvez lui donner accès à d’autres applications dans votre annuaire, pour accéder à l’application proxy dans le cas présent. Activez l’application native à exposer à l’application proxy, en procédant comme suit :

1. Dans la barre latérale de la nouvelle page d’inscription d’application, sélectionnez **Autorisations des API**. La page **Autorisations des API** de la nouvelle inscription d’application s’affiche.
1. Sélectionnez **Ajouter une autorisation**. La page **Demander des autorisations d’API** apparaît.
1. Sous le paramètre **Sélectionner une API**, choisissez **API utilisées par mon organisation**. Une liste s’affiche, affichant les applications de votre annuaire qui exposent des API.
1. Saisissez du texte dans la zone de recherche ou faites défiler la page pour trouver l’application proxy que vous avez publiée à [l’étape 1 : Publiez votre application proxy](#step-1-publish-your-proxy-application), puis sélectionnez l’application proxy.
1. Dans le titre **Quel type d'autorisation votre application nécessite-t-elle ?** , sélectionnez le type d’autorisation. Si votre application native doit accéder à l’API de l’application proxy en tant qu’utilisateur connecté, choisissez **Autorisations déléguées**.
1. Dans l’en-tête **Sélectionner les autorisations**, choisissez l’autorisation voulue, puis cliquez sur **Ajout d’autorisations**. La page **Autorisations des API** de votre application native affiche désormais l’application proxy et l’API d’autorisation que vous avez ajoutées.

## <a name="step-4-add-the-microsoft-authentication-library-to-your-code-net-c-sample"></a>Étape 4 : Ajoutez la bibliothèque d’authentification Microsoft à votre code (exemple C# .NET)

Modifiez le code de l’application native dans le contexte de l’authentification de la bibliothèque d’authentification Microsoft (MSAL) de manière à inclure le texte suivant : 

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

Les informations requises dans l’exemple de code sont disponibles dans le portail Azure AD, comme suit :

| Informations requises | Emplacement dans le portail Azure AD |
| --- | --- |
| \<Tenant ID> | **Azure Active Directory** > **Propriétés** > **ID de répertoire** |
| \<App ID of the Native app> | **Inscription d’application** > *votre application native* > **Présentation** > **ID d’application** |
| \<Scope> | **Inscription d’application** > *votre application native* > **Autorisations de l’API** > Cliquez sur l’API d’autorisation (user_impersonation). > Un panneau avec la légende **user_impersonation** apparaît sur le côté droit. > L’étendue est l’URL dans la zone d’édition.
| \<Proxy App Url> | URL externe et chemin d’accès à l’API

Une fois que vous avez modifié le code MSAL avec ces paramètres, vos utilisateurs peuvent s’authentifier auprès des applications clientes natives, même lorsqu’ils ne sont pas sur le réseau d’entreprise.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur le flux d’application native, voir [Applications natives dans Azure Active Directory](../azuread-dev/native-app.md).

Découvrez comment configurer [l’authentification unique SAML auprès des applications dans Azure Active Directory](sso-options.md#choosing-a-single-sign-on-method).