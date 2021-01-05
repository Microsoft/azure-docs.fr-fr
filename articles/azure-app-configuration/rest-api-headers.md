---
title: API REST Azure App Configuration - En-têtes
description: Pages de référence pour les en-têtes utilisés avec l’API REST Azure App Configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 80b20b0b55219766872166685c0b1257f3c39c55
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932572"
---
# <a name="headers"></a>En-têtes

Cet article fournit des liens vers des pages de référence pour les en-têtes utilisés avec l’API REST Azure App Configuration.

## <a name="request-headers"></a>En-têtes de requête

Le tableau suivant décrit les en-têtes de requête courants utilisés dans Azure App Configuration.

| En-tête | Description | Exemple |
|--|--|--|
| **Autorisation** | Utilisé pour [authentifier](./rest-api-authentication-index.md) une requête au service. Voir la [section 14.8](https://tools.ietf.org/html/rfc2616#section-14.8) | `Authorization: HMAC-SHA256 Credential=<Credential>&SignedHeaders=Host;x-ms-date;x-ms-content-sha256&Signature=<Signature>` |
| **Accepter** | Indique au serveur le type de média que le client acceptera dans une réponse HTTP. Voir la [section 14.1](https://tools.ietf.org/html/rfc2616#section-14.1) | `Accept: application/vnd.microsoft.appconfig.kv+json;` |
| **Accept-Datetime** | Demande au serveur de retourner son contenu en tant que représentation de son état antérieur. La valeur de cet en-tête est la valeur DateHeure demandée de cet état. Voir [RFC 7089](https://tools.ietf.org/html/rfc7089#section-2.1.1) | `Accept-Datetime: Sat, 12 May 2018 02:10:00 GMT` |
| **Content-Type** | Contient le type de média du contenu dans le corps de la requête HTTP. Voir la [section 14.17](https://tools.ietf.org/html/rfc2616#section-14.17) | `Content-Type: application/vnd.microsoft.appconfig.kv+json; charset=utf-8;` |
| **Date** | Date et heure auxquelles la requête HTTP a été émise. Cet en-tête est utilisé dans l’[authentification HMAC](./rest-api-authentication-hmac.md). Voir la [section 14.18](https://tools.ietf.org/html/rfc2616#section-14.18) | `Date: Fri, 11 May 2018 18:48:36 GMT` |
| **Hôte** | Spécifie le locataire pour lequel la requête a été émise. Cet en-tête est utilisé dans l’[authentification HMAC](./rest-api-authentication-hmac.md). Voir la [section 14.23](https://tools.ietf.org/html/rfc2616#section-14.23) | `Host: contoso.azconfig.io` |
| **If-Match** | Utilisé pour rendre une requête HTTP conditionnelle. Cette requête ne devrait aboutir que si l’ETag de la ressource ciblée correspond à la valeur de cet en-tête. La valeur '*' correspond à n’importe quel ETag. Voir la [section 14.24](https://tools.ietf.org/html/rfc2616#section-14.24) | `If-Match: "4f6dd610dd5e4deebc7fbaef685fb903"` |
| **If-None-Match** | Utilisé pour rendre une requête HTTP conditionnelle. Cette requête ne devrait aboutir que si l’ETag de la ressource ciblée ne correspond pas à la valeur de cet en-tête. La valeur '*' correspond à n’importe quel ETag. Voir la [section 14.26](https://tools.ietf.org/html/rfc2616#section-14.26) | `If-None-Match: "4f6dd610dd5e4deebc7fbaef685fb903"` |
| **Sync-Token** | Utilisé pour activer la cohérence en temps réel pendant une séquence de requêtes. | `Sync-Token: jtqGc1I4=MDoyOA==;sn=28` |
| **x-ms-client-request-id** | ID unique fourni par le client et utilisé pour suivre l’aller-retour d’une requête. | `x-ms-client-request-id: 00000000-0000-0000-0000-000000000000` |
| **x-ms-content-sha256** | Un condensé sha256 du corps de la requête HTTP. Cet en-tête est utilisé dans l’[authentification HMAC](./rest-api-authentication-hmac.md). | `x-ms-content-sha256: 47DEQpj8HBSa+/TImW+5JCeuQeRkm5NMpJWZG3hSuFU=` |
| **x-ms-date** | Cet en-tête peut être défini et utilisé à la place de l’en-tête `Date` si l’en-tête de date n’est pas accessible. Cet en-tête est utilisé dans l’[authentification HMAC](./rest-api-authentication-hmac.md). | `x-ms-date: Fri, 11 May 2018 18:48:36 GMT` |
| **x-ms-return-client-request-id** | Utilisé conjointement avec l’en-tête `x-ms-client-request-id`. Si la valeur de cet en-tête est « true », le serveur est invité à retourner la valeur de l’en-tête de demande `x-ms-client-request-id`. | `x-ms-return-client-request-id: true` |

## <a name="response-headers"></a>En-têtes de réponse

Le serveur peut inclure les en-têtes HTTP suivants dans ses réponses.

| En-tête | Description | Exemple |
|--|--|--|
| **Content-Type** | Contient le type de média du contenu dans le corps de la réponse HTTP. Voir la [section 14.17](https://tools.ietf.org/html/rfc2616#section-14.17) | `Content-Type: application/vnd.microsoft.appconfig.kv+json; charset=utf-8;` |
| **ETag** | Jeton opaque représentant l’état d’une ressource donnée. Peut être utilisé dans des opérations conditionnelles. Voir la [section 14.19](https://tools.ietf.org/html/rfc2616#section-14.19) | `ETag: "4f6dd610dd5e4deebc7fbaef685fb903"` |
| **Last-Modified** | Décrit le moment où la ressource demandée a été modifiée pour la dernière fois. Présente le format [HTTP-Date](https://tools.ietf.org/html/rfc2616#section-3.3.1). Voir la [section 14.29](https://tools.ietf.org/html/rfc2616#section-14.29) | `Last-Modified: Tue, 05 Dec 2017 02:41:26 GMT` |
| **Lien** | Fournit des liens vers les ressources associées à la réponse. Cet en-tête est utilisé pour la pagination à l’aide du lien _suivant_. Voir [RFC 5988](https://tools.ietf.org/html/rfc5988) | `Link: </kv?after={token}>; rel="next"` |
| **Memento-Datetime** | Indique que le contenu d’une réponse représente un état antérieur. La valeur de cet en-tête est la valeur DateHeure de cet état. Voir [RFC 7089](https://tools.ietf.org/html/rfc7089#section-2.1.1) | `Memento-Datetime: Sat, 12 May 2018 02:10:00 GMT` |
| **retry-after-ms** | Fournit une période suggérée (en millisecondes) pendant laquelle le client doit attendre avant de retenter une requête ayant échoué. | `retry-after-ms: 10` |
| **x-ms-request-id** | ID unique généré par le serveur et utilisé pour effectuer le suivi de la requête au sein du service. | `x-ms-request-id: 00000000-0000-0000-0000-000000000000` |
| **WWW-Authenticate** | Utilisé pour inviter les clients à s’authentifier et pour fournir une raison expliquant pourquoi une tentative d’authentification a échoué. Voir la [section 14.47](https://tools.ietf.org/html/rfc2616#section-14.47) | `WWW-Authenticate: HMAC-SHA256 error="invalid_token" error_description="Invalid Signature"` |
