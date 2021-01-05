---
title: Éclairage de la scène
description: Description et propriétés des sources de lumière
author: florianborn71
ms.author: flborn
ms.date: 02/10/2020
ms.topic: article
ms.openlocfilehash: 49027899d66a2192cc311fb4dba66e441155b527
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92206847"
---
# <a name="scene-lighting"></a>Éclairage de la scène

Par défaut, une [lumière ambiante](sky.md) est utilisée pour l'éclairage des objets rendus à distance. Dans la plupart des cas, cet éclairage est suffisant, mais vous pouvez ajouter d'autres sources de lumière à la scène.

> [!IMPORTANT]
> Seuls les [matériaux à rendu physique réaliste](pbr-materials.md) sont sensibles aux sources de lumière. La luminosité des [matériaux en couleur](color-materials.md) est toujours optimale.

> [!NOTE]
> La projection d'ombres n'est actuellement pas prise en charge. Azure Remote Rendering est optimisé pour le rendu d'un nombre considérable de géométries, quitte à avoir recours à plusieurs GPU si nécessaire. Les approches classiques en matière de projection d'ombres ne conviennent pas à de tels scénarios.

## <a name="common-light-component-properties"></a>Propriétés communes des composants liés à la lumière

Tous les types de lumière sont issus de la classe de base abstraite `LightComponent` et partagent les propriétés suivantes :

* **Color :** couleur de la lumière dans l'[espace Gamma](https://en.wikipedia.org/wiki/SRGB). Alpha est ignoré.

* **Intensity :** intensité de la lumière. Pour la lumière ponctuelle et la lumière projetée, l'intensité définit également la distance à laquelle la lumière éclaire la scène.

## <a name="point-light"></a>Lumière ponctuelle

Dans Azure Remote Rendering, le composant `PointLightComponent` peut non seulement émettre de la lumière à partir d'un seul point, mais également à partir d'une petite sphère ou d'un petit tube, pour simuler des sources de lumière plus douces.

### <a name="pointlightcomponent-properties"></a>Propriétés du composant PointLightComponent

* **Radius :** le rayon par défaut est zéro, auquel cas la lumière agit comme une lumière ponctuelle. Si le rayon est supérieur à zéro, il agit comme une source de lumière sphérique, ce qui modifie l'apparence des surbrillances spéculaires.

* **Length :** si les propriétés `Length` et `Radius` ne sont pas nulles, la lumière agit comme un tube lumineux. Cela peut être utilisé pour simuler des tubes néon.

* **AttenuationCutoff :** si elle reste définie sur (0,0), l'atténuation de la lumière ne dépend que de son intensité (`Intensity`). Cependant, vous pouvez fournir des distances min/max personnalisées sur lesquelles l'intensité de la lumière est réduite linéairement à 0. Cette fonctionnalité peut être utilisée pour réduire la plage d'influence d'une lumière spécifique.

* **ProjectedCubemap :** si elle est définie sur un [cubemap](../../concepts/textures.md) valide, la texture est projetée sur la géométrie environnante de la lumière. La couleur du cubemap est modulée par la couleur de la lumière.

## <a name="spot-light"></a>Lumière projetée

Le composant `SpotLightComponent` est semblable au composant `PointLightComponent` mais la lumière est limitée à la forme d'un cône. L'orientation du cône est définie par l'*axe z négatif de l'entité propriétaire*.

### <a name="spotlightcomponent-properties"></a>Propriétés du composant SpotLightComponent

* **Radius :** identique au composant `PointLightComponent`.

* **SpotAngleDeg :** cet intervalle définit l'angle intérieur et extérieur du cône, mesuré en degrés. Tout ce qui se trouve dans l'angle intérieur est pleinement éclairé. Une atténuation est appliquée en direction de l'angle extérieur, ce qui génère un effet de pénombre.

* **FalloffExponent :** définit le degré de transition de l'atténuation entre l'angle intérieur et l'angle extérieur du cône. Plus la valeur est élevée, plus la transition est importante. La valeur par défaut 1.0 se traduit par une transition linéaire.

* **AttenuationCutoff :** identique au composant `PointLightComponent`.

* **Projected2dTexture :** si elle est définie sur une [texture 2D](../../concepts/textures.md) valide, l'image est projetée sur la géométrie que la lumière éclaire. La couleur de la texture est modulée par la couleur de la lumière.

## <a name="directional-light"></a>Lumière directionnelle

Le composant `DirectionalLightComponent` simule une source de lumière infiniment éloignée. La lumière éclaire dans la direction de l'*axe z négatif de l'entité propriétaire*. La position de l'entité est ignorée.

Il n'y a pas d'autres propriétés.

## <a name="performance-considerations"></a>Considérations relatives aux performances

Les sources de lumière ont un impact significatif sur les performances de rendu. Utilisez-les avec précaution et uniquement si l'application l'exige. Toute condition d'éclairage global statique, y compris un composant directionnel statique, peut être obtenue avec une [texture ambiante personnalisée](sky.md), sans coût de rendu supplémentaire.

## <a name="api-documentation"></a>Documentation de l’API

* [LightComponentBase, classe C#](/dotnet/api/microsoft.azure.remoterendering.lightcomponentbase)
* [PointLightComponent, classe C#](/dotnet/api/microsoft.azure.remoterendering.pointlightcomponent)
* [SpotLightComponent, classe C#](/dotnet/api/microsoft.azure.remoterendering.spotlightcomponent)
* [DirectionalLightComponent, classe C#](/dotnet/api/microsoft.azure.remoterendering.directionallightcomponent)
* [LightComponentBase, classe C++](/cpp/api/remote-rendering/lightcomponentbase)
* [PointLightComponent, classe C++](/cpp/api/remote-rendering/pointlightcomponent)
* [SpotLightComponent, classe C++](/cpp/api/remote-rendering/spotlightcomponent)
* [DirectionalLightComponent, classe C++](/cpp/api/remote-rendering/directionallightcomponent)

## <a name="next-steps"></a>Étapes suivantes

* [Matériaux](../../concepts/materials.md)
* [Lumière ambiante](sky.md)