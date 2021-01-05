---
title: Guide de référence du schéma StylesObject pour les cartes Azure dynamiques
description: Guide de référence du schéma et de la syntaxe StylesObject pour les cartes Azure dynamiques.
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: reference
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: 08379e66c97d34eea53410190475e90e156a58e2
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96903341"
---
# <a name="stylesobject-schema-reference-guide-for-dynamic-maps"></a>Guide de référence du schéma StylesObject pour les cartes dynamiques

> [!IMPORTANT]
> Les services Azure Maps Creator sont disponibles en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

 `StylesObject` est un tableau de `StyleObject` représentant des styles de stateset. Utilisez le [service État de la fonctionnalité](/rest/api/maps/featurestate) d’Azure Maps Creator (préversion) pour appliquer vos styles StateSet à des caractéristiques de données de cartes d’intérieur. Une fois que vous avez créé vos styles de stateset et que vous les avez associés à des caractéristiques de cartes d’intérieur, vous pouvez les utiliser pour créer des cartes d’intérieur dynamiques. Pour plus d’informations sur la création de cartes d’intérieur dynamiques, consultez [Implémenter des styles dynamiques pour les cartes d’intérieur Creator](indoor-map-dynamic-styling.md).

## <a name="styleobject"></a>StyleObject

Un `StyleObject` est l’une des règles de style suivantes :

 * [`BooleanTypeStyleRule`](#booleantypestylerule)
 * [`NumericTypeStyleRule`](#numerictypestylerule)
 * [`StringTypeStyleRule`](#stringtypestylerule)

Le code JSON ci-dessous montre un exemple d’utilisation de chacun des trois types de style.  La règle `BooleanTypeStyleRule` est utilisée pour déterminer le style dynamique des fonctionnalités dont la propriété `occupied` a la valeur true et false.  La `NumericTypeStyleRule` est utilisée pour déterminer le style des fonctionnalités dont la propriété `temperature` se situe dans une certaine plage. Enfin, la `StringTypeStyleRule` est utilisé pour faire correspondre des styles spécifiques à `meetingType`.



```json
 "styles": [
     {
        "keyname": "occupied",
        "type": "boolean",
        "rules": [
            {
                "true": "#FF0000",
                "false": "#00FF00"
            }
        ]
    },
    {
        "keyname": "temperature",
        "type": "number",
        "rules": [
             {
                "range": {
                "minimum": 50,
                "exclusiveMaximum": 70
                },
                "color": "#343deb"
            },
            {
                "range": {
                "maximum": 70,
                "exclusiveMinimum": 30
              },
              "color": "#eba834"
            }
        ]
    },
    {
      "keyname": "meetingType",
      "type": "string",
      "rules": [
        {
          "private": "#FF0000",
          "confidential": "#FF00AA",
          "allHands": "#00FF00",
          "brownBag": "#964B00"
        }
      ]
    }
]
```

## <a name="numerictypestylerule"></a>NumericTypeStyleRule

 Un `NumericTypeStyleRule` est un [`StyleObject`](#styleobject) et est constitué des propriétés suivantes :

| Propriété | Type | Description | Obligatoire |
|-----------|----------|-------------|-------------|
| `keyName` | string | *État* ou nom de propriété dynamique. Un `keyName` doit être unique au sein du tableau `StyleObject`.| Oui |
| `type` | string | La valeur est « numérique ». | Oui |
| `rules` | [`NumberRuleObject`](#numberruleobject)[]| Tableau de plages de style numériques avec les couleurs associées. Chaque plage définit une couleur à utiliser quand la valeur de l’*état* correspond à la plage.| Oui |

### <a name="numberruleobject"></a>NumberRuleObject

Un `NumberRuleObject` se compose d’un [`RangeObject`](#rangeobject) et d’une propriété `color`. Si la valeur d’*état* est comprise dans la plage, sa couleur d’affichage sera celle spécifiée dans la propriété `color`.

Si vous définissez plusieurs plages qui se chevauchent, la couleur choisie est celle qui est définie dans la première plage satisfaite.

Dans l’exemple JSON suivant, les deux plages seront satisfaites quand la valeur d’*état* sera comprise entre 50 et 60. Toutefois, la couleur utilisée sera `#343deb`, car il s’agit de la première plage de la liste qui a été satisfaite.

```json

    {
        "rules":[
            {
                "range": {
                "minimum": 50,
                "exclusiveMaximum": 70
                },
                "color": "#343deb"
            },
            {
                "range": {
                "minimum": 50,
                "maximum": 60
                },
                "color": "#eba834"
            }
        ]
    }
]
```

| Propriété | Type | Description | Obligatoire |
|-----------|----------|-------------|-------------|
| `range` | [RangeObject](#rangeobject) | [RangeObject](#rangeobject) définit un ensemble de conditions de plage logique qui, si `true`, modifient la couleur d’affichage de l’*état* en lui affectant celle spécifiée dans la propriété `color`. Si `range` n’est pas spécifié, la couleur définie dans la propriété `color` sera toujours utilisée.   | Non |
| `color` | string | Couleur à utiliser quand la valeur d’état est comprise dans la plage. La propriété `color` est une chaîne JSON ayant l’un des formats suivants : <ul><li> Valeurs hexadécimales de style HTML </li><li> RGB (« #ff0 », « #ffff00 », « rgb(255, 255, 0) »)</li><li> RGBA (« rgba(255, 255, 0, 1) »)</li><li> HSL(« hsl(100, 50%, 50%) »)</li><li> HSLA(« hsla(100, 50%, 50%, 1) »)</li><li> Des noms de couleurs HTML prédéfinis, tels que le jaune et le bleu.</li></ul> | Oui |

### <a name="rangeobject"></a>RangeObject

`RangeObject` définit une valeur de plage numérique d’un [`NumberRuleObject`](#numberruleobject). Pour que la valeur d’*état* soit comprise dans la plage, toutes les conditions définies doivent être remplies.

| Propriété | Type | Description | Obligatoire |
|-----------|----------|-------------|-------------|
| `minimum` | double | Tout nombre x où x ≥ `minimum`.| Non |
| `maximum` | double | Tout nombre x où x ≤ `maximum`. | Non |
| `exclusiveMinimum` | double | Tout nombre x où x > `exclusiveMinimum`.| Non |
| `exclusiveMaximum` | double | Tout nombre x où x < `exclusiveMaximum`.| Non |

### <a name="example-of-numerictypestylerule"></a>Exemple de NumericTypeStyleRule

Le code JSON suivant illustre un *état* `NumericTypeStyleRule` nommé `temperature`. Dans cet exemple, le [`NumberRuleObject`](#numberruleobject) contient deux plages de température définies et les styles de couleur qui leur sont associés. Si la plage de températures est 50-69, l’affichage doit utiliser la couleur `#343deb`.  Si la plage de températures est 31-70, l’affichage doit utiliser la couleur `#eba834`.

```json
{
    "keyname": "temperature",
    "type": "number",
    "rules":[
        {
            "range": {
            "minimum": 50,
            "exclusiveMaximum": 70
            },
            "color": "#343deb"
        },
        {
            "range": {
            "maximum": 70,
            "exclusiveMinimum": 30
            },
            "color": "#eba834"
        }
    ]
}
```

## <a name="stringtypestylerule"></a>StringTypeStyleRule

Un `StringTypeStyleRule` est un [`StyleObject`](#styleobject) et est constitué des propriétés suivantes :

| Propriété | Type | Description | Obligatoire |
|-----------|----------|-------------|-------------|
| `keyName` | string |  *État* ou nom de propriété dynamique.  Un `keyName` doit être unique au sein du tableau `StyleObject`.| Oui |
| `type` | string |La valeur est « string ». | Oui |
| `rules` | [`StringRuleObject`](#stringruleobject)[]| Tableau d’un nombre N de valeurs d’*état*.| Oui |

### <a name="stringruleobject"></a>StringRuleObject

Un `StringRuleObject` est constitué d’un nombre maximum de N valeurs d’état qui sont les valeurs de chaîne possibles de la propriété d’une fonctionnalité. Si la valeur de propriété de la fonctionnalité ne correspond à aucune des valeurs d’état définies, cette fonctionnalité n’aura pas de style dynamique. Si des valeurs d’état dupliquées sont fournies, la première est prioritaire.

La correspondance des valeurs des chaînes respecte la casse.

| Propriété | Type | Description | Obligatoire |
|-----------|----------|-------------|-------------|
| `stateValue1` | string | Couleur lorsque la chaîne de valeur est stateValue1. | Non |
| `stateValue2` | string | Couleur lorsque la chaîne de valeur est stateValue. | Non |
| `stateValueN` | string | Couleur lorsque la chaîne de valeur est stateValueN. | Non |

### <a name="example-of-stringtypestylerule"></a>Exemple de StringTypeStyleRule

Le code JSON suivant illustre une `StringTypeStyleRule` qui définit des styles associés à des types de réunion spécifiques.

```json
    {
      "keyname": "meetingType",
      "type": "string",
      "rules": [
        {
          "private": "#FF0000",
          "confidential": "#FF00AA",
          "allHands": "#00FF00",
          "brownBag": "#964B00"
        }
      ]
    }

```

## <a name="booleantypestylerule"></a>BooleanTypeStyleRule

Un `BooleanTypeStyleRule` est un [`StyleObject`](#styleobject) et est constitué des propriétés suivantes :

| Propriété | Type | Description | Obligatoire |
|-----------|----------|-------------|-------------|
| `keyName` | string |  *État* ou nom de propriété dynamique.  Un `keyName` doit être unique au sein du tableau `StyleObject`.| Oui |
| `type` | string |La valeur est « booléenne ». | Oui |
| `rules` | [`BooleanRuleObject`](#booleanruleobject)[1]| Paire booléenne avec des couleurs pour les valeurs d *’état* `true` et `false`.| Oui |

### <a name="booleanruleobject"></a>BooleanRuleObject

Un `BooleanRuleObject` définit des couleurs pour les valeurs `true` et `false`.

| Propriété | Type | Description | Obligatoire |
|-----------|----------|-------------|-------------|
| `true` | string | Couleur à utiliser quand la valeur d’*état* est `true`. La propriété `color` est une chaîne JSON ayant l’un des formats suivants : <ul><li> Valeurs hexadécimales de style HTML </li><li> RGB (« #ff0 », « #ffff00 », « rgb(255, 255, 0) »)</li><li> RGBA (« rgba(255, 255, 0, 1) »)</li><li> HSL(« hsl(100, 50%, 50%) »)</li><li> HSLA(« hsla(100, 50%, 50%, 1) »)</li><li> Des noms de couleurs HTML prédéfinis, tels que le jaune et le bleu.</li></ul>| Oui |
| `false` | string | Couleur à utiliser quand la valeur d’*état* est `false`. | Oui |

### <a name="example-of-booleantypestylerule"></a>Exemple de BooleanTypeStyleRule

Le code JSON suivant illustre un *état* `BooleanTypeStyleRule` nommé `occupied`. Le [`BooleanRuleObject`](#booleanruleobject) définit des couleurs pour les valeurs `true` et `false`.

```json
{
    "keyname": "occupied",
    "type": "boolean",
    "rules": [
    {
        "true": "#FF0000",
        "false": "#00FF00"
    }
    ]
}
```