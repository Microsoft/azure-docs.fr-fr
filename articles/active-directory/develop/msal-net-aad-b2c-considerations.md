---
title: Azure AD B2C et MSAL.NET
titleSuffix: Microsoft identity platform
description: Considérations particulières à prendre en compte lors de l’utilisation d’Azure AD B2C avec la bibliothèque d’authentification Microsoft pour .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2020
ms.author: jeferrie
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 15f65da83f14f43a7892d52c6a2ed4e08580d367
ms.sourcegitcommit: 86acfdc2020e44d121d498f0b1013c4c3903d3f3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97614912"
---
# <a name="use-msalnet-to-sign-in-users-with-social-identities"></a>Utiliser MSAL.NET pour connecter les utilisateurs avec des identités sociales

Vous pouvez utiliser MSAL.NET pour connecter des utilisateurs au moyen d’identités sociales avec [Azure Active Directory B2C (Azure AD B2C)](../../active-directory-b2c/overview.md). Azure AD B2C s’appuie sur la notion de stratégies. Dans MSAL.NET, la spécification d’une stratégie se traduit par la fourniture d’une autorité.

- Lorsque vous instanciez l’application cliente publique, vous devez préciser la stratégie comme élément de cette autorité.
- Lorsque vous souhaitez appliquer une stratégie, appelez un remplacement de `AcquireTokenInteractive` qui accepte le paramètre `authority`.

Cet article s’applique à MSAL.NET 3.x. Pour MSAL.NET 2.x, consultez l’article concernant les [caractéristiques d’Azure AD B2C dans MSAL 2.x](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-Specifics-MSAL-2.x) du Wiki MSAL.NET sur GitHub.

## <a name="authority-for-an-azure-ad-b2c-tenant-and-policy"></a>Autorité pour une stratégie et un locataire Azure AD B2C

Le format d’autorité pour Azure AD B2C est le suivant : `https://{azureADB2CHostname}/tfp/{tenant}/{policyName}`

- `azureADB2CHostname` : nom du locataire Azure AD B2C plus l’hôte. Par exemple, *contosob2c.b2clogin.com*.
- `tenant` : nom de domaine ou ID de répertoire (locataire) du locataire Azure AD B2C. Par exemple, *contosob2c.onmicrosoft.com* ou un GUID.
- `policyName` : nom du flux d’utilisateur ou de la stratégie personnalisée à appliquer. Par exemple, une stratégie d’inscription/connexion comme *b2c_1_susi*.

Pour plus d’informations sur les autorités Azure AD B2C, consultez [Définir des URL de redirection vers b2clogin.com](../../active-directory-b2c/b2clogin.md).

## <a name="instantiating-the-application"></a>Instanciation de l’application

Fournissez l’autorité en appelant `WithB2CAuthority()` lorsque vous créez l’objet d’application :

```csharp
// Azure AD B2C Coordinates
public static string Tenant = "fabrikamb2c.onmicrosoft.com";
public static string AzureADB2CHostname = "fabrikamb2c.b2clogin.com";
public static string ClientID = "90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6";
public static string PolicySignUpSignIn = "b2c_1_susi";
public static string PolicyEditProfile = "b2c_1_edit_profile";
public static string PolicyResetPassword = "b2c_1_reset";

public static string AuthorityBase = $"https://{AzureADB2CHostname}/tfp/{Tenant}/";
public static string Authority = $"{AuthorityBase}{PolicySignUpSignIn}";
public static string AuthorityEditProfile = $"{AuthorityBase}{PolicyEditProfile}";
public static string AuthorityPasswordReset = $"{AuthorityBase}{PolicyResetPassword}";

application = PublicClientApplicationBuilder.Create(ClientID)
               .WithB2CAuthority(Authority)
               .Build();
```

## <a name="acquire-a-token-to-apply-a-policy"></a>Acquérir un jeton pour appliquer une stratégie

L’acquisition d’un jeton pour une API protégée par Azure AD B2C dans une application cliente publique vous oblige à utiliser les remplacements avec une autorité :

```csharp
IEnumerable<IAccount> accounts = await application.GetAccountsAsync();
AuthenticationResult ar = await application.AcquireTokenInteractive(scopes)
                                           .WithAccount(GetAccountByPolicy(accounts, policy))
                                           .WithParentActivityOrWindow(ParentActivityOrWindow)
                                           .ExecuteAsync();
```

Dans l’extrait de code précédent :

- `policy` est une chaîne contenant le nom de votre flux d’utilisateur ou stratégie personnalisée Azure AD B2C (par exemple, `PolicySignUpSignIn`).
- `ParentActivityOrWindow` est requis pour Android (l’activité) et facultatif pour d’autres plateformes qui prennent en charge une interface utilisateur parente, comme les fenêtres dans Microsoft Windows et UIViewController dans iOS. Pour plus d’informations sur la boîte de dialogue de l’interface utilisateur, consultez [WithParentActivityOrWindow](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Acquiring-tokens-interactively#withparentactivityorwindow) sur le Wiki MSAL.
- `GetAccountByPolicy(IEnumerable<IAccount>, string)` est une méthode qui trouve un compte pour une stratégie donnée. Par exemple :

  ```csharp
  private IAccount GetAccountByPolicy(IEnumerable<IAccount> accounts, string policy)
  {
      foreach (var account in accounts)
      {
          string userIdentifier = account.HomeAccountId.ObjectId.Split('.')[0];
          if (userIdentifier.EndsWith(policy.ToLower()))
              return account;
      }
      return null;
  }
  ```

L’application d’un flux utilisateur ou d’une stratégie personnalisée (par exemple, laisser l’utilisateur final modifier son profil ou réinitialiser son mot de passe) se fait actuellement en appelant `AcquireTokenInteractive`. Pour ces deux stratégies, vous n’utilisez pas le résultat de jeton/d’authentification renvoyé.

## <a name="profile-edit-policies"></a>Stratégies de modification de profil

Pour permettre à vos utilisateurs de se connecter avec une identité sociale et de modifier leur profil, appliquez la stratégie de modification de profil Azure AD B2C.

Pour cela, appelez `AcquireTokenInteractive` avec l’autorité pour cette stratégie. L’utilisateur étant déjà connecté et disposant d’une session de cookie active, utilisez `Prompt.NoPrompt` pour empêcher l’affichage de la boîte de dialogue de sélection de compte.

```csharp
private async void EditProfileButton_Click(object sender, RoutedEventArgs e)
{
    IEnumerable<IAccount> accounts = await app.GetAccountsAsync();
    try
    {
        var authResult = await app.AcquireToken(scopes:App.ApiScopes)
                            .WithAccount(GetUserByPolicy(accounts, App.PolicyEditProfile)),
                            .WithPrompt(Prompt.NoPrompt),
                            .WithB2CAuthority(App.AuthorityEditProfile)
                            .ExecuteAsync();
        DisplayBasicTokenInfo(authResult);
    }
    catch
    {
    }
}
```

## <a name="resource-owner-password-credentials-ropc"></a>Informations d’identification de mot de passe du propriétaire de ressource (ROPC)

Pour plus d’informations sur le flux ROPC, consultez [Se connecter avec l’octroi des informations de mot de passe du propriétaire de la ressource](v2-oauth-ropc.md).

Le flux ROPC est **déconseillé**, car il n’est pas sécurisé de demander son mot de passe à un utilisateur dans votre application. Pour plus d’informations sur ce problème, consultez l’article sur [la solution aux problèmes croissants associés aux mots de passe](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/).

En utilisant la combinaison nom d’utilisateur/mot de passe dans un flux ROPC, vous sacrifiez plusieurs choses :

- Les principes fondamentaux de l’identité moderne : le mot de passe peut être récupéré ou relu, car le secret partagé peut être intercepté. Par définition, ROPC est incompatible avec les flux sans mot de passe.
- Les utilisateurs devant utiliser l’authentification multifacteur (MFA) ne pourront pas se connecter (en l’absence de toute interaction).
- Les utilisateurs ne pourront pas utiliser l’authentification unique (SSO).

### <a name="configure-the-ropc-flow-in-azure-ad-b2c"></a>Configurer le flux ROPC dans Azure AD B2C

Dans votre locataire Azure AD B2C, créez un flux d’utilisateur et sélectionnez **Se connecter à l’aide de ROPC** pour activer ROPC pour ce flux d’utilisateur. Pour plus d’informations, consultez [Configurer le flux des informations d’identification par mot de passe du propriétaire de ressource](../../active-directory-b2c/add-ropc-policy.md).

`IPublicClientApplication` contient la méthode `AcquireTokenByUsernamePassword` :

```csharp
AcquireTokenByUsernamePassword(
            IEnumerable<string> scopes,
            string username,
            SecureString password)
```

Cette méthode `AcquireTokenByUsernamePassword` utilise les paramètres suivants :

- Les *étendues* pour lesquelles obtenir un jeton d’accès.
- Un nom d'utilisateur : *username*.
- Un mot de passe (*password*) SecureString pour l’utilisateur.

### <a name="limitations-of-the-ropc-flow"></a>Limitations du flux ROPC

Le flux ROPC **fonctionne uniquement pour les comptes locaux**, où vos utilisateurs se sont inscrits auprès d’Azure AD B2C au moyen d’une adresse e-mail ou d’un nom d’utilisateur. Ce flux ne fonctionne pas s’il y a fédération à un fournisseur d’identité externe pris en charge par Azure AD B2C (Facebook, Google, etc.).

## <a name="google-auth-and-embedded-webview"></a>Authentification Google et affichage web incorporé

Si vous utilisez Google comme fournisseur d’identité, nous vous conseillons d’utiliser le navigateur du système, car Google n’autorise pas [l’authentification à partir d’affichages web incorporés](https://developers.googleblog.com/2016/08/modernizing-oauth-interactions-in-native-apps.html). Actuellement, `login.microsoftonline.com` est une autorité approuvée avec Google et fonctionne avec l’affichage web incorporé. En revanche, `b2clogin.com` ne constitue pas une autorité approuvée auprès de Google, les utilisateurs ne seront donc pas en mesure de s’authentifier.

Nous fournirons une mise à jour concernant ce [problème](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues/688) en cas de changement.

## <a name="token-caching-in-msalnet"></a>Mise en cache de jeton dans MSAL.NET

### <a name="known-issue-with-azure-ad-b2c"></a>Problème connu avec Azure AD B2C

MSAL.NET prend en charge un [cache de jeton](/dotnet/api/microsoft.identity.client.tokencache). La clé de la mise en cache de jeton est basée sur les revendications renvoyées par le fournisseur d’identité.

Actuellement, MSAL.NET a besoin de deux revendications pour générer une clé de cache de jeton :

- `tid` (ID du locataire Azure AD)
- `preferred_username`

Ces deux revendications peuvent être absentes des scénarios Azure AD B2C, car tous les fournisseurs d’identité sociale (Facebook, Google et autres) ne les renvoient pas dans les jetons qu’ils renvoient à Azure AD B2C.

Un symptôme d’un tel scénario est que MSAL.NET renvoie `Missing from the token response` lorsque vous accédez à la valeur de revendication `preferred_username` dans les jetons émis par Azure AD B2C. MSAL utilise la valeur `Missing from the token response` pour `preferred_username` afin de maintenir la compatibilité croisée du cache entre les bibliothèques.

### <a name="workarounds"></a>Solutions de contournement

#### <a name="mitigation-for-missing-tenant-id"></a>Atténuation des risques pour l’ID de locataire manquant

La solution de contournement suggérée consiste à utiliser la [mise en cache par stratégie](#acquire-a-token-to-apply-a-policy) décrite précédemment.

Vous pouvez également utiliser la revendication `tid` si vous utilisez des [stratégies personnalisées](../../active-directory-b2c/custom-policy-get-started.md) dans Azure AD B2C. Les stratégies personnalisées peuvent renvoyer des revendication supplémentaires à votre application à l’aide de la [transformation des revendications](../../active-directory-b2c/claims-transformation-technical-profile.md).

#### <a name="mitigation-for-missing-from-the-token-response"></a>Atténuation pour « Absent de la réponse de jeton »

L’une des options consiste à utiliser la revendication `name` au lieu de `preferred_username`. Pour inclure la revendication `name` dans les jetons d’ID émis par Azure AD B2C, sélectionnez **Nom complet** lorsque vous configurez votre flux d’utilisateur.

Pour plus d’informations sur la spécification des revendications renvoyées par vos flux d’utilisateur, consultez [Tutoriel : Créer des flux d’utilisateur dans Azure AD B2C](../../active-directory-b2c/tutorial-create-user-flows.md).

## <a name="next-steps"></a>Étapes suivantes

D’autres informations détaillées sur l’acquisition de jetons de manière interactive avec MSAL.NET pour les applications Azure AD B2C sont fournies dans l’exemple suivant.

| Exemple | Plateforme | Description|
|------ | -------- | -----------|
|[active-directory-b2c-xamarin-native](https://github.com/Azure-Samples/active-directory-b2c-xamarin-native) | Xamarin iOS, Xamarin Android, UWP | Une application Xamarin Forms qui utilise MSAL.NET pour authentifier les utilisateurs via Azure AD B2C et accéder à une API web avec les jetons renvoyés.|
