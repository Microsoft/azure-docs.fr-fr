---
title: Configurer un fournisseur OpenID Connect (préversion)
description: Découvrez comment configurer un fournisseur OpenID Connect en tant que fournisseur d’identité pour votre application App Service ou Azure Functions.
ms.topic: article
ms.date: 07/08/2020
ms.reviewer: mahender
ms.openlocfilehash: e8112f2dc20175e81cfa8388440b2d9aef6a419c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90983870"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-login-using-an-openid-connect-provider-preview"></a>Configurer votre application App Service ou Azure Functions pour la connexion à l’aide d’un fournisseur OpenID Connect (préversion)

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

Cet article vous montre comment configurer Azure App Service ou Azure Functions pour utiliser un fournisseur d’authentification personnalisé qui respecte la [spécification OpenID Connect](https://openid.net/connect/). OpenID Connect (OIDC) est une norme de l’industrie utilisée par de nombreux fournisseurs d’identité. Vous n’avez pas besoin de comprendre les détails de la spécification pour configurer votre application de manière à utiliser un fournisseur d’identité conforme.

Vous pouvez configurer votre application pour qu’elle utilise un ou plusieurs fournisseurs OIDC. Chacun doit se voir attribuer un nom unique dans la configuration, et un seul peut servir de cible de redirection par défaut.

> [!CAUTION]
> L’activation d’un fournisseur OpenID Connect désactive la gestion de la fonctionnalité d’authentification/d’autorisation App Service pour votre application via certains clients, comme le portail Azure, Azure CLI et Azure PowerShell. La fonctionnalité s’appuie sur une nouvelle surface d’API qui, dans la préversion, n’est pas encore prise en compte dans toutes les expériences de gestion.

## <a name="register-your-application-with-the-identity-provider"></a><a name="register"> </a>Inscrire votre application auprès du fournisseur d’identité

Votre fournisseur vous demandera d’inscrire les détails de votre application. Veuillez consulter les instructions relatives à ce fournisseur. Vous devrez collecter un **ID client** et un **secret client** pour votre application.

> [!IMPORTANT]
> La clé secrète de l'application est une information d'identification de sécurité importante. Ne partagez cette clé secrète avec personne et ne la distribuez pas dans une application cliente.
>

> [!NOTE]
> Certains fournisseurs peuvent exiger des étapes supplémentaires pour leur configuration et l’utilisation des valeurs qu’ils fournissent. Par exemple, Apple fournit une clé privée qui n’est pas elle-même utilisée comme secret client OIDC. Vous devez plutôt l’utiliser pour créer un jeton JWT qui est traité comme le secret que vous fournissez dans la configuration de votre application (voir la section « Création du secret client » de la [documentation Connexion avec Apple](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens))
>

Ajoutez le secret client en tant que [paramètre d’application](./configure-common.md#configure-app-settings) pour l’application, en utilisant un nom de votre choix pour le paramètre. Prenez note de ce nom pour une utilisation ultérieure.

En outre, vous aurez besoin des métadonnées OpenID Connect pour le fournisseur. Celles-ci sont souvent exposées via un [document de métadonnées de configuration](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig), qui est l’URL d’émetteur du fournisseur avec le suffixe `/.well-known/openid-configuration`. Notez cette URL de configuration.

Si vous ne pouvez pas utiliser un document de métadonnées de configuration, vous devrez collecter les valeurs suivantes séparément :

- L’URL d’émetteur (parfois indiquée comme `issuer`)
- Le [point de terminaison d’autorisation OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-3.1) (parfois indiqué comme `authorization_endpoint`)
- Le [point de terminaison de jeton OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-3.2) (parfois indiqué comme `token_endpoint`)
- L’URL du document [Jeu de clés web OAuth 2.0 JSON](https://tools.ietf.org/html/rfc8414#section-2) (parfois indiqué comme `jwks_uri`)

## <a name="add-provider-information-to-your-application"></a><a name="configure"> </a>Ajouter des informations sur le fournisseur à votre application

> [!NOTE]
> La configuration requise est dans un nouveau format d’API, actuellement pris en charge uniquement par la [configuration basée sur fichier (préversion)](.\app-service-authentication-how-to.md#config-file). Vous devrez suivre les étapes ci-dessous à l’aide d’un fichier de ce type.

Cette section vous guide tout au long de la mise à jour de la configuration pour inclure votre nouveau fournisseur d’identité. Voici un exemple de configuration.

1. Dans l’objet `identityProviders`, ajoutez un objet `openIdConnectProviders` s’il n’en existe pas déjà un.
1. Dans l’objet `openIdConnectProviders`, ajoutez une clé pour votre nouveau fournisseur. Il s’agit d’un nom convivial utilisé pour référencer le fournisseur dans le reste de la configuration. Par exemple, si vous souhaitez exiger que toutes les demandes soient authentifiées auprès de ce fournisseur, vous devez définir `globalValidation.unauthenticatedClientAction` sur « RedirectToLoginPage » et définir `redirectToProvider` sur ce même nom convivial.
1. Attribuez un objet à cette clé avec un objet `registration` dans celui-ci, et éventuellement un objet `login` :
    
    ```json
    "myCustomIDP" : {
       "registration" : {},
       "login": {
             "nameClaimType": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name",
             "scope": [],
             "loginParameterNames": [],
       }
    }
    ```

1. Dans l’objet inscription, définissez `clientId` sur l’ID client que vous avez collecté, définissez `clientCredential.secretSettingName` sur le nom du paramètre d’application où vous avez stocké le secret client, puis créez un objet `openIdConnectConfiguration` :

    ```json
    "registration": {
      "clientId": "bd96cf8a-3f2b-4806-b180-d3c5fd11a2be",
      "clientCredential": {
         "secretSettingName": "IDP_CLIENT_SECRET"
      },
      "openIdConnectConfiguration" : {}
    }
    ```

1. Dans l’objet `openIdConnectConfiguration`, fournissez les métadonnées OpenID Connect que vous avez collectées précédemment. Il existe deux options pour cela, en fonction des informations que vous avez collectées :

    - Affectez à la propriété `wellKnownOpenIdConfiguration` l’URL des métadonnées de configuration que vous avez collectée précédemment.
    - Vous pouvez également définir les quatre valeurs individuelles collectées comme suit :
        - Définissez `issuer` sur l’URL d’émetteur
        - Définissez `authorizationEndpoint` sur le point de terminaison d’autorisation
        - Définissez `tokenEndpoint` sur le point de terminaison de jeton
        - Définissez `certificationUri` sur l’URL du document du jeu de clés web JSON

    Ces deux options sont mutuellement exclusives.

Une fois cette configuration définie, vous êtes prêt à utiliser votre fournisseur OpenID Connect pour l’authentification dans votre application.

Un exemple de configuration peut ressembler à ce qui suit (exemple utilisé : Connexion avec Apple, où le paramètre APPLE_GENERATED_CLIENT_SECRET pointe vers un JWT généré conformément à la [documentation Apple](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens)) :

```json
{
    "platform": {
        "enabled": true
    },
    "globalValidation": {
        "redirectToProvider": "apple",
        "unauthenticatedClientAction": "RedirectToLoginPage"
    },
    "identityProviders": {
        "openIdConnectProviders": {
            "apple": {
                "registration": {
                    "clientId": "com.contoso.example.client",
                    "clientCredential": {
                        "secretSettingName": "APPLE_GENERATED_CLIENT_SECRET"
                    },
                    "openIdConnectConfiguration": {
                        "wellKnownOpenIdConfiguration": "https://appleid.apple.com/.well-known/openid-configuration"
                    }
                },
                "login": {
                    "nameClaimType": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name",
                    "scope": [],
                    "loginParameterNames": []
                }
            }
        }
    },
    "login": {
        "tokenStore": {
            "enabled": true
        }
    }     
}
```

## <a name="next-steps"></a><a name="related-content"> </a>Étapes suivantes

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]
