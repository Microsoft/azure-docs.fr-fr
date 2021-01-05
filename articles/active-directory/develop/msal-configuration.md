---
title: Fichier de configuration Android MSAL | Azure
titleSuffix: Microsoft identity platform
description: Vue d’ensemble du fichier de configuration Microsoft Authentication Library (MSAL) Android, qui représente la configuration d’une application dans Azure Active Directory.
services: active-directory
author: shoatman
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: reference
ms.workload: identity
ms.date: 09/12/2019
ms.author: shoatman
ms.custom: aaddev
ms.reviewer: shoatman
ms.openlocfilehash: aa0ce6a5f909e67f0551c8667bb7e5c5e6d7eb04
ms.sourcegitcommit: b6f3ccaadf2f7eba4254a402e954adf430a90003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92275605"
---
# <a name="android-microsoft-authentication-library-configuration-file"></a>Fichier de configuration de la bibliothèque d’authentification Microsoft Authentication Android

La bibliothèque d’authentification Microsoft (MSAL) Android est fourni avec un [fichier JSON de configuration par défaut](https://github.com/AzureAD/microsoft-authentication-library-for-android/blob/dev/msal/src/main/res/raw/msal_default_config.json) que vous personnalisez pour définir le comportement de votre application cliente publique, notamment l’autorité par défaut, les autorités que vous allez utiliser, etc.

Cet article vous permet de comprendre les différents paramètres inclus dans ce fichier de configuration et la manière de spécifier le fichier de configuration à utiliser dans votre application MSAL.

## <a name="configuration-settings"></a>Paramètres de configuration

### <a name="general-settings"></a>Paramètres généraux :

| Propriété | Type de données | Obligatoire | Notes |
|-----------|------------|-------------|-------|
| `client_id` | String | Oui | ID client de votre application indiqué dans la [page d’inscription d’application](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) |
| `redirect_uri`   | String | Oui | URI de redirection de votre application indiqué dans la [page d’inscription d’application](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) |
| `broker_redirect_uri_registered` | Boolean | Non | Valeurs possibles : `true`, `false` |
| `authorities` | Liste \<Authority> | Non | Liste des autorités dont votre application a besoin |
| `authorization_user_agent` | AuthorizationAgent (enum) | Non | Valeurs possibles : `DEFAULT`, `BROWSER`, `WEBVIEW` |
| `http` | HttpConfiguration | Non | Configurez `HttpUrlConnection` `connect_timeout` et `read_timeout` |
| `logging` | LoggingConfiguration | Non | Spécifie le niveau de détail de la journalisation. Les configurations facultatives incluent : `pii_enabled`, qui prend une valeur booléenne et `log_level`, qui prend `ERROR`, `WARNING`, `INFO` ou `VERBOSE`. |

### <a name="client_id"></a>client_id

ID client ou ID d’application créé lors de l’inscription de votre application.

### <a name="redirect_uri"></a>redirect_uri

URI de redirection que vous avez inscrit lorsque vous avez inscrit votre application. Si l’URI de redirection pointe vers une application de répartiteur, reportez-vous à [URI de redirection pour les applications clientes publiques](msal-client-application-configuration.md#redirect-uri-for-public-client-apps) pour veiller à utiliser le bon format d’URI de redirection pour votre application de répartiteur.

### <a name="broker_redirect_uri_registered"></a>broker_redirect_uri_registered

Si vous souhaitez utiliser l'authentification répartie, la propriété `broker_redirect_uri_registered` doit être définie sur `true`. Dans un scénario d'authentification répartie, si l'application n'est pas au bon format pour communiquer avec le répartiteur, comme décrit dans [URI de redirection pour les applications clientes publiques](msal-client-application-configuration.md#redirect-uri-for-public-client-apps), l'application valide votre URI de redirection et renvoie une exception lorsqu'elle démarre.

### <a name="authorities"></a>authorities

Liste des autorités que vous connaissez et que vous approuvez. En plus des autorités listées ici, MSAL interroge aussi Microsoft pour obtenir la liste des clouds et autorités connus de Microsoft. Dans cette liste d’autorités, spécifiez le type d’autorité et tous les paramètres facultatifs supplémentaires, comme `"audience"`, à adapter au public de votre application en fonction de votre inscription d’application. Voici un exemple de liste d’autorités :

```javascript
// Example AzureAD and Personal Microsoft Account
{
    "type": "AAD",
    "audience": {
        "type": "AzureADandPersonalMicrosoftAccount"
    },
    "default": true // Indicates that this is the default to use if not provided as part of the acquireToken call
},
// Example AzureAD My Organization
{
    "type": "AAD",
    "audience": {
        "type": "AzureADMyOrg",
        "tenant_id": "contoso.com" // Provide your specific tenant ID here
    }
},
// Example AzureAD Multiple Organizations
{
    "type": "AAD",
    "audience": {
        "type": "AzureADMultipleOrgs"
    }
},
//Example PersonalMicrosoftAccount
{
    "type": "AAD",
    "audience": {
        "type": "PersonalMicrosoftAccount"
    }
}
```

#### <a name="map-aad-authority--audience-to-microsoft-identity-platform-endpoints"></a>Mapper l’autorité AAD et le public aux points de terminaison de la plateforme d’identités Microsoft

| Type | Public visé | ID client | Authority_Url | Point de terminaison obtenu | Notes |
|------|------------|------------|----------------|----------------------|---------|
| AAD | AzureADandPersonalMicrosoftAccount | | | `https://login.microsoftonline.com/common` | `common` est un alias de locataire pour l’emplacement du compte. Exemple : un locataire Azure Active Directory spécifique ou le système de comptes Microsoft. |
| AAD | AzureADMyOrg | contoso.com | | `https://login.microsoftonline.com/contoso.com` | Seuls les comptes présents dans contoso.com peuvent acquérir un jeton. Tout domaine vérifié, ou GUID de locataire, peut être utilisé comme ID de locataire. |
| AAD | AzureADMultipleOrgs | | | `https://login.microsoftonline.com/organizations` | Seuls les comptes Azure Active Directory peuvent être utilisés avec ce point de terminaison. Les comptes Microsoft peuvent être membres d’organisations. Pour acquérir un jeton à l’aide d’un compte Microsoft pour une ressource au sein d’une organisation, spécifiez le locataire organisationnel à partir duquel vous souhaitez obtenir le jeton. |
| AAD | PersonalMicrosoftAccount | | | `https://login.microsoftonline.com/consumers` | Seuls les comptes Microsoft peuvent utiliser ce point de terminaison. |
| B2C | | | Consultez Point de terminaison obtenu. | `https://login.microsoftonline.com/tfp/contoso.onmicrosoft.com/B2C_1_SISOPolicy/` | Seuls les comptes présents dans le locataire contoso.onmicrosoft.com peuvent acquérir un jeton. Dans cet exemple, la stratégie B2C fait partie du chemin de l’URL de l’autorité. |

> [!NOTE]
> La validation des autorités ne peut pas être activée et désactivée dans MSAL.
> Les autorités sont soit connues par vous en tant que développeur comme spécifié par le biais de la configuration, soit connues de Microsoft par le biais des métadonnées.
> Si MSAL reçoit une demande de jeton d’une autorité inconnue, une `MsalClientException` de type `UnknownAuthority` est levée.
> L'authentification répartie ne fonctionne pas pour Azure AD B2C.

#### <a name="authority-properties"></a>Propriétés des autorités

| Propriété | Type de données  | Obligatoire | Notes |
|-----------|-------------|-----------|--------|
| `type` | String | Oui | Reflète le public ou type de compte ciblé par votre application. Valeurs possibles : `AAD`, `B2C` |
| `audience` | Object | Non | S’applique uniquement quand le type est `AAD`. Spécifie l’identité ciblée par votre application. Utilisez la valeur issue de votre inscription d’application. |
| `authority_url` | String | Oui | Obligatoire uniquement quand le type est `B2C`. Spécifie l’URL de l’autorité ou la stratégie que votre application doit utiliser.  |
| `default` | boolean | Oui | Une seule propriété `"default":true` est exigée quand une ou plusieurs autorités sont spécifiées. |

#### <a name="audience-properties"></a>Propriétés du public

| Propriété | Type de données  | Obligatoire | Notes |
|-----------|-------------|------------|-------|
| `type` | String | Oui | Spécifie le public que votre application souhaite cibler. Valeurs possibles : `AzureADandPersonalMicrosoftAccount`, `PersonalMicrosoftAccount`, `AzureADMultipleOrgs`, `AzureADMyOrg` |
| `tenant_id` | String | Oui | Obligatoire uniquement avec `"type":"AzureADMyOrg"`. Facultatif pour les autres valeurs de `type`. Il peut s’agir d’un domaine de locataire, comme `contoso.com`, ou d’un ID de locataire comme `72f988bf-86f1-41af-91ab-2d7cd011db46`). |

### <a name="authorization_user_agent"></a>authorization_user_agent

Indique d’utiliser ou non une vue web incorporée, ou le navigateur installé par défaut sur l’appareil, lors de la connexion d’un compte ou de l’autorisation de l’accès à une ressource.

Valeurs possibles :
- `DEFAULT`: Préfère le navigateur système. Utilise la vue web incorporée si aucun navigateur n’est disponible sur l’appareil.
- `WEBVIEW`: Utilise la vue web incorporée.
- `BROWSER`: Utilise le navigateur par défaut installé sur l’appareil.

### <a name="multiple_clouds_supported"></a>multiple_clouds_supported

Pour les clients qui prennent en charge plusieurs clouds nationaux, spécifiez `true`. La plateforme d’identités Microsoft est alors automatiquement redirigée vers le cloud national approprié pendant l’autorisation et l’échange de jetons. Vous pouvez déterminer le cloud national du compte connecté en examinant l’autorité associée à `AuthenticationResult`. Notez que `AuthenticationResult` ne fournit pas l’adresse du point de terminaison propre au cloud national de la ressource pour laquelle vous demandez un jeton.

### <a name="broker_redirect_uri_registered"></a>broker_redirect_uri_registered

Valeur booléenne qui indique si vous utilisez un URI de redirection dans un répartiteur compatible avec le répartiteur Microsoft Identity. Affectez la valeur `false` si vous ne voulez pas utiliser le répartiteur au sein de votre application.

Si vous utilisez l’autorité AAD avec un public défini sur `"MicrosoftPersonalAccount"`, le répartiteur n’est pas utilisé.

### <a name="http"></a>http

Configurez des paramètres globaux pour les délais d’expiration HTTP, comme :

| Propriété | Type de données | Obligatoire | Notes |
| ---------|-----------|------------|--------|
| `connect_timeout` | int | Non | Durée en millisecondes |
| `read_timeout` | int | Non | Durée en millisecondes |

### <a name="logging"></a>journalisation

Les paramètres globaux suivants concernent la propriété logging :

| Propriété | Type de données  | Obligatoire | Notes |
| ----------|-------------|-----------|---------|
| `pii_enabled`  | boolean | Non | Indique d’émettre ou non des données personnelles. |
| `log_level`   | string | Non | Indique quels messages de journal générer. Les niveaux de journalisation pris en charge sont les suivants : `ERROR`, `WARNING`, `INFO` et `VERBOSE`. |
| `logcat_enabled` | boolean | Non | Indique de générer une sortie vers logcat en plus de l’interface de journalisation. |

### <a name="account_mode"></a>account_mode

Spécifie le nombre de comptes pouvant être utilisés à la fois dans votre application. Les valeurs possibles sont les suivantes :

- `MULTIPLE` (valeur par défaut)
- `SINGLE`

La construction d’une `PublicClientApplication` à l’aide d’un mode de compte qui ne correspond pas à ce paramètre lève une exception.

Pour plus d’informations sur les différences entre les monocomptes et les multicomptes, consultez [Applications clientes publiques monocomptes et multicomptes](single-multi-account.md).

### <a name="browser_safelist"></a>browser_safelist

Liste verte des navigateurs compatibles avec MSAL. Ces navigateurs gèrent correctement les redirections vers des intentions personnalisées. Vous pouvez ajouter des éléments à cette liste. La valeur par défaut est fournie dans la configuration par défaut indiquée ci-dessous.
``
## <a name="the-default-msal-configuration-file"></a>Fichier de configuration MSAL par défaut

La configuration MSAL par défaut fournie avec MSAL est indiquée ci-dessous. Vous pouvez voir la dernière version sur [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-android/blob/dev/msal/src/main/res/raw/msal_default_config.json).

Cette configuration est complétée par les valeurs que vous fournissez. Les valeurs que vous fournissez remplacent les valeurs par défaut.

```javascript
{
  "authorities": [
    {
      "type": "AAD",
      "audience": {
        "type": "AzureADandPersonalMicrosoftAccount"
      },
      "default": true
    }
  ],
  "authorization_user_agent": "DEFAULT",
  "multiple_clouds_supported": false,
  "broker_redirect_uri_registered": false,
  "http": {
    "connect_timeout": 10000,
    "read_timeout": 30000
  },
  "logging": {
    "pii_enabled": false,
    "log_level": "WARNING",
    "logcat_enabled": false
  },
  "shared_device_mode_supported": false,
  "account_mode": "MULTIPLE",
  "browser_safelist": [
    {
      "browser_package_name": "com.android.chrome",
      "browser_signature_hashes": [
        "7fmduHKTdHHrlMvldlEqAIlSfii1tl35bxj1OXN5Ve8c4lU6URVu4xtSHc3BVZxS6WWJnxMDhIfQN0N0K2NDJg=="
      ],
      "browser_use_customTab" : true,
      "browser_version_lower_bound": "45"
    },
    {
      "browser_package_name": "com.android.chrome",
      "browser_signature_hashes": [
        "7fmduHKTdHHrlMvldlEqAIlSfii1tl35bxj1OXN5Ve8c4lU6URVu4xtSHc3BVZxS6WWJnxMDhIfQN0N0K2NDJg=="
      ],
      "browser_use_customTab" : false
    },
    {
      "browser_package_name": "org.mozilla.firefox",
      "browser_signature_hashes": [
        "2gCe6pR_AO_Q2Vu8Iep-4AsiKNnUHQxu0FaDHO_qa178GByKybdT_BuE8_dYk99G5Uvx_gdONXAOO2EaXidpVQ=="
      ],
      "browser_use_customTab" : false
    },
    {
      "browser_package_name": "org.mozilla.firefox",
      "browser_signature_hashes": [
        "2gCe6pR_AO_Q2Vu8Iep-4AsiKNnUHQxu0FaDHO_qa178GByKybdT_BuE8_dYk99G5Uvx_gdONXAOO2EaXidpVQ=="
      ],
      "browser_use_customTab" : true,
      "browser_version_lower_bound": "57"
    },
    {
      "browser_package_name": "com.sec.android.app.sbrowser",
      "browser_signature_hashes": [
        "ABi2fbt8vkzj7SJ8aD5jc4xJFTDFntdkMrYXL3itsvqY1QIw-dZozdop5rgKNxjbrQAd5nntAGpgh9w84O1Xgg=="
      ],
      "browser_use_customTab" : true,
      "browser_version_lower_bound": "4.0"
    },
    {
      "browser_package_name": "com.sec.android.app.sbrowser",
      "browser_signature_hashes": [
        "ABi2fbt8vkzj7SJ8aD5jc4xJFTDFntdkMrYXL3itsvqY1QIw-dZozdop5rgKNxjbrQAd5nntAGpgh9w84O1Xgg=="
      ],
      "browser_use_customTab" : false
    },
    {
      "browser_package_name": "com.cloudmosa.puffinFree",
      "browser_signature_hashes": [
        "1WqG8SoK2WvE4NTYgr2550TRhjhxT-7DWxu6C_o6GrOLK6xzG67Hq7GCGDjkAFRCOChlo2XUUglLRAYu3Mn8Ag=="
      ],
      "browser_use_customTab" : false
    },
    {
      "browser_package_name": "com.duckduckgo.mobile.android",
      "browser_signature_hashes": [
        "S5Av4cfEycCvIvKPpKGjyCuAE5gZ8y60-knFfGkAEIZWPr9lU5kA7iOAlSZxaJei08s0ruDvuEzFYlmH-jAi4Q=="
      ],
      "browser_use_customTab" : false
    },
    {
      "browser_package_name": "com.explore.web.browser",
      "browser_signature_hashes": [
        "BzDzBVSAwah8f_A0MYJCPOkt0eb7WcIEw6Udn7VLcizjoU3wxAzVisCm6bW7uTs4WpMfBEJYf0nDgzTYvYHCag=="
      ],
      "browser_use_customTab" : false
    },

    {
      "browser_package_name": "com.ksmobile.cb",
      "browser_signature_hashes": [
        "lFDYx1Rwc7_XUn4KlfQk2klXLufRyuGHLa3a7rNjqQMkMaxZueQfxukVTvA7yKKp3Md3XUeeDSWGIZcRy7nouw=="
      ],
      "browser_use_customTab" : false
    },

    {
      "browser_package_name": "com.microsoft.emmx",
      "browser_signature_hashes": [
        "Ivy-Rk6ztai_IudfbyUrSHugzRqAtHWslFvHT0PTvLMsEKLUIgv7ZZbVxygWy_M5mOPpfjZrd3vOx3t-cA6fVQ=="
      ],
      "browser_use_customTab" : false
    },

    {
      "browser_package_name": "com.opera.browser",
      "browser_signature_hashes": [
        "FIJ3IIeqB7V0qHpRNEpYNkhEGA_eJaf7ntca-Oa_6Feev3UkgnpguTNV31JdAmpEFPGNPo0RHqdlU0k-3jWJWw=="
      ],
      "browser_use_customTab" : false
    },

    {
      "browser_package_name": "com.opera.mini.native",
      "browser_signature_hashes": [
        "TOTyHs086iGIEdxrX_24aAewTZxV7Wbi6niS2ZrpPhLkjuZPAh1c3NQ_U4Lx1KdgyhQE4BiS36MIfP6LbmmUYQ=="
      ],
      "browser_use_customTab" : false
    },

    {
      "browser_package_name": "mobi.mgeek.TunnyBrowser",
      "browser_signature_hashes": [
        "RMVoXuK1sfJZuGZ8onG1yhMc-sKiAV2NiB_GZfdNlN8XJ78XEE2wPM6LnQiyltF25GkHiPN2iKQiGwaO2bkyyQ=="
      ],
      "browser_use_customTab" : false
    },

    {
      "browser_package_name": "org.mozilla.focus",
      "browser_signature_hashes": [
        "L72dT-stFqomSY7sYySrgBJ3VYKbipMZapmUXfTZNqOzN_dekT5wdBACJkpz0C6P0yx5EmZ5IciI93Q0hq0oYA=="
      ],
      "browser_use_customTab" : false
    }
  ]
}
```
## <a name="example-basic-configuration"></a>Exemple de configuration de base

L’exemple suivant illustre une configuration de base qui spécifie l’ID client, l’URI de redirection, l’inscription ou non d’une redirection répartiteur et la liste des autorités.

```javascript
{
  "client_id" : "4b0db8c2-9f26-4417-8bde-3f0e3656f8e0",
  "redirect_uri" : "msauth://com.microsoft.identity.client.sample.local/1wIqXSqBj7w%2Bh11ZifsnqwgyKrY%3D",
  "broker_redirect_uri_registered": true,
  "authorities" : [
    {
      "type": "AAD",
      "audience": {
        "type": "AzureADandPersonalMicrosoftAccount"
      }
      "default": true
    }
  ]
}
```

## <a name="how-to-use-a-configuration-file"></a>Comment utiliser un fichier de configuration

1. Créez un fichier de configuration. Nous vous recommandons de créer votre fichier de configuration personnalisé dans `res/raw/auth_config.json`. Mais vous pouvez le placer à l’endroit qui vous convient.
2. Indiquez à MSAL où rechercher votre configuration quand vous construisez `PublicClientApplication`. Par exemple :

   ```java
   //On Worker Thread
   IMultipleAccountPublicClientApplication sampleApp = null; 
   sampleApp = new PublicClientApplication.createMultipleAccountPublicClientApplication(getApplicationContext(), R.raw.auth_config);
   ```
