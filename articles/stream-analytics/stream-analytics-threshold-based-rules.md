---
title: Règles configurables basées sur un seuil dans Azure Stream Analytics
description: Cet article explique comment utiliser des données de référence pour obtenir une solution d’alerte comportant des règles configurables basées sur un seuil dans Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 04/30/2018
ms.openlocfilehash: 215835bf7f1e6676adba6541da70dcb86fc3500c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86039039"
---
# <a name="process-configurable-threshold-based-rules-in-azure-stream-analytics"></a>Traiter des règles configurables basées sur un seuil dans Azure Stream Analytics
Cet article explique comment utiliser des données de référence pour obtenir une solution d’alerte utilisant des règles configurables basées sur un seuil dans Azure Stream Analytics.

## <a name="scenario-alerting-based-on-adjustable-rule-thresholds"></a>Scénario : génération d’alertes basées sur des seuils réglables
Vous devrez peut-être générer une alerte lorsque des événements entrants en flux continu atteignent une certaine valeur, ou lorsqu’une valeur agrégée basée sur les événements entrants en flux continu dépasse un certain seuil. Vous pouvez facilement définir une requête Stream Analytics qui compare une valeur à un seuil statique fixe et prédéterminé. Un seuil fixe peut être codé en dur dans la syntaxe de requête de diffusion en continu en utilisant de simples comparaisons numériques (supérieur à, inférieur à et égal à).

Dans certains cas, les valeurs de seuil doivent être plus facilement configurables, sans avoir à modifier la syntaxe de la requête chaque fois qu’une valeur de seuil est modifiée. Dans d’autres cas, vous devrez peut-être traiter un grand nombre d’appareils ou d’utilisateurs avec la même requête, en utilisant à chaque fois des valeurs de seuil différentes sur chaque type d’appareil. 

Ce modèle peut être utilisé pour configurer dynamiquement des seuils, sélectionner le type d’appareil auquel le seuil s’applique en filtrant les données d’entrée, et choisir les champs à inclure dans le résultat.

## <a name="recommended-design-pattern"></a>Modèle de conception recommandé
Utilisez une entrée de données de référence d’une tâche Stream Analytics pour rechercher les seuils d’alerte :
- Stockez les valeurs de seuil dans des données de référence, une valeur par clé.
- Joignez les événements d’entrée de données de diffusion en continu aux données de référence sur la colonne clé.
- Utilisez la valeur indexée provenant des données de référence comme valeur de seuil.

## <a name="example-data-and-query"></a>Exemples de données et de requête
Dans cet exemple, des alertes sont générées lorsque l’agrégation des données de diffusion en continu provenant d’appareils dans une fenêtre d’une minute correspond aux valeurs définies dans la règle fournie comme données de référence.

Dans la requête, pour chaque propriété deviceId et chaque valeur metricName sous deviceId, vous pouvez configurer de 0 à 5 dimensions à regrouper (GROUP BY). Seuls les événements dont les valeurs de filtre correspondent sont regroupés. Une fois ces événements regroupés, les agrégations fenêtrées Min, Max et Avg, sont calculées sur une fenêtre bascule de 60 secondes. Les filtres sur les valeurs d’agrégation sont ensuite calculés selon le seuil configuré dans la référence afin de générer l’événement d’alerte.

Prenons par exemple une tâche Stream Analytics avec une entrée de données de référence nommée **rules** et une entrée de données de diffusion en continu nommée **metrics**. 

## <a name="reference-data"></a>Données de référence
Cet exemple de données de référence montre comment une règle de seuil peut être représentée. Un fichier JSON contenant les données de référence est enregistré dans le stockage d’objets blob Azure, et ce conteneur de stockage d’objets blob est utilisé comme une entrée de données de référence nommée **rules**. Vous pouvez remplacer ce fichier JSON ainsi que la configuration de la règle au fil du temps, sans arrêter ou démarrer la tâche de diffusion en continu.

- L’exemple de règle sert à représenter une alerte réglable lorsque le processeur dépasse (moyenne supérieure ou égale à) la valeur de pourcentage `90`. Le champ `value` est configurable selon les besoins.
- Notez que la règle comporte un champ **opérateur**, qui est ensuite interprété dynamiquement dans la syntaxe de la requête avec `AVGGREATEROREQUAL`. 
- La règle filtre les données selon une certaine clé de dimension `2` avec la valeur `C1`. Les autres champs affichent une chaîne vide, indiquant que le flux d’entrée ne sera pas filtré selon ces champs d’événements. Vous pouvez configurer d’autres règles de processeur afin de filtrer d’autres champs correspondants, selon vos besoins.
- Toutes les colonnes ne seront incluses dans l’événement d’alerte de sortie. Dans ce cas, la clé `includedDim` numéro `2` est définie sur `TRUE` pour indiquer que le champ numéro 2 des données d’événement dans le flux de données sera inclus dans les événements de sortie correspondants. Les autres champs ne sont pas inclus dans la sortie d’alerte, mais la liste des champs peut être ajustée.


```json
{
    "ruleId": 1234, 
    "deviceId" : "978648", 
    "metricName": "CPU", 
    "alertName": "hot node AVG CPU over 90",
    "operator" : "AVGGREATEROREQUAL",
    "value": 90, 
    "includeDim": {
        "0": "FALSE", 
        "1": "FALSE", 
        "2": "TRUE", 
        "3": "FALSE", 
        "4": "FALSE"
    },
    "filter": {
        "0": "", 
        "1": "",
        "2": "C1", 
        "3": "", 
        "4": ""
    }    
}
```

## <a name="example-streaming-query"></a>Exemple de requête de diffusion en continu
Cet exemple de requête Stream Analytics regroupe les données de référence **rules** de l’exemple ci-dessus avec un flux d’entrée de données nommé **metrics**.

```sql
WITH transformedInput AS
(
    SELECT
        dim0 = CASE rules.includeDim.[0] WHEN 'TRUE' THEN metrics.custom.dimensions.[0].value ELSE NULL END,
        dim1 = CASE rules.includeDim.[1] WHEN 'TRUE' THEN metrics.custom.dimensions.[1].value ELSE NULL END,
        dim2 = CASE rules.includeDim.[2] WHEN 'TRUE' THEN metrics.custom.dimensions.[2].value ELSE NULL END,
        dim3 = CASE rules.includeDim.[3] WHEN 'TRUE' THEN metrics.custom.dimensions.[3].value ELSE NULL END,
        dim4 = CASE rules.includeDim.[4] WHEN 'TRUE' THEN metrics.custom.dimensions.[4].value ELSE NULL END,
        metric = metrics.metric.value,
        metricName = metrics.metric.name,
        deviceId = rules.deviceId, 
        ruleId = rules.ruleId, 
        alertName = rules.alertName,
        ruleOperator = rules.operator, 
        ruleValue = rules.value
    FROM 
        metrics
        timestamp by eventTime
    JOIN 
        rules
        ON metrics.deviceId = rules.deviceId AND metrics.metric.name = rules.metricName
    WHERE
        (rules.filter.[0] = '' OR metrics.custom.filters.[0].value = rules.filter.[0]) AND 
        (rules.filter.[1] = '' OR metrics.custom.filters.[1].value = rules.filter.[1]) AND
        (rules.filter.[2] = '' OR metrics.custom.filters.[2].value = rules.filter.[2]) AND
        (rules.filter.[3] = '' OR metrics.custom.filters.[3].value = rules.filter.[3]) AND
        (rules.filter.[4] = '' OR metrics.custom.filters.[4].value = rules.filter.[4])
)

SELECT
    System.Timestamp as time, 
    transformedInput.deviceId as deviceId,
    transformedInput.ruleId as ruleId,
    transformedInput.metricName as metric,
    transformedInput.alertName as alert,
    AVG(metric) as avg,
    MIN(metric) as min, 
    MAX(metric) as max, 
    dim0, dim1, dim2, dim3, dim4
FROM
    transformedInput
GROUP BY
    transformedInput.deviceId,
    transformedInput.ruleId,
    transformedInput.metricName,
    transformedInput.alertName,
    dim0, dim1, dim2, dim3, dim4,
    ruleOperator, 
    ruleValue, 
    TumblingWindow(second, 60)
HAVING
    (
        (ruleOperator = 'AVGGREATEROREQUAL' AND avg(metric) >= ruleValue) OR
        (ruleOperator = 'AVGEQUALORLESS' AND avg(metric) <= ruleValue) 
    )
```

## <a name="example-streaming-input-event-data"></a>Exemple de données d’événement d’entrée de diffusion en continu
Cet exemple de données JSON représente les données d’entrée **metrics** utilisées dans la requête de diffusion en continu ci-dessus. 

- Trois exemples d’événements sont répertoriés dans l’intervalle de 1 minute, avec la valeur `T14:50`. 
- Ces trois exemples ont la même valeur `deviceId``978648`.
- Les valeurs de mesure du processeur varient dans chaque événement, `98`, `95` et `80`, respectivement. Seuls les deux premiers exemples d’événements dépassent la règle d’alerte de processeur établie dans la règle.
- Le champ includeDim de la règle d’alerte affiche la clé numéro 2. Le champ de la clé 2 correspondant dans les exemples d’événements se nomme `NodeName`. Les trois exemples d’événements affichent les valeurs `N024`, `N024` et `N014`, respectivement. Dans la sortie, vous voyez uniquement le nœud `N024` car ce sont les seules données correspondant aux critères d’alerte en cas d’utilisation excessive du processeur. `N014` n’atteint pas le seuil d’utilisation excessive du processeur.
- La règle d’alerte est configurée avec une valeur `filter` uniquement sur la clé numéro 2, qui correspond au champ `cluster` dans les exemples d’événements. Les trois exemples d’événements ont tous la valeur `C1` et correspondent aux critères de filtre.

```json
{
    "eventTime": "2018-04-30T14:50:23.1324132Z",
    "deviceId": "978648",
    "custom": {
        "dimensions": {
            "0": {
                "name": "NodeType",
                "value": "N1"
            },
            "1": {
                "name": "Cluster",
                "value": "C1"
            },
            "2": {
                "name": "NodeName",
                "value": "N024"
            }
        },
        "filters": {
            "0": {
                "name": "application",
                "value": "A1"
            },
            "1": {
                "name": "deviceType",
                "value": "T1"
            },
            "2": {
                "name": "cluster",
                "value": "C1"
            },
            "3": {
                "name": "nodeType",
                "value": "N1"
            }
        }
    },
    "metric": {
        "name": "CPU",
        "value": 98,
        "count": 1.0,
        "min": 98,
        "max": 98,
        "stdDev": 0.0
    }
}
{
    "eventTime": "2018-04-30T14:50:24.1324138Z",
    "deviceId": "978648",
    "custom": {
        "dimensions": {
            "0": {
                "name": "NodeType",
                "value": "N2"
            },
            "1": {
                "name": "Cluster",
                "value": "C1"
            },
            "2": {
                "name": "NodeName",
                "value": "N024"
            }
        },
        "filters": {
            "0": {
                "name": "application",
                "value": "A1"
            },
            "1": {
                "name": "deviceType",
                "value": "T1"
            },
            "2": {
                "name": "cluster",
                "value": "C1"
            },
            "3": {
                "name": "nodeType",
                "value": "N2"
            }
        }
    },
    "metric": {
        "name": "CPU",
        "value": 95,
        "count": 1,
        "min": 95,
        "max": 95,
        "stdDev": 0
    }
}
{
    "eventTime": "2018-04-30T14:50:37.1324130Z",
    "deviceId": "978648",
    "custom": {
        "dimensions": {
            "0": {
                "name": "NodeType",
                "value": "N3"
            },
            "1": {
                "name": "Cluster",
                "value": "C1 "
            },
            "2": {
                "name": "NodeName",
                "value": "N014"
            }
        },
        "filters": {
            "0": {
                "name": "application",
                "value": "A1"
            },
            "1": {
                "name": "deviceType",
                "value": "T1"
            },
            "2": {
                "name": "cluster",
                "value": "C1"
            },
            "3": {
                "name": "nodeType",
                "value": "N3"
            }
        }
    },
    "metric": {
        "name": "CPU",
        "value": 80,
        "count": 1,
        "min": 80,
        "max": 80,
        "stdDev": 0
    }
}
```

## <a name="example-output"></a>Exemple de sortie
Cet exemple de données JSON de sortie affiche un seul événement d’alerte généré en fonction de la règle de seuil de processeur définie dans les données de référence. L’événement de sortie contient le nom de l’alerte ainsi que l’agrégation (moyenne, min, max) des champs pris en compte. Les données d’événements de sortie incluent la clé de champ numéro 2 `NodeName` avec la valeur `N024` en raison de la configuration de la règle. (Le fichier JSON a été modifié afin d’afficher les sauts de ligne pour une meilleure lisibilité.)

```JSON
{"time":"2018-05-01T02:03:00.0000000Z","deviceid":"978648","ruleid":1234,"metric":"CPU",
"alert":"hot node AVG CPU over 90","avg":96.5,"min":95.0,"max":98.0,
"dim0":null,"dim1":null,"dim2":"N024","dim3":null,"dim4":null}
```
