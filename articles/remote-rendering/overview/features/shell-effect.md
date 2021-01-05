---
title: Rendu de châssis
description: Explique comment utiliser l’effet de rendu de châssis
author: jumeder
ms.author: jumeder
ms.date: 10/23/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: f59c4f8225d31b61df08f30863c8b9300e20e820
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94447307"
---
# <a name="shell-rendering"></a>Rendu de châssis

L’état de châssis du [composant de remplacement d’état hiérarchique](../../overview/features/override-hierarchical-state.md) est un effet de transparence. Contrairement à un rendu [semi-transparent](../../overview/features/override-hierarchical-state.md), seule la couche la plus en avant des objets est visible, comme dans un rendu opaque. De plus, l’aspect normal des objets peut être modifié en cas de rendu sous forme de châssis. L’effet est prévu pour les cas d’usage où l’utilisateur doit être visuellement guidé vers les parties importantes tout en conservant une sensibilité spatiale sur l’ensemble de la scène.

Vous pouvez configurer l’apparence des objets rendus en tant que châssis par le biais de l’état global `ShellRenderingSettings`. Tous les objets qui utilisent le rendu de châssis utilisent le même paramètre. Il n’existe aucun paramètre d’objet.

## <a name="shellrenderingsettings-parameters"></a>Paramètres ShellRenderingSettings

La classe `ShellRenderingSettings` contient les paramètres relatifs aux propriétés de rendu de châssis globales :

| Paramètre      | Type    | Description                                             |
|----------------|---------|---------------------------------------------------------|
| `Desaturation` | float   | Quantité de désaturation à appliquer à la couleur de l’objet final habituel, comprise dans une plage allant de 0 (aucune désaturation) à 1 (désaturation complète) |
| `Opacity`      | float   | Opacité des objets rendus en tant que châssis, comprise dans une plage allant de 0 (invisible) à 1 (entièrement opaque) |

Consultez aussi le tableau suivant pour obtenir des exemples des effets des paramètres quand ils sont appliqués à une scène entière :

|                | 0 | 0,25 | 0.5 | 0,75 | 1.0 | 
|----------------|:-:|:----:|:---:|:----:|:---:|
| **Désaturation** | ![Désaturation - 0,0](./media/shell-desaturation-00.png) | ![Désaturation - 0,25](./media/shell-desaturation-025.png) | ![Désaturation - 0,5](./media/shell-desaturation-05.png) | ![Désaturation - 0,75](./media/shell-desaturation-075.png) | ![Désaturation - 1.0](./media/shell-desaturation-10.png) |
| **Opacity**      | ![Opacité - 0,0](./media/shell-opacity-00.png) | ![Opacité -0,25](./media/shell-opacity-025.png) | ![Opacité - 0,5](./media/shell-opacity-05.png) | ![Opacité - 0,75](./media/shell-opacity-075.png) | ![Opacité - 1,0](./media/shell-opacity-10.png) |

L’effet de châssis est appliqué sur la couleur opaque finale dans laquelle la scène serait rendue autrement. Cet effet comprend le [remplacement de l’état hiérarchique de la teinte](../../overview/features/override-hierarchical-state.md).

## <a name="example"></a>Exemple

Le code suivant illustre un exemple d’utilisation de l’état `ShellRenderingSettings` par le biais de l’API :

```cs
void SetShellSettings(AzureSession session)
{
    ShellRenderingSettings shellRenderingSettings = session.Actions.ShellRenderingSettings;
    shellRenderingSettings.Desaturation = 0.5f;
    shellRenderingSettings.Opacity = 0.1f;
}
```

```cpp
void SetShellSettings(ApiHandle<AzureSession> session)
{
    ApiHandle<ShellRenderingSettings> shellRenderingSettings = session->Actions()->GetShellRenderingSettings();
    shellRenderingSettings->SetDesaturation(0.5f);
    shellRenderingSettings->SetOpacity(0.1f);
}
```

## <a name="performance"></a>Performances

La fonctionnalité de rendu de châssis comporte une petite surcharge constante en comparaison avec le rendu opaque standard. Elle est beaucoup plus rapide que l’utilisation de documents transparents sur des objets ou le rendu [semi-transparent](../../overview/features/override-hierarchical-state.md). Les performances peuvent se dégrader plus fortement si seules certaines parties de la scène passent en rendu de châssis. Cette dégradation peut se produire en raison d’un nombre d’objets plus élevé nécessitant un rendu. À cet égard, les performances se comportent de la même façon que la fonctionnalité [Plans de coupe](../../overview/features/cut-planes.md).

## <a name="next-steps"></a>Étapes suivantes

* [Composant de remplacement d’état hiérarchique](../../overview/features/override-hierarchical-state.md)