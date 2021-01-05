---
title: API REST Azure Active Directory – Authentification
description: Utilisez Azure Active Directory pour vous authentifier auprès d’Azure App Configuration à l’aide de l’API REST.
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: cbf05245768a663e324e9bb6e1ad422eeee3ab1a
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96930515"
---
# <a name="azure-active-directory-authentication"></a>Authentification Azure Active Directory

Vous pouvez authentifier les requêtes HTTP suivant le schéma d’authentification `Bearer` avec un jeton obtenu auprès d’Azure Active Directory (Azure AD). Vous devez transmettre ces demandes via le protocole TLS (Transport Layer Security).

## <a name="prerequisites"></a>Prérequis

Vous devez affecter au principal utilisé pour demander un jeton Azure AD l’un des [rôles Azure App Configuration](./rest-api-authorization-azure-ad.md) applicables.

Fournissez à chaque demande tous les en-têtes HTTP requis pour l’authentification. Voici les prérequis :

|  En-tête de requête | Description  |
| --------------- | ------------ |
| `Authorization` | Informations d’authentification requises par le schéma `Bearer`. |

**Exemple :**

```http
Host: {myconfig}.azconfig.io
Authorization: Bearer {{AadToken}}
```

## <a name="azure-ad-token-acquisition"></a>Acquisition de jeton Azure AD

Pour pouvoir acquérir un jeton Azure AD, vous devez identifier l’utilisateur à utiliser pour l’authentification, l’audience pour laquelle vous demandez le jeton et le point de terminaison Azure AD (autorité) à employer.

### <a name="audience"></a>Public visé

Demandez le jeton Azure AD avec une audience appropriée. Pour Azure App Configuration, utilisez l’une des audiences suivantes. L’audience peut également être considérée comme la *ressource* pour laquelle est demandé le jeton.

- {configurationStoreName}.azconfig.io
- *.azconfig.io

> [!IMPORTANT]
> Lorsque l’audience demandée est `{configurationStoreName}.azconfig.io`, elle doit correspondre exactement à l’en-tête de demande `Host` (avec respect de la casse) utilisé pour envoyer la demande.

### <a name="azure-ad-authority"></a>Autorité Azure AD

L’autorité Azure AD est le point de terminaison utilisé pour acquérir un jeton Azure AD. Il est au format suivant `https://login.microsoftonline.com/{tenantId}`. Le segment `{tenantId}` fait référence à l’ID de locataire Azure AD auquel appartient l’utilisateur ou l’application qui tente de s’authentifier.

### <a name="authentication-libraries"></a>Bibliothèques d’authentification

Azure fournit un ensemble de bibliothèques, nommées Bibliothèques d’authentification Azure Active Directory, pour simplifier le processus d’acquisition d’un jeton Azure AD. Azure les crée pour plusieurs langages. Pour plus d’informations, consultez la [documentation](../active-directory/azuread-dev/active-directory-authentication-libraries.md).

## <a name="errors"></a>Erreurs

Vous risquez de rencontrer les erreurs suivantes.

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: HMAC-SHA256, Bearer
```

**Raison :** vous n’avez pas fourni l’en-tête de demande d’autorisation avec le schéma `Bearer`.

**Solution :** fournissez un en-tête de requête HTTP `Authorization` valide.

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: HMAC-SHA256, Bearer error="invalid_token", error_description="Authorization token failed validation"
```

**Raison :** le jeton Azure AD n’est pas valide.

**Solution :** récupérez un jeton Azure AD auprès de l’autorité Azure AD et vérifiez que vous avez utilisé l’audience appropriée.

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: HMAC-SHA256, Bearer error="invalid_token", error_description="The access token is from the wrong issuer. It must match the AD tenant associated with the subscription to which the configuration store belongs. If you just transferred your subscription and see this error message, please try back later."
```

**Raison :** le jeton Azure AD n’est pas valide.

**Solution :** récupérez un jeton Azure AD auprès de l’autorité Azure AD. Vérifiez que le locataire Azure AD est celui associé à l’abonnement auquel appartient le magasin de configuration. Cette erreur est susceptible de se produire si le principal appartient à plusieurs locataires Azure AD.