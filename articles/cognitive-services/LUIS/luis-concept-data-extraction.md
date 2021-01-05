---
title: Extraction des données : LUIS
description: Extraire des données d’un texte d’énoncé avec des intentions et des entités. Découvrez quel type de données il est possible d’extraire à partir de Language Understanding (LUIS).
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 05/01/2020
ms.openlocfilehash: e6f01354bb5aa2b78d3c9962bac49be39dd2c81f
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025991"
---
# <a name="extract-data-from-utterance-text-with-intents-and-entities"></a>Extraire des données d’un texte d’énoncé avec des intentions et des entités
LUIS donne la possibilité d’obtenir des informations à partir des énoncés d’un utilisateur en langage naturel. Les informations sont extraites de façon à pouvoir être utilisées par un programme, une application ou un chatbot de manière exploitable. Dans les sections suivantes, découvrez quelles sont les données retournées à partir des intentions et des entités avec des exemples de JSON.

Les données les plus difficiles à extraire sont les données de machine-learning, car il n’y a pas de correspondance de texte exacte. L’extraction de données à partir [d’entités](luis-concept-entity-types.md) de machine-learning doit faire partie du [cycle de création](luis-concept-app-iteration.md) jusqu’à ce que vous ayez la certitude de recevoir les données attendues.

## <a name="data-location-and-key-usage"></a>Emplacement des données et utilisation de la clé
LUIS extrait des données à partir de l’énoncé de l’utilisateur au [point de terminaison](luis-glossary.md#endpoint) publié. La **requête HTTPS** (POST ou GET) contient l’énoncé ainsi que certaines configurations facultatives, comme l’environnement de production ou l’environnement intermédiaire.

**Demande de point de terminaison de prédiction V2**

`https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/<appID>?subscription-key=<subscription-key>&verbose=true&timezoneOffset=0&q=book 2 tickets to paris`

**Demande de point de terminaison de prédiction V3**

`https://westus.api.cognitive.microsoft.com/luis/v3.0-preview/apps/<appID>/slots/<slot-type>/predict?subscription-key=<subscription-key>&verbose=true&timezoneOffset=0&query=book 2 tickets to paris`

La valeur `appID` est disponible dans la page **Paramètres** de l’application LUIS ; elle fait également partie de l’URL (après `/apps/`) dans la page de modification de l’application. La clé `subscription-key` est la clé du point de terminaison utilisée pour interroger l’application. Vous pouvez utiliser votre clé gratuite de démarrage/création tant que vous apprenez à vous servir de LUIS, mais il est important de remplacer la clé de point de terminaison par une clé qui prend en charge votre [utilisation attendue de LUIS](luis-limits.md#key-limits). L’unité `timezoneOffset` est exprimée en minutes.

La **réponse HTTPS** contient toutes les informations que LUIS peut déterminer sur les intentions et les entités à partir du modèle publié à un instant t d’un point de terminaison intermédiaire ou de production. L’URL de point de terminaison se trouve sur le site web [LUIS](luis-reference-regions.md), dans la section **Manage** (Gérer), dans la page **Keys and endpoints** (Clés et points de terminaison).

## <a name="data-from-intents"></a>Données issues des intentions
La principale donnée est le **nom de l’intention** qui a obtenu le meilleur score. La réponse du point de terminaison est la suivante :

#### <a name="v2-prediction-endpoint-response"></a>[Réponse de point de terminaison de prédiction V2](#tab/V2)

```JSON
{
  "query": "when do you open next?",
  "topScoringIntent": {
    "intent": "GetStoreInfo",
    "score": 0.984749258
  },
  "entities": []
}
```

#### <a name="v3-prediction-endpoint-response"></a>[Réponse de point de terminaison de prédiction V3](#tab/V3)

```JSON
{
  "query": "when do you open next?",
  "prediction": {
    "normalizedQuery": "when do you open next?",
    "topIntent": "GetStoreInfo",
    "intents": {
        "GetStoreInfo": {
            "score": 0.984749258
        }
    }
  },
  "entities": []
}
```

Découvrez-en plus sur le [point de terminaison de prédiction V3](luis-migration-api-v3.md).

* * *

|Objet de données|Type de données|Emplacement des données|Valeur|
|--|--|--|--|
|Intentionnel|String|topScoringIntent.intent|"GetStoreInfo"|

Si votre chatbot ou votre application d’appel à LUIS prend une décision sur la base du score de plusieurs intentions, retournez le score de toutes les intentions.


#### <a name="v2-prediction-endpoint-response"></a>[Réponse de point de terminaison de prédiction V2](#tab/V2)

Définissez le paramètre de chaîne de requête, `verbose=true`. La réponse du point de terminaison est la suivante :

```JSON
{
  "query": "when do you open next?",
  "topScoringIntent": {
    "intent": "GetStoreInfo",
    "score": 0.984749258
  },
  "intents": [
    {
      "intent": "GetStoreInfo",
      "score": 0.984749258
    },
    {
      "intent": "None",
      "score": 0.2040639
    }
  ],
  "entities": []
}
```

#### <a name="v3-prediction-endpoint-response"></a>[Réponse de point de terminaison de prédiction V3](#tab/V3)

Définissez le paramètre de chaîne de requête, `show-all-intents=true`. La réponse du point de terminaison est la suivante :

```JSON
{
    "query": "when do you open next?",
    "prediction": {
        "normalizedQuery": "when do you open next?",
        "topIntent": "GetStoreInfo",
        "intents": {
            "GetStoreInfo": {
                "score": 0.984749258
            },
            "None": {
                 "score": 0.2040639
            }
        },
        "entities": {
        }
    }
}
```

Découvrez-en plus sur le [point de terminaison de prédiction V3](luis-migration-api-v3.md).

* * *

Les intentions sont triées du score le plus élevé au score le plus faible.

|Objet de données|Type de données|Emplacement des données|Valeur|Score|
|--|--|--|--|:--|
|Intentionnel|String|intents[0].intent|"GetStoreInfo"|0.984749258|
|Intentionnel|String|intents[1].intent|"None"|0.0168218873|

Si vous ajoutez des domaines prédéfinis, le nom de l’intention indique le domaine, par exemple `Utilties` ou `Communication`, ainsi que l’intention :

#### <a name="v2-prediction-endpoint-response"></a>[Réponse de point de terminaison de prédiction V2](#tab/V2)

```JSON
{
  "query": "Turn on the lights next monday at 9am",
  "topScoringIntent": {
    "intent": "Utilities.ShowNext",
    "score": 0.07842206
  },
  "intents": [
    {
      "intent": "Utilities.ShowNext",
      "score": 0.07842206
    },
    {
      "intent": "Communication.StartOver",
      "score": 0.0239675418
    },
    {
      "intent": "None",
      "score": 0.0168218873
    }],
  "entities": []
}
```

#### <a name="v3-prediction-endpoint-response"></a>[Réponse de point de terminaison de prédiction V3](#tab/V3)

```JSON
{
    "query": "Turn on the lights next monday at 9am",
    "prediction": {
        "normalizedQuery": "Turn on the lights next monday at 9am",
        "topIntent": "Utilities.ShowNext",
        "intents": {
            "Utilities.ShowNext": {
                "score": 0.07842206
            },
            "Communication.StartOver": {
                "score": 0.0239675418
            },
            "None": {
                "score": 0.00085447653
            }
        },
        "entities": []
    }
}
```

Découvrez-en plus sur le [point de terminaison de prédiction V3](luis-migration-api-v3.md).

* * *

|Domain|Objet de données|Type de données|Emplacement des données|Valeur|
|--|--|--|--|--|
|Services|Intentionnel|String|intents[0].intent|"<b>Utilities</b>.ShowNext"|
|Communication|Intentionnel|String|intents[1].intent|<b>Communication</b>.StartOver"|
||Intentionnel|String|intents[2].intent|"None"|


## <a name="data-from-entities"></a>Données issues des entités
Pour la plupart des applications et chatbots, le nom d’intention n’est pas suffisant. Les données supplémentaires facultatives proviennent d’entités découvertes dans l’énoncé. Chaque type d’entité retourne des informations différentes sur la correspondance.

Un seul mot ou une seule expression dans l’énoncé peut correspondre à plusieurs entités. Dans ce cas, chaque entité présentant une correspondance est retournée avec son score.

Toutes les entités sont renvoyées dans le tableau **entities** de la réponse provenant du point de terminaison

## <a name="tokenized-entity-returned"></a>Entité retournée segmentée en unités lexicales

Examinez la [prise en charge des jetons](luis-language-support.md#tokenization) dans LUIS.


## <a name="prebuilt-entity-data"></a>Données d’entité prédéfinie
Les entités [prédéfinies](luis-concept-entity-types.md) sont découvertes par correspondance avec une expression régulière à l’aide du projet [Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) open source. Elles sont retournées dans le tableau entities et utilisent le nom de type avec le préfixe `builtin::`.

## <a name="list-entity-data"></a>Données d’entité de liste

Les [entités de liste](reference-entity-list.md) représentent un ensemble fixe, fermé de mots associés, ainsi que leurs synonymes. LUIS ne détecte pas les valeurs supplémentaires pour les entités de liste. Utilisez la fonctionnalité **Recommander** pour trouver des suggestions de nouveaux mots à partir de la liste actuelle. S’il existe plusieurs entités de liste avec la même valeur, chaque entité est retournée dans la requête du point de terminaison.

## <a name="regular-expression-entity-data"></a>Données d’entité d’expression régulière

Une [entité d’expression régulière](reference-entity-regular-expression.md) extrait une entité en fonction d’une expression régulière que vous fournissez.

## <a name="extracting-names"></a>Extraction de noms
Il est difficile d’extraire des noms d’un énoncé, car un nom peut être pratiquement n’importe quelle combinaison de lettres et de mots. Les possibilités dépendent du type de nom extrait. Les suggestions suivantes ne sont pas des règles, mais plutôt des recommandations.

### <a name="add-prebuilt-personname-and-geographyv2-entities"></a>Ajouter des entités PersonName et GeographyV2 prédéfinies

Les entités [PersonName](luis-reference-prebuilt-person.md) et [GeographyV2](luis-reference-prebuilt-geographyV2.md) sont disponibles dans certaines [cultures de langue](luis-reference-prebuilt-entities.md).

### <a name="names-of-people"></a>Noms de personnes

Les noms de personnes peuvent avoir un format légèrement en fonction de la langue et de la culture. Utilisez une entité **[personName](luis-reference-prebuilt-person.md)** prédéfinie ou une **[entité simple](luis-concept-entity-types.md)** avec les rôles du prénom et du nom.

Si vous utilisez l’entité simple, veillez à donner des exemples qui utilisent le prénom et le nom à différents endroits de l’énoncé, dans des énoncés de longueurs différentes et pour toutes les intentions, y compris l’intention None. [Vérifiez](./luis-how-to-review-endpoint-utterances.md) régulièrement les énoncés du point de terminaison pour étiqueter les noms qui n’ont pas été prédits correctement.

### <a name="names-of-places"></a>Noms de lieux

Les noms d’endroits sont définis et connus : villes, départements, États, provinces et pays/régions. Utilisez l’entité prédéfinie **[geographyV2](luis-reference-prebuilt-geographyv2.md)** pour extraire des informations de localisation.

### <a name="new-and-emerging-names"></a>Nouveaux noms

Certaines applications doivent être capables de rechercher les nouveaux noms, comme les produits ou les entreprises. Ces types de noms sont les plus difficiles à extraire. Commencez par une **[entité simple](luis-concept-entity-types.md#simple-entity)** et ajoutez une [liste d’expressions](luis-concept-feature.md). [Vérifiez](./luis-how-to-review-endpoint-utterances.md) régulièrement les énoncés du point de terminaison pour étiqueter les noms qui n’ont pas été prédits correctement.

## <a name="patternany-entity-data"></a>Données d’entité Pattern.any

[Pattern.any](reference-entity-pattern-any.md) est un espace réservé à longueur variable utilisé uniquement dans le gabarit d’énoncé d’un modèle pour marquer où l’entité commence et se termine. L’entité utilisée dans le modèle doit être trouvée pour que celui-ci soit appliqué.

## <a name="sentiment-analysis"></a>analyse de sentiments
Si l’analyse des sentiments est configurée lors de la [publication](luis-how-to-publish-app.md#sentiment-analysis), la réponse JSON de LUIS l’intègre. Pour plus d’informations sur l’analyse des sentiments, consultez la documentation [Analyse de texte](../text-analytics/index.yml).

## <a name="key-phrase-extraction-entity-data"></a>Données d’entité d’extraction de phrases clés
L’[entité d’extraction de phrases clés](luis-reference-prebuilt-keyphrase.md) renvoie les phrases clés de l’énoncé, fournies par l’[Analyse de texte](../text-analytics/index.yml).

## <a name="data-matching-multiple-entities"></a>Données correspondant à plusieurs entités

LUIS retourne toutes les entités découvertes dans l’énoncé. Le chatbot devra donc dans certains cas prendre des décisions en fonction des résultats.

## <a name="data-matching-multiple-list-entities"></a>Données correspondant à plusieurs entités de liste

Si un mot ou une phrase correspond à plusieurs entités Liste, la requête du point de terminaison retourne chaque entité Liste.

Pour la requête `when is the best time to go to red rock?`, et si l’application contient le mot `red` dans plusieurs listes, LUIS reconnaît toutes les entités et renvoie un tableau d’entités dans le cadre de la réponse du point de terminaison JSON.

## <a name="next-steps"></a>Étapes suivantes

Consulter [Ajouter des entités](luis-how-to-add-entities.md) pour découvrir comment ajouter des entités à votre application LUIS.
