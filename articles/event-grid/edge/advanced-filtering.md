---
title: Filtrage avancé – Azure Event Grid IoT Edge | Microsoft Docs
description: Filtrage avancé dans Event Grid sur IoT Edge.
author: HiteshMadan
manager: rajarv
ms.author: himad
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 64b8956c47cbdbf31bb8253dac0c1e1f12833bf7
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001046"
---
# <a name="advanced-filtering"></a>Filtrage avancé
Event Grid permet de spécifier des filtres sur n’importe quelle propriété de la charge utile JSON. Ces filtres sont modélisés sous la forme d’un ensemble de conditions `AND`, chaque condition externe comportant des conditions `OR` internes facultatives. Pour chaque condition `AND` sont spécifiées les valeurs suivantes :

* `OperatorType` : type de comparaison.
* `Key` : chemin JSON de la propriété sur laquelle sera appliqué le filtre.
* `Value` : valeur de référence du filtre (ou) `Values` : ensemble des valeurs de référence du filtre.

## <a name="json-syntax"></a>Syntaxe JSON

La syntaxe JSON d’un filtre avancé est la suivante :

```json
{
    "filter": {
        "advancedFilters": [{
                "operatorType": "NumberGreaterThanOrEquals",
                "key": "Data.Key1",
                "value": 5
            }, {
                "operatorType": "StringContains",
                "key": "Subject",
                "values": ["container1", "container2"]
            }
        ]
    }
}
```

## <a name="filtering-on-array-values"></a>Filtrage sur des valeurs de tableau

Event Grid ne prend pas en charge le filtrage sur un tableau de valeurs à l’heure actuelle. Si un événement entrant a pour clé du filtre avancé une valeur de tableau, l’opération de mise en correspondance échoue, en conséquence de quoi l’événement entrant ne correspond pas à l’abonnement aux événements.

## <a name="and-or-not-semantics"></a>Sémantique AND-OR-NOT

Comme vous l’avez probablement remarqué, `AdvancedFilters` est un tableau dans l’exemple JSON précédent. Considérez chacun des éléments de tableau `AdvancedFilter` comme une condition `AND`.

Dans le cas des opérateurs qui acceptent plusieurs valeurs (par exemple, `NumberIn`, `NumberNotIn` ou `StringIn`), chacune d’elles est traitée comme une condition `OR`. Ainsi, `StringBeginsWith("a", "b", "c")` correspond à toutes les valeurs de chaîne qui commencent par `a`, `b` ou `c`.

> [!CAUTION]
> Les opérateurs NOT (`NumberNotIn` et `StringNotIn`) se comportent comme des conditions AND sur chaque valeur donnée dans le champ `Values`.
>
> Sinon, le filtre deviendrait un filtre Accept-All (Tout accepter), ce qui irait à l’encontre du but recherché par le filtrage.

## <a name="floating-point-rounding-behavior"></a>Comportement d’arrondi à virgule flottante

Event Grid utilise le type .NET `decimal` pour gérer toutes les valeurs numériques. Les valeurs numériques spécifiées dans le JSON d’abonnement aux événements ne sont pas soumises au comportement d’arrondi à virgule flottante.

## <a name="case-sensitivity-of-string-filters"></a>Respect de la casse des filtres de chaîne

Aucune des comparaisons de chaînes n’est sensible à la casse. Il n’existe à l’heure actuelle aucun moyen de modifier ce comportement.

## <a name="allowed-advanced-filter-keys"></a>Clés de filtre avancées autorisées

La propriété `Key` peut être une propriété de niveau supérieur connue ou bien un chemin JSON comportant plusieurs points, dont chacun indique d’entrer dans un objet JSON imbriqué.

Event Grid ne donne pas de signification particulière au caractère `$` dans la clé, contrairement à la spécification JSONPath.

### <a name="event-grid-schema"></a>Schéma Event Grid

Dans le cas des événements du schéma Event Grid :

* id
* Rubrique
* Objet
* Type d’événement
* DataVersion
* Data.Prop1
* Data.Prop*Prop2.Prop3.Prop4.Prop5

### <a name="custom-event-schema"></a>Schéma d’événements personnalisé

Aucune restriction ne s’impose à `Key` dans un schéma d’événements personnalisé, car Event Grid n’applique pas de schéma d’enveloppe à la charge utile.

## <a name="numeric-single-value-filter-examples"></a>Exemples de filtres numériques à valeur unique

* NumberGreaterThan
* NumberGreaterThanOrEquals
* NumberLessThan
* NumberLessThanOrEquals

```json
{
    "filter": {
        "advancedFilters": [
            {
                "operatorType": "NumberGreaterThan",
                "key": "Data.Key1",
                "value": 5
            },
            {
                "operatorType": "NumberGreaterThanOrEquals",
                "key": "Data.Key2",
                "value": *456
            },
            {
                "operatorType": "NumberLessThan",
                "key": "Data.P*P2.P3",
                "value": 1000
            },
            {
                "operatorType": "NumberLessThanOrEquals",
                "key": "Data.P*P2",
                "value": 999
            }
        ]
    }
}
```

## <a name="numeric-range-value-filter-examples"></a>Exemples de filtres numériques de type plage

* NumberIn
* NumberNotIn

```json
{
    "filter": {
        "advancedFilters": [
            {
                "operatorType": "NumberIn",
                "key": "Data.Key1",
                "values": [1, 10, 100]
            },
            {
                "operatorType": "NumberNotIn",
                "key": "Data.Key2",
                "values": [2, 3, 4.56]
            }
        ]
    }
}
```

## <a name="string-range-value-filter-examples"></a>Exemples de filtres de chaîne de type plage

* StringContains
* StringBeginsWith
* StringEndsWith
* StringIn
* StringNotIn

```json
{
    "filter": {
        "advancedFilters": [
            {
                "operatorType": "StringContains",
                "key": "Data.Key1",
                "values": ["microsoft", "azure"]
            },
            {
                "operatorType": "StringBeginsWith",
                "key": "Data.Key2",
                "values": ["event", "grid"]
            },
            {
                "operatorType": "StringEndsWith",
                "key": "Data.P3.P4",
                "values": ["jpg", "jpeg", "png"]
            },
            {
                "operatorType": "StringIn",
                "key": "RootKey",
                "values": ["exact", "string", "matches"]
            },
            {
                "operatorType": "StringNotIn",
                "key": "RootKey",
                "values": ["aws", "bridge"]
            }
        ]
    }
}
```

## <a name="boolean-single-value-filter-examples"></a>Exemples de filtres booléens à valeur unique

* BoolEquals

```json
{
    "filter": {
        "advancedFilters": [
            {
                "operatorType": "BoolEquals",
                "key": "BoolKey1",
                "value": true
            },
            {
                "operatorType": "BoolEquals",
                "key": "BoolKey2",
                "value": false
            }
        ]
    }
}
```
