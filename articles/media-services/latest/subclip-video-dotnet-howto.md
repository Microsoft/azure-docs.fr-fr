---
title: Sous-découper une vidéo lors de l’encodage avec Azure Media Services
description: Cette rubrique décrit comment sous-découper une vidéo lors de l’encodage avec Azure Media Services à l’aide du Kit de développement logiciel (SDK) .NET
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: how-to
ms.date: 06/09/2019
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 1725fab30ddb4d40e0bf5345683588c878658052
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89296564"
---
# <a name="subclip-a-video-when-encoding-with-media-services---net"></a>Sous-découper une vidéo lors de l’encodage avec Media Services - .NET

Vous pouvez découper ou sous-découper une vidéo lors de son encodage à l’aide d’un [travail](/rest/api/media/jobs). Cette fonctionnalité peut être utilisée avec n’importe quelle [transformation](/rest/api/media/transforms) qui est générée en utilisant les présélections [BuiltInStandardEncoderPreset](/rest/api/media/transforms/createorupdate#builtinstandardencoderpreset) ou [StandardEncoderPreset](/rest/api/media/transforms/createorupdate#standardencoderpreset).

L’exemple C# suivant crée un travail qui découpe une vidéo dans un élément multimédia à mesure qu’il envoie un travail d’encodage. 

## <a name="prerequisites"></a>Prérequis

Pour suivre les étapes décrites dans cette rubrique, vous devez :

- [Créer un compte Azure Media Services](./create-account-howto.md)
- Créez une transformation, ainsi que les éléments multimédias d’entrée et de sortie. Vous pouvez voir comment créer une transformation et des éléments multimédias d’entrée et de sortie dans le didacticiel [Upload, encode, and stream videos using .NET](stream-files-tutorial-with-api.md) (Charger, encoder et diffuser en continu des vidéos à l’aide de .NET).
- Consultez la rubrique dédiée au [concept d’encodage](encoding-concept.md).

## <a name="example"></a>Exemple

```csharp
/// <summary>
/// Submits a request to Media Services to apply the specified Transform to a given input video.
/// </summary>
/// <param name="client">The Media Services client.</param>
/// <param name="resourceGroupName">The name of the resource group within the Azure subscription.</param>
/// <param name="accountName"> The Media Services account name.</param>
/// <param name="transformName">The name of the transform.</param>
/// <param name="jobName">The (unique) name of the job.</param>
/// <param name="inputAssetName">The name of the input asset.</param>
/// <param name="outputAssetName">The (unique) name of the  output asset that will store the result of the encoding job. </param>
// <SubmitJob>
private static async Task<Job> JobWithBuiltInStandardEncoderWithSingleClipAsync(
    IAzureMediaServicesClient client,
    string resourceGroupName,
    string accountName,
    string transformName,
    string jobName,
    string inputAssetName,
    string outputAssetName)
{
    var jobOutputs = new List<JobOutputAsset>
    {
        new JobOutputAsset(state: JobState.Queued, progress: 0, assetName: outputAssetName)
    };

    var clipStart = new AbsoluteClipTime()
    {
        Time = new TimeSpan(0, 0, 20)
    };

    var clipEnd = new AbsoluteClipTime()
    {
        Time = new TimeSpan(0, 0, 30)
    };

    var jobInput = new JobInputAsset(assetName: inputAssetName, start: clipStart, end: clipEnd);

    Job job = await client.Jobs.CreateAsync(
        resourceGroupName,
        accountName,
        transformName,
        jobName,
        new Job(input: jobInput, outputs: jobOutputs.ToArray(), name: jobName)
        {
            Description = $"A Job with transform {transformName} and single clip.",
            Priority = Priority.Normal,
        });

    return job;
}
```

## <a name="next-steps"></a>Étapes suivantes

[Comment encoder avec une transformation personnalisée](customize-encoder-presets-how-to.md) 
