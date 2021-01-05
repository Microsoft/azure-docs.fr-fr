---
title: Atténuation du Z-fighting
description: Décrit les techniques permettant d’atténuer les artefacts du z-fighting
author: florianborn71
ms.author: flborn
ms.date: 02/06/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: e7550d0f997182b3938285f1d0a360a31bf05177
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92207459"
---
# <a name="z-fighting-mitigation"></a>Atténuation du Z-fighting

Lorsque deux surfaces se chevauchent, il n’est pas évident de savoir laquelle doit être affichée au-dessus de l’autre. Le résultat varie même selon le pixel, ce qui génère des artefacts dépendants de l’affichage de l’appareil photo. Par conséquent, lorsque la caméra ou le maillage se déplace, ces éléments scintillent franchement. Cet artefact est appelé le *z-fighting*. Pour les applications de réalité augmentée (AR) et de réalité virtuelle (VR), le problème est amplifié, car les casques audiovisuels se déplacent bien évidemment en permanence. Afin d’éviter une gêne pour l’utilisateur, la fonctionnalité d’atténuation du z-fighting est disponible dans Azure Remote Rendering.

## <a name="z-fighting-mitigation-modes"></a>Modes d’atténuation du z-fighting

|Situation                        | Résultats                               |
|---------------------------------|:-------------------------------------|
|Z-fighting normal               |![Aucune priorité déterministe entre les quads rouge et vert](./media/zfighting-0.png)|
|Atténuation du z-fighting activée    |![Le quad rouge a la priorité](./media/zfighting-1.png)|
|Mise en surbrillance du damier activée|![Préférence de basculement entre les quads rouge et vert dans le modèle de damier](./media/zfighting-2.png)|

Le code suivant permet l’atténuation du z-fighting :

```cs
void EnableZFightingMitigation(AzureSession session, bool highlight)
{
    ZFightingMitigationSettings settings = session.Actions.ZFightingMitigationSettings;

    // enabling z-fighting mitigation
    settings.Enabled = true;

    // enabling checkerboard highlighting of z-fighting potential
    settings.Highlighting = highlight;
}
```

```cpp
void EnableZFightingMitigation(ApiHandle<AzureSession> session, bool highlight)
{
    ApiHandle<ZFightingMitigationSettings> settings = session->Actions()->GetZFightingMitigationSettings();

    // enabling z-fighting mitigation
    settings->SetEnabled(true);

    // enabling checkerboard highlighting of z-fighting potential
    settings->SetHighlighting(highlight);
}
```

> [!NOTE]
> L’atténuation du z-fighting est un paramètre global qui influe sur tous les maillages affichés.

## <a name="reasons-for-z-fighting"></a>Causes du z-fighting

Le z-fighting se produit principalement pour deux raisons :

1. lorsque les surfaces sont très éloignées de la caméra, la précision de leurs valeurs de profondeur se dégrade et les valeurs deviennent impossibles à distinguer ;
1. quand les surfaces d’un maillage se chevauchent physiquement.

Le premier problème peut toujours se produire, et il est difficile à éliminer. Si cela se produit dans votre application, assurez-vous que le rapport entre la distance du *plan proche* et celle du *plan lointain* est aussi faible que possible. Par exemple, un plan proche à une distance de 0,01 et un plan lointain à une distance de 1000 crée ce problème bien plus tôt qu’un plan proche à 0,1 et un plan lointain à une distance de 20.

Le deuxième problème est un indicateur de contenu mal écrit. Dans le monde réel, deux objets ne peuvent pas se trouver au même endroit en même temps. En fonction de l’application, les utilisateurs peuvent souhaiter savoir si des surfaces se chevauchant existent, et à quel endroit. Par exemple, une simulation par CAO d’un bâtiment à la base d’une construction réelle ne doit contenir aucune intersection de surfaces matériellement irréalisable. Pour permettre l’inspection visuelle, le mode de mise en surbrillance est disponible ; il affiche la zone de z-fighting potentielle sous la forme d’un motif en damier animé.

## <a name="limitations"></a>Limites

L’atténuation du z-fighting est la meilleure solution dont nous disposons. Il n’est pas garanti qu’elle supprime entièrement le z-fighting. Par ailleurs, elle préfèrera automatiquement une surface à une autre. Ainsi, lorsque des surfaces sont trop proches l’une de l’autre, il peut arriver que la « mauvaise » surface se retrouve au-dessus. Un problème courant se pose lorsque du texte et d’autres décalques sont appliqués à une surface. Avec l’atténuation du z-fighting activée, ces détails peuvent tout simplement disparaître.

## <a name="performance-considerations"></a>Considérations relatives aux performances

* L’activation de l’atténuation du z-fighting entraîne peu voire pas de surcharge de performances.
* De plus, l’activation de la superposition du z-fighting entraîne bien une surcharge de performances non négligeable, même si elle peut varier en fonction de la scène.

## <a name="api-documentation"></a>Documentation de l’API

* [RemoteManager.ZFightingMitigationSettings, propriété C#](/dotnet/api/microsoft.azure.remoterendering.remotemanager.zfightingmitigationsettings)
* [RemoteManager::ZFightingMitigationSettings(), C++](/cpp/api/remote-rendering/remotemanager#zfightingmitigationsettings)

## <a name="next-steps"></a>Étapes suivantes

* [Modes de rendu](../../concepts/rendering-modes.md)
* [Reprojection en phase tardive](late-stage-reprojection.md)