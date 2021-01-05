---
title: Matériaux
description: Description des matériaux pour le rendu et propriétés des matériaux
author: jakrams
ms.author: jakras
ms.date: 02/11/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: 8dca22303edc3b75625df88dac98557c6a2e6162
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92202850"
---
# <a name="materials"></a>Matériaux

Les matériaux sont des [ressources partagées](../concepts/lifetime.md) qui définissent la façon dont les [maillages](meshes.md) sont rendus. Les matériaux sont utilisés pour spécifier les [textures](textures.md) à appliquer, si les objets doivent être transparents ou non et comment l’éclairage sera calculé.

Les matériaux sont créés automatiquement pendant la [conversion de modèle](../how-tos/conversion/model-conversion.md) et sont accessibles au moment de l’exécution. Vous pouvez également créer des matériaux personnalisés à partir du code et remplacer des matériaux existants. Ce scénario s’applique en particulier quand vous souhaitez partager un matériau entre plusieurs maillages. Les modifications apportées à un matériau sont visibles sur chaque maillage qui y fait référence. Cette méthode peut donc être utilisée pour appliquer facilement des modifications.

> [!NOTE]
> La modification de matériaux peut être appropriée pour certains cas d’usage tels que la mise en évidence d’un objet spécifique. Cependant, le composant [HierarchicalStateOverrideComponent](../overview/features/override-hierarchical-state.md) offre un moyen beaucoup plus simple d’obtenir le résultat souhaité.

## <a name="material-types"></a>Types de matériau

Azure Remote Rendering offre deux types de matériau :

* Les [matériaux PBR](../overview/features/pbr-materials.md) sont utilisés pour les surfaces qui doivent être rendues sous une forme aussi correcte que possible d’un point de vue physique. Pour ces matériaux, l’éclairage réaliste est calculé à l’aide du *rendu physique réaliste* (PBR, Physically-Based Rendering). Pour tirer le meilleur parti de ce type de matériau, il est important de fournir des données d’entrée de haute qualité comme les cartes de rugosité et de normale.

* Les [matériaux de couleur](../overview/features/color-materials.md) sont utilisés pour les cas où aucun éclairage supplémentaire n’est souhaité. Ces matériaux sont toujours de couleur vive et sont plus faciles à configurer. Les matériaux de couleur sont utilisés pour les données qui ne doivent avoir aucun éclairage ou qui intègrent déjà un éclairage statique. Il peut s’agir, par exemple, de modèles obtenus par [photogrammétrie](https://en.wikipedia.org/wiki/Photogrammetry).

## <a name="mesh-vs-meshcomponent-material-assignment"></a>Affectation de matériaux : maillage et MeshComponent

Les [maillages](meshes.md) comportent un ou plusieurs sous-maillages. Chaque sous-maillage fait référence à un matériau. Vous pouvez modifier le matériau à utiliser directement sur le maillage, ou vous pouvez remplacer le matériau à utiliser pour un sous-maillage sur un [MeshComponent](meshes.md#meshcomponent).

Quand vous modifiez un matériau directement sur la ressource de maillage, toutes les instances de ce maillage sont affectées. Toutefois, sa modification sur le MeshComponent affecte uniquement cette instance de maillage. La méthode la plus appropriée dépend du comportement souhaité, mais l’approche la plus courante consiste à modifier un MeshComponent.

## <a name="material-de-duplication"></a>Déduplication de matériaux

Lors de la conversion, plusieurs matériaux ayant les mêmes propriétés et textures sont automatiquement dédupliqués en un seul matériau. Vous pouvez désactiver cette fonctionnalité dans les [paramètres de conversion](../how-tos/conversion/configure-model-conversion.md), mais nous vous recommandons de la laisser activée pour des performances optimales.

## <a name="material-classes"></a>Classes de matériaux

Tous les matériaux fournis par l’API sont dérivés de la classe de base `Material`. Vous pouvez demander leur type avec `Material.MaterialSubType` ou en les castant directement :

```cs
void SetMaterialColorToGreen(Material material)
{
    if (material.MaterialSubType == MaterialType.Color)
    {
        ColorMaterial colorMaterial = material as ColorMaterial;
        colorMaterial.AlbedoColor = new Color4(0, 1, 0, 1);
        return;
    }

    PbrMaterial pbrMat = material as PbrMaterial;
    if (pbrMat != null)
    {
        pbrMat.AlbedoColor = new Color4(0, 1, 0, 1);
        return;
    }
}
```

```cpp
void SetMaterialColorToGreen(ApiHandle<Material> material)
{
    if (material->GetMaterialSubType() == MaterialType::Color)
    {
        ApiHandle<ColorMaterial> colorMaterial = material.as<ColorMaterial>();
        colorMaterial->SetAlbedoColor({ 0, 1, 0, 1 });
        return;
    }

    if (material->GetMaterialSubType() == MaterialType::Pbr)
    {
        ApiHandle<PbrMaterial> pbrMat = material.as<PbrMaterial>();
        pbrMat->SetAlbedoColor({ 0, 1, 0, 1 });
        return;
    }
}
```

## <a name="api-documentation"></a>Documentation de l’API

* [Material, classe C#](/dotnet/api/microsoft.azure.remoterendering.material)
* [ColorMaterial, classe C#](/dotnet/api/microsoft.azure.remoterendering.colormaterial)
* [PbrMaterial, classe C#](/dotnet/api/microsoft.azure.remoterendering.pbrmaterial)
* [RemoteManager.CreateMaterial(), C#](/dotnet/api/microsoft.azure.remoterendering.remotemanager.creatematerial)
* [Material, classe C++](/cpp/api/remote-rendering/material)
* [ColorMaterial, classe C++](/cpp/api/remote-rendering/colormaterial)
* [PbrMaterial, classe C++](/cpp/api/remote-rendering/pbrmaterial)
* [RemoteManager::CreateMaterial(), C++](/cpp/api/remote-rendering/remotemanager#creatematerial)

## <a name="next-steps"></a>Étapes suivantes

* [Matériaux PBR](../overview/features/pbr-materials.md)
* [Matériaux de couleur](../overview/features/color-materials.md)