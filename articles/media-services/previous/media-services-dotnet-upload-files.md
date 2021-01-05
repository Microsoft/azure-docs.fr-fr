---
title: Charger des fichiers dans un compte Media Services à l’aide de .NET | Microsoft Docs
description: Apprenez à obtenir du contenu multimédia dans Media Services en créant et en chargeant des ressources à l’aide de .NET.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: c9c86380-9395-4db8-acea-507c52066f73
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 83e9b0278e99867cafa7e633bc382e490ec273c1
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91250524"
---
# <a name="upload-files-into-a-media-services-account-using-net"></a>Charger des fichiers dans un compte Media Services à l’aide de .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Dans Media Services, vous téléchargez (ou réceptionnez) vos fichiers numériques dans un élément multimédia. L’entité **Asset** peut contenir des fichiers vidéo et audio, des images, des collections de miniatures, des pistes textuelles et des légendes (et les métadonnées concernant ces fichiers).  Une fois les fichiers téléchargés, votre contenu est stocké en toute sécurité dans le cloud et peut faire l’objet d’un traitement et d’une diffusion en continu.

Les fichiers de l'élément multimédia sont appelés **fichiers d'élément multimédia**. L'instance **AssetFile** et le fichier multimédia réel sont deux objets distincts. L’instance AssetFile contient des métadonnées concernant le fichier multimédia, tandis que le fichier multimédia contient le contenu multimédia réel.

## <a name="considerations"></a>Considérations

Les considérations suivantes s'appliquent :
 
 * Media Services utilise la valeur de la propriété IAssetFile.Name lors de la génération d’URL pour le contenu de streaming (par exemple, http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters). Pour cette raison, l’encodage par pourcentage n’est pas autorisé. La valeur de la propriété **Name** ne peut pas comporter les [caractères réservés à l’encodage en pourcentage suivants](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters) : !* '();:@&=+$,/?%#[]". En outre, il ne peut exister qu’un ’.’ pour l’extension de nom de fichier.
* La longueur du nom ne doit pas dépasser 260 caractères.
* Une limite est appliquée à la taille maximale de fichier prise en charge pour le traitement dans Media Services. Consultez [cet](media-services-quotas-and-limitations.md) article pour en savoir plus sur les limites de taille des fichiers.
* Un nombre limite de 1 000 000 a été défini pour les différentes stratégies AMS (par exemple, pour la stratégie de localisateur ou pour ContentKeyAuthorizationPolicy). Vous devez utiliser le même ID de stratégie si vous utilisez toujours les mêmes jours / autorisations d’accès, par exemple, les stratégies pour les localisateurs destinées à demeurer en place pendant une longue période (stratégies sans chargement). Pour plus d’informations, consultez [cet](media-services-dotnet-manage-entities.md#limit-access-policies) article.

Lorsque vous créez des éléments multimédias, vous pouvez spécifier les options de chiffrement suivantes :

* **None** : aucun chiffrement. Il s’agit de la valeur par défaut. Quand vous utilisez cette option, votre contenu n'est pas protégé pendant le transit ou le repos dans le stockage.
  Si vous prévoyez de fournir un MP4 sous forme de téléchargement progressif, utilisez cette option : 
* **CommonEncryption** : utilisez cette option quand vous téléchargez du contenu qui a déjà été chiffré et protégé par chiffrement commun ou gestion des droits numériques (DRM) PlayReady (par exemple, une diffusion en continu lisse, « Smooth Streaming », protégée par gestion des droits numériques (DRM) PlayReady).
* **EnvelopeEncrypted** : utilisez cette option quand vous téléchargez du contenu au format HLS chiffré avec AES. Notez que les fichiers doivent avoir été encodés et chiffrés par le gestionnaire de transformation Transform Manager.
* **StorageEncrypted** : permet de chiffrer votre contenu en clair localement en utilisant le chiffrement AES-256 bits, puis de le télécharger vers Azure Storage où il est chiffré pour le stockage, au repos. Les éléments multimédias protégés par le chiffrement de stockage sont automatiquement déchiffrés et placés dans un système de fichiers chiffré avant d’être encodés, puis éventuellement rechiffrés avant d’être rechargés sous la forme d’un nouvel élément multimédia de sortie. Le principal cas d'utilisation du chiffrement de stockage concerne la sécurisation de fichiers multimédias d'entrée de haute qualité avec un chiffrement renforcé au repos sur le disque.
  
    Media Services fournit pour vos éléments multimédias un chiffrement de stockage sur disque, et non pas sur le réseau comme pour la gestion des droits numériques (DRM).
  
    Si votre ressource est stockée sous forme chiffrée, vous devez configurer une stratégie de remise de ressources. Pour plus d'informations, consultez [Configuration de la stratégie de remise de ressources](media-services-dotnet-configure-asset-delivery-policy.md).

Si votre élément multimédia est chiffré avec l’option **CommonEncrypted** ou **EnvelopeEncrypted**, vous devez l’associer à une **ContentKey**. Pour plus d'informations, consultez [Comment créer une ContentKey](media-services-dotnet-create-contentkey.md) 

Si votre élément multimédia est chiffré avec l’option **StorageEncrypted**, le SDK Media Services pour .NET crée une **ContentKey** **StorageEncrypted** pour votre élément multimédia.

Cet article montre comment utiliser le Kit de développement logiciel (SDK) Media Services pour .NET, ainsi que les extensions du SDK Media Services pour .NET, pour charger des fichiers vers un élément multimédia Media Services.

## <a name="upload-a-single-file-with-media-services-net-sdk"></a>Téléchargement d’un fichier unique avec le Kit de développement logiciel (SDK) .NET de Media Services

Le code suivant utilise .NET pour charger un fichier unique. AccessPolicy et Locator sont créés et détruits par la fonction de chargement. 

```csharp
        static public IAsset CreateAssetAndUploadSingleFile(AssetCreationOptions assetCreationOptions, string singleFilePath)
        {
            if (!File.Exists(singleFilePath))
            {
                Console.WriteLine("File does not exist.");
                return null;
            }

            var assetName = Path.GetFileNameWithoutExtension(singleFilePath);
            IAsset inputAsset = _context.Assets.Create(assetName, assetCreationOptions);

            var assetFile = inputAsset.AssetFiles.Create(Path.GetFileName(singleFilePath));

            Console.WriteLine("Upload {0}", assetFile.Name);

            assetFile.Upload(singleFilePath);
            Console.WriteLine("Done uploading {0}", assetFile.Name);

            return inputAsset;
        }
```


## <a name="upload-multiple-files-with-media-services-net-sdk"></a>Téléchargement de plusieurs fichiers avec le Kit de développement logiciel (SDK) .NET de Media Services
Le code qui suit présente la création d’un élément multimédia et le chargement de plusieurs fichiers.

Le code effectue les actions suivantes :

* Création d’un élément multimédia vide à l’aide de la méthode CreateEmptyAsset définie dans l’étape précédente.
* Création d'une instance **AccessPolicy** qui définit les autorisations et la durée de l'accès à l'élément multimédia.
* Création d'une instance **Locator** qui fournit l'accès à l'élément multimédia.
* Création d'une instance **BlobTransferClient**. Ce type représente un client qui opère sur les objets blob Azure. Dans cet exemple, le client surveille la progression du chargement. 
* Énumère les fichiers dans le répertoire spécifié et crée une instance **AssetFile** pour chaque fichier.
* Télécharge les fichiers dans Media Services à l'aide de la méthode **UploadAsync** . 

> [!NOTE]
> Utilisez la méthode UploadAsync afin de garantir que les appels ne sont pas bloqués et que les fichiers sont téléchargés en parallèle.
> 
> 

```csharp
        static public IAsset CreateAssetAndUploadMultipleFiles(AssetCreationOptions assetCreationOptions, string folderPath)
        {
            var assetName = "UploadMultipleFiles_" + DateTime.UtcNow.ToString();

            IAsset asset = _context.Assets.Create(assetName, assetCreationOptions);

            var accessPolicy = _context.AccessPolicies.Create(assetName, TimeSpan.FromDays(30),
                                                                AccessPermissions.Write | AccessPermissions.List);

            var locator = _context.Locators.CreateLocator(LocatorType.Sas, asset, accessPolicy);

            var blobTransferClient = new BlobTransferClient();
            blobTransferClient.NumberOfConcurrentTransfers = 20;
            blobTransferClient.ParallelTransferThreadCount = 20;

            blobTransferClient.TransferProgressChanged += blobTransferClient_TransferProgressChanged;

            var filePaths = Directory.EnumerateFiles(folderPath);

            Console.WriteLine("There are {0} files in {1}", filePaths.Count(), folderPath);

            if (!filePaths.Any())
            {
                throw new FileNotFoundException(String.Format("No files in directory, check folderPath: {0}", folderPath));
            }

            var uploadTasks = new List<Task>();
            foreach (var filePath in filePaths)
            {
                var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
                Console.WriteLine("Created assetFile {0}", assetFile.Name);

                // It is recommended to validate AssetFiles before upload. 
                Console.WriteLine("Start uploading of {0}", assetFile.Name);
                uploadTasks.Add(assetFile.UploadAsync(filePath, blobTransferClient, locator, CancellationToken.None));
            }

            Task.WaitAll(uploadTasks.ToArray());
            Console.WriteLine("Done uploading the files");

            blobTransferClient.TransferProgressChanged -= blobTransferClient_TransferProgressChanged;

            locator.Delete();
            accessPolicy.Delete();

            return asset;
        }

    static void  blobTransferClient_TransferProgressChanged(object sender, BlobTransferProgressChangedEventArgs e)
    {
        if (e.ProgressPercentage > 4) // Avoid startup jitter, as the upload tasks are added.
        {
            Console.WriteLine("{0}% upload competed for {1}.", e.ProgressPercentage, e.LocalFile);
        }
    }
```


Lorsque vous chargez un grand nombre d’éléments multimédias, prenez en compte les points suivants :

* Créez un objet **CloudMediaContext** par thread. La classe **CloudMediaContext** n’est pas thread-safe.
* Augmentez la valeur par défaut (2) de NumberOfConcurrentTransfers à une valeur supérieure à 5. Cette propriété affecte toutes les instances de **CloudMediaContext**. 
* Conservez la valeur par défaut de 10 pour ParallelTransferThreadCount.

## <a name="ingesting-assets-in-bulk-using-media-services-net-sdk"></a><a id="ingest_in_bulk"></a>Réception d’éléments multimédias en bloc à l’aide du Kit de développement logiciel (SDK) .NET de Media Services
Le téléchargement de fichiers multimédias volumineux peut entraîner un goulot d’étranglement lors de la création de l'élément multimédia. La réception des éléments multimédias en bloc ou « réception en bloc » implique de découpler la création des éléments multimédias du processus de téléchargement. Pour utiliser une approche de réception en bloc, créez un manifeste (IngestManifest) qui décrit l'élément multimédia et ses fichiers associés. Utilisez ensuite la méthode de téléchargement de votre choix pour télécharger les fichiers associés sur le conteneur d’objets blob du manifeste. Microsoft Azure Media Services surveille le conteneur d’objets blob associé au manifeste. Une fois qu’un fichier est téléchargé vers le conteneur d’objets blob, Microsoft Azure Media Services termine la création des éléments multimédias selon la configuration de l'élément multimédia du manifeste (IngestManifestAsset).

Pour créer un IngestManifest, appelez la méthode Create exposée par la collection IngestManifests sur le CloudMediaContext. Cette méthode crée un IngestManifest avec le nom de manifeste que vous fournissez.

```csharp
    IIngestManifest manifest = context.IngestManifests.Create(name);
```

Créez les éléments multimédias associés à l’IngestManifest en bloc. Configurez les options de chiffrement souhaitées sur l'élément multimédia pour la réception en bloc.

```csharp
    // Create the assets that will be associated with this bulk ingest manifest
    IAsset destAsset1 = _context.Assets.Create(name + "_asset_1", AssetCreationOptions.None);
    IAsset destAsset2 = _context.Assets.Create(name + "_asset_2", AssetCreationOptions.None);
```

Un IngestManifestAsset associe un élément multimédia à un IngestManifest en bloc pour la réception en bloc. Il associe également les AssetFiles qui constituent chaque élément multimédia. Pour créer un IngestManifestAsset, utilisez la méthode Create dans le contexte de serveur.

L’exemple suivant illustre l’ajout de deux IngestManifestAssets qui associent les deux éléments multimédias précédemment créés au manifeste de réception en bloc. Chaque IngestManifestAsset associe également un ensemble de fichiers qui sont chargés pour chaque élément multimédia lors de la réception en bloc.  

```csharp
    string filename1 = _singleInputMp4Path;
    string filename2 = _primaryFilePath;
    string filename3 = _singleInputFilePath;

    IIngestManifestAsset bulkAsset1 =  manifest.IngestManifestAssets.Create(destAsset1, new[] { filename1 });
    IIngestManifestAsset bulkAsset2 =  manifest.IngestManifestAssets.Create(destAsset2, new[] { filename2, filename3 });
```

Vous pouvez utiliser n’importe quelle application cliente rapide capable de charger les fichiers d’éléments multimédias sur l’URI du conteneur de stockage blob fourni par la propriété **IIngestManifest.BlobStorageUriForUpload** de l’IngestManifest. 

Le code suivant montre comment utiliser le SDK .NET pour charger les fichiers de ressources.

```csharp
    static void UploadBlobFile(string containerName, string filename)
    {
        Task copytask = new Task(() =>
        {
            var storageaccount = new CloudStorageAccount(new StorageCredentials(_storageAccountName, _storageAccountKey), true);
            CloudBlobClient blobClient = storageaccount.CreateCloudBlobClient();
            CloudBlobContainer blobContainer = blobClient.GetContainerReference(containerName);

            string[] splitfilename = filename.Split('\\');
            var blob = blobContainer.GetBlockBlobReference(splitfilename[splitfilename.Length - 1]);

            using (var stream = System.IO.File.OpenRead(filename))
                blob.UploadFromStream(stream);

            lock (consoleWriteLock)
            {
                Console.WriteLine("Upload for {0} completed.", filename);
            }
        });

        copytask.Start();
    }
```

Le code pour charger les fichiers d’éléments multimédias de l’exemple utilisé dans cet article est illustré dans l’exemple de code suivant :

```csharp
    UploadBlobFile(manifest.BlobStorageUriForUpload, filename1);
    UploadBlobFile(manifest.BlobStorageUriForUpload, filename2);
    UploadBlobFile(manifest.BlobStorageUriForUpload, filename3);
```

Vous pouvez déterminer la progression de la réception en bloc de tous les éléments multimédias associés à un **IngestManifest** en interrogeant la propriété Statistics de **l’IngestManifest**. Pour mettre à jour les informations de progression, vous devez utiliser un nouveau **CloudMediaContext** chaque fois que vous interrogez la propriété Statistics.

L'exemple suivant illustre l'interrogation d'un IngestManifest par son **Id**.

```csharp
    static void MonitorBulkManifest(string manifestID)
    {
       bool bContinue = true;
       while (bContinue)
       {
          CloudMediaContext context = GetContext();
          IIngestManifest manifest = context.IngestManifests.Where(m => m.Id == manifestID).FirstOrDefault();

          if (manifest != null)
          {
             lock(consoleWriteLock)
             {
                Console.WriteLine("\nWaiting on all file uploads.");
                Console.WriteLine("PendingFilesCount  : {0}", manifest.Statistics.PendingFilesCount);
                Console.WriteLine("FinishedFilesCount : {0}", manifest.Statistics.FinishedFilesCount);
                Console.WriteLine("{0}% complete.\n", (float)manifest.Statistics.FinishedFilesCount / (float)(manifest.Statistics.FinishedFilesCount + manifest.Statistics.PendingFilesCount) * 100);

                if (manifest.Statistics.PendingFilesCount == 0)
                {
                   Console.WriteLine("Completed\n");
                   bContinue = false;
                }
             }

             if (manifest.Statistics.FinishedFilesCount < manifest.Statistics.PendingFilesCount)
                Thread.Sleep(60000);
          }
          else // Manifest is null
             bContinue = false;
       }
    }
```


## <a name="upload-files-using-net-sdk-extensions"></a>Téléchargement de fichiers à l’aide des extensions du Kit de développement logiciel (SDK) .NET
L’exemple suivant montre comment charger un fichier unique à l’aide des extensions du Kit de développement logiciel (SDK) .NET. Dans ce cas, on utilise la méthode **CreateFromFile**, mais la version asynchrone est également disponible (**CreateFromFileAsync**). La méthode **CreateFromFile** vous permet de spécifier le nom de fichier, l'option de chiffrement et un rappel pour signaler la progression du téléchargement du fichier.

```csharp
    static public IAsset UploadFile(string fileName, AssetCreationOptions options)
    {
        IAsset inputAsset = _context.Assets.CreateFromFile(
            fileName,
            options,
            (af, p) =>
            {
                Console.WriteLine("Uploading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
            });

        Console.WriteLine("Asset {0} created.", inputAsset.Id);

        return inputAsset;
    }
```

L’exemple suivant appelle la fonction UploadFile et spécifie le chiffrement de stockage en tant qu’option de création d'éléments multimédias.  

```csharp
    var asset = UploadFile(@"C:\VideoFiles\BigBuckBunny.mp4", AssetCreationOptions.StorageEncrypted);
```

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez désormais encoder vos éléments multimédias téléchargés. Pour plus d'informations, consultez [Encode an asset using Media Encoder Standard with the Azure portal (Encoder un élément multimédia à l’aide de Media Encoder Standard avec le portail Azure)](media-services-portal-encode.md).

Vous pouvez également utiliser les fonctions Azure pour déclencher une tâche de codage à partir d’un fichier entrant dans le conteneur configuré. Pour plus d’informations, consultez [cet exemple](https://azure.microsoft.com/resources/samples/media-services-dotnet-functions-integration/ ).

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-step"></a>Étape suivante
Après avoir chargé un élément multimédia dans Media Services, consultez l’article [Guide pratique pour obtenir un processeur multimédia][How to Get a Media Processor].

[How to Get a Media Processor]: media-services-get-media-processor.md
