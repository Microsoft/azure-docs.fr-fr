---
title: Guide de migration Python ADAL vers MSAL | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment migrer votre application Python ADAL (bibliothèque d’authentification Active Directory) Azure vers Microsoft Authentication Library (MSAL) pour Python.
services: active-directory
author: rayluo
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.tgt_pltfrm: Python
ms.workload: identity
ms.date: 11/11/2019
ms.author: rayluo
ms.reviewer: marsma, rayluo, nacanuma
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: 731c608b8ac280a3a3c4e77c3a94533d258cb85d
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618040"
---
# <a name="adal-to-msal-migration-guide-for-python"></a>Guide de migration ADAL vers MSAL pour Python

Cet article souligne les changements que vous devez apporter pour migrer une application qui utilise la bibliothèque d’authentification Active Directory (ADAL) Azure vers une application qui utilise Microsoft Authentication Library (MSAL).

## <a name="difference-highlights"></a>Principales différences

ADAL fonctionne avec le point de terminaison Azure Active Directory v1.0 (Azure AD). Microsoft Authentication Library (MSAL) fonctionne avec la plateforme d’identités Microsoft, anciennement connue comme point de terminaison Azure Active Directory v2.0. La plateforme d’identités Microsoft diffère d’Azure AD v1.0 des manières suivantes :

Prend en charge :
  - Comptes professionnels et scolaires (comptes Azure AD approvisionnés)
  - Comptes personnels (comme Outlook.com ou Hotmail.com)
  - Vos clients qui ont déjà leur propre adresse e-mail ou identité sociale (comme LinkedIn, Facebook, Google) via l’offre Azure AD B2C

- Ses normes sont compatibles avec :
  - OAuth v2.0
  - OpenID Connect (OIDC)

Consultez l’article sur les [différences du point de terminaison de la plateforme d’identités Microsoft (v2.0)](../azuread-dev/azure-ad-endpoint-comparison.md) pour plus d’informations.

### <a name="scopes-not-resources"></a>Étendues au lieu de ressources

ADAL Python acquiert des jetons pour les ressources, alors que MSAL Python en acquiert pour les étendues. La surface d’API n’a plus de paramètre de ressource dans MSAL Python. Vous devez fournir des étendues sous la forme d’une liste de chaînes qui déclarent les autorisations souhaitées et les ressources demandées. Pour découvrir des exemples d’étendues, consultez [Étendues de Microsoft Graph](/graph/permissions-reference).

Vous pouvez ajouter le suffixe d’étendue `/.default` à la ressource pour faciliter la migration de vos applications du point de terminaison v1.0 (ADAL) vers le point de terminaison de la Plateforme d’identités Microsoft (MSAL). Par exemple, pour la valeur de ressource `https://graph.microsoft.com`, la valeur d’étendue équivalente est `https://graph.microsoft.com/.default`.  Si la ressource ne se trouve pas dans la forme d’URL, mais dans un ID de ressource de la forme `XXXXXXXX-XXXX-XXXX-XXXXXXXXXXXX`, vous pouvez toujours utiliser la valeur d’étendue `XXXXXXXX-XXXX-XXXX-XXXXXXXXXXXX/.default`.

Pour plus d’informations sur les différents types d’étendues, consultez [Autorisations et consentement dans le point de terminaison de la Plateforme d’identités Microsoft](./v2-permissions-and-consent.md) et les articles [Étendues pour une API web acceptant des jetons v1.0](./msal-v1-app-scopes.md).

### <a name="error-handling"></a>Gestion des erreurs

La bibliothèque d’authentification Active Directory (ADAL) pour Python utilise l’exception `AdalError` pour indiquer qu’un problème est survenu. MSAL pour Python utilise généralement des codes d’erreur à la place. Pour plus d’informations, consultez la rubrique [Gestion des erreurs MSAL pour Python](./msal-handling-exceptions.md?tabs=python).

### <a name="api-changes"></a>Modifications d'API

Le tableau suivant répertorie une API dans ADAL pour Python et celle à utiliser à sa place dans MSAL pour Python :

| API ADAL pour Python  | API MSAL pour Python |
| ------------------- | ---------------------------------- |
| [AuthenticationContext](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext)  | [PublicClientApplication ou ConfidentialClientApplication](https://msal-python.readthedocs.io/en/latest/#msal.ClientApplication.__init__)  |
| N/A  | [get_authorization_request_url()](https://msal-python.readthedocs.io/en/latest/#msal.ClientApplication.get_authorization_request_url)  |
| [acquire_token_with_authorization_code()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_authorization_code) | [acquire_token_by_authorization_code()](https://msal-python.readthedocs.io/en/latest/#msal.ClientApplication.acquire_token_by_authorization_code) |
| [acquire_token()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token) | [acquire_token_silent()](https://msal-python.readthedocs.io/en/latest/#msal.ClientApplication.acquire_token_silent) |
| [acquire_token_with_refresh_token()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_refresh_token) | N/A |
| [acquire_user_code()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_user_code) | [initiate_device_flow()](https://msal-python.readthedocs.io/en/latest/#msal.PublicClientApplication.initiate_device_flow) |
| [acquire_token_with_device_code()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_device_code) et [cancel_request_to_get_token_with_device_code()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.cancel_request_to_get_token_with_device_code) | [acquire_token_by_device_flow()](https://msal-python.readthedocs.io/en/latest/#msal.PublicClientApplication.acquire_token_by_device_flow) |
| [acquire_token_with_username_password()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_username_password) | [acquire_token_by_username_password()](https://msal-python.readthedocs.io/en/latest/#msal.PublicClientApplication.acquire_token_by_username_password) |
| [acquire_token_with_client_credentials()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_client_credentials) et [acquire_token_with_client_certificate()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_client_certificate) | [acquire_token_for_client()](https://msal-python.readthedocs.io/en/latest/#msal.ConfidentialClientApplication.acquire_token_for_client) |
| N/A | [acquire_token_on_behalf_of()](https://msal-python.readthedocs.io/en/latest/#msal.ConfidentialClientApplication.acquire_token_on_behalf_of) |
| [TokenCache()](https://adal-python.readthedocs.io/en/latest/#adal.TokenCache) | [SerializableTokenCache()](https://msal-python.readthedocs.io/en/latest/#msal.SerializableTokenCache) |
| N/A | Cache avec persistance, disponible à partir des [Extensions MSAL](https://github.com/marstr/original-microsoft-authentication-extensions-for-python) |

## <a name="migrate-existing-refresh-tokens-for-msal-python"></a>Migrer les jetons d’actualisation existants pour MSAL Python

La bibliothèque d’authentification Microsoft (MSAL) résume le concept de jetons d’actualisation. MSAL Python fournit un cache de jetons en mémoire par défaut, ce qui vous évite d’avoir à stocker, rechercher ou mettre à jour des jetons d’actualisation. Les utilisateurs verront également moins d’invites de connexion, car les jetons d’actualisation peuvent généralement être mis à jour sans intervention de l’utilisateur. Pour plus d’informations sur le cache de jetons, consultez [Sérialisation du cache de jetons personnalisés dans MSAL pour Python](msal-python-token-cache-serialization.md).

Le code suivant vous aidera à migrer vos jetons d’actualisation gérés par une autre bibliothèque OAuth2 (y compris, mais pas uniquement, ADAL Python) pour qu’ils soient gérés par MSAL pour Python. L’une des raisons de la migration de ces jetons d’actualisation est d’empêcher les utilisateurs existants d’avoir à se reconnecter lorsque vous migrez votre application vers MSAL pour Python.

La méthode de migration d’un jeton d’actualisation consiste à utiliser MSAL pour Python afin d’obtenir un nouveau jeton d’accès à l’aide du jeton d’actualisation précédent. Lorsque le nouveau jeton d’actualisation est retourné, MSAL pour Python le stocke dans le cache.
Depuis MSAL Python 1.3.0, nous fournissons une API à l’intérieur de MSAL à cet effet.
Reportez-vous à l’extrait de code suivant, mentionné dans [un exemple complet de migration de jetons d’actualisation avec MSAL Python](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.3.0/sample/migrate_rt.py#L28-L67).

```python
import msal
def get_preexisting_rt_and_their_scopes_from_elsewhere():
    # Maybe you have an ADAL-powered app like this
    #   https://github.com/AzureAD/azure-activedirectory-library-for-python/blob/1.2.3/sample/device_code_sample.py#L72
    # which uses a resource rather than a scope,
    # you need to convert your v1 resource into v2 scopes
    # See https://docs.microsoft.com/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison#scopes-not-resources
    # You may be able to append "/.default" to your v1 resource to form a scope
    # See https://docs.microsoft.com/azure/active-directory/develop/v2-permissions-and-consent#the-default-scope

    # Or maybe you have an app already talking to Microsoft identity platform v2,
    # powered by some 3rd-party auth library, and persist its tokens somehow.

    # Either way, you need to extract RTs from there, and return them like this.
    return [
        ("old_rt_1", ["scope1", "scope2"]),
        ("old_rt_2", ["scope3", "scope4"]),
        ]


# We will migrate all the old RTs into a new app powered by MSAL
app = msal.PublicClientApplication(
    "client_id", authority="...",
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache
    )

# We choose a migration strategy of migrating all RTs in one loop
for old_rt, scopes in get_preexisting_rt_and_their_scopes_from_elsewhere():
    result = app.acquire_token_by_refresh_token(old_rt, scopes)
    if "error" in result:
        print("Discarding unsuccessful RT. Error: ", json.dumps(result, indent=2))

print("Migration completed")
```


## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations, consultez la [comparaison entre v1.0 et v2.0](../azuread-dev/azure-ad-endpoint-comparison.md).
