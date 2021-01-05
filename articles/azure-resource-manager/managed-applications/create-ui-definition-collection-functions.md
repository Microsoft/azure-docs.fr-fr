---
title: Créer des fonctions de collection de définitions d’interface utilisateur
description: Décrit les fonctions à utiliser lors de l’utilisation de collections, telles que des tableaux et des objets.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: 2a075c5c99f457681cd49e75014487bf9cca263c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87094192"
---
# <a name="createuidefinition-collection-functions"></a>Fonctions de collection CreateUiDefinition

Ces fonctions peuvent être utilisées avec les collections, comme les chaînes JSON, les tableaux et les objets.

## <a name="contains"></a>contains

Retourne `true` si une chaîne contient la sous-chaîne spécifiée, un tableau contient la valeur spécifiée, ou un objet contient la clé spécifiée.

### <a name="example-string-contains"></a>Exemple : chaîne contains

L’exemple suivant retourne `true`:

```json
"[contains('webapp', 'web')]"
```

### <a name="example-array-contains"></a>Exemple : tableau contains

Supposons que `element1` retourne `[1, 2, 3]`. L’exemple suivant retourne `false`:

```json
"[contains(steps('demoStep').element1, 4)]"
```

### <a name="example-object-contains"></a>Exemple : objet contains

Supposons que `element1` retourne :

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

L’exemple suivant retourne `true`:

```json
"[contains(steps('demoStep').element1, 'key1')]"
```

## <a name="empty"></a>empty

Retourne `true` si la chaîne, le tableau ou l’objet est null ou vide.

### <a name="example-string-empty"></a>Exemple : chaîne empty

L’exemple suivant retourne `true`:

```json
"[empty('')]"
```

### <a name="example-array-empty"></a>Exemple : tableau empty

Supposons que `element1` retourne `[1, 2, 3]`. L’exemple suivant retourne `false`:

```json
"[empty(steps('demoStep').element1)]"
```

### <a name="example-object-empty"></a>Exemple : objet empty

Supposons que `element1` retourne :

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

L’exemple suivant retourne `false`:

```json
"[empty(steps('demoStep').element1)]"
```

### <a name="example-null-and-undefined"></a>Exemple 4 : null et non défini

Supposons que `element1` est `null` ou n’est pas défini. L’exemple suivant retourne `true`:

```json
"[empty(steps('demoStep').element1)]"
```

## <a name="filter"></a>Filter

Retourne un nouveau tableau après l’application de la logique de filtrage fournie en tant que fonction lambda. Le premier paramètre est le tableau à utiliser pour le filtrage. Le deuxième paramètre est la fonction lambda qui spécifie la logique de filtrage.

L’exemple suivant retourne le tableau `[ { "name": "abc" } ]`.

```json
"[filter(parse('[{\"name\":\"abc\"},{\"name\":\"xyz\"}]'), (item) => contains(item.name, 'abc'))]"
```

## <a name="first"></a>first

Retourne le premier caractère de la chaîne spécifiée, la première valeur du tableau spécifié, ou la première clé et la valeur de l’objet spécifié.

### <a name="example-string-first"></a>Exemple : chaîne first

L’exemple suivant retourne `"c"`:

```json
"[first('contoso')]"
```

### <a name="example-array-first"></a>Exemple : tableau first

Supposons que `element1` retourne `[1, 2, 3]`. L’exemple suivant retourne `1`:

```json
"[first(steps('demoStep').element1)]"
```

#### <a name="example-object-first"></a>Exemple : objet first

Supposons que `element1` retourne :

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

L’exemple suivant retourne `{"key1": "Linux"}`:

```json
"[first(steps('demoStep').element1)]"
```

## <a name="last"></a>last

Retourne le dernier caractère de la chaîne spécifiée, la dernière valeur du tableau spécifié, ou la dernière clé et la valeur de l’objet spécifié.

### <a name="example-string-last"></a>Exemple : chaîne last

L’exemple suivant retourne `"o"`:

```json
"[last('contoso')]"
```

### <a name="example-array-last"></a>Exemple : tableau last

Supposons que `element1` retourne `[1, 2, 3]`. L’exemple suivant retourne `3`:

```json
"[last(steps('demoStep').element1)]"
```

### <a name="example-object-last"></a>Exemple : objet last

Supposons que `element1` retourne :

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

L’exemple suivant retourne `{"key2": "Windows"}`:

```json
"[last(steps('demoStep').element1)]"
```

## <a name="length"></a>length

Retourne le nombre de caractères dans une chaîne, le nombre de valeurs dans un tableau, ou le nombre de clés dans un objet.

### <a name="example-string-length"></a>Exemple : chaîne length

L’exemple suivant retourne `7`:

```json
"[length('Contoso')]"
```

### <a name="example-array-length"></a>Exemple : tableau length

Supposons que `element1` retourne `[1, 2, 3]`. L’exemple suivant retourne `3`:

```json
"[length(steps('demoStep').element1)]"
```

### <a name="example-object-length"></a>Exemple : objet length

Supposons que `element1` retourne :

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

L’exemple suivant retourne `2`:

```json
"[length(steps('demoStep').element1)]"
```

## <a name="map"></a>map

Retourne un nouveau tableau après l’appel d’une fonction lambda sur un tableau fourni. Le premier paramètre est le tableau à utiliser pour la fonction lambda. Le deuxième paramètre est la fonction lambda.

L’exemple suivant retourne un nouveau tableau avec chaque valeur doublée. Le résultat est `[2, 4, 6]`.

```json
"[map(parse('[1, 2, 3]'), (item) => mul(2, item))]"
```

L’exemple suivant retourne un nouveau tableau `["abc", "xyz"]`.

```json
"[map(parse('[{\"name\":\"abc\"},{\"name\":\"xyz\"}]'), (item) => item.name)]"
```

## <a name="skip"></a>skip

Ignore un nombre spécifié d’éléments dans une collection, puis retourne les éléments restants.

### <a name="example-string-skip"></a>Exemple : chaîne skip

L’exemple suivant retourne `"app"`:

```json
"[skip('webapp', 3)]"
```

### <a name="example-array-skip"></a>Exemple : tableau skip

Supposons que `element1` retourne `[1, 2, 3]`. L’exemple suivant retourne `[3]`:

```json
"[skip(steps('demoStep').element1, 2)]"
```

### <a name="example-object-skip"></a>Exemple : objet skip

Supposons que `element1` retourne :

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```
L’exemple suivant retourne `{"key2": "Windows"}`:

```json
"[skip(steps('demoStep').element1, 1)]"
```

## <a name="split"></a>split

Retourne un tableau de chaînes contenant les sous-chaînes de l’entrée délimitée par le séparateur.

L’exemple suivant retourne le tableau `[ "555", "867", "5309" ]`.

```json
"[split('555-867-5309', '-')]"
```

## <a name="take"></a>take

Retourne un nombre spécifié de caractères contigus à partir du début de la chaîne, un nombre spécifié de valeurs contiguës à partir du début du tableau, ou un nombre spécifié de valeurs et clés contiguës à partir du début de l’objet.

### <a name="example-string-take"></a>Exemple : chaîne take

L’exemple suivant retourne `"web"`:

```json
"[take('webapp', 3)]"
```

### <a name="example-array-take"></a>Exemple : tableau take

Supposons que `element1` retourne `[1, 2, 3]`. L’exemple suivant retourne `[1, 2]`:

```json
"[take(steps('demoStep').element1, 2)]"
```

### <a name="example-object-take"></a>Exemple : objet take

Supposons que `element1` retourne :

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

L’exemple suivant retourne `{"key1": "Linux"}`:

```json
"[take(steps('demoStep').element1, 1)]"
```

## <a name="next-steps"></a>Étapes suivantes

* Pour découvrir Azure Resource Manager, consultez [Vue d’ensemble d’Azure Resource Manager](../management/overview.md).
