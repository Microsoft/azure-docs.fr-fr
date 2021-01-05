---
title: Composants et objets de jeu Unity
description: Décrit les méthodes spécifiques à Unity pour utiliser des entités et composants Remote Rendering.
author: jakrams
ms.author: jakras
ms.date: 02/28/2020
ms.topic: how-to
ms.custom: devx-track-csharp
ms.openlocfilehash: 662c28196b06f5fbe49f69cb7145fdd33805e000
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89019043"
---
# <a name="interact-with-unity-game-objects-and-components"></a>Interagir avec les composants et objets de jeu Unity

Azure Remote Rendering (ARR) est optimisé pour un grand nombre d’objets (cf. [Limitations](../../reference/limits.md)). Même s’il est possible de gérer des hiérarchies volumineuses et complexes sur l’hôte, il est impossible de toutes les répliquer dans Unity sur des appareils de faible puissance.

Ainsi, quand un modèle est chargé sur l’hôte, Azure Remote Rendering met en miroir les informations sur la structure du modèle sur l’appareil client (ce qui entraîne un trafic réseau), mais ne réplique pas les objets et les composants dans Unity. Au lieu de cela, il s’attend à ce que vous demandiez les composants et objets de jeu Unity manuellement pour limiter la surcharge à ce qui est réellement nécessaire. Vous avez ainsi davantage de contrôle sur les performances côté client.

L’intégration à Unity d’Azure Remote Rendering offre donc une fonctionnalité supplémentaire permettant de répliquer la structure Remote Rendering à la demande.

## <a name="load-a-model-in-unity"></a>Charger un modèle dans Unity

Quand vous chargez un modèle, vous obtenez une référence à l’objet racine du modèle chargé. Cette référence n’est pas un objet de jeu Unity. Mais vous pouvez en faire un tel objet en utilisant la méthode d’extension `Entity.GetOrCreateGameObject()`. Cette fonction attend un argument de type `UnityCreationMode`. Si vous transmettez `CreateUnityComponents`, le nouvel objet de jeu Unity sera également renseigné avec les composants de proxy pour tous les composants Remote Rendering qui existent sur l’hôte. Toutefois, il est recommandé de privilégier `DoNotCreateUnityComponents` pour minimiser la surcharge.

### <a name="load-model-with-task"></a>Charger un modèle avec une tâche

```cs
LoadModelAsync _pendingLoadTask = null;
void LoadModelWithTask()
{
    _pendingLoadTask = RemoteManagerUnity.CurrentSession.Actions.LoadModelFromSASAsync(new LoadModelFromSASParams("builtin://Engine"));

    _pendingLoadTask.Completed += (LoadModelAsync res) =>
    {
        // turn the root object into a Unity game object
        var gameObject = res.Result.Root?.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
        _pendingLoadTask = null;
    };

    // also listen to progress updates:
    _pendingLoadTask.ProgressUpdated += (float progress) =>
    {
        // progress is a fraction in [0..1] range
        int percentage = (int)(progress * 100.0f);
        // do something...
        // Since the updates are triggered by the main thread, we may access unity objects here.
    };
}
```

### <a name="load-model-with-unity-coroutines"></a>Charger un modèle avec les coroutines Unity

```cs
IEnumerator LoadModelWithCoroutine()
{
    LoadModelAsync task = RemoteManagerUnity.CurrentSession.Actions.LoadModelFromSASAsync(new LoadModelFromSASParams("builtin://Engine"));

    while (!task.IsCompleted)
    {
        int percentage = (int)(task.Progress * 100.0f);
        yield return null;
    }

    if (!task.IsFaulted)
    {
        var gameObject = task.Result.Root?.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
    }

    task = null;
}
```

### <a name="load-model-with-await-pattern"></a>Charger un modèle avec le modèle d’awaiter

```cs
async void LoadModelWithAwait()
{
    var result = await RemoteManagerUnity.CurrentSession.Actions.LoadModelFromSASAsync(new LoadModelFromSASParams("builtin://Engine")).AsTask();
    var gameObject = result.Root?.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
}
```

Les exemples de code ci-dessus utilisent le chemin de chargement du modèle par le biais d’une SAS, car le modèle intégré est chargé. L’envoi du modèle par le biais de conteneurs d’objets Blob (avec `LoadModelAsync` et `LoadModelParams`) fonctionne exactement de la même manière.

## <a name="remoteentitysyncobject"></a>RemoteEntitySyncObject

La création d’un objet de jeu Unity ajoute implicitement un composant `RemoteEntitySyncObject` à l’objet de jeu. Ce composant est utilisé pour synchroniser la transformation d’entité avec le serveur. Par défaut, `RemoteEntitySyncObject` implique que l’utilisateur appelle explicitement `SyncToRemote()` pour synchroniser l’état local dans Unity avec le serveur. L’activation de `SyncEveryFrame` synchronise automatiquement l’objet.

Les enfants distants des objets avec un `RemoteEntitySyncObject` peuvent être instanciés et vous pouvez les afficher dans l’éditeur Unity à l’aide du bouton **:::no-loc text="Show children":::** .

![RemoteEntitySyncObject](media/remote-entity-sync-object.png)

## <a name="wrapper-components"></a>Composants wrapper

Les [composants](../../concepts/components.md) attachés aux entités Remote Rendering sont exposés sur Unity par le biais de proxys `MonoBehavior`. Ces proxys représentent le composant distant dans Unity et transfèrent toutes les modifications à l’hôte.

Pour créer des composants Remote Rendering proxys, utilisez la méthode d’extension `GetOrCreateArrComponent` :

```cs
var cutplane = gameObject.GetOrCreateArrComponent<ARRCutPlaneComponent>(RemoteManagerUnity.CurrentSession);
```

## <a name="coupled-lifetimes"></a>Durées de vie couplées

Les durées de vie d’une entité [distante](../../concepts/entities.md) et d’un objet de jeu Unity sont couplées quand ces derniers sont liés avec `RemoteEntitySyncObject`. Si vous appelez `UnityEngine.Object.Destroy(...)` avec un objet de jeu de ce type, l’entité distante est également supprimée.

Pour détruire l’objet de jeu Unity sans affecter l’entité distante, vous devez d’abord appeler `Unbind()` sur le `RemoteEntitySyncObject`.

Il en va de même pour tous les composants de proxy. Pour détruire uniquement la représentation côté client, vous devez d’abord appeler `Unbind()` sur le composant de proxy :

```cs
var cutplane = gameObject.GetComponent<ARRCutPlaneComponent>();
if (cutplane != null)
{
    cutplane.Unbind();
    UnityEngine.Object.Destroy(cutplane);
}
```

## <a name="next-steps"></a>Étapes suivantes

* [Configurer Remote Rendering pour Unity](unity-setup.md)
* [Tutoriel : Manipulation d’entités distantes dans Unity](../../tutorials/unity/manipulate-models/manipulate-models.md)
