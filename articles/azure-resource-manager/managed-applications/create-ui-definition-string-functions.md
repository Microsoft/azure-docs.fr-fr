---
title: Créer des fonctions de chaîne de définition d’interface utilisateur
description: Décrit les fonctions de chaîne à utiliser lors de l’élaboration de définitions d’interface utilisateur pour Applications managées Azure
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: c662948542c36cd93f889ca045ee245c15c7bb11
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87094184"
---
# <a name="createuidefinition-string-functions"></a>Fonctions de chaîne CreateUiDefinition

Ces fonctions sont à utiliser avec des chaînes JSON.

## <a name="concat"></a>concat

Concatène une ou plusieurs chaînes.

Par exemple, si la valeur de sortie `element1` est `"Contoso"`, cet exemple retourne la chaîne `"Demo Contoso app"` :

```json
"[concat('Demo ', steps('step1').element1, ' app')]"
```

## <a name="endswith"></a>endsWith

Détermine si une chaîne se termine par une valeur.

L’exemple suivant retourne la valeur true.

```json
"[endsWith('tuvwxyz', 'xyz')]"
```

## <a name="guid"></a>guid

Génère une chaîne globale unique (GUID).

L’exemple suivant retourne une valeur telle que `"c7bc8bdc-7252-4a82-ba53-7c468679a511"` :

```json
"[guid()]"
```

## <a name="indexof"></a>indexOf

Retourne la première position d’une valeur dans une chaîne ou -1 si introuvable.

L’exemple suivant retourne 2.

```json
"[indexOf('abcdef', 'cd')]"
```

## <a name="lastindexof"></a>lastIndexOf

Retourne la dernière position d’une valeur dans une chaîne ou -1 si introuvable.

L’exemple suivant retourne 3.

```json
"[lastIndexOf('test', 't')]"
```

## <a name="replace"></a>remplacer

Retourne une chaîne dans laquelle toutes les occurrences de la chaîne spécifiée dans la chaîne actuelle sont remplacées par une autre chaîne.

L’exemple suivant retourne `"Contoso.com web app"`:

```json
"[replace('Contoso.net web app', '.net', '.com')]"
```

## <a name="startswith"></a>startsWith

Détermine si une chaîne commence par une valeur.

L’exemple suivant retourne la valeur true.

```json
"[startsWith('abcdefg', 'ab')]"
```

## <a name="substring"></a>substring

Retourne la sous-chaîne de la chaîne spécifiée. La sous-chaîne commence à l’index spécifié et présente la longueur spécifiée.

L’exemple suivant retourne `"web"`:

```json
"[substring('Contoso.com web app', 12, 3)]"
```

## <a name="tolower"></a>toLower

Retourne une chaîne convertie en minuscules.

L’exemple suivant retourne `"contoso"`:

```json
"[toLower('CONTOSO')]"
```

## <a name="toupper"></a>toUpper

Retourne une chaîne convertie en majuscules.

L’exemple suivant retourne `"CONTOSO"`:

```json
"[toUpper('contoso')]"
```

## <a name="next-steps"></a>Étapes suivantes

* Pour découvrir Azure Resource Manager, consultez [Vue d’ensemble d’Azure Resource Manager](../management/overview.md).

