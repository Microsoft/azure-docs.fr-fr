---
title: Azure AD B2C (MSAL Android) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez les considérations particulières à prendre en compte lors de l’utilisation d’Azure AD B2C avec Microsoft Authentication Library pour Android (MSAL.Android).
services: active-directory
author: brianmel
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 9/18/2019
ms.author: brianmel
ms.reviewer: rapong
ms.custom: aaddev
ms.openlocfilehash: 1c4f47fd771cfb92b3896963c96b39d9eb7d97b8
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96344876"
---
# <a name="use-msal-for-android-with-b2c"></a>Utiliser MSAL pour Android avec B2C

Microsoft Authentication Library (MSAL) permet aux développeurs d’applications d’authentifier les utilisateurs avec des identités sociales et locales à l’aide d’[Azure Active Directory B2C (Azure AD) B2C](../../active-directory-b2c/index.yml). Azure AD B2C est un service de gestion des identités. Utilisez-le pour personnaliser et contrôler la façon dont les clients s’inscrivent, se connectent et gèrent leurs profils lorsqu’ils utilisent vos applications.

## <a name="configure-known-authorities-and-redirect-uri"></a>Configurer les autorités connues et l’URI de redirection

Dans MSAL pour Android, les stratégies B2C (parcours utilisateur) sont configurées en tant qu’autorités individuelles.

À partir d’une application B2C qui a deux stratégies :
- Inscription/Connexion
    * Appelée `B2C_1_SISOPolicy`
- Modifier le profil
    * Appelée `B2C_1_EditProfile`

Le fichier de configuration de l’application va déclarer deux `authorities`. Une pour chaque stratégie. La propriété `type` de chaque autorité est `B2C`.

### `app/src/main/res/raw/msal_config.json`
```json
{
    "client_id": "<your_client_id_here>",
    "redirect_uri": "<your_redirect_uri_here>",
    "authorities": [{
            "type": "B2C",
            "authority_url": "https://contoso.b2clogin.com/tfp/contoso.onmicrosoft.com/B2C_1_SISOPolicy/",
            "default": true
        },
        {
            "type": "B2C",
            "authority_url": "https://contoso.b2clogin.com/tfp/contoso.onmicrosoft.com/B2C_1_EditProfile/"
        }
    ]
}
```

`redirect_uri` doit être inscrit dans la configuration de l’application, ainsi que dans `AndroidManifest.xml` pour prendre en charge la redirection pendant le [flux de l’octroi du code d’autorisation](../../active-directory-b2c/authorization-code-flow.md).

## <a name="initialize-ipublicclientapplication"></a>Initialiser IPublicClientApplication

`IPublicClientApplication` est construit par une méthode de fabrique pour permettre l’analyse asynchrone de la configuration de l’application.

```java
PublicClientApplication.createMultipleAccountPublicClientApplication(
    context, // Your application Context
    R.raw.msal_config, // Id of app JSON config
    new IPublicClientApplication.ApplicationCreatedListener() {
        @Override
        public void onCreated(IMultipleAccountPublicClientApplication pca) {
            // Application has been initialized.
        }

        @Override
        public void onError(MsalException exception) {
            // Application could not be created.
            // Check Exception message for details.
        }
    }
);
```

## <a name="interactively-acquire-a-token"></a>Acquérir un jeton de manière interactive

Pour acquérir un jeton de manière interactive avec MSAL, générez une instance `AcquireTokenParameters` et fournissez-la à la méthode `acquireToken`. La demande de jeton ci-dessous utilise l’autorité `default`.

```java
IMultipleAccountPublicClientApplication pca = ...; // Initialization not shown

AcquireTokenParameters parameters = new AcquireTokenParameters.Builder()
    .startAuthorizationFromActivity(activity)
    .withScopes(Arrays.asList("https://contoso.onmicrosoft.com/contosob2c/read")) // Provide your registered scope here
    .withPrompt(Prompt.LOGIN)
    .callback(new AuthenticationCallback() {
        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) {
            // Token request was successful, inspect the result
        }

        @Override
        public void onError(MsalException exception) {
            // Token request was unsuccessful, inspect the exception
        }

        @Override
        public void onCancel() {
            // The user cancelled the flow
        }
    }).build();

pca.acquireToken(parameters);
```

## <a name="silently-renew-a-token"></a>Renouveler un jeton silencieusement

Pour acquérir un jeton silencieusement avec MSAL, générez une instance `AcquireTokenSilentParameters` et fournissez-la à la méthode `acquireTokenSilentAsync`. Contrairement à la méthode `acquireToken`, une `authority` doit être spécifiée pour acquérir un jeton silencieusement.

```java
IMultilpeAccountPublicClientApplication pca = ...; // Initialization not shown
AcquireTokenSilentParameters parameters = new AcquireTokenSilentParameters.Builder()
    .withScopes(Arrays.asList("https://contoso.onmicrosoft.com/contosob2c/read")) // Provide your registered scope here
    .forAccount(account)
    // Select a configured authority (policy), mandatory for silent auth requests
    .fromAuthority("https://contoso.b2clogin.com/tfp/contoso.onmicrosoft.com/B2C_1_SISOPolicy/")
    .callback(new SilentAuthenticationCallback() {
        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) {
            // Token request was successful, inspect the result
        }

        @Override
        public void onError(MsalException exception) {
            // Token request was unsuccessful, inspect the exception
        }
    })
    .build();

pca.acquireTokenSilentAsync(parameters);
```

## <a name="specify-a-policy"></a>Spécifier une stratégie

Étant donné que les stratégies sont représentées dans B2C en tant qu’autorités distinctes, l’appel d’une stratégie autre que celle par défaut s’effectue en spécifiant une clause `fromAuthority` lors de la construction des paramètres `acquireToken` ou `acquireTokenSilent`.  Par exemple :

```java
AcquireTokenParameters parameters = new AcquireTokenParameters.Builder()
    .startAuthorizationFromActivity(activity)
    .withScopes(Arrays.asList("https://contoso.onmicrosoft.com/contosob2c/read")) // Provide your registered scope here
    .withPrompt(Prompt.LOGIN)
    .callback(...) // provide callback here
    .fromAuthority("<url_of_policy_defined_in_configuration_json>")
    .build();
```

## <a name="handle-password-change-policies"></a>Gérer les stratégies de changement de mot de passe

Le flux utilisateur d’inscription ou de connexion de compte local présente un lien « **Mot de passe oublié ?**  » . Lorsque vous cliquez sur ce lien, le flux d’utilisateur de réinitialisation du mot de passe n’est pas automatiquement déclenché.

Au lieu de cela, le code d’erreur `AADB2C90118` est retourné à votre application. Votre application doit gérer ce code d’erreur en exécutant un flux utilisateur spécifique qui réinitialise le mot de passe.

Pour intercepter un code d’erreur de réinitialisation de mot de passe, vous pouvez utiliser l’implémentation suivante à l’intérieur de votre `AuthenticationCallback` :

```java
new AuthenticationCallback() {

    @Override
    public void onSuccess(IAuthenticationResult authenticationResult) {
        // ..
    }

    @Override
    public void onError(MsalException exception) {
        final String B2C_PASSWORD_CHANGE = "AADB2C90118";

        if (exception.getMessage().contains(B2C_PASSWORD_CHANGE)) {
            // invoke password reset flow
        }
    }

    @Override
    public void onCancel() {
        // ..
    }
}
```

## <a name="use-iauthenticationresult"></a>Utiliser IAuthenticationResult

Une acquisition de jeton réussie génère un objet `IAuthenticationResult`. Cet objet contient le jeton d’accès, des revendications utilisateur et des métadonnées.

### <a name="get-the-access-token-and-related-properties"></a>Obtenir le jeton d’accès et les propriétés associées

```java
// Get the raw bearer token
String accessToken = authenticationResult.getAccessToken();

// Get the scopes included in the access token
String[] accessTokenScopes = authenticationResult.getScope();

// Gets the access token's expiry
Date expiry = authenticationResult.getExpiresOn();

// Get the tenant for which this access token was issued
String tenantId = authenticationResult.getTenantId();
```

### <a name="get-the-authorized-account"></a>Obtenir le compte autorisé

```java
// Get the account from the result
IAccount account = authenticationResult.getAccount();

// Get the id of this account - note for B2C, the policy name is a part of the id
String id = account.getId();

// Get the IdToken Claims
//
// For more information about B2C token claims, see reference documentation
// https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-reference-tokens
Map<String, ?> claims = account.getClaims();

// Get the 'preferred_username' claim through a convenience function
String username = account.getUsername();

// Get the tenant id (tid) claim through a convenience function
String tenantId = account.getTenantId();
```

### <a name="idtoken-claims"></a>Revendications IdToken

Les revendications retournées dans IdToken sont renseignées par le service d’émission de jeton de sécurité (STS), et non par MSAL. Selon le fournisseur d’identité (IdP) utilisé, certaines revendications peuvent être absentes. Actuellement, certains IdP ne fournissent pas la revendication `preferred_username`. Étant donné que cette revendication est utilisée par MSAL pour la mise en cache, une valeur d’espace réservé, `MISSING FROM THE TOKEN RESPONSE`, est utilisée à la place. Pour plus d’informations sur les revendications IdToken B2C, consultez [Vue d’ensemble des jetons dans Azure Active Directory B2C](../../active-directory-b2c/tokens-overview.md#claims).

## <a name="managing-accounts-and-policies"></a>Gestion des comptes et des stratégies

B2C traite chaque stratégie comme une autorité distincte. Ainsi, les jetons d’accès, les jetons d’actualisation et les jetons d’ID retournés par chaque stratégie ne sont pas interchangeables. Cela signifie que chaque stratégie retourne un objet `IAccount` distinct dont les jetons ne peuvent pas être utilisés pour appeler d’autres stratégies.

Chaque stratégie ajoute un `IAccount` au cache pour chaque utilisateur. Si un utilisateur se connecte à une application et appelle deux stratégies, il aura deux `IAccount`. Pour supprimer cet utilisateur du cache, vous devez appeler `removeAccount()` pour chaque stratégie.

Quand vous renouvelez des jetons pour une stratégie avec `acquireTokenSilent`, indiquez le même `IAccount` que celui retourné à l’issue des appels précédents de la stratégie à `AcquireTokenSilentParameters`. Si vous fournissez un compte retourné par une autre stratégie, une erreur se produit.

## <a name="next-steps"></a>Étapes suivantes

Découvrez-en plus sur Azure Active Directory B2C (Azure AD B2C) dans [Qu’est-ce qu’Azure Active Directory B2C ?](../../active-directory-b2c/overview.md)