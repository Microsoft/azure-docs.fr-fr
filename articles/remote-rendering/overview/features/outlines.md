---
title: Rendu en mode contour
description: Explique comment effectuer le rendu en mode contour de la sélection
author: florianborn71
ms.author: flborn
ms.date: 02/11/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: 55151ab965b13a944fee5517a2ca8b4f9eb64bb0
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92207238"
---
# <a name="outline-rendering"></a>Rendu en mode contour

Les objets sélectionnés peuvent être mis en surbrillance en ajoutant le rendu en mode contour via le [composant de remplacement d’état hiérarchique](../../overview/features/override-hierarchical-state.md). Ce chapitre explique comment les paramètres globaux du rendu en mode contour sont modifiés via l’API cliente.

Les propriétés de contour constituent un paramètre global. Tous les objets utilisant le rendu en mode contour ont recours au même paramètre ; il n’est pas possible d’utiliser une couleur de contour par objet.

## <a name="parameters-for-outlinesettings"></a>Paramètres pour `OutlineSettings`

La classe `OutlineSettings` contient les paramètres relatifs aux propriétés de contour globales. Elle expose les membres suivants :

| Paramètre      | Type    | Description                                             |
|----------------|---------|---------------------------------------------------------|
| `Color`          | Color4Ub | Couleur utilisée pour dessiner le contour. La partie alpha est ignorée.         |
| `PulseRateHz`    | float   | Vitesse d'oscillation du contour par seconde|
| `PulseIntensity` | float   | Intensité de l’effet de pulsation du contour. Doit être compris entre 0,0 pour aucune pulsation et 1,0 pour une pulsation totale. L’intensité définit implicitement l’opacité minimale du contour comme `MinOpacity = 1.0 - PulseIntensity`. |

![An object rendered three times with different outline parameters](./media/outlines.png) Effet de modification du paramètre `color` de jaune (gauche) à magenta (centre) et `pulseIntensity` de 0 à 0,8 (droite).

## <a name="example"></a>Exemple

Le code suivant montre un exemple de définition de paramètres de contour via l’API :

```cs
void SetOutlineParameters(AzureSession session)
{
    OutlineSettings outlineSettings = session.Actions.OutlineSettings;
    outlineSettings.Color = new Color4Ub(255, 255, 0, 255);
    outlineSettings.PulseRateHz = 2.0f;
    outlineSettings.PulseIntensity = 0.5f;
}
```

```cpp
void SetOutlineParameters(ApiHandle<AzureSession> session)
{
    ApiHandle<OutlineSettings> outlineSettings = session->Actions()->GetOutlineSettings();
    Color4Ub outlineColor;
    outlineColor.channels = { 255, 255, 0, 255 };
    outlineSettings->SetColor(outlineColor);
    outlineSettings->SetPulseRateHz(2.0f);
    outlineSettings->SetPulseIntensity(0.5f);
}
```

## <a name="performance"></a>Performances

Le rendu en mode contour peut avoir un impact significatif sur les performances de rendu. Cet impact varie en fonction de la relation spatiale écran-espace entre les objets sélectionnés et non sélectionnés pour un cadre donné.

## <a name="api-documentation"></a>Documentation de l’API

* [RemoteManager.OutlineSettings, propriété C#](/dotnet/api/microsoft.azure.remoterendering.remotemanager.outlinesettings)
* [RemoteManager::OutlineSettings(), C++](/cpp/api/remote-rendering/remotemanager#outlinesettings)

## <a name="next-steps"></a>Étapes suivantes

* [Composant de remplacement d’état hiérarchique](../../overview/features/override-hierarchical-state.md)