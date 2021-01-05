---
title: Modèles
description: Décrit ce qu’est un modèle dans Azure Remote Rendering
author: jakrams
ms.author: jakras
ms.date: 02/05/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: ff69486ab24c999e40b0afc13c91d6f729c352a0
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92206558"
---
# <a name="models"></a>Modèles

Un *modèle* dans Azure Remote Rendering fait référence à une représentation complète de l’objet, composée d’[entités](entities.md) et de [composants](components.md). Les modèles sont le principal moyen d’utiliser des données personnalisées dans le service de rendu distant.

## <a name="model-structure"></a>Structure du modèle

Un modèle a exactement une [entité](entities.md) comme nœud racine. En dessous, il peut avoir une hiérarchie arbitraire d’entités enfants. Lors du chargement d’un modèle, une référence à cette entité racine est retournée.

Chaque entité peut avoir des [composants](components.md) attachés. Dans le cas le plus courant, les entités ont des *MeshComponents*, qui font référence aux [ressources de maillage](meshes.md).

## <a name="creating-models"></a>Création de modèles

La création de modèles pour le runtime s’effectue en [convertissant les modèles d’entrée](../how-tos/conversion/model-conversion.md) à partir de formats de fichiers tels que FBX et GLTF. Le processus de conversion extrait toutes les ressources, telles que les textures, les matériaux et les maillages, et les convertit en formats de runtime optimisés. Il extraira également les informations structurelles et les convertira en structure graphique d’entité ou de composant ARR.

> [!IMPORTANT]
> La [conversion de modèle](../how-tos/conversion/model-conversion.md) est la seule façon de créer des [maillages](meshes.md). Bien que les maillages puissent être partagés entre les entités au moment de l’exécution, il n’existe aucun autre moyen d’obtenir un maillage dans le runtime, autre que le chargement d’un modèle.

## <a name="loading-models"></a>Chargement de modèles

Une fois qu’un modèle est converti, il peut être chargé à partir du stockage blob Azure dans le runtime.

Il existe deux fonctions de chargement distinctes qui diffèrent par la méthode d’adressage de la ressource dans le stockage blob :

* Le modèle peut être adressé par son URI SAS. La fonction de chargement appropriée est `LoadModelFromSASAsync` avec le paramètre `LoadModelFromSASParams`. Utilisez également cette méthode pour le chargement de [modèles intégrés](../samples/sample-model.md).
* Le modèle peut être adressé directement par le biais des paramètres de stockage blob si le [stockage blob est lié au compte](../how-tos/create-an-account.md#link-storage-accounts). Dans ce cas, la fonction de chargement appropriée est `LoadModelAsync` avec le paramètre `LoadModelParams`.

Les extraits de code suivants montrent comment charger des modèles avec l’une ou l’autre des fonctions. Pour charger un modèle à l’aide de l’URI SAS, utilisez un code similaire à celui ci-dessous :

```csharp
async void LoadModel(AzureSession session, Entity modelParent, string modelUri)
{
    // load a model that will be parented to modelParent
    var modelParams = new LoadModelFromSASParams(modelUri, modelParent);

    var loadOp = session.Actions.LoadModelFromSASAsync(modelParams);

    loadOp.ProgressUpdated += (float progress) =>
    {
        Debug.Log($"Loading: {progress * 100.0f}%");
    };

    await loadOp.AsTask();
}
```

```cpp
ApiHandle<LoadModelAsync> LoadModel(ApiHandle<AzureSession> session, ApiHandle<Entity> modelParent, std::string modelUri)
{
    LoadModelFromSASParams modelParams;
    modelParams.ModelUrl = modelUri;
    modelParams.Parent = modelParent;

    ApiHandle<LoadModelAsync> loadOp = *session->Actions()->LoadModelFromSASAsync(modelParams);

    loadOp->Completed([](const ApiHandle<LoadModelAsync>& async)
    {
        printf("Loading: finished.");
    });
    loadOp->ProgressUpdated([](float progress)
    {
        printf("Loading: %.1f%%", progress*100.f);
    });

    return loadOp;
}
```

Si vous souhaitez charger un modèle en utilisant directement ses paramètres de stockage blob, utilisez un code similaire à l’extrait de code suivant :

```csharp
async void LoadModel(AzureSession session, Entity modelParent, string storageAccount, string containerName, string assetFilePath)
{
    // load a model that will be parented to modelParent
    var modelParams = new LoadModelParams(
        storageAccount, // storage account name + '.blob.core.windows.net', e.g., 'mystorageaccount.blob.core.windows.net'
        containerName,  // name of the container in your storage account, e.g., 'mytestcontainer'
        assetFilePath,  // the file path to the asset within the container, e.g., 'path/to/file/myAsset.arrAsset'
        modelParent
    );

    var loadOp = session.Actions.LoadModelAsync(modelParams);

    // ... (identical to the SAS URI snippet above)
}
```

```cpp
ApiHandle<LoadModelAsync> LoadModel(ApiHandle<AzureSession> session, ApiHandle<Entity> modelParent, std::string storageAccount, std::string containerName, std::string assetFilePath)
{
    LoadModelParams modelParams;
    modelParams.Parent = modelParent;
    modelParams.Blob.StorageAccountName = std::move(storageAccount);
    modelParams.Blob.BlobContainerName = std::move(containerName);
    modelParams.Blob.AssetPath = std::move(assetFilePath);

    ApiHandle<LoadModelAsync> loadOp = *session->Actions()->LoadModelAsync(modelParams);
    // ... (identical to the SAS URI snippet above)
}
```

Par la suite, vous pouvez parcourir la hiérarchie d’entités et modifier les entités et les composants. Le chargement répété du même modèle crée plusieurs instances, chacune avec sa propre copie de la structure des entités et des composants. Étant donné que les maillages, les matériaux et les textures sont des [ressources partagées](../concepts/lifetime.md), leurs données ne seront pas rechargées. Par conséquent, l’instanciation d’un modèle plus d’une fois nécessite relativement peu de mémoire.

> [!CAUTION]
> Toutes les fonctions *Async* dans ARR retournent des objets d’opérations asynchrones. Vous devez stocker une référence à ces objets jusqu’à ce que l’opération soit terminée. Sinon, le récupérateur de mémoire C# peut supprimer l’opération prématurément et elle ne pourra jamais se terminer. Dans l’exemple de code ci-dessus, l’utilisation de *await* garantit que la variable locale « loadOp » contient une référence jusqu’à ce que le chargement du modèle soit terminé. Toutefois, si vous deviez utiliser à la place l’événement *Completed*, vous devez stocker l’opération asynchrone dans une variable membre.

## <a name="api-documentation"></a>Documentation de l’API

* [RemoteManager.LoadModelAsync(), C#](/dotnet/api/microsoft.azure.remoterendering.remotemanager.loadmodelasync)
* [RemoteManager.LoadModelFromSASAsync(), C#](/dotnet/api/microsoft.azure.remoterendering.remotemanager.loadmodelfromsasasync)
* [RemoteManager::LoadModelAsync(), C++](/cpp/api/remote-rendering/remotemanager#loadmodelasync)
* [RemoteManager::LoadModelFromSASAsync(), C++](/cpp/api/remote-rendering/remotemanager#loadmodelfromsasasync)

## <a name="next-steps"></a>Étapes suivantes

* [Entités](entities.md)
* [Maillages](meshes.md)