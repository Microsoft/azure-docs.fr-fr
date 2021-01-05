---
title: Obtenir des informations sur les conversions
description: Obtenir des informations sur les conversions
author: malcolmtyrrell
ms.author: matyrr
ms.date: 03/05/2020
ms.topic: how-to
ms.openlocfilehash: 89ec0ad40822785457e988cf9e0f9bd6d00ed81f
ms.sourcegitcommit: a422b86148cba668c7332e15480c5995ad72fa76
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/30/2020
ms.locfileid: "91576623"
---
# <a name="get-information-about-conversions"></a>Obtenir des informations sur les conversions

## <a name="information-about-a-conversion-the-result-file"></a>Informations sur une conversion : Le fichier de résultats

Lorsque le service de conversion convertit un élément multimédia, il écrit un résumé de tous les problèmes dans un « fichier de résultats ». Par exemple, si un fichier `buggy.gltf` est converti, le conteneur de sortie contient un fichier nommé `buggy.result.json`.

Le fichier de résultats répertorie les erreurs et les avertissements qui se sont produits pendant la conversion et fournit un résumé des résultats : `succeeded`, `failed` ou `succeeded with warnings`.
Le fichier de résultats est structuré sous la forme d’un tableau JSON d’objets, dont chacun a une propriété de type chaîne : `warning`,`error`, `internal warning`, `internal error` et `result`. Il y aura au plus une erreur (`error` ou `internal error`) et il y aura toujours un `result`.

## <a name="example-result-file"></a>Exemple de fichier de *résultats*

L’exemple suivant décrit une conversion qui a généré avec succès un arrAsset. Toutefois, étant donné qu’il existait une texture manquante, le arrAsset résultant ne peut pas être le même que prévu.

```JSON
[
  {"warning":"4004","title":"Missing texture","details":{"texture":"buggy_baseColor.png","material":"buggy_col"}},
  {"result":"succeeded with warnings"}
]
```

## <a name="information-about-a-converted-model-the-info-file"></a>Informations sur un modèle converti : Le fichier d’informations

Le fichier arrAsset produit par le service de conversion est destiné exclusivement à l’usage du service de rendu. Toutefois, il peut vous arriver de vouloir accéder à des informations sur un modèle sans démarrer de session de rendu. Pour prendre en charge ce workflow, le service de conversion place un fichier JSON à côté du fichier arrAsset dans le conteneur de sortie. Par exemple, si un fichier `buggy.gltf` est converti, le conteneur de sortie contient un fichier nommé `buggy.info.json` en regard de la ressource convertie `buggy.arrAsset`. Ce fichier contient des informations sur le modèle source, le modèle converti et la conversion proprement dite.

## <a name="example-info-file"></a>Exemple de fichier *info*

Voici un exemple de fichier *info* produit par la conversion d’un fichier nommé `buggy.gltf` :

```JSON
{
    "files": {
        "input": "Buggy.gltf"
    },
    "conversionSettings": {
        "recenterToOrigin": true
    },
    "inputInfo": {
        "sourceAssetExtension": ".gltf",
        "sourceAssetFormat": "glTF2 Importer",
        "sourceAssetFormatVersion": "2.0",
        "sourceAssetGenerator": "COLLADA2GLTF"
    },
    "inputStatistics": {
        "numMeshes": 148,
        "numFaces": 308306,
        "numVertices": 245673,
        "numMaterial": 149,
        "numFacesSmallestMesh": 2,
        "numFacesBiggestMesh": 8764,
        "numNodes": 206,
        "numMeshUsagesInScene": 236,
        "maxNodeDepth": 3
    },
    "materialOverrides": {
        "numOverrides": 4,
        "numOverriddenMaterials": 4
    },
    "outputInfo": {
        "conversionToolVersion": "3b28d840de9916f9d628342f474d38c3ab949590",
        "conversionHash": "CCDB1F7A4C09F565"
    },
    "outputStatistics": {
        "numMeshPartsCreated": 236,
        "numMeshPartsInstanced": 88,
        "recenteringOffset": [
            -24.1,
            -50.9,
            -16.5974
        ],
        "boundingBox": {
            "min": [
                -43.52,
                -61.775,
                -79.6416
            ],
            "max": [
                43.52,
                61.775,
                79.6416
            ]
        }
    }
}
```

## <a name="information-in-the-info-file"></a>Informations contenues dans le fichier info

### <a name="the-files-section"></a>Section *files*

Cette section contient les noms de fichiers fournis.

* `input`: Nom du fichier source.
* `output`: Nom du fichier de sortie, quand l’utilisateur a spécifié un nom autre que celui par défaut.

### <a name="the-conversionsettings-section"></a>Section *conversionSettings*

Cette section contient une copie des paramètres [ConversionSettings](configure-model-conversion.md#settings-file) spécifiés lors de la conversion du modèle.

### <a name="the-inputinfo-section"></a>Section *inputInfo*

Cette section enregistre des informations sur le format du fichier source.

* `sourceAssetExtension`: Extension de fichier du fichier source.
* `sourceAssetFormat`: Description du format du fichier source.
* `sourceAssetFormatVersion`: Version du format du fichier source.
* `sourceAssetGenerator`: Nom de l’outil qui a généré le fichier source, s’il est disponible.

### <a name="the-materialoverrides-section"></a>Section *materialOverrides*

Cette section fournit des informations sur le [remplacement matériel](override-materials.md) lorsqu’un fichier de remplacement matériel a été fourni au service de conversion.
Il contient les informations suivantes :
* `numOverrides` : Nombre d’entrées de remplacement lues à partir du fichier de remplacement matériel.
* `numOverriddenMaterials` : Nombre de matériaux qui ont été remplacés.

### <a name="the-inputstatistics-section"></a>Section *inputStatistics*

Cette section fournit des informations sur la scène source. Il y a souvent des discordances entre les valeurs de cette section et les valeurs équivalentes dans l’outil qui a créé le modèle source. De telles différences sont attendues, car le modèle est modifié pendant les étapes d’exportation et de conversion.

* `numMeshes`: Nombre de composants du maillage, où chaque composant peut référencer un seul matériau.
* `numFaces`: Nombre total de _triangles_ dans le modèle. Notez que le maillage est triangulé lors de la conversion. Ce nombre contribue à la limite de polygones dans la [taille de serveur de rendu standard](../../reference/vm-sizes.md#how-the-renderer-evaluates-the-number-of-polygons).
* `numVertices`: Nombre total de vertex dans le modèle.
* `numMaterial`: Nombre total de matériaux dans le modèle.
* `numFacesSmallestMesh`: Nombre de triangles dans le plus petit maillage du modèle.
* `numFacesBiggestMesh`: Nombre de triangles dans le plus grand maillage du modèle.
* `numNodes`: Nombre de nœuds dans le graphique de scène du modèle.
* `numMeshUsagesInScene`: Nombre de fois que des nœuds référencent des maillages. Plusieurs nœuds peuvent référencer le même maillage.
* `maxNodeDepth`: Profondeur maximale des nœuds dans le graphique de scène.

### <a name="the-outputinfo-section"></a>Section *outputInfo*

Cette section enregistre des informations générales sur la sortie générée.

* `conversionToolVersion`: Version du convertisseur de modèle.
* `conversionHash`: Hachage des données dans l’arrAsset, qui peut contribuer au rendu. Peut être utilisé pour déterminer si le service de conversion a produit un résultat différent lors de la réexécution sur le même fichier.

### <a name="the-outputstatistics-section"></a>Section *outputStatistics*

Cette section enregistre les informations calculées à partir de la ressources converti.

* `numMeshPartsCreated`: Nombre de maillages dans l’arrAsset. Il peut différer de la valeur `numMeshes` dans la section `inputStatistics`, car l’instanciation est affectée par le processus de conversion.
* `numMeshPartsInstanced`: Nombre de maillages réutilisés dans l’arrAsset.
* `recenteringOffset`: Quand l’option `recenterToOrigin` dans la section [ConversionSettings](configure-model-conversion.md) est activée, cette valeur indique la translation qui ramènerait le modèle converti dans sa position d’origine.
* `boundingBox`: Limites du modèle.

## <a name="deprecated-features"></a>Fonctionnalités dépréciées

Le service de conversion écrit les fichiers `stdout.txt` et `stderr.txt` dans le conteneur de sortie qui étaient la seule source d’avertissements et d’erreurs.
Ces fichiers sont désormais déconseillés. Au lieu de cela, utilisez des [fichiers de résultats](#information-about-a-conversion-the-result-file) à cet effet.

## <a name="next-steps"></a>Étapes suivantes

* [Conversion de modèle](model-conversion.md)
* [Configurer la conversion de modèle](configure-model-conversion.md)
