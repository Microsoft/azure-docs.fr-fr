---
title: Télécharger des éléments multimédias Media Services sur votre ordinateur - Azure | Microsoft Docs
description: Découvrez comment télécharger des éléments multimédias sur votre ordinateur. Les exemples de code sont écrits en C# et utilisent le Kit de développement logiciel (SDK) Media Services pour .NET.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 8908a1dd-3ffb-4f18-955d-4c8e2d82fc5d
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 5ad40feacdccc847ed66caed3d586db2490b0b50
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89266525"
---
# <a name="how-to-deliver-an-asset-by-download"></a>Procédure : Fourniture d’un élément multimédia par téléchargement

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Cet article présente les options disponibles pour fournir des éléments multimédias chargés sur Media Services. De nombreux scénarios d'application permettent de fournir du contenu Media Services. Après l’encodage, téléchargez les éléments multimédias générés ou accédez-y à l’aide d’un localisateur de diffusion en continu. Pour améliorer les performances et l’évolutivité, vous pouvez également fournir du contenu en utilisant un réseau de distribution de contenu (CDN).

Cet exemple montre comment télécharger des éléments multimédias depuis Media Services sur un ordinateur local. Le code lance une requête sur les tâches associées au compte Media Services par ID de tâche et accède à l'ensemble **OutputMediaAssets** du compte (qui regroupe un ou plusieurs éléments multimédias en sortie, suite à l'exécution d'une tâche). Cet exemple indique comment télécharger des éléments multimédias en sortie depuis une tâche, mais il est possible d'appliquer la même approche pour télécharger d'autres éléments.

>[!NOTE]
>Un nombre limite de 1 000 000 a été défini pour les différentes stratégies AMS (par exemple, pour la stratégie de localisateur ou pour ContentKeyAuthorizationPolicy). Utilisez le même ID de stratégie si vous utilisez toujours les mêmes jours/autorisations d’accès, par exemple, les stratégies pour les localisateurs destinés à demeurer en place pendant une longue période (stratégies sans chargement). Pour plus d’informations, consultez [cet](media-services-dotnet-manage-entities.md#limit-access-policies) article.

```csharp
    // Download the output asset of the specified job to a local folder.
    static IAsset DownloadAssetToLocal( string jobId, string outputFolder)
    {
        // This method illustrates how to download a single asset. 
        // However, you can iterate through the OutputAssets
        // collection, and download all assets if there are many. 

        // Get a reference to the job. 
        IJob job = GetJob(jobId);

        // Get a reference to the first output asset. If there were multiple 
        // output media assets you could iterate and handle each one.
        IAsset outputAsset = job.OutputMediaAssets[0];

        // Create a SAS locator to download the asset
        IAccessPolicy accessPolicy = _context.AccessPolicies.Create("File Download Policy", TimeSpan.FromDays(30), AccessPermissions.Read);
        ILocator locator = _context.Locators.CreateLocator(LocatorType.Sas, outputAsset, accessPolicy);

        BlobTransferClient blobTransfer = new BlobTransferClient
        {
            NumberOfConcurrentTransfers = 20,
            ParallelTransferThreadCount = 20
        };

        var downloadTasks = new List<Task>();
        foreach (IAssetFile outputFile in outputAsset.AssetFiles)
        {
            // Use the following event handler to check download progress.
            outputFile.DownloadProgressChanged += DownloadProgress;

            string localDownloadPath = Path.Combine(outputFolder, outputFile.Name);

            Console.WriteLine("File download path:  " + localDownloadPath);

            downloadTasks.Add(outputFile.DownloadAsync(Path.GetFullPath(localDownloadPath), blobTransfer, locator, CancellationToken.None));

            outputFile.DownloadProgressChanged -= DownloadProgress;
        }

        Task.WaitAll(downloadTasks.ToArray());

        return outputAsset;
    }

    static void DownloadProgress(object sender, DownloadProgressChangedEventArgs e)
    {
        Console.WriteLine(string.Format("{0} % download progress. ", e.Progress));
    }
```


## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>Voir aussi
[de diffusion de contenu en continu](media-services-deliver-streaming-content.md)

