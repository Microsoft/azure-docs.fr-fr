---
title: Créer des fonctions de conversion de définition de l’interface utilisateur
description: Décrit les fonctions à utiliser pour convertir des valeurs entre des types de données et des encodages.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: b69cd35b27b343da08727b4c4ee9b4fd025e1df7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87094191"
---
# <a name="createuidefinition-conversion-functions"></a>Fonctions de conversion CreateUiDefinition

Ces fonctions peuvent être utilisées pour convertir des valeurs entre des types de données JSON et des encodages.

## <a name="bool"></a>bool

Convertit le paramètre en valeur booléenne. Cette fonction prend en charge les paramètres de type nombre, chaîne et booléen. Tout comme les booléens en JavaScript, toutes les valeurs, à l’exception de `0` ou `'false'`, retournent `true`.

L’exemple suivant retourne `true`:

```json
"[bool(1)]"
```

L’exemple suivant retourne `false`:

```json
"[bool(0)]"
```

L’exemple suivant retourne `true`:

```json
"[bool(true)]"
```

L’exemple suivant retourne `true`:

```json
"[bool('true')]"
```

## <a name="decodebase64"></a>decodeBase64

Encode le paramètre à partir d’une chaîne codée en base 64. Cette fonction prend en charge les paramètres de type chaîne uniquement.

L’exemple suivant retourne `"Contoso"`:

```json
"[decodeBase64('Q29udG9zbw==')]"
```

## <a name="decodeuricomponent"></a>decodeUriComponent

Encode le paramètre à partir d’une chaîne d’URL encodée. Cette fonction prend en charge les paramètres de type chaîne uniquement.

L’exemple suivant retourne `"https://portal.azure.com/"`:

```json
"[decodeUriComponent('https%3A%2F%2Fportal.azure.com%2F')]"
```

## <a name="encodebase64"></a>encodeBase64

Encode le paramètre vers une chaîne codée en base 64. Cette fonction prend en charge les paramètres de type chaîne uniquement.

L’exemple suivant retourne `"Q29udG9zbw=="`:

```json
"[encodeBase64('Contoso')]"
```

## <a name="encodeuricomponent"></a>encodeUriComponent

Encode le paramètre vers une chaîne d’URL encodée. Cette fonction prend en charge les paramètres de type chaîne uniquement.

L’exemple suivant retourne `"https%3A%2F%2Fportal.azure.com%2F"`:

```json
"[encodeUriComponent('https://portal.azure.com/')]"
```

## <a name="float"></a>float

Convertit le paramètre en nombre à virgule flottante. Cette fonction prend en charge les paramètres de type nombre et chaîne.

L’exemple suivant retourne `1.0`:

```json
"[float('1.0')]"
```

L’exemple suivant retourne `2.9`:

```json
"[float(2.9)]"
```

## <a name="int"></a>int

Convertit le paramètre en entier. Cette fonction prend en charge les paramètres de type nombre et chaîne.

L’exemple suivant retourne `1`:

```json
"[int('1')]"
```

L’exemple suivant retourne `2`:

```json
"[int(2.9)]"
```

## <a name="parse"></a>parse

Convertit le paramètre en type natif. En d’autres termes, cette fonction est l’inverse de `string()`. Cette fonction prend en charge les paramètres de type chaîne uniquement.

L’exemple suivant retourne `1`:

```json
"[parse('1')]"
```

L’exemple suivant retourne `true`:

```json
"[parse('true')]"
```

L’exemple suivant retourne `[1,2,3]`:

```json
"[parse('[1,2,3]')]"
```

L’exemple suivant retourne `{"type":"webapp"}`:

```json
"[parse('{\"type\":\"webapp\"}')]"
```

## <a name="string"></a>string

Convertit le paramètre en chaîne. Cette fonction prend en charge les paramètres de tous les types de données JSON.

L’exemple suivant retourne `"1"`:

```json
"[string(1)]"
```

L’exemple suivant retourne `"2.9"`:

```json
"[string(2.9)]"
```

L’exemple suivant retourne `"[1,2,3]"`:

```json
"[string([1,2,3])]"
```

L’exemple suivant retourne `"{"type":"webapp"}"`:

```json
"[string({\"type\":\"webapp\"})]"
```

## <a name="next-steps"></a>Étapes suivantes

* Pour découvrir Azure Resource Manager, consultez [Vue d’ensemble d’Azure Resource Manager](../management/overview.md).
