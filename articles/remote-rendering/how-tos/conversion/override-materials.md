---
title: Remplacer des matériaux pendant la conversion d’un modèle
description: Explique le workflow de remplacement de matériaux au moment de la conversion
author: florianborn71
ms.author: flborn
ms.date: 02/13/2020
ms.topic: how-to
ms.openlocfilehash: 11bd79a1bc88d2605a20744f5a6b6536d754c100
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91576640"
---
# <a name="override-materials-during-model-conversion"></a>Remplacer des matériaux pendant la conversion d’un modèle

Les paramètres de matériau dans le modèle source sont utilisés pour définir les [matériaux PBR](../../overview/features/pbr-materials.md) utilisés par le renderer.
Parfois, la [conversion par défaut](../../reference/material-mapping.md) ne donne pas les résultats souhaités et vous devez apporter des modifications.
Quand un modèle est converti pour être utilisé dans Azure Remote Rendering, vous pouvez fournir un fichier de remplacement de matériau pour personnaliser la façon dont la conversion est effectuée pour chaque matériau.
Si un fichier appelé `<modelName>.MaterialOverrides.json` se trouve dans le conteneur d’entrée à côté du modèle d’entrée `<modelName>.<ext>`, il sera utilisé comme fichier de remplacement de matériau.

## <a name="the-override-file-used-during-conversion"></a>Fichier de remplacement utilisé pendant la conversion

Pour illustrer cela de façon simple, imaginons qu’un modèle de boîte possède un seul matériau appelé « Default ».
Par ailleurs, supposons que la couleur albedo doit être ajustée pour une utilisation dans ARR.
Dans ce cas, un fichier `box.MaterialOverrides.json` peut être créé comme suit :

```json
[
    {
        "name": "Default",
        "albedoColor": {
            "r": 0.33,
            "g": 0.33,
            "b": 0.33,
            "a": 1.0
        }
    }
]
```

Le fichier `box.MaterialOverrides.json` est placé dans le conteneur d’entrée à côté de `box.fbx`, ce qui indique au service de conversion d’appliquer les nouveaux paramètres.

### <a name="color-materials"></a>Matériaux couleur

Le modèle de [matériau de couleur](../../overview/features/color-materials.md) décrit une surface constamment ombrée indépendante de l’éclairage.
Les matériaux couleur s’avèrent utiles pour les composants créés avec des algorithmes de photogrammétrie, par exemple.
Dans les fichiers de remplacement de matériau, vous pouvez déclarer un matériau comme matériau de couleur en définissant `unlit` sur `true`.

```json
[
    {
        "name": "Photogrametry_mat1",
        "unlit" : true
    },
    {
        "name": "Photogrametry_mat2",
        "unlit" : true
    }
]
```

### <a name="ignore-specific-texture-maps"></a>Ignorer des cartes de texture spécifiques

Dans certains cas, vous souhaiterez peut-être que le processus de conversion ignore des cartes de texture spécifiques, notamment quand votre modèle a été généré par un outil qui crée des cartes spéciales que le renderer ne comprend pas correctement. Il peut s’agir, par exemple, d’une « OpacityMap » utilisée pour définir autre chose que l’opacité ou d’un modèle où la « NormalMap » est stockée comme « BumpMap ». (Dans ce dernier cas, vous devrez ignorer « NormalMap ». Ainsi, le convertisseur utilisera « BumpMap » comme « NormalMap ».)

Le principe est simple. Ajoutez simplement une propriété appelée `ignoreTextureMaps` et ajoutez les cartes de texture que vous souhaitez ignorer :

```json
[
    {
        "name": "Default",
        "ignoreTextureMaps": ["OpacityMap", "NormalMap"]
    }
]
```

Pour obtenir la liste complète des cartes de texture que vous pouvez ignorer, consultez le schéma JSON ci-dessous.

### <a name="applying-the-same-overrides-to-multiple-materials"></a>Application des mêmes remplacements à plusieurs matériaux

Par défaut, une entrée dans le fichier de remplacement de matériau s’applique lorsque son nom correspond exactement au nom du matériau.
Étant donné qu’il est assez courant que le même remplacement s’applique à plusieurs matériaux, vous pouvez éventuellement fournir une expression régulière comme nom d’entrée.
Le champ `nameMatching` a une valeur par défaut `exact`, mais il peut être défini sur `regex` pour indiquer que l’entrée doit s’appliquer à chaque matériau correspondant.
La syntaxe utilisée est la même que celle utilisée pour JavaScript. L’exemple suivant montre un remplacement qui s’applique aux matériaux portant des noms tels que « Material2 », « Material01 » et « Material999 ».

```json
[
    {
        "name": "Material[0-9]+",
        "nameMatching": "regex",
        "albedoColor": {
            "r": 0.0,
            "g": 0.0,
            "b": 1.0,
            "a": 1.0
        }
    }
]
```

Au maximum une entrée dans un fichier de remplacement de matériau s’applique à un matériau.
S’il existe une correspondance exacte (c’est-à-dire que `nameMatching` est absent ou égal à `exact`) pour le nom du matériau, cette entrée est celle choisie.
Dans le cas contraire, la première entrée de regex du fichier qui correspond au nom du matériau est choisie.

### <a name="getting-information-about-which-entries-applied"></a>Obtention d’informations sur les entrées appliquées

Le [fichier d’informations](get-information.md#information-about-a-converted-model-the-info-file) qui est écrit dans le conteneur de sortie contient des informations sur le nombre de remplacements fournis et le nombre de matériaux qui ont été remplacés.

## <a name="json-schema"></a>Schéma JSON

Le schéma JSON complet pour les fichiers de matériaux est fourni ici. À l’exception de `unlit` et `ignoreTextureMaps`, les propriétés disponibles sont un sous-ensemble des propriétés décrites dans les sections sur les modèles de [matériau de couleur](../../overview/features/color-materials.md) et de [matériau PBR](../../overview/features/pbr-materials.md).

```json
{
    "definitions" :
    {
        "color":
        {
            "type" : "object",
            "description" : "Color as 4 components vector",
            "properties":
            {
                "r": {"type":"number"},
                "g": {"type":"number"},
                "b": {"type":"number"},
                "a": {"type":"number"}
            },
            "required": ["r", "g", "b"]
        },
        "alpha":
        {
            "type" : "object",
            "description" : "Alpha channel for color",
            "properties":
            {
                "a": {"type":"number"}
            },
            "required": ["a"]
        },
        "colorOrAlpha":
        {
            "anyOf": [
                {"$ref": "#/definitions/color"},
                {"$ref": "#/definitions/alpha"}
            ]
        },
        "listOfMaps":
        {
            "type": "array",
            "items": {
                "type": "string",
                "enum": ["AlbedoMap",
                            "EmissiveMap",
                            "NormalMap",
                            "OcclusionMap",
                            "RoughnessMap",
                            "MetalnessMap",
                            "ReflectivityMap",
                            "BumpMap",
                            "OpacityMap",
                            "DiffuseMap",
                            "SpecularMap",
                            "ShininessMap",
                            "MetallicRoughnessMap",
                            "SpecularGlossinessMap"]
            }
        }
    },
    "type" : "array",
    "description" : "List of materials to override",
    "items":
    {
        "type" : "object",
        "description" : "List of parameters to override",
        "properties":
        {
            "name": { "type" : "string"},
            "nameMatching" : { "type" : "string", "enum" : ["exact", "regex"] },
            "unlit": { "type" : "boolean" },
            "albedoColor": { "$ref": "#/definitions/colorOrAlpha" },
            "roughness": { "type": "number" },
            "metalness": { "type": "number" },
            "transparent": { "type" : "boolean" },
            "alphaClipEnabled": { "type" : "boolean" },
            "alphaClipThreshold": { "type": "number" },
            "useVertexColor": { "type" : "boolean" },
            "isDoubleSided": { "type" : "boolean" },
            "ignoreTextureMaps": { "$ref" : "#/definitions/listOfMaps" }
        },
        "required": ["name"],
        "additionalProperties" : false
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

* [Matériaux couleur](../../overview/features/color-materials.md)
* [Matériaux PBR](../../overview/features/pbr-materials.md)
