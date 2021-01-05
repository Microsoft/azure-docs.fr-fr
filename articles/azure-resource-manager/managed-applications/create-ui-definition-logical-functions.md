---
title: Créer des fonctions logiques de définition de l’interface utilisateur
description: Décrit les fonctions permettant d’effectuer des opérations logiques.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: 00d2f0eeb5d353c8ebd7ad30f6866f890d6cb42e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87094188"
---
# <a name="createuidefinition-logical-functions"></a>Fonctions logiques CreateUiDefinition

Ces fonctions peuvent être utilisées dans des expressions conditionnelles. Certaines fonctions ne peuvent pas prendre en charge tous les types de données JSON.

## <a name="and"></a>and

Retourne `true` si tous les paramètres correspondent à `true`. Cette fonction prend en charge plusieurs paramètres de type booléen uniquement.

L’exemple suivant retourne `true`:

```json
"[and(equals(0, 0), equals('web', 'web'), less(1, 2))]"
```

L’exemple suivant retourne `false`:

```json
"[and(equals(0, 0), greater(1, 2))]"
```

## <a name="coalesce"></a>coalesce

Retourne la valeur du premier paramètre non null. Cette fonction prend en charge tous les types de données JSON.

Supposons que `element1` et `element2` ne soient pas définis. L’exemple suivant retourne `"Contoso"`:

```json
"[coalesce(steps('demoStep').element1, steps('demoStep').element2, 'Contoso')]"
```

Cette fonction est particulièrement utile dans le contexte d’un appel facultatif qui fait suite à une action de l’utilisateur après le chargement de la page. Par exemple, si les contraintes placées sur un champ de l’interface utilisateur dépendent de la valeur sélectionnée d’un autre champ **initialement non visible**. Dans ce cas, `coalesce()` peut être utilisé pour permettre à la fonction d’être syntaxiquement valide au moment du chargement de la page tout en produisant l’effet souhaité lorsque l’utilisateur interagit avec le champ.

Observez ce `DropDown`, qui permet à l’utilisateur de choisir entre différents types de base de données :

```
{
    "name": "databaseType",
    "type": "Microsoft.Common.DropDown",
    "label": "Choose database type",
    "toolTip": "Choose database type",
    "defaultValue": "Oracle Database",
    "visible": "[bool(steps('section_database').connectToDatabase)]"
    "constraints": {
        "allowedValues": [
            {
                "label": "Azure Database for PostgreSQL",
                "value": "postgresql"
            },
            {
                "label": "Oracle Database",
                "value": "oracle"
            },
            {
                "label": "Azure SQL",
                "value": "sqlserver"
            }
        ],
        "required": true
    },
```

Pour conditionner l’action d’un autre champ sur la valeur choisie actuelle de ce champ, utilisez `coalesce()`, comme dans l’exemple ci-dessous :

```
"regex": "[concat('^jdbc:', coalesce(steps('section_database').databaseConnectionInfo.databaseType, ''), '.*$')]",
```

Cela est nécessaire, car le `databaseType` est initialement non visible et n’a donc pas de valeur. L’expression tout entière ne peut alors pas évaluer correctement.

## <a name="equals"></a>equals

Retourne `true` si les deux paramètres ont le même type et la même valeur. Cette fonction prend en charge tous les types de données JSON.

L’exemple suivant retourne `true`:

```json
"[equals(0, 0)]"
```

L’exemple suivant retourne `true`:

```json
"[equals('web', 'web')]"
```

L’exemple suivant retourne `false`:

```json
"[equals('abc', ['a', 'b', 'c'])]"
```

## <a name="greater"></a>greater

Retourne `true` si le premier paramètre est strictement supérieur au second paramètre. Cette fonction prend en charge les paramètres de type nombre et chaîne uniquement.

L’exemple suivant retourne `false`:

```json
"[greater(1, 2)]"
```

L’exemple suivant retourne `true`:

```json
"[greater('9', '10')]"
```

## <a name="greaterorequals"></a>greaterOrEquals

Retourne `true` si le premier paramètre est supérieur ou égal au second paramètre. Cette fonction prend en charge les paramètres de type nombre et chaîne uniquement.

L’exemple suivant retourne `true`:

```json
"[greaterOrEquals(2, 2)]"
```

## <a name="if"></a>if

Retourne une valeur indiquant si une condition est true ou false. Le premier paramètre est la condition à tester. Le deuxième paramètre est la valeur à retourner si la condition est vérifiée (true). Le troisième paramètre est la valeur à retourner si la condition n’est pas vérifiée (false).

L’exemple suivant retourne `yes`.

```json
"[if(equals(42, mul(6, 7)), 'yes', 'no')]"
```

## <a name="less"></a>less

Retourne `true` si le premier paramètre est strictement inférieur au second paramètre. Cette fonction prend en charge les paramètres de type nombre et chaîne uniquement.

L’exemple suivant retourne `true`:

```json
"[less(1, 2)]"
```

L’exemple suivant retourne `false`:

```json
"[less('9', '10')]"
```

## <a name="lessorequals"></a>lessOrEquals

Retourne `true` si le premier paramètre est inférieur ou égal au second paramètre. Cette fonction prend en charge les paramètres de type nombre et chaîne uniquement.

L’exemple suivant retourne `true`:

```json
"[lessOrEquals(2, 2)]"
```

## <a name="not"></a>not

Retourne `true` si le paramètre correspond à `false`. Cette fonction prend en charge les paramètres de type booléen uniquement.

L’exemple suivant retourne `true`:

```json
"[not(false)]"
```

L’exemple suivant retourne `false`:

```json
"[not(equals(0, 0))]"
```

## <a name="or"></a>or

Retourne `true` si au moins l’un des paramètres correspond à `true`. Cette fonction prend en charge plusieurs paramètres de type booléen uniquement.

L’exemple suivant retourne `true`:

```json
"[or(equals(0, 0), equals('web', 'web'), less(1, 2))]"
```

L’exemple suivant retourne `true`:

```json
"[or(equals(0, 0), greater(1, 2))]"
```

## <a name="next-steps"></a>Étapes suivantes

* Pour découvrir Azure Resource Manager, consultez [Vue d’ensemble d’Azure Resource Manager](../management/overview.md).
