---
title: Effet Fresnel
description: Page expliquant la fonctionnalité de l’effet Fresnel sur les matériaux
author: jumeder
ms.author: jumeder
ms.date: 11/09/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: c8ecd3afc95cf4b8054cd7db3530c15afebea3ce
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498232"
---
# <a name="fresnel-effect"></a>Effet Fresnel

La fonctionnalité de l’effet Fresnel sur les matériaux correspond à un effet ad hoc, non correct physiquement. La fonctionnalité est basée sur l’observation physique des objets qui deviennent plus réfléchissants à ces angles. Le principe de réflectance de Fresnel est déjà incorporé physiquement dans le [modèle de matériau PBR](../../overview/features/pbr-materials.md) utilisé dans Azure Remote Rendering. En revanche, la fonctionnalité de l’effet Fresnel sur les matériaux est simplement un ajout d’effet de couleur sans dépendance par rapport aux [lumières](../../overview/features/lights.md) ou à l’[environnement du ciel](../../overview/features/sky.md).

L’effet Fresnel donne aux objets affectés une couleur brillante sur leurs bords. Vous trouverez des informations sur la personnalisation des effets et des exemples de résultats de rendu dans les sections suivantes.

## <a name="enabling-the-fresnel-effect"></a>Activation de l’effet Fresnel

Pour utiliser la fonctionnalité d’effet Fresnel, vous devez l’activer sur les matériaux en question. Pour l’activer, définissez le FresnelEffect de [PbrMaterialFeatures](/dotnet/api/microsoft.azure.remoterendering.pbrmaterialfeatures) sur le [matériau PBR](../../overview/features/pbr-materials.md). Le même modèle s’applique à [ColorMaterialFeatures](/dotnet/api/microsoft.azure.remoterendering) et au [matériau de couleur](../../overview/features/color-materials.md). Consultez la section des exemples de code pour une démonstration d’utilisation.

Une fois activé, l’effet Fresnel est immédiatement visible. Par défaut, la brillance est blanche (1, 1, 1, 1) et a un exposant égal à 1. Vous pouvez personnaliser ces paramètres à l’aide des méthodes setter de paramètres ci-dessous.

## <a name="customizing-the-effect-appearance"></a>Personnalisation de l’apparence de l’effet

Vous pouvez personnaliser l’effet Fresnel en fonction de chaque matériau à l’aide des propriétés suivantes :

| Propriété du matériau | Type | Explication |
|-------------------|------|-------------|
| FresnelEffectColor | Color4 | Couleur ajoutée au maximum au fur et à mesure de la brillance de Fresnel. Le canal alpha est ignoré. |
| FresnelEffectExponent | float | Diffusion de la brillance de Fresnel. Va de 0.01 (réparti sur tout l’objet) à 10 (uniquement les angles les plus gracieux). |

En pratique, les différents paramètres de couleur et d’exposant se présentent ainsi :

![Exemples d’effet Fresnel](./media/fresnel-effect-examples.png)

L’exposant de l’effet Fresnel est progressivement augmenté de 1 à 10 pour chaque ligne de couleur. Cela entraîne progressivement une brillance de Fresnel vers les bords des objets visualisés. L’effet Fresnel n’est pas non plus affecté par la transparence, comme vous pouvez le voir dans l’exemple suivant :

![Exemples de transparence d’effet Fresnel](./media/fresnel-effect-transparent-examples.png)

Comme indiqué, les objets de la diagonale sont entièrement transparents, mais la brillance de Fresnel est conservée. À cet égard, l’effet imite l’effet Fresnel physique, qui est également présent dans ces captures d’écran.

## <a name="code-samples"></a>Exemples de code

Les exemples de code suivants illustrent l’activation et la personnalisation de l’effet Fresnel pour un [matériau PBR](../../overview/features/pbr-materials.md) et un [matériau de couleur](../../overview/features/color-materials.md) :

```cs
    void SetFresnelEffect(AzureSession session, Material material)
    {
        if (material.MaterialSubType == MaterialType.Pbr)
        {
            var pbrMaterial = material as PbrMaterial;
            pbrMaterial.PbrFlags |= PbrMaterialFeatures.FresnelEffect;
            pbrMaterial.FresnelEffectColor = new Color4(1.0f, 0.5f, 0.1f, 1.0f);
            pbrMaterial.FresnelEffectExponent = 3.141592f;
        }
        else if (material.MaterialSubType == MaterialType.Color)
        {
            var colorMaterial = material as ColorMaterial;
            colorMaterial.ColorFlags |= ColorMaterialFeatures.FresnelEffect;
            colorMaterial.FresnelEffectColor = new Color4(0.25f, 1.0f, 0.25f, 1.0f);
            colorMaterial.FresnelEffectExponent = 7.654321f;
        }
    }
```

```cpp
void SetFresnelEffect(ApiHandle<AzureSession> session, ApiHandle<Material> material)
{
    if (material->GetMaterialSubType() == MaterialType::Pbr)
    {
        auto pbrMaterial = material.as<PbrMaterial>();
        auto featureFlags = PbrMaterialFeatures((int32_t)pbrMaterial->GetPbrFlags() | (int32_t)PbrMaterialFeatures::FresnelEffect);
        pbrMaterial->SetPbrFlags(featureFlags);
        pbrMaterial->SetFresnelEffectColor(Color4{ 1.f, 0.5f, 0.1f, 1.f });
        pbrMaterial->SetFresnelEffectExponent(3.141592f);

    }
    else if (material->GetMaterialSubType() == MaterialType::Color)
    {
        auto colorMaterial = material.as<ColorMaterial>();
        auto featureFlags = ColorMaterialFeatures((int32_t)colorMaterial->GetColorFlags() | (int32_t)ColorMaterialFeatures::FresnelEffect);
        colorMaterial->SetColorFlags(featureFlags);
        colorMaterial->SetFresnelEffectColor(Color4{ 0.25f, 1.f, 0.25f, 1.f });
        colorMaterial->SetFresnelEffectExponent(7.654321f);
    }
}
```

## <a name="api-documentation"></a>Documentation de l’API

* [C# PbrMaterialFeatures](/dotnet/api/microsoft.azure.remoterendering.pbrmaterialfeatures)
* [C++ PbrMaterialFeatures](/cpp/api/remote-rendering/pbrmaterialfeatures)
* [C# ColorMaterialFeatures](/dotnet/api/microsoft.azure.remoterendering.colormaterialfeatures)
* [C++ ColorMaterialFeatures](/cpp/api/remote-rendering/colormaterialfeatures)

## <a name="next-steps"></a>Étapes suivantes

* [Matériaux](../../concepts/materials.md)
* [Matériaux PBR](../../overview/features/pbr-materials.md)
* [Matériaux de couleur](../../overview/features/color-materials.md)