---
title: Étendre l’application au niveau du runtime – LUIS
description: ''
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 04/14/2020
ms.openlocfilehash: 69e2608fb01ece81f555aae2f3d4a2e4a05cfc90
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91322799"
---
# <a name="extend-app-at-prediction-runtime"></a>Étendre l’application au niveau du runtime de prédiction

Le schéma de l’application (modèles et caractéristiques) est formé et publié sur le point de terminaison de prédiction. Ce modèle publié est utilisé sur le runtime de prédiction. Vous pouvez transmettre de nouvelles informations, en même temps que l’énoncé de l’utilisateur, au runtime de prédiction pour améliorer la prédiction.

Deux modifications du schéma du runtime de prédiction sont incluses :
* [Entités externes](#external-entities)
* [Listes dynamiques](#dynamic-lists)

<a name="external-entities-passed-in-at-prediction-time"></a>

## <a name="external-entities"></a>Entités externes

Les entités externes permettent à votre application LUIS d’identifier et d’étiqueter les entités pendant l’exécution, qui peuvent servir de fonctionnalités aux entités existantes. Cela vous permet d’utiliser vos propres extracteurs d’entités distincts et personnalisés avant d’envoyer des requêtes à votre point de terminaison de prédiction. Cette opération étant exécutée sur le point de terminaison de prédiction de requête, vous n’avez pas besoin de réentraîner votre modèle et de le publier.

L’application cliente fournit son propre extracteur d’entités en gérant la mise en correspondance des entités et en déterminant l’emplacement au sein de l’énoncé des entités mises en correspondance, puis en envoyant ces informations avec la demande.

Les entités externes constituent le mécanisme qui permet d’étendre n’importe quel type d’entité tout en étant toujours utilisées comme signaux pour d’autres modèles.

Cela s’avère utile pour une entité qui n’a accès à des données qu’au moment de l’exécution de la prédiction de requête. Les données qui évoluent constamment ou qui sont propres à chaque utilisateur sont des exemples de ce type de données. Vous pouvez étendre une entité de contact LUIS avec des informations externes provenant de la liste de contacts d’un utilisateur.

Les entités externes font partie de l’API de création V3. En savoir plus sur la [migration](luis-migration-api-v3.md) vers cette version.

### <a name="entity-already-exists-in-app"></a>L’entité existe déjà dans l’application

La valeur de `entityName` pour l’entité externe, qui est transmise dans le corps POST de demande du point de terminaison, doit déjà exister dans l’application entraînée et publiée au moment où la demande est effectuée. Tous les types étant pris en charge, le type d’entité n’a pas d’importance.

### <a name="first-turn-in-conversation"></a>Premier tour de la conversation

Imaginez un premier énoncé de conversation de bot conversationnel où un utilisateur entre les informations incomplètes suivantes :

`Send Hazem a new message`

La demande du chatbot à LUIS peut transmettre des informations dans le corps POST concernant `Hazem` de façon à les faire correspondre directement à l’un des contacts de l’utilisateur.

```json
    "externalEntities": [
        {
            "entityName":"contacts",
            "startIndex": 5,
            "entityLength": 5,
            "resolution": {
                "employeeID": "05013",
                "preferredContactType": "TeamsChat"
            }
        }
    ]
```

La réponse de prédiction comprend cette entité externe, avec toutes les autres entités prédites, car elle est définie dans la demande.

### <a name="second-turn-in-conversation"></a>Deuxième tour de la conversation

L’énoncé utilisateur suivant dans le bot conversationnel utilise un terme plus vague :

`Send him a calendar reminder for the party.`

Dans ce tour de la conversation, l’énoncé utilise `him` comme référence à `Hazem`. Dans le corps POST, le bot conversationnel peut mapper `him` à la valeur d’entité extraite du premier énoncé, `Hazem`.

```json
    "externalEntities": [
        {
            "entityName":"contacts",
            "startIndex": 5,
            "entityLength": 3,
            "resolution": {
                "employeeID": "05013",
                "preferredContactType": "TeamsChat"
            }
        }
    ]
```

La réponse de prédiction comprend cette entité externe, avec toutes les autres entités prédites, car elle est définie dans la demande.

### <a name="override-existing-model-predictions"></a>Remplacer les prédictions de modèle existantes

La propriété des options `preferExternalEntities` indique que si l’utilisateur envoie une entité externe qui se chevauche avec une entité prédite de même nom, LUIS choisit l’entité transmise ou l’entité existant dans le modèle.

Prenons l’exemple de la requête `today I'm free`. LUIS détecte `today` comme datetimeV2 avec la réponse suivante :

```JSON
"datetimeV2": [
    {
        "type": "date",
        "values": [
            {
                "timex": "2019-06-21",
                "value": "2019-06-21"
            }
        ]
    }
]
```

Si l’utilisateur envoie l’entité externe :

```JSON
{
    "entityName": "datetimeV2",
    "startIndex": 0,
    "entityLength": 5,
    "resolution": {
        "date": "2019-06-21"
    }
}
```

Si `preferExternalEntities` a la valeur `false`, LUIS retourne une réponse comme si l’entité externe n’était pas envoyée.

```JSON
"datetimeV2": [
    {
        "type": "date",
        "values": [
            {
                "timex": "2019-06-21",
                "value": "2019-06-21"
            }
        ]
    }
]
```

Si `preferExternalEntities` a la valeur `true`, LUIS retourne une réponse comprenant ce qui suit :

```JSON
"datetimeV2": [
    {
        "date": "2019-06-21"
    }
]
```



#### <a name="resolution"></a>Résolution

La propriété _facultative_ `resolution` est retournée dans la réponse de prédiction, ce qui vous permet de transmettre les métadonnées associées à l’entité externe, puis de la recevoir en retour dans la réponse.

Le principal objectif est d’étendre les entités prédéfinies, mais il ne se limite pas à ce type d’entité.

La propriété `resolution` peut être un nombre, une chaîne, un objet ou un tableau :

* "Dallas"
* {"text": "value"}
* 12345
* ["a", "b", "c"]

<a name="dynamic-lists-passed-in-at-prediction-time"></a>

## <a name="dynamic-lists"></a>Listes dynamiques

Les listes dynamiques vous permettent d’étendre une entité de liste entraînée et publiée existante, déjà présente dans l’application LUIS.

Cette fonctionnalité est utile quand vos valeurs d’entité de liste doivent être modifiées périodiquement. Elle vous permet d’étendre une entité de liste déjà entraînée et publiée :

* Au moment de la demande du point de terminaison de prédiction de requête.
* Pour une demande unique.

L’entité de liste peut être vide dans l’application LUIS, mais elle doit exister. L’entité de liste n’est pas modifiée dans l’application LUIS, mais la capacité de prédiction au niveau du point de terminaison est étendue pour inclure jusqu’à 2 listes de plus de 1 000 éléments.

### <a name="dynamic-list-json-request-body"></a>Corps de demande JSON de liste dynamique

Envoyez le corps JSON suivant pour ajouter une nouvelle sous-liste constituée de synonymes à la liste et prédire l’entité de liste pour le texte, `LUIS`, avec le demande de prédiction de requête `POST` :

```JSON
{
    "query": "Send Hazem a message to add an item to the meeting agenda about LUIS.",
    "options":{
        "timezoneOffset": "-8:00"
    },
    "dynamicLists": [
        {
            "listEntity*":"ProductList",
            "requestLists":[
                {
                    "name": "Azure Cognitive Services",
                    "canonicalForm": "Azure-Cognitive-Services",
                    "synonyms":[
                        "language understanding",
                        "luis",
                        "qna maker"
                    ]
                }
            ]
        }
    ]
}
```

La réponse de prédiction comprend cette entité de liste, comprenant toutes les autres entités prédites, car elle est définie dans la demande.

## <a name="next-steps"></a>Étapes suivantes

* [Score de prédiction](luis-concept-prediction-score.md)
* [Modifications apportées à l’API de création V3](luis-migration-api-v3.md)
