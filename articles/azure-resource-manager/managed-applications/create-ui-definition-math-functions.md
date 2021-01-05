---
title: Créer des fonctions mathématiques de définition de l’interface utilisateur
description: Décrit les fonctions à utiliser pour effectuer des opérations mathématiques.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: 022e59d847a4d89b4243223637fc6fd1e73255a9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87094186"
---
# <a name="createuidefinition-math-functions"></a>Fonctions mathématiques CreateUiDefinition

Les fonctions vous permettent d’effectuer des opérations mathématiques.

## <a name="add"></a>add

Ajoute deux nombres et retourne le résultat.

L’exemple suivant retourne `3`:

```json
"[add(1, 2)]"
```

## <a name="ceil"></a>ceil

Retourne le plus grand nombre entier qui est supérieur ou égal au nombre spécifié.

L’exemple suivant retourne `4`:

```json
"[ceil(3.14)]"
```

## <a name="div"></a>div

Divise le premier nombre par le second nombre et retourne le résultat. Le résultat est toujours un entier.

L’exemple suivant retourne `2`:

```json
"[div(6, 3)]"
```

## <a name="floor"></a>floor

Retourne le plus grand nombre entier qui est inférieur ou égal au nombre spécifié.

L’exemple suivant retourne `3`:

```json
"[floor(3.14)]"
```

## <a name="max"></a>max

Retourne le plus grand des deux nombres.

L’exemple suivant retourne `2`:

```json
"[max(1, 2)]"
```

## <a name="min"></a>min

Retourne le plus petit des deux nombres.

L’exemple suivant retourne `1`:

```json
"[min(1, 2)]"
```

## <a name="mod"></a>mod

Divise le premier nombre par le second nombre et retourne le reste.

L’exemple suivant retourne `0`:

```json
"[mod(6, 3)]"
```

L’exemple suivant retourne `2`:

```json
"[mod(6, 4)]"
```

## <a name="mul"></a>mul

Multiplie deux nombres et retourne le résultat.

L’exemple suivant retourne `6`:

```json
"[mul(2, 3)]"
```

## <a name="rand"></a>rand

Retourne un nombre entier aléatoire dans la plage spécifiée. Cette fonction ne génère pas de nombres aléatoires sécurisés par chiffrement.

L’exemple suivant peut retourner `42` :

```json
"[rand(-100, 100)]"
```

## <a name="range"></a>range

Génère une séquence de nombres entiers dans la plage spécifiée.

L’exemple suivant retourne `[1,2,3]`:

```json
"[range(1, 3)]"
```

## <a name="sub"></a>sub

Soustrait le second nombre du premier nombre et retourne le résultat.

L’exemple suivant retourne `1`:

```json
"[sub(3, 2)]"
```

## <a name="next-steps"></a>Étapes suivantes

* Pour découvrir Azure Resource Manager, consultez [Vue d’ensemble d’Azure Resource Manager](../management/overview.md).
