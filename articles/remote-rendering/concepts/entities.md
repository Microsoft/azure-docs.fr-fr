---
title: Entités
description: Définition d’entités dans l’étendue de l’API Azure Remote Rendering
author: florianborn71
ms.author: flborn
ms.date: 02/03/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: bfcfa4c5ed57489c56ebf845d238198944150a96
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92202886"
---
# <a name="entities"></a>Entités

Une *entité* représente un objet déplaçable dans l’espace et constitue le bloc de construction de base d’un contenu rendu à distance.

## <a name="entity-properties"></a>Propriétés d’entité

Les entités ont une transformation définie par une position, une rotation et une échelle. En soi, les entités n’ont pas de fonctionnalité observable. Au lieu de cela, leur comportement est ajouté au travers de composants attachés à des entités. Par exemple, l’attachement du composant [CutPlaneComponent](../overview/features/cut-planes.md) crée un plan de coupe à la position de l’entité.

L’aspect le plus important de l’entité elle-même est la hiérarchie et la transformation hiérarchique qui en résulte. Par exemple, quand plusieurs entités sont jointes en tant qu’enfants à une entité parente partagée, toutes ces entités peuvent être déplacées, pivotées et mises à l’échelle en modifiant la transformation de l’entité parente. De même, l’état `enabled` de l’entité peut être utilisé pour désactiver la visibilité et les réponses aux ray casts pour un sous-graphique complet de la hiérarchie.

Une entité est la propriété exclusive de son entité parente. Cela signifie que, quand celle-ci est détruite à l’aide de la commande `Entity.Destroy()`, ses entités enfants et tous les composants [connectés](components.md) le sont également. Ainsi, la suppression d’un modèle de la scène s’effectue en appelant `Destroy` sur le nœud racine d’un modèle retourné par la commande `AzureSession.Actions.LoadModelAsync()` ou sa variante SAP `AzureSession.Actions.LoadModelFromSASAsync()`.

Des entités sont créées quand le serveur charge du contenu ou quand l’utilisateur souhaite ajouter un objet à la scène. Par exemple, si un utilisateur souhaite ajouter un plan de coupe pour visualiser l’intérieur d’un maillage, l’utilisateur peut créer une entité dans laquelle le plan doit exister, puis y ajouter le composant plan de coupe.

## <a name="create-an-entity"></a>Créer une entité

Pour ajouter une nouvelle entité à la scène, par exemple, afin de la transmettre en tant qu’objet racine pour le chargement de modèles ou d’y joindre des composants, utilisez le code suivant :

```cs
Entity CreateNewEntity(AzureSession session)
{
    Entity entity = session.Actions.CreateEntity();
    entity.Position = new LocalPosition(1, 2, 3);
    return entity;
}
```

```cpp
ApiHandle<Entity> CreateNewEntity(ApiHandle<AzureSession> session)
{
    ApiHandle<Entity> entity(nullptr);
    if (auto entityRes = session->Actions()->CreateEntity())
    {
        entity = entityRes.value();
        entity->SetPosition(Double3{ 1, 2, 3 });
        return entity;
    }
    return entity;
}
```

## <a name="query-functions"></a>Fonctions de requête

Il existe deux types de fonctions de requête sur les entités : les appels synchrones et asynchrones. Les requêtes synchrones ne peuvent être utilisées que pour des données présentes sur le client et n’impliquent pas beaucoup de calcul. Il s’agit, par exemple, de requêtes portant sur des composants, des transformations d’objets relatifs ou des relations parent/enfant. Les requêtes asynchrones sont utilisées pour des données qui résident uniquement sur le serveur ou qui impliquent un calcul supplémentaire dont l’exécution sur le client serait trop coûteuse. Il s’agit, par exemple, de requêtes portant sur des limites spatiales ou des métadonnées.

### <a name="querying-components"></a>Interrogation de composants

Pour rechercher un composant d’un type spécifique, utilisez la commande `FindComponentOfType` :

```cs
CutPlaneComponent cutplane = (CutPlaneComponent)entity.FindComponentOfType(ObjectType.CutPlaneComponent);

// or alternatively:
CutPlaneComponent cutplane = entity.FindComponentOfType<CutPlaneComponent>();
```

```cpp
ApiHandle<CutPlaneComponent> cutplane = entity->FindComponentOfType(ObjectType::CutPlaneComponent)->as<CutPlaneComponent>();

// or alternatively:
ApiHandle<CutPlaneComponent> cutplane = entity->FindComponentOfType<CutPlaneComponent>();
```

### <a name="querying-transforms"></a>Interrogation de transformations

Les requêtes de transformation sont des appels synchrones sur l’objet. Il est important de noter que les transformations interrogées via l’API sont des transformations d’espace local, par rapport au parent de l’objet. Y font exception les objets racine, pour lesquels l’espace local et l’espace universel sont identiques.

> [!NOTE]
> Il n’existe aucune API dédiée pour interroger la transformation d’espace universel d’objets arbitraires.

```cs
// local space transform of the entity
Double3 translation = entity.Position;
Quaternion rotation = entity.Rotation;
```

```cpp
// local space transform of the entity
Double3 translation = entity->GetPosition();
Quaternion rotation = entity->GetRotation();
```

### <a name="querying-spatial-bounds"></a>Interrogation des limites spatiales

Les requêtes de limites sont des appels asynchrones qui opèrent sur une hiérarchie d’objets complète, en utilisant une entité comme racine. Reportez-vous au chapitre dédié aux [limites d’objets](object-bounds.md).

### <a name="querying-metadata"></a>Interrogation des métadonnées

Les métadonnées sont des données supplémentaires stockées sur des objets, que le serveur ignore. Des métadonnées d’objet sont essentiellement un ensemble de paires (nom, valeur), où la _valeur_ peut être de type numérique, booléen ou chaîne. Les métadonnées peuvent être exportées avec le modèle.

Les requêtes de métadonnées sont des appels asynchrones sur une entité spécifique. La requête retourne uniquement les métadonnées d’une entité, non les informations fusionnées d’un sous-graphe.

```cs
MetadataQueryAsync metaDataQuery = entity.QueryMetaDataAsync();
metaDataQuery.Completed += (MetadataQueryAsync query) =>
{
    if (query.IsRanToCompletion)
    {
        ObjectMetaData metaData = query.Result;
        ObjectMetaDataEntry entry = metaData.GetMetadataByName("MyInt64Value");
        System.Int64 intValue = entry.AsInt64;

        // ...
    }
};
```

```cpp
ApiHandle<MetadataQueryAsync> metaDataQuery = *entity->QueryMetaDataAsync();
metaDataQuery->Completed([](const ApiHandle<MetadataQueryAsync>& query)
    {
        if (query->GetIsRanToCompletion())
        {
            ApiHandle<ObjectMetaData> metaData = query->GetResult();
            ApiHandle<ObjectMetaDataEntry> entry = *metaData->GetMetadataByName("MyInt64Value");
            int64_t intValue = *entry->GetAsInt64();

            // ...
        }
    });
```

La requête fonctionne même si l’objet ne contient pas de métadonnées.

## <a name="api-documentation"></a>Documentation de l’API

* [Entity, classe C#](/dotnet/api/microsoft.azure.remoterendering.entity)
* [RemoteManager.CreateEntity(), C#](/dotnet/api/microsoft.azure.remoterendering.remotemanager.createentity)
* [Entity, classe C++](/cpp/api/remote-rendering/entity)
* [RemoteManager::CreateEntity(), C++](/cpp/api/remote-rendering/remotemanager#createentity)

## <a name="next-steps"></a>Étapes suivantes

* [Composants](components.md)
* [Limites d’objets](object-bounds.md)