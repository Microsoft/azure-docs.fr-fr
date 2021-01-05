---
title: Mappage de matériaux pour les formats de modèle
description: Décrit la conversion par défaut des formats source de modèle en matériau PBR
author: jakrams
ms.author: jakras
ms.date: 02/11/2020
ms.topic: reference
ms.openlocfilehash: 8313243bf680ea1a1d63f2719b647149a04935a9
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96024042"
---
# <a name="material-mapping-for-model-formats"></a>Mappage de matériaux pour les formats de modèle

Quand une ressource source est [convertie en modèle](../how-tos/conversion/model-conversion.md), le convertisseur crée des [matériaux](../concepts/materials.md) pour chaque [maillage](../concepts/meshes.md). Vous pouvez [modifier](../how-tos/conversion/override-materials.md) la façon dont les matériaux sont créés. Toutefois, par défaut, la conversion crée des [matériaux PBR](../overview/features/pbr-materials.md). Étant donné que chaque format de fichier source, comme FBX, utilise ses propres conventions pour définir les matériaux, ces conventions doivent être mappées aux paramètres de matériau PBR d’Azure Remote Rendering. 

Cet article répertorie les mappages précis utilisés pour convertir les matériaux des ressources sources en matériaux de runtime.

## <a name="gltf"></a>glTF

Presque tous les éléments de la spécification glTF 2.0 sont pris en charge dans Azure Remote Rendering, à l’exception de *EmissiveFactor* et *EmissiveTexture*.

La table qui suit affiche le mappage :

| glTF | Azure Remote Rendering |
|:-------------------|:--------------------------|
|   baseColorFactor   |   albedoColor              |
|   baseColorTexture  |   albedoMap                |
|   metallicFactor    |   metalness                |
|   metallicTexture   |   metalnessMap             |
|   roughnessFactor   |   roughness                |
|   roughnessTexture  |   roughnessMap             |
|   occlusionFactor   |   occlusion                |
|   occlusionTexture  |   occlusionMap             |
|   normalTexture     |   normalMap                |
|   alphaCutoff       |   alphaClipThreshold       |
|   alphaMode.OPAQUE  |   alphaClipEnabled = false, isTransparent = false |
|   alphaMode.MASK    |   alphaClipEnabled = true, isTransparent = false  |
|   alphaMode.BLEND   |   isTransparent = true     |
|   doubleSided       |   isDoubleSided            |
|   emissiveFactor    |   -                        |
|   emissiveTexture   |   -                        |

Chaque texture dans la spécification glTF peut avoir une valeur `texCoord` également prise en charge dans les matériaux Azure Remote Rendering.

### <a name="embedded-textures"></a>Textures incorporées

Les textures incorporées dans les fichiers *\*.bin* ou *\*.glb* sont prises en charge.

### <a name="supported-gltf-extension"></a>Extension glTF prise en charge

En plus de l’ensemble de fonctionnalités de base, Azure Remote Rendering prend en charge les extensions glTF suivantes :

* [MSFT_packing_occlusionRoughnessMetallic](https://github.com/KhronosGroup/glTF/blob/master/extensions/2.0/Vendor/MSFT_packing_occlusionRoughnessMetallic/README.md)
* [KHR_materials_unlit](https://github.com/KhronosGroup/glTF/blob/master/extensions/2.0/Khronos/KHR_materials_unlit/README.md) : Correspond aux [matériaux couleur](../overview/features/color-materials.md). Pour les matériaux *émissifs*, il est recommandé d’utiliser cette extension.
* [KHR_materials_pbrSpecularGlossiness](https://github.com/KhronosGroup/glTF/blob/master/extensions/2.0/Khronos/KHR_materials_pbrSpecularGlossiness/README.md) : Au lieu de textures d’ébauche métallique, vous pouvez fournir des textures diffuse-specular-glossiness. L’implémentation d’Azure Remote Rendering suit directement les formules de conversion de l’extension.

## <a name="fbx"></a>FBX

Le format FBX est propriétaire et les matériaux FBX ne sont pas compatibles avec les matériaux PBR en général. Il utilise une description complexe des surfaces avec bon nombre de paramètres et propriétés uniques **qui ne sont pas tous utilisés par le pipeline Azure Remote Rendering**.

> [!IMPORTANT]
> Le pipeline de conversion de modèle d’Azure Remote Rendering prend uniquement en charge le **format FBX 2011 et versions ultérieures**.

Le format FBX définissant une approche conservatrice des matériaux, il n’existe que deux types dans la spécification FBX officielle :

* *Lambert* : rarement utilisé depuis un certain temps déjà, mais toujours pris en charge par transformation en Phong au moment de la conversion.
* *Phong* : presque tous les matériaux et la plupart des outils de contenu utilisent ce type.

Le modèle Phong est plus précis et c’est *le seul* utilisé pour les matériaux FBX. Nous l’appelons ci-dessous *Matériau FBX*.

> Maya utilise deux extensions personnalisées pour FBX en définissant des propriétés personnalisées pour PBR et les types Stingray d’un matériau. Ces détails n’étant pas inclus dans la spécification FBX, Azure Remote Rendering ne prend actuellement pas en charge ce format.

Les matériaux FBX utilisant le concept Diffuse-Specular-SpecularLevel, pour passer d’une texture diffuse à une carte Albedo, nous devons calculer les autres paramètres pour les soustraire de structure diffuse.

> Toutes les couleurs et textures dans FBX sont dans un espace sRVB (également appelé « espace Gamma ») mais Azure Remote Rendering fonctionne avec un espace linéaire pendant la visualisation et, à la fin du frame, reconvertit tout en espace sRVB. Le pipeline de ressources d’Azure Remote Rendering convertit le tout en espace linéaire pour l’envoyer en tant que données préparées au renderer.

Ce tableau montre comment les textures sont mappées de Matériaux FBX à des matériaux Azure Remote Rendering. Certaines ne sont pas utilisées directement, mais en association avec d’autres textures intervenant dans les formules (par exemple, la texture diffuse) :

| FBX | Azure Remote Rendering |
|:-----|:----|
| AmbientColor | Occlusion Map   |
| DiffuseColor | *utilisée pour Albedo, Metalness* |
| TransparentColor | *utilisée pour le canal alpha d’Albedo* |
| TransparencyFactor | *utilisée pour le canal alpha d’Albedo* |
| Opacity | *utilisée pour le canal alpha d’Albedo* |
| SpecularColor | *utilisée pour Albedo, Metalness, Roughness* |
| SpecularFactor| *utilisée pour Albedo, Metalness, Roughness* |
| ShininessExponent | *utilisée pour Albedo, Metalness, Roughness* |
| NormalMap | NormalMap |
| Bump | *convertie en NormalMap* |
| EmissiveColor | - |
| EmissiveFactor | - |
| ReflectionColor | - |
| DisplacementColor | - |

Le mappage ci-dessus est la partie la plus complexe de la conversion de matériau en raison des nombreuses hypothèses à effectuer. Nous évoquons ces hypothèses ci-dessous.

Voici certaines définitions utilisées ci-dessous :

* `Specular` =  `SpecularColor` * `SpecularFactor`
* `SpecularIntensity` = `Specular`.Red ∗ 0.2125 +  `Specular`.Green ∗ 0.7154 + `Specular`.Blue ∗ 0.0721
* `DiffuseBrightness` = 0.299 * `Diffuse`.Red<sup>2</sup> + 0.587 * `Diffuse`.Green<sup>2</sup> + 0.114 * `Diffuse`.Blue<sup>2</sup>
* `SpecularBrightness` = 0.299 * `Specular`.Red<sup>2</sup> + 0.587 * `Specular`.Green<sup>2</sup> + 0.114 * `Specular`.Blue<sup>2</sup>
* `SpecularStrength` = max(`Specular`.Red, `Specular`.Green, `Specular`.Blue)

La formule SpecularIntensity est obtenue à partir d’[ici](https://en.wikipedia.org/wiki/Luma_(video)).
La formule de luminosité est décrite dans cette [spécification](http://www.itu.int/dms_pubrec/itu-r/rec/bt/R-REC-BT.601-7-201103-I!!PDF-E.pdf).

### <a name="roughness"></a>Roughness

La valeur `Roughness` est calculée à partir des `Specular` et `ShininessExponent` à l’aide de [cette formule](https://www.cs.cornell.edu/~srm/publications/EGSR07-btdf.pdf). La formule est une approximation de roughness effectuée à partir de l’exposant spéculaire Phong :

```cpp
Roughness = sqrt(2 / (ShininessExponent * SpecularIntensity + 2))
```

### <a name="metalness"></a>Metalness

La valeur `Metalness` est calculée à partir des valeurs de `Diffuse` et de `Specular` à l’aide de cette [formule de la spécification glTF](https://github.com/bghgary/glTF/blob/gh-pages/convert-between-workflows-bjs/js/babylon.pbrUtilities.js).

L’idée ici est que nous résolvons l’équation : Ax<sup>2</sup> + Bx + C = 0.
Fondamentalement, les surfaces diélectriques réfléchissent environ 4 % de la lumière de manière spéculaire, le reste l’étant de manière diffuse. Les surfaces métalliques ne reflètent aucune lumière de manière diffuse, mais toutes de manière spéculaire.
Cette formule présente quelques inconvénients, car il n’existe aucun moyen de faire la distinction entre des surfaces brillantes métalliques et plastiques. La plupart du temps, nous supposons que la surface a des propriétés métalliques. Par conséquent, il peut arriver que des surfaces en plastique/caoutchouc brillants ne présentent pas l’aspect prévu.

```cpp
dielectricSpecularReflectance = 0.04
oneMinusSpecularStrength = 1 - SpecularStrength

A = dielectricSpecularReflectance
B = (DiffuseBrightness * (oneMinusSpecularStrength / (1 - A)) + SpecularBrightness) - 2 * A
C = A - SpecularBrightness
squareRoot = sqrt(max(0.0, B * B - 4 * A * C))
value = (-B + squareRoot) / (2 * A)
Metalness = clamp(value, 0.0, 1.0);
```

### <a name="albedo"></a>Albedo

`Albedo` est calculé à partir des valeurs `Diffuse`, `Specular` et `Metalness`.

Comme décrit dans la section Metalness, les surfaces diélectriques reflètent environ 4 % de la lumière.  
L’idée ici est d’effectuer une interpolation linéaire entre les couleurs `Dielectric` et `Metal` en utilisant la valeur `Metalness` en tant que facteur. Si la valeur de metalness est `0.0`, selon la valeur de specular, soit il s’agit d’une couleur foncée (si la valeur de specular est élevée), soit la valeur de diffuse ne change pas (si aucune valeur de specular n’est présente). Si la valeur de metalness est élevée, la couleur diffuse disparaît en faveur d’une couleur spéculaire.

```cpp
dielectricSpecularReflectance = 0.04
oneMinusSpecularStrength = 1 - SpecularStrength

dielectricColor = diffuseColor * (oneMinusSpecularStrength / (1.0f - dielectricSpecularReflectance) / max(1e-4, 1.0 - metalness))
metalColor = (Specular - dielectricSpecularReflectance * (1.0 - metalness)) * (1.0 / max(1e-4, metalness))
albedoRawColor = lerpColors(dielectricColor, metalColor, metalness * metalness)
AlbedoRGB = clamp(albedoRawColor, 0.0, 1.0);
```

La valeur `AlbedoRGB` a été calculée par la formule ci-dessus, mais le canal alpha requiert des calculs supplémentaires. Le format FBX est vague concernant la transparence, et utilise plusieurs moyens pour la définir. Les différents outils de contenu utilisent des méthodes différentes. L’idée ici est de les unifier en une seule formule. Il a cependant pour effet que certaines ressources s’affichent incorrectement comme transparentes si elles ne sont pas créées de manière commune.

Ceci est calculé à partir des valeurs `TransparentColor`, `TransparencyFactor` et `Opacity` :

si la valeur `Opacity` est définie, alors l’utiliser directement : `AlbedoAlpha` = `Opacity`, sinon  
si la valeur `TransparencyColor` est définie, alors `AlbedoAlpha` = 1.0 - ((`TransparentColor`.Red + `TransparentColor`.Green + `TransparentColor`.Blue) / 3.0), sinon  
si `TransparencyFactor`, alors `AlbedoAlpha` = 1.0 - `TransparencyFactor`

La couleur `Albedo` finale comporte quatre canaux, combinant `AlbedoRGB` avec `AlbedoAlpha`.

### <a name="summary"></a>Résumé

En résumé, la valeur `Albedo` est très proche de la valeur `Diffuse` d’origine si la valeur `Specular` est proche de zéro. Dans le cas contraire, la surface présente l’aspect d’une surface métallique et perd la couleur diffuse. La surface semple plus polie et réfléchissante si la valeur `ShininessExponent` est suffisamment élevée et la valeur `Specular` brillante. Dans le cas contraire, la surface présente un aspect rugueux et reflète grossièrement l’environnement.

### <a name="known-issues"></a>Problèmes connus

* La formule actuelle ne fonctionne pas correctement pour une géométrie colorée simple. Si la valeur `Specular` est suffisamment brillante, toutes les géométries deviennent des surfaces métalliques réfléchissantes sans couleur. La solution de contournement consiste à abaisser la valeur `Specular` à 30 % de la valeur d’origine ou à utiliser le paramètre de conversion [fbxAssumeMetallic](../how-tos/conversion/configure-model-conversion.md#converting-from-older-fbx-formats-with-a-phong-material-model).
* Des matériaux PBR ont été récemment ajoutés aux outils de création de contenu `Maya` et `3DS Max`. Ils utilisent des propriétés de boîte noire personnalisées définies par l’utilisateur pour les passer à FBX. Azure Remote Rendering ne lit pas ces propriétés supplémentaires, car elles ne sont pas documentées et le format est propriétaire.

## <a name="next-steps"></a>Étapes suivantes

* [Conversion de modèle](../how-tos/conversion/model-conversion.md)
* [Remplacement de matériaux pendant la conversion d’un modèle](../how-tos/conversion/override-materials.md)
