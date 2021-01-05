---
title: Type d’entité Pattern.any - LUIS
titleSuffix: Azure Cognitive Services
description: Pattern.any est un espace réservé à longueur variable utilisé uniquement dans le gabarit d’énoncé d’un modèle pour marquer où l’entité commence et se termine.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 09/29/2019
ms.openlocfilehash: ec23be3709cebc534c059a21c52452abff683b18
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91542204"
---
# <a name="patternany-entity"></a>Entité Pattern.any

Pattern.any est un espace réservé à longueur variable utilisé uniquement dans le gabarit d’énoncé d’un modèle pour marquer où l’entité commence et se termine.  

Les entités pattern.any doivent être marquées dans les exemples de modèles [Pattern](luis-how-to-model-intent-pattern.md), et non dans les exemples d’utilisateurs d’intention.

**L’entité convient bien quand :**

* La fin de l’entité peut être confondue avec le reste du texte de l’énoncé.

## <a name="usage"></a>Usage

Si une application cliente recherche des livres en fonction du titre, pattern.any extrait le titre complet. Un modèle d’énoncé utilisant pattern.any pour cette recherche de livre est `Was {BookTitle} written by an American this year[?]`.

Dans le tableau suivant, chaque ligne contient deux versions de l’énoncé. L’énoncé principal est la manière dont LUIS a initialement pu voir l’énoncé. Il est clair que le titre du livre a un début et une fin. L’énoncé inférieur utilise une entité pattern.any pour marquer le début et la fin de l’entité.

|Énoncé d’une entité en gras|
|--|
|`Was The Man Who Mistook His Wife for a Hat and Other Clinical Tales written by an American this year?`<br><br>**L’Homme qui prenait sa femme pour un chapeau** a-t-il été écrit par un Americain cette année ?|
|`Was Half Asleep in Frog Pajamas written by an American this year?`<br><br>**Nature morte avec pivert** a-t-il été par un Américain cette année ?|
|`Was The Particular Sadness of Lemon Cake: A Novel written by an American this year?`<br><br>La **singulière tristesse du gâteau au citron** est-il un roman écrit par un Américain cette année ?|
|`Was There's A Wocket In My Pocket! written by an American this year?`<br><br>**Le Petit Prince** a-t-il été écrit par un Américain cette année ?|
||



## <a name="example-json"></a>Exemple JSON

Considérez la requête suivante :

`where is the form Understand your responsibilities as a member of the community and who needs to sign it after I read it?`

Avec le nom de formulaire incorporé à extraire en tant que Pattern.any :

`Understand your responsibilities as a member of the community`

#### <a name="v2-prediction-endpoint-response"></a>[Réponse de point de terminaison de prédiction V2](#tab/V2)

```JSON
"entities": [
  {
    "entity": "understand your responsibilities as a member of the community",
    "type": "FormName",
    "startIndex": 18,
    "endIndex": 78,
    "role": ""
  }
```


#### <a name="v3-prediction-endpoint-response"></a>[Réponse de point de terminaison de prédiction V3](#tab/V3)

Il s’agit du JSON si `verbose=false` est défini dans la chaîne de requête :

```json
"entities": {
    "FormName": [
        "Understand your responsibilities as a member of the community"
    ]
}
```

Il s’agit du JSON si `verbose=true` est défini dans la chaîne de requête :

```json
"entities": {
    "FormName": [
        "Understand your responsibilities as a member of the community"
    ],
    "$instance": {
        "FormName": [
            {
                "type": "FormName",
                "text": "Understand your responsibilities as a member of the community",
                "startIndex": 18,
                "length": 61,
                "modelTypeId": 7,
                "modelType": "Pattern.Any Entity Extractor",
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

Dans ce [tutoriel](luis-tutorial-pattern.md), vous allez utiliser l’entité **Pattern.any** pour extraire des données à partir d’énoncés correctement mis en forme et où la fin des données peut être facilement confondue avec les mots restants de l’énoncé.
