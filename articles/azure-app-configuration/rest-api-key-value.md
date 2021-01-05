---
title: API REST Azure App Configuration – Paire clé-valeur
description: Pages de référence pour l’utilisation de paires clé-valeur à l’aide de l’API REST Azure App Configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 95081d6f8c2770d01f7836e08b6851860bf47ba8
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932555"
---
# <a name="key-values"></a>Clés-valeurs

Une paire clé-valeur est une ressource identifiée par une combinaison unique de `key` + `label`. `label` est facultatif. Pour référencer explicitement une paire clé-valeur sans étiquette, utilisez « \0 » (URL encodée sous la forme ``%00``). Consultez les détails de chaque opération.

Cet article s’applique à la version 1.0 de l’API.

## <a name="operations"></a>Opérations

- Obtenir
- Liste multiple
- Définissez
- DELETE

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-rest-api-prereqs.md)]

## <a name="syntax"></a>Syntaxe

```json
{
  "etag": [string],
  "key": [string],
  "label": [string, optional],
  "content_type": [string, optional],
  "value": [string],
  "last_modified": [datetime ISO 8601],
  "locked": [boolean],
  "tags": [object with string properties, optional]
}
```

## <a name="get-key-value"></a>Obtenir une paire clé-valeur

Obligatoire : ``{key}``, ``{api-version}``  
Facultatif : ``label`` (Si omis, cela implique une paire clé-valeur sans étiquette.)

```http
GET /kv/{key}?label={label}&api-version={api-version}
```

**Réponses :**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kv+json; charset=utf-8;
Last-Modified: Tue, 05 Dec 2017 02:41:26 GMT
ETag: "4f6dd610dd5e4deebc7fbaef685fb903"
```

```json
{
  "etag": "4f6dd610dd5e4deebc7fbaef685fb903",
  "key": "{key}",
  "label": "{label}",
  "content_type": null,
  "value": "example value",
  "last_modified": "2017-12-05T02:41:26+00:00",
  "locked": "false",
  "tags": {
    "t1": "value1",
    "t2": "value2"
  }
}
```

Si la clé n’existe pas, la réponse suivante est retournée :

```http
HTTP/1.1 404 Not Found
```

## <a name="get-conditionally"></a>Obtenir (de manière conditionnelle)

Pour améliorer la mise en cache du client, utilisez les en-têtes de demande `If-Match` ou `If-None-Match`. L’argument `etag` fait partie de la représentation clé. Pour plus d’informations, consultez les [sections 14.24 et 14.26](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html).

La requête suivante récupère la paire clé-valeur uniquement si la représentation actuelle ne correspond pas à l’élément `etag`spécifié :

```http
GET /kv/{key}?api-version={api-version} HTTP/1.1
Accept: application/vnd.microsoft.appconfig.kv+json;
If-None-Match: "{etag}"
```

**Réponses :**

```http
HTTP/1.1 304 NotModified
```

ou

```http
HTTP/1.1 200 OK
```

## <a name="list-key-values"></a>Lister les paires clé-valeur

Facultatif : ``key`` (Si non spécifié, cela implique n’importe quelle clé.) Facultatif : ``label`` (Si non spécifié, cela implique n’importe quelle étiquette.)

```http
GET /kv?label=*&api-version={api-version} HTTP/1.1
```

**Réponse :**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kvset+json; charset=utf-8
```

Pour obtenir des options supplémentaires, consultez la section « Filtrage » plus loin dans cet article.

## <a name="pagination"></a>Pagination

Le résultat est paginé si le nombre d’éléments retournés dépasse le nombre limite de réponses. Suivez les en-têtes de réponse `Link` facultatifs et utilisez `rel="next"` pour la navigation.
Le contenu fournit également un lien suivant sous forme de la propriété `@nextLink`. L’URI lié comprend l’argument `api-version`.

```http
GET /kv?api-version={api-version} HTTP/1.1
```

**Réponse :**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kvs+json; charset=utf-8
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

Une combinaison de filtre `key` et `label` est prise en charge.
Utilisez les paramètres de chaîne de requête `key` et `label` facultatifs.

```http
GET /kv?key={key}&label={label}&api-version={api-version}
```

### <a name="supported-filters"></a>Filtres pris en charge

|Filtre de clé|Résultat|
|--|--|
|`key` est omis ou `key=*`|Correspond à **toute** clé|
|`key=abc`|Correspond à une clé nommée **abc**|
|`key=abc*`|Correspond aux noms de clé qui commencent par **abc**|
|`key=abc,xyz`|Correspond aux noms de clé **abc** ou **xyz** (limité à 5 CSV)|

|Filtre d’étiquette|Résultat|
|--|--|
|`label` est omis ou `label=*`|Correspond à **toute** étiquette|
|`label=%00`|Correspond à une paire clé-valeur sans étiquette|
|`label=prod`|Correspond à l’étiquette **prod**|
|`label=prod*`|Correspond aux étiquettes qui commencent par **prod**|
|`label=prod,test`|Correspond aux étiquettes **prod** ou **test** (limité à 5 CSV)|

**_Caractères réservés_* _

`_`, `\`, `,`

Si un caractère réservé fait partie de la valeur, il doit être placé dans une séquence d’échappement à l’aide de `\{Reserved Character}`. Les caractères non réservés peuvent également être placés dans une séquence d’échappement.

***Validation de filtre** _

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

_ *Exemples**

- Tous

    ```http
    GET /kv?api-version={api-version}
    ```

- Le nom de clé commence par **abc** et inclut toutes les étiquettes

    ```http
    GET /kv?key=abc*&label=*&api-version={api-version}
    ```

- Le nom de clé commence par **abc** et l’étiquette est égale à **v1** ou **v2**

    ```http
    GET /kv?key=abc*&label=v1,v2&api-version={api-version}
    ```

## <a name="request-specific-fields"></a>Champs spécifiques de la demande

Utilisez le paramètre de chaîne de requête facultatif `$select` et fournissez une liste séparée par des virgules des champs demandés. Si le paramètre `$select` est omis, la réponse contient l’ensemble par défaut.

```http
GET /kv?$select=key,value&api-version={api-version} HTTP/1.1
```

## <a name="time-based-access"></a>Accès basé sur le temps

Obtenez une représentation du résultat tel qu’il était à un moment passé. Pour plus d’informations, voir la section [2.1.1](https://tools.ietf.org/html/rfc7089#section-2.1). La pagination est toujours prise en charge, comme défini précédemment dans cet article.

```http
GET /kv?api-version={api-version} HTTP/1.1
Accept-Datetime: Sat, 12 May 2018 02:10:00 GMT
```

**Réponse :**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kvset+json"
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

## <a name="set-key"></a>Définir la clé

- Obligatoire : ``{key}``
- Facultatif : ``label`` (Si non spécifié ou si étiquette = %00, cela implique une paire clé-valeur sans étiquette.)

```http
PUT /kv/{key}?label={label}&api-version={api-version} HTTP/1.1
Content-Type: application/vnd.microsoft.appconfig.kv+json
```

```json
{
  "value": "example value",         // optional
  "content_type": "user defined",   // optional
  "tags": {                         // optional
    "tag1": "value1",
    "tag2": "value2",
  }
}
```

**Réponses :**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kv+json; charset=utf-8
Last-Modified: Tue, 05 Dec 2017 02:41:26 GMT
ETag: "4f6dd610dd5e4deebc7fbaef685fb903"
```

```json
{
  "etag": "4f6dd610dd5e4deebc7fbaef685fb903",
  "key": "{key}",
  "label": "{label}",
  "content_type": "user defined",
  "value": "example value",
  "last_modified": "2017-12-05T02:41:26.4874615+00:00",
  "tags": {
    "tag1": "value1",
    "tag2": "value2",
  }
}
```

Si l’élément est verrouillé, vous recevrez la réponse suivante :

```http
HTTP/1.1 409 Conflict
Content-Type: application/problem+json; charset="utf-8"
```

```json
{
    "type": "https://azconfig.io/errors/key-locked",
    "title": "Modifing key '{key}' is not allowed",
    "name": "{key}",
    "detail": "The key is read-only. To allow modification unlock it first.",
    "status": "409"
}
```

## <a name="set-key-conditionally"></a>Définir la clé (de manière conditionnelle)

Pour éviter les conditions de concurrence, utilisez les en-têtes de demande `If-Match` ou `If-None-Match`. L’argument `etag` fait partie de la représentation clé.
Si `If-Match` et `If-None-Match` sont omis, l’opération est inconditionnelle.

La réponse suivante met à jour la valeur uniquement si la représentation actuelle correspond à l’objet `etag` spécifié :

```http
PUT /kv/{key}?label={label}&api-version={api-version} HTTP/1.1
Content-Type: application/vnd.microsoft.appconfig.kv+json
If-Match: "4f6dd610dd5e4deebc7fbaef685fb903"
```

La réponse suivante met à jour la valeur uniquement si la représentation actuelle ne correspond pas à l’objet `etag` spécifié :

```http
PUT /kv/{key}?label={label}&api-version={api-version} HTTP/1.1
Content-Type: application/vnd.microsoft.appconfig.kv+json;
If-None-Match: "4f6dd610dd5e4deebc7fbaef685fb903"
```

La requête suivante ajoute la valeur uniquement si une représentation existe déjà :

```http
PUT /kv/{key}?label={label}&api-version={api-version} HTTP/1.1
Content-Type: application/vnd.microsoft.appconfig.kv+json;
If-Match: "*"
```

La requête suivante ajoute la valeur uniquement si une représentation n’existe pas déjà :

```http
PUT /kv/{key}?label={label}&api-version={api-version} HTTP/1.1
Content-Type: application/vnd.microsoft.appconfig.kv+json
If-None-Match: "*"
```

**Réponses**

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kv+json; charset=utf-8
...
```

ou

```http
HTTP/1.1 412 PreconditionFailed
```

## <a name="delete"></a>DELETE

- Obligatoire : `{key}`, `{api-version}`
- Facultatif : `{label}` (Si non spécifié ou si étiquette = %00, cela implique une paire clé-valeur sans étiquette.)

```http
DELETE /kv/{key}?label={label}&api-version={api-version} HTTP/1.1
```

**Réponse :** Retourne la paire clé-valeur supprimée, ou aucune si la paire clé-valeur n’existe pas.

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.microsoft.appconfig.kv+json; charset=utf-8
...
```

ou

```http
HTTP/1.1 204 No Content
```

## <a name="delete-key-conditionally"></a>Supprimer la clé (de manière conditionnelle)

Équivaut à la section « Définir la clé (de manière conditionnelle) » plus haut dans cet article.
