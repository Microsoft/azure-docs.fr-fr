---
title: API REST Azure App Configuration – Clés
description: Pages de référence pour l’utilisation de clés à l’aide de l’API REST Azure App Configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 65ca190d7fbd6d8d4df473fbe2112eafbd031fde
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932538"
---
# <a name="keys"></a>Keys

api-version : 1.0

La syntaxe suivante représente une ressource de clé :

```http
{
    "name": [string]             // Name of the key
}
```

## <a name="operations"></a>Opérations

Les ressources de clé prennent en charge l’opération suivante :

- List

Pour toutes les opérations, `name` est un paramètre de filtre facultatif. Son omission implique *toute* clé.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-rest-api-prereqs.md)]

## <a name="list-keys"></a>Afficher la liste des clés

```http
GET /keys?api-version={api-version} HTTP/1.1
```

**Réponses :**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.keyset+json; charset=utf-8"
```

```json
{
    "items": [
        {
          "name": "{key-name}"
        },
        ...
    ],
    "@nextLink": "{relative uri}"
}
```

## <a name="pagination"></a>Pagination

Le résultat est paginé si le nombre d’éléments retournés dépasse le nombre limite de réponses. Suivez les en-têtes de réponse `Link` facultatifs et utilisez `rel="next"` pour la navigation. Le contenu fournit également un lien suivant sous la forme de la propriété `@nextLink`. Le lien suivant contient le paramètre `api-version`.

```http
GET /keys?api-version={api-version} HTTP/1.1
```

**Réponse :**

```http
HTTP/1.1 OK
Content-Type: application/vnd.microsoft.appconfig.keyset+json; charset=utf-8
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

## <a name="filtering"></a>Filtrage

Le filtrage par ```name``` est pris en charge.

```http
GET /keys?name={key-name}&api-version={api-version}
```

Les filtres suivants sont pris en charge :

|Filtre de clé|Résultat|
|--|--|
|`name` est omis ou `name=*`|Correspond à **toute** clé|
|`name=abc`|Correspond à une clé nommée **abc**|
|`name=abc*`|Correspond aux noms de clé qui commencent par **abc**|
|`name=abc,xyz`|Correspond aux noms de clé **abc** ou **xyz** (limités à 5 valeurs séparées par des virgules)|

Les caractères suivants sont réservés : `*`, `\`, `,`

Si un caractère réservé fait partie de la valeur, il doit être placé dans une séquence d’échappement à l’aide de `\{Reserved Character}`. Les caractères non réservés peuvent également être placés dans une séquence d’échappement.

## <a name="filter-validation"></a>Validation de filtre

Si une erreur de validation de filtre se produit, la réponse est HTTP `400` avec les détails de l’erreur :

```http
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json; charset=utf-8
```

```json
{
  "type": "https://azconfig.io/errors/invalid-argument",
  "title": "Invalid request parameter 'name'",
  "name": "name",
  "detail": "name(2): Invalid character",
  "status": 400
}
```

## <a name="examples"></a>Exemples

- Tous

    ```http
    GET /keys?api-version={api-version}
    ```

- Le nom de clé commence par **abc**

    ```http
    GET  /keys?name=abc*&api-version={api-version}
    ```

- Le nom de clé est **abc** ou **xyz**

    ```http
    GET /keys?name=abc,xyz&api-version={api-version}
    ```

## <a name="request-specific-fields"></a>Champs spécifiques de la demande

Utilisez le paramètre de chaîne de requête facultatif `$select` et fournissez une liste séparée par des virgules des champs demandés. Si le paramètre `$select` est omis, la réponse contient l’ensemble par défaut.

```http
GET /keys?$select=name&api-version={api-version} HTTP/1.1
```

## <a name="time-based-access"></a>Accès basé sur le temps

Obtenez une représentation du résultat tel qu’il était à un moment passé. Voir la section [2.1.1](https://tools.ietf.org/html/rfc7089#section-2.1)

```http
GET /keys&api-version={api-version} HTTP/1.1
Accept-Datetime: Sat, 12 May 2018 02:10:00 GMT
```

**Réponse :**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.keyset+json"
Memento-Datetime: Sat, 12 May 2018 02:10:00 GMT
Link: <relative uri>; rel="original"
```

```json
{
    "items": [
        ....
    ]
}
```
