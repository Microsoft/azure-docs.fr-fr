---
title: Requêtes spatiales
description: Comment effectuer des requêtes spatiales dans une scène
author: jakrams
ms.author: jakras
ms.date: 02/07/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: d1a7baa25497cf1ba697725ac8530bc04c458aa5
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92207432"
---
# <a name="spatial-queries"></a>Requêtes spatiales

Les requêtes spatiales sont des opérations grâce auxquelles vous pouvez demander au service de rendu distant les objets qui se trouvent dans une zone. Les requêtes spatiales sont souvent utilisées pour implémenter des interactions, telles que l’identification de l’objet vers lequel pointe l’utilisateur.

Toutes les requêtes spatiales sont évaluées sur le serveur. Il s’agit par conséquent d’opérations asynchrones dont les résultats arriveront avec un délai dépendant de la latence de votre réseau. Étant donné que chaque requête spatiale génère du trafic réseau, veillez à ne pas en formuler un trop grand nombre simultanément.

## <a name="collision-meshes"></a>Maillages de collision

Les requêtes spatiales sont optimisées par le moteur [Havok Physics](https://www.havok.com/products/havok-physics) et nécessitent la présence d’un maillage de collision dédié. Par défaut, la [conversion de modèle](../../how-tos/conversion/model-conversion.md) génère des maillages de collision. Si vous n’avez pas besoin de requêtes spatiales sur un modèle complexe, envisagez de désactiver la génération du maillage de collision dans les [options de conversion](../../how-tos/conversion/configure-model-conversion.md), car l’impact de cette option est multiple :

* La [conversion de modèle](../../how-tos/conversion/model-conversion.md) prend beaucoup plus de temps.
* Les tailles de fichier des modèles convertis sont beaucoup plus volumineuses, ce qui a une incidence sur la vitesse de téléchargement.
* Les temps de chargement du runtime sont plus longs.
* La consommation de la mémoire processeur du runtime est élevée.
* Il y a une légère surcharge des performances d’exécution pour chaque instance de modèle.

## <a name="ray-casts"></a>Ray casts

Un *ray cast* est une requête spatiale dans laquelle le runtime vérifie quels objets sont croisés par un rayon, en démarrant à une position donnée et en pointant dans une certaine direction. En matière d’optimisation, une distance de rayon maximale est également donnée pour ne pas rechercher des objets trop éloignés.

```cs
async void CastRay(AzureSession session)
{
    // trace a line from the origin into the +z direction, over 10 units of distance.
    RayCast rayCast = new RayCast(new Double3(0, 0, 0), new Double3(0, 0, 1), 10);

    // only return the closest hit
    rayCast.HitCollection = HitCollectionPolicy.ClosestHit;

    RayCastHit[] hits = await session.Actions.RayCastQueryAsync(rayCast).AsTask();

    if (hits.Length > 0)
    {
        var hitObject = hits[0].HitObject;
        var hitPosition = hits[0].HitPosition;
        var hitNormal = hits[0].HitNormal;

        // do something with the hit information
    }
}
```

```cpp
void CastRay(ApiHandle<AzureSession> session)
{
    // trace a line from the origin into the +z direction, over 10 units of distance.
    RayCast rayCast;
    rayCast.StartPos = { 0, 0, 0 };
    rayCast.EndPos = { 0, 0, 1 };
    rayCast.MaxHits = 10;

    // only return the closest hit
    rayCast.HitCollection = HitCollectionPolicy::ClosestHit;

    ApiHandle<RaycastQueryAsync> castQuery = *session->Actions()->RayCastQueryAsync(rayCast);

    castQuery->Completed([](const ApiHandle<RaycastQueryAsync>& async)
        {
            std::vector<RayCastHit> hits;
            async->GetResult(hits);

            if (hits.size() > 0)
            {
                auto hitObject = hits[0].HitObject;
                auto hitPosition = hits[0].HitPosition;
                auto hitNormal = hits[0].HitNormal;

                // do something with the hit information
            }
        });
}
```


Il existe trois modes de collecte de hits :

* **`Closest` :** dans ce mode, seul le hit le plus proche sera signalé.
* **`Any` :** préférez ce mode lorsque vous souhaitez savoir *si* un rayon peut atteindre quelque chose, sans être intéressé de savoir exactement ce qui a été touché. Cette requête peut être beaucoup moins coûteuse à évaluer, mais elle n’a que peu d’applications.
* **`All` :** dans ce mode, tout ce que touche le rayon est signalé, trié par distance. N’utilisez pas ce mode, à moins que le premier hit à lui seul ne soit pas suffisant. Limitez le nombre de hits signalés avec l’option `MaxHits`.

L’exclusion sélective d’objets, pour qu’ils ne soient pas pris en compte par les ray casts, peut se faire en utilisant le composant [HierarchicalStateOverrideComponent](override-hierarchical-state.md).

<!--
The CollisionMask allows the query to consider or ignore some objects based on their collision layer. If an object has layer L, it will be hit only if the mask has bit L set.
It is useful in case you want to ignore objects, for instance when setting an object transparent, and trying to select another object behind it.
TODO : Add an API to make that possible.
-->

### <a name="hit-result"></a>Résultat des hits

Le résultat d’une requête de ray cast est un tableau de hits. Le tableau est vide si aucun objet n’a été touché.

Un hit présente les propriétés suivantes :

* **`HitEntity` :** l’[entité](../../concepts/entities.md) touchée.
* **`SubPartId` :** le *sous-maillage*touché dans un [MeshComponent](../../concepts/meshes.md). Peut être utilisé pour une indexation dans `MeshComponent.UsedMaterials` et une recherche des [matériaux](../../concepts/materials.md) à ce stade.
* **`HitPosition` :** position dans l’espace universel où le rayon a croisé l’objet.
* **`HitNormal` :** perpendiculaire à la surface de l’espace universel du maillage, au point de l’intersection.
* **`DistanceToHit` :** distance, de la position de départ du rayon au hit.

## <a name="api-documentation"></a>Documentation de l’API

* [RemoteManager.RayCastQueryAsync(), C#](/dotnet/api/microsoft.azure.remoterendering.remotemanager.raycastqueryasync)
* [RemoteManager::RayCastQueryAsync(), C++](/cpp/api/remote-rendering/remotemanager#raycastqueryasync)

## <a name="next-steps"></a>Étapes suivantes

* [Limites des objets](../../concepts/object-bounds.md)
* [Remplacement des états hiérarchiques](override-hierarchical-state.md)