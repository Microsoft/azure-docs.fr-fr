---
title: Règles de mise à plat et d’échappement JSON – Azure Time Series Insights Gen2 | Microsoft Docs
description: En savoir plus sur la mise à plat, l’échappement et la gestion des tableaux JSON dans Azure Time Series Insights Gen2.
author: lyrana
ms.author: lyhughes
manager: deepakpalled
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 09/28/2020
ms.openlocfilehash: 0839d2c734418824952f37cb177490e56e1133c5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017965"
---
# <a name="json-flattening-escaping-and-array-handling"></a>Mise à plat, échappement et gestion des tableaux JSON

Votre environnement Azure Time Series Insights Gen2 crée de manière dynamique les colonnes de vos stockages chauds et froids, en suivant un ensemble particulier de conventions d’affectation de noms. Lorsqu’un événement est ingéré, un ensemble de règles est appliqué à la charge utile JSON et aux noms de propriétés. Celles-ci incluent l’échappement de certains caractères spéciaux et l’aplatissement des objets JSON imbriqués. Il est important de connaître ces règles afin que vous compreniez comment la forme de votre JSON influencera le stockage et l’interrogation de vos événements. Pour obtenir une liste des règles, consultez le tableau ci-dessous. Les exemples A et B démontrent également comment vous pouvez efficacement traiter par lot plusieurs séries chronologiques dans un tableau.

> [!IMPORTANT]
>
> * Passez en revue les règles ci-dessous avant de sélectionner une [propriété de l’ID de la série chronologique](./how-to-select-tsid.md) et/ou votre source d’événements de l’[horodateur](concepts-streaming-ingestion-event-sources.md#event-source-timestamp). Si votre ID TS ou horodateur se trouve dans un objet imbriqué ou contient un ou plusieurs caractères spéciaux ci-dessous, il est important de s’assurer que le nom de la propriété que vous fournissez correspond au nom de la colonne *après* que les règles d’ingestion ont été appliquées. Voir l’exemple [B](concepts-json-flattening-escaping-rules.md#example-b) ci-dessous.

| Règle | Exemple JSON | [Syntaxe Time Series Expression](/rest/api/time-series-insights/reference-time-series-expression-syntax) | Nom de la colonne de propriété dans Parquet
|---|---|---|---|
| Le type de données Azure Time Series Insights Gen2 est ajouté à la fin de votre nom de colonne en tant que « _\<dataType\> ». | ```"type": "Accumulated Heat"``` | `$event.type.String` |`type_string` |
| La [propriété d’horodatage](concepts-streaming-ingestion-event-sources.md#event-source-timestamp) de la source de l’événement sera enregistrée dans Azure Time Series Insights Gen2 sous « timestamp » dans le stockage, et la valeur stockée au format UTC. Vous pouvez personnaliser la propriété d’horodatage de la ou des sources d’événements pour répondre aux besoins de votre solution, mais le nom de la colonne dans stockage à chaud et à froid est « timestamp ». Les autres propriétés DateHeure JSON qui ne sont pas liées à l’horodatage de la source de l’événement sont enregistrées sous « _datetime » dans le nom de la colonne, comme indiqué dans la règle ci-dessus.  | ```"ts": "2020-03-19 14:40:38.318"``` |  `$event.$ts` | `timestamp` |
| Les noms des propriétés JSON qui incluent les caractères spéciaux. [  \ et ' sont placés dans une séquence d’échappement [' et ']  |  ```"id.wasp": "6A3090FD337DE6B"``` |  `$event['id.wasp'].String` | `['id.wasp']_string` |
| Dans ['et'], des guillemets simples et des barres obliques inverses supplémentaires sont inclus. Un guillemet simple est écrit sous la forme \ et une barre oblique inverse est écrite sous la forme \\\ | ```"Foo's Law Value": "17.139999389648"``` | `$event['Foo\'s Law Value'].Double` | `['Foo\'s Law Value']_double` |
| Les objets JSON imbriqués sont aplatis à l’aide d’un point comme séparateur. L’imbrication jusqu’à 10 niveaux est prise en charge. |  ```"series": {"value" : 316 }``` | `$event.series.value.Long`, `$event['series']['value'].Long` ou `$event.series['value'].Long` |  `series.value_long` |
| Les tableaux de types primitifs sont stockés en tant que type dynamique |  ```"values": [154, 149, 147]``` | Les types dynamiques peuvent uniquement être récupérés via l’API [GetEvents](/rest/api/time-series-insights/dataaccessgen2/query/execute#getevents). | `values_dynamic` |
| Les tableaux contenant des objets ont deux comportements en fonction du contenu de l’objet : Si les ID TS ou la/les propriété(s) d’horodatage se trouvent dans les objets d’un tableau, le tableau sera déroulé de sorte que la charge utile JSON initiale génère plusieurs événements. Cela vous permet de traiter par lot plusieurs événements dans une structure JSON. Toutes les propriétés de niveau supérieur qui sont des homologues du tableau seront enregistrées avec chaque objet non restauré. Si vos ID et horodateurs TS ne sont *pas* dans le tableau, ils sont enregistrés dans son intégralité en tant que type dynamique. | Consultez les exemples [A](concepts-json-flattening-escaping-rules.md#example-a), [B](concepts-json-flattening-escaping-rules.md#example-b) et [C](concepts-json-flattening-escaping-rules.md#example-c) ci-dessous
| Les tableaux contenant des éléments mixtes ne sont pas aplatis. |  ```"values": ["foo", {"bar" : 149}, 147]``` | Les types dynamiques peuvent uniquement être récupérés via l’API [GetEvents](/rest/api/time-series-insights/dataaccessgen2/query/execute#getevents). | `values_dynamic` |
| 512 caractères correspond à la limite du nom de propriété JSON. Si le nom dépasse 512 caractères, il sera tronqué à 512 et '_<'code de hachage'>' sera ajouté. **Notez** que cela s’applique également aux noms de propriétés qui ont été concaténés à partir d’un objet aplati, désignant un chemin d’objet imbriqué. |``"data.items.datapoints.values.telemetry<...continuing to over 512 chars>" : 12.3440495`` |`"$event.data.items.datapoints.values.telemetry<...continuing to include all chars>.Double"` | `data.items.datapoints.values.telemetry<...continuing to 512 chars>_912ec803b2ce49e4a541068d495ab570_double` |

## <a name="understanding-the-dual-behavior-for-arrays"></a>Fonctionnement du double comportement pour les tableaux

Les tableaux d’objets seront stockés dans leur totalité ou répartis dans plusieurs événements en fonction de la façon dont vous avez modélisé vos données. Cela vous permet d'utiliser un tableau pour regrouper les événements et éviter de répéter les propriétés de télémétrie qui sont définies au niveau de l'objet racine. Le traitement par lot peut être avantageux, car moins de messages Event Hubs ou IoT Hub sont envoyés.

Toutefois, dans certains cas, les tableaux contenant des objets ne sont significatifs que dans le contexte d’autres valeurs. La création de plusieurs événements rendrait les données incompréhensibles. Pour vous assurer qu’un tableau d’objets est stocké en tant que type dynamique, suivez les instructions de modélisation des données ci-dessous et examinez l’[exemple C](concepts-json-flattening-escaping-rules.md#example-c)

### <a name="how-to-know-if-my-array-of-objects-will-produce-multiple-events"></a>Comment savoir si mon tableau d’objets produira plusieurs événements

Si un ou plusieurs de vos ID de série chronologique sont imbriqués dans des objets d’un tableau *ou* si la propriété d’horodatage de la source de l’événement est imbriquée, le moteur d’ingestion le fractionne pour créer plusieurs événements. Les noms de propriétés que vous avez fournis pour votre/vos ID TS et/ou horodateur doivent suivre les règles d’aplatissement ci-dessus. Ils indiquent donc la forme de votre JSON. Consultez les exemples ci-dessous, et consultez le guide pour apprendre à [sélectionner une propriété d’ID de série chronologique.](./how-to-select-tsid.md)

### <a name="example-a"></a>Exemple A

ID de la série chronologique au niveau de la racine de l’objet et de l’horodateur imbriqué\
**ID de série chronologique de l’environnement :** `"id"`\
**Horodateur de la source de l’événement :** `"values.time"`\
**Charge utile JSON :**

```JSON
[
    {
        "id": "caaae533-1d6c-4f58-9b75-da102bcc2c8c",
        "values": [
            {
                "time": "2020-05-01T00:59:59.000Z",
                "value": 25.6073
            },
            {
                "time": "2020-05-01T01:00:29.000Z",
                "value": 43.9077
            }
        ]
    },
    {
        "id": "1ac87b74-0865-4a07-b512-56602a3a576f",
        "values": [
            {
                "time": "2020-05-01T00:59:59.000Z",
                "value": 0.337288
            },
            {
                "time": "2020-05-01T01:00:29.000Z",
                "value": 4.76562
            }
        ]
    }
]
```

**Résultat dans le fichier Parquet :** \
La configuration et la charge utile ci-dessus génèrent trois colonnes et quatre événements

| timestamp  | id_string | values.value_double
| ---- | ---- | ---- |
| `2020-05-01T00:59:59.000Z` | `caaae533-1d6c-4f58-9b75-da102bcc2c8c`| ``25.6073`` |
| `2020-05-01T01:00:29.000Z` |`caaae533-1d6c-4f58-9b75-da102bcc2c8c` | ``43.9077`` |
| `2020-05-01T00:59:59.000Z` | `1ac87b74-0865-4a07-b512-56602a3a576f` | ``0.337288`` |
| `2020-05-01T01:00:29.000Z` | `1ac87b74-0865-4a07-b512-56602a3a576f` | ``4.76562`` |

### <a name="example-b"></a>Exemple B

ID de série chronologique composite avec une propriété imbriquée\
**ID de série chronologique de l’environnement :** `"plantId"` et `"telemetry.tagId"`\
**Horodateur de la source de l’événement :** `"timestamp"`\
**Charge utile JSON :**

```JSON
[
    {
        "plantId": "9336971",
        "timestamp": "2020-01-22T16:38:09Z",
        "telemetry": [
            {
                "tagId": "100231-A-A6",
                "tagValue": -31.149018
            },
            {
                "tagId": "100231-A-A1",
                "tagValue": 20.560796
            },
            {
                "tagId": "100231-A-A9",
                "tagValue": 177
            },
            {
                "tagId": "100231-A-A8",
                "tagValue": 420
            },
        ]
    },
    {
        "plantId": "9336971",
        "timestamp": "2020-01-22T16:42:14Z",
        "telemetry": [
            {
                "tagId": "103585-A-A7",
                "value": -30.9918
            },
            {
                "tagId": "103585-A-A4",
                "value": 19.960796
            }
        ]
    }
]
```

**Résultat dans le fichier Parquet :** \
La configuration et la charge utile ci-dessus génèrent quatre colonnes et six événements

| timestamp  | plantId_string | telemetry.tagId_string | telemetry.value_double
| ---- | ---- | ---- | ---- |
| `2020-01-22T16:38:09Z` | `9336971`| ``100231-A-A6`` |  -31.149018 |
| `2020-01-22T16:38:09Z` |`9336971` | ``100231-A-A1`` | 20.560796 |
| `2020-01-22T16:38:09Z` | `9336971` | ``100231-A-A9`` | 177 |
| `2020-01-22T16:38:09Z` | `9336971` | ``100231-A-A8`` | 420 |
| `2020-01-22T16:42:14Z` | `9336971` | ``100231-A-A7`` | -30.9918 |  
| `2020-01-22T16:42:14Z` | `9336971` | ``100231-A-A4`` | 19.960796 |

### <a name="example-c"></a>Exemple C

L’ID et l’horodateur de la série chronologique se trouvent à la racine de l’objet\
**ID de série chronologique de l’environnement :** `"id"`\
**Horodateur de la source de l’événement :** `"timestamp"`\
**Charge utile JSON :**

```JSON
{
    "id": "800500054755",
    "timestamp": "2020-11-01T10:00:00.000Z",
    "datapoints": [{
            "value": 120
        },
        {
            "value": 124
        }
    ]
}
```

**Résultat dans le fichier Parquet :** \
La configuration et la charge utile ci-dessus génèrent trois colonnes et un événement

| timestamp  | id_string | datapoints_dynamic  
| ---- | ---- | ---- |
| `2020-11-01T10:00:00.000Z` | `800500054755`| ``[{"value": 120},{"value":124}]`` |

## <a name="next-steps"></a>Étapes suivantes

* Comprendre les [limites de débit](./concepts-streaming-ingress-throughput-limits.md) de votre environnement