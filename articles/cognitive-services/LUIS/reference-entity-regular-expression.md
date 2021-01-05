---
title: Type d’entité d’expression régulière - LUIS
description: Une expression régulière est préférable à un texte d’énoncé brut. Elle ignore la casse et la variante culturelle.  La correspondance d’expression régulière est appliquée après les modifications de la vérification orthographique au niveau du caractère, et non au niveau du jeton.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 04/14/2020
ms.openlocfilehash: 18e44ec43e1169aa054e6e5b4591ccd8611a7f4d
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025223"
---
# <a name="regular-expression-entity"></a>Entité d’expression régulière

Une entité d’expression régulière extrait une entité en fonction du modèle d’expression régulière que vous fournissez.

Une expression régulière est préférable à un texte d’énoncé brut. Elle ignore la casse et la variante culturelle.  La correspondance d’expression régulière est appliquée après les modifications de la vérification orthographique au niveau du caractère, et non au niveau du jeton. Si l’expression régulière est trop complexe (par exemple, si elle utilise de nombreux crochets), vous ne pouvez pas l’ajouter au modèle. Utilise une partie seulement de la bibliothèque [.Net Regex](/dotnet/standard/base-types/regular-expressions).

**L’entité convient bien quand :**

* Les données sont constamment mis en forme avec toute variation également cohérente.
* L’expression régulière n’a pas besoin de plus de 2 niveaux d’imbrication.

![Entité d’expression régulière](./media/luis-concept-entities/regex-entity.png)

## <a name="usage-considerations"></a>Considérations sur l’utilisation

Les expressions régulières peuvent correspondre à plus que ce à quoi vous vous attendez. La correspondance de mots numériques comme `one` et `two` en est un exemple. L’expression régulière suivante, qui fait correspondre le nombre `one` avec d’autres nombres, en est un exemple :

```javascript
(plus )?(zero|one|two|three|four|five|six|seven|eight|nine)(\s+(zero|one|two|three|four|five|six|seven|eight|nine))*
```

Cette expression régulière fait également correspondre les mots qui se terminent par ces nombres, comme `phone`. Pour résoudre les problèmes de ce type, veillez à ce que les correspondances des expressions régulières prennent en compte les limites des mots. L’expression régulière pour utiliser les limites des mots pour cet exemple est utilisée dans l’expression régulière suivante :

```javascript
\b(plus )?(zero|one|two|three|four|five|six|seven|eight|nine)(\s+(zero|one|two|three|four|five|six|seven|eight|nine))*\b
```

### <a name="example-json"></a>Exemple JSON

Dans le cas de l’utilisation de `kb[0-9]{6}` comme définition de l’entité d’expression régulière, la réponse JSON suivante est un exemple d’énoncé avec les entités d’expression régulière retournées pour la requête :

`When was kb123456 published?`:

#### <a name="v2-prediction-endpoint-response"></a>[Réponse de point de terminaison de prédiction V2](#tab/V2)

```JSON
"entities": [
  {
    "entity": "kb123456",
    "type": "KB number",
    "startIndex": 9,
    "endIndex": 16
  }
]
```


#### <a name="v3-prediction-endpoint-response"></a>[Réponse de point de terminaison de prédiction V3](#tab/V3)


Il s’agit du JSON si `verbose=false` est défini dans la chaîne de requête :

```json
"entities": {
    "KB number": [
        "kb123456"
    ]
}
```

Il s’agit du JSON si `verbose=true` est défini dans la chaîne de requête :

```json
"entities": {
    "KB number": [
        "kb123456"
    ],
    "$instance": {
        "KB number": [
            {
                "type": "KB number",
                "text": "kb123456",
                "startIndex": 9,
                "length": 8,
                "modelTypeId": 8,
                "modelType": "Regex Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```

* * *

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les entités :

* [Concepts](luis-concept-entity-types.md)
* [Création](luis-how-to-add-entities.md)