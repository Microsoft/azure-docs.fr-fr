---
title: API REST Azure App Configuration – Révisions clé-valeur
description: Pages de référence pour l’utilisation des révisions clé-valeur à l’aide de l’API REST Azure App Configuration.
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: cfa117d1ed017170c279b7c4e0a146ae4edac108
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932470"
---
# <a name="key-value-revisions"></a>Révisions clé-valeur

Une *révision clé-valeur* définit la représentation historique d’une ressource clé-valeur. Les révisions expirent au bout de 7 jours pour les magasins de niveau Gratuit ou 30 jours pour les magasins de niveau Standard. Les révisions prennent en charge l’opération `List`.

Pour toutes les opérations, ``key`` est un paramètre facultatif. En cas d’omission, il implique n’importe quelle clé.

Pour toutes les opérations, ``label`` est un paramètre facultatif. En cas d’omission, il implique n’importe quelle étiquette.

Cet article s’applique à la version 1.0 de l’API.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-rest-api-prereqs.md)]

## <a name="list-revisions"></a>Lister les révisions

```http
GET /revisions?label=*&api-version={api-version} HTTP/1.1
```

**Réponses :**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kvset+json; charset=utf-8"
Accept-Ranges: items
```

```json
{
    "items": [
        {
          "etag": "4f6dd610dd5e4deebc7fbaef685fb903",
          "key": "{key}",
          "label": "{label}",
          "content_type": null,
          "value": "example value",
          "last_modified": "2017-12-05T02:41:26.4874615+00:00",
          "tags": []
        },
        ...
    ],
    "@nextLink": "{relative uri}"
}
```

## <a name="pagination"></a>Pagination

Le résultat est paginé si le nombre d’éléments retournés dépasse le nombre limite de réponses. Suivez l’en-tête de réponse ``Link`` facultatif et utilisez ``rel="next"`` pour la navigation. Le contenu fournit également un lien suivant sous la forme de la propriété ``@nextLink``.

```http
GET /revisions?api-version={api-version} HTTP/1.1
```

**Réponse :**

```http
HTTP/1.1 OK
Content-Type: application/vnd.microsoft.appconfig.kvs+json; charset=utf-8
Accept-Ranges: items
Link: <{relative uri}>; rel="next"
```

```json
{
    "items": [
        ...
    ],
    "@nextLink": "{relative uri}"
}
```

## <a name="list-subset-of-revisions"></a>Répertorier le sous-ensemble des révisions

Utilisez l’en-tête de requête `Range`. La réponse contient un en-tête `Content-Range`. Si le serveur ne peut pas satisfaire la plage demandée, il répond HTTP `416` (`RangeNotSatisfiable`).

```http
GET /revisions?api-version={api-version} HTTP/1.1
Range: items=0-2
```

**Réponse**

```http
HTTP/1.1 206 Partial Content
Content-Type: application/vnd.microsoft.appconfig.revs+json; charset=utf-8
Content-Range: items 0-2/80
```

## <a name="filtering"></a>Filtrage

Une combinaison de filtre `key` et `label` est prise en charge.
Utilisez les paramètres de chaîne de requête `key` et `label` facultatifs.

```http
GET /revisions?key={key}&label={label}&api-version={api-version}
```

### <a name="supported-filters"></a>Filtres pris en charge

|Filtre de clé|Résultat|
|--|--|
|`key` est omis ou `key=*`|Correspond à **n’importe quelle** clé|
|`key=abc`|Correspond à une clé nommée **abc**|
|`key=abc*`|Correspond aux noms de clé qui commencent par **abc**|
|`key=*abc`|Correspond aux noms de clé qui se terminent par **abc**|
|`key=*abc*`|Correspond aux noms de clé qui contiennent **abc**|
|`key=abc,xyz`|Correspond aux noms de clé **abc** ou **xyz** (limités à 5 valeurs séparées par des virgules (CSV))|

|Filtre d’étiquette|Résultat|
|--|--|
|`label` est omis ou `label=`|Correspond à l’entrée sans étiquette|
|`label=*`|Correspond à **n’importe quelle** étiquette|
|`label=prod`|Correspond à l’étiquette **prod**|
|`label=prod*`|Correspond aux étiquettes qui commencent par **prod**|
|`label=*prod`|Correspond aux étiquettes qui se terminent par **prod**|
|`label=*prod*`|Correspond aux étiquettes qui contiennent **prod**|
|`label=prod,test`|Correspond aux étiquettes **prod** ou **test** (limité à 5 valeurs séparées par des virgules (CSV))|

### <a name="reserved-characters"></a>Caractères réservés

Voici les caractères réservés :

`*`, `\`, `,`

Si un caractère réservé fait partie de la valeur, il doit être placé dans une séquence d’échappement à l’aide de `\{Reserved Character}`. Les caractères non réservés peuvent également être placés dans une séquence d’échappement.

### <a name="filter-validation"></a>Validation de filtre

Si une erreur de validation de filtre se produit, la réponse est HTTP `400` avec les détails de l’erreur :

```http
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json; charset=utf-8
```

```json
{
  "type": "https://azconfig.io/errors/invalid-argument",
  "title": "Invalid request parameter '{filter}'",
  "name": "{filter}",
  "detail": "{filter}(2): Invalid character",
  "status": 400
}
```

### <a name="examples"></a>Exemples

- Tout :

    ```http
    GET /revisions
    ```

- Éléments dont le nom de clé commence par **abc** :

    ```http
    GET /revisions?key=abc*&api-version={api-version}
    ```

- Éléments dont le nom de clé est **abc** ou **xyz** et les étiquettes contiennent **prod** :

    ```http
    GET /revisions?key=abc,xyz&label=*prod*&api-version={api-version}
    ```

## <a name="request-specific-fields"></a>Champs spécifiques à la requête

Utilisez le paramètre de chaîne de requête facultatif `$select` et donnez la liste séparée par des virgules des champs demandés. Si le paramètre `$select` est omis, la réponse contient l’ensemble par défaut.

```http
GET /revisions?$select=value,label,last_modified&api-version={api-version} HTTP/1.1
```

## <a name="time-based-access"></a>Accès basé sur l’heure

Obtenez une représentation du résultat tel qu’il était à un moment passé. Pour plus d’informations, consultez [Framework HTTP pour l’accès basé sur l’heure aux états des ressources – Mémento](https://tools.ietf.org/html/rfc7089#section-2.1), section 2.1.1.

```http
GET /revisions?api-version={api-version} HTTP/1.1
Accept-Datetime: Sat, 12 May 2018 02:10:00 GMT
```

**Réponse :**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.revs+json"
Memento-Datetime: Sat, 12 May 2018 02:10:00 GMT
Link: <{relative uri}>; rel="original"
```

```json
{
    "items": [
        ....
    ]
}
```
