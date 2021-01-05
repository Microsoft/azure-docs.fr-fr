---
title: Modifications de point de terminaison de prédiction dans l’API V3
description: Les API V3 de point de terminaison de prédiction de requête ont évolué. Servez-vous de ce guide pour comprendre comment migrer vers les API de point de terminaison de version 3.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 06/30/2020
ms.openlocfilehash: 59cf250a9db5a1f6759495c1b5a3c48cb07cde15
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95018784"
---
# <a name="prediction-endpoint-changes-for-v3"></a>Modifications de point de terminaison de prédiction pour V3

Les API V3 de point de terminaison de prédiction de requête ont évolué. Servez-vous de ce guide pour comprendre comment migrer vers les API de point de terminaison de version 3.

**État généralement disponible** : cette API V3 inclut des modifications significatives de requête et de réponse JSON par rapport à l’API V2.

L’API V3 fournit les nouvelles fonctionnalités suivantes :

* [Entités externes](schema-change-prediction-runtime.md#external-entities-passed-in-at-prediction-time)
* [Listes dynamiques](schema-change-prediction-runtime.md#dynamic-lists-passed-in-at-prediction-time)
* [Évolutions de JSON – Entités prédéfinies](#prebuilt-entity-changes)

La [requête](#request-changes) et la [réponse](#response-changes) du point de terminaison de prédiction ont évolué de manière significative afin de prendre en charge les nouvelles fonctionnalités mentionnées ci-dessus, notamment les suivantes :

* [Évolutions de l’objet réponse](#top-level-json-changes)
* [Remplacement des références au nom de rôle d’entité par le nom d’entité](#entity-role-name-instead-of-entity-name)
* [Propriétés visant à marquer les entités dans les énoncés](#marking-placement-of-entities-in-utterances)

La [documentation de référence](https://aka.ms/luis-api-v3) est disponible pour V3.

## <a name="v3-changes-from-preview-to-ga"></a>Modifications de V3 entre la préversion à GA

V3 a introduit les modifications suivantes dans le cadre du passage à GA :

* Les entités prédéfinies suivantes ont des réponses JSON différentes :
    * [OrdinalV1](luis-reference-prebuilt-ordinal.md)
    * [GeographyV2](luis-reference-prebuilt-geographyv2.md)
    * [DatetimeV2](luis-reference-prebuilt-datetimev2.md)
    * Nom de la clé d’unité mesurable de `units` à `unit`

* Modification JSON du corps de la demande :
    * de `preferExternalEntities` à `preferExternalEntities`
    * paramètre `score` facultatif pour les entités externes

* Modifications JSON du corps de la réponse :
    * `normalizedQuery` supprimé

## <a name="suggested-adoption-strategy"></a>Stratégie d’adoption suggérée

Si vous utilisez Bot Framework, Vérification orthographique Bing V7 ou si vous souhaitez migrer uniquement votre création d’application LUIS, continuez à utiliser le point de terminaison V2.

Si vous savez qu’aucune de vos applications clientes ou intégrations (Bot Framework et Vérification orthographique Bing V7) n’est affectée et si vous êtes familiarisé avec la migration simultanée de votre création d’application LUIS et de votre point de terminaison de prédiction, commencez à utiliser le point de terminaison de prédiction V3. Le point de terminaison de prédiction V2 sera toujours disponible et constitue une bonne stratégie de secours.


## <a name="not-supported"></a>Non pris en charge

### <a name="bing-spell-check"></a>Vérification orthographique Bing

Cette API n’est pas prise en charge dans le point de terminaison de prédiction V3. Continuez à utiliser le point de terminaison de prédiction d’API V2 pour les corrections orthographiques. Si vous avez besoin de la correction orthographique quand vous utilisez l’API V3, faites en sorte que l’application cliente appelle l’API [Vérification orthographique Bing](../bing-spell-check/overview.md), puis remplacez le texte par l’orthographe correcte, avant d’envoyer le texte à l’API LUIS.

## <a name="bot-framework-and-azure-bot-service-client-applications"></a>Applications clientes Bot Framework et Azure Bot Service

Continuez à utiliser l’API V2 de point de terminaison de prédiction jusqu’à ce que la version 4.7 de Bot Framework soit disponible.

## <a name="v2-api-deprecation"></a>Dépréciation de l’API V2

L’API de prédiction V2 ne sera pas déconseillée pendant au moins 9 mois après la préversion V3, à savoir le 8 juin 2020.

## <a name="endpoint-url-changes"></a>Modifications de l’URL de point de terminaison

### <a name="changes-by-slot-name-and-version-name"></a>Modifications par nom d’emplacement et nom de version

Le [format de l’appel HTTP de point de terminaison](developer-reference-resource.md#rest-endpoints) V3 a évolué.

Si vous souhaitez interroger par version, vous devez d’abord [publier via l’API](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c3b) avec `"directVersionPublish":true`. Interrogez le point de terminaison qui fait référence à l’ID de version plutôt qu’au nom d’emplacement.

|Valeurs valides pour `SLOT-NAME`|
|--|
|`production`|
|`staging`|

## <a name="request-changes"></a>Demander des changements

### <a name="query-string-changes"></a>Modifications de la chaîne de requête

[!INCLUDE [V3 query params](./includes/v3-prediction-query-params.md)]

### <a name="v3-post-body"></a>Corps de POST V3

```JSON
{
    "query":"your utterance here",
    "options":{
        "datetimeReference": "2019-05-05T12:00:00",
        "preferExternalEntities": true
    },
    "externalEntities":[],
    "dynamicLists":[]
}
```

|Propriété|Type|Version|Default|Objectif|
|--|--|--|--|--|
|`dynamicLists`|tableau|V3 uniquement|Non requis.|Les [listes dynamiques](schema-change-prediction-runtime.md#dynamic-lists-passed-in-at-prediction-time) vous permettent d’étendre une entité de liste entraînée et publiée existante, déjà présente dans l’application LUIS.|
|`externalEntities`|tableau|V3 uniquement|Non requis.|Les [entités externes](schema-change-prediction-runtime.md#external-entities-passed-in-at-prediction-time) permettent à votre application LUIS d’identifier et d’étiqueter les entités pendant l’exécution, qui peuvent servir de fonctionnalités aux entités existantes. |
|`options.datetimeReference`|string|V3 uniquement|Pas de valeur par défaut|Utilisé pour déterminer le [décalage de datetimeV2](luis-concept-data-alteration.md#change-time-zone-of-prebuilt-datetimev2-entity). Le format du datetimeReference est [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601).|
|`options.preferExternalEntities`|boolean|V3 uniquement|false|Indique si l’[entité externe (dont le nom est identique à celui de l’entité existante)](schema-change-prediction-runtime.md#override-existing-model-predictions) de l’utilisateur est utilisée ou si l’entité existante du modèle est utilisée pour la prédiction. |
|`query`|string|V3 uniquement|Obligatoire.|**Dans V2**, l’énoncé à prédire se trouve dans le paramètre `q`. <br><br>**Dans V3**, la fonctionnalité est transmise dans le paramètre `query`.|

## <a name="response-changes"></a>Évolutions de la réponse

Le JSON de réponse de requête a évolué pour permettre un accès programmatique élargi aux données les plus souvent utilisées.

### <a name="top-level-json-changes"></a>Évolutions JSON de niveau supérieur



Quand `verbose` est défini sur true, ce qui retourne toutes les intentions et leurs scores dans la propriété `intents`, les principales propriétés JSON pour V2 sont les suivantes :

```JSON
{
    "query":"this is your utterance you want predicted",
    "topScoringIntent":{},
    "intents":[],
    "entities":[],
    "compositeEntities":[]
}
```

Les principales propriétés JSON pour V3 sont :

```JSON
{
    "query": "this is your utterance you want predicted",
    "prediction":{
        "topIntent": "intent-name-1",
        "intents": {},
        "entities":{}
    }
}
```

L’objet `intents` est une liste non triée. Ne partez pas du principe que le premier enfant de `intents` correspond à `topIntent`. Utilisez à la place la valeur de `topIntent` pour rechercher le score :

```nodejs
const topIntentName = response.prediction.topIntent;
const score = intents[topIntentName];
```

Voici ce qu’autorisent les évolutions du schéma JSON de réponse :

* Distinction claire entre l’énoncé d’origine, `query`, et la prédiction retournée, `prediction`.
* Accès programmatique aux données prédites facilité. Au lieu de procéder à une énumération dans un tableau dans V2, vous pouvez accéder aux valeurs par **nom** aussi bien pour les intentions que pour les entités. Pour les rôles d’entité prédite, le nom de rôle est retourné, car il est unique dans toute l’application.
* Les types de données, s’ils sont déterminés, sont respectés. Les valeurs numériques ne sont plus retournées sous forme de chaînes.
* Distinction entre les informations de prédiction de première priorité et les métadonnées supplémentaires, retournées dans l’objet `$instance`.

### <a name="entity-response-changes"></a>Modifications de la réponse d’entité

#### <a name="marking-placement-of-entities-in-utterances"></a>Marquage de l’emplacement des entités dans les énoncés

**Dans V2**, une entité était marquée dans un énoncé avec `startIndex` et `endIndex`.

**Dans V3**, l’entité est marquée avec `startIndex` et `entityLength`.

#### <a name="access-instance-for-entity-metadata"></a>Accès à `$instance` pour les métadonnées d’entité

Si vous avez besoin de métadonnées d’entité, la chaîne de requête doit utiliser l’indicateur `verbose=true` et la réponse contient les métadonnées de l’objet `$instance`. Des exemples sont présentés dans les réponses JSON des sections suivantes.

#### <a name="each-predicted-entity-is-represented-as-an-array"></a>Chaque entité prédite est représentée sous forme de tableau

L’objet `prediction.entities.<entity-name>` contient un tableau, car chaque entité peut être prédite plusieurs fois dans l’énoncé.

<a name="prebuilt-entities-with-new-json"></a>

#### <a name="prebuilt-entity-changes"></a>Modifications de l’entité prédéfinie

L’objet réponse V3 inclut des modifications apportées aux entités prédéfinies. Consultez les [entités prédéfinies spécifiques](luis-reference-prebuilt-entities.md) pour en savoir plus.

#### <a name="list-entity-prediction-changes"></a>Évolutions des prédictions d’entité de liste

Le JSON d’une prédiction d’entité de liste a évolué pour devenir un tableau de tableaux :

```JSON
"entities":{
    "my_list_entity":[
        ["canonical-form-1","canonical-form-2"],
        ["canonical-form-2"]
    ]
}
```
Chaque tableau interne correspond au texte situé à l’intérieur de l’énoncé. L’objet interne est un tableau, car le même texte peut apparaître dans plusieurs sous-listes d’une entité de liste.

Lors d’un mappage entre l’objet `entities` et l’objet `$instance`, l’ordre des objets est conservé pour les prédictions d’entité de liste.

```nodejs
const item = 0; // order preserved, use same enumeration for both
const predictedCanonicalForm = entities.my_list_entity[item];
const associatedMetadata = entities.$instance.my_list_entity[item];
```

#### <a name="entity-role-name-instead-of-entity-name"></a>Le nom de rôle d’entité à la place du nom d’entité

Dans V2, le tableau `entities` retournait toutes les entités prédites avec le nom d’entité en guise d’identificateur unique. Dans V3, si l’entité utilise des rôles et que la prédiction porte sur un rôle d’entité, l’identificateur principal est le nom de rôle. Cela est possible, car les noms de rôle d’entité doivent être uniques dans toute l’application, y compris les autres noms de modèle (intention, entité).

L’exemple suivant contient un énoncé qui comporte le texte `Yellow Bird Lane`. Ce texte est prédit en tant que rôle `Destination` d’une entité `Location` personnalisée.

|Texte d’énoncé|Nom de l’entité|Nom de rôle|
|--|--|--|
|`Yellow Bird Lane`|`Location`|`Destination`|

Dans V2, l’entité est identifiée par le _nom d’entité_ et le rôle fait office de propriété de l’objet :

```JSON
"entities":[
    {
        "entity": "Yellow Bird Lane",
        "type": "Location",
        "startIndex": 13,
        "endIndex": 20,
        "score": 0.786378264,
        "role": "Destination"
    }
]
```

Dans V3, l’entité est référencée par le _rôle d’entité_, si la prédiction porte sur le rôle :

```JSON
"entities":{
    "Destination":[
        "Yellow Bird Lane"
    ]
}
```

Dans V3, le même résultat est obtenu avec l’indicateur `verbose` pour retourner les métadonnées d’entité :

```JSON
"entities":{
    "Destination":[
        "Yellow Bird Lane"
    ],
    "$instance":{
        "Destination": [
            {
                "role": "Destination",
                "type": "Location",
                "text": "Yellow Bird Lane",
                "startIndex": 25,
                "length":16,
                "score": 0.9837309,
                "modelTypeId": 1,
                "modelType": "Entity Extractor"
            }
        ]
    }
}
```

<a name="external-entities-passed-in-at-prediction-time"></a>
<a name="override-existing-model-predictions"></a>

## <a name="extend-the-app-at-prediction-time"></a>Étendre l’application au moment de la prédiction

Découvrez les [concepts](schema-change-prediction-runtime.md) permettant d’étendre l’application au niveau du runtime de prédiction.

## <a name="deprecation"></a>Dépréciation

L’API V2 ne sera pas déconseillée pendant au moins 9 mois après la préversion V3.

## <a name="next-steps"></a>Étapes suivantes

Consultez la documentation de l’API V3 pour mettre à jour les appels REST existants vers les API de [point de terminaison](https://westcentralus.dev.cognitive.microsoft.com/docs/services/luis-endpoint-api-v3-0/operations/5cb0a9459a1fe8fa44c28dd8) LUIS.