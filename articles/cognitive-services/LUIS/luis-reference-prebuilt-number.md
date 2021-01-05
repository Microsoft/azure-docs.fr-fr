---
title: Entité prédéfinie number – LUIS
titleSuffix: Azure Cognitive Services
description: Cet article contient des informations sur l’entité prédéfinie number dans Language Understanding (LUIS).
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 09/27/2019
ms.openlocfilehash: 13594886b83d4474ee2531185db5868a5198ca64
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91541954"
---
# <a name="number-prebuilt-entity-for-a-luis-app"></a>Entité prédéfinie number pour une application LUIS
Des valeurs numériques sont utilisées de plusieurs façons pour quantifier, exprimer et décrire des éléments d’information. Cet article ne couvre que certains exemples possibles. LUIS interprète les variantes des énoncés des utilisateurs, et renvoie des valeurs numériques cohérentes. Étant donné que cette entité est déjà formée, vous n’avez pas besoin d’ajouter d’exemples d’énoncés contenant un nombre aux intentions de l’application.

## <a name="types-of-number"></a>Types d’entités number
L’entité number est gérée à partir du dépôt GitHub [Recognizers-text](https://github.com/Microsoft/Recognizers-Text/blob/master/Patterns/English/English-Numbers.yaml)

## <a name="examples-of-number-resolution"></a>Exemples de résolution de l’entité number

| Énoncé        | Entité   | Résolution |
| ------------- |:----------------:| --------------:|
| ```one thousand times```  | ```"one thousand"``` |   ```"1000"```      |
| ```1,000 people```        | ```"1,000"```    |   ```"1000"```      |
| ```1/2 cup```         | ```"1 / 2"```    |    ```"0.5"```      |
|  ```one half the amount```     | ```"one half"```     |    ```"0.5"```      |
| ```one hundred fifty orders``` | ```"one hundred fifty"``` | ```"150"``` |
| ```one hundred and fifty books``` | ```"one hundred and fifty"``` | ```"150"```|
| ```a grade of one point five```| ```"one point five"``` |  ```"1.5"``` |
| ```buy two dozen eggs```    | ```"two dozen"``` | ```"24"``` |


LUIS inclut la valeur reconnue d’une entité **`builtin.number`** dans le champ `resolution` de la réponse JSON renvoyées.

## <a name="resolution-for-prebuilt-number"></a>Résolution pour une entité prédéfinie number

Les objets entité suivants sont retournés pour la requête :

`order two dozen eggs`

#### <a name="v3-response"></a>[Réponse V3](#tab/V3)

Le code JSON suivant a le paramètre `verbose` défini sur `false` :

```json
"entities": {
    "number": [
        24
    ]
}
```
#### <a name="v3-verbose-response"></a>[Réponse détaillée V3](#tab/V3-verbose)

Le code JSON suivant a le paramètre `verbose` défini sur `true` :

```json
"entities": {
    "number": [
        24
    ],
    "$instance": {
        "number": [
            {
                "type": "builtin.number",
                "text": "two dozen",
                "startIndex": 6,
                "length": 9,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```
#### <a name="v2-response"></a>[Réponse V2](#tab/V2)

L’exemple suivant montre une réponse JSON de LUIS, qui inclut la résolution de la valeur 24 pour l’énoncé "two dozen" (deux douzaines).

```json
"entities": [
  {
    "entity": "two dozen",
    "type": "builtin.number",
    "startIndex": 6,
    "endIndex": 14,
    "resolution": {
      "subtype": "integer",
      "value": "24"
    }
  }
]
```
* * *

## <a name="next-steps"></a>Étapes suivantes

Découvrez-en plus sur le [point de terminaison de prédiction V3](luis-migration-api-v3.md).

En savoir plus sur les entités [currency](luis-reference-prebuilt-currency.md), [ordinal](luis-reference-prebuilt-ordinal.md) et [percentage](luis-reference-prebuilt-percentage.md).
