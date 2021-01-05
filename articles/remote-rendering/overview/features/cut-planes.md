---
title: Plans de coupe
description: Explique ce que sont les plans de coupe et comment les utiliser
author: jakrams
ms.author: jakras
ms.date: 02/06/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: 7d8905fbdcfc03f2683698cca57ab6c066e77863
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92205929"
---
# <a name="cut-planes"></a>Plans de coupe

Un *plan de coupe* est une fonctionnalité visuelle qui découpe les pixels d’un côté d’un plan virtuel, et révèle l’intérieur des [maillages](../../concepts/meshes.md).
L’image ci-dessous illustre l’effet. La partie gauche affiche le maillage d’origine, celle de droite permet de regarder à l’intérieur du maillage :

![Plan de coupe](./media/cutplane-1.png)

## <a name="cutplanecomponent"></a>CutPlaneComponent

Vous ajoutez un plan de coupe à la scène en créant un élément *CutPlaneComponent*. L’emplacement et l’orientation du plan sont déterminés par l’[entité](../../concepts/entities.md) propriétaire du composant.

```cs
void CreateCutPlane(AzureSession session, Entity ownerEntity)
{
    CutPlaneComponent cutPlane = (CutPlaneComponent)session.Actions.CreateComponent(ObjectType.CutPlaneComponent, ownerEntity);
    cutPlane.Normal = Axis.X; // normal points along the positive x-axis of the owner object's orientation
    cutPlane.FadeColor = new Color4Ub(255, 0, 0, 128); // fade to 50% red
    cutPlane.FadeLength = 0.05f; // gradient width: 5cm
}
```

```cpp
void CreateCutPlane(ApiHandle<AzureSession> session, ApiHandle<Entity> ownerEntity)
{
    ApiHandle<CutPlaneComponent> cutPlane = session->Actions()->CreateComponent(ObjectType::CutPlaneComponent, ownerEntity)->as<CutPlaneComponent>();;
    cutPlane->SetNormal(Axis::X); // normal points along the positive x-axis of the owner object's orientation
    Color4Ub fadeColor;
    fadeColor.channels = { 255, 0, 0, 128 }; // fade to 50% red
    cutPlane->SetFadeColor(fadeColor);
    cutPlane->SetFadeLength(0.05f); // gradient width: 5cm
}
```

### <a name="cutplanecomponent-properties"></a>Propriétés CutPlaneComponent

Les propriétés suivantes sont exposées sur un composant de plan de coupe :

* `Enabled`: vous pouvez désactiver temporairement les plans de coupe en désactivant le composant. Les plans de coupe désactivés n’entraînent pas de surcharge de rendu, et ne sont pas comptabilisés dans la limite globale des plans de coupe.

* `Normal`: spécifie la direction (+X,-X,+Y,-Y,+Z,-Z) qui est utilisée comme plan perpendiculaire. Cette direction est relative à l’orientation de l’entité propriétaire. Déplacez et faites pivoter l’entité propriétaire pour obtenir un positionnement précis.

* `FadeColor` et `FadeLength` :

  si la valeur alpha de *FadeColor* est différente de zéro, les pixels proches du plan de coupe s’estompent vers la partie RVB de FadeColor. La puissance du canal alpha détermine s’il s’estompe entièrement dans la couleur du fondu, ou seulement partiellement. *FadeLength* définit la distance au-delà de laquelle ce fondu doit avoir lieu.

* `ObjectFilterMask`: masque de bits de filtre qui détermine quelle géométrie est affectée par le plan de coupe. Pour plus d’informations, consultez le paragraphe suivant.

### <a name="selective-cut-planes"></a>Plans de coupe sélectifs

Il est possible de configurer des plans de coupe individuels afin qu’ils s’appliquent uniquement à une géométrie spécifique. L’illustration suivante montre à quoi peut ressembler cette configuration dans la pratique :

![Plans de coupe sélectifs](./media/selective-cut-planes.png)

Le filtrage fonctionne via **comparaison logique des masques de bits** entre un masque de bits sur le côté du plan de coupe et un deuxième masque de bits défini sur la géométrie. Si le résultat d’une opération logique `AND` entre les masques n’est pas égal à zéro, le plan de coupe affecte la géométrie.

* Le masque de bits sur le composant du plan de coupe est défini par le biais de sa propriété `ObjectFilterMask`.
* Le masque de bits sur une sous-hiérarchie de géométrie est défini via [HierarchicalStateOverrideComponent](override-hierarchical-state.md#features).

Exemples :

| Masque de filtre de plan de coupe | Masque de filtre de géométrie  | Résultat de l’opérateur logique `AND` | Le plan de coupe affecte-t-il la géométrie ?  |
|--------------------|-------------------|-------------------|:----------------------------:|
| (0000 0001) == 1   | (0000 0001) == 1  | (0000 0001) == 1  | Oui |
| (1111 0000) == 240 | (0001 0001) == 17 | (0001 0000) == 16 | Oui |
| (0000 0001) == 1   | (0000 0010) == 2  | (0000 0000) == 0  | Non |
| (0000 0011) == 3   | (0000 1000) == 8  | (0000 0000) == 0  | Non |

>[!TIP]
> Le fait de définir la propriété `ObjectFilterMask` d’un plan de coupe sur 0 signifie qu’il n’affectera aucune géométrie, car le résultat de l’opérateur logique `AND` ne peut jamais être différent de nul. Le système de rendu ne considère pas ces plans en premier lieu, il s’agit donc d’une méthode légère pour désactiver des plans de coupe individuels. Ces plans de coupe ne sont pas non plus comptabilisés dans la limite de huit plans actifs.

## <a name="limitations"></a>Limites

* Azure Remote Rendering prend en charge un **nombre maximal de huit plans de coupe actifs**. Vous pouvez créer des composants de plan de coupe supplémentaires, mais si vous essayez d’en activer davantage simultanément, l’activation est ignorée. Désactivez d’abord les autres plans si vous souhaitez changer les composants qui doivent affecter la scène.
* Les plans de coupe sont uniquement une fonctionnalité visuelle, ils n’affectent pas le résultat des [requêtes spatiales](spatial-queries.md). Si vous ne souhaitez pas utiliser le ray casting sur un maillage ouvert en coupe, vous pouvez ajuster le point de départ du rayon pour qu’il se trouve sur le plan de coupe. De cette façon, le rayon ne peut atteindre que les parties visibles.

## <a name="performance-considerations"></a>Considérations relatives aux performances

Chaque plan de coupe actif entraîne une faible dépense pendant le rendu. Désactivez ou supprimez les plans de coupe lorsqu’ils ne sont pas nécessaires.

## <a name="api-documentation"></a>Documentation de l’API

* [CutPlaneComponent, classe C#](/dotnet/api/microsoft.azure.remoterendering.cutplanecomponent)
* [CutPlaneComponent, classe C++](/cpp/api/remote-rendering/cutplanecomponent)

## <a name="next-steps"></a>Étapes suivantes

* [Rendu unilatéral](single-sided-rendering.md)
* [Requêtes spatiales](spatial-queries.md)